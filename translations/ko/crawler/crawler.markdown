title: asyncio 코루틴을 사용한 웹 크롤러
author: A. Jesse Jiryu Davis and Guido van Rossum
<markdown>
_A. Jesse Jiryu Davis는 뉴욕에 위치한 MongoDB의 스태프 엔지니어입니다. 그는 비동기 MongoDB Python 드라이버인 Motor를 작성했으며, MongoDB C 드라이버의 리드 개발자이자 PyMongo 팀의 멤버입니다. 그는 asyncio와 Tornado에 기여하고 있습니다. 그의 글은 [http://emptysqua.re](http://emptysqua.re)에서 볼 수 있습니다._

_Guido van Rossum은 웹과 그 너머에서 주요한 프로그래밍 언어 중 하나인 Python의 창시자입니다. Python 커뮤니티는 그를 BDFL(자비로운 종신 독재자, Benevolent Dictator For Life)이라고 부르는데, 이는 몬티 파이선 스케치에서 따온 제목입니다. Guido의 웹상 홈은 [http://www.python.org/~guido/](http://www.python.org/~guido/)입니다._
</markdown>
## 개요

고전적인 컴퓨터 과학은 계산을 가능한 한 빠르게 완료하는 효율적인 알고리즘을 강조합니다. 하지만 많은 네트워크 프로그램들은 계산에 시간을 보내는 것이 아니라, 느리거나 이벤트가 빈번하지 않은 수많은 연결을 열어두는 데 시간을 보냅니다. 이러한 프로그램들은 매우 다른 도전을 제시합니다: 바로 엄청나게 많은 네트워크 이벤트를 효율적으로 기다리는 것입니다. 이 문제에 대한 현대적인 접근법이 비동기 I/O, 즉 "async"입니다.

이 장에서는 간단한 웹 크롤러를 소개합니다. 크롤러는 많은 응답을 기다리지만 계산은 거의 하지 않기 때문에 전형적인 비동기 애플리케이션입니다. 한 번에 가져올 수 있는 페이지가 많을수록 더 빨리 완료됩니다. 진행 중인 각 요청에 스레드를 할당한다면, 동시 요청 수가 증가할수록 소켓이 부족해지기 전에 메모리나 다른 스레드 관련 리소스가 먼저 부족해질 것입니다. 비동기 I/O를 사용하여 스레드의 필요성을 피할 수 있습니다.

우리는 이 예제를 세 단계로 제시합니다. 첫째, 비동기 이벤트 루프를 보여주고 콜백과 함께 이벤트 루프를 사용하는 크롤러를 스케치합니다: 매우 효율적이지만 더 복잡한 문제로 확장하면 관리할 수 없는 스파게티 코드가 됩니다. 둘째, 따라서 Python 코루틴이 효율적이면서도 확장 가능하다는 것을 보여줍니다. 제너레이터 함수를 사용하여 Python에서 간단한 코루틴을 구현합니다. 세 번째 단계에서는 Python의 표준 "asyncio" 라이브러리[^16]의 완전한 기능을 갖춘 코루틴을 사용하고, 비동기 큐를 사용하여 이들을 조정합니다.

## 작업

웹 크롤러는 웹사이트의 모든 페이지를 찾아서 다운로드하며, 아마도 이를 아카이브하거나 인덱싱합니다. 루트 URL부터 시작하여 각 페이지를 가져오고, 아직 보지 않은 페이지들로의 링크를 파싱하여 이를 큐에 추가합니다. 아직 보지 않은 링크가 없는 페이지를 가져오고 큐가 비게 되면 멈춥니다.

많은 페이지를 동시에 다운로드하여 이 프로세스를 가속화할 수 있습니다. 크롤러가 새 링크를 찾으면, 별도의 소켓에서 새 페이지에 대한 동시 가져오기 작업을 시작합니다. 응답이 도착하면 이를 파싱하여 새 링크를 큐에 추가합니다. 너무 많은 동시성이 성능을 저하시키는 수익 감소 지점이 있을 수 있으므로, 동시 요청 수에 제한을 두고, 진행 중인 일부 요청이 완료될 때까지 나머지 링크는 큐에 남겨둡니다.

## 전통적인 접근법

어떻게 크롤러를 동시적으로 만들 수 있을까요? 전통적으로는 스레드 풀을 생성할 것입니다. 각 스레드는 소켓을 통해 한 번에 하나의 페이지를 다운로드하는 역할을 담당합니다. 예를 들어, `xkcd.com`에서 페이지를 다운로드하려면:

```python
def fetch(url):
    sock = socket.socket()
    sock.connect(('xkcd.com', 80))
    request = 'GET {} HTTP/1.0\r\nHost: xkcd.com\r\n\r\n'.format(url)
    sock.send(request.encode('ascii'))
    response = b''
    chunk = sock.recv(4096)
    while chunk:
        response += chunk
        chunk = sock.recv(4096)

    # Page is now downloaded.
    links = parse_links(response)
    q.add(links)
```

기본적으로 소켓 연산은 *블로킹*입니다: 스레드가 `connect`나 `recv` 같은 메서드를 호출하면, 연산이 완료될 때까지 일시 중지됩니다.[^15] 따라서 많은 페이지를 한 번에 다운로드하려면 많은 스레드가 필요합니다. 정교한 애플리케이션은 유휴 스레드를 스레드 풀에 유지한 다음 후속 작업에 재사용하기 위해 체크아웃하여 스레드 생성 비용을 상각합니다. 연결 풀에서 소켓에 대해서도 동일하게 수행합니다.

그러나 스레드는 비싸고, 운영 체제는 프로세스, 사용자 또는 머신이 가질 수 있는 스레드 수에 다양한 하드 캡을 적용합니다. Jesse의 시스템에서 Python 스레드는 약 50k의 메모리를 소비하며, 수만 개의 스레드를 시작하면 실패를 야기합니다. 동시 소켓에서 수만 개의 동시 연산으로 확장하면, 소켓이 부족해지기 전에 스레드가 먼저 부족해집니다. 스레드별 오버헤드나 스레드에 대한 시스템 제한이 병목점입니다.

영향력 있는 기사 "The C10K problem"[^8]에서 Dan Kegel은 I/O 동시성에 대한 멀티스레딩의 한계를 설명합니다. 그는 다음과 같이 시작합니다,

> 웹 서버가 동시에 1만 개의 클라이언트를 처리할 시간이 되었다고 생각하지 않나요? 결국, 웹은 이제 큰 공간이니까요.

Kegel은 1999년에 "C10K"라는 용어를 만들었습니다. 1만 개의 연결은 이제 소규모로 들리지만, 문제는 종류가 아니라 규모만 바뀌었습니다. 그 당시 C10K에 대해 연결당 스레드를 사용하는 것은 비실용적이었습니다. 이제 한계는 훨씬 더 높습니다. 실제로 우리의 장난감 웹 크롤러는 스레드로도 잘 작동할 것입니다. 하지만 수십만 개의 연결을 가진 매우 대규모 애플리케이션의 경우, 한계가 여전히 남아 있습니다: 대부분의 시스템이 여전히 소켓을 생성할 수 있지만 스레드가 부족해지는 한계가 있습니다. 이를 어떻게 극복할 수 있을까요?

## Async

비동기 I/O 프레임워크는 *논블로킹* 소켓을 사용하여 단일 스레드에서 동시 연산을 수행합니다. 우리의 비동기 크롤러에서는 서버에 연결을 시작하기 전에 소켓을 논블로킹으로 설정합니다:

```python
sock = socket.socket()
sock.setblocking(False)
try:
    sock.connect(('xkcd.com', 80))
except BlockingIOError:
    pass
```

짜증스럽게도, 논블로킹 소켓은 정상적으로 작동하고 있을 때조차 `connect`에서 예외를 발생시킵니다. 이 예외는 기반 C 함수의 짜증스러운 동작을 복제하는데, 이 함수는 시작되었음을 알리기 위해 `errno`를 `EINPROGRESS`로 설정합니다.

이제 우리 크롤러는 연결이 설정된 시점을 알 수 있는 방법이 필요합니다. 그래야 HTTP 요청을 보낼 수 있습니다. 간단히 타이트 루프에서 계속 시도할 수 있습니다:

```python
request = 'GET {} HTTP/1.0\r\nHost: xkcd.com\r\n\r\n'.format(url)
encoded = request.encode('ascii')

while True:
    try:
        sock.send(encoded)
        break  # Done.
    except OSError as e:
        pass

print('sent')
```

이 방법은 전력을 낭비할 뿐만 아니라, *여러* 소켓의 이벤트를 효율적으로 기다릴 수 없습니다. 고대에 BSD Unix의 이 문제에 대한 해결책은 `select`였는데, 이는 논블로킹 소켓이나 소켓의 작은 배열에서 이벤트가 발생하기를 기다리는 C 함수였습니다. 요즘에는 엄청나게 많은 연결을 가진 인터넷 애플리케이션에 대한 요구로 인해 `poll`, 그다음 BSD의 `kqueue`와 Linux의 `epoll` 같은 대체재가 등장했습니다. 이러한 API들은 `select`와 유사하지만 매우 많은 수의 연결에서 잘 작동합니다.

Python 3.4의 `DefaultSelector`는 시스템에서 사용 가능한 최고의 `select` 계열 함수를 사용합니다. 네트워크 I/O에 대한 알림을 등록하기 위해, 논블로킹 소켓을 만들고 기본 선택기에 등록합니다:

```python
from selectors import DefaultSelector, EVENT_WRITE

selector = DefaultSelector()

sock = socket.socket()
sock.setblocking(False)
try:
    sock.connect(('xkcd.com', 80))
except BlockingIOError:
    pass

def connected():
    selector.unregister(sock.fileno())
    print('connected!')

selector.register(sock.fileno(), EVENT_WRITE, connected)
```

의미 없는 오류를 무시하고 `selector.register`를 호출하여, 소켓의 파일 디스크립터와 기다리고 있는 이벤트를 표현하는 상수를 전달합니다. 연결이 설정되었을 때 알림을 받기 위해 `EVENT_WRITE`를 전달합니다: 즉, 소켓이 언제 "쓰기 가능한지" 알고 싶습니다. 또한 해당 이벤트가 발생했을 때 실행할 Python 함수 `connected`를 전달합니다. 이러한 함수를 *콜백*이라고 합니다.

선택기가 I/O 알림을 수신하면 루프에서 처리합니다:

```python
def loop():
    while True:
        events = selector.select()
        for event_key, event_mask in events:
            callback = event_key.data
            callback()
```

`connected` 콜백은 `event_key.data`로 저장되며, 논블로킹 소켓이 연결되면 이를 검색하고 실행합니다.

위의 빠르게 회전하는 루프와 달리, 여기서 `select` 호출은 다음 I/O 이벤트를 기다리며 일시 중지됩니다. 그러면 루프는 이러한 이벤트를 기다리고 있는 콜백들을 실행합니다. 완료되지 않은 연산들은 이벤트 루프의 향후 틱까지 대기 상태로 남아 있습니다.

우리가 이미 무엇을 보여주었을까요? 연산을 시작하고 연산이 준비되면 콜백을 실행하는 방법을 보여주었습니다. 비동기 *프레임워크*는 우리가 보여준 두 가지 기능&mdash;논블로킹 소켓과 이벤트 루프&mdash;을 기반으로 구축되어 단일 스레드에서 동시 연산을 실행합니다.

우리는 여기서 "동시성(concurrency)"을 달성했지만, 전통적으로 "병렬성(parallelism)"이라고 불리는 것은 아닙니다. 즉, 우리는 겹치는 I/O를 수행하는 작은 시스템을 구축했습니다. 다른 연산들이 진행 중인 동안 새로운 연산을 시작할 수 있습니다. 실제로는 병렬로 계산을 실행하기 위해 여러 코어를 활용하지는 않습니다. 하지만 이 시스템은 CPU 바운드 문제가 아닌 I/O 바운드 문제를 위해 설계되었습니다.[^14]

따라서 우리의 이벤트 루프는 각 연결에 스레드 리소스를 할당하지 않기 때문에 동시 I/O에서 효율적입니다. 하지만 계속하기 전에, async가 멀티스레딩보다 *빠르다*는 일반적인 오해를 바로잡는 것이 중요합니다. 종종 그렇지 않습니다&mdash;실제로 Python에서는 우리와 같은 이벤트 루프가 소수의 매우 활성적인 연결을 처리할 때 멀티스레딩보다 약간 느립니다. 전역 인터프리터 락이 없는 런타임에서는 스레드가 이러한 워크로드에서 훨씬 더 나은 성능을 보일 것입니다. 비동기 I/O가 적합한 것은 빈번하지 않은 이벤트를 가진 많은 느리거나 비활성적인 연결을 가진 애플리케이션입니다.[^11]<latex>[^bayer]</latex>

## 콜백을 사용한 프로그래밍

지금까지 구축한 작은 비동기 프레임워크로 어떻게 웹 크롤러를 만들 수 있을까요? 간단한 URL 가져오기조차 작성하기 고통스럽습니다.

아직 가져오지 않은 URL들과 이미 본 URL들의 전역 집합으로 시작합니다:

```python
urls_todo = set(['/'])
seen_urls = set(['/'])
```

`seen_urls` 집합은 `urls_todo`와 완료된 URL들을 포함합니다. 두 집합 모두 루트 URL "/"로 초기화됩니다.

페이지를 가져오려면 일련의 콜백들이 필요합니다. `connected` 콜백은 소켓이 연결될 때 실행되어 서버에 GET 요청을 보냅니다. 하지만 그 후 응답을 기다려야 하므로 또 다른 콜백을 등록합니다. 해당 콜백이 실행될 때 전체 응답을 아직 읽을 수 없다면, 다시 등록하는 식으로 계속됩니다.

이러한 콜백들을 `Fetcher` 객체로 모아봅시다. URL, 소켓 객체, 그리고 응답 바이트를 축적할 공간이 필요합니다:

```python
class Fetcher:
    def __init__(self, url):
        self.response = b''  # Empty array of bytes.
        self.url = url
        self.sock = None
```

`Fetcher.fetch`를 호출하는 것으로 시작합니다:

```python
    # Method on Fetcher class.
    def fetch(self):
        self.sock = socket.socket()
        self.sock.setblocking(False)
        try:
            self.sock.connect(('xkcd.com', 80))
        except BlockingIOError:
            pass

        # Register next callback.
        selector.register(self.sock.fileno(),
                          EVENT_WRITE,
                          self.connected)
```

`fetch` 메서드는 소켓 연결을 시작합니다. 하지만 연결이 설정되기 전에 메서드가 반환된다는 점에 주목하세요. 연결을 기다리기 위해 이벤트 루프에게 제어권을 넘겨야 합니다. 왜 그런지 이해하려면, 전체 애플리케이션이 다음과 같이 구조화되어 있다고 상상해보세요:

```python
# Begin fetching http://xkcd.com/353/
fetcher = Fetcher('/353/')
fetcher.fetch()

while True:
    events = selector.select()
    for event_key, event_mask in events:
        callback = event_key.data
        callback(event_key, event_mask)
```

모든 이벤트 알림은 이벤트 루프가 `select`를 호출할 때 처리됩니다. 따라서 `fetch`는 프로그램이 소켓이 언제 연결되었는지 알 수 있도록 이벤트 루프에게 제어권을 넘겨야 합니다. 그래야만 루프가 위의 `fetch` 끝에서 등록된 `connected` 콜백을 실행합니다.

다음은 `connected`의 구현입니다:

```python
    # Method on Fetcher class.
    def connected(self, key, mask):
        print('connected!')
        selector.unregister(key.fd)
        request = 'GET {} HTTP/1.0\r\nHost: xkcd.com\r\n\r\n'.format(self.url)
        self.sock.send(request.encode('ascii'))

        # Register the next callback.
        selector.register(key.fd,
                          EVENT_READ,
                          self.read_response)
```

이 메서드는 GET 요청을 보냅니다. 실제 애플리케이션이라면 전체 메시지가 한 번에 보내지지 않을 경우를 대비해 `send`의 반환값을 확인할 것입니다. 하지만 우리의 요청은 작고 애플리케이션도 단순합니다. 태연하게 `send`를 호출한 다음 응답을 기다립니다. 물론, 또 다른 콜백을 등록하고 이벤트 루프에게 제어권을 넘겨야 합니다. 다음이자 마지막 콜백인 `read_response`는 서버의 응답을 처리합니다:

```python
    # Method on Fetcher class.
    def read_response(self, key, mask):
        global stopped

        chunk = self.sock.recv(4096)  # 4k chunk size.
        if chunk:
            self.response += chunk
        else:
            selector.unregister(key.fd)  # Done reading.
            links = self.parse_links()

            # Python set-logic:
            for link in links.difference(seen_urls):
                urls_todo.add(link)
                Fetcher(link).fetch()  # <- New Fetcher.

            seen_urls.update(links)
            urls_todo.remove(self.url)
            if not urls_todo:
                stopped = True
```

이 콜백은 선택기가 소켓이 "읽기 가능하다"고 볼 때마다 실행되며, 이는 두 가지를 의미할 수 있습니다: 소켓에 데이터가 있거나 닫혔다는 것입니다.

콜백은 소켓으로부터 최대 4킬로바이트의 데이터를 요청합니다. 준비된 데이터가 그보다 적다면, `chunk`는 사용 가능한 데이터만 포함합니다. 더 많다면, `chunk`는 4킬로바이트 길이이고 소켓은 읽기 가능한 상태로 남아있으므로, 이벤트 루프는 다음 틱에서 이 콜백을 다시 실행합니다. 응답이 완료되면, 서버가 소켓을 닫고 `chunk`는 비어있게 됩니다.

표시되지 않은 `parse_links` 메서드는 URL들의 집합을 반환합니다. 동시성 제한 없이 각 새로운 URL에 대해 새 가져오기 도구를 시작합니다. 콜백을 사용한 비동기 프로그래밍의 좋은 특징을 주목하세요: `seen_urls`에 링크를 추가할 때와 같이 공유 데이터의 변경에 대해 뮤텍스가 필요하지 않습니다. 선점적 멀티태스킹이 없으므로 코드의 임의의 지점에서 중단될 수 없습니다.

전역 `stopped` 변수를 추가하고 이를 사용해 루프를 제어합니다:

```python
stopped = False

def loop():
    while not stopped:
        events = selector.select()
        for event_key, event_mask in events:
            callback = event_key.data
            callback()
```

모든 페이지가 다운로드되면 가져오기 도구가 전역 이벤트 루프를 중지하고 프로그램이 종료됩니다.

이 예제는 비동기의 문제를 명확히 보여줍니다: 스파게티 코드입니다. 일련의 계산과 I/O 연산을 표현하고, 이러한 연산 시리즈 여러 개를 동시에 실행하도록 스케줄링하는 방법이 필요합니다. 하지만 스레드 없이는 일련의 연산들을 단일 함수로 모을 수 없습니다: 함수가 I/O 연산을 시작할 때마다, 미래에 필요한 상태를 명시적으로 저장한 다음 반환해야 합니다. 이러한 상태 저장 코드를 생각하고 작성하는 것은 여러분의 책임입니다.

이것이 무엇을 의미하는지 설명해보겠습니다. 기존의 블로킹 소켓을 사용해 스레드에서 URL을 얼마나 간단하게 가져왔는지 생각해보세요:

```python
# Blocking version.
def fetch(url):
    sock = socket.socket()
    sock.connect(('xkcd.com', 80))
    request = 'GET {} HTTP/1.0\r\nHost: xkcd.com\r\n\r\n'.format(url)
    sock.send(request.encode('ascii'))
    response = b''
    chunk = sock.recv(4096)
    while chunk:
        response += chunk
        chunk = sock.recv(4096)
    
    # Page is now downloaded.
    links = parse_links(response)
    q.add(links)
```

What state does this function remember between one socket operation and the next? It has the socket, a URL, and the accumulating `response`.  A function that runs on a thread uses basic features of the programming language to store this temporary state in local variables, on its stack. The function also has a "continuation"&mdash;that is, the code it plans to execute after I/O completes. The runtime remembers the continuation by storing the thread's instruction pointer. You need not think about restoring these local variables and the continuation after I/O. It is built in to the language.

But with a callback-based async framework, these language features are no help. While waiting for I/O, a function must save its state explicitly, because the function returns and loses its stack frame before I/O completes. In lieu of local variables, our callback-based example stores `sock` and `response` as attributes of `self`, the Fetcher instance. In lieu of the instruction pointer, it stores its continuation by registering the callbacks `connected` and `read_response`. As the application's features grow, so does the complexity of the state we manually save across callbacks. Such onerous bookkeeping makes the coder prone to migraines.

Even worse, what happens if a callback throws an exception, before it schedules the next callback in the chain? Say we did a poor job on the `parse_links` method and it throws an exception parsing some HTML:

```
Traceback (most recent call last):
  File "loop-with-callbacks.py", line 111, in <module>
    loop()
  File "loop-with-callbacks.py", line 106, in loop
    callback(event_key, event_mask)
  File "loop-with-callbacks.py", line 51, in read_response
    links = self.parse_links()
  File "loop-with-callbacks.py", line 67, in parse_links
    raise Exception('parse error')
Exception: parse error
```

The stack trace shows only that the event loop was running a callback. We do not remember what led to the error. The chain is broken on both ends: we forgot where we were going and whence we came. This loss of context is called "stack ripping", and in many cases it confounds the investigator. Stack ripping also prevents us from installing an exception handler for a chain of callbacks, the way a "try / except" block wraps a function call and its tree of descendents.[^7]

So, even apart from the long debate about the relative efficiencies of multithreading and async, there is this other debate regarding which is more error-prone: threads are susceptible to data races if you make a mistake synchronizing them, but callbacks are stubborn to debug due to stack ripping. 

## Coroutines

We entice you with a promise. It is possible to write asynchronous code that combines the efficiency of callbacks with the classic good looks of multithreaded programming. This combination is achieved with a pattern called "coroutines". Using Python 3.4's standard asyncio library, and a package called "aiohttp", fetching a URL in a coroutine is very direct[^10]:

```python
    @asyncio.coroutine
    def fetch(self, url):
        response = yield from self.session.get(url)
        body = yield from response.read()
```

It is also scalable. Compared to the 50k of memory per thread and the operating system's hard limits on threads, a Python coroutine takes barely 3k of memory on Jesse's system. Python can easily start hundreds of thousands of coroutines.

The concept of a coroutine, dating to the elder days of computer science, is simple: it is a subroutine that can be paused and resumed. Whereas threads are preemptively multitasked by the operating system, coroutines multitask cooperatively: they choose when to pause, and which coroutine to run next.

There are many implementations of coroutines; even in Python there are several. The coroutines in the standard "asyncio" library in Python 3.4 are built upon generators, a Future class, and the "yield from" statement. Starting in Python 3.5, coroutines are a native feature of the language itself[^17]; however, understanding coroutines as they were first implemented in Python 3.4, using pre-existing language facilities, is the foundation to tackle Python 3.5's native coroutines.

To explain Python 3.4's generator-based coroutines, we will engage in an exposition of generators and how they are used as coroutines in asyncio, and trust you will enjoy reading it as much as we enjoyed writing it. Once we have explained generator-based coroutines, we shall use them in our async web crawler.

## How Python Generators Work

Before you grasp Python generators, you have to understand how regular Python functions work. Normally, when a Python function calls a subroutine, the subroutine retains control until it returns, or throws an exception. Then control returns to the caller:

```python
>>> def foo():
...     bar()
...
>>> def bar():
...     pass
```

The standard Python interpreter is written in C. The C function that executes a Python function is called, mellifluously, `PyEval_EvalFrameEx`. It takes a Python stack frame object and evaluates Python bytecode in the context of the frame. Here is the bytecode for `foo`:

```python
>>> import dis
>>> dis.dis(foo)
  2           0 LOAD_GLOBAL              0 (bar)
              3 CALL_FUNCTION            0 (0 positional, 0 keyword pair)
              6 POP_TOP
              7 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

The `foo` function loads `bar` onto its stack and calls it, then pops its return value from the stack, loads `None` onto the stack, and returns `None`.

When `PyEval_EvalFrameEx` encounters the `CALL_FUNCTION` bytecode, it creates a new Python stack frame and recurses: that is, it calls `PyEval_EvalFrameEx` recursively with the new frame, which is used to execute `bar`.

It is crucial to understand that Python stack frames are allocated in heap memory! The Python interpreter is a normal C program, so its stack frames are normal stack frames. But the *Python* stack frames it manipulates are on the heap. Among other surprises, this means a Python stack frame can outlive its function call. To see this interactively, save the current frame from within `bar`:

```python
>>> import inspect
>>> frame = None
>>> def foo():
...     bar()
...
>>> def bar():
...     global frame
...     frame = inspect.currentframe()
...
>>> foo()
>>> # The frame was executing the code for 'bar'.
>>> frame.f_code.co_name
'bar'
>>> # Its back pointer refers to the frame for 'foo'.
>>> caller_frame = frame.f_back
>>> caller_frame.f_code.co_name
'foo'
```

\aosafigure[240pt]{crawler-images/function-calls.png}{Function Calls}{500l.crawler.functioncalls}

The stage is now set for Python generators, which use the same building blocks&mdash;code objects and stack frames&mdash;to marvelous effect.

This is a generator function:

```python
>>> def gen_fn():
...     result = yield 1
...     print('result of yield: {}'.format(result))
...     result2 = yield 2
...     print('result of 2nd yield: {}'.format(result2))
...     return 'done'
...     
```

When Python compiles `gen_fn` to bytecode, it sees the `yield` statement and knows that `gen_fn` is a generator function, not a regular one. It sets a flag to remember this fact:

```python
>>> # The generator flag is bit position 5.
>>> generator_bit = 1 << 5
>>> bool(gen_fn.__code__.co_flags & generator_bit)
True
```

When you call a generator function, Python sees the generator flag, and it does not actually run the function. Instead, it creates a generator:

```python
>>> gen = gen_fn()
>>> type(gen)
<class 'generator'>
```

A Python generator encapsulates a stack frame plus a reference to some code, the body of `gen_fn`:

```python
>>> gen.gi_code.co_name
'gen_fn'
```

All generators from calls to `gen_fn` point to this same code. But each has its own stack frame. This stack frame is not on any actual stack, it sits in heap memory waiting to be used:

\aosafigure[240pt]{crawler-images/generator.png}{Generators}{500l.crawler.generators}

The frame has a "last instruction" pointer, the instruction it executed most recently. In the beginning, the last instruction pointer is -1, meaning the generator has not begun:

```python
>>> gen.gi_frame.f_lasti
-1
```

When we call `send`, the generator reaches its first `yield`, and pauses. The return value of `send` is 1, since that is what `gen` passes to the `yield` expression:

```python
>>> gen.send(None)
1
```

The generator's instruction pointer is now 3 bytecodes from the start, part way through the 56 bytes of compiled Python:

```python
>>> gen.gi_frame.f_lasti
3
>>> len(gen.gi_code.co_code)
56
```

The generator can be resumed at any time, from any function, because its stack frame is not actually on the stack: it is on the heap. Its position in the call hierarchy is not fixed, and it need not obey the first-in, last-out order of execution that regular functions do. It is liberated, floating free like a cloud.

We can send the value "hello" into the generator and it becomes the result of the `yield` expression, and the generator continues until it yields 2:

```python
>>> gen.send('hello')
result of yield: hello
2
```

Its stack frame now contains the local variable `result`:

```python
>>> gen.gi_frame.f_locals
{'result': 'hello'}
```

Other generators created from `gen_fn` will have their own stack frames and local variables.

When we call `send` again, the generator continues from its second `yield`, and finishes by raising the special `StopIteration` exception:

```python
>>> gen.send('goodbye')
result of 2nd yield: goodbye
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration: done
```

The exception has a value, which is the return value of the generator: the string `"done"`.

## Building Coroutines With Generators

So a generator can pause, and it can be resumed with a value, and it has a return value. Sounds like a good primitive upon which to build an async programming model, without spaghetti callbacks! We want to build a "coroutine": a routine that is cooperatively scheduled with other routines in the program. Our coroutines will be a simplified version of those in Python's standard "asyncio" library. As in asyncio, we will use generators, futures, and the "yield from" statement.

First we need a way to represent some future result that a coroutine is waiting for. A stripped-down version:

```python
class Future:
    def __init__(self):
        self.result = None
        self._callbacks = []

    def add_done_callback(self, fn):
        self._callbacks.append(fn)

    def set_result(self, result):
        self.result = result
        for fn in self._callbacks:
            fn(self)
```

A future is initially "pending". It is "resolved" by a call to `set_result`.[^12]

Let us adapt our fetcher to use futures and coroutines. We wrote `fetch` with a callback:

```python
class Fetcher:
    def fetch(self):
        self.sock = socket.socket()
        self.sock.setblocking(False)
        try:
            self.sock.connect(('xkcd.com', 80))
        except BlockingIOError:
            pass
        selector.register(self.sock.fileno(),
                          EVENT_WRITE,
                          self.connected)

    def connected(self, key, mask):
        print('connected!')
        # And so on....
```

The `fetch` method begins connecting a socket, then registers the callback, `connected`, to be executed when the socket is ready. Now we can combine these two steps into one coroutine:

```python
    def fetch(self):
        sock = socket.socket()
        sock.setblocking(False)
        try:
            sock.connect(('xkcd.com', 80))
        except BlockingIOError:
            pass

        f = Future()

        def on_connected():
            f.set_result(None)

        selector.register(sock.fileno(),
                          EVENT_WRITE,
                          on_connected)
        yield f
        selector.unregister(sock.fileno())
        print('connected!')
```

Now `fetch` is a generator function, rather than a regular one, because it contains a `yield` statement. We create a pending future, then yield it to pause `fetch` until the socket is ready. The inner function `on_connected` resolves the future.

But when the future resolves, what resumes the generator? We need a coroutine *driver*. Let us call it "task":

```python
class Task:
    def __init__(self, coro):
        self.coro = coro
        f = Future()
        f.set_result(None)
        self.step(f)

    def step(self, future):
        try:
            next_future = self.coro.send(future.result)
        except StopIteration:
            return

        next_future.add_done_callback(self.step)

# Begin fetching http://xkcd.com/353/
fetcher = Fetcher('/353/')
Task(fetcher.fetch())

loop()
```

The task starts the `fetch` generator by sending `None` into it. Then `fetch` runs until it yields a future, which the task captures as `next_future`. When the socket is connected, the event loop runs the callback `on_connected`, which resolves the future, which calls `step`, which resumes `fetch`.

## Factoring Coroutines With `yield from`

Once the socket is connected, we send the HTTP GET request and read the server response. These steps need no longer be scattered among callbacks; we gather them into the same generator function:

```python
    def fetch(self):
        # ... connection logic from above, then:
        sock.send(request.encode('ascii'))

        while True:
            f = Future()

            def on_readable():
                f.set_result(sock.recv(4096))

            selector.register(sock.fileno(),
                              EVENT_READ,
                              on_readable)
            chunk = yield f
            selector.unregister(sock.fileno())
            if chunk:
                self.response += chunk
            else:
                # Done reading.
                break
```

This code, which reads a whole message from a socket, seems generally useful. How can we factor it from `fetch` into a subroutine? Now Python 3's celebrated `yield from` takes the stage. It lets one generator *delegate* to another.

To see how, let us return to our simple generator example:

```python
>>> def gen_fn():
...     result = yield 1
...     print('result of yield: {}'.format(result))
...     result2 = yield 2
...     print('result of 2nd yield: {}'.format(result2))
...     return 'done'
...     
```

To call this generator from another generator, delegate to it with `yield from`:

```python
>>> # Generator function:
>>> def caller_fn():
...     gen = gen_fn()
...     rv = yield from gen
...     print('return value of yield-from: {}'
...           .format(rv))
...
>>> # Make a generator from the
>>> # generator function.
>>> caller = caller_fn()
```

The `caller` generator acts as if it were `gen`, the generator it is delegating to:

```python
>>> caller.send(None)
1
>>> caller.gi_frame.f_lasti
15
>>> caller.send('hello')
result of yield: hello
2
>>> caller.gi_frame.f_lasti  # Hasn't advanced.
15
>>> caller.send('goodbye')
result of 2nd yield: goodbye
return value of yield-from: done
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration
```

While `caller` yields from `gen`, `caller` does not advance. Notice that its instruction pointer remains at 15, the site of its `yield from` statement, even while the inner generator `gen` advances from one `yield` statement to the next.[^13] From our perspective outside `caller`, we cannot tell if the values it yields are from `caller` or from the generator it delegates to. And from inside `gen`, we cannot tell if values are sent in from `caller` or from outside it. The `yield from` statement is a frictionless channel, through which values flow in and out of `gen` until `gen` completes.

A coroutine can delegate work to a sub-coroutine with `yield from` and receive the result of the work. Notice, above, that `caller` printed "return value of yield-from: done". When `gen` completed, its return value became the value of the `yield from` statement in `caller`:

```python
    rv = yield from gen
```

Earlier, when we criticized callback-based async programming, our most strident complaint was about "stack ripping": when a callback throws an exception, the stack trace is typically useless. It only shows that the event loop was running the callback, not *why*. How do coroutines fare?

```python
>>> def gen_fn():
...     raise Exception('my error')
>>> caller = caller_fn()
>>> caller.send(None)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 3, in caller_fn
  File "<input>", line 2, in gen_fn
Exception: my error
```

This is much more useful! The stack trace shows `caller_fn` was delegating to `gen_fn` when it threw the error. Even more comforting, we can wrap the call to a sub-coroutine in an exception handler, the same is with normal subroutines:

```python
>>> def gen_fn():
...     yield 1
...     raise Exception('uh oh')
...
>>> def caller_fn():
...     try:
...         yield from gen_fn()
...     except Exception as exc:
...         print('caught {}'.format(exc))
...
>>> caller = caller_fn()
>>> caller.send(None)
1
>>> caller.send('hello')
caught uh oh
```

So we factor logic with sub-coroutines just like with regular subroutines. Let us factor some useful sub-coroutines from our fetcher. We write a `read` coroutine to receive one chunk:

```python
def read(sock):
    f = Future()

    def on_readable():
        f.set_result(sock.recv(4096))

    selector.register(sock.fileno(), EVENT_READ, on_readable)
    chunk = yield f  # Read one chunk.
    selector.unregister(sock.fileno())
    return chunk
```

We build on `read` with a `read_all` coroutine that receives a whole message:

```python
def read_all(sock):
    response = []
    # Read whole response.
    chunk = yield from read(sock)
    while chunk:
        response.append(chunk)
        chunk = yield from read(sock)

    return b''.join(response)
```

If you squint the right way, the `yield from` statements disappear and these look like conventional functions doing blocking I/O. But in fact, `read` and `read_all` are coroutines. Yielding from `read` pauses `read_all` until the I/O completes. While `read_all` is paused, asyncio's event loop does other work and awaits other I/O events; `read_all` is resumed with the result of `read` on the next loop tick once its event is ready.

At the stack's root, `fetch` calls `read_all`:

```python
class Fetcher:
    def fetch(self):
		 # ... connection logic from above, then:
        sock.send(request.encode('ascii'))
        self.response = yield from read_all(sock)
```

Miraculously, the Task class needs no modification. It drives the outer `fetch` coroutine just the same as before:

```python
Task(fetcher.fetch())
loop()
```

When `read` yields a future, the task receives it through the channel of `yield from` statements, precisely as if the future were yielded directly from `fetch`. When the loop resolves a future, the task sends its result into `fetch`, and the value is received by `read`, exactly as if the task were driving `read` directly:

\aosafigure[240pt]{crawler-images/yield-from.png}{Yield From}{500l.crawler.yieldfrom}

To perfect our coroutine implementation, we polish out one mar: our code uses `yield` when it waits for a future, but `yield from` when it delegates to a sub-coroutine. It would be more refined if we used `yield from` whenever a coroutine pauses. Then a coroutine need not concern itself with what type of thing it awaits.

We take advantage of the deep correspondence in Python between generators and iterators. Advancing a generator is, to the caller, the same as advancing an iterator. So we make our Future class iterable by implementing a special method:

```python
    # Method on Future class.
    def __iter__(self):
        # Tell Task to resume me here.
        yield self
        return self.result
```

The future's `__iter__` method is a coroutine that yields the future itself. Now when we replace code like this:

```python
# f is a Future.
yield f
```

...with this:

```python
# f is a Future.
yield from f
```

...the outcome is the same! The driving Task receives the future from its call to `send`, and when the future is resolved it sends the new result back into the coroutine.

What is the advantage of using `yield from` everywhere? Why is that better than waiting for futures with `yield` and delegating to sub-coroutines with `yield from`? It is better because now, a method can freely change its implementation without affecting the caller: it might be a normal method that returns a future that will *resolve* to a value, or it might be a coroutine that contains `yield from` statements and *returns* a value. In either case, the caller need only `yield from` the method in order to wait for the result.

Gentle reader, we have reached the end of our enjoyable exposition of coroutines in asyncio. We peered into the machinery of generators, and sketched an implementation of futures and tasks. We outlined how asyncio attains the best of both worlds: concurrent I/O that is more efficient than threads and more legible than callbacks. Of course, the real asyncio is much more sophisticated than our sketch. The real framework addresses zero-copy I/O, fair scheduling, exception handling, and an abundance of other features.

To an asyncio user, coding with coroutines is much simpler than you saw here. In the code above we implemented coroutines from first principles, so you saw callbacks, tasks, and futures. You even saw non-blocking sockets and the call to ``select``. But when it comes time to build an application with asyncio, none of this appears in your code. As we promised, you can now sleekly fetch a URL:

```python
    @asyncio.coroutine
    def fetch(self, url):
        response = yield from self.session.get(url)
        body = yield from response.read()
```

Satisfied with this exposition, we return to our original assignment: to write an async web crawler, using asyncio.

## Coordinating Coroutines

We began by describing how we want our crawler to work. Now it is time to implement it with asyncio coroutines.

Our crawler will fetch the first page, parse its links, and add them to a queue. After this it fans out across the website, fetching pages concurrently. But to limit load on the client and server, we want some maximum number of workers to run, and no more. Whenever a worker finishes fetching a page, it should immediately pull the next link from the queue. We will pass through periods when there is not enough work to go around, so some workers must pause. But when a worker hits a page rich with new links, then the queue suddenly grows and any paused workers should wake and get cracking. Finally, our program must quit once its work is done.

Imagine if the workers were threads. How would we express the crawler's algorithm? We could use a synchronized queue[^5] from the Python standard library. Each time an item is put in the queue, the queue increments its count of "tasks". Worker threads call `task_done` after completing work on an item. The main thread blocks on `Queue.join` until each item put in the queue is matched by a `task_done` call, then it exits.

Coroutines use the exact same pattern with an asyncio queue! First we import it[^6]:

```python
try:
    from asyncio import JoinableQueue as Queue
except ImportError:
    # In Python 3.5, asyncio.JoinableQueue is
    # merged into Queue.
    from asyncio import Queue
```

We collect the workers' shared state in a crawler class, and write the main logic in its `crawl` method. We start `crawl` on a coroutine and run asyncio's event loop until `crawl` finishes:

```python
loop = asyncio.get_event_loop()

crawler = crawling.Crawler('http://xkcd.com',
                           max_redirect=10)

loop.run_until_complete(crawler.crawl())
```

The crawler begins with a root URL and `max_redirect`, the number of redirects it is willing to follow to fetch any one URL. It puts the pair `(URL, max_redirect)` in the queue. (For the reason why, stay tuned.)

```python
class Crawler:
    def __init__(self, root_url, max_redirect):
        self.max_tasks = 10
        self.max_redirect = max_redirect
        self.q = Queue()
        self.seen_urls = set()
        
        # aiohttp's ClientSession does connection pooling and
        # HTTP keep-alives for us.
        self.session = aiohttp.ClientSession(loop=loop)
        
        # Put (URL, max_redirect) in the queue.
        self.q.put((root_url, self.max_redirect))
```

The number of unfinished tasks in the queue is now one. Back in our main script, we launch the event loop and the `crawl` method:

```python
loop.run_until_complete(crawler.crawl())
```

The `crawl` coroutine kicks off the workers. It is like a main thread: it blocks on `join` until all tasks are finished, while the workers run in the background.

```python
    @asyncio.coroutine
    def crawl(self):
        """Run the crawler until all work is done."""
        workers = [asyncio.Task(self.work())
                   for _ in range(self.max_tasks)]

        # When all work is done, exit.
        yield from self.q.join()
        for w in workers:
            w.cancel()
```

If the workers were threads we might not wish to start them all at once. To avoid creating expensive threads until it is certain they are necessary, a thread pool typically grows on demand. But coroutines are cheap, so we simply start the maximum number allowed.

It is interesting to note how we shut down the crawler. When the `join` future resolves, the worker tasks are alive but suspended: they wait for more URLs but none come. So, the main coroutine cancels them before exiting. Otherwise, as the Python interpreter shuts down and calls all objects' destructors, living tasks cry out:

```
ERROR:asyncio:Task was destroyed but it is pending!
```

And how does `cancel` work? Generators have a feature we have not yet shown you. You can throw an exception into a generator from outside:

```python
>>> gen = gen_fn()
>>> gen.send(None)  # Start the generator as usual.
1
>>> gen.throw(Exception('error'))
Traceback (most recent call last):
  File "<input>", line 3, in <module>
  File "<input>", line 2, in gen_fn
Exception: error
```

The generator is resumed by `throw`, but it is now raising an exception. If no code in the generator's call stack catches it, the exception bubbles back up to the top. So to cancel a task's coroutine:

```python
    # Method of Task class.
    def cancel(self):
        self.coro.throw(CancelledError)
```

Wherever the generator is paused, at some `yield from` statement, it resumes and throws an exception. We handle cancellation in the task's `step` method:

```python
    # Method of Task class.
    def step(self, future):
        try:
            next_future = self.coro.send(future.result)
        except CancelledError:
            self.cancelled = True
            return
        except StopIteration:
            return

        next_future.add_done_callback(self.step)
```

Now the task knows it is cancelled, so when it is destroyed it does not rage against the dying of the light.

Once `crawl` has canceled the workers, it exits. The event loop sees that the coroutine is complete (we shall see how later), and it too exits:

```python
loop.run_until_complete(crawler.crawl())
```

The `crawl` method comprises all that our main coroutine must do. It is the worker coroutines that get URLs from the queue, fetch them, and parse them for new links. Each worker runs the `work` coroutine independently:

```python
    @asyncio.coroutine
    def work(self):
        while True:
            url, max_redirect = yield from self.q.get()

            # Download page and add new links to self.q.
            yield from self.fetch(url, max_redirect)
            self.q.task_done()
```

Python sees that this code contains `yield from` statements, and compiles it into a generator function. So in `crawl`, when the main coroutine calls `self.work` ten times, it does not actually execute this method: it only creates ten generator objects with references to this code. It wraps each in a Task. The Task receives each future the generator yields, and drives the generator by calling `send` with each future's result when the future resolves. Because the generators have their own stack frames, they run independently, with separate local variables and instruction pointers.

The worker coordinates with its fellows via the queue. It waits for new URLs with:

```python
    url, max_redirect = yield from self.q.get()
```

The queue's `get` method is itself a coroutine: it pauses until someone puts an item in the queue, then resumes and returns the item.

Incidentally, this is where the worker will be paused at the end of the crawl, when the main coroutine cancels it. From the coroutine's perspective, its last trip around the loop ends when `yield from` raises a `CancelledError`.

When a worker fetches a page it parses the links and puts new ones in the queue, then calls `task_done` to decrement the counter. Eventually, a worker fetches a page whose URLs have all been fetched already, and there is also no work left in the queue. Thus this worker's call to `task_done` decrements the counter to zero. Then `crawl`, which is waiting for the queue's `join` method, is unpaused and finishes.

We promised to explain why the items in the queue are pairs, like:

```python
# URL to fetch, and the number of redirects left.
('http://xkcd.com/353', 10)
```

New URLs have ten redirects remaining. Fetching this particular URL results in a redirect to a new location with a trailing slash. We decrement the number of redirects remaining, and put the next location in the queue:

```python
# URL with a trailing slash. Nine redirects left.
('http://xkcd.com/353/', 9)
```

The `aiohttp` package we use would follow redirects by default and give us the final response. We tell it not to, however, and handle redirects in the crawler, so it can coalesce redirect paths that lead to the same destination: if we have already seen this URL, it is in ``self.seen_urls`` and we have already started on this path from a different entry point:

\aosafigure[240pt]{crawler-images/redirects.png}{Redirects}{500l.crawler.redirects}

The crawler fetches "foo" and sees it redirects to "baz", so it adds "baz" to
the queue and to ``seen_urls``. If the next page it fetches is "bar", which
also redirects to "baz", the fetcher does not enqueue "baz" again. If the
response is a page, rather than a redirect, `fetch` parses it for links and
puts new ones in the queue.

```python
    @asyncio.coroutine
    def fetch(self, url, max_redirect):
        # Handle redirects ourselves.
        response = yield from self.session.get(
            url, allow_redirects=False)

        try:
            if is_redirect(response):
                if max_redirect > 0:
                    next_url = response.headers['location']
                    if next_url in self.seen_urls:
                        # We have been down this path before.
                        return
    
                    # Remember we have seen this URL.
                    self.seen_urls.add(next_url)
                    
                    # Follow the redirect. One less redirect remains.
                    self.q.put_nowait((next_url, max_redirect - 1))
    	     else:
    	         links = yield from self.parse_links(response)
    	         # Python set-logic:
    	         for link in links.difference(self.seen_urls):
                    self.q.put_nowait((link, self.max_redirect))
                self.seen_urls.update(links)
        finally:
            # Return connection to pool.
            yield from response.release()
```

If this were multithreaded code, it would be lousy with race conditions. For example, the worker checks if a link is in `seen_urls`, and if not the worker puts it in the queue and adds it to `seen_urls`. If it were interrupted between the two operations, then another worker might parse the same link from a different page, also observe that it is not in `seen_urls`, and also add it to the queue. Now that same link is in the queue twice, leading (at best) to duplicated work and wrong statistics.

However, a coroutine is only vulnerable to interruption at `yield from` statements. This is a key difference that makes coroutine code far less prone to races than multithreaded code: multithreaded code must enter a critical section explicitly, by grabbing a lock, otherwise it is interruptible. A Python coroutine is uninterruptible by default, and only cedes control when it explicitly yields.

We no longer need a fetcher class like we had in the callback-based program. That class was a workaround for a deficiency of callbacks: they need some place to store state while waiting for I/O, since their local variables are not preserved across calls. But the `fetch` coroutine can store its state in local variables like a regular function does, so there is no more need for a class.

When `fetch` finishes processing the server response it returns to the caller, `work`. The `work` method calls `task_done` on the queue and then gets the next URL from the queue to be fetched.

When `fetch` puts new links in the queue it increments the count of unfinished tasks and keeps the main coroutine, which is waiting for `q.join`, paused. If, however, there are no unseen links and this was the last URL in the queue, then when `work` calls `task_done` the count of unfinished tasks falls to zero. That event unpauses `join` and the main coroutine completes.

The queue code that coordinates the workers and the main coroutine is like this[^9]:

```python
class Queue:
    def __init__(self):
        self._join_future = Future()
        self._unfinished_tasks = 0
        # ... other initialization ...
    
    def put_nowait(self, item):
        self._unfinished_tasks += 1
        # ... store the item ...

    def task_done(self):
        self._unfinished_tasks -= 1
        if self._unfinished_tasks == 0:
            self._join_future.set_result(None)

    @asyncio.coroutine
    def join(self):
        if self._unfinished_tasks > 0:
            yield from self._join_future
```

The main coroutine, `crawl`, yields from `join`. So when the last worker decrements the count of unfinished tasks to zero, it signals `crawl` to resume, and finish.

The ride is almost over. Our program began with the call to `crawl`:

```python
loop.run_until_complete(self.crawler.crawl())
```

How does the program end? Since `crawl` is a generator function, calling it returns a generator. To drive the generator, asyncio wraps it in a task:

```python
class EventLoop:
    def run_until_complete(self, coro):
        """Run until the coroutine is done."""
        task = Task(coro)
        task.add_done_callback(stop_callback)
        try:
            self.run_forever()
        except StopError:
            pass

class StopError(BaseException):
    """Raised to stop the event loop."""

def stop_callback(future):
    raise StopError
```

When the task completes, it raises `StopError `, which the loop uses as a signal that it has arrived at normal completion.

But what's this? The task has methods called `add_done_callback` and `result`? You might think that a task resembles a future. Your instinct is correct. We must admit a detail about the Task class we hid from you: a task is a future.

```python
class Task(Future):
    """A coroutine wrapped in a Future."""
```

Normally a future is resolved by someone else calling `set_result` on it. But a task resolves *itself* when its coroutine stops. Remember from our earlier exploration of Python generators that when a generator returns, it throws the special `StopIteration` exception:

```python
    # Method of class Task.
    def step(self, future):
        try:
            next_future = self.coro.send(future.result)
        except CancelledError:
            self.cancelled = True
            return
        except StopIteration as exc:

            # Task resolves itself with coro's return
            # value.
            self.set_result(exc.value)
            return

        next_future.add_done_callback(self.step)
```

So when the event loop calls `task.add_done_callback(stop_callback)`, it prepares to be stopped by the task. Here is `run_until_complete` again:

```python
    # Method of event loop.
    def run_until_complete(self, coro):
        task = Task(coro)
        task.add_done_callback(stop_callback)
        try:
            self.run_forever()
        except StopError:
            pass
```

When the task catches `StopIteration` and resolves itself, the callback raises `StopError` from within the loop. The loop stops and the call stack is unwound to `run_until_complete`. Our program is finished.

## Conclusion

Increasingly often, modern programs are I/O-bound instead of CPU-bound. For such programs, Python threads are the worst of both worlds: the global interpreter lock prevents them from actually executing computations in parallel, and preemptive switching makes them prone to races. Async is often the right pattern. But as callback-based async code grows, it tends to become a dishevelled mess. Coroutines are a tidy alternative. They factor naturally into subroutines, with sane exception handling and stack traces.

If we squint so that the `yield from` statements blur, a coroutine looks like a thread doing traditional blocking I/O. We can even coordinate coroutines with classic patterns from multi-threaded programming. There is no need for reinvention. Thus, compared to callbacks, coroutines are an inviting idiom to the coder experienced with multithreading.

But when we open our eyes and focus on the `yield from` statements, we see they mark points when the coroutine cedes control and allows others to run. Unlike threads, coroutines display where our code can be interrupted and where it cannot. In his illuminating essay "Unyielding"[^4], Glyph Lefkowitz writes, "Threads make local reasoning difficult, and local reasoning is perhaps the most important thing in software development." Explicitly yielding, however, makes it possible to "understand the behavior (and thereby, the correctness) of a routine by examining the routine itself rather than examining the entire system."

This chapter was written during a renaissance in the history of Python and async. Generator-based coroutines, whose devising you have just learned, were released in the "asyncio" module with Python 3.4 in March 2014. In September 2015, Python 3.5 was released with coroutines built in to the language itself. These native coroutinesare declared with the new syntax "async def", and instead of "yield from", they use the new "await" keyword to delegate to a coroutine or wait for a Future.

Despite these advances, the core ideas remain. Python's new native coroutines will be syntactically distinct from generators but work very similarly; indeed, they will share an implementation within the Python interpreter. Task, Future, and the event loop will continue to play their roles in asyncio.

Now that you know how asyncio coroutines work, you can largely forget the details. The machinery is tucked behind a dapper interface. But your grasp of the fundamentals empowers you to code correctly and efficiently in modern async environments.

[^4]: [https://glyph.twistedmatrix.com/2014/02/unyielding.html](https://glyph.twistedmatrix.com/2014/02/unyielding.html)

[^5]: [https://docs.python.org/3/library/queue.html](https://docs.python.org/3/library/queue.html)

[^6]: [https://docs.python.org/3/library/asyncio-sync.html](https://docs.python.org/3/library/asyncio-sync.html)

[^7]: For a complex solution to this problem, see [http://www.tornadoweb.org/en/stable/stack_context.html](http://www.tornadoweb.org/en/stable/stack_context.html)

[^8]: [http://www.kegel.com/c10k.html](http://www.kegel.com/c10k.html)

[^9]: The actual `asyncio.Queue` implementation uses an `asyncio.Event` in place of the Future shown here. The difference is an Event can be reset, whereas a Future cannot transition from resolved back to pending.

[^10]: The `@asyncio.coroutine` decorator is not magical. In fact, if it decorates a generator function and the `PYTHONASYNCIODEBUG` environment variable is not set, the decorator does practically nothing. It just sets an attribute, `_is_coroutine`, for the convenience of other parts of the framework. It is possible to use asyncio with bare generators not decorated with `@asyncio.coroutine` at all.

<latex>
[^11]: Jesse listed indications and contraindications for using async in "What Is Async, How Does It Work, And When Should I Use It?", available at pyvideo.org. 
[^bayer]: Mike Bayer compared the throughput of asyncio and multithreading for different workloads in his "Asynchronous Python and Databases": http://techspot.zzzeek.org/2015/02/15/asynchronous-python-and-databases/
</latex>
<markdown>
[^11]: Jesse listed indications and contraindications for using async in ["What Is Async, How Does It Work, And When Should I Use It?":](http://pyvideo.org/video/2565/what-is-async-how-does-it-work-and-when-should). Mike Bayer compared the throughput of asyncio and multithreading for different workloads in ["Asynchronous Python and Databases":](http://techspot.zzzeek.org/2015/02/15/asynchronous-python-and-databases/)
</markdown>

[^12]: This future has many deficiencies. For example, once this future is resolved, a coroutine that yields it should resume immediately instead of pausing, but with our code it does not. See asyncio's Future class for a complete implementation.

[^13]: In fact, this is exactly how "yield from" works in CPython. A function increments its instruction pointer before executing each statement. But after the outer generator executes "yield from", it subtracts 1 from its instruction pointer to keep itself pinned at the "yield from" statement. Then it yields to *its* caller. The cycle repeats until the inner generator throws `StopIteration`, at which point the outer generator finally allows itself to advance to the next instruction.

[^14]: Python's global interpreter lock prohibits running Python code in parallel in one process anyway. Parallelizing CPU-bound algorithms in Python requires multiple processes, or writing the parallel portions of the code in C. But that is a topic for another day.

[^15]: Even calls to `send` can block, if the recipient is slow to acknowledge outstanding messages and the system's buffer of outgoing data is full.

<markdown>
[^16]: Guido introduced the standard asyncio library, called "Tulip" then, at [PyCon 2013](http://pyvideo.org/video/1667/keynote).
</markdown>
<latex>
[^16]: Guido introduced the standard asyncio library, called "Tulip" then, at PyCon 2013.
</latex>

[^17]: Python 3.5's built-in coroutines are described in [PEP 492](https://www.python.org/dev/peps/pep-0492/), "Coroutines with async and await syntax."
