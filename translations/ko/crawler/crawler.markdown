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

이 함수는 하나의 소켓 연산과 다음 연산 사이에 어떤 상태를 기억할까요? 소켓, URL, 그리고 누적되는 `response`를 가지고 있습니다. 스레드에서 실행되는 함수는 프로그래밍 언어의 기본 기능을 사용하여 이러한 임시 상태를 스택의 로컬 변수에 저장합니다. 함수는 또한 "연속성(continuation)"을 가지고 있습니다&mdash;즉, I/O가 완료된 후 실행할 계획인 코드입니다. 런타임은 스레드의 명령어 포인터를 저장함으로써 연속성을 기억합니다. I/O 후에 이러한 로컬 변수와 연속성을 복원하는 것에 대해 생각할 필요가 없습니다. 이는 언어에 내장되어 있습니다.

하지만 콜백 기반 비동기 프레임워크에서는 이러한 언어 기능들이 도움이 되지 않습니다. I/O를 기다리는 동안, 함수는 명시적으로 상태를 저장해야 합니다. 왜냐하면 I/O가 완료되기 전에 함수가 반환되어 스택 프레임을 잃기 때문입니다. 로컬 변수 대신, 우리의 콜백 기반 예제는 Fetcher 인스턴스인 `self`의 속성으로 `sock`과 `response`를 저장합니다. 명령어 포인터 대신, `connected`와 `read_response` 콜백을 등록하여 연속성을 저장합니다. 애플리케이션의 기능이 증가할수록, 콜백 간에 수동으로 저장해야 하는 상태의 복잡성도 증가합니다. 이러한 번거로운 부기 작업은 코더를 편두통에 취약하게 만듭니다.

더 나쁜 것은, 콜백이 체인의 다음 콜백을 스케줄링하기 전에 예외를 발생시키면 어떻게 될까요? `parse_links` 메서드를 제대로 작성하지 못해서 HTML을 파싱하는 중에 예외를 발생시킨다고 해봅시다:

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

스택 트레이스는 이벤트 루프가 콜백을 실행하고 있었다는 것만 보여줍니다. 우리는 오류로 이어진 것이 무엇인지 기억하지 못합니다. 체인이 양쪽 끝에서 끊어졌습니다: 우리가 어디로 가고 있었는지, 어디서 왔는지를 잊어버렸습니다. 이러한 컨텍스트 손실을 "스택 찢기(stack ripping)"라고 하며, 많은 경우에 조사자를 혼란스럽게 만듭니다. 스택 찢기는 또한 "try / except" 블록이 함수 호출과 그 하위 트리를 감싸는 방식처럼 콜백 체인에 대한 예외 핸들러를 설치하는 것을 방해합니다.[^7]

따라서 멀티스레딩과 비동기의 상대적 효율성에 대한 긴 논쟁과는 별개로, 어느 것이 더 오류가 발생하기 쉬운지에 대한 또 다른 논쟁이 있습니다: 스레드는 동기화를 실수하면 데이터 레이스에 취약하지만, 콜백은 스택 찢기로 인해 디버그하기 까다롭습니다. 

## Coroutines

우리는 약속으로 여러분을 유혹합니다. 콜백의 효율성과 멀티스레드 프로그래밍의 고전적인 아름다움을 결합한 비동기 코드를 작성하는 것이 가능합니다. 이러한 조합은 "코루틴(coroutines)"이라고 하는 패턴으로 달성됩니다. Python 3.4의 표준 asyncio 라이브러리와 "aiohttp"라는 패키지를 사용하면, 코루틴에서 URL을 가져오는 것이 매우 직접적입니다[^10]:

```python
    @asyncio.coroutine
    def fetch(self, url):
        response = yield from self.session.get(url)
        body = yield from response.read()
```

또한 확장 가능합니다. 스레드당 50k의 메모리와 스레드에 대한 운영 체제의 하드 제한과 비교하여, Python 코루틴은 Jesse의 시스템에서 겨우 3k의 메모리만 사용합니다. Python은 수십만 개의 코루틴을 쉽게 시작할 수 있습니다.

컴퓨터 과학의 초창기부터 시작된 코루틴의 개념은 간단합니다: 일시 중지되고 재개될 수 있는 서브루틴입니다. 스레드는 운영 체제에 의해 선점적으로 멀티태스킹되는 반면, 코루틴은 협력적으로 멀티태스킹합니다: 언제 일시 중지할지, 다음에 실행할 코루틴을 선택합니다.

코루틴에는 많은 구현이 있으며, Python에서도 여러 가지가 있습니다. Python 3.4의 표준 "asyncio" 라이브러리의 코루틴은 제너레이터, Future 클래스, "yield from" 문을 기반으로 구축됩니다. Python 3.5부터 코루틴은 언어 자체의 네이티브 기능이 되었습니다[^17]; 하지만 기존 언어 기능을 사용하여 Python 3.4에서 처음 구현된 코루틴을 이해하는 것이 Python 3.5의 네이티브 코루틴을 다루는 기반입니다.

