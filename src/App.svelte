<script>

	let infoWallet = {id: '', address: ''}
	let arrayWallets = []
	let valorWallet = ''
	let objetoAddress = {}
	let objetoToken = {}
	let arrayTokens = []

	if (localStorage.getItem("arrayWallets")){
		arrayWallets = JSON.parse(localStorage.getItem("arrayWallets"))
	}
	$: localStorage.setItem("arrayWallets", JSON.stringify(arrayWallets))

	// Add wallets
	const addWallet = async() => {
		// Si la consulta devuelve ok 200 es verdadera
		const res = await fetch(`https://api.ergoplatform.com/api/v1/addresses/${valorWallet}/balance/total`)
		if (res.ok){
			const data = await res.json()
			infoWallet.id = Date.now()
			infoWallet.address = valorWallet
			arrayWallets = [...arrayWallets, infoWallet]
			infoWallet = {id: '', address: ''}
			alert('Wallet add correct!')
		}else {
			alert('Wallet incorrect!')
		}
	}


	// Load wallet
	const loadWallet = async() => {
		// Si la consulta devuelve ok 200 es verdadera
		const res = await fetch(`https://api.ergoplatform.com/api/v1/addresses/${valorWallet}/balance/total`)
		if (res.ok){
			const data = await res.json()
			// Objeto datos confirmados
			objetoAddress = {
				nanoErgsConf: data.confirmed.nanoErgs,
				numberTokensConf: data.confirmed.tokens.length,
				nanoErgsUnconf: data.unconfirmed.nanoErgs,
				numberTokensUnconf: data.unconfirmed.tokens.length
			}

			// Si hay tokens confirmados
			if (data.confirmed.tokens.length){
			 	arrayTokens = []
				for (let i = 0; i < data.confirmed.tokens.length; i++) {
					const res2 = await fetch(`https://api.ergoplatform.com/api/v1/boxes/byTokenId/${data.confirmed.tokens[i].tokenId}`)
					if (res2.ok){
						const data2 = await res2.json()
						objetoToken = {
							tokenCreationHeightConf: data2.items[0].creationHeight,
							tokenTokenIdConf: data2.items[0].assets[0].tokenId,
							tokenNameConf: data2.items[0].assets[0].name,
							tokenAmountConf: data2.items[0].assets[0].amount,
							tokenDecimalsConf: data2.items[0].assets[0].decimals,
							tokenTypeConf: data2.items[0].assets[0].type,
						}
						if (data2.items[0].additionalRegisters.R4){
							objetoToken.tokenR4SigmaConf = data2.items[0].additionalRegisters.R4.sigmaType
							objetoToken.tokenR4Conf = data2.items[0].additionalRegisters.R4.renderedValue
						}
						if (data2.items[0].additionalRegisters.R5){
							objetoToken.tokenR5SigmaConf = data2.items[0].additionalRegisters.R5.sigmaType
							objetoToken.tokenR5Conf = data2.items[0].additionalRegisters.R5.renderedValue
						}
						if (data2.items[0].additionalRegisters.R6){
							objetoToken.tokenR6SigmaConf = data2.items[0].additionalRegisters.R6.sigmaType
							objetoToken.tokenR6Conf = data2.items[0].additionalRegisters.R6.renderedValue
						}
						if (data2.items[0].additionalRegisters.R7){
							objetoToken.tokenR7SigmaConf = data2.items[0].additionalRegisters.R7.sigmaType
							objetoToken.tokenR7Conf = data2.items[0].additionalRegisters.R7.renderedValue
						}
						if (data2.items[0].additionalRegisters.R8){
							objetoToken.tokenR8SigmaConf = data2.items[0].additionalRegisters.R8.sigmaType
							objetoToken.tokenR8Conf = data2.items[0].additionalRegisters.R8.renderedValue
						}
						if (data2.items[0].additionalRegisters.R9){
							objetoToken.tokenR9SigmaConf = data2.items[0].additionalRegisters.R9.sigmaType
							objetoToken.tokenR9Conf = toUtf8String(data2.items[0].additionalRegisters.R9.renderedValue)
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
		
		// Objeto datos sin confirmar
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

	// Transactions
	//const res = await fetch(`https://api.ergoplatform.com/api/v1/addresses/${valorWallet}/transactions`)
	
</script>

<svelte:head>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-KyZXEAg3QhqLMpG8r+8fhAXLRk2vvoC2f3B09zVXn8CA5QIVfZOJ3BCsw2P0p/We" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/js/bootstrap.bundle.min.js" integrity="sha384-U1DAWAznBHeqEIlVSCgzq+c9gqGAJn5c/t99JyeKa9xxaYpSvHU5awsuZVVFIhvj" crossorigin="anonymous"></script>
</svelte:head>

<main>
	<nav class="navbar navbar-expand-lg navbar-light bg-dark">
		<div class="container-fluid">
			<a href="https://ergonfts.org" class="mb-1 mx-3 separaI"><img src="ergo.png" alt="Logotype Ergo" width="100"></a>
		  	<button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
				<span class="navbar-toggler-icon"></span>
		  	</button>
			<div class="collapse navbar-collapse" id="navbarSupportedContent">
				<ul class="navbar-nav me-auto mb-2 mb-lg-0">
					<li class="nav-item dropdown">
						<a class="nav-link dropdown-toggle text-secondary" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
						<span class="text-secondary">Your Wallets</span>
						</a>
						<ul class="dropdown-menu" aria-labelledby="navbarDropdown">
							{#each arrayWallets as wallet}
								<li><a on:click={muestroWallet(wallet.address)} class="dropdown-item" href="#">{wallet.address.substring(0, 7)}...{wallet.address.slice(-7)}</a></li>
							{/each}
							<li><hr class="dropdown-divider"></li>
							<li><a class="dropdown-item" href="#" data-bs-toggle="modal" data-bs-target="#addWalletModal">Add wallet</a></li>
							<li><a class="dropdown-item" on:click={cleanLocalStorage}>Delete wallet</a></li>
							<li><hr class="dropdown-divider"></li>
							<li><a class="dropdown-item" href="https://ergoplatform.org/en/wallets/" target="_blank">ErgoPlatform Wallets</a></li>
						</ul>
					</li>
				</ul>
				<input bind:value={valorWallet} class="form-control me-2" type="search" placeholder="Your wallet" aria-label="Your wallet">
				<button on:click={loadWallet} class="btn btn-outline-success" type="submit">Accept</button>
			</div>
		</div>
	</nav>

	<!-- Info wallet  -->
	<ul class="list-group mt-3 bg-light " id="grupoInfo">
    	<li class="list-group-item h4 text-secondary">
        	  <span>Information</span>
        </li>
		<li class="list-group-item ">
			<span><strong>Address: </strong></span>
			{#if (valorWallet)}
				<span id="ergos">{valorWallet}</span>
			{:else}
				<span id="ergos"> </span>
			{/if}
		</li>
		<li class="list-group-item ">
			<span><strong>Ergos: </strong></span>
			{#if (objetoAddress.nanoErgsConf)}
				<span id="ergos">{objetoAddress.nanoErgsConf/1000000000}</span>
			{:else}
				<span id="ergos">0</span>
			{/if}
		</li>
		<li class="list-group-item ">
			<span><strong>Tokens: </strong></span>
			{#if (objetoAddress.numberTokensConf)}
				<span id="tokens">{objetoAddress.numberTokensConf}</span>
				<ul class="list-group">
					{#await loadWallet}
						<span>Loading...</span>
					{:then loadWallet}
						{#each arrayTokens as datos}
							<li class="list-group-item">{datos.tokenTokenIdConf}</li>
						{/each}
					{:catch error}
						<p class="text-secondary">Something went wrong: {error.message}</p>
					{/await}
				</ul>
			{:else}
				<span id="tokens">0</span>
			{/if}
		</li>
    </ul>
	<!-- Fin Info wallet -->


	  <!-- Transaciones -->
	  <ul class="list-group mt-3 bg-secondary " id="grupoTransactions">
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
      </ul>
	  <!-- Fin Transaciones -->
</main>

<!-- Modal -->
<div class="modal fade" id="addWalletModal" tabindex="-1" aria-labelledby="addWalletModalLabel" aria-hidden="true">
	<div class="modal-dialog">
	  <div class="modal-content">
		<div class="modal-header">
		  <h5 class="modal-title" id="addWalletModalLabel">Add wallet</h5>
		  <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
		</div>
		<div class="modal-body">
			<input bind:value={valorWallet} placeholder="Your wallet">
		</div>
		<div class="modal-footer">
		  <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
		  <button on:click={addWallet} type="button" class="btn btn-primary">Accept</button>
		</div>
	  </div>
	</div>
  </div>

<style>
	

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>