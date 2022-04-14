# M1 Mac + VSCode로 SICP 실습 환경 구성하기

MIT Scheme을 추천하지만, 개발 환경이 너무 구식이라 불편하다고 한다. 대신 Racket을 이용해서 해결할 수 있다.

Racket은 lisp-scheme 계열의 언어로, SICP에서 사용된 Scheme과 같은 언어는 아니지만 Racket으로 지원되는 SICP 패키지를 이용해 비슷한 환경을 만들 수 있다고 한다.

구글링으로 짜집기한 방법을 정리해둔다.

### 1. racket 설치

Homebrew를 사용한다.

```
brew install --cask racket
```

> 이 부분에서 racket은 deprecated되었고, 대신 minimal-racket을 설치한다는 문구가 뜸. 다음에 설치할 땐 minimal-racket으로 명시해줘도 될듯.

### 2. VSCode Extension - Magic Racket 설치

Dr.Racket이라는 Racket 전용 IDE가 있으나 일단은 VSC로 코드를 작성하고 싶어 익스텐션을 활용한다. 공부하다 디버깅 측면에서 불리하다 싶으면 DrRacket으로 넘어가긴 할 건데, 일단 VSC로 해보자.

VSCode에서 Magic Racket 익스텐션을 설치한다.

> 주워 들은 말로는 Headless버전 DrRacket이라고 봐도 된다고 한다.

그 후 설명대로 racket-langserver를 설치한다.
이 친구는 https://github.com/jeapostrophe/racket-langserver 여기 적힌대로 편의성 측면에서 도와준다. 사람들 참 똑똑하다..

```
raco pkg install racket-langserver
```

### 3. SICP 패키지 설치

https://docs.racket-lang.org/sicp-manual/index.html

SICP 패키지 Docs이다. DrRacket 기준으로 설명 해놨는데 raco 패키지로 인스톨하면 Magic Racket에서도 적용될 거 같아서 일단 설치해봤다.

```
raco pkg install sicp
```

패키지 설치후 다음 코드로 테스트해보자.

```scheme
#lang sicp

(inc 42)
```

`43`이 출력되면 성공이다.

> 만약 모듈을 찾을 수 없다는 에러 메시지가 뜨면 VSCode를 재시작하면 된다.

> 파일 확장자는 racket 확장자인 `.rkt`를 사용하자.
