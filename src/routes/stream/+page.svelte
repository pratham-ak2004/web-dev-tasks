<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import Input from '$lib/components/ui/input/input.svelte';
	import { onMount } from 'svelte';
	import io from 'socket.io-client'

	let userName = '';
	let localStream: any;
	let remoteStream: any;
	let localStreamElement: HTMLVideoElement;
	let remoteStreamElement: HTMLVideoElement;
	let peerConnection:RTCPeerConnection;

	const socket = io('https://192.168.22.131:3000/', {
		autoConnect: false
	}).connect()
	function connectToSocket(){
		if(!socket.active){
			socket.connect();
		}
	}

	socket.on('newOfferAwaiting',(offer:any)=>{
		console.log("offer available",offer);
	})

	const peerConfiguration = {
		iceServers: [
			{
				urls: [
					'stun:stun1.l.google.com:19302',
					'stun:stun2.l.google.com:19302',
					'stun:stun3.l.google.com:19302',
					'stun:stun4.l.google.com:19302'
				]
			}
		]
	};

	onMount(async () => {
		function fetchUserMedia() {
			return new Promise<void>(async (resolve, reject) => {
				try {
					localStream = await navigator.mediaDevices.getUserMedia({ video: true });
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

		await fetchUserMedia();
	});

	async function handleJoinRoom() {
		if (userName.trim() !== 'a') {
			// alert(`you are joining as ${userName}`);
			createPeerToPeerConnection();

			try {
				const offer = await peerConnection.createOffer({})
				await peerConnection.setLocalDescription(offer)
				connectToSocket();
				socket.emit('newOffer',offer)
			} catch (err) {
				console.log(err);
			}
		}
	}

	function createPeerToPeerConnection() {
		peerConnection = new RTCPeerConnection(peerConfiguration);

		// Add local stream tracks to the connection
		localStream.getTracks().forEach((track: MediaStreamTrack) => {
			peerConnection.addTrack(track, localStream);
		});

        // Listen for remote tracks
        peerConnection.ontrack = event => {
            if (event.streams && event.streams[0]) {
                remoteStreamElement.srcObject = event.streams[0];
            }
        };

        // Handle ICE candidates
        peerConnection.onicecandidate = event => {
            if (event.candidate) {
                console.log('Send this candidate to the remote peer:', event.candidate);
            }
        };

        // Listen for connection state changes
        peerConnection.onconnectionstatechange = () => {
            if (peerConnection.connectionState === 'connected') {
                console.log('Peers connected!');
            }
        };
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
			<video bind:this={remoteStreamElement} class="z-10 rounded-sm" autoplay muted playsinline>
				<track kind="captions" />
			</video>
			<video
				class="absolute bottom-2 right-2 z-20 w-20 rounded-lg sm:w-44"
				bind:this={localStreamElement}
				autoplay
				muted
				playsinline
			/>
		</div>
	</div>
	<div class="flex w-full max-w-7xl flex-row flex-nowrap justify-center gap-2">
		<Input placeholder="Enter username" class="w-72" bind:value={userName} />
		<Button on:click={handleJoinRoom}>Join</Button>
	</div>
</section>
