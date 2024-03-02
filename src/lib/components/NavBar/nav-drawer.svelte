<script lang='ts'>
    // import logo from '$lib/images/shadcn-svelte-logo.png';
    import Icon from '@iconify/svelte';
    import { components } from './components';
    
    export let hidden: boolean = true;

    const openComponent = (path:string) => {
        window.location.href = `${path}`;
    }

</script>

<div class={`h-screen w-72 fixed top-0 backdrop-blur-lg border transform ${hidden ? "translate-x-[-100%]" : "translate-x-0"} transition-transform duration-500 ease-in-out`}>
    <div class="flex flex-row">
        <button on:click={() => {hidden = true}}>
            <Icon icon="maki:cross"  style="color: #020817 dark:white" class="size-10 mt-4 ml-4 opacity-75 dark: white"  />
        </button>
        <!-- <div class="flex flex-row h-16">
            <img src={logo} alt="" class="p-1">
            <h1 class="mt-auto mb-auto text-lg font-semibold"><span>Shadcn-</span><span class=" text-[#cd4423]">Svelte</span></h1>
        </div> -->
    </div>
    <div class="flex flex-col ml-2 mt-8">
        <h1 class="text-2xl mb-4 font-bold text-center text-[#cd4423]">Components</h1>
        <div class="flex flex-col gap-2 object-contain h-[80vh] overflow-y-auto scrollbar-hide">
            {#each components as component}
                <button on:click={() => {openComponent(component[1])}} class="text-xl font-semibold text-center">{component[0]}</button>
            {/each}
        </div>
    </div>
</div>

<style>
    .scrollbar-hide::-webkit-scrollbar {
        display: none;
    }

    .scrollbar-hide {
        -ms-overflow-style: none;
        scrollbar-width: none;
    }
</style>