Python 3.4의 제너레이터 기반 코루틴을 설명하기 위해, 제너레이터와 asyncio에서 코루틴으로 사용되는 방법에 대한 설명을 하겠습니다. 우리가 즐겁게 작성한 만큼 여러분도 즐겁게 읽기를 바랍니다. 제너레이터 기반 코루틴을 설명한 후에는, 비동기 웹 크롤러에서 이를 사용하겠습니다.

## Python 제너레이터의 작동 방식

Python 제너레이터를 이해하기 전에, 일반적인 Python 함수가 어떻게 작동하는지 이해해야 합니다. 일반적으로 Python 함수가 서브루틴을 호출하면, 서브루틴은 반환되거나 예외를 발생시킬 때까지 제어권을 유지합니다. 그러면 제어권이 호출자에게 돌아갑니다:

```python
>>> def foo():
...     bar()
...
>>> def bar():
...     pass
```

표준 Python 인터프리터는 C로 작성되었습니다. Python 함수를 실행하는 C 함수는 아름답게도 `PyEval_EvalFrameEx`라고 불립니다. 이 함수는 Python 스택 프레임 객체를 받아서 프레임의 컨텍스트에서 Python 바이트코드를 평가합니다. 다음은 `foo`의 바이트코드입니다:

```python
>>> import dis
>>> dis.dis(foo)
  2           0 LOAD_GLOBAL              0 (bar)
              3 CALL_FUNCTION            0 (0 positional, 0 keyword pair)
              6 POP_TOP
              7 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

`foo` 함수는 `bar`를 스택에 로드하고 호출한 다음, 스택에서 반환값을 팝하고, `None`을 스택에 로드한 후 `None`을 반환합니다.

`PyEval_EvalFrameEx`가 `CALL_FUNCTION` 바이트코드를 만나면, 새로운 Python 스택 프레임을 생성하고 재귀합니다: 즉, `bar`를 실행하는 데 사용되는 새 프레임으로 `PyEval_EvalFrameEx`를 재귀적으로 호출합니다.

Python 스택 프레임이 힙 메모리에 할당된다는 것을 이해하는 것이 중요합니다! Python 인터프리터는 일반적인 C 프로그램이므로, 그 스택 프레임은 일반적인 스택 프레임입니다. 하지만 인터프리터가 조작하는 *Python* 스택 프레임은 힙에 있습니다. 다른 놀라운 점들 중에서, 이것은 Python 스택 프레임이 함수 호출보다 오래 살 수 있다는 것을 의미합니다. 이를 대화형으로 보려면, `bar` 내에서 현재 프레임을 저장해보세요:

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

이제 Python 제너레이터를 위한 무대가 준비되었습니다. 제너레이터는 동일한 구성 요소&mdash;코드 객체와 스택 프레임&mdash;을 사용하여 놀라운 효과를 만들어냅니다.

다음은 제너레이터 함수입니다:

```python
>>> def gen_fn():
...     result = yield 1
...     print('result of yield: {}'.format(result))
...     result2 = yield 2
...     print('result of 2nd yield: {}'.format(result2))
...     return 'done'
...     
```

Python이 `gen_fn`을 바이트코드로 컴파일할 때, `yield` 문을 보고 `gen_fn`이 일반 함수가 아닌 제너레이터 함수임을 알게 됩니다. 이 사실을 기억하기 위해 플래그를 설정합니다:

```python
>>> # The generator flag is bit position 5.
>>> generator_bit = 1 << 5
>>> bool(gen_fn.__code__.co_flags & generator_bit)
True
```

제너레이터 함수를 호출하면, Python은 제너레이터 플래그를 보고 실제로 함수를 실행하지 않습니다. 대신 제너레이터를 생성합니다:

```python
>>> gen = gen_fn()
>>> type(gen)
<class 'generator'>
```

Python 제너레이터는 스택 프레임과 코드에 대한 참조(즉, `gen_fn`의 본문)를 캡슐화합니다:

```python
>>> gen.gi_code.co_name
'gen_fn'
```

`gen_fn` 호출로부터 생성된 모든 제너레이터는 동일한 코드를 가리킵니다. 하지만 각각은 자체 스택 프레임을 가집니다. 이 스택 프레임은 실제 스택에 있지 않고, 사용을 기다리며 힙 메모리에 위치합니다:

\aosafigure[240pt]{crawler-images/generator.png}{Generators}{500l.crawler.generators}

프레임은 "마지막 명령어" 포인터를 가지며, 이는 가장 최근에 실행한 명령어입니다. 처음에는 마지막 명령어 포인터가 -1로, 제너레이터가 아직 시작되지 않았음을 의미합니다:

```python
>>> gen.gi_frame.f_lasti
-1
```

`send`를 호출하면, 제너레이터는 첫 번째 `yield`에 도달하고 일시 중지됩니다. `send`의 반환값은 1인데, 이는 `gen`이 `yield` 표현식에 전달하는 값이기 때문입니다:

```python
>>> gen.send(None)
1
```

제너레이터의 명령어 포인터는 이제 시작점에서 3 바이트코드 떨어진 곳에 있으며, 컴파일된 Python의 56바이트 중간 지점에 있습니다:

```python
>>> gen.gi_frame.f_lasti
3
>>> len(gen.gi_code.co_code)
56
```

제너레이터는 언제든지, 어떤 함수에서든 재개될 수 있습니다. 왜냐하면 스택 프레임이 실제로 스택에 있지 않고 힙에 있기 때문입니다. 호출 계층에서의 위치가 고정되지 않으며, 일반 함수가 따라야 하는 선입후출(first-in, last-out) 실행 순서를 따를 필요가 없습니다. 구름처럼 자유롭게 떠다니며 해방되어 있습니다.

제너레이터에 "hello" 값을 보내면 이것이 `yield` 표현식의 결과가 되고, 제너레이터는 2를 yield할 때까지 계속됩니다:

```python
>>> gen.send('hello')
result of yield: hello
2
```

이제 스택 프레임에는 로컬 변수 `result`가 포함되어 있습니다:

```python
>>> gen.gi_frame.f_locals
{'result': 'hello'}
```

`gen_fn`에서 생성된 다른 제너레이터들은 자체 스택 프레임과 로컬 변수를 가집니다.

`send`를 다시 호출하면, 제너레이터는 두 번째 `yield`부터 계속되고, 특별한 `StopIteration` 예외를 발생시키며 완료됩니다:

```python
>>> gen.send('goodbye')
result of 2nd yield: goodbye
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration: done
```

예외는 값을 가지며, 이는 제너레이터의 반환값입니다: 문자열 `"done"`입니다.

## Building Coroutines With Generators

따라서 제너레이터는 일시 중지될 수 있고, 값과 함께 재개될 수 있으며, 반환값을 가집니다. 스파게티 콜백 없이 비동기 프로그래밍 모델을 구축할 수 있는 좋은 기본 요소처럼 들립니다! 우리는 "코루틴"을 만들고자 합니다: 프로그램의 다른 루틴들과 협력적으로 스케줄링되는 루틴입니다. 우리의 코루틴은 Python의 표준 "asyncio" 라이브러리에 있는 것들의 단순화된 버전이 될 것입니다. asyncio에서와 마찬가지로, 제너레이터, 퓨처, "yield from" 문을 사용할 것입니다.

먼저 코루틴이 기다리고 있는 미래 결과를 나타내는 방법이 필요합니다. 단순화된 버전입니다:

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

퓨처는 처음에 "대기(pending)" 상태입니다. `set_result` 호출에 의해 "해결(resolved)"됩니다.[^12]

퓨처와 코루틴을 사용하도록 가져오기 도구를 적응시켜봅시다. 우리는 콜백으로 `fetch`를 작성했습니다:

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

`fetch` 메서드는 소켓 연결을 시작한 다음, 소켓이 준비되었을 때 실행될 콜백 `connected`를 등록합니다. 이제 이 두 단계를 하나의 코루틴으로 결합할 수 있습니다:

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

이제 `fetch`는 `yield` 문을 포함하고 있기 때문에 일반 함수가 아닌 제너레이터 함수입니다. 대기 중인 퓨처를 생성한 다음, 소켓이 준비될 때까지 `fetch`를 일시 중지하기 위해 이를 yield합니다. 내부 함수 `on_connected`가 퓨처를 해결합니다.

하지만 퓨처가 해결되면, 무엇이 제너레이터를 재개할까요? 코루틴 *드라이버*가 필요합니다. 이를 "작업(task)"이라고 부릅시다:

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

작업은 `None`을 보내 `fetch` 제너레이터를 시작합니다. 그러면 `fetch`는 퓨처를 yield할 때까지 실행되고, 작업은 이를 `next_future`로 캡처합니다. 소켓이 연결되면, 이벤트 루프가 콜백 `on_connected`를 실행하고, 이는 퓨처를 해결하며, 이는 `step`을 호출하여 `fetch`를 재개합니다.

## `yield from`으로 코루틴 분해하기

소켓이 연결되면, HTTP GET 요청을 보내고 서버 응답을 읽습니다. 이러한 단계들은 더 이상 콜백들 사이에 흩어질 필요가 없습니다; 같은 제너레이터 함수로 모을 수 있습니다:

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

소켓에서 전체 메시지를 읽는 이 코드는 일반적으로 유용해 보입니다. 이를 `fetch`에서 서브루틴으로 어떻게 분해할 수 있을까요? 이제 Python 3의 유명한 `yield from`이 등장합니다. 이는 하나의 제너레이터가 다른 제너레이터에게 *위임*할 수 있게 해줍니다.

이를 보기 위해, 간단한 제너레이터 예제로 돌아가봅시다:

```python
>>> def gen_fn():
...     result = yield 1
...     print('result of yield: {}'.format(result))
...     result2 = yield 2
...     print('result of 2nd yield: {}'.format(result2))
...     return 'done'
...
```

다른 제너레이터에서 이 제너레이터를 호출하려면, `yield from`으로 위임합니다:

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

`caller` 제너레이터는 위임하고 있는 제너레이터인 `gen`처럼 동작합니다:

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

`caller`가 `gen`으로부터 yield하는 동안, `caller`는 전진하지 않습니다. 내부 제너레이터 `gen`이 하나의 `yield` 문에서 다음 문으로 전진하는 동안에도, 명령어 포인터는 `yield from` 문의 위치인 15에 그대로 남아 있음을 주목하세요.[^13] `caller` 외부의 관점에서는, yield되는 값이 `caller`에서 온 것인지 위임받은 제너레이터에서 온 것인지 구별할 수 없습니다. 그리고 `gen` 내부에서는, 값이 `caller`에서 보내진 것인지 외부에서 보내진 것인지 구별할 수 없습니다. `yield from` 문은 마찰 없는 채널로, `gen`이 완료될 때까지 값들이 `gen` 안팎으로 흐르게 합니다.

코루틴은 `yield from`으로 서브 코루틴에게 작업을 위임하고 작업의 결과를 받을 수 있습니다. 위에서 `caller`가 "return value of yield-from: done"을 출력했음을 주목하세요. `gen`이 완료되었을 때, 그 반환값이 `caller`의 `yield from` 문의 값이 되었습니다:

```python
    rv = yield from gen
