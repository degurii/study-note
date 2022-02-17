# Let's Build a Video Chat App with Javascript and WebRTC

#### Date: 2022. 02. 14.

#### Tags

- [x] webrtc
- [x] javascript

---

다음 글의 내용을 정리한 것

https://javascript.plainenglish.io/lets-build-a-video-chat-app-with-javascript-and-webrtc-de745072c38c

## Part1: Understanding WebRTC

### Fundamentals

1. 전형적인 채팅 웹 앱의 경우, 서버가 필요함

1. 중앙에 서버를 두면 브라우저간 커뮤니케이션에 딜레이가 생길 수 있음
1. 채팅 앱 같은건 딜레이가 크게 상관 없다
1. 근데 화상 회의 앱에서 딜레이가 생긴다? 진짜 개화난다
1. 그러니까 브라우저간 **Real-Time Communication**이 필요하다.
1. 이건 어떻게 해결하나, 중앙에 존재하는 서버를 없애버리자.
1. WebRTC는 이를 해결해줄 수 있다

### WebRTC

1. WebRTC === **Web Real-Time Communication**

1. WebRTC는 음성, 영상, 피어간 통신에 필요한 데이터 등을 교환하는 서버 없이 P2P 통신을 가능하게 한다.
1. 서버는 두 피어가 서로에 대한 정보를 알게 하고, 초기 연결을 수립하는 역할만 한다.
1. WebRTC 없이 P2P 통신이 필요한 앱을 구축하려면, 다음 이슈를 다루는 여러 프레임워크와 라이브러리가 필요하다.
   - data loss
   - connection dropping
   - NAT traversal
   - Echo cancellation
   - Bandwidth adptivity
   - Dynamic jitter buffering
   - Automatic gain control
   - Noise reduction and suppression
1. 브라우저에 내장된 WebRTC는 이런 이슈들을 알아서 해결해준다.
1. 사용하기 전에 브라우저가 WebRTC를 지원하는지 확인할 것.

### WebRTC APIs

