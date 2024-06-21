<script lang="ts">
	import Message from '$lib/components/custom/message.svelte';
	import Button from '$lib/components/ui/button/button.svelte';
	import Input from '$lib/components/ui/input/input.svelte';
	import { io } from 'socket.io-client';

	const socket = io('http://localhost:3000', {
		autoConnect: false
	});

	socket.on('connect', () => {
		// console.log(socket.id);
		socket.emit('join', 'chat');
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

	socket.connect()

	function connectSocket() {
		if (!socket.active) {
			socket.connect();
		}
	}

	socket.on('receive', (msg) => {
		messages.push({
			text: msg.text,
			by: msg.by
		});
		messages = messages;
	});

	let text: string = '';
	let messages: any[] = [];

	function handleSendMessage() {
		if (text.trim() !== '') {
			connectSocket();
			socket.emit('send', { text: text, by: socket.id, room: 'chat' });
			// messages.push({
			// 	text: text,
			// 	by: socket.id
			// });
			messages = messages;
		}
		text = '';
	}
</script>

<div class="flex h-full w-full flex-col items-center">
	<div
		class="mx-4 flex h-96 w-full max-w-6xl flex-col gap-2 overflow-y-auto rounded-lg border-2 p-4 shadow-md"
	>
		{#each messages as msg}
			<Message text={msg.text} status={msg.by === socket.id} />
		{/each}
	</div>
	<div class="mx-4 mt-4 flex h-14 w-full max-w-6xl items-center gap-2 rounded-lg">
		<Input class="h-full w-full text-lg shadow-lg" bind:value={text} />
		<Button class="text-md h-full w-fit shadow-lg" on:click={handleSendMessage}>Send</Button>
	</div>
</div>
