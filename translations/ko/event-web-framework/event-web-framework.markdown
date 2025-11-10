title: 이벤트 기반 웹 프레임워크
author: Leo Zovic
<markdown>
_Leo(온라인에서는 inaimathi로 더 잘 알려진)는 그래픽 디자이너 출신으로, Scheme, Common Lisp, Erlang, Javascript, Haskell, Clojure, Go, Python, PHP, C를 전문적으로 다뤄왔습니다. 현재 프로그래밍에 관해 블로그를 쓰고, 보드게임을 즐기며, 토론토 온타리오의 Ruby 기반 스타트업에서 일하고 있습니다._
</markdown>

2013년에 저는 카드 게임과 보드게임을 위한 [웹 기반 게임 프로토타이핑 도구](https://github.com/Inaimathi/deal)인 _House_를 작성하기로 결정했습니다. 이런 유형의 게임에서는 한 플레이어가 다른 플레이어의 움직임을 기다리는 것이 일반적입니다. 하지만 상대방이 마침내 행동을 취했을 때, 기다리고 있던 플레이어가 그 움직임을 빠르게 통보받기를 원합니다.

이것은 처음 보기보다 훨씬 복잡한 문제로 밝혀졌습니다. 이 장에서는 HTTP를 사용하여 이러한 종류의 상호작용을 구축할 때의 문제점들을 살펴보고, 미래에 유사한 문제들을 해결할 수 있도록 해주는 _웹 프레임워크_를 Common Lisp로 구축해보겠습니다.

## HTTP 서버의 기초

가장 단순한 수준에서, HTTP 교환은 하나의 요청과 그에 따른 하나의 응답으로 이루어집니다. _클라이언트_는 리소스 식별자, HTTP 버전 태그, 일부 헤더와 매개변수들을 포함한 요청을 보냅니다. _서버_는 그 요청을 파싱하고, 어떻게 처리할지 결정한 후, 동일한 HTTP 버전 태그, 응답 코드, 일부 헤더와 응답 본문을 포함한 응답을 보냅니다.

이 설명에서 주목할 점은, 서버가 특정 클라이언트로부터의 요청에 응답한다는 것입니다. 우리의 경우, 각 플레이어는 자신의 움직임이 만들어졌을 때만 알림을 받는 것이 아니라, _어떤_ 움직임이라도 일어나는 즉시 업데이트를 받기를 원합니다. 이는 서버가 정보에 대한 요청을 먼저 받지 않고도 클라이언트에게 메시지를 _푸시_해야 함을 의미합니다.[^polling]

[^polling]: 이 문제에 대한 한 가지 해결책은 클라이언트가 서버를 _폴링_하도록 강제하는 것입니다. 즉, 각 클라이언트가 주기적으로 서버에 변경사항이 있는지 묻는 요청을 보내는 것입니다. 이는 간단한 애플리케이션에서는 작동할 수 있지만, 이 장에서는 이 모델이 작동하지 않을 때 사용할 수 있는 해결책에 초점을 맞추겠습니다.

HTTP를 통한 서버 푸시를 가능하게 하는 몇 가지 표준 접근법이 있습니다.

### Comet/Long Poll

"long poll" 기법은 클라이언트가 응답을 받는 즉시 서버에 새로운 요청을 보내도록 합니다. 그 요청을 즉시 처리하는 대신, 서버는 후속 이벤트가 발생할 때까지 기다렸다가 응답합니다. 클라이언트가 여전히 모든 업데이트에 대해 행동을 취하므로, 이는 다소 의미론적인 구분입니다.

### Server-Sent Events (SSE)

서버 전송 이벤트는 클라이언트가 연결을 시작한 후 그 연결을 열어둘 것을 요구합니다. 서버는 연결을 닫지 않고 주기적으로 새로운 데이터를 연결에 씁니다. 클라이언트는 응답 연결이 종료되기를 기다리지 않고 도착하는 새로운 메시지들을 해석합니다. 각 메시지가 새로운 HTTP 헤더의 오버헤드를 감수할 필요가 없기 때문에 이는 Comet/long poll 접근법보다 약간 더 효율적입니다.

### WebSockets

WebSocket은 HTTP 위에 구축된 통신 프로토콜입니다. 서버와 클라이언트는 HTTP 대화를 시작한 후 핸드셰이크와 프로토콜 에스컬레이션을 수행합니다. 최종 결과는 여전히 TCP/IP를 통해 통신하지만, HTTP를 전혀 사용하지 않는다는 것입니다. 이것이 SSE에 비해 갖는 장점은 효율성을 위해 프로토콜을 사용자 정의할 수 있다는 것입니다.

### 장기간 지속되는 연결

이 세 가지 접근법은 서로 상당히 다르지만, 모두 중요한 특성을 공유합니다: 모두 장기간 지속되는 연결에 의존한다는 것입니다. Long polling은 새로운 데이터가 사용 가능해질 때까지 서버가 요청을 보관하는 것에 의존하고, SSE는 클라이언트와 서버 간에 데이터가 주기적으로 쓰여지는 열린 스트림을 유지하며, WebSocket은 특정 연결이 사용하는 프로토콜을 변경하지만 연결은 열어둡니다.

이것이 일반적인 HTTP 서버에 문제를 일으킬 수 있는 이유를 알아보기 위해, 기본적인 구현이 어떻게 작동하는지 살펴봅시다.

### 전통적인 HTTP 서버 아키텍처
\label{sec.eventsweb.serverarch}

하나의 HTTP 서버는 많은 요청을 동시에 처리합니다. 역사적으로, 많은 HTTP 서버는 _요청별 스레드_ 아키텍처를 사용해왔습니다. 즉, 들어오는 각 요청에 대해, 서버는 응답하는 데 필요한 작업을 수행하기 위한 스레드를 생성합니다.

이러한 연결들은 각각 단기간 지속되도록 의도되었으므로, 모든 연결을 처리하기 위해 병렬로 실행되는 많은 스레드가 필요하지 않습니다. 이 모델은 또한 서버 프로그래머가 주어진 시점에 하나의 연결만 처리되는 것처럼 코드를 작성할 수 있도록 함으로써 서버의 _구현_을 단순화합니다. 또한 해당 스레드를 종료하고 가비지 컬렉터가 그 역할을 하도록 함으로써 실패한 연결이나 "좀비" 연결과 관련된 리소스를 정리할 자유를 제공합니다.

핵심 관찰은 $N$명의 동시 사용자를 갖는 "전통적인" 웹 애플리케이션을 호스팅하는 HTTP 서버는 성공하기 위해 $N$ 요청의 아주 작은 부분만을 _병렬로_ 처리하면 될 수도 있다는 것입니다. 우리가 구축하려는 상호작용 애플리케이션의 경우, $N$명의 사용자는 거의 확실히 애플리케이션이 최소 $N$개의 연결을 동시에 병렬로 유지하도록 요구할 것입니다.

장기간 지속되는 연결을 유지하는 결과로, 우리는 다음 중 하나가 필요할 것입니다:

- 한 번에 많은 수를 사용할 수 있을 정도로 스레드가 "저렴한" 플랫폼
- 단일 스레드로 많은 연결을 처리할 수 있는 서버 아키텍처

첫 번째 옵션을 고려할 만큼 "경량"인 스레드 유사 구조를 제공하는 [Racket](http://racket-lang.org/), [Erlang](http://www.erlang.org/), [Haskell](http://hackage.haskell.org/package/base-4.7.0.1/docs/Control-Concurrent.html) 같은 프로그래밍 환경들이 있습니다. 이 접근법은 프로그래머가 동기화 문제를 명시적으로 처리하도록 요구하는데, 이는 연결이 오랜 시간 동안 열려 있고 아마도 모두 유사한 리소스를 놓고 경쟁하는 시스템에서 훨씬 더 널리 퍼져 있을 것입니다. 특히, 여러 사용자가 동시에 공유하는 어떤 종류의 중앙 데이터가 있다면, 그 데이터의 읽기와 쓰기를 어떤 방식으로든 조정해야 할 것입니다.

저렴한 스레드를 사용할 수 없거나 명시적인 동기화를 다루고 싶지 않다면, 단일 스레드가 많은 연결을 처리하도록 하는 것을 고려해야 합니다.[^mn] 이 모델에서, 우리의 단일 스레드는 많은 요청의 작은 "조각들"을 한 번에 처리하면서, 가능한 한 효율적으로 그들 사이를 전환할 것입니다. 이러한 시스템 아키텍처 패턴은 가장 일반적으로 _이벤트 기반_ 또는 _이벤트 주도_라고 불립니다.[^eventbased]

[^mn]: 구성 가능한 $M$ 값에 대해 $M$개의 스레드로 $N$명의 동시 사용자를 처리하는 더 일반적인 시스템을 고려할 수 있습니다. 이 모델에서 $N$개의 연결은 $M$개의 스레드에 걸쳐 _멀티플렉싱_된다고 합니다. 이 장에서는 $M$이 1로 고정된 프로그램 작성에 초점을 맞출 것입니다. 그러나 여기서 배운 교훈은 더 일반적인 모델에 부분적으로 적용될 수 있을 것입니다.

[^eventbased]: 이 명명법은 다소 혼란스럽며, 초기 운영체제 연구에서 기원을 가지고 있습니다. 이는 여러 동시 프로세스 간의 통신이 어떻게 이루어지는지를 의미합니다. 스레드 기반 시스템에서는 공유 메모리와 같은 동기화된 리소스를 통해 통신이 이루어집니다. 이벤트 기반 시스템에서는 프로세스들이 일반적으로 실행 단일 스레드에 의해 유지되는 큐를 통해 통신하며, 여기에 그들이 수행한 것 또는 수행하기를 원하는 것을 설명하는 항목들을 게시합니다. 이러한 항목들은 일반적으로 원하는 또는 과거의 행동을 설명하므로 '이벤트'라고 불립니다.

단일 스레드만을 관리하므로, 동시 접근으로부터 공유 리소스를 보호하는 것에 대해 그렇게 많이 걱정할 필요가 없습니다. 그러나 이 모델에서는 고유한 문제가 있습니다. 단일 스레드가 모든 진행 중인 요청을 한 번에 처리하고 있으므로, 그것이 __절대 블록되지 않도록__ 해야 합니다. 어떤 연결에서 블록되면 전체 서버가 다른 요청에서 진행하는 것을 막습니다. 현재 클라이언트를 더 이상 서비스할 수 없다면 다른 클라이언트로 넘어갈 수 있어야 하며, 지금까지 수행한 작업을 버리지 않는 방식으로 그렇게 할 수 있어야 합니다.[^crawler]

[^crawler]: 이 문제에 대한 다른 접근법은 \aosachapref{s:crawler}를 참조하세요.

프로그래머가 명시적으로 스레드에게 작업을 중단하라고 말하는 것은 드물지만, 많은 일반적인 작업들이 블로킹의 위험을 수반합니다. 스레드가 매우 널리 퍼져 있고, 비동기성에 대한 추론이 프로그래머에게 큰 부담이기 때문에, 많은 언어와 그들의 프레임워크는 I/O에서 블로킹하는 것이 바람직한 속성이라고 가정합니다. 이는 _우연히_ 어딘가에서 블록하기를 매우 쉽게 만듭니다. 다행히도, Common Lisp는 우리가 그 위에 구축할 수 있는 최소한의 비동기 I/O 프리미티브 세트를 제공합니다.

### 아키텍처 결정

이 문제의 배경을 연구한 후, 우리는 _무엇을_ 구축할지에 대해 정보에 기반한 결정을 내려야 하는 지점에 도달했습니다.

이 프로젝트에 대해 생각하기 시작했을 때, Common Lisp는 완전한 그린 스레드 구현을 갖지 않았고, [표준 포터블 스레딩 라이브러리](http://common-lisp.net/project/bordeaux-threads/)는 "정말 정말 저렴하다"고 인정할 수 없었습니다. 선택지는 다른 언어를 선택하거나, 내 목적을 위해 이벤트 기반 웹 서버를 구축하는 것으로 귀결되었습니다. 저는 후자를 선택했습니다.

서버 아키텍처에 더해서, 세 가지 서버 푸시 접근법 중 어느 것을 사용할지도 선택해야 합니다. 우리가 고려하고 있는 사용 사례(상호작용적인 멀티플레이어 보드 게임)는 각 클라이언트에 대한 빈번한 업데이트를 요구하지만, 각 클라이언트_로부터의_ 요청은 상대적으로 드문데, 이는 업데이트를 푸시하는 SSE 접근법에 적합하므로 이것을 사용하겠습니다.

이제 우리의 아키텍처 결정을 동기부여했고 클라이언트와 서버 간의 양방향 통신을 시뮬레이션하는 메커니즘을 결정했으므로, 웹 프레임워크 구축을 시작해봅시다. 먼저 상대적으로 "단순한" 서버를 구축한 후, 이를 우리의 상호작용이 많은 프로그램이 _어떻게_ 하는지가 아닌 _무엇을_ 해야 하는지에 초점을 맞출 수 있도록 하는 웹 애플리케이션 프레임워크로 확장할 것입니다.

## 이벤트 기반 웹 서버 구축

동시 작업 스트림을 관리하기 위해 단일 프로세스를 사용하는 대부분의 프로그램은 _이벤트 루프_라고 불리는 패턴을 사용합니다. 우리 웹 서버의 이벤트 루프가 어떤 모습일지 살펴봅시다.

### 이벤트 루프

우리의 이벤트 루프는 다음이 필요합니다:

- 들어오는 연결을 수신
- 모든 새로운 핸드셰이크 또는 기존 연결의 들어오는 데이터를 처리
- 예기치 않게 종료된(예: 인터럽트에 의해) 댕글링 소켓을 정리

```lisp
(defmethod start ((port integer))
  (let ((server (socket-listen
		 usocket:*wildcard-host* port
		 :reuse-address t
		 :element-type 'octet))
	(conns (make-hash-table)))
    (unwind-protect
	 (loop (loop for ready
		  in (wait-for-input
		      (cons server (alexandria:hash-table-keys conns))
		      :ready-only t)
		  do (process-ready ready conns)))
      (loop for c being the hash-keys of conns
	 do (loop while (socket-close c)))
      (loop while (socket-close server)))))
```

이전에 Common Lisp 프로그램을 작성해본 적이 없다면, 이 코드 블록은 약간의 설명이 필요합니다. 여기서 작성한 것은 _메서드 정의_입니다. Lisp가 함수형 언어로 널리 알려져 있지만, "The Common Lisp Object System"이라고 불리는 자체적인 객체 지향 프로그래밍 시스템도 가지고 있으며, 이는 보통 "CLOS"로 축약됩니다.[^CLOSpronounce]

[^CLOSpronounce]: 누구와 대화하느냐에 따라 "kloss", "see-loss" 또는 "see-lows"로 발음됩니다.

### CLOS와 제네릭 함수

CLOS에서는 클래스와 메서드에 초점을 맞추는 대신, _메서드_들의 집합으로 구현되는 [_제네릭 함수_](http://www.gigamonkeys.com/book/object-reorientation-generic-functions.html)를 작성합니다. 이 모델에서 메서드는 클래스에 _속하지_ 않고, 타입에 _특화됩니다_.[^juliachap] 방금 작성한 `start` 메서드는 인수 `port`가 `integer` 타입에 _특화된_ 단항 메서드입니다. 이는 `port`가 타입별로 다른 여러 `start` 구현을 가질 수 있고, 런타임이 `start`가 호출될 때 `port`의 타입에 따라 어떤 구현을 사용할지 선택한다는 것을 의미합니다.

[^juliachap]: Julia 프로그래밍 언어는 객체 지향 프로그래밍에 유사한 접근법을 취합니다. \aosachapref{s:static-analysis}에서 더 자세히 알아볼 수 있습니다.

더 일반적으로, 메서드는 하나 이상의 인수에 특화될 수 있습니다. `method`가 호출될 때, 런타임은:

- 인수의 타입에 디스패치하여 어떤 메서드 본문이 실행되어야 하는지 파악하고,
- 적절한 함수를 실행합니다.

### 소켓 처리

이벤트 루프에서 이전에 호출된 `process-ready`에서 작동하는 또 다른 제네릭 함수를 볼 수 있습니다. 이것은 우리가 처리하는 소켓의 타입에 따라 두 가지 메서드 중 하나로 준비된 소켓을 처리합니다.

우리가 관심을 갖는 두 가지 타입은 요청을 하고 데이터를 다시 받기를 기대하는 클라이언트 소켓을 나타내는 `stream-usocket`과, 우리가 처리할 새로운 클라이언트 연결을 가질 로컬 TCP 리스너를 나타내는 `stream-server-usocket`입니다.

`stream-server-socket`이 `ready`라면, 그것은 대화를 시작하기 위해 기다리고 있는 새로운 클라이언트 소켓이 있다는 것을 의미합니다. 우리는 `socket-accept`를 호출하여 연결을 수락하고, 그 다음 이벤트 루프가 다른 것들과 함께 처리를 시작할 수 있도록 그 결과를 연결 테이블에 넣습니다.

```lisp
(defmethod process-ready ((ready stream-server-usocket) (conns hash-table))
  (setf (gethash (socket-accept ready :element-type 'octet) conns) nil))
```

`stream-usocket`이 `ready`일 때는, 읽을 수 있는 바이트가 준비되어 있다는 것을 의미합니다. (상대방이 연결을 종료했을 가능성도 있습니다.)

```lisp
(defmethod process-ready ((ready stream-usocket) (conns hash-table))
  (let ((buf (or (gethash ready conns)
		 (setf (gethash ready conns)
		       (make-instance 'buffer :bi-stream (flex-stream ready))))))
    (if (eq :eof (buffer! buf))
	(ignore-errors
	  (remhash ready conns)
	  (socket-close ready))
	(let ((too-big?
	       (> (total-buffered buf)
		  +max-request-size+))
	      (too-old?
	       (> (- (get-universal-time) (started buf))
		  +max-request-age+))
	      (too-needy?
	       (> (tries buf)
		  +max-buffer-tries+)))
	  (cond (too-big?
		 (error! +413+ ready)
		 (remhash ready conns))
		((or too-old? too-needy?)
		 (error! +400+ ready)
		 (remhash ready conns))
		((and (request buf) (zerop (expecting buf)))
		 (remhash ready conns)
		 (when (contents buf)
		   (setf (parameters (request buf))
			 (nconc (parse buf) (parameters (request buf)))))
		 (handler-case
		     (handle-request ready (request buf))
		   (http-assertion-error () (error! +400+ ready))
		   ((and (not warning)
		     (not simple-error)) (e)
		     (error! +500+ ready e))))
		(t
		 (setf (contents buf) nil)))))))
```

이것은 첫 번째 경우보다 더 복잡합니다. 우리는:

1. 이 소켓과 연관된 버퍼를 가져오거나, 아직 존재하지 않으면 생성합니다;
2. 그 버퍼로 출력을 읽어들이는데, 이는 `buffer!` 호출에서 발생합니다;
3. 그 읽기가 `:eof`를 가져왔다면, 상대측이 연결을 끊었으므로 소켓 _과_ 그 버퍼를 폐기합니다;
4. 그렇지 않으면, 버퍼가 `complete?`, `too-big?`, `too-old?` 또는 `too-needy?` 중 하나인지 확인합니다. 그렇다면, 연결 테이블에서 제거하고 적절한 HTTP 응답을 반환합니다.

이는 이벤트 루프에서 I/O를 보는 첫 번째 시간입니다. \aosasecref{sec.eventsweb.serverarch}의 논의에서, 우리가 실수로 단일 스레드를 블록할 수 있기 때문에 이벤트 기반 시스템에서 I/O에 대해 매우 조심해야 한다고 언급했습니다. 그렇다면 이것이 발생하지 않도록 보장하기 위해 여기서 무엇을 할까요? 이것이 정확히 어떻게 작동하는지 알아내기 위해 `buffer!`의 구현을 탐색해야 합니다.

### 블로킹 없이 연결 처리하기

블로킹 없이 연결을 처리하는 우리 접근법의 기초는 사용 가능한 데이터가 없는 스트림에서 호출될 때 즉시 `nil`을 반환하는 라이브러리 함수 [`read-char-no-hang`](http://clhs.lisp.se/Body/f_rd_c_1.htm)입니다. 읽을 데이터가 있는 곳에서는, 이 연결에 대한 중간 입력을 저장하기 위해 버퍼를 사용합니다.

```lisp
(defmethod buffer! ((buffer buffer))
  (handler-case
      (let ((stream (bi-stream buffer)))
    	(incf (tries buffer))
    	(loop for char = (read-char-no-hang stream) until (null char)
    	   do (push char (contents buffer))
    	   do (incf (total-buffered buffer))
    	   when (request buffer) do (decf (expecting buffer))
    	   when (line-terminated? (contents buffer))
    	   do (multiple-value-bind (parsed expecting) (parse buffer)
    		(setf (request buffer) parsed
    		      (expecting buffer) expecting)
    		(return char))
    	   when (> (total-buffered buffer) +max-request-size+) return char
    	   finally (return char)))
    (error () :eof)))
```

`buffer`에서 `buffer!`가 호출될 때, 이것은:

- `tries` 카운트를 증가시켜 `process-ready`에서 "needy" 버퍼를 제거할 수 있도록 합니다;
- 입력 스트림에서 문자를 읽기 위해 루프를 돌고,
- 사용 가능한 모든 입력을 읽었다면 마지막으로 읽은 문자를 반환합니다.

또한 나중에 완전한 요청을 감지할 수 있도록 `\r\n\r\n` 시퀀스를 추적합니다. 마지막으로, 어떤 오류가 발생하면 `process-ready`가 이 연결을 폐기해야 한다는 신호로 `:eof`를 반환합니다.

`buffer` 타입은 CLOS [_클래스_](http://www.gigamonkeys.com/book/object-reorientation-classes.html)입니다. CLOS의 클래스는 `slots`라고 불리는 필드를 가진 타입을 정의할 수 있게 해줍니다. 클래스 정의에서 `buffer`와 연관된 행동들을 보지 못하는 이유는, (이미 배웠듯이) `buffer!`와 같은 제네릭 함수를 사용해서 그것을 하기 때문입니다.

`defclass`는 getters/setters(`reader`들/`accessor`들)와 슬롯 이니셜라이저를 지정할 수 있게 해줍니다; `:initform`은 기본값을 지정하고, `:initarg`는 \newline `make-instance`의 호출자가 기본값을 제공하기 위해 사용할 수 있는 훅을 식별합니다.

```lisp
(defclass buffer ()
  ((tries :accessor tries :initform 0)
   (contents :accessor contents :initform nil)
   (bi-stream :reader bi-stream :initarg :bi-stream)
   (total-buffered :accessor total-buffered :initform 0)
   (started :reader started :initform (get-universal-time))
   (request :accessor request :initform nil)
   (expecting :accessor expecting :initform 0)))
```

우리의 `buffer` 클래스는 일곱 개의 슬롯을 가집니다:

- `tries`: 이 버퍼로 읽기를 시도한 횟수를 세는 것
- `contents`: 지금까지 읽은 내용을 담고 있는 것
- `bi-stream`: 앞서 언급한 Common Lisp 특유의 비블로킹 I/O 문제들을 해결하기 위한 해킹
- `total-buffered`: 지금까지 읽은 문자의 개수
- `started`: 이 버퍼를 언제 생성했는지 알려주는 타임스탬프
- `request`: 결국 버퍼된 데이터로부터 구성한 요청을 포함하게 될 것
- `expecting`: 요청 헤더를 버퍼링한 후 (만약 있다면) 얼마나 더 많은 문자를 기대하고 있는지 신호를 보내는 것

### 요청 해석하기
\label{sec.eventsweb.handlerfunc}
우리의 버퍼에 모인 데이터 조각들로부터 완전한 요청을 점진적으로 조립하는 방법을 보았으니, 처리할 준비가 된 완전한 요청이 있을 때 무슨 일이 일어날까요? 이는 `handle-request` 메서드에서 발생합니다.

```lisp
(defmethod handle-request ((socket usocket) (req request))
  (aif (lookup (resource req) *handlers*)
       (funcall it socket (parameters req))
       (error! +404+ socket)))
```

이 메서드는 요청이 오래되었거나, 크거나, 요구가 많을 때 클라이언트가 나쁘거나 느린 데이터를 제공했다는 것을 나타내는 `400` 응답을 보낼 수 있도록 또 다른 오류 처리 계층을 추가합니다. 그러나 여기서 _다른_ 오류가 발생한다면, 그것은 프로그래머가 _핸들러_를 정의하는 데 실수를 했기 때문이므로 `500` 오류로 처리되어야 합니다. 이는 클라이언트의 정당한 요청의 결과로 서버에서 뭔가 잘못되었다는 것을 클라이언트에게 알려줍니다.

요청이 잘 형성되었다면, 우리는 `*handlers*` 테이블에서 요청된 리소스를 찾는 작고 명백한 작업을 수행합니다. 하나를 찾으면, 클라이언트 `socket`과 파싱된 요청 매개변수들을 함께 전달하여 `it`을 `funcall`합니다. `*handlers*` 테이블에 일치하는 핸들러가 없다면, 대신 `404` 오류를 보냅니다. 핸들러 시스템은 나중 섹션에서 논의할 본격적인 _웹 프레임워크_의 일부가 될 것입니다.

그러나 우리는 아직 요청이 우리 버퍼 중 하나에서 어떻게 파싱되고 해석되는지 보지 못했습니다. 다음에 그것을 살펴봅시다:

```lisp
(defmethod parse ((buf buffer))
  (let ((str (coerce (reverse (contents buf)) 'string)))
    (if (request buf)
	    (parse-params str)
	    (parse str))))
```

이 고수준 메서드는 일반 문자열과 작동하는 `parse`의 특화나, 버퍼 내용을 HTTP 매개변수로 해석하는 `parse-params`에 위임합니다. 이들은 우리가 이미 얼마나 많은 요청을 처리했는지에 따라 호출됩니다; 마지막 `parse`는 `buffer`에 이미 부분적인 `request`가 저장되어 있을 때 발생하며, 이 시점에서 우리는 요청 본문만을 파싱하려고 합니다.


```lisp
(defmethod parse ((str string))
  (let ((lines (split "\\r?\\n" str)))
    (destructuring-bind (req-type path http-version) (split " " (pop lines))
      (declare (ignore req-type))
      (assert-http (string= http-version "HTTP/1.1"))
      (let* ((path-pieces (split "\\?" path))
	     (resource (first path-pieces))
	     (parameters (second path-pieces))
	     (req (make-instance 'request :resource resource)))
	(loop
	   for header = (pop lines)
	   for (name value) = (split ": " header)
	   until (null name)
	   do (push (cons (->keyword name) value) (headers req)))
	(setf (parameters req) (parse-params parameters))
	req))))

(defmethod parse-params ((params null)) nil)

(defmethod parse-params ((params string))
  (loop for pair in (split "&" params)
     for (name val) = (split "=" pair)
     collect (cons (->keyword name) (or val ""))))
```

`string`에 특화된 `parse` 메서드에서는 내용을 사용 가능한 조각들로 변환합니다. 버퍼와 직접 작업하는 대신 문자열에서 이렇게 하는 이유는 인터프리터나 REPL과 같은 환경에서 실제 파싱 코드를 테스트하기 더 쉽게 만들기 때문입니다.

파싱 과정은 다음과 같습니다:

1. `"\\r?\\n"`으로 분할합니다.
2. 그 첫 번째 줄을 `" "`으로 분할하여 요청 타입(`POST`, `GET` 등)/URI 경로/HTTP 버전을 가져옵니다.
3. `HTTP/1.1` 요청을 다루고 있는지 어설트합니다.
4. URI 경로를 `"?"`으로 분할하여 `GET` 매개변수들과 분리된 순수 리소스를 얻습니다.
5. 리소스가 제자리에 있는 새로운 `request` 인스턴스를 만듭니다.
6. 분할된 각 헤더 줄로 그 `request` 인스턴스를 채웁니다.
7. 그 `request`의 매개변수들을 우리의 `GET` 매개변수들을 파싱한 결과로 설정합니다.

이제 예상하겠지만, `request`는 CLOS 클래스의 인스턴스입니다:

```lisp
	(defclass request ()
	  ((resource :accessor resource :initarg :resource)
	   (headers :accessor headers :initarg :headers :initform nil)
	   (parameters :accessor parameters :initarg :parameters :initform nil)))
```

이제 클라이언트들이 요청을 보내고 서버가 이를 해석하고 처리하는 방법을 보았습니다. 핵심 서버 인터페이스의 일부로 구현해야 할 마지막 것은 클라이언트에게 응답을 다시 쓸 수 있는 능력입니다.

### 응답 렌더링하기

응답 렌더링을 논의하기 전에, 클라이언트들에게 반환할 수 있는 두 가지 종류의 응답이 있다는 것을 고려해야 합니다. 첫 번째는 HTTP 헤더와 본문을 모두 갖춘 "일반적인" HTTP 응답입니다. 우리는 이런 종류의 응답들을 `response` 클래스의 인스턴스로 나타냅니다:

```lisp
(defclass response ()
  ((content-type
    :accessor content-type :initform "text/html" :initarg :content-type)
   (charset
    :accessor charset :initform "utf-8")
   (response-code
    :accessor response-code :initform "200 OK" :initarg :response-code)
   (keep-alive?
    :accessor keep-alive? :initform nil :initarg :keep-alive?)
   (body
    :accessor body :initform nil :initarg :body)))
```

두 번째는 클라이언트들에게 점진적 업데이트를 보내는 데 사용할 [SSE 메시지](http://www.w3.org/TR/eventsource/)입니다.

```lisp
(defclass sse ()
  ((id :reader id :initarg :id :initform nil)
   (event :reader event :initarg :event :initform nil)
   (retry :reader retry :initarg :retry :initform nil)
   (data :reader data :initarg :data)))
```

완전한 HTTP 요청을 받을 때마다 HTTP 응답을 보낼 것입니다; 그러나 원래 클라이언트 요청 없이 언제 어디에 SSE 메시지를 보낼지 어떻게 알 수 있을까요?

간단한 해결책은 필요에 따라 `socket`들을 구독할 _채널_들[^defparameter]을 등록하는 것입니다.

```lisp
(defparameter *channels* (make-hash-table))

(defmethod subscribe! ((channel symbol) (sock usocket))
  (push sock (gethash channel *channels*))
  nil)
```

[^defparameter]: 여기서 우연히 새로운 문법을 소개합니다. 이것은 변경 가능한 변수를 선언하는 우리의 방법입니다. `(defparameter <name> <value> <optional docstring>)` 형태를 가집니다.

그러면 사용 가능해지는 즉시 해당 채널들에 알림을 `publish!`할 수 있습니다.

```lisp
(defmethod publish! ((channel symbol) (message string))
  (awhen (gethash channel *channels*)
	 (setf (gethash channel *channels*)
	       (loop with msg = (make-instance 'sse :data message)
		  for sock in it
		  when (ignore-errors
			 (write! msg sock)
			 (force-output (socket-stream sock))
			 sock)
		  collect it))))
```

`publish!`에서는 실제로 `sse`를 소켓에 쓰기 위해 `write!`를 호출합니다. 완전한 HTTP 응답도 쓸 수 있도록 `response`들에 대한 `write!`의 특화도 필요할 것입니다. HTTP 경우를 먼저 처리해봅시다.

```lisp
(defmethod write! ((res response) (socket usocket))
  (handler-case
      (with-timeout (.2)
	(let ((stream (flex-stream socket)))
	  (flet ((write-ln (&rest sequences)
		   (mapc (lambda (seq) (write-sequence seq stream)) sequences)
		   (crlf stream)))
	    (write-ln "HTTP/1.1 " (response-code res))
	    (write-ln
	     "Content-Type: " (content-type res) "; charset=" (charset res))
	    (write-ln "Cache-Control: no-cache, no-store, must-revalidate")
	    (when (keep-alive? res)
	      (write-ln "Connection: keep-alive")
	      (write-ln "Expires: Thu, 01 Jan 1970 00:00:01 GMT"))
	    (awhen (body res)
	      (write-ln "Content-Length: " (write-to-string (length it)))
	      (crlf stream)
	      (write-ln it))
	    (values))))
    (trivial-timeout:timeout-error ()
      (values))))
```

이 버전의 `write!`는 `response`와 `sock`이라는 `usocket`을 받아서 `sock`이 제공하는 스트림에 내용을 씁니다. 우리는 몇 개의 시퀀스를 받아서 그것들을 스트림에 쓰고 `crlf`를 따라 붙이는 `write-ln` 함수를 로컬로 정의합니다. 이는 가독성을 위한 것이며, 대신 `write-sequence`/`crlf`를 직접 호출할 수도 있습니다.

"블록하면 안 된다"는 것을 다시 하고 있다는 점에 주목하세요. 쓰기는 버퍼링될 가능성이 있고 읽기보다 블로킹 위험이 낮지만, 여기서 뭔가 잘못되면 서버가 멈추는 것을 원하지 않습니다. 쓰기가 0.2초[^timeout]보다 오래 걸리면, 더 이상 기다리지 않고 그냥 (현재 소켓을 버리고) 넘어갑니다.

[^timeout]: `with-timeout`은 다른 Lisp에서 다른 구현을 가집니다. 일부 환경에서는 호출한 것을 모니터하기 위해 다른 스레드나 프로세스를 만들 수 있습니다. 한 번에 최대 하나만 생성하지만, 쓰기마다 수행하기에는 상대적으로 무거운 작업입니다. 그런 환경에서는 대안적인 접근을 고려하고 싶을 것입니다.

`SSE`를 쓰는 것은 개념적으로 `response`를 쓰는 것과 유사합니다:

```lisp
(defmethod write! ((res sse) (socket usocket))
  (let ((stream (flex-stream socket)))
    (handler-case
    (with-timeout (.2)
      (format
       stream "~@[id: ~a~%~]~@[event: ~a~%~]~@[retry: ~a~%~]data: ~a~%~%"
       (id res) (event res) (retry res) (data res)))
      (trivial-timeout:timeout-error ()
        (values)))))
```

SSE 메시지 표준이 `CRLF` 줄 끝을 지정하지 않으므로 완전한 HTTP 응답을 다루는 것보다 단순하며, 단일 `format` 호출로 해결할 수 있습니다. `~@[`...`~]` 블록은 _조건부 지시어_로, `nil` 슬롯을 우아하게 처리할 수 있게 해줍니다. 예를 들어, `(id res)`가 nil이 아니라면 `id: <여기에 id> `를 출력하고, 그렇지 않으면 지시어를 완전히 무시합니다. 점진적 업데이트의 페이로드인 `data`는 `sse`의 유일한 필수 슬롯이므로, `nil`인지 걱정하지 않고 포함시킬 수 있습니다. 그리고 다시, 우리는 _너무_ 오래 기다리지 않습니다. 0.2초 후, 그때까지 쓰기가 완료되지 않았다면 타임아웃하고 다음 것으로 넘어갑니다.

### 오류 응답

지금까지의 요청/응답 사이클에 대한 우리의 처리는 뭔가 잘못되었을 때 무슨 일이 일어나는지 다루지 않았습니다. 구체적으로, 우리는 `handle-request`와 `process-ready`에서 `error!` 함수를 사용했지만 그것이 무엇을 하는지 설명하지 않았습니다.

```lisp
(define-condition http-assertion-error (error)
  ((assertion :initarg :assertion :initform nil :reader assertion))
  (:report (lambda (condition stream)
	     (format stream "Failed assertions '~s'"
		     (assertion condition)))))
```

`define-condition`은 Common Lisp에서 새로운 오류 클래스를 생성합니다. 이 경우, 우리는 HTTP 어설션 오류를 정의하고 있으며, 그것이 작용하고 있는 실제 어설션을 구체적으로 알고, 스트림에 자신을 출력할 방법이 필요하다고 명시하고 있습니다. 다른 언어에서는 이를 메서드라고 부를 것입니다. 여기서는 클래스의 슬롯 값인 함수입니다.

클라이언트에게 오류를 어떻게 나타낼까요? 자주 사용할 `4xx`와 `5xx` 클래스 HTTP 오류들을 정의해봅시다:

```lisp
(defparameter +404+
  (make-instance
   'response :response-code "404 Not Found"
   :content-type "text/plain"
   :body "Resource not found..."))

(defparameter +400+
  (make-instance
   'response :response-code "400 Bad Request"
   :content-type "text/plain"
   :body "Malformed, or slow HTTP request..."))

(defparameter +413+
  (make-instance
   'response :response-code "413 Request Entity Too Large"
   :content-type "text/plain"
   :body "Your request is too long..."))

(defparameter +500+
  (make-instance
   'response :response-code "500 Internal Server Error"
   :content-type "text/plain"
   :body "Something went wrong on our end..."))
```

Now we can see what `error!` does:

```lisp
(defmethod error! ((err response) (sock usocket) &optional instance)
  (declare (ignorable instance))
  (ignore-errors
    (write! err sock)
    (socket-close sock)))
```

오류 응답과 소켓을 받아서, 응답을 소켓에 쓰고 닫습니다 (상대편이 이미 연결을 끊었을 경우를 대비해 오류를 무시합니다). 여기서 `instance` 인수는 로깅/디버깅 목적을 위한 것입니다.

이것으로, HTTP 요청에 응답하거나 SSE 메시지를 보낼 수 있는 완전한 오류 처리를 갖춘 이벤트 기반 웹 서버를 갖게 되었습니다!


## 서버를 웹 프레임워크로 확장하기

이제 클라이언트와 요청, 응답, 메시지를 주고받을 수 있는 합리적으로 기능적인 웹 서버를 구축했습니다. 이 서버에서 호스팅되는 웹 애플리케이션의 실제 작업은 \aosasecref{sec.eventsweb.handlerfunc}에서 소개되었지만 명세가 부족했던 핸들러 함수들에 위임함으로써 수행됩니다.

우리 서버와 호스팅되는 애플리케이션 간의 인터페이스는 중요한데, 이는 애플리케이션 프로그래머들이 우리 인프라와 얼마나 쉽게 작업할 수 있는지를 좌우하기 때문입니다. 이상적으로, 우리의 핸들러 인터페이스는 요청의 매개변수들을 실제 작업을 수행하는 함수에 매핑할 것입니다:

```lisp
(define-handler (source :is-stream? nil) (room)
  (subscribe! (intern room :keyword) sock))

(define-handler (send-message) (room name message)
  (publish! (intern room :keyword)
	    (encode-json-to-string
	     `((:name . ,name) (:message . ,message)))))

(define-handler (index) ()
  (with-html-output-to-string (s nil :prologue t :indent t)
    (:html
     (:head (:script
	     :type "text/javascript"
	     :src "/static/js/interface.js"))
     (:body (:div :id "messages")
	    (:textarea :id "input")
	    (:button :id "send" "Send")))))
```

House를 작성할 때 염두에 두었던 우려 중 하나는, 더 넓은 인터넷에 열린 어떤 애플리케이션과 마찬가지로 신뢰할 수 없는 클라이언트들로부터의 요청을 처리하게 될 것이라는 점이었습니다. 데이터를 설명하는 작은 _스키마_를 제공함으로써 각 요청이 구체적으로 어떤 _타입_의 데이터를 포함해야 하는지 말할 수 있다면 좋을 것입니다. 그러면 이전의 핸들러 목록은 다음과 같이 보일 것입니다:

```lisp
(defun len-between (min thing max)
  (>= max (length thing) min))

(define-handler (source :is-stream? nil)
    ((room :string (len-between 0 room 16)))
  (subscribe! (intern room :keyword) sock))

(define-handler (send-message)
    ((room :string (len-between 0 room 16))
     (name :string (len-between 1 name 64))
     (message :string (len-between 5 message 256)))
  (publish! (intern room :keyword)
	    (encode-json-to-string
	     `((:name . ,name) (:message . ,message)))))

(define-handler (index) ()
  (with-html-output-to-string (s nil :prologue t :indent t)
    (:html
     (:head (:script
	     :type "text/javascript"
	     :src "/static/js/interface.js"))
     (:body (:div :id "messages")
	    (:textarea :id "input")
	    (:button :id "send" "Send")))))
```

여전히 Lisp 코드로 작업하고 있지만, 이 인터페이스는 핸들러가 검증하기를 원하는 _것_을 그것들이 _어떻게_ 할지에 대해 너무 생각하지 않고 명시하는 _선언적 언어_처럼 보이기 시작합니다. 우리가 하고 있는 것은 핸들러 함수들을 위한 _도메인 특화 언어_(DSL)를 구축하는 것입니다; 즉, 우리의 핸들러가 검증하기를 원하는 것을 정확히 간결하게 표현할 수 있도록 하는 특정한 관례와 문법을 만들고 있습니다. 당면한 문제를 해결하기 위해 작은 언어를 구축하는 이 접근법은 Lisp 프로그래머들에 의해 자주 사용되며, 다른 프로그래밍 언어들에서도 적용될 수 있는 유용한 기법입니다.

### 핸들러를 위한 DSL

이제 핸들러 DSL이 어떻게 보이기를 원하는지에 대한 느슨한 명세를 갖게 되었는데, 이를 어떻게 구현할까요? 즉, `define-handler`를 호출할 때 구체적으로 무슨 일이 일어나기를 기대할까요? 위에서 나온 `send-message`의 정의를 고려해봅시다:

```lisp
(define-handler (send-message)
    ((room :string (len-between 0 room 16))
     (name :string (len-between 1 name 64))
     (message :string (len-between 5 message 256)))
  (publish! (intern room :keyword)
	    (encode-json-to-string
	     `((:name . ,name) (:message . ,message)))))
```

여기서 `define-handler`가 하기를 원하는 것은:

1. 핸들러 테이블에서 URI `/send-message`에 액션 `(publish! ...)`을 바인드합니다.
2. 이 URI에 요청이 만들어질 때:
    - HTTP 매개변수 `room`, `name`, `message`가 포함되었는지 확인합니다.
    - `room`이 16자 이하의 문자열이고, `name`이 1자에서 64자 사이의 문자열이며 (포함), `message`가 5자에서 256자 사이의 문자열인지 (역시 포함) 검증합니다.
3. 응답이 반환된 후, 채널을 닫습니다.

이 모든 일을 하는 Lisp 함수들을 작성하고 수동으로 조각들을 조립할 수도 있지만, 더 일반적인 접근법은 `매크로`라고 불리는 Lisp 기능을 사용하여 우리를 위해 Lisp 코드를 _생성_하는 것입니다. 이는 DSL이 무엇을 하기를 원하는지를 간결하게 표현할 수 있게 해주며, 그것을 하기 위한 많은 코드를 유지할 필요가 없습니다. 매크로를 런타임에 Lisp 코드로 확장될 "실행 가능한 템플릿"으로 생각할 수 있습니다.

다음은 우리의 `define-handler` 매크로입니다[^indentation]:

[^indentation]: 아래 코드 블록은 Common Lisp에서 매우 비관례적인 들여쓰기라는 점을 주목해야 합니다. 인수 목록들은 일반적으로 여러 줄로 나뉘지 않으며, 보통 매크로/함수 이름과 같은 줄에 유지됩니다. 이 책의 줄 너비 지침을 준수하기 위해 그렇게 해야 했지만, 그렇지 않았다면 코드 내용에 의해 결정되는 자연스러운 곳에서 줄바꿈하는 더 긴 줄을 선호했을 것입니다.

```lisp
(defmacro define-handler
    ((name &key (is-stream? t) (content-type "text/html")) (&rest args)
     &body body)
  (if is-stream?
      `(bind-handler
	,name (make-closing-handler
	       (:content-type ,content-type)
	       ,args ,@body))
      `(bind-handler
	,name (make-stream-handler ,args ,@body))))
```

It delegates to three other macros (`bind-handler`, `make-closing-handler`, \newline `make-stream-handler`) that we will define later. `make-closing-handler` will create a handler for a full HTTP request/response cycle; `make-stream-handler` will instead handle an SSE message. The predicate `is-stream?` distinguishes between these cases for us. The backtick and comma are macro-specific operators that we can use to "cut holes" in our code that will be filled out by values specified in our Lisp code when we actually use `define-handler`.

Notice how closely our macro conforms to our specification of what we wanted `define-handler` to do: If we were to write a series of Lisp functions to do all of these things, the intent of the code would be much more difficult to discern by inspection.

### Expanding a Handler

Let's step through the expansion for the `send-message` handler so that we better understand what is actually going on when Lisp "expands" our macro for us. We'll use the macro expansion feature from the [SLIME](https://common-lisp.net/project/slime/) Emacs mode to do this. Calling `macro-expander` on `define-handler` will expand our macro by one "level", leaving our helper macros in their still-condensed form:

```lisp
(BIND-HANDLER
 SEND-MESSAGE
 (MAKE-CLOSING-HANDLER
  (:CONTENT-TYPE "text/html")
  ((ROOM :STRING (LEN-BETWEEN 0 ROOM 16))
   (NAME :STRING (LEN-BETWEEN 1 NAME 64))
   (MESSAGE :STRING (LEN-BETWEEN 5 MESSAGE 256)))
  (PUBLISH! (INTERN ROOM :KEYWORD)
	    (ENCODE-JSON-TO-STRING
	     `((:NAME ,@NAME) (:MESSAGE ,@MESSAGE))))))
```

Our macro has already saved us a bit of typing by substituting our `send-message` specific code into our handler template. `bind-handler` is another macro which maps a URI to a handler function on our handlers table; since it's now at the root of our expansion, let's see how it is defined before expanding this further.

```lisp
(defmacro bind-handler (name handler)
  (assert (symbolp name) nil "`name` must be a symbol")
  (let ((uri (if (eq name 'root) "/" (format nil "/~(~a~)" name))))
    `(progn
       (when (gethash ,uri *handlers*)
	 (warn ,(format nil "Redefining handler '~a'" uri)))
       (setf (gethash ,uri *handlers*) ,handler))))
```

The binding happens in the last line: `(setf (gethash ,uri *handlers*) ,handler)`, which is what hash-table assignments look like in Common Lisp (modulo the commas, which are part of our macro). Note that the `assert` is outside of the quoted area, which means that it'll be run as soon as the macro is _called_ rather than when its result is evaluated.

When we further expand our expansion of the `send-message` `define-handler` above, we get:

```lisp
(PROGN
  (WHEN (GETHASH "/send-message" *HANDLERS*)
    (WARN "Redefining handler '/send-message'"))
  (SETF (GETHASH "/send-message" *HANDLERS*)
	(MAKE-CLOSING-HANDLER
	 (:CONTENT-TYPE "text/html")
	 ((ROOM :STRING (LEN-BETWEEN 0 ROOM 16))
	  (NAME :STRING (LEN-BETWEEN 1 NAME 64))
	  (MESSAGE :STRING (LEN-BETWEEN 5 MESSAGE 256)))
	 (PUBLISH! (INTERN ROOM :KEYWORD)
		   (ENCODE-JSON-TO-STRING
		    `((:NAME ,@NAME) (:MESSAGE ,@MESSAGE)))))))
```

This is starting to look more like a custom implementation of what we would have written to marshal a request from a URI to a handler function, had we written it all ourselves. But we didn't have to!

We still have `make-closing-handler` left to go in our expansion. Here is its definition:

```lisp
(defmacro make-closing-handler
    ((&key (content-type "text/html")) (&rest args) &body body)
  `(lambda (sock parameters)
     (declare (ignorable parameters))
     ,(arguments
       args
       `(let ((res (make-instance
		    'response
		    :content-type ,content-type
		    :body (progn ,@body))))
	  (write! res sock)
	  (socket-close sock)))))
```

So making a closing-handler involves making a `lambda`, which is just what you call anonymous functions in Common Lisp. We also set up an interior scope that makes a `response` out of the `body` argument we're passing in, performs a `write!` to the requesting socket, then closes it. The remaining question is, what is `arguments`?

```lisp
(defun arguments (args body)
  (loop with res = body
     for arg in args
     do (match arg
	 ((guard arg-sym (symbolp arg-sym))
	  (setf res `(let ((,arg-sym ,(arg-exp arg-sym))) ,res)))
	 ((list* arg-sym type restrictions)
	  (setf res
		(let ((sym (or (type-expression
				(arg-exp arg-sym)
				type restrictions)
			       (arg-exp arg-sym))))
		  `(let ((,arg-sym ,sym))
		     ,@(awhen (type-assertion arg-sym type restrictions)
			 `((assert-http ,it)))
		     ,res)))))
     finally (return res)))
```

Welcome to the hard part. `arguments` turns the validators we registered with our handler into a tree of parse attempts and assertions. `type-expression`, `arg-exp`, and `type-assertion` are used to implement and enforce a "type system" for the kinds of data we're expecting in our responses; we'll discuss them in \aosasecref{sec.eventsweb.types}. Using this together with `make-closing-handler` would implement the validation rules we wrote here:

```lisp
(define-handler (send-message)
    ((room :string (>= 16 (length room)))
     (name :string (>= 64 (length name) 1))
     (message :string (>= 256 (length message) 5)))
  (publish! (intern room :keyword)
	    (encode-json-to-string
	     `((:name . ,name) (:message . ,message)))))
```

...as an "unrolled" sequence of checks needed to validate the request:

```lisp
(LAMBDA (SOCK #:COOKIE?1111 SESSION PARAMETERS)
  (DECLARE (IGNORABLE SESSION PARAMETERS))
  (LET ((ROOM (AIF (CDR (ASSOC :ROOM PARAMETERS))
		   (URI-DECODE IT)
		   (ERROR (MAKE-INSTANCE
			   'HTTP-ASSERTION-ERROR
			   :ASSERTION 'ROOM)))))
    (ASSERT-HTTP (>= 16 (LENGTH ROOM)))
    (LET ((NAME (AIF (CDR (ASSOC :NAME PARAMETERS))
		     (URI-DECODE IT)
		     (ERROR (MAKE-INSTANCE
			     'HTTP-ASSERTION-ERROR
			     :ASSERTION 'NAME)))))
      (ASSERT-HTTP (>= 64 (LENGTH NAME) 1))
      (LET ((MESSAGE (AIF (CDR (ASSOC :MESSAGE PARAMETERS))
			  (URI-DECODE IT)
			  (ERROR (MAKE-INSTANCE
				  'HTTP-ASSERTION-ERROR
				  :ASSERTION 'MESSAGE)))))
	(ASSERT-HTTP (>= 256 (LENGTH MESSAGE) 5))
	(LET ((RES (MAKE-INSTANCE
		    'RESPONSE :CONTENT-TYPE "text/html"
		    :COOKIE (UNLESS #:COOKIE?1111
			      (TOKEN SESSION))
		    :BODY (PROGN
			    (PUBLISH!
			     (INTERN ROOM :KEYWORD)
			     (ENCODE-JSON-TO-STRING
			      `((:NAME ,@NAME)
				(:MESSAGE ,@MESSAGE))))))))
	  (WRITE! RES SOCK)
	  (SOCKET-CLOSE SOCK))))))