```

앞서 콜백 기반 비동기 프로그래밍을 비판했을 때, 가장 강력한 불만은 "스택 찢기"에 관한 것이었습니다: 콜백이 예외를 발생시킬 때 스택 트레이스는 일반적으로 쓸모가 없습니다. 이벤트 루프가 콜백을 실행하고 있었다는 것만 보여줄 뿐, *왜* 그랬는지는 알 수 없습니다. 코루틴은 어떨까요?

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

이는 훨씬 더 유용합니다! 스택 트레이스는 오류를 발생시킬 때 `caller_fn`이 `gen_fn`에게 위임하고 있었음을 보여줍니다. 더욱 안심이 되는 것은, 일반 서브루틴과 마찬가지로 서브 코루틴 호출을 예외 핸들러로 감쌀 수 있다는 것입니다:

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

따라서 우리는 일반적인 서브루틴과 마찬가지로 서브 코루틴으로 로직을 분해할 수 있습니다. 가져오기 도구에서 유용한 서브 코루틴들을 분해해봅시다. 하나의 청크를 받기 위한 `read` 코루틴을 작성합니다:

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

`read`를 기반으로 전체 메시지를 받는 `read_all` 코루틴을 구축합니다:

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

적절히 눈을 가늘게 뜨면 `yield from` 문들이 사라지고 이들이 블로킹 I/O를 수행하는 기존의 함수들처럼 보입니다. 하지만 실제로는 `read`와 `read_all`은 코루틴입니다. `read`로부터 yield하면 I/O가 완료될 때까지 `read_all`을 일시 중지시킵니다. `read_all`이 일시 중지되어 있는 동안, asyncio의 이벤트 루프는 다른 작업을 수행하고 다른 I/O 이벤트를 기다립니다; 해당 이벤트가 준비되면 다음 루프 틱에서 `read`의 결과와 함께 `read_all`이 재개됩니다.

스택의 루트에서 `fetch`는 `read_all`을 호출합니다:

```python
class Fetcher:
    def fetch(self):
		 # ... connection logic from above, then:
        sock.send(request.encode('ascii'))
        self.response = yield from read_all(sock)
