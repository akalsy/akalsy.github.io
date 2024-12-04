---
title: WebRTC视频通话Demo
date: 2024-11-24 22:59:13
tags:
  - WebRTC
---

使用 React 和 WebRTC 实现一个简单的视频通话 DEMO，WebRTC（Web Real-Time Communication）是一个支持网页和移动应用进行实时音视频通信的技术。下面我将详细描述如何创建一个基本的视频通话 DEMO。

### 1. 准备工作

首先，我们需要确保以下几点：

* 你已经安装了 Node.js 和 npm（或 yarn）。
* React 项目已经创建并运行起来（如果还没有，可以使用 `create-react-app` 创建一个新的 React 项目）。

### 2. 创建 React 项目

如果你还没有 React 项目，可以通过以下命令创建一个新的 React 项目：

```bash
npx create-react-app webrtc-demo
cd webrtc-demo
npm start
```

### 3. 安装 WebRTC 所需依赖

WebRTC 本身不需要外部依赖，但为了更方便的管理代码，可以使用 WebRTC API 自带的功能。

### 4. 代码结构

为了实现视频通话功能，涉及到以下几个关键的技术点：

* 获取用户的媒体（摄像头和麦克风）流。
* 创建和管理 WebRTC 连接。
* 处理 ICE（Interactive Connectivity Establishment）候选者。
* 建立信号传输机制（例如使用 WebSocket 进行信令交换）。

假设我们使用 WebSocket 作为信令服务器，前后端通信可以通过 WebSocket 来传递 SDP（Session Description Protocol）和 ICE 候选者。

### 5. 完整示例代码

#### 1. 创建一个信令服务器（使用 Node.js 和 WebSocket）

首先，你需要一个简单的 WebSocket 信令服务器来传递 SDP 和 ICE 候选信息。可以创建一个新的 Node.js 项目并安装 `ws` 库：

```bash
npm init -y
npm install ws
```

在 `server.js` 文件中：

```js
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
    ws.on('message', (message) => {
        // 广播给所有连接的客户端
        wss.clients.forEach(client => {
            if (client !== ws && client.readyState === WebSocket.OPEN) {
                client.send(message);
            }
        });
    });
});
```

启动信令服务器：

```bash
node server.js
```

#### 2. React 前端实现

在 `src` 文件夹中，创建一个 `VideoCall.js` 组件来管理视频通话的逻辑。

```jsx
// VideoCall.js
import React, { useState, useRef, useEffect } from 'react';

// WebSocket 服务器地址
const SIGNALING_SERVER = "ws://localhost:8080";

function VideoCall() {
  const [isCaller, setIsCaller] = useState(false);
  const localVideoRef = useRef(null);
  const remoteVideoRef = useRef(null);
  const peerConnectionRef = useRef(null);
  const socketRef = useRef(null);

  useEffect(() => {
    // 初始化 WebSocket 连接
    socketRef.current = new WebSocket(SIGNALING_SERVER);
  
    socketRef.current.onmessage = (message) => {
      const data = JSON.parse(message.data);
    
      if (data.type === 'offer') {
        handleOffer(data);
      } else if (data.type === 'answer') {
        handleAnswer(data);
      } else if (data.type === 'candidate') {
        handleCandidate(data);
      }
    };

    return () => {
      if (socketRef.current) {
        socketRef.current.close();
      }
    };
  }, []);

  // 获取用户的媒体流（摄像头和麦克风）
  const startStream = async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: true,
        audio: true,
      });
      localVideoRef.current.srcObject = stream;
    } catch (error) {
      console.error("Error accessing media devices.", error);
    }
  };

  // 创建 WebRTC 连接并发送 offer
  const startCall = async () => {
    setIsCaller(true);
    const peerConnection = new RTCPeerConnection({
      iceServers: [{ urls: "stun:stun.l.google.com:19302" }],
    });

    peerConnectionRef.current = peerConnection;

    // 获取本地媒体流并添加到 PeerConnection 中
    const localStream = localVideoRef.current.srcObject;
    localStream.getTracks().forEach(track => peerConnection.addTrack(track, localStream));

    // 创建 offer 并发送给对方
    const offer = await peerConnection.createOffer();
    await peerConnection.setLocalDescription(offer);

    socketRef.current.send(
      JSON.stringify({ type: 'offer', offer: offer })
    );

    // 监听 ICE 候选者
    peerConnection.onicecandidate = (event) => {
      if (event.candidate) {
        socketRef.current.send(
          JSON.stringify({ type: 'candidate', candidate: event.candidate })
        );
      }
    };

    // 监听远程流
    peerConnection.ontrack = (event) => {
      remoteVideoRef.current.srcObject = event.streams[0];
    };
  };

  // 处理收到的 offer
  const handleOffer = async (data) => {
    const peerConnection = new RTCPeerConnection({
      iceServers: [{ urls: "stun:stun.l.google.com:19302" }],
    });

    peerConnectionRef.current = peerConnection;

    const localStream = localVideoRef.current.srcObject;
    localStream.getTracks().forEach(track => peerConnection.addTrack(track, localStream));

    // 设置远程描述
    await peerConnection.setRemoteDescription(new RTCSessionDescription(data.offer));

    // 创建 answer 并发送给对方
    const answer = await peerConnection.createAnswer();
    await peerConnection.setLocalDescription(answer);

    socketRef.current.send(
      JSON.stringify({ type: 'answer', answer: answer })
    );

    // 监听 ICE 候选者
    peerConnection.onicecandidate = (event) => {
      if (event.candidate) {
        socketRef.current.send(
          JSON.stringify({ type: 'candidate', candidate: event.candidate })
        );
      }
    };

    // 监听远程流
    peerConnection.ontrack = (event) => {
      remoteVideoRef.current.srcObject = event.streams[0];
    };
  };

  // 处理收到的 answer
  const handleAnswer = (data) => {
    peerConnectionRef.current.setRemoteDescription(new RTCSessionDescription(data.answer));
  };

  // 处理收到的 ICE 候选者
  const handleCandidate = (data) => {
    const candidate = new RTCIceCandidate(data.candidate);
    peerConnectionRef.current.addIceCandidate(candidate);
  };

  return (
    <div>
      <h2>WebRTC Video Call</h2>
      <div>
        <video ref={localVideoRef} autoPlay muted />
        <video ref={remoteVideoRef} autoPlay />
      </div>
      <div>
        {!isCaller && <button onClick={startCall}>Start Call</button>}
        <button onClick={startStream}>Start Stream</button>
      </div>
    </div>
  );
}

export default VideoCall;
```

#### 3. 在 `App.js` 中引入 `VideoCall` 组件

```jsx
// App.js
import React from 'react';
import VideoCall from './VideoCall';

function App() {
  return (
    <div className="App">
      <h1>WebRTC Video Call Demo</h1>
      <VideoCall />
    </div>
  );
}

export default App;
```

### 6. 启动 React 项目

在项目根目录下，启动 React 项目：

```bash
npm start
```

### 7. 测试视频通话

1. 打开两个浏览器窗口，分别运行你的 React 应用。
2. 点击 `Start Stream` 按钮来获取摄像头和麦克风流。
3. 在其中一个窗口，点击 `Start Call` 按钮，另外一个窗口会接收到呼叫请求，并建立视频通话。

### 结语

这只是一个简单的 WebRTC 视频通话 Demo。实际生产环境中，还需要处理更多的情况，例如错误处理、信号服务器的健壮性、安全性等。希望这个示例能帮助你理解 WebRTC 的基本概念和如何在 React 中实现它！
