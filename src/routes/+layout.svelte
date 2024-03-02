<script lang='ts'>
	import "../app.pcss";
    import { NavBar , MobileNavBar } from '$lib/components/NavBar';
	import { onMount } from 'svelte';
	import { siteConfig } from '$lib/config/site';
	import Icon from '@iconify/svelte';

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

	<main class="min-h-[70vh]">
		<slot></slot>
	</main>

	<footer class="mt-10 pt-8 h-40 dark:bg-[#020b21] flex flex-col items-center gap-y-6">
		<p class="text-center">Made with &#10084 by <a href={siteConfig.githubUser}><u>pratham-ak2004</u></a></p>
		<div class="flex flex-row gap-x-2">
			<a href={siteConfig.github}>
				<Icon icon="ri:github-fill"  style="color: #020817 dark:white" class="size-8 opacity-75"/>
			</a>
			<a href={siteConfig.instagram}>
				<Icon icon="mdi:instagram"  style="color: #020817 dark:white" class="size-8 opacity-75"/>
			</a>
		</div>
	</footer>
</div>

<style>
	footer{
		box-shadow: -5px -5px 15px -5px #000000;
	}
</style>
