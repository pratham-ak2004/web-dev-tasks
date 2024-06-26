<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import Input from '$lib/components/ui/input/input.svelte';
	import { onMount } from 'svelte';
	import io from 'socket.io-client';
	import { peerConfiguration } from '$lib/utils';

	let userName = 'caller';
	let localStream: MediaStream;
	let remoteStream: MediaStream;
	let localStreamElement: HTMLVideoElement;
	let remoteStreamElement: HTMLVideoElement;
	let peerConnection: RTCPeerConnection;
    let answer:any;

	const socket = io(`${import.meta.env.VITE_SOCKET_SERVER}/stream`, {
		autoConnect: false,
		auth: {
			userName
		},
		query: {
			room: 'publicStream'
		}
	}).connect();

	function connectToSocket() {
		if (!socket.active) {
			socket.connect();
		}
	}

	function fetchUserMedia() {
		return new Promise<void>(async (resolve, reject) => {
			try {
				localStream = await navigator.mediaDevices.getUserMedia({
					video: {
						// width: { ideal: 1280 },
						// height: { ideal: 720 },
						aspectRatio: 16 / 10
					},
					audio: false
					// {
					//     echoCancellation: true,
					//     noiseSuppression: true,
					//     autoGainControl: true
					// }
				});
				if (localStream) {
					localStreamElement.srcObject = localStream;
				}
				resolve();
			} catch (err) {
				console.log(err);
				reject();
			}
		});
	}

	function createPeerConnection() {
		peerConnection = new RTCPeerConnection(peerConfiguration);

		localStream.getTracks().forEach((track: MediaStreamTrack) => {
			peerConnection.addTrack(track, localStream);
		});

		peerConnection.onicecandidate = (event) => {
			if (event.candidate) {
				socket.emit('newIceCandidate', {
					candidate: event.candidate,
					userName: userName
				});
			}
		};

		peerConnection.ontrack = (event) => {
			console.log('New Track', event.streams[0]);
			remoteStreamElement.srcObject = event.streams[0];
		};
	}

	async function createOffer() {
		const offer = await peerConnection.createOffer();
		await peerConnection.setLocalDescription(offer); // ICE candidates starts to come in
		connectToSocket();
		socket.emit('newOffer', {
			offer: offer,
			user: userName
		});
	}

	socket.on('getOffer', (data) => {
		console.log(data);
	});

	socket.on('getAnswer', (data) => {
		console.log('Received ansewer: ', data);
		peerConnection.setRemoteDescription(data.answer);
        answer = data
	});

	socket.on('receiveIceCandidate', (data) => {
		console.log('Received Ice Candidate: ', data);
        data.forEach((item:any) => {
            peerConnection.addIceCandidate(item.candidate);
        });
	});
</script>

<section class="flex h-full w-full flex-col items-center gap-4 p-4">
	<div class="flex w-full max-w-7xl justify-center">
		<div class="relative">
			<video
				bind:this={remoteStreamElement}
				class="z-10 aspect-[16/10] min-h-96 w-full rounded-sm bg-green-400"
				autoplay
				muted
				playsinline
			>
				<track kind="captions" />
			</video>
			<video
				class="absolute bottom-2 right-2 z-20 aspect-[16/10] w-full min-w-20 rounded-lg sm:w-44"
				bind:this={localStreamElement}
				autoplay
				muted
				playsinline
			/>
		</div>
	</div>
	<div class="flex w-fit max-w-7xl flex-col flex-nowrap justify-center gap-2">
		<Button
			on:click={async () => {
				await fetchUserMedia();
			}}>Step 1 Start Video</Button
		>
		<Button
			on:click={async () => {
				await createPeerConnection();
			}}>Step 2 create peerConnection</Button
		>
		<Button
			on:click={async () => {
				await createOffer();
			}}>step 3 Create Offer <span class="opacity-60">(Fetch Ice Candidates)</span></Button
		>
		<Button
			on:click={() => {
                console.log("Answer : ",answer);
                
				socket.emit('getIceCandidates', {
					userName: userName,
					socketId: answer.socketId
				});
			}}>Get Answeres ICE Candidates</Button
		>
	</div>
	<!-- <div class="flex h-auto max-w-7xl flex-row flex-wrap gap-3 p-2">
		{#each availableOffers as avail}
			{#if avail.userName !== userName}
				<Button
					class="text-md rounded bg-primary p-6"
					on:click={() => handleStreamWithOffer(avail.content)}
				>
					{avail.userName}
				</Button>
			{/if}
		{/each}
	</div> -->
</section>
