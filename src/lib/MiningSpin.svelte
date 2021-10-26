<script>
	import { elasticOut } from 'svelte/easing';

	function spin(node, { duration }) {
		return {
			duration,
			css: (t) => {
				const eased = elasticOut(t);

				return `
          transform: scale(${eased}) rotate(${eased * 1080}deg);
          color: hsl(
            ${~~(t * 360)},
            ${Math.min(100, 1000 - 1000 * t)}%,
            ${Math.min(50, 500 - 500 * t)}%
          );`;
			}
		};
	}
</script>

<div class="centered" in:spin={{ duration: 30000 }} out:spin={{ duration: 1000 }}>
	<span>mining...</span>
</div>

<style>
	.centered {
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
	}

	span {
		position: absolute;
		transform: translate(-50%, -50%);
		font-size: 4em;
	}
</style>
