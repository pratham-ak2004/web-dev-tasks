<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import Input from '$lib/components/ui/input/input.svelte';
	import { onMount } from 'svelte';
	import io from 'socket.io-client';
	import { peerConfiguration } from '$lib/utils';

	let userName = '';
	let localStream: any;
	let remoteStream: any;
	let localStreamElement: HTMLVideoElement;
	let remoteStreamElement: HTMLVideoElement;
	let peerConnection: RTCPeerConnection;
	let availableOffers: any = [];
	let isOffered = false;

	$: {
		isOffered = peerConnection !== undefined;
		// console.log("availabel : " , availableOffers);
	}

	const socket = io(`${import.meta.env.VITE_SOCKET_SERVER}`, {
		autoConnect: false
	});
	// .connect();

	function connectToSocket() {
		if (!socket.active) {
			socket.auth = {
				userName
			};
			socket.connect();
		}
	}

	socket.on('getAvailableOffers', (data) => {
		console.log('get available : ', data);

		availableOffers = data;
	});

	socket.on('newOfferAwaiting', (offer: any) => {
		console.log('New Offer', offer);

		availableOffers.push(offer);
		availableOffers = availableOffers;
	});

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

	onMount(() => {
		function disconnectSocket() {
			if (peerConnection.connectionState === 'connected') {
				peerConnection.close();
			}
			socket.disconnect();
		}

		window.addEventListener('beforeunload', disconnectSocket);

		return () => window.removeEventListener('beforeunload', disconnectSocket);
	});

	async function handleJoinSocket() {
		if (userName.trim() !== '') {
			await fetchUserMedia();
			connectToSocket();
		}
	}

	async function handleJoinRoom() {
		if (userName.trim() !== '') {
			await fetchUserMedia();

			if (!localStream) {
				alert('Please allow camera access');
				return;
			}

			if (isOffered) {
				alert('You are already in a room');
				return;
			}

			createPeerToPeerConnection();

			try {
				const offer = await peerConnection.createOffer({});
				await peerConnection.setLocalDescription(offer);
				connectToSocket();
				socket.emit('newOffer', {
					content: offer,
					userName: userName,
					socketId: socket.id
				});
			} catch (err) {
				console.log(err);
			}
		}
	}

	function createPeerToPeerConnection(offerObj?: RTCSessionDescription) {
		return new Promise<void>(async(resolve, reject) => {
			peerConnection = new RTCPeerConnection(peerConfiguration);
			remoteStream = new MediaStream();
			remoteStreamElement.srcObject = remoteStream

			// Add local stream tracks to the connection
			localStream.getTracks().forEach((track: MediaStreamTrack) => {
				peerConnection.addTrack(track, localStream);
			});

			// will be invoked when new tracks arrive
			// Listen for remote tracks
			peerConnection.ontrack = (event) => {
				if (event.streams && event.streams[0]) {
					remoteStreamElement.srcObject = event.streams[0];
				}
			};

			// Handle ICE candidates
			peerConnection.onicecandidate = (event) => {
				if (event.candidate) {
					socket.emit('newIceCandidate', {
						candidate: event.candidate,
						userName: userName
						// isOffered: offerObj ? true : false
					});
				}
			};

			// Listen for connection state changes
			peerConnection.onconnectionstatechange = () => {
				if (peerConnection.connectionState === 'connected') {
					console.log('Peers connected!');
				}
			};

			if (offerObj) {
				await peerConnection.setRemoteDescription(offerObj);
			}
			resolve();
		});
	}

	async function handleStreamWithOffer(offer: any) {
		console.log('conneting with : ', offer);

		await fetchUserMedia();
		await createPeerToPeerConnection(offer);
	}

	// async function toggleFullScreen() {
	//     if (!document.fullscreenElement) {
	//         await videoElement.requestFullscreen();
	//     } else {
	//         await document.exitFullscreen();
	//     }
	// }

	// async function togglePictureInPicture() {
	//     if (!document.pictureInPictureElement) {
	//         await videoElement.requestPictureInPicture();
	//     } else {
	//         await document.exitPictureInPicture();
	//     }
	// }
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
	<div class="flex w-full max-w-7xl flex-row flex-nowrap justify-center gap-2">
		<Input
			placeholder="Enter username"
			class="w-72"
			bind:value={userName}
			disabled={socket.active}
		/>
		<Button on:click={handleJoinSocket} disabled={socket.active}>Join</Button>
		<Button on:click={handleJoinRoom} disabled={isOffered || !socket.active}>Call</Button>
	</div>
	<div class="flex h-auto max-w-7xl flex-row flex-wrap gap-3 p-2">
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
	</div>
</section>
