<script lang="ts">
	let { children } = $props();
</script>

<style>
	@font-face {
		font-family: 'Libre Baskerville';
		font-style: normal;
		font-weight: 500;
		src: local(''), url('/Libre_Baskerville/LibreBaskerville-Regular.ttf') format('truetype')
	}

	@font-face {
		font-family: 'Libre Baskerville';
		font-style: normal;
		font-weight: bold;
		src: local(''), url('/Libre_Baskerville/LibreBaskerville-Bold.ttf') format('truetype')
	}

	:global(body) {
		overflow: hidden;
	}
</style>

{@render children()}