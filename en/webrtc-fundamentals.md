# WebRTC Fundamentals: Building a Video Calling Application

**WebRTC (Web Real-Time Communication)** is a technology that enables browsers and applications to communicate in real time, allowing video calls, audio exchange, and data transfer without the need for plugins. This technology, operating over peer-to-peer connections, integrates multiple APIs and protocols to establish robust, secure, and low-latency communication.

In this article, we will explore the fundamental concepts of WebRTC, using real examples from the [carubbi-video-link](https://github.com/rcarubbi/carubbi-video-link) repository and complementing them with insights from other articles and official documentation to ensure the explanation is as clear and comprehensive as possible.

---

## How Does WebRTC Work?

Essentially, WebRTC enables direct communication between browsers (or applications) without needing to route media traffic through a central server. However, for two peers to establish a connection, an initial exchange of information (signaling) is required to define how the data will be transmitted. This exchange involves:

- **Session Description Protocol (SDP):** Defines session parameters such as codecs, media formats, and transport types.
- **ICE (Interactive Connectivity Establishment):** A framework that identifies the best route for the connection.
- **STUN/TURN:** Servers that assist in discovering the public IP address and, if necessary, act as intermediaries when direct connections are not possible.

> These steps ensure that communication can be efficiently established even in networks with NATs and firewalls.

---

## Media Capture with `getUserMedia`

The first step in a WebRTC application is capturing the user's audio and video. The `navigator.mediaDevices.getUserMedia` API requests access to capture devices (such as a camera and microphone) and returns a `MediaStream` object.

In the **carubbi-video-link** repository, this step is implemented robustly. Here’s a simplified example:

```javascript
async function requestMediaPermissions() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true,
    });
    // To avoid keeping the camera active unnecessarily, the tracks are stopped.
    stream.getTracks().forEach((track) => track.stop());
  } catch (err) {
    ui.showError("Access to user media denied");
  }
}
```
This code is necessary for the browser to request permissions to access audio and video devices, allowing us to then call the `enumerateDevices` method, which populates the dropdown lists, enabling the user to select the device they want to use for the call. This selection is available on the settings screen.

Additionally, when starting a call, the code captures media using **constraints** that define parameters such as resolution, frame rate, and audio settings (echo cancellation, noise suppression, etc.):

```javascript
_localStream = await navigator.mediaDevices.getUserMedia({
  video: { deviceId: _videoDeviceId, ...videoConstraints },
  audio: { deviceId: _audioDeviceId, ...audioConstraints },
});
ui.showLocalVideoStream(_localStream);
```
> This approach ensures high definition and quality, adapting the available devices according to the user's needs.

---

## Connectivity

### ICE Framework

To discuss connectivity, it is important to understand **ICE (Interactive Connectivity Establishment)**, which is an essential part of the WebRTC stack responsible for discovering and selecting the best route for communication between two peers.

ICE is a set of methods and protocols that allows browsers (or other applications) to identify possible network paths (candidates) to establish a peer-to-peer connection. These candidates can be:

- **Host candidates:** Local addresses of the machine.
- **Server Reflexive candidates (STUN):** Public addresses discovered via **STUN (Session Traversal Utilities for NAT)** servers, which will be explained later.
- **Relayed candidates (TURN):** Addresses provided by **TURN (Traversal Using Relays around NAT)** servers, which will also be explained later.

These candidates are exchanged between peers through the **Signaling Server**. The exchange of candidates is crucial for both sides to determine which addresses can be used for the connection.

ICE first attempts to establish a direct connection. If this fails, it tries a **STUN** server to discover a possible path. If the connection is still not possible due to restrictive firewalls or NATs, ICE uses a **TURN server**, which acts as a relay to ensure communication.

---

#### The Candidate Exchange Process

Within the WebRTC context, the exchange of ICE candidates occurs as follows:

1. **Candidate Collection:**  
   When an `RTCPeerConnection` is created, it begins collecting possible candidates based on available devices and network conditions. This collection is performed asynchronously.

2. **`onicecandidate` Callback:**  
   Each time a new candidate is found, the `onicecandidate` callback is triggered. An example extracted from the [webrtc.js](https://github.com/rcarubbi/carubbi-video-link/blob/main/client/src/js/webrtc.js) file demonstrates:

   ```javascript
   _peerConnection.onicecandidate = (event) => {
     if (event.candidate) {
       signalingClient.sendIceCandidate({
         candidate: event.candidate,
         remoteUserId: _remoteUserId,
       });
     }
   };
   ```

   In this snippet, each candidate is sent to the remote peer through the signaling mechanism (abstracted here by `signalingClient`).

3. **Adding Candidates to the Connection:**  
   On the receiving side, the candidates are added to the `RTCPeerConnection` instance using the `addIceCandidate` method.

  ```javascript
   export async function addIceCandidate(candidate) {
    if (!_peerConnection || !_peerConnection.remoteDescription) {
        _pendingIceCandidates.push(candidate);
    } else {
        await _peerConnection.addIceCandidate(new RTCIceCandidate(candidate));
    }
   }
   ```

   ---

   #### Why Store Candidates Temporarily?

A common issue during connection establishment is that ICE candidates may arrive **before** the remote peer is ready to process them. More specifically, the `addIceCandidate()` method only works correctly if the remote description (obtained via `setRemoteDescription()`) has already been set on the connection. If candidates are sent before this configuration, the `RTCPeerConnection` is not yet able to integrate them, potentially resulting in errors or the loss of important candidates.

To prevent this issue, the example in the repository uses an array (`_pendingIceCandidates`) to temporarily store received ICE candidates. Once the remote description is set, a function (such as `flushPendingIceCandidates()`) is called to add all the stored candidates:

```javascript
async function flushPendingIceCandidates() {
  for (const candidate of _pendingIceCandidates) {
    await _peerConnection.addIceCandidate(new RTCIceCandidate(candidate));
  }
  _pendingIceCandidates = [];
}
```

This mechanism ensures that no candidate is discarded and that all are processed as soon as the connection is ready to receive new candidates.

### STUN and TURN Servers

ICE relies on two essential components to enable peer-to-peer connections even in networks with NAT and firewalls: **STUN** and **TURN**. Each has distinct functions and different impacts in terms of cost and infrastructure.

---

#### STUN (Session Traversal Utilities for NAT)

STUN is used to allow a client to discover its public IP address and the port used for the connection. This is done by sending a request to a STUN server, which returns the necessary information.

Since STUN only provides address information without relaying traffic, it does not require much bandwidth. As a result, many companies offer STUN servers for free. For example, Google provides servers such as `stun.l.google.com:19302`.

---

#### TURN (Traversal Using Relays around NAT)

TURN is used when a direct connection cannot be established (for example, in networks with restrictive NATs or firewalls). In this case, media traffic is relayed through a TURN server, acting as an intermediary.

Because TURN needs to relay audio and video streams, it consumes a significant amount of bandwidth. Due to its high resource demand, TURN services are usually charged.

You can also set up a TURN server using **COTURN**, an open-source solution. However, this requires dedicated infrastructure and sufficient bandwidth to support relay traffic.

---

##### Practical Example: Using the Metered Service

In the [carubbi-video-link](https://github.com/rcarubbi/carubbi-video-link/blob/main/client/src/js/webrtc.js) repository, we can see the ICE configuration, which includes both STUN and TURN servers. For example:

```javascript
const iceServers = [
  { urls: "stun:stun.relay.metered.ca:80" },
  {
    urls: "turn:global.relay.metered.ca:80",
    username: "**********",
    credential: "**********",
  },
  // Other TURN configurations, including TCP and secure (turns) options
];
```

In this case, we are using the [Metered](https://dashboard.metered.ca) service, which offers:
- A **free tier** with up to 0.5GB of monthly data for stream relaying.
- If usage exceeds this limit, the account is charged based on data consumption.

This approach is quite practical for development or small-scale applications where traffic volume is limited. For larger applications, scalability may need to be considered, or even setting up your own TURN server with COTURN.

---

### Configuring `RTCPeerConnection`

Now that we have explored the concepts of ICE, STUN, and TURN, let's take a deeper look at the classes provided by the WebRTC API.

The core of WebRTC communication is the `RTCPeerConnection` object, which manages the direct connection between peers. In the repository example, the `createPeerConnection` function configures the object with a list of ICE servers (including STUN and TURN):

```javascript
async function createPeerConnection() {
  _peerConnection = new RTCPeerConnection({ iceServers });
  
  _peerConnection.onicecandidate = (event) => {
    if (event.candidate) {
      signalingClient.sendIceCandidate({
        candidate: event.candidate,
        remoteUserId: _remoteUserId,
      });
    }
  };

  _peerConnection.ontrack = (event) => {
    ui.showRemoteVideoStream(event.streams[0]);
  };

  // Add all tracks from local stream to the connection
  _localStream.getTracks().forEach((track) => {
    _peerConnection.addTrack(track, _localStream);
  });
}
```

> This function also registers callbacks for receiving ICE candidates and displaying remote media in the user interface.

### Negotiation Process (Offer/Answer)

The process of initiating a call involves the following steps:

1. **Local media capture:** Using `getUserMedia`, as previously shown.
2. **Creating an offer (Offer):** Client A, who initiates the call, generates a session description.
3. **Sending the offer via signaling:** The offer is transmitted to Client B through a signaling server (typically via WebSocket or another mechanism).
4. **Creating an answer (Answer):** The receiver (Client B), upon receiving the offer, assigns it to its `RTCPeerConnection`, encapsulating the offer in an `RTCSessionDescription` object. Then, it configures its media, creates an answer, and sends it back to Client A.
5. **Receiving the answer:** Upon receiving the answer, Client A assigns it to its `RTCPeerConnection`, also encapsulating it in an `RTCSessionDescription`.
6. **ICE Candidate Exchange:** During and after this negotiation, both sides exchange candidates to optimize the connection route, as explained in the ICE chapter.

Example of the function that initiates a call:

```javascript
export async function startCall({ localUserId, remoteUserId }) {
  _remoteUserId = remoteUserId;
  
  // Capture the local media with defined constraints
  _localStream = await navigator.mediaDevices.getUserMedia({
    video: { deviceId: _videoDeviceId, ...videoConstraints },
    audio: { deviceId: _audioDeviceId, ...audioConstraints },
  });
  ui.showLocalVideoStream(_localStream);
  
  await createPeerConnection();
  
  // Create the offer and configure the local description
  const offer = await _peerConnection.createOffer();
  await _peerConnection.setLocalDescription(offer);
  
  // send offer to remote user
  signalingClient.sendOffer({
    to: _remoteUserId,
    from: localUserId,
    offer,
  });
}
```

> This flow follows the "offer/answer" pattern defined by the SDP protocol and is essential for synchronizing and establishing the P2P connection.

---

### Signaling and Data Flow

Although WebRTC defines the model for media and data exchange, the signaling mechanism—responsible for the initial exchange of information (such as SDP and ICE candidates)—is not specified. It is up to the developer to implement signaling using tools like WebSockets, SIP, or other technologies.

> This flexibility allows each application to adapt the signaling mechanism to its specific needs.

---

## Session and Resource Management

In addition to establishing the connection, the application must manage media exchange throughout the call. Functions for toggling audio or video (e.g., `toggleAudio` and `toggleVideo`) demonstrate how to dynamically control media resources:

```javascript
export function toggleVideo() {
  _localStream.getVideoTracks()[0].enabled = !_localStream.getVideoTracks()[0].enabled;
}

export function toggleAudio() {
  _localStream.getAudioTracks()[0].enabled = !_localStream.getAudioTracks()[0].enabled;
}
```

Other important functions include:
- **Accepting or rejecting a call:** Processing the received offer and sending a response.
- **Ending the call:** Stopping media tracks, closing the connection, and releasing resources.

> These controls are essential for a smooth and responsive communication experience.

---

## Conclusion

WebRTC has revolutionized the way we implement real-time communications, enabling video calls and data exchange to occur directly between browsers without the need for plugins or heavy infrastructure. By exploring the concepts of media capture, P2P connection establishment, SDP negotiation, and the use of ICE/STUN/TURN, you can build robust and scalable applications.

The example from the [carubbi-video-link](https://github.com/rcarubbi/carubbi-video-link) repository provides a practical demonstration of how to implement these concepts using JavaScript, offering a solid foundation for you to start developing your own communication solutions.

If you want to dive deeper, I recommend reading the [MDN WebRTC documentation](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API) and exploring tutorials that cover both the theoretical and practical aspects of the technology.
