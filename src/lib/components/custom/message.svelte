<script lang="ts">
	export let text: string;
	export let status: boolean;
	export let user: string;
	export let time: string;
</script>

{#if status}
	<div class="flex w-auto flex-row-reverse">
		<div class="text-md ml-20 flex flex-col rounded-lg bg-primary p-3 text-background">
			{text}
			<span id='chat' class="opacity-50">{user} <br />{time}</span>
		</div>
	</div>
{:else}
	<div class="flex w-auto flex-row">
		<div class="text-md mr-20 flex flex-col rounded-lg bg-[#414651] p-3 text-background dark:bg-slate-800 dark:text-white">
			{text}
			<span id="chat" class="opacity-50">{user} <br />{time}</span>
		</div>
	</div>
{/if}

<style>
	#chat {
		margin-top: 5px;
		font-size: 0.8rem /* 14px */;
		line-height: 1rem /* 20px */;
	}
</style>
