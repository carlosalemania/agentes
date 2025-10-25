# WebRTC Examples

## Example: Simple Video Chat

```javascript
// Client-side video chat
class VideoChat {
  constructor(roomId) {
    this.roomId = roomId;
    this.localStream = null;
    this.peers = new Map();
    this.socket = io('https://signaling.example.com');

    this.setupSignaling();
  }

  async init() {
    // Get local media
    this.localStream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true,
    });

    // Display local video
    document.getElementById('localVideo').srcObject = this.localStream;

    // Join room
    this.socket.emit('join-room', this.roomId);
  }

  setupSignaling() {
    // New user joined
    this.socket.on('user-connected', async (userId) => {
      console.log('User connected:', userId);
      await this.connectToUser(userId, true);
    });

    // Receive offer
    this.socket.on('offer', async ({ from, offer }) => {
      await this.handleOffer(from, offer);
    });

    // Receive answer
    this.socket.on('answer', async ({ from, answer }) => {
      const peer = this.peers.get(from);
      if (peer) {
        await peer.pc.setRemoteDescription(new RTCSessionDescription(answer));
      }
    });

    // Receive ICE candidate
    this.socket.on('ice-candidate', async ({ from, candidate }) => {
      const peer = this.peers.get(from);
      if (peer) {
        await peer.pc.addIceCandidate(new RTCIceCandidate(candidate));
      }
    });

    // User disconnected
    this.socket.on('user-disconnected', (userId) => {
      console.log('User disconnected:', userId);
      this.removePeer(userId);
    });
  }

  async connectToUser(userId, createOffer) {
    const pc = new RTCPeerConnection({
      iceServers: [{ urls: 'stun:stun.l.google.com:19302' }],
    });

    // Add local tracks
    this.localStream.getTracks().forEach((track) => {
      pc.addTrack(track, this.localStream);
    });

    // Handle incoming tracks
    pc.ontrack = (event) => {
      const [remoteStream] = event.streams;
      this.addRemoteVideo(userId, remoteStream);
    };

    // Handle ICE candidates
    pc.onicecandidate = (event) => {
      if (event.candidate) {
        this.socket.emit('ice-candidate', {
          to: userId,
          candidate: event.candidate,
        });
      }
    };

    this.peers.set(userId, { pc });

    if (createOffer) {
      const offer = await pc.createOffer();
      await pc.setLocalDescription(offer);

      this.socket.emit('offer', {
        to: userId,
        offer,
      });
    }
  }

  async handleOffer(from, offer) {
    await this.connectToUser(from, false);

    const peer = this.peers.get(from);
    await peer.pc.setRemoteDescription(new RTCSessionDescription(offer));

    const answer = await peer.pc.createAnswer();
    await peer.pc.setLocalDescription(answer);

    this.socket.emit('answer', {
      to: from,
      answer,
    });
  }

  addRemoteVideo(userId, stream) {
    const video = document.createElement('video');
    video.id = `video-${userId}`;
    video.srcObject = stream;
    video.autoplay = true;
    video.playsInline = true;

    document.getElementById('remoteVideos').appendChild(video);
  }

  removePeer(userId) {
    const peer = this.peers.get(userId);
    if (peer) {
      peer.pc.close();
      this.peers.delete(userId);

      const video = document.getElementById(`video-${userId}`);
      if (video) {
        video.remove();
      }
    }
  }

  async toggleVideo() {
    const videoTrack = this.localStream.getVideoTracks()[0];
    videoTrack.enabled = !videoTrack.enabled;
  }

  async toggleAudio() {
    const audioTrack = this.localStream.getAudioTracks()[0];
    audioTrack.enabled = !audioTrack.enabled;
  }

  disconnect() {
    this.localStream.getTracks().forEach((track) => track.stop());

    this.peers.forEach((peer) => peer.pc.close());
    this.peers.clear();

    this.socket.disconnect();
  }
}

// Usage
const chat = new VideoChat('room-123');
await chat.init();
```

## Example: Screen Sharing with Audio

```javascript
class ScreenShare {
  constructor() {
    this.screenStream = null;
    this.audioStream = null;
    this.combinedStream = null;
  }

  async start() {
    // Get screen
    this.screenStream = await navigator.mediaDevices.getDisplayMedia({
      video: { cursor: 'always' },
      audio: true, // System audio
    });

    // Get microphone
    this.audioStream = await navigator.mediaDevices.getUserMedia({
      audio: {
        echoCancellation: true,
        noiseSuppression: true,
      },
    });

    // Combine streams
    this.combinedStream = new MediaStream([
      ...this.screenStream.getVideoTracks(),
      ...this.screenStream.getAudioTracks(), // System audio
      ...this.audioStream.getAudioTracks(), // Microphone
    ]);

    // Handle stop button
    this.screenStream.getVideoTracks()[0].onended = () => {
      this.stop();
    };

    return this.combinedStream;
  }

  stop() {
    if (this.screenStream) {
      this.screenStream.getTracks().forEach((track) => track.stop());
    }

    if (this.audioStream) {
      this.audioStream.getTracks().forEach((track) => track.stop());
    }

    this.screenStream = null;
    this.audioStream = null;
    this.combinedStream = null;
  }
}

// Usage with peer connection
const screenShare = new ScreenShare();
const stream = await screenShare.start();

// Replace video track in existing peer connection
const videoTrack = stream.getVideoTracks()[0];
const sender = peerConnection
  .getSenders()
  .find((s) => s.track && s.track.kind === 'video');

if (sender) {
  await sender.replaceTrack(videoTrack);
}
```
