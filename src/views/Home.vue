<template>
  <div class="home">
	<video src="" ref="localVideo"></video>
	<video src="" ref="remoteVideo"></video>
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
	}

	mounted() {
		this.socket = new WebSocket('ws://192.168.0.103:9090')

		this.localVideo = this.$refs.localVideo
		this.remoteVideo = this.$refs.remoteVideo

		this.localVideo.addEventListener('loadedmetadata', () => {
			this.localVideo.play()
		});

		this.socket.onmessage = this.onmessage

		navigator.mediaDevices.getUserMedia({ audio: true, video: true }).then((stream: any) => {
			this.localVideo.srcObject = stream
			this.pc.addStream(stream)

			this.pc.ontrack = (e: any) => this.remoteVideo.srcObject = e.streams[0]
		}).catch(function(err) {
			console.log(err.name + ": " + err.message); 
		})
	}

	start() {
		// this.pc.createOffer(this.mediaConstraints).then(this.createOffer);
	}

}
</script>
