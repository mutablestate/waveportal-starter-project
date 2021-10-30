<script context="module">
	export const prerender = true;
</script>

<script>
	import { onMount } from 'svelte';
	import { ethers } from 'ethers';
	import wavePortalAbi from '$lib/WavePortal.json';
	import Cooldown from '$lib/Cooldown.svelte';
	import MiningSpin from '$lib/MiningSpin.svelte';
	import WaveInfo from '$lib/WaveInfo.svelte';
	import { currentAccount, allWaves } from '../stores';

	const contractAddress = '0xC9B3838447a267E9B6E22BDb128ed2422029a74B';
	const contractAbi = wavePortalAbi.abi;

	let ethereum = null;
	let mining = false;
	let count = 0;
	let waveMsg = 'Change me!';
	let showCooldown = false;

	async function getAllWaves() {
		try {
			const provider = new ethers.providers.Web3Provider(ethereum);
			const signer = provider.getSigner();
			const wavePortalContract = new ethers.Contract(contractAddress, contractAbi, signer);

			const waves = await wavePortalContract.getAllWaves();

			let wavesCleaned = [];
			waves.forEach((wave) => {
				wavesCleaned.push({
					address: wave.waver,
					timestamp: new Date(wave.timestamp * 1000),
					message: wave.message
				});
			});

			allWaves.set(wavesCleaned);
		} catch (error) {
			console.log('Ooof! Fetching all waves failed.', error);
		}
	}

	async function handleConnectWallet() {
		const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
		console.log('Connected', accounts[0]);
		currentAccount.set(accounts[0]);
	}

	async function handleWave() {
		try {
			const provider = new ethers.providers.Web3Provider(ethereum);
			const signer = provider.getSigner();
			const wavePortalContract = new ethers.Contract(contractAddress, contractAbi, signer);

			count = await wavePortalContract.getTotalWaves();
			console.log('Retrieved total wave count...', count.toNumber());

			const waveTxn = await wavePortalContract.wave(waveMsg || 'Catch this wave!');
			mining = true;
			console.log('Mining...', waveTxn.hash);

			await waveTxn.wait();
			console.log('Mined -- ', waveTxn.hash);
			mining = false;
			waveMsg = '';

			count = await wavePortalContract.getTotalWaves();
			console.log('Retrieved total wave count...', count.toNumber());
			getAllWaves();
		} catch (error) {
			mining = false;
			showCooldown = true;
			console.log('Ooof! Your wave failed.', error);
		}
	}

	async function checkWalletConnection() {
		const accounts = await ethereum.request({ method: 'eth_accounts' });

		if (accounts.length !== 0) {
			const account = accounts[0];
			console.log('Found an authorized account:', account);
			currentAccount.set(account);
			getAllWaves();
		} else {
			console.log('No authorized account found');
		}
	}

	function handleCooldown() {
		showCooldown = false;
	}

	onMount(() => {
		ethereum = window?.ethereum;
		if (ethereum) {
			checkWalletConnection();
		} else {
			alert('Please login to the MetaMask browser extension!');
		}
	});
</script>

<svelte:head>
	<title>WavePortal</title>
</svelte:head>

<div class="mainContainer">
	<div class="dataContainer">
		<div class="header">Join {count} amazing people that 👋</div>

		{#if !$currentAccount}
			<div class="connect">Connect your Ethereum wallet!</div>
			<button class="waveButton" on:click={handleConnectWallet}>Connect Wallet</button>
		{:else}
			<div class="connect">Connected: {$currentAccount}</div>
		{/if}

		<div style="margin-top: 32px;">
			{#if mining || showCooldown}
				{#if mining}
					<MiningSpin />
				{/if}
				{#if showCooldown}
					<Cooldown on:message={handleCooldown} />
				{/if}
			{:else}
				<div class="grid gap" style="grid-template-columns: 300px minmax(0, 1fr);">
					<input id="msg" type="text" class="text-input" bind:value={waveMsg} />
					<button class="waveButton" on:click={handleWave}>Say hi</button>
				</div>
			{/if}
		</div>

		{#if $allWaves.length > 0}
			<div class="hof-title">Hall of Waves</div>
		{/if}
		{#each $allWaves as wave}
			<WaveInfo {wave} />
		{/each}
	</div>
</div>

<style>
	.text-input {
		font-weight: 400;
		padding: 8px;
		--tw-shadow-color: 0, 0, 0;
		--tw-shadow: 0 10px 15px -3px rgba(var(--tw-shadow-color), 0.1),
			0 4px 6px -2px rgba(var(--tw-shadow-color), 0.05);
		-webkit-box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000),
			var(--tw-shadow);
		box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000),
			var(--tw-shadow);
	}

	.hof-title {
		padding-top: 48px;
		padding-bottom: 16px;
		font-size: 20px;
		font-weight: 500;
		color: #004899;
		text-align: center;
	}
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

	.connect {
		text-align: center;
		color: #03c03c;
		margin-top: 16px;
	}

	.grid {
		display: -ms-grid;
		display: grid;
	}

	.gap {
		grid-gap: 8px;
		gap: 8px;
	}

	.waveButton {
		cursor: pointer;
		padding: 8px;
		border: 0;
		border-radius: 5px;
		color: white;
		font-weight: 500;
		background-color: #ff3e00;
	}
</style>
