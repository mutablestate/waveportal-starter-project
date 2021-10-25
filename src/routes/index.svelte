<script context="module">
	export const prerender = true;
</script>

<script>
	function handleWave() {}
</script>

<svelte:head>
	<title>WavePortal</title>
</svelte:head>

<div class="mainContainer">
	<div class="dataContainer">
		<div class="header">👋 Hey there!</div>

		<div class="bio">
			<div>I used SvelteKit to build and Cloudflare to deploy my WavePortal!</div>
			<div>Connect your Ethereum wallet and wave at me!</div>
		</div>

		<button class="waveButton" on:click={handleWave}>Wave at Me</button>
	</div>
</div>

<style>
	.mainContainer {
		display: flex;
		justify-content: center;
		width: 100%;
		margin-top: 64px;
	}

	.dataContainer {
		display: flex;
		flex-direction: column;
		justify-content: center;
		max-width: 600px;
	}

	.header {
		text-align: center;
		font-size: 32px;
		font-weight: 600;
	}

	.bio {
		text-align: center;
		color: gray;
		margin-top: 16px;
	}

	.waveButton {
		cursor: pointer;
		margin-top: 16px;
		padding: 8px;
		border: 0;
		border-radius: 5px;
	}
</style>