```

놀랍게도, Task 클래스는 수정이 필요하지 않습니다. 이전과 마찬가지로 외부 `fetch` 코루틴을 동일하게 구동합니다:

```python
Task(fetcher.fetch())
loop()
```

`read`가 퓨처를 yield할 때, 작업은 `yield from` 문들의 채널을 통해 이를 받습니다. 마치 퓨처가 `fetch`에서 직접 yield된 것처럼 정확히 동작합니다. 루프가 퓨처를 해결하면, 작업은 그 결과를 `fetch`에 보내고, 그 값은 `read`에 의해 받아집니다. 마치 작업이 `read`를 직접 구동하는 것처럼 정확히 동작합니다:

\aosafigure[240pt]{crawler-images/yield-from.png}{Yield From}{500l.crawler.yieldfrom}

코루틴 구현을 완벽하게 하기 위해, 하나의 흠을 다듬어봅시다: 우리 코드는 퓨처를 기다릴 때 `yield`를 사용하지만, 서브 코루틴에게 위임할 때는 `yield from`을 사용합니다. 코루틴이 일시 중지될 때마다 `yield from`을 사용한다면 더 세련될 것입니다. 그러면 코루틴은 기다리는 것의 유형에 대해 신경 쓸 필요가 없습니다.

Python에서 제너레이터와 이터레이터 간의 깊은 대응성을 활용합니다. 호출자에게는 제너레이터를 전진시키는 것이 이터레이터를 전진시키는 것과 같습니다. 따라서 특별한 메서드를 구현하여 Future 클래스를 이터러블로 만듭니다:

```python
    # Method on Future class.
    def __iter__(self):
        # Tell Task to resume me here.
        yield self
        return self.result