1. 몇 가지 데모들 (링크)
   - [getUserMedia()](https://webrtc.github.io/samples/src/content/getusermedia/gum/): capture audio and video.
   - [MediaRecorder](https://webrtc.github.io/samples/src/content/getusermedia/record/): record audio and video.
   - [jRTCPeerConnection](https://webrtc.github.io/samples/src/content/peerconnection/pc1/): stream audio and video between users.
   - [RTCDataChannel](https://webrtc.github.io/samples/src/content/datachannel/basic/): stream data between users.

### Signaling

1. 피어끼리 통신하기 위해 많은 정보가 필요함

   - 통신 가능한 다른 피어가 있는지
   - 외부에서 보여지는 피어의 IP 주소, 포트 번호와 같은 **네트워크 정보**
   - **Session-control 메시지**: 통신을 open, close 하는데 사용
   - **에러 메시지**
   - **Media metadata**: 피어가 보낼 코덱, 코덱 설정, 대역폭, 미디어 유형 등
   - 보안 연결을 수립하는데 사용되는 **key data**

1. 뭐 일단은 얘네가 뭔지 다 이해할 필요는 없고, 많은 정보를 교환해야 직접 연결이 설정될 수 있다는 사실이 중요한거다.
1. 이런 정보들을 **metadata**라 한다.
1. **Signaling**은 초기 통신을 조정하고 피어끼리 메타데이터를 교환할 수 있게 해주는 매커니즘이다.
1. 피어는 처음에 시그널링 매커니즘을 이용하여 통신한다.
1. 주로 다른 피어를 발견하고 직접 연결을 만드는데 필요한 정보를 교환한다.
1. 일단 피어끼리 direct connection이 수립되면, 이후로는 시그널링이 필요 없다.

#### Session Description Protocol (SDP)

1. WebRTC는 시그널링 매커니즘을 구체적으로 명시하지 않는다.

1. 너네가 편한대로 해라 이거임
1. WebRTC는 딱 두 종류의 메타데이터를 교환한다.
1. **offer**와 **answer**
1. offer와 answer는 **SDP** 포맷으로 전달된다.
   ```
   v=0
   o=- 7614219274584779017 2 IN IP4 127.0.0.1
   s=-
   t=0 0
   a=group:BUNDLE audio video
   a=msid-semantic: WMS
   m=audio 1 RTP/SAVPF 111 103 104 0 8 107 106 105 13 126
   c=IN IP4 0.0.0.0
   ...
   ```
1. 복잡해보여도 걱정하지 말라, WebRTC가 사용자의 오디오, 비디오 장치에 따라 자동으로 만들어준다.

### So how does a WebRTC Application work?

1. 각 디바이스마다 IP 주소, PORT 번호가 있다.

1. `RTCPeerConnection` API는 사용자간 오디오, 비디오 스트리밍에 사용된다.
1. Signalling은 direct connection을 위해 `RTCPeerConnection`과 함께 사용된다.
1. 이 프로세스를 초기화하기 위해 `RTCPeerConnection`은 두 가지 일을 한다.
   - 해상도 및 codec capabilities(이게 뭐지?)와 같은 local media condition을 확인한다. 이건 **offer-and-answer**에 사용되는 메타데이터이고, 시그널링을 통해 전송된다.
   - 잠재적 네트워크 주소를 얻어온다. 이를 **candidate**라 한다.
1. 이런 로컬 데이터를 확인하고, 시그널링 매커니즘을 통해 다른 피어와 교환해야한다.

#### 영균이와 지원이를 예로 들어보자

1. 메타 데이터 교환
   ![image](https://user-images.githubusercontent.com/29361570/154531434-c9170e0e-6a79-4fd2-be3f-16276494476d.png)

   > :warning: 동기적으로 작동하는 것 처럼 설명하지만, 각 과정이 모두 동기적으로 작동하지는 않는다. 플로우만 생각하자.

   1. **영균**이는 `RTCPeerConnection` 객체를 생성한다.
   1. **영균**이는 `RTCPeerConnection`의 `createOffer()`메서드를 이용해 offer (SDP session description)를 생성한다.[^createoffer]
      [^createoffer]:`2022.02.18 추가`:
      원문에서는 offer를 따로 stringify 한다고 했지만, 아마 이제는 따로 처리해주지 않아도 되는듯 함.
      `createOffer()`는 `Promise<RTCSessionDescription>` 타입을 반환하고, `RTCSessionDescription`는 충분히 잘 serialize되어있기 때문인 것 같음. `createAnswer()`의 경우도 마찬가지.
      https://developer.mozilla.org/ko/docs/Web/API/RTCPeerConnection/setLocalDescription
   1. **영균**이는 생성될 connection의 local media의 description으로 위의 offer를 설정하기 위해 `setLocalDescription()`을 호출한다.
   1. **영균**이는 시그널링 매커니즘을 이용[^usesignalingmechanism]하여 stringify된 offer를 **지원**이에게 보낸다.
      [^usesignalingmechanism]: 어떤 방식으로든 전달하기만 하면 된다는 뜻.
      시그널링 방식에 대한 세부 구현은 webrtc 표준에 포함되어있지 않음.
   1. **지원**이는 **영균**이의 offer와 함께 `setRemoteDescription()`을 호출하여 `RTCPeerConnection`객체가 **영균**이의 정보를 알 수 있게 한다.
   1. **지원**이는 `createAnswer()`를 호출하고, 성공 콜백은 local session description(= **지원**이의 answer)으로 전달된다.(여긴 이해가 잘 안됨. 코드를 봐야 알 수 있을 듯. 원문: `Bernadette calls createAnswer() and the success callback function for this is passed a local session description—Bernadette's answer.`)[^createanswercallback]
      [^createanswercallback]:`2022.02.18 추가`:
      원문의 설명은 Promise 메서드가 지원되기 전 콜백 형식의 메서드를 기준으로 설명한 듯 함. 우린 그냥 평소에 promise 쓰는 것 처럼 하면됨
   1. **지원**이는 `setLocalDescription()`을 호출하여 answer를 local description으로 설정한다.
   1. **지원**이는 stringify된 answer[^createoffer]를 시그널링 매커니즘을 이용해[^usesignalingmechanism] **영균**이에게 전달한다.
   1. **영균**이는 **지원**이의 answer를 `setRemoteDescription()`을 이용하여 remote session description으로 설정한다.

1. **finding candidates**: ICE framework를 이용하여 **network interfaces and ports** 찾기
   1. **영균**이는 `onicecandidate`핸들러와 함께 `RTCPeerConnection` 객체를 만든다.
   1. 핸들러는 network candidates이 <U>활성화될 때(아마 내부적으로 이벤트가 dispatch 되었을 때)</U> 호출된다.
   1. 핸들러 내에서, **영균**이는 시그널링 채널을 통해 stringified candidate data를 **지원**이에게 보낸다.
   1. **영균**이한테 온 candidate 메시지를 받으면, **지원**이는 remote peer description에 candidate을 추가하기 위해 `addIceCandidate()`을 호출한다.
1. WebRTC는 ICE Candidate Trickling을 지원한다.
1. `WebRTC supports ICE Candidate Trickling, which allows the caller to incrementally and automatically provide candidates to the callee after the initial offer, and for the callee to automatically begin acting on the call and set up a connection without waiting for all candidates to arrive`(원문으로 보는게 의미가 잘 와닿아서 그냥 씀)
1. 중요 포인트: WebRTC는 피어가 offer를 만들면, 자동으로 ICE candidates를 생성한다는 것
1. ICE Candidate Trickling 잘 몰라도 됨. 우리는 candidates을 주고 받는데 사용할 메서드만 잘 구현해주면 된다 이말임
1. 위 두 과정을 거쳐 media condition, ice candidate이 피어간에 공유되면, WebRTC가 자동으로 direct connection을 만든다.

### After Signalling - Use ICE to cope with NATs and firewalls

1. 이거만 하면 통신할 수 있을 줄 알았지? 사실 두 가지 문제가 더 있다.

#### Problem 1 - NAT

![NAT](https://miro.medium.com/max/1400/1*EtSQG-F77u-aK8HSzedT1Q.png)

1. NAT 뭔지 알죠?
1. 각 라우터는 하나의 퍼블릭 IP를 여러개의 프라이빗 IP와 맵핑한다.
1. 연결된 디바이스(클라이언트)들은 퍼블릭 IP가 아니라 자기 프라이빗 IP만 알고 있음.
1. network ICE candidate에 퍼블릭 IP 대신 프라이빗 IP 주소가 포함되어 문제가 될 수 있다.
1. 그러니까, 브라우저가 퍼블릭 IP 주소를 알 수 있어야 candidate에 제대로 된 주소를 넣어줄 수 있음
1. **STUN(Session Traversal Utilities for NAT)** 서버를 이용하자.
1. 디바이스가 STUN 서버로 요청을 보내고, STUN 서버는 장치가 연결된 라우터의 퍼블릭 IP 정보를 포함하여 응답해준다.

#### Problem 2 - Firewall

1. 실상황에서 대부분의 디바이스는 하나 이상의 방화벽 계층 뒤에 존재할 수 있다.
1. WebRTC는 여러개의 non-standard 포트를 사용하기 때문에, 방화벽이 브라우저간 direct connection을 허용하지 않을 수 있음.
   ![TURN](https://miro.medium.com/max/1400/1*Xak-wVX3xUL_QgdD8u1zGQ.png)
1. **TURN(Traversal Using Relay NAT)** 서버를 이용하자.
1. 피어간 direct connection이 불발되면 TURN 서버가 사이에서 데이터를 교환해준다.

코드상에서는 다음과 같이 config를 넣어주기만 하면 된다.

```js
//Object containing TURN/STUN URLs.
var pcConfig = {
  'iceServers': [
    {
      'urls': 'stun:stun.l.google.com:19302'
    },
    {
      'urls': 'turn:192.158.29.39:3478?transport=udp',
      'credential': 'JZEOEt2V3Qb0y27GRntt2u2PAYA=',
      'username': '28224511:1379330808'
    },
    {
      'urls': 'turn:192.158.29.39:3478?transport=tcp',
      'credential': 'JZEOEt2V3Qb0y27GRntt2u2PAYA=',
      'username': '28224511:1379330808'
    }
  ]
}

........
//Passing the above object to RTCPeerConnection
RTCPeerConnection(pcConfig);
```

![final](https://miro.medium.com/max/1400/1*OyKvaHYQDvGHfK53RE2IOg.png)

[^createoffer]:
