<script lang='ts'>
    import logo from '$lib/images/shadcn-svelte-logo.png';
    import Icon from '@iconify/svelte';
    import { theme } from '$lib/stores';
    import { siteConfig } from '$lib/config/site';
    import { NavDrawer } from '.';

    let hidden = true;

    function toggle() {
        const upcoming_theme = $theme.current === 'light' ? 'dark' : 'light';
        if (
            upcoming_theme ===
            (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light')
        ) {
            // Switch the preference to `system`
            $theme.preference = 'system';
        } else {
            // Switch the preference to `light` or `dark`
            $theme.preference = upcoming_theme;
        }

        $theme.current = upcoming_theme;
    }
</script>

<nav>
    <div class="w-full h-16 border-b shadow-lg mb-8 flex justify-center content-center p-2 dark:bg-[#020b21]">
        <div class="max-w-7xl w-full flex flex-row content-center justify center mx-4">
            <button on:click={() => {hidden = !hidden}}>
                <Icon icon="material-symbols-light:bottom-drawer"  style="color: black dark: white" class="size-8 mr-2 my-auto" />
            </button>
            <a class="flex flex-row" href="/">
                <img src={logo} alt="">
                <h1 class="mt-auto mb-auto text-lg font-semibold"><span>Shadcn-</span><span class=" text-[#cd4423]">Svelte</span></h1>
            </a>
            <div class="flex flex-row ml-auto items-center">
                <a href={siteConfig.github}>
                    <Icon icon="ri:github-fill"  style="color: #020817 dark:white" class="size-8 opacity-75"/>
                </a>
                <button class="flex items-center ml-4" on:click={toggle}>
                    {#if $theme.current == "dark"}
                        <Icon icon="ph:moon-fill"  style="color: white" class="size-6"/>
                    {:else}
                        <Icon icon="ph:sun-bold"  style="color: #020817" class="size-6"/>
                    {/if}
                </button>
            </div>
        </div>
    </div>
    <NavDrawer bind:hidden={hidden}/>
</nav>

<style>
</style>