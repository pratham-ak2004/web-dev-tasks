<script>
	import "../app.pcss";
    import { NavBar , MobileNavBar } from '$lib/components/NavBar';
	import { onMount } from "svelte";

	let screen = window.matchMedia("(max-width: 768px)");
	let isMobile = false;
	
	onMount(() => {
		function handleResize(e) {
			isMobile = screen.matches;
		}
		screen.addEventListener("change", handleResize);
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
