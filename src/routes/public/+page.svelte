<script lang="ts">
	import Message from '$lib/components/custom/message.svelte';
	import Button from '$lib/components/ui/button/button.svelte';
	import Input from '$lib/components/ui/input/input.svelte';
	import * as AlertDialog from '$lib/components/ui/alert-dialog';
	import { io } from 'socket.io-client';
	import { onMount, tick } from 'svelte';

	onMount(() => {
		scrollToBottom();
		document.body.addEventListener('keydown', (e) => {
			if (e.key === 'Enter') {
				if (userName !== '') {
					handleSendMessage();
				} else {
					openDialog = true;
				}
			}
		});
		return () => {
			document.body.removeEventListener('keydown', () => console.log('key event removed'));
		};
	});

	let openDialog = false;
	let isUserNameSet = false;
	let userName = '';
	const ROOM = 'public';
	let messageBox: HTMLElement | null;	

	const socket = io(`${import.meta.env.VITE_SOCKET_SERVER}`, {
		autoConnect: false
	});

	socket.on('connect', () => {
		socket.emit('join', ROOM);
	});

	// socket.on('disconnect', () => {
	// 	console.log('socket disconnected');
	// });

	// socket.io.on('reconnect', () => {
	// 	console.log('socket reconnected');
	// });

	// socket.io.on('reconnect_attempt', () => {
	// 	console.log('attempting reconnect');
	// });

	socket.connect();

	function connectSocket() {
		if (!socket.active) {
			socket.connect();
		}
	}

	socket.on('receive', async (msg) => {
		messages.push({
			text: msg.text,
			by: msg.by || 'anonymus',
			time: msg.time
		});
		messages = messages;
		await tick();
		scrollToBottom();
	});

	let text: string = '';
	let messages: any[] = [];

	function handleSendMessage() {
		if (text.trim() !== '') {
			connectSocket();
			socket.emit('send', {
				text: text,
				by: userName || 'anonymus',
				room: ROOM,
				time: new Date().toLocaleTimeString()
			});
			// messages.push({
			// 	text: text,
			// 	by: socket.id
			// });
			messages = messages;
		}
		text = '';
	}

	function scrollToBottom() {
		if (messageBox) {
			messageBox.scrollTop = messageBox.scrollHeight;
		}
	}
</script>

<div class="flex h-full w-full flex-col items-center">
	<div
		bind:this={messageBox}
		class="mx-4 flex h-96 w-full max-w-6xl flex-col gap-2 overflow-y-auto rounded-lg border-2 p-4 shadow-md"
	>
		{#each messages as msg}
			<Message text={msg.text} status={msg.by === userName} user={msg.by} time={msg.time} />
		{/each}
	</div>
	<div class="mx-4 mt-4 flex h-14 w-full max-w-6xl items-center gap-2 rounded-lg">
		<Input class="h-full w-full text-lg shadow-lg text-wrap" bind:value={text} />
		{#if !isUserNameSet}
			<AlertDialog.Root open={openDialog}>
				<AlertDialog.Trigger class="h-full">
					<Button class="text-md h-full w-fit shadow-lg">Send</Button>
				</AlertDialog.Trigger>
				<AlertDialog.Content>
					<AlertDialog.Header>
						<AlertDialog.Title>Enter your username</AlertDialog.Title>
					</AlertDialog.Header>
					<AlertDialog.Description>
						<Input bind:value={userName} />
					</AlertDialog.Description>
					<AlertDialog.Footer>
						<AlertDialog.Cancel>Cancel</AlertDialog.Cancel>
						<AlertDialog.Action disabled={userName === ''} on:click={() => (isUserNameSet = true)}
							>Continue</AlertDialog.Action
						>
					</AlertDialog.Footer>
				</AlertDialog.Content>
			</AlertDialog.Root>
		{:else}
			<Button
				class="text-md h-full w-fit shadow-lg"
				on:click={handleSendMessage}
				disabled={userName === ''}>Send</Button
			>
		{/if}
	</div>
</div>