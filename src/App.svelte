<script>
	import Footer from './footer.svelte'
	import {location, querystring} from 'svelte-spa-router'

	let ObjetoInfoErgo = {}
	let infoWallet = {id: '', name: '', address: ''}
	let arrayWallets = []
	let valorWallet = ''
	let nameWallet = ''
	let objetoAddress = {}
	let objetoToken = {}
	let arrayTokens = []
	let totalErgos = 0
	let valorIdWallet = ''
	let claseNoImage = ' border border-secondary'
	const loadCoin = async() => {
		const res = await fetch(`https://api.ergoplatform.com/api/v0/info`)
		const data = await res.json()
		if (res.ok){
			ObjetoInfoErgo = {
				hashRateErgo: data.hashRate,
				transactionsErgo: data.transactionAverage,
				totalCoinsErgo: new Intl.NumberFormat().format((data.supply/1000000000)),
			}
		}
		const res2 = await fetch(`https://api.nanopool.org/v1/ergo/prices`)
		const data2 = await res2.json()
		if (res2.ok){
			ObjetoInfoErgo.EurErgo = data2.data.price_eur
			ObjetoInfoErgo.UsdErgo = data2.data.price_usd
		}
	}
	loadCoin()
	if (localStorage.getItem("arrayWallets")){
		arrayWallets = JSON.parse(localStorage.getItem("arrayWallets"))
	}
	$: localStorage.setItem("arrayWallets", JSON.stringify(arrayWallets))
	// Add wallets
	const addWallet = async() => {
		// Si la consulta devuelve ok 200 es verdadera
		const res = await fetch(`https://api.ergoplatform.com/api/v1/addresses/${valorWallet}/balance/confirmed`)
		if (res.ok){
			const data = await res.json()
			infoWallet.id = Date.now()
			infoWallet.address = valorWallet
			infoWallet.name = nameWallet
			infoWallet.ergos = (data.nanoErgs/1000000000).toFixed(2)
			arrayWallets = [...arrayWallets, infoWallet]
			infoWallet = {id: '', name: '', address: '', ergos: ''}
			loadWallet()
		}else {
			alert('Wallet incorrect!')
		}
	}
	// Load wallet
	const loadWallet = async() => {
		const res = await fetch(`https://api.ergoplatform.com/api/v1/addresses/${valorWallet}/balance/confirmed`)
		const data = await res.json()
		if (res.ok){
			// Objeto datos confirmados
			objetoAddress = {
				nanoErgsConf: data.nanoErgs,
				numberTokensConf: data.tokens.length,
			}
			// Si hay tokens confirmados
			if (data.tokens.length){
			 	arrayTokens = []
				for (let i = 0; i < data.tokens.length; i++) {
					//const res2 = await fetch(`https://api.ergoplatform.com/api/v1/tokens/${data.confirmed.tokens[i].tokenId}`)
					//const res2 = await fetch(`https://api.ergoplatform.com/api/v1/boxes/byTokenId/${data.confirmed.tokens[i].tokenId}`)
					const res2 = await fetch(`https://api.ergoplatform.com/api/v0/assets/${data.tokens[i].tokenId}/issuingBox`)
					const data2 = await res2.json()
					if (res2.ok){
						objetoToken = {
							tokenCreationHeightConf: data2[0].creationHeight,
							tokenTokenIdConf: data2[0].assets[0].tokenId,
							tokenAmountConf: data.tokens[i].amount,
							tokenAmountConfEmit: data2[0].assets[0].amount,
							tokenNameConf: data2[0].assets[0].name,
							tokenDecimalsConf: data2[0].assets[0].decimals,
							tokenTypeConf: data2[0].assets[0].type,
							tokenR5Conf: toUtf8String(data2[0].additionalRegisters.R5).substr(2),
							tokenR6Conf: data2[0].additionalRegisters.R6,
							tokenR7Conf: data2[0].additionalRegisters.R7,
							tokenR8Conf: data2[0].additionalRegisters.R8,
							tokenR9Conf: resolveIpfs(toUtf8String(data2[0].additionalRegisters.R9).substr(2)),
							tokenTransactionConf: 'https://explorer.ergoplatform.com/en/transactions/' + data2[0].spentTransactionId
						}
					}
				arrayTokens[i] = objetoToken
				}
			} else {
				alert('No hay tokens')
			}
		}else {
			alert('Wallet incorrect!')
		}
	}
	// Clean wallets
	const cleanLocalStorage = () => {
		localStorage.clear()
		window.location.href = 'https://ergowallets.org'
	}
	const muestroWallet = (wallet) => {
		valorWallet = wallet
		loadWallet()
	}

	// Rescatar valor y mostrar token desde URL
	valorIdWallet = JSON.stringify($querystring)
	if (valorIdWallet.substring(1, 7) == 'wallet'){
		valorIdWallet = valorIdWallet.substring(8, valorIdWallet.length - 1)
		muestroWallet(valorIdWallet)
		
	}
	 
	/////

	function toUtf8String(hex) {
		if(!hex){
			hex = ''
		}
		var str = '';
		for (var i = 0; i < hex.length; i += 2) {
			str += String.fromCharCode(parseInt(hex.substr(i, 2), 16));
		}
		return str;
	}
	
		function sumarErgos() {
			if (localStorage.getItem("arrayWallets")){
				arrayWallets = JSON.parse(localStorage.getItem("arrayWallets"))
			}
			for(let i=0; i < arrayWallets.length; i++){
				fetch(`https://api.ergoplatform.com/api/v1/addresses/${arrayWallets[i].address}/balance/confirmed`)
					.then(response => response.json())
        			.then(consulta => {
						arrayWallets[i].ergos = (consulta.nanoErgs/1000000000).toFixed(2)
      			})
      			.catch(error => console.error(error))
			}
			
			totalErgos = 0
			for (let i=0; i < arrayWallets.length; i++){
				totalErgos = totalErgos + parseFloat(arrayWallets[i].ergos)
			}
		}

		function resolveIpfs(url) {
			const ipfsPrefix = 'ipfs://'
			if (!url.startsWith(ipfsPrefix)) return url
			else return url.replace(ipfsPrefix, 'https://cloudflare-ipfs.com/ipfs/')
		}
		
		sumarErgos()
	// Transactions
	//const res = await fetch(`https://api.ergoplatform.com/api/v1/addresses/${valorWallet}/transactions`)
	
