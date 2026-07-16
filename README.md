# VideoHub Client SDK

Lightweight browser SDK for building **real-time video, audio, and screen-sharing applications** using VideoHub infrastructure.
The SDK provides a simple API to connect users to rooms, publish media tracks, subscribe to remote participants, and handle real-time events.

It is designed to be **simple, extensible, and developer-friendly** while still exposing powerful real-time capabilities.

---

# Features

* Real-time **video and audio streaming**
* **Screen sharing**
* **Participant management**
* **Dynamic event system**
* **Remote track subscription**
* **Microphone control**
* Works in **modern browsers**
* Simple integration with **CDN or local script**

---

# Installation

## Option 1 — CDN

```html
<script src="https://cdn.jsdelivr.net/npm/videohub-client/dist/videohub-client.umd.min.js"></script>
```

## Option 2 — Local Script

Download the SDK and include it:

```html
<script src="/js/videohub-client.umd.min.js"></script>
```

---

# Basic Usage

## 1. Initialize SDK

```javascript
const hub = new VideoHub({
  ws: rtc
});
```

**Parameters**

| Option | Description          |
| ------ | -------------------- |
| ws     | rtc server           |

---

# Join a Room

To join a video room you need an **access token from your backend**.

```javascript
const token = "SERVER_GENERATED_TOKEN";

await hub.join({
  token: token,
  autoPublish: true
});
```

**Options**

| Parameter   | Description                              |
| ----------- | ---------------------------------------- |
| token       | Room access token                        |
| autoPublish | Automatically publish camera/mic on join |

---

# Publish Camera & Microphone

Start sending camera and microphone streams.
### Option 1 — Publish tracks manually
```javascript
await hub.media.publishCustomTracks({
  video: true,
  audio: true
});
```

### Option 2 — Publish tracks manually
```javascript
hub.events.on("localTrackPublished", (pub) => {
  const tracks = hub.media.localTracks;
  const videoTrack = tracks.find(t => t.kind === "video");

  if (videoTrack) {
    const el = videoTrack.attach();
    el.muted = true;
    el.autoplay = true;
    el.playsInline = true;
  }
});
```

Stop camera and microphone:

```javascript
await hub.media.unpublishTracks();
```

---

# Toggle Microphone

```javascript
await hub.media.toggleMic();
```

Returns:

```
true  = microphone enabled
false = microphone muted
```

---

# Screen Sharing

## Start screen share

```javascript
await hub.screen.start({
  video: true,
  audio: false
});
```

## Stop screen share

```javascript
await hub.screen.stop();
```

Check screen state:

```javascript
hub.screen.isActive()
```

---

# Event System

VideoHub provides a flexible **event emitter** to listen for real-time room events.

```javascript
hub.events.on("eventName", callback)
```

Example:

```javascript
hub.events.on("participantConnected", (participant) => {
  console.log("User joined:", participant.identity);
});
```

---

# Track Events

Listen for remote tracks.

```javascript
hub.events.on("trackSubscribed", (track, publication, participant) => {

  if (track.kind === "video") {

    const video = track.attach();
    document.body.appendChild(video);

  }

});
```

Parameters:

| Parameter   | Description        |
| ----------- | ------------------ |
| track       | Media track        |
| publication | Track metadata     |
| participant | Remote participant |

---

# Common Events

| Event                    | Description              |
| ------------------------ | ------------------------ |
| connected                | Connected to room        |
| disconnected             | Disconnected from room   |
| reconnecting             | Reconnection started     |
| reconnected              | Reconnected successfully |
| participantConnected     | New user joined          |
| participantDisconnected  | User left room           |
| trackSubscribed          | Remote track received    |
| trackUnsubscribed        | Remote track removed     |
| trackPublished           | Track published          |
| trackUnpublished         | Track removed            |
| dataReceived             | Data message received    |
| activeSpeakersChanged    | Active speaker update    |
| connectionQualityChanged | Network quality changed  |

---

# Render Video Example