```

This gets us the validation we need for full HTTP request/response cycles. What about our SSEs? `make-stream-handler` does the same basic thing as `make-closing-handler`, except that it writes an `SSE` rather than a `RESPONSE`, and it calls `force-output` instead of `socket-close` because we want to flush data over the connection without closing it:

```lisp
(defmacro make-stream-handler ((&rest args) &body body)
  `(lambda (sock parameters)
     (declare (ignorable parameters))
     ,(arguments
       args
       `(let ((res (progn ,@body)))
	  (write! (make-instance
		   'response
		   :keep-alive? t
		   :content-type "text/event-stream")
		  sock)
	  (write!
	   (make-instance 'sse :data (or res "Listening..."))
	   sock)
	  (force-output
	   (socket-stream sock))))))

(defmacro assert-http (assertion)
  `(unless ,assertion
     (error (make-instance
	     'http-assertion-error
	     :assertion ',assertion))))
```

`assert-http` is a macro that creates the boilerplate code we need in error cases. It expands into a check of the given assertion, throws an `http-assertion-error` if it fails, and packs the original assertion along in that event.

```lisp
(defmacro assert-http (assertion)
  `(unless ,assertion
     (error (make-instance
	     'http-assertion-error
	     :assertion ',assertion))))
```

### HTTP "Types"
\label{sec.eventsweb.types}

In the previous section, we briefly touched on three expressions that we're using to implement our HTTP type validation system: `arg-exp`, `type-expression` and `type-assertion`. Once you understand those, there will be no magic left in our framework. We'll start with the easy one first.

#### arg-exp

`arg-exp` takes a symbol and creates an `aif` expression that checks for the presence of a parameter.

```lisp
(defun arg-exp (arg-sym)
  `(aif (cdr (assoc ,(->keyword arg-sym) parameters))
	(uri-decode it)
	(error (make-instance
		'http-assertion-error
		:assertion ',arg-sym))))
```

Evaluating `arg-exp` on a symbol looks like:

```lisp
HOUSE> (arg-exp 'room)
(AIF (CDR (ASSOC :ROOM PARAMETERS))
     (URI-DECODE IT)
     (ERROR (MAKE-INSTANCE
	     'HTTP-ASSERTION-ERROR
	     :ASSERTION 'ROOM)))
HOUSE>
```

We've been using forms like `aif` and `awhen` without understanding how they work, so let's take some time to explore them now.

Recall that Lisp code is itself represented as a tree. That's what the parentheses are for; they show us how leaves and branches fit together. If we step back to what we were doing in the previous section, `make-closing-handler` calls a function called `arguments` to generate part of the Lisp tree it's constructing, which in turn calls some tree-manipulating helper functions, including `arg-exp`, to generate its return value.

That is, we've built a small system that takes a Lisp expression as input, and produces a different Lisp expression as output. Possibly the simplest way of conceptualizing this is as a simple Common–Lisp-to-Common–Lisp compiler that is specialized to the problem at hand.

A widely used classification of such compilers is as _anaphoric macros_. This term comes from the linguistic concept of an _anaphor_, which is the use of one word as a substitute for a group of words that preceded it. `aif` and `awhen` are anaphoric macros, and they're the only ones that I tend to often use. There are many more availabile in the [`anaphora` package](http://www.cliki.net/Anaphora).

As far as I know, anaphoric macros were first defined by Paul Graham in an [OnLisp chapter](http://dunsmor.com/lisp/onlisp/onlisp_18.html). The use case he gives is a situation where you want to do some sort of expensive or semi-expensive check, then do something conditionally on the result. In the above context, we're using `aif` to do a check the result of an `alist` traversal.

```lisp
(aif (cdr (assoc :room parameters))
     (uri-decode it)
     (error (make-instance
	     'http-assertion-error
	     :assertion 'room)))
```

This takes the `cdr` of looking up the symbol `:room` in the association list `parameters`. If that returns a non-nil value, `uri-decode` it, otherwise throw an error of the type `http-assertion-error`.

In other words, the above is equivalent to:

```lisp
(let ((it (cdr (assoc :room parameters))))
  (if it
      (uri-decode it)
      (error (make-instance
	      'http-assertion-error
	      :assertion 'room))))
```

Strongly-typed functional languages like Haskell often use a `Maybe` type in this situation. In Common Lisp, we capture the symbol `it` in the expansion as the name for the result of the check.

Understanding this, we should be able to see that `arg-exp` is generating a specific, repetitive, piece of the code tree that we eventually want to evaluate. In this case, the piece that checks for the presence of the given parameter among the handlers' `parameters`. Now, let's move onto...

#### type-expression

```lisp
(defgeneric type-expression (parameter type)
  (:documentation
   "A type-expression will tell the server
how to convert a parameter from a string to
a particular, necessary type."))
...
(defmethod type-expression (parameter type) nil)
```

This is a generic function that generates new tree structures (coincidentally Lisp code), rather than just a function. The only thing the above tells you is that by default, a `type-expression` is `NIL`. Which is to say, we don't have one. If we encounter a `NIL`, we use the raw output of `arg-exp`, but that doesn't tell us much about the most common case. To see that, let's take a look at a built-in (to `:house`) `define-http-type` expression.

```lisp
(define-http-type (:integer)
    :type-expression `(parse-integer ,parameter :junk-allowed t)
    :type-assertion `(numberp ,parameter))
```

An `:integer` is something we're making from a `parameter` by using `parse-integer`. The `junk-allowed` parameter tells `parse-integer` that we're not confident the data we're giving it is actually parseable, so we need to make sure that the returned result is an integer. If it isn't, we get this behaviour:

```
HOUSE> (type-expression 'blah :integer)
(PARSE-INTEGER BLAH :JUNK-ALLOWED T)
HOUSE>
```

`define-http-handler`[^readable] is one of the exported symbols for our framework. This lets our application programmers define their own types to simplify parsing above the handful of "builtins" that we give them (`:string`, `:integer`, `:keyword`, `:json`, `:list-of-keyword` and `:list-of-integer`).

```lisp
(defmacro define-http-type ((type) &key type-expression type-assertion)
  (with-gensyms (tp)
    `(let ((,tp ,type))
       ,@(when type-expression
	  `((defmethod type-expression (parameter (type (eql ,tp)))
	      ,type-expression)))
       ,@(when type-assertion
	  `((defmethod type-assertion (parameter (type (eql ,tp)))
	      ,type-assertion))))))
```

[^readable]: This macro is difficult to read because it tries hard to make its output human-readable, by expanding `NIL`s away using `,@` where possible.

It works by creating `type-expression` and `type-assertion` method definitions for the type being defined. We could let users of our framework do this manually without much trouble; however, adding this extra level of indirection gives us, the framework programmers, the freedom to change _how_ types are implemented without forcing our users to re-write their specifications. This isn't just an academic consideration; I've personally made radical changes to this part of the system when first building it, and was pleased to find that I had to make very few edits to the applications that depended on it.

Let's take a look at the expansion of that integer definition to see how it works in detail:

```lisp
(LET ((#:TP1288 :INTEGER))
  (DEFMETHOD TYPE-EXPRESSION (PARAMETER (TYPE (EQL #:TP1288)))
    `(PARSE-INTEGER ,PARAMETER :JUNK-ALLOWED T))
  (DEFMETHOD TYPE-ASSERTION (PARAMETER (TYPE (EQL #:TP1288)))
    `(NUMBERP ,PARAMETER)))
```

As we said, it doesn't reduce code size by much, but it does prevent us from needing to care what the specific parameters of those methods are, or even that they're methods at all.

#### type-assertion

Now that we can define types, let's look at how we use `type-assertion` to validate that a parse satisfies our requirements. It, too, takes the form of a complementary `defgeneric`/`defmethod` pair just like `type-expression`:

```lisp
(defgeneric type-assertion (parameter type)
  (:documentation
   "A lookup assertion is run on a parameter
immediately after conversion. Use it to restrict
 the space of a particular parameter."))
...
(defmethod type-assertion (parameter type) nil)
```

Here's what this one outputs:

```lisp
HOUSE> (type-assertion 'blah :integer)
(NUMBERP BLAH)
HOUSE>
```

There are cases where `type-assertion` won't need to do anything. For example, since HTTP parameters are given to us as strings, our `:string` type assertion has nothing to validate:

```lisp
HOUSE> (type-assertion 'blah :string)
NIL
HOUSE>
```

### All Together Now

We did it! We built a web framework on top of an event-driven webserver implementation. Our framework (and handler DSL) defines new applications by:

- Mapping URLs to handlers;
- Defining handlers to enforce the type safety and validation rules on requests;
- Optionally specifying new types for handlers as required.

Now we can describe our application like this:

```lisp
(defun len-between (min thing max)
  (>= max (length thing) min))

(define-handler (source :is-stream? nil)
    ((room :string (len-between 0 room 16)))
  (subscribe! (intern room :keyword) sock))

(define-handler (send-message)
    ((room :string (len-between 0 room 16))
     (name :string (len-between 1 name 64))
     (message :string (len-between 5 message 256)))
  (publish! (intern room :keyword)
	    (encode-json-to-string
	     `((:name . ,name) (:message . ,message)))))

(define-handler (index) ()
  (with-html-output-to-string (s nil :prologue t :indent t)
    (:html
     (:head (:script
	     :type "text/javascript"
	     :src "/static/js/interface.js"))
     (:body (:div :id "messages")
	    (:textarea :id "input")
	    (:button :id "send" "Send")))))

(start 4242)
```

Once we write `interface.js` to provide the client-side interactivity, this will start an HTTP chat server on port `4242` and listen for incoming connections.
