<script>
	import welcome from '$lib/images/svelte-welcome.png';
	import { components } from '$lib/components/NavBar/components';
	import Badge from '$lib/components/ui/badge/badge.svelte';
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section class="flex flex-col items-center">
	<img src={welcome} alt="Welcome to Svelte custom web components using shadcn" />
	<p class="font-medium text-center mx-10">
		Welcome to Svelte custom web components using <span class="font-bold"><span>Shadcn-</span><span class=" text-[#cd4423]">Svelte</span></span>.
	</p>
	<h1 class="mt-20 text-2xl font-semibold text-center">Browse components</h1>
	<div class="gap-x-3 gap-y-2 flex flex-wrap items-center justify-center max-w-3xl mt-10 mx-10">
		{#each components as component}
			<a href={component[1]}><Badge variant="outline" class="text-base">{component[0]}</Badge></a>
		{/each}
	</div>
</section>

<style>
</style>
