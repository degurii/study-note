# WebRTC X React

## Media

### mediaDevices.enumerateDevices()

- 기기에 연결된 미디어 장치 목록을 불러오고싶으면

```js
const getConnectedDevices = async () => {
  const devices = await navigator.mediaDevices.enumerateDevices();
  return devices;
};
```

- 각 장치는 `MediaDeviceInfo` 타입

```js
interface MediaDeviceInfo {
    readonly deviceId: string;
    readonly groupId: string;
    readonly kind: MediaDeviceKind;
    readonly label: string;
    toJSON(): any;
}
```

| 속성       | 설명                                                                                                                     |
| ---------- | ------------------------------------------------------------------------------------------------------------------------ |
| `deviceId` | 각 장치별 고유 ID. 각 앱마다, 혹은 앱이 실행될 때마다 다를 수 있다. 같은 장치가 항상 같은 id를 가진다고 보장하지 않는다. |
| `groupId`  | 같은 물리적 장치 내의 두 장치(예를 들어 마이크, 스피커가 함께 있는 이어폰)는 같은 groupId를 가진다.                      |
| `kind`     | `videoinput`, `audioinput`, `audiooutput` 세 종류가 있다. 카메라, 마이크, 스피커라고 생각하면 됨.                        |
| `label`    | 각 장치의 이름. ex) `MacBook Air 마이크 (built-in)`, `AirPods`                                                           |

- mediaDevices의 `devicechange`이벤트를 구독하여 새로운 스피커를 연결하거나, 블루투스 장치가 연결/해제되는 등의 경우에 대한 동작을 정의할 수 있다.

```js
const handleDeviceChange = (evt) => { ... };
navigator.mediaDevices.addEventListener('devicechange', handleDeviceChange);
```

### mediaDevices.getUserMedia(constraints)

- `constraints`를 인자로 받아 적절한 `stream`을 반환한다. ([MediaStream - MDN](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream), [MediaTrackConstraints - MDN](https://developer.mozilla.org/en-US/docs/Web/API/MediaTrackConstraints))

- stream은 사실상 `track`이 본체라고 생각해도 무방한 듯 함. ([MediaStreamTrack - MDN](https://developer.mozilla.org/ko/docs/Web/API/MediaStreamTrack))

- constraints에는 video, audio 설정을 넣을 수 있다. 이때 audio는 output(스피커)이 아니라 input(마이크)이다.

- 만약 constraints에 video, audio 설정을 모두 포함했는데, 모종의 이유로 둘 중 하나라도 문제가 있으면 **둘 다 터진다.** 예를 들면 마이크가 없다거나, 원하는 해상도의 카메라가 없다던가 등

- 그러니까 한 번에 stream을 생성하지 말고, video, auido 각각 stream을 생성한 후 적절한 예외처리를 하고, 성공적으로 생성된 stream의 track을 하나로 합쳐주어야 한다.

- 나는 이런식으로 Custom Hook을 만들어 사용했다. 근데 **예외 처리 부분은 개발 진행하면서 더 세분화해서 핸들링해야 함.** 일단은 constraints가 간단하니까 에러나면 그쪽 장치는 안쓰는걸로 했음.

```js
const getMediaTracksWithConstraints = async (
  constraints: MediaStreamConstraints,
) => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    return [...stream.getTracks()];
  } catch (err) {
    return [];
  }
};

export function useUserMediaStream(constraints: MediaStreamConstraints) {
  const [userMediaStream, setUserMediaStream] = useState<MediaStream>(
    new MediaStream(),
  );

  useEffect(
    function () {
      const getUserMedia = async () => {
        const videoTracks = await getMediaTracksWithConstraints({
          video: constraints.video,
        });
        const audioTracks = await getMediaTracksWithConstraints({
          audio: constraints.audio,
        });
        const tracks = [...videoTracks, ...audioTracks];
        const stream = new MediaStream(tracks);

        setUserMediaStream((prevStream) => {
          prevStream.getTracks().forEach((track) => track.stop());
          return stream;
        });
      };

      getUserMedia();
    },
    [constraints],
  );

  return userMediaStream;
}
```

### input 장치 변경하기

- constraints에 deviceId를 실어 그 장치로 입력되는 stream을 얻어와야 함.
- 나는 현재 선택된 장치 정보를 유지하면서, 이게 변경되면 constraints를 업데이트 해서 위의 hook에 넣어줌.

```js
type SelectedDevice = Record<MediaDeviceKind, MediaDeviceInfo | null>;

const initialSelectedDevice: SelectedDevice = {
  audioinput: null,
  audiooutput: null,
  videoinput: null,
};

const MediaStore = (props: PropsWithChildren<Props>) => {
  ...

  const [selectedDevice, setSelectedDevice] = useState<SelectedDevice>(
    initialSelectedDevice,
  );

  const constraints: MediaStreamConstraints = useMemo(() => {
    const videoinputDeviceId = selectedDevice.videoinput?.deviceId;
    const audioinputDeviceId = selectedDevice.audioinput?.deviceId;
    return {
      video: {
        deviceId: videoinputDeviceId && {
          exact: videoinputDeviceId,
        },
      },
      audio: {
        deviceId: audioinputDeviceId && {
          exact: audioinputDeviceId,
        },
      },
    };
  }, [selectedDevice]);

  const userMediaStream = useUserMediaStream(constraints);
  ...
}
```

#### 생각해볼 점

1. 지금은 stream이 변경되면 그걸 교체하는 방식으로 구현했는데, 꼭 stream 자체를 변경할 필요가 있을까? 기존 track만 제거하고 새로운 track을 붙여주기만 해도 충분할 것 같음. stream이 변경되면 괜히 리렌더링만 더 일어남.

2. 근데 WebRTC랑 연동되면 track만 똑 떼서 연결할 수도 있는데 이 상황에선 track이 변경됐다는 사실을 알려줄 필요가 있음. 1) 처럼 구현하면 그게 불가능함. `RtcPeerConnection`에 stream 자체를 연결하는 방법을 찾아보자.

### audioouput 장치 변경하기

- 이 부분은 아직 구현 안함

- input 장치 변경이랑 아예 다르다.

- [WebRTC Sample](https://github.com/degurii/samples/blob/gh-pages/src/content/devices/input-output/js/main.js) 코드를 보면 `setSinkId` 메서드를 이용함.

- deviceId를 얻어와서, 특정 엘리먼트의 audio 출력을 그 기기로 이어주는 메서드인 것 같음.

- 근데 거의 **chrome only**임. ([Can I use setSinkId?](https://caniuse.com/?search=setsinkid))

- 뭐 다른 방법 없으면 샘플 코드 방식으로 해결하고.. 찾아보고 대체 방법이 있으면 그 방식으로 구현하도록 하자.
