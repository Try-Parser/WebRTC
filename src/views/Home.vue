<template>
  <div class="home">
	<video src="" ref="localVideo" autoplay></video>
	<video src="" ref="remoteVideo" autoplay></video>
	<br>
	<button @click="start">Start</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import HelloWorld from '@/components/HelloWorld.vue'; // @ is an alias to /src

@Component({
  components: {
    HelloWorld,
  },
})
export default class Home extends Vue {
	localVideo: any = null
	remoteVideo: any = null
	localStream: any = null
	socket: any  = null

	connected: boolean = false

	config: any = { iceServers: [{url: "stun:stun.l.google.com:19302"}]}
	pc: any = new RTCPeerConnection(this.config)
	pc2 = new RTCPeerConnection(this.config);

	mediaConstraints: any = {
	  mandatory: {
	    OfferToReceiveAudio:true, 
	    OfferToReceiveVideo:true
	  }
	}

	onmessage(message: any) {
		var msg = JSON.parse(message.data);

		console.log(msg)

		switch(msg.type) {
			case 'offer':
				this.consumeOffer(msg.offer)
				break;
			case 'answer':
				this.readAnswer(msg.answer)
				break;
			case 'candidate':
				this.setCandidate(msg.candidate)
				break;
		}
	}

	readAnswer(answer: any) { this.pc.setRemoteDescription(answer).catch((err: any) => console.log(err)) }

	setCandidate(candidate: any) { this.pc.addIceCandidate(candidate).catch((err: any) => console.log(err)) }

	consumeOffer(offer: any) { 
		this.pc
			.setRemoteDescription(new RTCSessionDescription(offer))
			.then(() => {
   				this.pc.createAnswer().then((answer: any) => {
   					this.pc.setLocalDescription(answer).then(() => {
   						this.socket.send(this.stringify({ type: "answer", answer: answer}))
   					})
   				})
  			})
	}

	mounted() {
		this.socket = new WebSocket('ws://192.168.0.103:8080')

		this.localVideo = this.$refs.localVideo
		this.remoteVideo = this.$refs.remoteVideo

		this.socket.onmessage = this.onmessage

		navigator.mediaDevices.getUserMedia({ audio: true, video: true }).then((stream: any) => {
			this.localVideo.srcObject = stream
			this.pc.addStream(stream)

			this.pc.ontrack = (e: any) => this.remoteVideo.srcObject = e.streams[0]
			this.pc.onicecandidate = (ice: any) => {
				console.log(ice)
				if(ice.candidate)
					this.socket.send(this.stringify({ type: "candidate", candidate: ice.candidate}))
			}
		}).catch(function(err) {
			console.log(err.name + ": " + err.message); 
		})
	}

	stringify = (data: any) => JSON.stringify(data)

	start() {
		this.pc
			.createOffer(this.mediaConstraints)
			.then((description: any) => this.socket.send(this.stringify({ type: "offer", offer: description })))
	}

}
</script>