</script>

<svelte:head>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-KyZXEAg3QhqLMpG8r+8fhAXLRk2vvoC2f3B09zVXn8CA5QIVfZOJ3BCsw2P0p/We" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/js/bootstrap.bundle.min.js" integrity="sha384-U1DAWAznBHeqEIlVSCgzq+c9gqGAJn5c/t99JyeKa9xxaYpSvHU5awsuZVVFIhvj" crossorigin="anonymous"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.1/font/bootstrap-icons.css">
</svelte:head>

<main>
	<nav class="navbar navbar-expand-lg navbar-light bg-dark">
		<div class="container-fluid">
			<a href="https://ergowallets.org" class="mb-1 mx-3 separaI"><img src="ergo.png" alt="Logotype Ergo" width="100"></a>
		  	<button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
				<span class="navbar-toggler-icon"></span>
		  	</button>
			<div class="collapse navbar-collapse" id="navbarSupportedContent">
				<ul class="navbar-nav me-auto mb-2 mb-lg-0">
					<li class="nav-item dropdown">
						<a on:click={sumarErgos} class="nav-link dropdown-toggle text-secondary" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
						<span class="text-secondary">Your Wallets</span>
						</a>
						<ul class="dropdown-menu bg-light border border-secondary" aria-labelledby="navbarDropdown">
							{#each arrayWallets as wallet}
								<li><a on:click={muestroWallet(wallet.address)} class="dropdown-item text-dark small" href="#">{wallet.name}<i class="bi bi-arrow-right-short"></i>{wallet.ergos}</a> </li>
							{/each}
							<li><hr class="dropdown-divider text-danger"></li>

							<li><a class="dropdown-item text-dark small disabled" href="#">{(totalErgos).toFixed(2)} Σ</a></li>

							<li><hr class="dropdown-divider text-info"></li>
							<li><a class="dropdown-item text-dark small" href="#" data-bs-toggle="modal" data-bs-target="#addWalletModal">Add wallet</a></li>
							<li><a class="dropdown-item text-dark small" href="#" data-bs-toggle="modal" data-bs-target="#deleteWalletModal">Delete wallet</a></li>
							<li><hr class="dropdown-divider text-danger"></li>
							<li><a class="dropdown-item text-dark small" href="https://ergoplatform.org/en/wallets/" target="_blank">ErgoPlatform Wallets</a></li>
						</ul>
					</li>
				</ul>
				<input bind:value={valorWallet} class="form-control me-2" type="search" placeholder="Your wallet" aria-label="Your wallet">
				<button on:click={loadWallet} class="separaI btn btn-outline-info" type="submit"> Accept</button>
			</div>
		</div>
	</nav>

	<!-- Info Ergo -->
	<ul class="list-group mt-3 bg-dark" id="grupoInfo">
    	<li class="list-group-item bg-dark border-light">
			<span class="small text-info">Σ {(totalErgos).toFixed(2)} </span><i class="bi bi-handbag-fill"></i><span class="small text-secondary"><strong>Price: </strong><span class="small text-light">{ObjetoInfoErgo.EurErgo}€ <i class="bi bi-arrow-right-short"></i> ${ObjetoInfoErgo.UsdErgo} </span>  <strong>Circulating supply: </strong><span class="small text-light">{ObjetoInfoErgo.totalCoinsErgo}</span> <strong>Transactions per Day: </strong><span class="small text-light">{ObjetoInfoErgo.transactionsErgo}</span>  <strong>Hashrate: </strong><span class="small text-light">{(ObjetoInfoErgo.hashRateErgo/1000000000000).toFixed(2)} TH/s</span></span>
        </li>
	</ul>
	<!-- Fin info Ergo-->

	<!-- Info wallet  -->
	<ul class="list-group mt-3 bg-light border border-light" id="grupoInfo">
		<li class="list-group-item bg-dark">
			{#if (objetoAddress.nanoErgsConf)}
				<span class="small text-info ">{(objetoAddress.nanoErgsConf/1000000000).toFixed(2)}ERG <i class="bi bi-arrow-right-short"></i> {((objetoAddress.nanoErgsConf/1000000000)*ObjetoInfoErgo.EurErgo).toFixed(2)}€ <i class="bi bi-arrow-right-short"></i> ${((objetoAddress.nanoErgsConf/1000000000)*ObjetoInfoErgo.UsdErgo).toFixed(2)}</span>
			{:else}
				<span class="small text-info">0.00</span>
			{/if}
		</li>
		<li class="list-group-item bg-secondary text-secondary">
			<span class="text-light small"><strong>Tokens </strong></span>
			{#if (objetoAddress.numberTokensConf)}
				<span id="tokens">{objetoAddress.numberTokensConf}</span>
					{#await loadWallet}
						<span class="text-light">Loading...</span>
					{:then}
						{#each arrayTokens as datos}
						<div class="container">
							<div class="card fondoGris px-2 py-2 mb-2" >
								<div class="row no-gutters">
									{#if (datos.tokenR7Conf == '0e020101')}
										<div class="col-sm-3">
											<img class="img-thumbnail border border-light" src={datos.tokenR9Conf} alt={datos.name}>
										</div>
									{:else if datos.tokenR7Conf == '0e0430313031'}
										<center>
											<div class="mark mt-3 py-3 {claseNoImage}">
												Token minted with other standards. <br>
												<h4>Look at its Descriptions.</h4>
											</div>
										</center>
									{:else if datos.tokenR7Conf == '0e020102'}
										<div class="col-sm-3 align-self-center mr-3">
											<span>
												<center><audio src={resolveIpfs(toUtf8String(datos.tokenR9Conf).substr(2))} style="width: 12rem;" title={datos.name} controls></audio> </center>
											</span>
										</div>
									{:else if (toUtf8String(datos.tokenR9Conf).slice(-4) == '.mp4') || (toUtf8String(datos.tokenR9Conf).slice(-4) == '.mov') || (toUtf8String(datos.tokenR9Conf).slice(-4) == '.3gp')}
										<div class="col-sm-3">	
											<span>
												<video src={resolveIpfs(toUtf8String(datos.tokenR9Conf).substr(2))} controls></video>
											</span>
										</div>
									{:else}
										<div class="col-sm-3 align-self-center mr-3">
											<center>
												<div class="mark mt-3 py-3 {claseNoImage}">No image-audio token</div>
											</center>
										</div>
									{/if}
									
									<div class="col-sm-8">
										<div class="card-body text-dark">
											<h5 class="card-title">{datos.tokenNameConf}</h5>
											<span class="text-break">{datos.tokenR5Conf}</span>
											<br>
											<p class="card-text small bg-white px-3 py-2 mt-2 rounded">
												<i class="bi bi-bricks"></i>
												<span><strong>Creation Height: </strong></span><span class="text-secondary">{datos.tokenCreationHeightConf}</span>
												<br>
												{#if datos.tokenR7Conf == '0e020101'}
													<i class="bi bi-file-image"></i>
												{:else if datos.tokenR7Conf == '0e020102'}
													<i class="bi bi-file-earmark-music"></i>
												{:else}
													<i class="bi bi-file-earmark-x"></i>
												{/if}
												<span class="text-break"><strong>Id: </strong></span><span class="text-secondary">{datos.tokenTokenIdConf}</span>
												{#if (datos.tokenR8Conf)}
													<br>
													<i class="bi bi-patch-check"></i>
													<span class="text-break"><strong>Sha256: </span><span class="text-secondary">{datos.tokenR8Conf}</span>
												{/if}
												<br>
												<i class="bi bi-stack"></i>
												<span><strong>Amount: </strong></span><span class="text-secondary">{datos.tokenAmountConf} of {datos.tokenAmountConfEmit}</span>
												<br>
												<i class="bi bi-file-binary"></i>
												<span><strong>Decimals: </strong></span><span class="text-secondary">{datos.tokenDecimalsConf}</span>
												<br>
												<i class="bi bi-file-earmark-spreadsheet"></i>
												<span><strong>Type: </strong></span><span class="text-secondary">{datos.tokenTypeConf}</span>
												<br>
												<a href="https://ergotokens.org/#/?token={datos.tokenTokenIdConf}" class="btn btn-sm btn-white text-dark border border-secondary mt-2" target="_blank">Details in ErgoTokens.org</a>
												<a href={datos.tokenTransactionConf} class="btn btn-sm btn-white text-dark border border-secondary mt-2" target="_blank">Transaction</a>
											</p>
										</div>
									</div>
								</div>
							</div>
						</div>
						{/each}
					{/await}
			{:else}
				<span id="tokens">0</span>
			{/if}
		</li>
    </ul>
	<!-- Fin Info wallet -->


	  <!-- Transaciones -->
	  <!-- <ul class="list-group mt-3 bg-secondary border border-light " id="grupoTransactions">
        <li class="list-group-item h4 bg-secondary text-light">
          <span>Transactions</span>
        </li>
			<li class="list-group-item bg-secondary">
				<span><strong>Ergos: </strong></span>
				{#if (objetoAddress.nanoErgs)}
					<span id="ergos">{objetoAddress.nanoErgs/1000000000}</span>
				{:else}
					<span id="ergos">0</span>
				{/if}
			</li>
			<li class="list-group-item bg-secondary">
				<span><strong>Tokens: </strong></span>
				{#if (objetoAddress.numberTokens)}
					<span id="tokens">{objetoAddress.numberTokens}</span>
				{:else}
					<span id="tokens">0</span>
				{/if}
			</li>
      </ul> -->
	  <!-- Fin Transaciones -->
</main>

<Footer />

<!-- Modal -->
<div class="modal fade" id="addWalletModal" tabindex="-1" aria-labelledby="addWalletModalLabel" aria-hidden="true">
	<div class="modal-dialog">
	  <div class="modal-content">
		<div class="modal-header bg-secondary">
			<h5 class="modal-title bg-dark text-light py-1 px-3" id="exampleModalLabel">Add wallets</h5>
			<button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
		  </div>
		<div class="modal-body">
			<div class="container">
				<div class="row">
				  <div class="row pb-2">
					<span class="mt-2">Name: </span><input bind:value={nameWallet} placeholder="Your address name">
					<span class="mt-2">Address: </span><input bind:value={valorWallet} placeholder="Your address">
				  </div>
				</div>
			  </div>
		</div>
		<div class="modal-footer">
		  <button type="close" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
		  <button on:click={addWallet} type="close" data-bs-dismiss="modal" class="btn btn-info">Accept</button>
		</div>
	  </div>
	</div>
  </div>

  <!-- Modal Delete wallets-->
<div class="modal fade" id="deleteWalletModal" tabindex="-1" aria-labelledby="deleteWalletModalLabel" aria-hidden="true">
	<div class="modal-dialog">
	  <div class="modal-content">
		<div class="modal-header bg-secondary">
			<h5 class="modal-title bg-dark text-light py-1 px-3" id="exampleModalLabel">Delete wallets</h5>
			<button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
		  </div>
		<div class="modal-body">
			<div class="container">
				<div class="row">
				  <span>Are you sure you want to delete your addresses?</span>
				</div>
			  </div>
		</div>
		<div class="modal-footer">
		  <button type="close" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
		  <button on:click={cleanLocalStorage} type="close" data-bs-dismiss="modal" class="btn btn-info">Delete</button>
		</div>
	  </div>
	</div>
  </div>

<style>
	.fondoGris{
		background-color: #e4e4e4;
	}
	
	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
	@media only screen and (max-width: 768px) {
		.separaI{
			margin-top: 5px;;
		}
	}
</style>