```

퓨처의 `__iter__` 메서드는 퓨처 자체를 yield하는 코루틴입니다. 이제 다음과 같은 코드를:

```python
# f is a Future.
yield f
```

...다음과 같이 바꾸면:

```python
# f is a Future.
yield from f
```

...결과는 동일합니다! 구동하는 Task는 `send` 호출로부터 퓨처를 받고, 퓨처가 해결되면 새로운 결과를 코루틴에 다시 보냅니다.

모든 곳에서 `yield from`을 사용하는 장점은 무엇일까요? 퓨처를 기다릴 때 `yield`를 사용하고 서브 코루틴에게 위임할 때 `yield from`을 사용하는 것보다 왜 더 나을까요? 이제 메서드가 호출자에게 영향을 주지 않고 자유롭게 구현을 변경할 수 있기 때문에 더 좋습니다: 값으로 *해결*될 퓨처를 반환하는 일반 메서드일 수도 있고, `yield from` 문을 포함하여 값을 *반환*하는 코루틴일 수도 있습니다. 어느 경우든, 호출자는 결과를 기다리기 위해 메서드를 `yield from`하기만 하면 됩니다.

친애하는 독자 여러분, asyncio의 코루틴에 대한 즐거운 설명의 끝에 도달했습니다. 우리는 제너레이터의 내부 메커니즘을 살펴보고, 퓨처와 작업의 구현을 스케치했습니다. asyncio가 어떻게 두 세계의 장점을 모두 얻는지 개요를 설명했습니다: 스레드보다 효율적이고 콜백보다 읽기 쉬운 동시 I/O입니다. 물론, 실제 asyncio는 우리의 스케치보다 훨씬 더 정교합니다. 실제 프레임워크는 제로 카피 I/O, 공정한 스케줄링, 예외 처리, 그리고 풍부한 다른 기능들을 다룹니다.

asyncio 사용자에게는 코루틴으로 코딩하는 것이 여기서 본 것보다 훨씬 간단합니다. 위의 코드에서 우리는 코루틴을 원리부터 구현했기 때문에, 콜백, 작업, 퓨처를 보았습니다. 논블로킹 소켓과 `select` 호출까지 보았습니다. 하지만 실제로 asyncio로 애플리케이션을 구축할 때는, 이 중 어느 것도 여러분의 코드에 나타나지 않습니다. 약속했던 대로, 이제 URL을 우아하게 가져올 수 있습니다:

```python
    @asyncio.coroutine
    def fetch(self, url):
        response = yield from self.session.get(url)
        body = yield from response.read()
```

이 설명에 만족하며, 원래 과제로 돌아갑니다: asyncio를 사용하여 비동기 웹 크롤러를 작성하는 것입니다.

## 코루틴 조정하기

우리는 크롤러가 어떻게 작동하기를 원하는지 설명하는 것으로 시작했습니다. 이제 asyncio 코루틴으로 이를 구현할 시간입니다.

우리 크롤러는 첫 번째 페이지를 가져와서 링크를 파싱하고 큐에 추가할 것입니다. 그 후 웹사이트 전체에 퍼져서 페이지들을 동시에 가져옵니다. 하지만 클라이언트와 서버의 부하를 제한하기 위해, 실행할 워커의 최대 개수를 원하며, 그 이상은 안 됩니다. 워커가 페이지 가져오기를 완료할 때마다, 큐에서 다음 링크를 즉시 가져와야 합니다. 할 일이 부족한 기간을 거쳐갈 것이므로, 일부 워커들은 일시 중지해야 합니다. 하지만 워커가 새로운 링크가 풍부한 페이지를 만나면, 큐가 갑자기 커지고 일시 중지된 모든 워커들이 깨어나서 작업을 시작해야 합니다. 마지막으로, 우리 프로그램은 작업이 완료되면 종료해야 합니다.

워커들이 스레드라고 상상해보세요. 크롤러의 알고리즘을 어떻게 표현할까요? Python 표준 라이브러리의 동기화된 큐[^5]를 사용할 수 있습니다. 큐에 항목이 들어갈 때마다, 큐는 "작업" 카운트를 증가시킵니다. 워커 스레드들은 항목에 대한 작업을 완료한 후 `task_done`을 호출합니다. 메인 스레드는 큐에 넣은 각 항목이 `task_done` 호출과 일치할 때까지 `Queue.join`에서 블록하다가, 그 후 종료합니다.

코루틴은 asyncio 큐와 정확히 동일한 패턴을 사용합니다! 먼저 이를 임포트합니다[^6]:

```python
try:
    from asyncio import JoinableQueue as Queue
except ImportError:
    # In Python 3.5, asyncio.JoinableQueue is
    # merged into Queue.
    from asyncio import Queue
```

워커들의 공유 상태를 크롤러 클래스에 수집하고, `crawl` 메서드에 주요 로직을 작성합니다. `crawl`을 코루틴에서 시작하고 `crawl`이 완료될 때까지 asyncio의 이벤트 루프를 실행합니다:

```python
loop = asyncio.get_event_loop()

