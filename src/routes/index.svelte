<script context="module">
	export const prerender = true;
</script>

<script>
	import { onMount } from 'svelte';
	import { ethers } from 'ethers';
	import wavePortalAbi from '$lib/WavePortal.json';
	import MiningSpin from '$lib/MiningSpin.svelte';
	import { currentAccount } from '../stores';

	const contractAddress = '0x22E06b837004f4756690a6cD9F1ddDB7eF7eBf7b';
	const contractAbi = wavePortalAbi.abi;

	let ethereum = null;
	let mining = false;
	let count = 0;

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

			const waveTxn = await wavePortalContract.wave();
			mining = true;
			console.log('Mining...', waveTxn.hash);

			await waveTxn.wait();
			console.log('Mined -- ', waveTxn.hash);
			mining = false;

			count = await wavePortalContract.getTotalWaves();
			console.log('Retrieved total wave count...', count.toNumber());
		} catch (error) {
			console.log('Ooof! Your wave failed.', error);
		}
	}

	async function checkWalletConnection() {
		const accounts = await ethereum.request({ method: 'eth_accounts' });

		if (accounts.length !== 0) {
			const account = accounts[0];
			console.log('Found an authorized account:', account);
			currentAccount.set(account);
		} else {
			console.log('No authorized account found');
		}
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
		<div class="header">Join ∞ + {count} people that 👋</div>

		<div class="bio">
			<div>Connect your Ethereum wallet to wave!</div>
		</div>

		<button class="waveButton" on:click={handleWave}>Wave at Me</button>

		{#if mining}
			<MiningSpin />
		{/if}

		{#if !$currentAccount}
			<button class="waveButton" on:click={handleConnectWallet}>Connect Wallet</button>
		{/if}
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
