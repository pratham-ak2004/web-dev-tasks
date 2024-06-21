<script lang="ts">
	export let text: string;
	export let status: boolean;
</script>

{#if status}
	<div class="flex w-auto flex-row-reverse">
		<div class="text-md ml-20 rounded-lg bg-primary p-3 text-background">
			{text}
		</div>
	</div>
{:else}
	<div class="flex w-auto flex-row">
		<div class="text-md mr-20 rounded-lg bg-[#414651] p-3 text-background">
			{text}
		</div>
	</div>
{/if}