crawler = crawling.Crawler('http://xkcd.com',
                           max_redirect=10)

loop.run_until_complete(crawler.crawl())
```

크롤러는 루트 URL과 `max_redirect`로 시작합니다. `max_redirect`는 어떤 하나의 URL을 가져오기 위해 따라갈 의향이 있는 리디렉션 횟수입니다. 큐에 `(URL, max_redirect)` 쌍을 넣습니다. (왜 그런지는 계속 지켜보세요.)

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

큐의 미완성 작업 수는 이제 하나입니다. 메인 스크립트로 돌아가서, 이벤트 루프와 `crawl` 메서드를 시작합니다:

```python
loop.run_until_complete(crawler.crawl())
```

`crawl` 코루틴은 워커들을 시작시킵니다. 메인 스레드와 같습니다: 워커들이 백그라운드에서 실행되는 동안 모든 작업이 완료될 때까지 `join`에서 블록됩니다.

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

워커들이 스레드였다면 모두 한 번에 시작하고 싶지 않을 수도 있습니다. 필요하다고 확실해질 때까지 비싼 스레드들을 생성하는 것을 피하기 위해, 스레드 풀은 일반적으로 요구에 따라 증가합니다. 하지만 코루틴은 저렴하므로, 허용된 최대 개수를 단순히 시작합니다.

크롤러를 종료하는 방법을 주목하는 것은 흥미롭습니다. `join` 퓨처가 해결되면, 워커 작업들은 살아있지만 일시 중지됩니다: 더 많은 URL을 기다리지만 아무것도 오지 않습니다. 따라서 메인 코루틴이 종료하기 전에 이들을 취소합니다. 그렇지 않으면, Python 인터프리터가 종료되고 모든 객체의 소멸자를 호출할 때, 살아있는 작업들이 외칩니다:

```
ERROR:asyncio:Task was destroyed but it is pending!
```

그리고 `cancel`은 어떻게 작동할까요? 제너레이터에는 아직 보여주지 않은 기능이 있습니다. 외부에서 제너레이터에 예외를 던질 수 있습니다:

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

제너레이터는 `throw`에 의해 재개되지만, 이제 예외를 발생시키고 있습니다. 제너레이터의 호출 스택에 있는 어떤 코드도 이를 잡지 않으면, 예외가 맨 위로 버블링됩니다. 따라서 작업의 코루틴을 취소하려면:

```python
    # Method of Task class.
    def cancel(self):
        self.coro.throw(CancelledError)
```

제너레이터가 일시 중지된 곳이 어디든, 어떤 `yield from` 문에서든, 재개되어 예외를 던집니다. 작업의 `step` 메서드에서 취소를 처리합니다:

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

이제 작업은 취소되었음을 알므로, 소멸될 때 빛의 죽음에 맞서 격노하지 않습니다.

`crawl`이 워커들을 취소한 후, 종료합니다. 이벤트 루프는 코루틴이 완료되었음을 보고(나중에 어떻게 하는지 보겠습니다), 마찬가지로 종료합니다:

```python
loop.run_until_complete(crawler.crawl())
```

`crawl` 메서드는 메인 코루틴이 해야 할 모든 것을 포함합니다. 큐에서 URL을 가져와서 페치하고 새 링크를 파싱하는 것은 워커 코루틴들입니다. 각 워커는 `work` 코루틴을 독립적으로 실행합니다:

```python
    @asyncio.coroutine
    def work(self):
        while True:
            url, max_redirect = yield from self.q.get()

            # Download page and add new links to self.q.
            yield from self.fetch(url, max_redirect)
            self.q.task_done()
```

Python은 이 코드가 `yield from` 문을 포함하는 것을 보고, 제너레이터 함수로 컴파일합니다. 따라서 `crawl`에서 메인 코루틴이 `self.work`를 열 번 호출할 때, 실제로 이 메서드를 실행하지 않습니다: 이 코드에 대한 참조를 가진 열 개의 제너레이터 객체만 생성합니다. 각각을 Task로 감쌉니다. Task는 제너레이터가 yield하는 각 퓨처를 받고, 퓨처가 해결될 때 각 퓨처의 결과와 함께 `send`를 호출하여 제너레이터를 구동합니다. 제너레이터들은 각자의 스택 프레임을 가지므로, 별도의 로컬 변수와 명령어 포인터로 독립적으로 실행됩니다.

워커는 큐를 통해 동료들과 조정합니다. 다음으로 새 URL을 기다립니다:

```python
    url, max_redirect = yield from self.q.get()