```javascript
hub.events.on("trackSubscribed", (track, pub, participant) => {

  if (track.kind === "video") {

    const video = track.attach();
    video.autoplay = true;
    video.playsInline = true;

    document.getElementById("videos").appendChild(video);

  }

});
```

---

# Access Remote Participants

```javascript
const participants = hub.core.getRemoteParticipants();

participants.forEach(p => {
  console.log(p.identity);
});
```

---

# Leave Room

```javascript
await hub.destroy();
```

This will:

* disconnect from the room
* stop all media tracks
* clean up internal resources

---

# Example HTML Layout

```html
<div id="videos"></div>
<button onclick="startCamera()">Start Camera</button>
<button onclick="shareScreen()">Share Screen</button>
```

---

# Browser Requirements

Supported browsers:

* Chrome
* Edge
* Firefox
* Safari (latest versions)

Ensure the site runs over:

```
https
```

because camera and microphone require **secure context**.

---

# Backend Token Generation

The SDK requires a **server-generated token** to join rooms securely.

Typical flow:

```
Client → Request token
Server → Generate token
Client → Join room with token
```

Example API request:

```
POST /api/video/create-host
```

Response:

```json
{
  "token": "ACCESS_TOKEN"
}
```

---

# Best Practices

* Always **destroy the room** when leaving.
* Handle **participantDisconnected** to clean UI.
* Handle **trackUnsubscribed** to remove video elements.
* Avoid auto-publishing tracks unless necessary.

---

# Example Project Structure

```
project
 ├── index.html
 ├── app.js
 └── videohub-client.umd.min.js
```

---

# License

MIT License

---

# Contributing

Contributions and improvements are welcome.

Steps:

1. Fork repository
2. Create feature branch
3. Submit pull request

---

# Support

If you encounter issues or need help integrating VideoHub, please open an issue in the repository.

---

# Summary

VideoHub Client SDK enables developers to build **real-time video communication apps** with minimal setup.

With a simple API and flexible event system, you can quickly implement:

* video calls
* group meetings
* webinars
* screen sharing
* live collaboration

---

# Real-Time Messaging (RTM)

VideoHub provides built-in real-time messaging features including:

* Text messages
* Stickers
* GIFs / Animated Images
* Emoji Reactions
* File Attachments
* Call Invitations

---

# Send Text Message

```javascript
hub.chat.send(
  roomId,
  "Hello VideoHub 👋"
);
```

---

# Receive Messages

```javascript
hub.rtm.on(
  hub.rtm.EVENTS.CHAT_MESSAGE,
  payload => {

    console.log(
      payload.sender_id,
      payload.message
    );

  }
);
```

---

# Send Sticker

```javascript
hub.chat.sendRichMessage(
  roomId,
  {
    
    type: "sticker",
    sticker: stickerUrl
                  
  }
);
```

---

# Receive 

```javascript
hub.rtm.on()
```

---

# Send GIF / Animated Image

```javascript
{
  type: "gif",
  url: gif
}
```

---

# Send Attachment Message

```javascript
hub.attachment.send(
  roomId,
  [
    {
      file_id:
        attachment.attachment_id,

      name:
        attachment.file_name,

      size:
        attachment.file_size,

      mime_type:
        attachment.mime_type,

      url:
        attachment.file_url
    }
  ]
);
```

---

# Download Attachment

---

# Call Invitation

```javascript
hub.call.invite(
  targetUserID,
  roomID,
  type
)
```

---

# RTM Events 

| Event                                 | Description                           |
| ------------------------------------- | ------------------------------------- |
| CHAT_MESSAGE                          | Text, sticker, GIF, reaction message  |
| CHAT_DELETE                           | Message deleted                       |
| TYPING_START                          | User started typing                   |
| TYPING_STOP                           | User stopped typing                   |
| USER_JOINED                           | User joined room                      |
| USER_LEFT                             | User left room                        |
| PRESENCE_UPDATE                       | Presence status update                |
| ROOM_STATE_SYNC                       | Room state synchronization            |

---

## Documentation

Full API documentation is available at:

https://docs.videohub.dev