<script lang='ts'>
	import "../app.pcss";
    import { NavBar , MobileNavBar } from '$lib/components/NavBar';
	import { onMount } from 'svelte';

	let isMobile = false;

	onMount(() => {
		function handleResize() {
			if (window.innerWidth < 640) {
				isMobile = true;
			} else {
				isMobile = false;
			}
		}
		isMobile = window.innerWidth < 640;
		window.addEventListener('resize', handleResize);
	})
</script>

<div class="app">
	{#if isMobile}
		<MobileNavBar />
	{:else}
		<NavBar />
	{/if}
	
	<main>
		<slot></slot>
	</main>

	<footer>
		
	</footer>
</div>

<style>
</style>