```

큐의 `get` 메서드 자체가 코루틴입니다: 누군가가 큐에 항목을 넣을 때까지 일시 중지하다가, 재개되어 항목을 반환합니다.

참고로, 이것이 크롤링의 끝에서 메인 코루틴이 워커를 취소할 때 워커가 일시 중지되는 곳입니다. 코루틴의 관점에서, 루프를 도는 마지막 여행은 `yield from`이 `CancelledError`를 발생시킬 때 끝납니다.

워커가 페이지를 가져오면 링크를 파싱하고 새로운 것들을 큐에 넣은 다음, 카운터를 감소시키기 위해 `task_done`을 호출합니다. 결국, 워커는 URL이 모두 이미 가져와진 페이지를 가져오게 되고, 큐에도 남은 작업이 없습니다. 따라서 이 워커의 `task_done` 호출이 카운터를 0으로 감소시킵니다. 그러면 큐의 `join` 메서드를 기다리고 있던 `crawl`이 일시 중지 해제되고 완료됩니다.

큐의 항목이 다음과 같은 쌍인 이유를 설명하겠다고 약속했습니다:

```python
# URL to fetch, and the number of redirects left.
('http://xkcd.com/353', 10)
```

새 URL은 10개의 리디렉션이 남아있습니다. 이 특정 URL을 가져오면 끝에 슬래시가 있는 새 위치로 리디렉션됩니다. 남은 리디렉션 수를 감소시키고, 다음 위치를 큐에 넣습니다:

```python
# URL with a trailing slash. Nine redirects left.
('http://xkcd.com/353/', 9)
```

우리가 사용하는 `aiohttp` 패키지는 기본적으로 리디렉션을 따라가서 최종 응답을 줄 것입니다. 하지만 우리는 그렇게 하지 않도록 설정하고, 크롤러에서 리디렉션을 처리합니다. 그래서 같은 목적지로 이어지는 리디렉션 경로들을 통합할 수 있습니다: 이 URL을 이미 보았다면, `self.seen_urls`에 있고 다른 진입점에서 이미 이 경로를 시작했다는 뜻입니다:

\aosafigure[240pt]{crawler-images/redirects.png}{Redirects}{500l.crawler.redirects}

크롤러가 "foo"를 가져와서 "baz"로 리디렉션되는 것을 보면, "baz"를 큐와 `seen_urls`에 추가합니다. 다음으로 가져온 페이지가 "bar"이고, 이것도 "baz"로 리디렉션된다면, 가져오기 도구는 "baz"를 다시 큐에 넣지 않습니다. 응답이 리디렉션이 아닌 페이지라면, `fetch`는 링크를 파싱하고
새로운 것들을 큐에 넣습니다.

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

이것이 멀티스레드 코드였다면, 레이스 컨디션으로 엉망이 될 것입니다. 예를 들어, 워커가 링크가 `seen_urls`에 있는지 확인하고, 없으면 워커가 이를 큐에 넣고 `seen_urls`에 추가합니다. 두 연산 사이에 중단된다면, 다른 워커가 다른 페이지에서 같은 링크를 파싱하고, 마찬가지로 `seen_urls`에 없다는 것을 관찰하고, 또한 큐에 추가할 수도 있습니다. 이제 같은 링크가 큐에 두 번 있게 되어 (최선의 경우) 중복 작업과 잘못된 통계를 초래합니다.

하지만 코루틴은 `yield from` 문에서만 중단에 취약합니다. 이는 코루틴 코드가 멀티스레드 코드보다 레이스에 훨씬 덜 취약하게 만드는 핵심 차이점입니다: 멀티스레드 코드는 락을 잡아서 명시적으로 임계 섹션에 들어가야 하며, 그렇지 않으면 중단 가능합니다. Python 코루틴은 기본적으로 중단되지 않으며, 명시적으로 yield할 때만 제어권을 양보합니다.

콜백 기반 프로그램에서 가졌던 것과 같은 페처 클래스는 더 이상 필요하지 않습니다. 그 클래스는 콜백의 결함에 대한 해결책이었습니다: I/O를 기다리는 동안 상태를 저장할 곳이 필요한데, 로컬 변수는 호출 간에 보존되지 않기 때문입니다. 하지만 `fetch` 코루틴은 일반 함수와 같이 로컬 변수에 상태를 저장할 수 있으므로, 더 이상 클래스가 필요하지 않습니다.

`fetch`가 서버 응답 처리를 완료하면 호출자인 `work`에게 반환됩니다. `work` 메서드는 큐에서 `task_done`을 호출하고 가져올 다음 URL을 큐에서 가져옵니다.

`fetch`가 큐에 새 링크를 넣으면 미완성 작업 수가 증가하고 `q.join`을 기다리고 있는 메인 코루틴을 일시 중지 상태로 유지합니다. 하지만 보지 않은 링크가 없고 이것이 큐의 마지막 URL이었다면, `work`가 `task_done`을 호출할 때 미완성 작업 수가 0으로 떨어집니다. 이 이벤트는 `join`의 일시 중지를 해제하고 메인 코루틴을 완료합니다.

워커들과 메인 코루틴을 조정하는 큐 코드는 다음과 같습니다[^9]:

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

메인 코루틴 `crawl`은 `join`으로부터 yield합니다. 따라서 마지막 워커가 미완성 작업 수를 0으로 감소시키면, `crawl`에게 재개하고 완료하라는 신호를 보냅니다.

여행이 거의 끝났습니다. 우리 프로그램은 `crawl` 호출로 시작되었습니다:

```python
loop.run_until_complete(self.crawler.crawl())
```

프로그램은 어떻게 끝날까요? `crawl`은 제너레이터 함수이므로, 이를 호출하면 제너레이터를 반환합니다. 제너레이터를 구동하기 위해, asyncio는 이를 작업으로 감쌉니다:

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

작업이 완료되면, `StopError`를 발생시키며, 루프는 이를 정상 완료에 도달했다는 신호로 사용합니다.

그런데 이게 뭐죠? 작업에 `add_done_callback`과 `result`라는 메서드가 있네요? 작업이 퓨처와 비슷하다고 생각할 수도 있습니다. 여러분의 직감이 맞습니다. Task 클래스에 대해 숨긴 세부사항을 인정해야 합니다: 작업은 퓨처입니다.

```python
class Task(Future):
    """A coroutine wrapped in a Future."""
