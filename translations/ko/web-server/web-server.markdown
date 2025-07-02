title: 간단한 웹 서버
author: Greg Wilson
<markdown>
_[Greg Wilson](https://twitter.com/gvwilson)은 과학자와 엔지니어를 위한 컴퓨팅 기술 속성 과정인 Software Carpentry의 설립자입니다. 그는 산업계와 학계에서 30년간 일해왔으며, 2008년 Jolt Award를 수상한 *Beautiful Code*와 *The Architecture of Open Source Applications*의 첫 두 권을 포함해 컴퓨팅 관련 여러 책의 저자이자 편집자입니다. Greg은 1993년 에든버러 대학교에서 컴퓨터 과학 박사 학위를 받았습니다._
</markdown>
## 개요

웹은 지난 20년간 수많은 방식으로 사회를 변화시켰지만,
그 핵심은 거의 변하지 않았습니다.
대부분의 시스템은 여전히 Tim Berners-Lee가 25년 전에 제시한
규칙들을 따르고 있습니다.
특히,
대부분의 웹 서버는 여전히 예전과 같은 방식으로
같은 종류의 메시지들을 처리하고 있습니다.

이 장에서는 웹 서버가 어떻게 작동하는지 살펴보겠습니다.
동시에,
개발자가 새로운 기능을 추가하기 위해
전체를 다시 작성할 필요가 없는
소프트웨어 시스템을 어떻게 만들 수 있는지도 알아보겠습니다.

## 배경

웹상의 거의 모든 프로그램은
인터넷 프로토콜(Internet Protocol, IP)이라는 통신 표준 패밀리 위에서 실행됩니다.
이 패밀리 중에서 우리가 관심 있게 봐야 할 것은 전송 제어 프로토콜(Transmission Control Protocol, TCP/IP)인데,
이는 컴퓨터 간의 통신을 파일을 읽고 쓰는 것처럼 보이게 만들어줍니다.

IP를 사용하는 프로그램들은 소켓을 통해 통신합니다.
각 소켓은 점대점 통신 채널의 한쪽 끝으로,
전화기가 전화 통화의 한쪽 끝인 것과 같습니다.
소켓은 특정 머신을 식별하는 IP 주소와
그 머신상의 포트 번호로 구성됩니다.
IP 주소는 `174.136.14.108`과 같이 4개의 8비트 숫자로 구성되며,
도메인 네임 시스템(Domain Name System, DNS)은 이러한 숫자를 `aosabook.org`와 같이
사람이 기억하기 쉬운 기호적인 이름과 매치시켜줍니다.

포트 번호는 0-65535 범위의 숫자로
호스트 머신에서 소켓을 고유하게 식별합니다.
(IP 주소가 회사의 대표 전화번호라면,
포트 번호는 내선번호와 같습니다.)
0-1023번 포트는 운영체제 사용을 위해 예약되어 있고,
나머지 포트는 누구나 사용할 수 있습니다.

하이퍼텍스트 전송 프로토콜(Hypertext Transfer Protocol, HTTP)은
프로그램들이 IP를 통해 데이터를 교환할 수 있는 한 가지 방법을 설명합니다.
HTTP는 의도적으로 단순합니다:
클라이언트가 소켓 연결을 통해 원하는 것을 명시하는 요청을 보내면,
서버가 응답으로 데이터를 보냅니다(\aosafigref{500l.web-server.cycle}.)
이 데이터는 디스크의 파일에서 복사되거나,
프로그램에 의해 동적으로 생성되거나,
또는 둘의 조합일 수 있습니다.

\aosafigure[240pt]{web-server-images/http-cycle.png}{The HTTP Cycle}{500l.web-server.cycle}

HTTP 요청에서 가장 중요한 것은 이것이 단순한 텍스트라는 점입니다:
원하는 프로그램이라면 누구든지 HTTP 요청을 생성하거나 파싱할 수 있습니다.
하지만 제대로 인식되려면,
이 텍스트는 \aosafigref{500l.web-server.request}에 나타난 부분들을 가져야 합니다.

\aosafigure[240pt]{web-server-images/http-request.png}{An HTTP Request}{500l.web-server.request}

HTTP 메서드는 거의 항상 "GET"(정보를 가져오기 위해) 또는 "POST"(폼 데이터를 제출하거나 파일을 업로드하기 위해) 중 하나입니다.
URL은 클라이언트가 원하는 것을 지정합니다;
종종 `/research/experiments.html`과 같이 디스크상의 파일 경로이지만,
(이 부분이 핵심입니다)
그것을 어떻게 처리할지는 완전히 서버가 결정하는 것입니다.
HTTP 버전은 보통 "HTTP/1.0" 또는 "HTTP/1.1"입니다;
이 둘 사이의 차이는 우리에게는 중요하지 않습니다.

HTTP 헤더는 아래에 나타난 세 개의 예시와 같은 키/값 쌍입니다:

```
Accept: text/html
Accept-Language: en, fr
If-Modified-Since: 16-May-2005
```

해시 테이블의 키와는 달리,
HTTP 헤더에서는 키가 몇 번이든 나타날 수 있습니다.
이를 통해 요청은 여러 유형의 콘텐츠를 받아들일 의향이 있다고
명시하는 것과 같은 일들을 할 수 있습니다.

마지막으로,
요청의 본문(body)은 요청과 연관된 추가 데이터입니다.
이는 웹 폼을 통해 데이터를 제출할 때,
파일을 업로드할 때,
그리고 기타 등등에 사용됩니다.
헤더의 끝을 알리기 위해
마지막 헤더와 본문의 시작 사이에는 반드시 빈 줄이 있어야 합니다.

`Content-Length`라고 불리는 헤더는
서버에게 요청의 본문에서 읽어야 할 바이트 수를 알려줍니다.

HTTP 응답은 HTTP 요청과 같은 형식을 갖습니다(\aosafigref{500l.web-server.response}):

\aosafigure[240pt]{web-server-images/http-response.png}{An HTTP Response}{500l.web-server.response}

버전, 헤더, 본문은 같은 형태와 의미를 갖습니다.
상태 코드는 요청이 처리될 때 무엇이 일어났는지를 나타내는 숫자입니다:
200은 "모든 것이 정상적으로 작동했다"는 뜻이고,
404는 "찾을 수 없다"는 뜻이며,
다른 코드들은 다른 의미를 갖습니다.
상태 구문은 "OK"나 "not found"와 같이 사람이 읽을 수 있는 구문으로 그 정보를 반복합니다.

이 장의 목적상
HTTP에 대해 알아야 할 다른 것들은 단 두 가지뿐입니다.

첫 번째는 HTTP가 *무상태(stateless)*라는 것입니다:
각 요청은 독립적으로 처리되며,
서버는 한 요청과 다음 요청 사이에 아무것도 기억하지 않습니다.
애플리케이션이 사용자의 신원과 같은 정보를 추적하고 싶다면,
스스로 그렇게 해야 합니다.

이를 위한 일반적인 방법은 쿠키를 사용하는 것인데,
쿠키는 서버가 클라이언트에게 보내는 짧은 문자열이며,
클라이언트가 나중에 서버로 다시 돌려보냅니다.
사용자가 여러 요청에 걸쳐 상태를 저장해야 하는 기능을 수행할 때,
서버는 새로운 쿠키를 생성하고,
이를 데이터베이스에 저장한 다음,
사용자의 브라우저로 보냅니다.
브라우저가 쿠키를 다시 보낼 때마다,
서버는 이를 사용해 사용자가 무엇을 하고 있는지에 대한 정보를 조회합니다.

HTTP에 대해 알아야 할 두 번째는
URL이 더 많은 정보를 제공하기 위해
매개변수로 보완될 수 있다는 것입니다.
예를 들어,
검색 엔진을 사용한다면,
검색어가 무엇인지 명시해야 합니다.
이것들을 URL의 경로에 추가할 수도 있지만,
해야 할 일은 URL에 매개변수를 추가하는 것입니다.
URL에 '?'를 추가한 다음
'&amp;'로 구분된 'key=value' 쌍들을 따라오게 하여 이를 수행합니다.
예를 들어,
URL `http://www.google.ca?q=Python`은
Google에게 Python과 관련된 페이지를 검색하라고 요청합니다:
키는 문자 'q'이고,
값은 'Python'입니다.
더 긴 쿼리인
`http://www.google.ca/search?q=Python&amp;client=Firefox`는
Google에게 우리가 Firefox를 사용하고 있다고 알려주며,
이런 식으로 계속됩니다.
원하는 매개변수는 무엇이든 전달할 수 있지만,
다시 말하지만,
어떤 것들에 주의를 기울일지,
그리고 그것들을 어떻게 해석할지 결정하는 것은
웹 사이트에서 실행되는 애플리케이션에 달려 있습니다.

물론,
'?'와 '&amp;'가 특수 문자라면,
이들을 이스케이프하는 방법이 있어야 합니다.
마치 큰따옴표로 구분된 문자열 안에
큰따옴표 문자를 넣는 방법이 있어야 하는 것과 같습니다.
URL 인코딩 표준은
'%' 다음에 2자리 코드를 사용하여 특수 문자를 표현하고,
공백을 '+' 문자로 바꿉니다.
따라서,
Google에서 "grade&nbsp;=&nbsp;A+" (공백 포함)를 검색하려면,
URL `http://www.google.ca/search?q=grade+%3D+A%2B`를 사용해야 합니다.

소켓을 열고, HTTP 요청을 구성하고, 응답을 파싱하는 것은 지루한 일이므로,
대부분의 사람들은 라이브러리를 사용해 대부분의 작업을 처리합니다.
Python에는 `urllib2`라는 라이브러리가 함께 제공되는데
(이는 `urllib`라는 이전 라이브러리의 대체재이기 때문입니다),
하지만 이는 대부분의 사람들이 신경 쓰고 싶어하지 않는 많은 내부 구조를 노출합니다.
[Requests](https://pypi.python.org/pypi/requests) 라이브러리는 `urllib2`보다 사용하기 쉬운 대안입니다.
다음은 AOSA 도서 사이트에서 페이지를 다운로드하는 데 사용하는 예시입니다:

```python
import requests
response = requests.get('http://aosabook.org/en/500L/web-server/testpage.html')
print 'status code:', response.status_code
print 'content length:', response.headers['content-length']
print response.text
```

``` 
status code: 200
content length: 61
<html>
  <body>
    <p>Test page.</p>
  </body>
</html>
```

`request.get`은 서버에 HTTP GET 요청을 보내고
응답을 담고 있는 객체를 반환합니다.
이 객체의 `status_code` 멤버는 응답의 상태 코드이고;
`content_length` 멤버는 응답 데이터의 바이트 수이며,
`text`는 실제 데이터입니다
(이 경우에는 HTML 페이지).

## Hello, Web

이제 첫 번째 간단한 웹 서버를 작성할 준비가 되었습니다.
기본 아이디어는 간단합니다:

1.  누군가가 우리 서버에 연결하여 HTTP 요청을 보낼 때까지 기다립니다;
2.  그 요청을 파싱합니다;
3.  무엇을 요청하고 있는지 파악합니다;
4.  그 데이터를 가져옵니다 (또는 동적으로 생성합니다);
5.  데이터를 HTML로 포맷합니다; 그리고
6.  다시 보내줍니다.

1, 2, 6단계는 애플리케이션마다 동일하므로,
Python 표준 라이브러리에는 `BaseHTTPServer`라는 모듈이 있어
이를 대신 처리해줍니다.
우리는 3-5단계만 처리하면 되는데,
아래의 작은 프로그램에서 이를 수행합니다:

```python
import BaseHTTPServer

class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):
    '''Handle HTTP requests by returning a fixed 'page'.'''

    # Page to send back.
    Page = '''\
<html>
<body>
<p>Hello, web!</p>
</body>
</html>
'''

    # Handle a GET request.
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/html")
        self.send_header("Content-Length", str(len(self.Page)))
        self.end_headers()
        self.wfile.write(self.Page)

#----------------------------------------------------------------------

if __name__ == '__main__':
    serverAddress = ('', 8080)
    server = BaseHTTPServer.HTTPServer(serverAddress, RequestHandler)
    server.serve_forever()
```

라이브러리의 `BaseHTTPRequestHandler` 클래스는
들어오는 HTTP 요청을 파싱하고
어떤 메서드를 포함하고 있는지 결정하는 역할을 합니다.
메서드가 GET이면,
클래스는 `do_GET`이라는 메서드를 호출합니다.
우리의 `RequestHandler` 클래스는 이 메서드를 오버라이드하여
간단한 페이지를 동적으로 생성합니다:
텍스트는 클래스 수준 변수인 `Page`에 저장되며,
200 응답 코드,
클라이언트에게 우리 데이터를 HTML로 해석하라고 알려주는 `Content-Type` 헤더,
그리고 페이지의 길이를 보낸 후에
이를 클라이언트에게 다시 보냅니다.
(`end_headers` 메서드 호출은 헤더와 페이지 자체를 구분하는
빈 줄을 삽입합니다.)

하지만 `RequestHandler`가 전부는 아닙니다:
실제로 서버를 시작하려면 마지막 세 줄이 여전히 필요합니다.
이 줄들 중 첫 번째는 서버의 주소를 튜플로 정의합니다:
빈 문자열은 "현재 머신에서 실행"을 의미하고,
8080은 포트입니다.
그다음 그 주소와 요청 핸들러 클래스의 이름을 매개변수로 하여
`BaseHTTPServer.HTTPServer`의 인스턴스를 생성한 다음,
영원히 실행하도록 요청합니다
(실제로는 Control-C로 종료할 때까지를 의미합니다).

이 프로그램을 명령줄에서 실행하면,
아무것도 표시되지 않습니다:

```bash
$ python server.py
```

하지만 브라우저로 `http://localhost:8080`에 접속하면,
브라우저에서 다음과 같이 나타납니다:

```
Hello, web!
```

그리고 셸에서는 다음과 같이 나타납니다:

```
127.0.0.1 - - [24/Feb/2014 10:26:28] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [24/Feb/2014 10:26:28] "GET /favicon.ico HTTP/1.1" 200 -
```

첫 번째 줄은 간단합니다:
특정 파일을 요청하지 않았으므로,
브라우저는 '/' (서버가 제공하는 것의 루트 디렉터리)를 요청했습니다.
두 번째 줄이 나타나는 이유는
브라우저가 자동으로 `/favicon.ico`라는 이미지 파일에 대한
두 번째 요청을 보내기 때문인데,
이 파일이 존재하면 주소 표시줄에 아이콘으로 표시됩니다.

## 값 표시하기

HTTP 요청에 포함된 일부 값들을 표시하도록
웹 서버를 수정해봅시다.
(디버깅할 때 이를 꽤 자주 할 것이므로,
연습해볼 만합니다.)
코드를 깔끔하게 유지하기 위해,
페이지 생성과 전송을 분리하겠습니다:

```python
class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):

    # ...page template...

    def do_GET(self):
        page = self.create_page()
        self.send_page(page)

    def create_page(self):
        # ...fill in...

    def send_page(self, page):
        # ...fill in...
```

`send_page`는 이전에 가지고 있던 것과 거의 같습니다:

```python
    def send_page(self, page):
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.send_header("Content-Length", str(len(page)))
        self.end_headers()
        self.wfile.write(page)
```

표시하고 싶은 페이지의 템플릿은
몇 가지 포맷팅 플레이스홀더가 있는
HTML 테이블을 포함하는 단순한 문자열입니다:

```python
    Page = '''\
<html>
<body>
<table>
<tr>  <td>Header</td>         <td>Value</td>          </tr>
<tr>  <td>Date and time</td>  <td>{date_time}</td>    </tr>
<tr>  <td>Client host</td>    <td>{client_host}</td>  </tr>
<tr>  <td>Client port</td>    <td>{client_port}s</td> </tr>
<tr>  <td>Command</td>        <td>{command}</td>      </tr>
<tr>  <td>Path</td>           <td>{path}</td>         </tr>
</table>
</body>
</html>
'''
```

\noindent 그리고 이를 채우는 메서드는 다음과 같습니다:

```python
    def create_page(self):
        values = {
            'date_time'   : self.date_time_string(),
            'client_host' : self.client_address[0],
            'client_port' : self.client_address[1],
            'command'     : self.command,
            'path'        : self.path
        }
        page = self.Page.format(**values)
        return page
```

프로그램의 메인 부분은 변경되지 않았습니다:
이전과 같이,
주소와 이 요청 핸들러를 매개변수로 하여
`HTTPServer` 클래스의 인스턴스를 생성한 다음,
영원히 요청을 처리합니다.
브라우저에서 `http://localhost:8080/something.html`에 대한
요청을 실행하고 보내면,
다음과 같이 나타납니다:

```
  Date and time  Mon, 24 Feb 2014 17:17:12 GMT
  Client host    127.0.0.1
  Client port    54548
  Command        GET
  Path           /something.html
```

`something.html` 페이지가 디스크의 파일로 존재하지 않음에도 불구하고
404 오류가 *발생하지 않는다*는 점에 주목하세요.
이는 웹 서버가 단순한 프로그램이고,
요청을 받았을 때 원하는 무엇이든 할 수 있기 때문입니다:
이전 요청에서 명명된 파일을 다시 보내거나,
무작위로 선택된 위키피디아 페이지를 제공하거나,
또는 우리가 프로그래밍한 다른 무엇이든 할 수 있습니다.

## 정적 페이지 제공하기

다음 단계는 즉석에서 생성하는 대신
디스크에서 페이지를 제공하기 시작하는 것입니다.
`do_GET`을 다시 작성하는 것부터 시작하겠습니다:

```python
    def do_GET(self):
        try:

            # Figure out what exactly is being requested.
            full_path = os.getcwd() + self.path

            # It doesn't exist...
            if not os.path.exists(full_path):
                raise ServerException("'{0}' not found".format(self.path))

            # ...it's a file...
            elif os.path.isfile(full_path):
                self.handle_file(full_path)

            # ...it's something we don't handle.
            else:
                raise ServerException("Unknown object '{0}'".format(self.path))

        # Handle errors.
        except Exception as msg:
            self.handle_error(msg)
```

이 메서드는 웹 서버가 실행되고 있는 디렉터리나
그 하위 디렉터리의 모든 파일을 제공할 수 있다고 가정합니다
(`os.getcwd`를 사용하여 얻습니다).
이를 URL에서 제공된 경로와 결합하여
(라이브러리가 자동으로 `self.path`에 넣어주며,
항상 앞에 '/'가 있습니다)
사용자가 원하는 파일의 경로를 얻습니다.

그것이 존재하지 않거나,
파일이 아닌 경우,
메서드는 예외를 발생시키고 잡아서 오류를 보고합니다.
반면에 경로가 파일과 일치하면,
내용을 읽고 반환하기 위해
`handle_file`이라는 도우미 메서드를 호출합니다.
이 메서드는 단순히 파일을 읽고
기존의 `send_content`를 사용하여 클라이언트에게 다시 보냅니다:

```python 
    def handle_file(self, full_path):
        try:
            with open(full_path, 'rb') as reader:
                content = reader.read()
            self.send_content(content)
        except IOError as msg:
            msg = "'{0}' cannot be read: {1}".format(self.path, msg)
            self.handle_error(msg)
```

파일을 바이너리 모드로 열었다는 점에 주목하세요&mdash;'rb'의 'b'&mdash;
Python이 Windows 라인 엔딩처럼 보이는 바이트 시퀀스를 변경하여
"도움"을 주려고 하지 않도록 하기 위해서입니다.
또한 파일을 제공할 때 전체 파일을 메모리로 읽는 것은
파일이 몇 기가바이트의 비디오 데이터일 수 있는 실제 상황에서는
좋지 않은 아이디어라는 점도 주목하세요.
이런 상황을 처리하는 것은 이 장의 범위를 벗어납니다.

이 클래스를 완성하기 위해,
오류 처리 메서드와
오류 보고 페이지의 템플릿을 작성해야 합니다:

```python 
    Error_Page = """\
        <html>
        <body>
        <h1>Error accessing {path}</h1>
        <p>{msg}</p>
        </body>
        </html>
        """

    def handle_error(self, msg):
        content = self.Error_Page.format(path=self.path, msg=msg)
        self.send_content(content)
```

이 프로그램은 작동하지만,
너무 자세히 보지 않을 때만 그렇습니다.
문제는 요청된 페이지가 존재하지 않을 때에도
항상 200 상태 코드를 반환한다는 것입니다.
맞습니다,
그런 경우에 다시 보내진 페이지에는 오류 메시지가 포함되어 있지만,
브라우저는 영어를 읽을 수 없으므로,
요청이 실제로 실패했다는 것을 알지 못합니다.
이를 명확히 하기 위해,
다음과 같이 `handle_error`와 `send_content`를 수정해야 합니다:

```python 
    # Handle unknown objects.
    def handle_error(self, msg):
        content = self.Error_Page.format(path=self.path, msg=msg)
        self.send_content(content, 404)

    # Send actual content.
    def send_content(self, content, status=200):
        self.send_response(status)
        self.send_header("Content-type", "text/html")
        self.send_header("Content-Length", str(len(content)))
        self.end_headers()
        self.wfile.write(content)
```

파일을 찾을 수 없을 때 `ServerException`을 발생시키지 않고,
대신 오류 페이지를 생성한다는 점에 주목하세요.
`ServerException`은 서버 코드의 내부 오류를 알리기 위한 것입니다,
즉,
*우리가* 잘못한 것을 의미합니다.
반면에 `handle_error`에 의해 생성된 오류 페이지는
*사용자가* 무언가를 잘못했을 때 나타납니다,
즉,
존재하지 않는 파일의 URL을 우리에게 보냈을 때입니다. [^handleerror]

[^handleerror]: 이 장 전체에서 `handle_error`를 여러 번 사용할 것인데, 상태 코드 `404`가 적절하지 않은 여러 경우가 포함됩니다. 읽어나가면서, 각 경우에 상태 응답 코드를 쉽게 제공할 수 있도록 이 프로그램을 어떻게 확장할지 생각해보세요.

## 디렉터리 목록 표시

다음 단계로,
URL의 경로가 파일이 아닌 디렉터리일 때
디렉터리의 내용 목록을 표시하도록 웹 서버를 가르칠 수 있습니다.
한 뱸 더 나아가서
그 디렉터리에서 표시할 `index.html` 파일을 찾아보고,
그 파일이 없을 때만 디렉터리 내용의 목록을 보여주도록 할 수도 있습니다.

하지만 이러한 규칙들을 `do_GET`에 구축하는 것은 실수일 것입니다,
결과적으로 메서드는 특수 동작을 제어하는
긴 `if` 문들의 뒤엉킨 볏이 될 것이기 때문입니다.
올바른 해결책은 한 및 물러서서 일반적인 문제를 해결하는 것인데,
즉 URL로 무엇을 할지 파악하는 것입니다.
다음은 `do_GET` 메서드의 다시 작성된 버전입니다:

```python
    def do_GET(self):
        try:

            # Figure out what exactly is being requested.
            self.full_path = os.getcwd() + self.path

            # Figure out how to handle it.
            for case in self.Cases:
                handler = case()
                if handler.test(self):
                    handler.act(self)
                    break

        # Handle errors.
        except Exception as msg:
            self.handle_error(msg)
```

첫 번째 단계는 동일합니다:
요청되는 것의 전체 경로를 파악합니다.
그 다음은,
하지만,
코드가 상당히 다르게 보입니다.
많은 인라인 테스트 대신,
이 버전은 리스트에 저장된 케이스 집합을 루프로 돌며 처리합니다.
각 케이스는 두 개의 메서드를 가진 객체입니다:
요청을 처리할 수 있는지 알려주는 `test`,
그리고 실제로 어떤 작업을 수행하는 `act`.
올바른 케이스를 찾자마자,
그것이 요청을 처리하도록 하고
루프에서 빠져나옵니다.

These three case classes reproduce the behavior of our previous server:

```python
class case_no_file(object):
    '''File or directory does not exist.'''

    def test(self, handler):
        return not os.path.exists(handler.full_path)

    def act(self, handler):
        raise ServerException("'{0}' not found".format(handler.path))


class case_existing_file(object):
    '''File exists.'''

    def test(self, handler):
        return os.path.isfile(handler.full_path)

    def act(self, handler):
        handler.handle_file(handler.full_path)


class case_always_fail(object):
    '''Base case if nothing else worked.'''

    def test(self, handler):
        return True

    def act(self, handler):
        raise ServerException("Unknown object '{0}'".format(handler.path))
```

\noindent and here's how we construct the list of case handlers
at the top of the `RequestHandler` class:

```python 
class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):
    '''
    If the requested path maps to a file, that file is served.
    If anything goes wrong, an error page is constructed.
    '''

    Cases = [case_no_file(),
             case_existing_file(),
             case_always_fail()]

    ...everything else as before...
```

이제,
표면적으로는 이것이 우리 서버를 더 복잡하게 만들었습니다,
덜 복잡하게 한 것이 아니라:
파일이 74줄에서 99줄로 늘어났고,
새로운 기능 없이 추가적인 간접 계층이 있습니다.
이점은 이 장을 시작한 작업으로 돌아가서
디렉터리에 `index.html` 페이지가 있으면 그것을 제공하고,
없으면 디렉터리 목록을 제공하도록
서버를 가르치려고 할 때 나타납니다.
전자를 위한 핸들러는 다음과 같습니다:

```python
class case_directory_index_file(object):
    '''Serve index.html page for a directory.'''

    def index_path(self, handler):
        return os.path.join(handler.full_path, 'index.html')

    def test(self, handler):
        return os.path.isdir(handler.full_path) and \
               os.path.isfile(self.index_path(handler))

    def act(self, handler):
        handler.handle_file(self.index_path(handler))
```

여기서,
도우미 메서드 `index_path`는 `index.html` 파일의 경로를 구성합니다;
이를 케이스 핸들러에 넣으면 메인 `RequestHandler`의 혼란을 방지합니다.
`test`는 경로가 `index.html` 페이지를 포함하는 디렉터리인지 확인하고,
`act`는 메인 요청 핸들러에게 그 페이지를 제공하도록 요청합니다.

`RequestHandler`에 필요한 유일한 변경사항은 `Cases` 리스트에 `case_directory_index_file` 객체를 추가하는 것입니다:

```python 
    Cases = [case_no_file(),
             case_existing_file(),
             case_directory_index_file(),
             case_always_fail()]
```

`index.html` 페이지를 포함하지 않는 디렉터리는 어떨까요?
테스트는 위의 것과 동일하지만
전략적으로 `not`이 삽입되었고,
`act` 메서드는 어떨까요?
무엇을 해야 할까요?

```python
class case_directory_no_index_file(object):
    '''Serve listing for a directory without an index.html page.'''

    def index_path(self, handler):
        return os.path.join(handler.full_path, 'index.html')

    def test(self, handler):
        return os.path.isdir(handler.full_path) and \
               not os.path.isfile(self.index_path(handler))

    def act(self, handler):
        ???
```

우리가 다시 막다른 지경에 빠진 것 같습니다.
논리적으로,
`act` 메서드는 디렉터리 목록을 생성하고 반환해야 하지만,
기존 코드는 이를 허용하지 않습니다:
`RequestHandler.do_GET`은 `act`를 호출하지만,
그것으로부터의 반환 값을 기대하거나 처리하지 않습니다.
일단은,
디렉터리 목록을 생성하는 메서드를 `RequestHandler`에 추가하고,
케이스 핸들러의 `act`에서 그것을 호출해봅시다:

```python 
class case_directory_no_index_file(object):
    '''Serve listing for a directory without an index.html page.'''

    # ...index_path and test as above...

    def act(self, handler):
        handler.list_dir(handler.full_path)


class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):

    # ...all the other code...

    # How to display a directory listing.
    Listing_Page = '''\
        <html>
        <body>
        <ul>
        {0}
        </ul>
        </body>
        </html>
        '''

    def list_dir(self, full_path):
        try:
            entries = os.listdir(full_path)
            bullets = ['<li>{0}</li>'.format(e) 
                for e in entries if not e.startswith('.')]
            page = self.Listing_Page.format('\n'.join(bullets))
            self.send_content(page)
        except OSError as msg:
            msg = "'{0}' cannot be listed: {1}".format(self.path, msg)
            self.handle_error(msg)
```

## CGI 프로토콜

물론,
대부분의 사람들은 새로운 기능을 추가하기 위해
웹 서버의 소스 코드를 편집하고 싶어하지 않을 것입니다.
그들이 그렇게 할 필요가 없도록 하기 위해,
서버들은 항상 CGI(Common Gateway Interface)라고 불리는
메커니즘을 지원해왔는데,
이는 웹 서버가 요청을 만족시키기 위해
외부 프로그램을 실행하는 표준적인 방법을 제공합니다.

예를 들어,
서버가 HTML 페이지에 로컬 시간을 표시할 수 있기를 원한다고 가정해봅시다.
단지 보 줄의 코드로 독립실행형 프로그램에서 이를 수행할 수 있습니다:

```python
from datetime import datetime
print '''\
<html>
<body>
<p>Generated {0}</p>
</body>
</html>'''.format(datetime.now())
```

웹 서버가 우리를 위해 이 프로그램을 실행하도록 하기 위해,
이 케이스 핸들러를 추가합니다:

```python 
class case_cgi_file(object):
    '''Something runnable.'''

    def test(self, handler):
        return os.path.isfile(handler.full_path) and \
               handler.full_path.endswith('.py')

    def act(self, handler):
        handler.run_cgi(handler.full_path)
```

테스트는 간단합니다:
파일 경로가 `.py`로 끝나는가?
그렇다면, `RequestHandler`가 이 프로그램을 실행합니다.

```python 
    def run_cgi(self, full_path):
        cmd = "python " + full_path
        child_stdin, child_stdout = os.popen2(cmd)
        child_stdin.close()
        data = child_stdout.read()
        child_stdout.close()
        self.send_content(data)
```

이것은 보안에 매우 취약합니다:
누군가가 우리 서버의 Python 파일 경로를 알고 있다면,
그것이 어떤 데이터에 액세스할 수 있는지,
무한 루프를 포함할 수 있는지,
또는 다른 어떤 것에 대한 걱정 없이
우리는 단순히 그것을 실행하도록 요청합니다.[^popen]

[^popen]: 우리 코드는 더 이상 사용되지 않는 `popen2` 라이브러리 함수를 사용하는데, `subprocess` 모듈을 사용하는 것이 선호됩니다. 하지만 `popen2`가 이 예시에서는 덜 복잡하고 이해하기 쉬운 도구였습니다.

그것을 제쳐두고,
핵심 아이디어는 간단합니다:

1.  프로그램을 서브프로세스에서 실행합니다.
2.  그 서브프로세스가 표준 출력으로 보내는 모든 것을 캐처합니다.
3.  요청을 한 클라이언트에게 그것을 다시 보냅니다.

완전한 CGI 프로토콜은 이보다 훨씬 풍부합니다&mdash;특히,
URL에서 매개변수를 허용하여
서버가 실행 중인 프로그램에 전달합니다&mdash;하지만
그러한 세부사항들은 시스템의 전체 아키텍처에 영향을 주지 않습니다...

...다시 한 번 상당히 복잡해지고 있는 시스템입니다.
`RequestHandler`는 처음에 콘텐츠를 처리하기 위한
하나의 메서드인 `handle_file`을 가지고 있었습니다.
이제 우리는 `list_dir`과 `run_cgi` 형태로
두 개의 특수 케이스를 추가했습니다.
이 세 메서드들은 주로 다른 것들에 의해 사용되므로,
실제로는 있는 곳에 속하지 않습니다.

수정은 간단합니다:
모든 케이스 핸들러를 위한 부모 클래스를 생성하고,
두 개 이상의 핸들러에 의해 공유되는 경우에만
다른 메서드들을 그 클래스로 이동시킵니다.
완료되면,
`RequestHandler` 클래스는 다음과 같이 보입니다:

```python
class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):

    Cases = [case_no_file(),
             case_cgi_file(),
             case_existing_file(),
             case_directory_index_file(),
             case_directory_no_index_file(),
             case_always_fail()]

    # How to display an error.
    Error_Page = """\
        <html>
        <body>
        <h1>Error accessing {path}</h1>
        <p>{msg}</p>
        </body>
        </html>
        """

    # Classify and handle request.
    def do_GET(self):
        try:

            # Figure out what exactly is being requested.
            self.full_path = os.getcwd() + self.path

            # Figure out how to handle it.
            for case in self.Cases:
                if case.test(self):
                    case.act(self)
                    break

        # Handle errors.
        except Exception as msg:
            self.handle_error(msg)

    # Handle unknown objects.
    def handle_error(self, msg):
        content = self.Error_Page.format(path=self.path, msg=msg)
        self.send_content(content, 404)

    # Send actual content.
    def send_content(self, content, status=200):
        self.send_response(status)
        self.send_header("Content-type", "text/html")
        self.send_header("Content-Length", str(len(content)))
        self.end_headers()
        self.wfile.write(content)
```

\noindent 그리고 케이스 핸들러들의 부모 클래스는 다음과 같습니다:

```python 
class base_case(object):
    '''Parent for case handlers.'''

    def handle_file(self, handler, full_path):
        try:
            with open(full_path, 'rb') as reader:
                content = reader.read()
            handler.send_content(content)
        except IOError as msg:
            msg = "'{0}' cannot be read: {1}".format(full_path, msg)
            handler.handle_error(msg)

    def index_path(self, handler):
        return os.path.join(handler.full_path, 'index.html')

    def test(self, handler):
        assert False, 'Not implemented.'

    def act(self, handler):
        assert False, 'Not implemented.'
```

\noindent 그리고 기존 파일을 위한 핸들러는
(무작위로 예시를 선택하자면) 다음과 같습니다:

```python 
class case_existing_file(base_case):
    '''File exists.'''

    def test(self, handler):
        return os.path.isfile(handler.full_path)

    def act(self, handler):
        self.handle_file(handler, handler.full_path)
```

## 토론

원래 코드와 리팩터링된 버전 사이의 차이점은
두 가지 중요한 아이디어를 반영합니다.
첫 번째는 클래스를 관련된 서비스들의 집합으로 생각하는 것입니다.
`RequestHandler`와 `base_case`는 결정을 내리거나 행동을 취하지 않습니다;
다른 클래스들이 그런 일들을 할 수 있도록 도구를 제공합니다.

두 번째는 확장성입니다:
사람들은 외부 CGI 프로그램을 작성하거나,
케이스 핸들러 클래스를 추가함으로써
우리 웹 서버에 새로운 기능을 추가할 수 있습니다.
후자는 `RequestHandler`에 한 줄의 변경이 필요하긴 하지만
(`Cases` 리스트에 케이스 핸들러를 삽입하기 위해),
웹 서버가 설정 파일을 읽고 그것으로부터 핸들러 클래스들을 로드하도록 함으로써
이를 제거할 수도 있습니다.
두 경우 모두,
`BaseHTTPRequestHandler` 클래스의 작성자들이
소켓 연결 처리와 HTTP 요청 파싱의 세부사항을 무시할 수 있게 해준 것처럼,
그들은 대부분의 낮은 수준의 세부사항들을 무시할 수 있습니다.

이러한 아이디어들은 일반적으로 유용합니다;
여러분의 프로젝트에서 이를 사용할 방법을 찾아보세요. 
