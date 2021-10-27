<script>
	export let wave = null;

	function toIterable(obj) {
		if (!obj) return [];
		let attrs = Object.entries(obj).map(([key, value]) => ({
			[key]: value
		}));
		return attrs;
	}
	let waveIterable = toIterable(wave);
</script>

<section class="waveContainer">
	{#each waveIterable as attr}
		<div class="grid gap" style="grid-template-columns: 80px minmax(0, 1fr);">
			<div class="label">
				{Object.keys(attr).toString().toUpperCase()}
			</div>
			<div class="info">
				👋 {Object.values(attr).toString()}
			</div>
		</div>
	{/each}
</section>

<style>
	.waveContainer {
		background-color: beige;
		padding: 8px;
		margin-bottom: 12px;
		border-radius: 8px;
		box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px;
	}

	.grid {
		display: -ms-grid;
		display: grid;
	}

	.gap {
		grid-gap: 8px;
		gap: 8px;
	}

	.label {
		font-size: 13px;
		font-weight: 500;
	}

	.info {
		font-size: 13px;
		font-weight: 300;
		font-style: italic;
	}
</style>