```

일반적으로 퓨처는 다른 누군가가 `set_result`를 호출하여 해결됩니다. 하지만 작업은 코루틴이 중지될 때 *스스로* 해결됩니다. Python 제너레이터에 대한 초기 탐구에서 기억하세요. 제너레이터가 반환될 때, 특별한 `StopIteration` 예외를 던집니다:

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

따라서 이벤트 루프가 `task.add_done_callback(stop_callback)`을 호출할 때, 작업에 의해 중지될 준비를 합니다. 다시 `run_until_complete`입니다:

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

작업이 `StopIteration`을 잡고 스스로를 해결하면, 콜백이 루프 내에서 `StopError`를 발생시킵니다. 루프가 멈추고 호출 스택이 `run_until_complete`까지 풀립니다. 우리 프로그램이 완료되었습니다.

## 결론

현대 프로그램들은 점점 더 CPU 바운드가 아닌 I/O 바운드가 되고 있습니다. 이러한 프로그램들에게 Python 스레드는 두 세계의 최악입니다: 전역 인터프리터 락이 실제로 병렬로 계산을 실행하는 것을 방해하고, 선점적 전환이 레이스에 취약하게 만듭니다. 비동기가 종종 올바른 패턴입니다. 하지만 콜백 기반 비동기 코드가 증가하면서, 지저분한 혼란이 되는 경향이 있습니다. 코루틴은 깔끔한 대안입니다. 합리적인 예외 처리와 스택 트레이스와 함께 자연스럽게 서브루틴으로 분해됩니다.

`yield from` 문들이 흐릿해지도록 눈을 가늘게 뜨면, 코루틴은 전통적인 블로킹 I/O를 수행하는 스레드처럼 보입니다. 멀티스레드 프로그래밍의 고전적인 패턴으로 코루틴을 조정할 수도 있습니다. 재발명할 필요가 없습니다. 따라서 콜백과 비교하여, 코루틴은 멀티스레딩에 경험이 있는 코더에게 매력적인 관용구입니다.

하지만 눈을 열고 `yield from` 문에 집중하면, 이들이 코루틴이 제어권을 양보하고 다른 것들이 실행되도록 허용하는 지점을 표시하는 것을 볼 수 있습니다. 스레드와 달리, 코루틴은 우리 코드가 중단될 수 있는 곳과 그렇지 않은 곳을 보여줍니다. 통찰력 있는 에세이 "Unyielding"[^4]에서 Glyph Lefkowitz는 "스레드는 지역적 추론을 어렵게 만들고, 지역적 추론은 아마도 소프트웨어 개발에서 가장 중요한 것입니다"라고 씁니다. 하지만 명시적으로 양보하는 것은 "전체 시스템을 검토하는 것이 아니라 루틴 자체를 검토하여 루틴의 동작(따라서 정확성)을 이해하는" 것을 가능하게 합니다.

이 장은 Python과 비동기 역사의 르네상스 시기에 작성되었습니다. 방금 고안 과정을 배운 제너레이터 기반 코루틴은 2014년 3월 Python 3.4의 "asyncio" 모듈로 릴리스되었습니다. 2015년 9월에 Python 3.5가 코루틴이 언어 자체에 내장되어 릴리스되었습니다. 이러한 네이티브 코루틴은 새로운 구문 "async def"로 선언되며, "yield from" 대신 새로운 "await" 키워드를 사용하여 코루틴에게 위임하거나 Future를 기다립니다.

이러한 발전에도 불구하고, 핵심 아이디어는 남아있습니다. Python의 새로운 네이티브 코루틴은 제너레이터와 구문적으로 구별되지만 매우 유사하게 작동합니다; 실제로 Python 인터프리터 내에서 구현을 공유할 것입니다. Task, Future, 그리고 이벤트 루프는 asyncio에서 계속해서 역할을 할 것입니다.

이제 asyncio 코루틴이 어떻게 작동하는지 알았으니, 세부사항은 대부분 잊어버릴 수 있습니다. 메커니즘은 깔끔한 인터페이스 뒤에 숨겨져 있습니다. 하지만 기본 원리에 대한 여러분의 이해는 현대 비동기 환경에서 올바르고 효율적으로 코딩할 수 있는 힘을 줍니다.

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
