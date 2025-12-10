title: 동일 출처 정책
author: Eunsuk Kang, Santiago Perez De Rosso, and Daniel Jackson
<markdown>
_Eunsuk Kang은 MIT 소프트웨어 설계 그룹(Software Design Group)의 박사과정 학생이자 멤버입니다. 그는 MIT에서 컴퓨터 과학 석사 학위(2010)를, 워털루 대학교에서 소프트웨어 공학 학사 학위(2007)를 받았습니다. 그의 연구 프로젝트는 보안 및 안전 중요 시스템에 적용되는 소프트웨어 모델링 및 검증 도구와 기법 개발에 중점을 두고 있습니다._

_Santiago Perez De Rosso는 MIT 소프트웨어 설계 그룹의 박사과정 학생입니다. 그는 MIT에서 컴퓨터 과학 석사 학위(2015)를, ITBA에서 학사 학위(2011)를 받았습니다. 그는 Google에서 엔지니어들의 생산성을 높이는 프레임워크와 도구를 개발하는 일을 했으며(2012), 현재는 대부분의 시간을 설계와 버전 관리에 대해 생각하며 보내고 있습니다._

_Daniel Jackson은 MIT 전기공학 및 컴퓨터과학과(Department of Electrical Engineering and Computer Science) 교수이며, 컴퓨터과학 및 인공지능 연구소(Computer Science and Artificial Intelligence Laboratory)의 소프트웨어 설계 그룹을 이끌고 있습니다. 그는 옥스퍼드 대학교에서 물리학 석사 학위(1984)를, MIT에서 컴퓨터 과학 석사(1988)와 박사 학위(1992)를 받았습니다. Logica UK Ltd.에서 소프트웨어 엔지니어로 근무했고(1984-1986), 카네기 멜론 대학교에서 컴퓨터과학과 조교수를 역임했으며(1992-1997), 1997년부터 MIT에 재직하고 있습니다. 그는 소프트웨어 공학 분야, 특히 개발 방법론, 설계 및 명세, 형식적 방법론, 안전 중요 시스템에 폭넓은 관심을 가지고 있습니다._
</markdown>
## 도입

동일 출처 정책(Same-Origin Policy, SOP)은 모든 현대 브라우저의 보안 메커니즘에서 중요한 부분입니다. 이 정책은 브라우저에서 실행되는 스크립트들이 언제 서로 통신할 수 있는지를 제어합니다(대략적으로는, 같은 웹사이트에서 출처한 경우). 넷스케이프 네비게이터에서 처음 도입된 SOP는 이제 웹 애플리케이션 보안에서 중요한 역할을 담당하고 있습니다. SOP가 없다면, 악의적인 해커가 페이스북의 개인 사진을 훔쳐보거나, 이메일을 읽거나, 은행 계좌를 비우는 것이 훨씬 쉬워질 것입니다.

하지만 SOP는 완벽하지 않습니다. 때로는 너무 제한적이어서, 서로 다른 출처의 스크립트들이 리소스를 공유해야 하는 경우(매시업 등)에도 불가능한 상황이 있습니다. 반대로 때로는 충분히 제한적이지 않아서, 사이트 간 요청 위조(cross-site request forgery)와 같은 일반적인 공격에 의해 악용될 수 있는 코너 케이스들이 남아있기도 합니다. 게다가 SOP의 설계는 수년에 걸쳐 유기적으로 진화해왔기 때문에 많은 개발자들을 당황시키고 있습니다.

이 장의 목표는 중요하지만 종종 오해받는 이 기능의 본질을 파악하는 것입니다. 특히 다음과 같은 질문들에 대답해보고자 합니다:

* SOP가 왜 필요한가? 이것이 방지하는 보안 위반의 유형은 무엇인가?
* 웹 애플리케이션의 동작은 SOP에 의해 어떻게 영향을 받는가?
* SOP를 우회하는 여러 메커니즘들은 무엇인가?
* 이러한 메커니즘들은 얼마나 안전한가? 이들이 도입하는 잠재적 보안 문제는 무엇인가?

SOP를 전체적으로 다루는 것은 관련된 부분들의 복잡성을 고려하면 어려운 작업입니다—웹 서버, 브라우저, HTTP, HTML 문서, 클라이언트 측 스크립트 등등 말입니다. 이러한 모든 부분들의 세부 사항에 얽매이게 되면 (SOP에 도달하기도 전에 500라인을 모두 소비하게 될 것입니다). 하지만 중요한 세부 사항을 표현하지 않고서는 어떻게 정확성을 기대할 수 있을까요?

## Alloy를 활용한 모델링

이 장은 이 책의 다른 장들과는 다소 다릅니다. 실제로 동작하는 구현체를 만드는 대신, SOP의 간단하면서도 정확한 설명 역할을 하는 실행 가능한 모델을 구축할 것입니다. 구현체와 마찬가지로 이 모델은 시스템의 동적 행동을 탐색하기 위해 실행될 수 있지만, 구현체와는 달리 핵심 개념을 이해하는 데 방해가 될 수 있는 저수준 세부 사항들은 생략합니다.

우리가 취하는 접근 방식은 애자일 프로그래밍과의 유사성 때문에 "애자일 모델링"이라고 불릴 수 있습니다. 우리는 점진적으로 작업하며, 모델을 조각조각 조립해 나갑니다. 진화하는 우리의 모델은 매 순간 실행할 수 있는 상태입니다. 진행하면서 테스트를 공식화하고 실행하므로, 결국에는 모델 자체뿐만 아니라 그것이 만족하는 _속성들_의 모음도 얻게 됩니다.

이 모델을 구축하기 위해 우리는 소프트웨어 설계를 모델링하고 분석하는 언어인 _Alloy_를 사용합니다. Alloy 모델은 전통적인 프로그램 실행 의미에서 실행될 수 없습니다. 대신, 모델은 (1) 시스템의 유효한 시나리오나 구성을 나타내는 _인스턴스_를 생성하기 위해 _시뮬레이션_되거나, (2) 모델이 원하는 속성을 만족하는지 확인하기 위해 _검사_될 수 있습니다.

위의 유사점들에도 불구하고, 애자일 모델링은 한 가지 핵심적인 면에서 애자일 프로그래밍과 다릅니다: 테스트를 실행하기는 하지만, 실제로는 테스트를 작성하지 않습니다. Alloy의 분석기가 테스트 케이스를 자동으로 생성하며, 제공해야 하는 것은 검사할 속성뿐입니다. 말할 필요도 없이, 이는 많은 수고(와 텍스트)를 덜어줍니다. 분석기는 실제로 특정 크기(스코프_scope_라고 함)까지의 모든 가능한 테스트 케이스를 실행합니다. 이는 일반적으로 최대 몇 개의 객체를 가진 모든 시작 상태를 생성한 다음, 몇 단계까지 적용할 연산과 인수를 선택하는 것을 의미합니다. 매우 많은 테스트(일반적으로 수십억 개)가 실행되고, 상태가 취할 수 있는 모든 가능한 구성이 (스코프 내에서나마) 커버되기 때문에, 이러한 분석은 기존의 테스트보다 더 효과적으로 버그를 노출하는 경향이 있습니다(때로는 "경계 검증"이라고 설명되기도 합니다).

### 단순화

SOP는 브라우저, 서버, HTTP 등의 맥락에서 동작하므로, 완전한 설명은 압도적일 것입니다. 따라서 우리 모델은 (모든 모델이 그렇듯이) 네트워크 패킷이 구조화되고 라우팅되는 방법과 같이 관련 없는 측면들을 추상화합니다. 하지만 일부 관련 있는 측면들도 단순화하는데, 이는 모델이 가능한 모든 보안 취약점을 완전히 설명할 수 없다는 것을 의미합니다.

예를 들어, 우리는 HTTP 요청을 원격 프로시저 호출처럼 다루며, 요청에 대한 응답이 순서대로 오지 않을 수 있다는 사실을 무시합니다. 또한 DNS(도메인 네임 서비스)가 정적이라고 가정하므로, 상호작용 중에 DNS 바인딩이 변경되는 공격은 고려할 수 없습니다. 하지만 원칙적으로는 이러한 모든 측면을 다루도록 모델을 확장하는 것이 가능할 것입니다. 다만 보안 분석의 본질상 어떤 모델도 (전체 코드베이스를 표현하더라도) 완전하다고 보장될 수는 없습니다.

## 로드맵

다음은 SOP 모델을 진행할 순서입니다. 우리는 SOP에 대해 논의하기 위해 필요한 세 가지 핵심 구성요소의 모델을 구축하는 것으로 시작할 것입니다: HTTP, 브라우저, 그리고 클라이언트 측 스크립팅입니다. 이러한 기본 모델들을 바탕으로 웹 애플리케이션이 _안전하다_는 것이 무엇을 의미하는지 정의하고, 그런 다음 필요한 보안 속성들을 달성하려고 시도하는 메커니즘으로서 SOP를 도입할 것입니다.

그 후 SOP가 때때로 너무 제한적이어서 웹 애플리케이션의 적절한 기능에 방해가 될 수 있음을 보게 될 것입니다. 따라서 정책에 의해 부과되는 제한을 우회하기 위해 일반적으로 사용되는 네 가지 다른 기법들을 소개할 것입니다.

원하는 순서대로 자유롭게 섹션들을 탐색해보세요. Alloy를 처음 접한다면, 모델링 언어의 기본 개념들을 소개하는 처음 세 섹션(HTTP, 브라우저, 스크립트)부터 시작하는 것을 권합니다. 이 장을 진행하는 동안, Alloy 분석기에서 모델들을 가지고 놀아보기를 권장합니다; 실행해보고, 생성된 시나리오들을 탐색하고, 수정을 해보면서 그 효과를 확인해보세요. Alloy 분석기는 [무료로 다운로드할 수 있습니다](http://alloy.mit.edu).

## 웹의 모델

### HTTP

Alloy 모델을 구축하는 첫 번째 단계는 객체들의 집합을 선언하는 것입니다. 리소스부터 시작해보겠습니다:

```alloy
sig Resource {}
```

키워드 `sig`는 이것이 Alloy _시그니처_ 선언임을 나타냅니다. 이것은 리소스 객체들의 집합을 도입합니다. 인스턴스 변수가 없는 클래스의 객체들처럼, 이들을 정체성은 있지만 내용은 없는 덩어리로 생각하세요. 분석이 실행될 때 이 집합이 결정될 것이며, 이는 객체지향 언어에서 프로그램이 실행될 때 클래스가 객체들의 집합을 나타내게 되는 것과 같습니다.

리소스들은 URL(*uniform resource locators*)로 명명됩니다:

```alloy
sig Url {
  protocol: Protocol,
  host: Domain,
  port: lone Port,
  path: Path
}
sig Protocol, Port, Path {}
sig Domain { subsumes: set Domain }
```

여기서 우리는 다섯 개의 시그니처 선언을 가지고 있으며, URL들의 집합과 그것들을 구성하는 기본 객체 종류들 각각에 대한 네 개의 추가 집합을 도입하고 있습니다. URL 선언 내에서 우리는 네 개의 _필드_를 가지고 있습니다. 필드는 클래스의 인스턴스 변수와 같습니다. 예를 들어, `u`가 URL이라면 `u.protocol`은 그 URL의 프로토콜을 나타낼 것입니다(Java의 점 연산자와 같이). 하지만 사실, 나중에 보겠지만 이러한 필드들은 관계입니다. 각각을 두 열로 된 데이터베이스 테이블처럼 생각할 수 있습니다. 따라서 `protocol`은 첫 번째 열에 URL들을, 두 번째 열에 프로토콜들을 포함하는 테이블입니다. 그리고 무해해 보이는 점 연산자는 사실 상당히 일반적인 종류의 관계형 조인이므로, 프로토콜 `p`를 가진 모든 URL에 대해 `protocol.p`라고 쓸 수도 있습니다—하지만 이에 대한 자세한 내용은 나중에 설명하겠습니다.

URL과 달리 경로는 구조가 없는 것으로 취급된다는 점에 주목하세요—이것은 단순화입니다. 키워드 `lone`("1 이하"로 읽을 수 있음)은 각 URL이 최대 하나의 포트를 가진다는 것을 의미합니다. 경로는 URL에서 호스트 이름 뒤에 오는 문자열로, (간단한 정적 서버의 경우) 리소스의 파일 경로에 해당합니다. 우리는 경로가 항상 존재한다고 가정하지만, 빈 경로일 수도 있습니다.

경로에서 리소스로의 매핑을 포함하는 클라이언트와 서버를 도입해보겠습니다:

```alloy
abstract sig Endpoint {}
abstract sig Client extends Endpoint {}
abstract sig Server extends Endpoint {
  resources: Path -> lone Resource
}
```

`extends` 키워드는 부분집합을 도입하므로, 예를 들어 모든 클라이언트의 집합인 `Client`는 모든 엔드포인트의 집합인 `Endpoint`의 부분집합입니다. 확장은 서로소이므로, 어떤 엔드포인트도 클라이언트이면서 동시에 서버일 수는 없습니다. `abstract` 키워드는 시그니처의 모든 확장이 그것을 완전히 포함한다는 것을 의미하므로, 예를 들어 `Endpoint` 선언에서의 이 키워드는 모든 엔드포인트가 부분집합 중 하나(이 시점에서는 `Client`와 `Server`)에 속해야 한다는 것을 의미합니다. 서버 `s`에 대해 표현식 `s.resources`는 경로에서 리소스로의 맵을 나타낼 것입니다(따라서 선언에서 화살표가 있습니다). 각 필드는 실제로 소유하는 시그니처를 첫 번째 열로 포함하는 관계라는 것을 기억하세요. 따라서 이 필드는 `Server`, `Path`, `Resource`에 대한 세 열 관계를 나타냅니다.

URL을 서버에 매핑하기 위해, 도메인에서 서버로의 매핑을 가진 도메인 네임 서버들의 집합 `Dns`를 도입합니다:

```alloy
one sig Dns {
  map: Domain -> Server
}
```

시그니처 선언에서 키워드 `one`은 (단순함을 위해) 정확히 하나의 도메인 네임 서버가 있다고 가정한다는 것을 의미하며, `Dns.map` 표현식으로 주어지는 단일 DNS 매핑이 있을 것입니다. 서빙 리소스와 마찬가지로, 이것도 동적일 수 있지만(실제로 상호작용 중에 DNS 바인딩 변경에 의존하는 알려진 보안 공격이 있습니다) 우리는 단순화하고 있습니다.

HTTP 요청을 모델링하기 위해서는 _쿠키_ 개념도 필요하므로, 이를 선언해보겠습니다:

```alloy
sig Cookie {
  domains: set Domain
}
```

각 쿠키는 도메인 집합으로 범위가 지정됩니다. 이는 쿠키가 `*.mit.edu`에 적용될 수 있다는 사실을 포착하는 것으로, `mit.edu` 접미사를 가진 모든 도메인을 포함할 것입니다.

마지막으로, 이 모든 것을 종합하여 HTTP의 모델을 구축할 수 있습니다
requests:

```alloy
abstract sig HttpRequest extends Call {
  url: Url,
  sentCookies: set Cookie,
  body: lone Resource,
  receivedCookies: set Cookie,
  response: lone Resource,
}{
  from in Client
  to in Dns.map[url.host]
}
```

We're modeling an HTTP request and response in a single object; the
`url`, `sentCookies` and `body` are sent by the client, and the
`receivedCookies` and `response` are sent back by the server.

When writing the `HttpRequest` signature, we found that it contained
generic features of calls, namely that they are from and to particular
things. So we actually wrote a little Alloy module that declares the
`Call` signature, and to use it here we need to import it:

```alloy
open call[Endpoint]
```

It's a polymorphic module, so it's instantiated with `Endpoint`, the
set of things calls are from and to. (The module appears in full in
 \aosasecref{500l.sop.appendix}.)

Following the field declarations in `HttpRequest` is a collection of
constraints. Each of these constraints applies to all members of the
set of HTTP requests. The constraints say that (1) each request comes
from a client, and (2) each request is sent to one of the servers
specified by the URL host under the DNS mapping.

One of the prominent features of Alloy is that a model, no matter how
simple or detailed, can be executed at any time to generate sample
system instances. Let's use the `run` command to ask the Alloy
Analyzer to execute the HTTP model that we have so far:

```alloy
run {} for 3	-- generate an instance with up to 3 objects of every signature type
```

As soon as the analyzer finds a possible instance of the system, it
automatically produces a diagram of the instance, like in \aosafigref{500l.same-origin-policy.fig-http-1}.

\aosafigure[240pt]{same-origin-policy-images/fig-http-1.png}{A possible instance}{500l.same-origin-policy.fig-http-1}

This instance shows a client (represented by node `Client`) sending an
`HttpRequest` to `Server`, which, in response, returns a resource
object and instructs the client to store `Cookie` at `Domain`.

Even though this is a tiny instance with seemingly few details, it
signals a flaw in our model. Note that the resource returned from the
request (`Resource1`) does not exist in the server. We neglected to
specify an obvious fact about the server; namely, that every response
to a request is a resource that the server stores. We can go back to
our definition of `HttpRequest` and add a constraint:

```alloy
abstract sig HttpRequest extends Call { ... }{
  ...
  response = to.resources[url.path]
}
```

이제 다시 실행하면 결함이 없는 인스턴스들이 생성됩니다.

샘플 인스턴스를 생성하는 대신, 분석기에게 모델이 속성을 만족하는지
*검사*하도록 요청할 수 있습니다. 예를 들어, 클라이언트가 동일한 요청을
여러 번 보낼 때 항상 동일한 응답을 받는다는 속성을 원할 수 있습니다:

```alloy
check { 
    all r1, r2: HttpRequest | r1.url = r2.url implies r1.response = r2.response 
} for 3 
```

이 `check` 명령이 주어지면, 분석기는 (명시된 범위까지) 시스템의 모든 가능한 동작을 탐색하고, 속성을 위반하는 것을 찾으면 그 인스턴스를 \aosafigref{500l.same-origin-policy.fig-http-2a}와 \aosafigref{500l.same-origin-policy.fig-http-2b}에서 보여지는 바와 같이 *반례*로 표시합니다.

\aosafigure[180pt]{same-origin-policy-images/fig-http-2a.png}{Counterexample at time 0}{500l.same-origin-policy.fig-http-2a}

\aosafigure[180pt]{same-origin-policy-images/fig-http-2b.png}{Counterexample at time 1}{500l.same-origin-policy.fig-http-2b}

This counterexample again shows an HTTP request being made by a
client, but with two different servers. (In the Alloy visualizer,
objects of the same type are distinguished by appending numeric
suffixes to their names; if there is only one object of a given type,
no suffix is added. Every name that appears in a snapshot diagram is
the name of an object. So&mdash;perhaps confusingly at first sight&mdash;the
names `Domain`, `Path`, `Resource`, `Url` all refer to individual
objects, not to types.)

Note that while the DNS maps `Domain` to both `Server0` and
`Server1` (in reality, this is a common practice for load balancing),
only `Server1` maps `Path` to a resource object, causing
`HttpRequest1` to result in an empty response: another error in our
model. To fix this, we add an Alloy *fact* recording 
that any two servers to which DNS maps a single host provide the same
set of resources:

```alloy
fact ServerAssumption {
  all s1, s2: Server | 
    (some Dns.map.s1 & Dns.map.s2) implies s1.resources = s2.resources
}
```

When we re-run the `check` command after adding this fact, the
analyzer no longer reports any counterexamples for the property. This
doesn't mean the property has been proven to be true, since there
might be a counterexample in a larger scope. But it is unlikely that
the property is false, since the analyzer has tested all possible
instances involving 3 objects of each type.

하지만 원한다면, 신뢰도를 높이기 위해 더 큰 범위로 분석을 다시 실행할 수 있습니다. 예를 들어, 위의 확인을 범위 10으로 실행해도 여전히 반례가 생성되지 않으므로, 이 속성이 유효할 가능성이 높다는 것을 시사합니다. 하지만 더 큰 범위가 주어지면, 분석기는 더 많은 수의 인스턴스를 테스트해야 하므로 완료하는 데 더 오래 걸릴 것이라는 점을 명심하세요.

### Browser

Let's now introduce browsers into our model:

```alloy
sig Browser extends Client {
  documents: Document -> Time,
  cookies: Cookie -> Time,
}
```

This is our first example of a signature with *dynamic fields*. Alloy
has no built-in notions of time or behavior, which means that a
variety of idioms can be used. In this model, we're using a common
idiom in which you introduce a notion of `Time`, and attach it as a
final column for every time-varying field. For example, the expression
`b.cookies.t` represents the set of cookies that are stored in browser
`b` at a particular time `t`. Likewise, the `documents` field associates
a set of documents with each browser at a given time. (For more details
about how we model the dynamic behavior, see \aosasecref{500l.sop.appendix}.)

Documents are created from a response to an HTTP request. They can
also be destroyed if, for example, the user closes a tab or the
browser, but we leave this out of the model. A document has a URL
(the one from which the document was originated), some content (the
DOM), and a domain:

```alloy
sig Document {
  src: Url,
  content: Resource -> Time,
  domain: Domain -> Time
}
```

The inclusion of the `Time` column for the latter two fields tells us
that they can vary over time, and its omission for the first (`src`,
representing the source URL of the document) indicates that the source
URL은 고정되어 있습니다.

To model the effect of an HTTP request on a browser, we introduce a
new signature, since not all HTTP requests will originate at the level
of the browser; the rest will come from scripts.

```alloy
sig BrowserHttpRequest extends HttpRequest {
  doc: Document
}{
  -- the request comes from a browser
  from in Browser
  -- the cookies being sent exist in the browser at the time of the request
  sentCookies in from.cookies.start
  -- every cookie sent must be scoped to the url of the request
  all c: sentCookies | url.host in c.domains

  -- a new document is created to display the content of the response
  documents.end = documents.start + from -> doc
  -- the new document has the response as its contents
  content.end = content.start ++ doc -> response
  -- the new document has the host of the url as its domain
  domain.end = domain.start ++ doc -> url.host
  -- the document's source field is the url of the request
  doc.src = url

  -- new cookies are stored by the browser
  cookies.end = cookies.start + from -> sentCookies
}
```

This kind of request has one new field, `doc`, representing the
document created in the browser from the resource returned by the
request. As with `HttpRequest`, the behavior is described as a
collection of constraints. Some of these say when the call can happen:
for example, that the call has to come from a browser. Some constrain
the arguments of the call: for example, that the cookies must be
scoped appropriately. And some constrain the effect, using a common
idiom that relates the value of a relation after the call to its value
before.

For example, to understand the constraint `documents.end =
documents.start + from -> doc` remember that `documents` is a 3-column
relation on browsers, documents and times. The fields `start` and
`end` come from the declaration of `Call` (which we haven't seen, but
is included in the listing at the end), and represent the times at the
beginning and end of the call. The expression `documents.end` gives
the mapping from browsers to documents when the call has ended. So
this constraint says that after the call, the mapping is the same,
except for a new entry in the table mapping `from` to `doc`.

일부 제약조건은 `++` 관계형 _재정의_ 연산자를 사용합니다: `e1 ++ e2`는
`e2`의 모든 튜플을 포함하고, 추가로 첫 번째 요소가 `e2`의 튜플의
첫 번째 요소가 아닌 `e1`의 튜플들을 포함합니다. 예를 들어,
`content.end = content.start ++ doc -> response` 제약조건은
호출 후 `content` 매핑이 `doc`을 `response`에 매핑하도록
업데이트된다는 것을 의미합니다(기존 `doc` 매핑을 재정의).
만약 합집합 연산자 `+`를 대신 사용한다면,
same document might (incorrectly) be mapped to multiple resources in
the after state.

### Script

Next, we will build on the HTTP and browser models to introduce *client-side scripts*, which represent pieces of code (typically in JavaScript) executing inside a browser document (`context`). 

```alloy
sig Script extends Client { context: Document }
```

스크립트는 두 가지 종류의 작업을 수행할 수 있는 동적 엔티티입니다:
(1) HTTP 요청(즉, Ajax 요청)을 보낼 수 있고 (2) 문서의 콘텐츠와
속성을 조작하는 브라우저 작업을 수행할 수 있습니다. 클라이언트 측
스크립트의 유연성은 웹 2.0의 급속한 발전의 주요 촉매제 중 하나이지만,
SOP가 처음 만들어진 이유이기도 합니다. SOP가 없다면 스크립트는
서버에 임의의 요청을 보내거나 브라우저 내의 문서를 자유롭게
수정할 수 있을 것입니다&mdash;하나 이상의 스크립트가 악성으로
판명될 경우 이는 나쁜 소식이 될 것입니다.

스크립트는 `XmlHttpRequest`를 보내서 서버와 통신할 수 있습니다:
sig XmlHttpRequest extends HttpRequest {}{
  from in Script
  noBrowserChange[start, end] and noDocumentChange[start, end]
}
```

An `XmlHttpRequest` can be used by a script to send/receive resources
to/from a server, but unlike `BrowserHttpRequest`, it does not
immediately result in the creation of a new page or other changes to the
browser and its documents. To say that a call does not modify these
aspects of the system, we define predicates `noBrowserChange` and
`noDocumentChange`:

```alloy
pred noBrowserChange[start, end: Time] {
  documents.end = documents.start and cookies.end = cookies.start  
}
pred noDocumentChange[start, end: Time] {
  content.end = content.start and domain.end = domain.start  
}
```

What kind of operations can a script perform on documents? First, we
introduce a generic notion of *browser operations* to represent a set
of browser API functions that can be invoked by a script:

```alloy
abstract sig BrowserOp extends Call { doc: Document }{
  from in Script and to in Browser
  doc + from.context in to.documents.start
  noBrowserChange[start, end]
}
```

Field `doc` refers to the document that will be accessed or
manipulated by this call. The second constraint in the signature facts
says that both `doc` and the document in which the script executes
(`from.context`) must be documents that currently exist inside the
browser. Finally, a `BrowserOp` may modify the state of a document,
but not the set of documents or cookies that are stored in the
browser. (Actually, cookies can be associated with a document and
modified using a browser API, but we omit this detail for now.)

스크립트는 문서의 다양한 부분(보통 DOM 요소라고 함)을 읽고
쓸 수 있습니다. 일반적인 브라우저에는 DOM에 접근하기 위한
많은 API 함수(예: `document.getElementById`)가 있지만,
우리의 목적을 위해 그것들을 모두 열거하는 것은 중요하지 않습니다.
대신, 우리는 그것들을 두 종류&mdash;`ReadDom`과 `WriteDom`&mdash;로
간단히 그룹화하고, 수정을 전체 문서의 도매 교체로 모델링할 것입니다:
```alloy
sig ReadDom extends BrowserOp { result: Resource }{
  result = doc.content.start
  noDocumentChange[start, end]
}
sig WriteDom extends BrowserOp { newDom: Resource }{
  content.end = content.start ++ doc -> newDom
  domain.end = domain.start
}
```

`ReadDom` returns the content of the target document, but does not modify it; `WriteDom`, on the other hand, sets the new content of the target document to `newDom`.

In addition, a script can modify various properties of a document,
such as its width, height, domain, and title. For our discussion of
the SOP, we are only interested in the domain property, which we will
introduce in a later section.

## Example Applications

As we've seen earlier, given a `run` or `check` command, the Alloy
Analyzer generates a scenario (if it exists) that is consistent with
the description of the system in the model. By default, the analyzer
arbitrarily picks _any_ one of the possible system scenarios (up to
the specified bound), and assigns numeric identifiers to signature
instances (`Server0`, `Browser1`, etc.) in the scenario.

때때로, 무작위 서버와 클라이언트 구성으로 시나리오를 탐색하는 대신
_특정_ 웹 애플리케이션의 동작을 분석하고 싶을 수 있습니다.
예를 들어, 브라우저 내에서 실행되는 이메일 애플리케이션(Gmail과 같은)을
구축하고 싶다고 상상해 보세요. 기본적인 이메일 기능을 제공하는 것 외에도,
우리의 애플리케이션은 잠재적으로 악의적인 행위자가 제어하는
제3자 광고 서비스의 배너를 표시할 수 있습니다.
In Alloy, the keywords `one sig` introduce a _singleton_ signature
containing exactly one object; we saw an example above with
`Dns`. This syntax can be used to specify concrete atoms. For example,
to say that there is one inbox page and one ad banner (each of which
is a document) we can write:

```alloy
one sig InboxPage, AdBanner extends Document {}
```

With this declaration, every scenario that Alloy generates will
contain at least these two `Document` objects.

Likewise, we can specify particular servers, domains and so on, with a
constraint (which we've called `Configuration`) to specify the
relationships between them:

```alloy
one sig EmailServer, EvilServer extends Server {}
one sig EvilScript extends Script {}
one sig EmailDomain, EvilDomain extends Domain {}
fact Configuration {
  EvilScript.context = AdBanner
  InboxPage.domain.first = EmailDomain
  AdBanner.domain.first = EvilDomain  
  Dns.map = EmailDomain -> EmailServer + EvilDomain -> EvilServer
}
```

For example, the last constraint in the fact specifies how the DNS is
configured to map domain names for the two servers in our
system. Without this constraint, the Alloy Analyzer may generate
scenarios where `EmailDomain` is mapped to `EvilServer`, which are not
of interest to us. (In practice, such a mapping may be possible due to
an attack called _DNS spoofing_, but we will rule it out from our
model since it lies outside the class of attacks that the SOP is
designed to prevent.)

Let us introduce two additional applications: an online calendar and a
blog site:

```alloy
one sig CalendarServer, BlogServer extends Document {} 
one sig CalendarDomain, BlogDomain extends Domain {}
```

We should update the constraint about the DNS mapping above to
incorporate the domain names for these two servers:

```alloy
fact Configuration {
  ...
  Dns.map = EmailDomain -> EmailServer + EvilDomain -> EvilServer + 
            CalendarDomain -> CalendarServer + BlogDomain -> BlogServer  
}
```

In addition, let us say that that the email, blog, and calendar applications
are all developed by a single organization, and thus, share the same base
domain name. Conceptually, we can think of `EmailServer` and `CalendarServer`
having subdomains `email` and `calendar`, sharing `example.com` as the common
superdomain. In our model, this can be represented by introducing a domain name
that _subsumes_ others:

```alloy 
one sig ExampleDomain extends Domain {}{
  subsumes = EmailDomain + EvilDomain + CalendarDomain + this
}   
```

Note that `this` is included as a member of `subsumes`, since every
domain name subsumes itself.

이러한 애플리케이션에 대한 다른 세부 사항은 여기서 생략합니다
(전체 모델은 `example.als` 참조). 하지만 이 장의 나머지 부분에서
이 애플리케이션들을 계속 예제로 사용할 것입니다.

## Security Properties

Before we get to the SOP itself, there is an important question that we
have not discussed yet: What exactly do we mean when we say our system
is _secure_?

Not surprisingly, this is a tricky question to answer. For our
purposes, we will turn to two well-studied concepts in information
security&mdash;_confidentiality_ and _integrity_. Both of these concepts
talk about how information should be allowed to pass through the
various parts of the system. Roughly, _confidentiality_ means that a
critical piece of data should only be accessible to parts that are
deemed trusted, and _integrity_ means that trusted parts only rely on
data that have not been maliciously tampered with.

### Dataflow Properties

In order to specify these security properties more precisely, we first
need to define what it means for a piece of data to _flow_ from one
part of the system to another. In our model so far, we have described
interactions between two endpoints as being carried out through
_calls_; e.g., a browser interacts with a server by making HTTP
requests, and a script interacts with the browser by invoking browser
API operations. Intuitively, during each call, a piece of data may flow
from one endpoint to another as an _argument_ or _return value_ of the
call. To represent this, we introduce a notion of `DataflowCall` into the
model, and associate each call with a set of `args` and `returns` data
fields:

```alloy
sig Data in Resource + Cookie {}

sig DataflowCall in Call {
  args, returns: set Data,  --- arguments and return data of this call
}{
 this in HttpRequest implies
    args = this.sentCookies + this.body and
    returns = this.receivedCookies + this.response
 ...
}
```

For example, during each call of type `HttpRequest`, the client
transfers `sentCookies` and `body` to the server, and
receives `receivedCookies` and `response` as return values. 

More generally, arguments flow from the sender of the call to the
receiver, and return values flow from the receiver to the sender. This
means that the only way for an endpoint to access a new piece of data
is by receiving it as an argument of a call that the endpoint accepts,
or a return value of a call that the endpoint invokes. We introduce a
notion of `DataflowModule`, and assign field `accesses` to represent the
set of data elements that the module can access at each time step:

```alloy
sig DataflowModule in Endpoint {
  -- Set of data that this component initially owns
  accesses: Data -> Time
}{
  all d: Data, t: Time - first |
	 -- This endpoint can only access a piece of data "d" at time "t" only when
    d -> t in accesses implies
      -- (1) It already had access in the previous time step, or
      d -> t.prev in accesses or
      -- there is some call "c" that ended at "t" such that
      some c: Call & end.t |
        -- (2) the endpoint receives "c" that carries "d" as one of its arguments or
        c.to = this and d in c.args or
        -- (3) the endpoint sends "c" that returns "d" 
        c.from = this and d in c.returns 
}
```

또한 모듈이 호출의 인수나 반환 값으로 제공할 수 있는 데이터 요소들을 제한할 필요가 있습니다. 그렇지 않으면, 모듈이 접근할 수 없는 인수로 호출을 만들 수 있는 이상한 시나리오가 발생할 수 있습니다.

```alloy
sig DataflowCall in Call { ... } {
  -- (1) Any arguments must be accessible to the sender
  args in from.accesses.start
  -- (2) Any data returned from this call must be accessible to the receiver
  returns in to.accesses.start
}
```

Now that we have means to describe data flow between different parts
of the system, we are (almost) ready to state security properties that
we care about. But recall that confidentiality and integrity are
_context-dependent_ notions; these properties make sense only if we
can talk about some agents within the system as being trusted (or
malicious). Similarly, not all information is equally important: we
need to distinguish between data elements that we consider to be
critical or malicious (or neither):

```alloy
sig TrustedModule, MaliciousModule in DataflowModule {}
sig CriticalData, MaliciousData in Data {}
```

그러면 기밀성 속성은 시스템의 신뢰할 수 없는 부분으로
중요 데이터가 흐르는 것에 대한 _단언_으로 표현할 수 있습니다:

```alloy
// No malicious module should be able to access critical data
assert Confidentiality {
  no m: Module - TrustedModule, t: Time |
    some CriticalData & m.accesses.t 
}
```

The integrity property is the dual of confidentiality: 

```alloy
// No malicious data should ever flow into a trusted module
assert Integrity {
  no m: TrustedModule, t: Time | 
    some MaliciousData & m.accesses.t
}
```

### Threat Model

위협 모델은 공격자가 시스템의 보안 속성을 침해하려는 시도에서
수행할 수 있는 일련의 행동을 설명합니다. 위협 모델을 구축하는 것은
모든 보안 시스템 설계에서 중요한 단계입니다; 이를 통해 시스템과
환경에 대해 우리가 가진 (잠재적으로 잘못된) 가정을 식별하고,
완화해야 할 다양한 유형의 위험에 우선순위를 매길 수 있습니다.
script or a client. As a server, the attacker may set up malicious web
pages to solicit visits from unsuspecting users, who, in turn, may
inadvertently send sensitive information to the attacker as part of a
HTTP request. The attacker may create a malicious script that invokes
DOM operations to read data from other pages and relays those data to
the attacker's server. Finally, as a client, the attacker may
impersonate a normal user and send malicious requests to a server in
an attempt to access the user's data. We do not consider attackers
that eavesdrop on the connection between different network endpoints;
although it is a threat in practice, the SOP is not
designed to prevent it, and thus it lies outside the scope of our model.

### Checking Properties

Now that we have defined the security properties and the attacker's
behavior, let us show how the Alloy Analyzer can be used to
automatically check that those properties hold even in the presence of
the attacker.  When prompted with a `check` command, the analyzer
explores _all_ possible dataflow traces in the system and produces a
counterexample (if one exists) that demonstrates how an assertion
might be violated:

```
check Confidentiality for 5
```

For example, when checking the model of our example application against the
confidentiality property, the analyzer generates the scenario seen in
\aosafigref{500l.same-origin-policy.fig-attack-1a} and
\aosafigref{500l.same-origin-policy.fig-attack-1b}, which shows how
`EvilScript` may access a piece of critical data (`MyInboxInfo`).

\aosafigure[180pt]{same-origin-policy-images/fig-attack-1a.png}{Confidentiality counterexample at time 0}{500l.same-origin-policy.fig-attack-1a}
\aosafigure[180pt]{same-origin-policy-images/fig-attack-1b.png}{Confidentiality counterexample at time 1}{500l.same-origin-policy.fig-attack-1b}

이 반례는 두 단계를 포함합니다. 첫 번째 단계(\aosafigref{500l.same-origin-policy.fig-attack-1a})에서, `EvilDomain`의 `AdBanner` 내부에서 실행되는 `EvilScript`는 `EmailDomain`에서 유래한 `InboxPage`의 콘텐츠를 읽습니다. 다음 단계(\aosafigref{500l.same-origin-policy.fig-attack-1b})에서, `EvilScript`는 `XmlHttpRequest` 호출을 만들어 동일한 콘텐츠(`MyInboxInfo`)를 `EvilServer`에 보냅니다. 여기서 문제의 핵심은 한 도메인 하에서 실행되는 스크립트가 다른 도메인의 문서 콘텐츠를 읽을 수 있다는 것입니다; 다음 섹션에서 보게 되겠지만, 이는 정확히 SOP가 방지하도록 설계된 시나리오 중 하나입니다.

단일 단언에 대해 여러 반례가 있을 수 있습니다. 시스템이 기밀성 속성을 위반할 수 있는 다른 방법을 보여주는 \aosafigref{500l.same-origin-policy.fig-attack-2}를 고려해보세요.

\aosafigure[180pt]{same-origin-policy-images/fig-attack-2.png}{Another confidentiality violation}{500l.same-origin-policy.fig-attack-2}

In this scenario, instead of reading the content of the inbox page,
`EvilScript` directly makes a `GetInboxInfo` request to `EmailServer`.
Note that the request includes a cookie (`MyCookie`), which is scoped
to the same domain as the destination server. This is potentially
dangerous, because if the cookie is used to represent the user's
identity (e.g., a session cookie), `EvilScript` can effectively
pretend to be the user and trick the server into responding with the
user's private data (`MyInboxInfo`). Here, the problem is again
related to the liberal ways in which a script may be used to access
information across different domains&mdash;namely, that a script executing
under one domain is able to make an HTTP request to a server with a
different domain.

이 두 반례는 스크립트의 동작을 제한하기 위해 추가적인 조치가
필요하다는 것을 알려줍니다, 특히 일부 스크립트가 악의적일 수 있기
때문입니다. 이것이 바로 SOP가 등장하는 이유입니다.
## Same-Origin Policy

Before we can state the SOP, the first thing we should do is to introduce the
notion of an _origin_, which is composed of a protocol, host, and optional port:

```alloy
sig Origin {
  protocol: Protocol,
  host: Domain,
  port: lone Port
}
```

For convenience, let us define a function that, given a URL, returns the corresponding origin:

```alloy
fun origin[u: Url] : Origin {
    {o: Origin | o.protocol = u.protocol and o.host = u.host and o.port = u.port }
}
```
The SOP itself has two parts, restricting the ability of a script to (1) make DOM API calls and (2) send HTTP requests. The first part of the policy states that a script can only read from and write to a document that comes from the same origin as the script:
```alloy
fact domSop {
  all o: ReadDom + WriteDom |  let target = o.doc, caller = o.from.context |
    origin[target] = origin[caller] 
}
```
첫 번째 스크립트 시나리오(이전 섹션의)와 같은 인스턴스는 `domSop` 하에서는 불가능합니다. `Script`가 다른 출처의 문서에 대해 `ReadDom`을 호출하는 것이 허용되지 않기 때문입니다.

정책의 두 번째 부분은 스크립트가 그 컨텍스트가 대상 URL과 동일한 출처를 가지지 않는 한 서버에 HTTP 요청을 보낼 수 없다고 말합니다—두 번째 스크립트 시나리오와 같은 인스턴스들을 효과적으로 방지합니다.
```alloy
fact xmlHttpReqSop { 
  all x: XmlHttpRequest | origin[x.url] = origin[x.from.context.src] 
}
```
보다시피, SOP는 악의적인 스크립트의 행동으로부터 발생할 수 있는 두 가지 유형의 취약점을 방지하도록 설계되었습니다; 그것 없이는 웹이 오늘날보다 훨씬 더 위험한 곳이 될 것입니다.

It turns out, however, that the SOP can be *too* restrictive. For
example, sometimes you *do* want to allow communication between two
documents of different origins. By the above definition of an origin,
a script from `foo.example.com` would not be able to read the content
of `bar.example.com`, or send a HTTP request to `www.example.com`,
because these are all considered distinct hosts.

In order to allow some form of cross-origin communication when
necessary, browsers implemented a variety of mechanisms for relaxing
the SOP. Some of these are more well-thought-out than others, and some
have pitfalls that, when badly used, can undermine the security
benefits of the SOP. In the following sections, we will describe the
most common of these mechanisms, and discuss their potential security
pitfalls.

## SOP 우회 기법들

SOP는 기능성과 보안 사이의 긴장 관계를 보여주는 전형적인 예입니다. 우리는 사이트가 견고하고 기능적이기를 원하지만, 보안을 위한 메커니즘이 때로는 방해가 될 수 있습니다. 실제로 SOP가 처음 도입되었을 때, 개발자들은 도메인 간 통신을 합법적으로 사용하는 사이트(예: 매시업)를 구축하는 데 어려움을 겪었습니다.

이 섹션에서는 웹 개발자들이 SOP에 의해 부과된 제한을 우회하기 위해 고안하고 자주 사용하는 네 가지 기법에 대해 논의할 것입니다: (1) `document.domain` 속성 완화; (2) JSONP; (3) PostMessage; (4) CORS. 이들은 유용한 도구들이지만, 주의 없이 사용된다면 웹 애플리케이션을 SOP가 애초에 방지하도록 설계된 바로 그 종류의 공격에 취약하게 만들 수 있습니다.

이 네 가지 기법 각각은 놀랍도록 복잡하며, 완전히 자세하게 설명한다면 각각 별도의 장이 필요할 것입니다. 따라서 여기서는 이들이 어떻게 작동하는지, 이들이 도입하는 잠재적 보안 문제들, 그리고 이러한 문제들을 방지하는 방법에 대한 간략한 인상만 제공합니다. 특히, 각 기법에 대해 공격자가 앞서 정의한 두 보안 속성을 훼손하기 위해 악용할 수 있는지 Alloy 분석기에 확인을 요청할 것입니다:

```
check Confidentiality for 5
check Integrity for 5
```

분석기가 생성하는 반례들로부터 얻은 통찰력을 바탕으로, 보안 함정에 빠지지 않고 이러한 기법들을 안전하게 사용하기 위한 가이드라인을 논의할 것입니다.

### Domain Property

목록의 첫 번째 기법으로, SOP를 우회하는 방법으로 `document.domain` 속성의 사용을 살펴보겠습니다. 이 기법의 아이디어는 서로 다른 출처의 두 문서가 단순히 `document.domain` 속성을 동일한 값으로 설정함으로써 서로의 DOM에 접근할 수 있도록 하는 것입니다. 예를 들어, `email.example.com`의 스크립트가 두 문서의 스크립트가 모두 `document.domain` 속성을 `example.com`으로 설정한다면 (두 소스 URL이 동일한 프로토콜과 포트를 가진다고 가정) `calendar.example.com`의 문서 DOM을 읽거나 쓸 수 있습니다.

우리는 `document.domain` 속성을 설정하는 동작을 `SetDomain`이라는 브라우저 연산의 한 유형으로 모델링합니다:

```alloy
// Modify the document.domain property
sig SetDomain extends BrowserOp { newDomain: Domain }{
  doc = from.context
  domain.end = domain.start ++ doc -> newDomain
  -- no change to the content of the document
  content.end = content.start
}
```

`newDomain` 필드는 속성이 설정되어야 할 값을 나타냅니다. 하지만 주의할 점이 있습니다: 스크립트는 호스트명의 우측, 완전히 정규화된 조각에만 도메인 속성을 설정할 수 있습니다. (즉, `email.example.com`은 `example.com`으로는 설정할 수 있지만 `google.com`으로는 설정할 수 없습니다.) 우리는 서브도메인에 대한 이 규칙을 포착하기 위해 팩트를 사용합니다:

```alloy
// Scripts can only set the domain property to only one that is a right-hand,
// fully-qualified fragment of its hostname
fact setDomainRule {
  all d: Document | d.src.host in (d.domain.Time).subsumes
}
```

이 규칙이 없다면, 어떤 사이트든 `document.domain` 속성을 임의의 값으로 설정할 수 있으며, 이는 예를 들어 악의적인 사이트가 도메인 속성을 여러분의 은행 도메인으로 설정하고, 은행 계정을 iframe에 로드한 다음, (은행 페이지가 도메인 속성을 설정했다고 가정하고) 여러분의 은행 페이지의 DOM을 읽을 수 있다는 것을 의미합니다.

`document.domain` 속성의 효과를 고려하기 위해 DOM 접근에 대한 제한을 완화하여 SOP의 원래 정의로 돌아가봅시다. 두 스크립트가 속성을 동일한 값으로 설정하고, 동일한 프로토콜과 포트를 가진다면, 이 두 스크립트는 서로 상호작용할 수 있습니다 (즉, 서로의 DOM을 읽고 쓸 수 있습니다).

```alloy
fact domSop {
  -- For every successful read/write DOM operation,
  all o: ReadDom + WriteDom |  let target = o.doc, caller = o.from.context |
    -- (1) target and caller documents are from the same origin, or
    origin[target] = origin[caller] or
    -- (2) domain properties of both documents have been modified
    (target + caller in (o.prevs <: SetDomain).doc and
      -- ...and they have matching origin values.
      currOrigin[target, o.start] = currOrigin[caller, o.start])
}
```

여기서 `currOrigin[d, t]`는 시간 `t`에서 `document.domain` 속성을 호스트명으로 하는 문서 `d`의 출처를 반환하는 함수입니다.

_두_ 문서의 `document.domain` 속성이 모두 브라우저에 로드된 후 어느 시점에 _명시적으로_ 설정되어야 한다는 점을 지적할 가치가 있습니다. 문서 A가 `example.com`에서 로드되고, `calendar.example.com`의 문서 B가 도메인 속성을 `example.com`으로 수정했다고 가정해봅시다. 비록 두 문서가 이제 동일한 도메인 속성을 가지지만, 문서 A도 명시적으로 속성을 `example.com`으로 설정하지 않는 한 서로 상호작용할 수 _없을_ 것입니다. 처음에는 이것이 다소 이상한 동작처럼 보입니다. 하지만 이것 없이는 다양한 나쁜 일들이 일어날 수 있습니다. 예를 들어, 사이트가 서브도메인으로부터 사이트 간 스크립팅 공격을 받을 수 있습니다: 문서 B의 악의적인 스크립트가 도메인 속성을 `example.com`으로 수정하고 문서 A의 DOM을 조작할 수 있으며, 후자는 결코 문서 B와 상호작용할 의도가 없었음에도 불구하고 말입니다.

**분석:** 이제 특정 상황에서 도메인 간 통신을 허용하도록 SOP를 완화했으니, SOP의 보안 보장이 여전히 유지될까요? Alloy 분석기에 `document.domain` 속성이 공격자에 의해 사용자의 민감한 데이터에 접근하거나 변조하는 데 악용될 수 있는지 알려달라고 요청해봅시다.

실제로, 새롭고 완화된 SOP 정의가 주어지면, 분석기는 기밀성 속성에 대한 반례 시나리오를 생성합니다:

```
check Confidentiality for 5
```

이 시나리오는 다섯 단계로 구성됩니다; 처음 세 단계는 서로 다른 출처의 두 문서인 `CalendarPage`와 `InboxPage`가 도메인 속성을 공통 값(`ExampleDomain`)으로 설정하여 통신하는 `document.domain`의 전형적인 사용을 보여줍니다. 마지막 두 단계는 다른 두 문서의 콘텐츠에 접근을 시도하는 악의적인 스크립트로 손상된 또 다른 문서 `BlogPage`를 소개합니다.

시나리오의 시작에서 (\aosafigref{500l.same-origin-policy.fig-setdomain-1a}와 \aosafigref{500l.same-origin-policy.fig-setdomain-1b}), `InboxPage`와 `CalendarPage`는 두 개의 서로 다른 값(`EmailDomain`과 `ExampleDomain` 각각)을 가진 도메인 속성을 가지므로, 브라우저는 이들이 서로의 DOM에 접근하는 것을 방지할 것입니다. 문서 내부에서 실행되는 스크립트들(`InboxScript`와 `CalendarScript`)은 각각 `SetDomain` 연산을 실행하여 도메인 속성을 `ExampleDomain`으로 수정합니다 (`ExampleDomain`이 원래 도메인의 상위 도메인이므로 허용됨).

\aosafigure[180pt]{same-origin-policy-images/fig-setdomain-1a.png}{도메인 간 반례 시간 0}{500l.same-origin-policy.fig-setdomain-1a}
\aosafigure[180pt]{same-origin-policy-images/fig-setdomain-1b.png}{도메인 간 반례 시간 1}{500l.same-origin-policy.fig-setdomain-1b}

이렇게 함으로써, 이들은 이제 \aosafigref{500l.same-origin-policy.fig-setdomain-1c}에서와 같이 `ReadDom` 또는 `WriteDom` 연산을 실행하여 서로의 DOM에 접근할 수 있습니다.

\aosafigure[180pt]{same-origin-policy-images/fig-setdomain-1c.png}{도메인 간 반례 시간 2}{500l.same-origin-policy.fig-setdomain-1c}

주목할 점은 `email.example.com`과 `calendar.example.com`의 도메인을 `example.com`으로 설정할 때, 이 두 페이지가 서로 통신할 수 있을 뿐만 아니라 `example.com`을 상위 도메인으로 가진 _모든_ 다른 페이지(예: `blog.example.com`)도 통신할 수 있게 된다는 것입니다. 공격자도 이를 깨닫고, 공격자의 블로그 페이지(`BlogPage`) 내부에서 실행되는 특별한 스크립트(`EvilScript`)를 구성합니다. 다음 단계(\aosafigref{500l.same-origin-policy.fig-setdomain-2a})에서, 스크립트는 `SetDomain` 연산을 실행하여 `BlogPage`의 도메인 속성을 `ExampleDomain`으로 수정합니다.

\aosafigure[180pt]{same-origin-policy-images/fig-setdomain-2a.png}{도메인 간 반례 시간 3}{500l.same-origin-policy.fig-setdomain-2a}

이제 `BlogPage`가 다른 두 문서와 동일한 도메인 속성을 가지게 되었으므로, `ReadDOM` 연산을 성공적으로 실행하여 그들의 콘텐츠에 접근할 수 있습니다 (\aosafigref{500l.same-origin-policy.fig-setdomain-2b}).

\aosafigure[180pt]{same-origin-policy-images/fig-setdomain-2b.png}{도메인 간 반례 시간 4}{500l.same-origin-policy.fig-setdomain-2b}

이 공격은 도메인 간 통신을 위한 도메인 속성 방법의 한 가지 중요한 약점을 지적합니다: 이 방법을 사용하는 애플리케이션의 보안은 동일한 기본 도메인을 공유하는 모든 페이지 중 가장 약한 링크만큼만 강합니다. 곧 PostMessage라는 또 다른 방법에 대해 논의할 것인데, 이는 더 일반적인 클래스의 도메인 간 통신에 사용될 수 있으면서도 더 안전합니다.

### JSON with Padding (JSONP)

CORS(곧 논의할 예정)가 도입되기 전에는, JSONP가 XMLHttpRequest에 대한 SOP 제한을 우회하는 가장 인기 있는 기법이었으며, 오늘날에도 여전히 널리 사용되고 있습니다. JSONP는 HTML의 스크립트 포함 태그(즉, `<script>`)가 SOP에서 면제된다는 사실을 활용합니다*; 즉, _어떤_ URL에서든 스크립트를 포함할 수 있으며, 브라우저는 현재 문서에서 그것을 즉시 실행합니다:

(\* 이 면제가 없다면, JQuery와 같은 JavaScript 라이브러리를 다른 도메인에서 로드하는 것이 불가능할 것입니다.)

```html
<script src="http://www.example.com/myscript.js"></script>
```

스크립트 태그는 코드를 얻는 데 사용될 수 있지만, 서로 다른 도메인에서 임의의 _데이터_(예: JSON 객체)를 받기 위해 어떻게 사용할 수 있을까요? 문제는 브라우저가 `src`의 내용이 JavaScript 코드 조각이기를 기대한다는 것이며, 따라서 단순히 데이터 소스(예: JSON 또는 HTML 파일)를 가리키게 하면 구문 오류가 발생한다는 것입니다.

한 가지 해결 방법은 브라우저가 유효한 JavaScript 코드로 인식하는 문자열 내부에 원하는 데이터를 감싸는 것입니다; 이 문자열은 때때로 _패딩_(따라서 "JSON with padding"이라는 이름)이라고 불립니다. 이 패딩은 임의의 JavaScript 코드일 수 있지만, 관례적으로는 응답 데이터에 대해 실행될 (현재 문서에 이미 정의된) 콜백 함수의 이름입니다:

```html
<script src="http://www.example.com/mydata?jsonp=processData"></script>
```

`www.example.com`의 서버는 이것을 JSONP 요청으로 인식하고, 요청된 데이터를 `jsonp` 매개변수 내부에 감쌉니다:

```javascript
processData(mydata)
```

이것은 유효한 JavaScript 문(즉, 값 "mydata"에 대한 함수 "processData"의 적용)이며, 브라우저에 의해 현재 문서에서 실행됩니다.

우리 모델에서 JSONP는 `padding` 필드에 콜백 함수의 식별자를 포함하는 HTTP 요청의 한 종류로 모델링됩니다. JSONP 요청을 받은 후, 서버는 요청된 리소스(`payload`)가 콜백 함수(`cb`) 내부에 감싸진 응답을 반환합니다.

```alloy
sig CallbackID {}  // identifier of a callback function
// Request sent as a result of <script> tag
sig JsonpRequest in BrowserHttpRequest {
  padding: CallbackID
}{
  response in JsonpResponse
}
sig JsonpResponse in Resource {
  cb: CallbackID,
  payload: Resource
}
```

브라우저가 응답을 받으면, 페이로드에 대해 콜백 함수를 실행합니다:

```alloy
sig JsonpCallback extends EventHandler {
  cb: CallbackID,
  payload: Resource
}{
  causedBy in JsonpRequest
  let resp = causedBy.response |
    cb = resp.@cb and
    -- result of JSONP request is passed on as an argument to the callback
    payload = resp.@payload
}
```

(`EventHandler`는 다른 호출 이후 어느 시점에 발생해야 하는 특별한 유형의 호출로, `causedBy`로 표시됩니다; 우리는 브라우저 이벤트에 응답하여 스크립트가 수행하는 작업을 모델링하기 위해 이벤트 핸들러를 사용할 것입니다.)

실행되는 콜백 함수는 응답에 포함된 것과 동일하지만(`cb = resp.@cb`), 원래 JSONP 요청의 `padding`과 _반드시_ 동일한 것은 아닙니다. 다시 말해, JSONP 통신이 작동하려면, 서버는 원래 패딩을 콜백 함수로 포함하는 응답을 적절히 구성할 책임이 있습니다 (즉, `JsonRequest.padding = JsonpResponse.cb`를 보장). 원칙적으로, 서버는 요청의 `padding`과 아무 관련이 없는 것을 포함하여 모든 콜백 함수(또는 모든 JavaScript 조각)를 포함하도록 선택할 수 있습니다. 이는 JSONP의 잠재적 위험을 강조합니다: JSONP 요청을 받는 서버는 클라이언트 문서에서 모든 JavaScript 코드 조각을 실행할 능력을 가지고 있기 때문에 신뢰할 수 있고 안전해야 합니다.

**분석:** Alloy 분석기로 `Confidentiality` 속성을 확인하면 JSONP의 한 가지 잠재적 보안 위험을 보여주는 반례가 반환됩니다. 이 시나리오에서 캘린더 애플리케이션(`CalendarServer`)은 JSONP 엔드포인트(`GetSchedule`)를 사용하여 제3자 사이트에서 리소스를 사용할 수 있도록 합니다. 리소스에 대한 접근을 제한하기 위해, `CalendarServer`는 요청에 해당 사용자를 올바르게 식별하는 쿠키가 포함된 경우에만 사용자의 일정이 포함된 응답을 다시 보냅니다.

서버가 HTTP 엔드포인트를 JSONP 서비스로 제공하면, 악의적인 사이트를 포함하여 누구나 JSONP 요청을 만들 수 있다는 점에 주목하세요. 이 시나리오에서 `EvilServer`의 광고 배너 페이지는 `Leak`라는 콜백 함수를 `padding`으로 하는 `GetSchedule` 요청을 발생시키는 _script_ 태그를 포함합니다. 일반적으로, `AdBanner`의 개발자는 `CalendarServer`에 대한 피해자 사용자의 세션 쿠키(`MyCookie`)에 직접 접근할 수 없습니다. 하지만 JSONP 요청이 `CalendarServer`로 보내지기 때문에, 브라우저는 자동으로 `MyCookie`를 요청의 일부로 포함합니다; `MyCookie`가 포함된 JSONP 요청을 받은 `CalendarServer`는 피해자의 리소스(`MySchedule`)를 패딩 `Leak` 내부에 감싸서 반환할 것입니다 (\aosafigref{500l.same-origin-policy.fig-jsonp-1}).

\aosafigure[240pt]{same-origin-policy-images/fig-jsonp-1.png}{JSONP 반례 시간 0}{500l.same-origin-policy.fig-jsonp-1}

다음 단계에서, 브라우저는 JSONP 응답을 `Leak(MySchedule)` 호출로 해석합니다 (\aosafigref{500l.same-origin-policy.fig-jsonp-2}). 나머지 공격은 간단합니다; `Leak`는 단순히 입력 인수를 `EvilServer`로 전달하도록 프로그래밍될 수 있어, 공격자가 피해자의 민감한 정보에 접근할 수 있게 됩니다.

\aosafigure[180pt]{same-origin-policy-images/fig-jsonp-2.png}{JSONP 반례 시간 1}{500l.same-origin-policy.fig-jsonp-2}

_사이트 간 요청 위조_(CSRF)의 예인 이 공격은 JSONP의 내재적 약점을 보여줍니다; 웹상의 _모든_ 사이트가 단순히 `<script>` 태그를 포함함으로써 JSONP 요청을 만들고 패딩 내부의 페이로드에 접근할 수 있습니다. 이 위험은 두 가지 방법으로 완화될 수 있습니다: (1) JSONP 요청이 민감한 데이터를 결코 반환하지 않도록 보장하거나, (2) 요청을 승인하기 위해 쿠키 대신 다른 메커니즘(예: 비밀 토큰)을 사용하는 것입니다.

### PostMessage

PostMessage는 (가능하면 서로 다른 출처의) 두 문서의 스크립트가 서로 통신할 수 있게 해주는 HTML5의 새로운 기능입니다. 이는 `domain` 속성을 설정하는 방법에 대한 더 체계적인 대안을 제공하지만, 고유한 보안 위험을 가져옵니다.

`PostMessage`는 두 개의 인수를 취하는 브라우저 API 함수입니다: (1) 전송될 데이터(`message`), (2) 메시지를 받는 문서의 출처(`targetOrigin`):

```alloy
sig PostMessage extends BrowserOp {
  message: Resource,
  targetOrigin: Origin
}
```

다른 문서로부터 메시지를 받기 위해, 받는 문서는 `PostMessage`의 결과로 브라우저에 의해 호출되는 이벤트 핸들러를 등록합니다:

```alloy
sig ReceiveMessage extends EventHandler {
  data: Resource,
  srcOrigin: Origin
}{
  causedBy in PostMessage
  -- "ReceiveMessage" event is sent to the script with the correct context
  origin[to.context.src] = causedBy.targetOrigin
  -- messages match
  data = causedBy.@message
  -- the origin of the sender script is provided as "srcOrigin" param
  srcOrigin = origin[causedBy.@from.context.src]
}
```

브라우저는 `ReceiveMessage`에 두 개의 매개변수를 전달합니다: 전송되는 메시지에 해당하는 리소스(`data`)와 송신자 문서의 출처(`srcOrigin`)입니다. 시그니처 팩트는 각 `ReceiveMessage`가 해당하는 `PostMessage`와 관련하여 잘 구성되도록 보장하기 위한 네 가지 제약을 포함합니다.

**분석:** 다시 한번, `PostMessage`가 도메인 간 통신을 수행하는 안전한 방법인지 Alloy 분석기에 물어봅시다. 이번에는 분석기가 `Integrity` 속성에 대한 반례를 반환하는데, 이는 공격자가 `PostMessage`의 약점을 악용하여 신뢰할 수 있는 애플리케이션에 악의적인 데이터를 도입할 수 있다는 것을 의미합니다.

기본적으로 PostMessage 메커니즘은 누가 PostMessage를 보낼 수 있는지를 제한하지 않는다는 점에 주목하세요; 다시 말해, 후자가 `ReceiveMessage` 핸들러를 등록한 한 모든 문서가 다른 문서에 메시지를 보낼 수 있습니다. 예를 들어, Alloy에서 생성된 다음 인스턴스에서 `AdBanner` 내부에서 실행되는 `EvilScript`는 `EmailDomain`의 대상 출처를 가진 문서에 악의적인 `PostMessage`를 보냅니다 (\aosafigref{500l.same-origin-policy.fig-postmessage-1}).

\aosafigure[240pt]{same-origin-policy-images/fig-postmessage-1.png}{PostMessage 반례 시간 0}{500l.same-origin-policy.fig-postmessage-1}

그러면 브라우저는 이 메시지를 해당 출처의 문서(이 경우 `InboxPage`)로 전달합니다. `InboxScript`가 원하지 않는 출처의 메시지를 필터링하기 위해 `srcOrigin` 값을 구체적으로 확인하지 않는 한, `InboxPage`는 악의적인 데이터를 받아들일 것이며, 이는 추가적인 보안 공격으로 이어질 수 있습니다. (예를 들어, XSS 공격을 수행하기 위한 JavaScript 조각을 포함할 수 있습니다.) 이는 \aosafigref{500l.same-origin-policy.fig-postmessage-2}에 나와 있습니다.

\aosafigure[240pt]{same-origin-policy-images/fig-postmessage-2.png}{PostMessage 반례 시간 1}{500l.same-origin-policy.fig-postmessage-2}

이 예제가 보여주는 바와 같이, `PostMessage`는 기본적으로 안전하지 않으며, 메시지가 신뢰할 수 있는 문서에서 오는 것임을 보장하기 위해 `srcOrigin` 매개변수를 _추가적으로_ 확인하는 것은 받는 문서의 책임입니다. 불행히도, 실제로는 많은 사이트들이 이 확인을 생략하여, 악의적인 문서가 `PostMessage`의 일부로 나쁜 콘텐츠를 주입할 수 있게 합니다[^postMessageStudy].

하지만 출처 확인의 생략은 단순히 프로그래머의 무지의 결과가 아닐 수도 있습니다. 들어오는 PostMessage에 대한 적절한 확인을 구현하는 것은 까다로울 수 있습니다; 일부 애플리케이션에서는 메시지가 받아질 것으로 예상되는 신뢰할 수 있는 출처들의 목록을 미리 결정하기 어렵습니다. (일부 앱에서는 이 목록이 동적으로 변경될 수도 있습니다.) 이것은 다시 보안과 기능성 사이의 긴장을 강조합니다: PostMessage는 안전한 도메인 간 통신에 사용될 수 있지만, 신뢰할 수 있는 출처들의 화이트리스트가 알려져 있을 때만 가능합니다.

### Cross-Origin Resource Sharing (CORS)

Cross-Origin Resource Sharing (CORS)은 서버가 서로 다른 출처의 사이트들과 리소스를 공유할 수 있도록 설계된 메커니즘입니다. 특히, CORS는 한 출처의 스크립트가 서로 다른 출처의 서버에 요청을 만드는 데 사용될 수 있어, 도메인 간 Ajax 요청에 대한 SOP의 제한을 효과적으로 우회합니다.

간략하게, 전형적인 CORS 프로세스는 두 단계를 포함합니다: (1) 외부 서버의 리소스에 접근하려는 스크립트는 요청에 스크립트의 출처를 명시하는 "Origin" 헤더를 포함하고, (2) 서버는 응답의 일부로 서버의 리소스에 접근할 수 있는 출처들의 집합을 나타내는 "Access-Control-Allow-Origin" 헤더를 포함합니다. 일반적으로, CORS가 없다면 브라우저는 SOP를 준수하여 처음부터 스크립트가 도메인 간 요청을 만드는 것을 방지할 것입니다. 하지만 CORS가 활성화되면, 브라우저는 스크립트가 요청을 보내고 응답에 접근할 수 있게 하지만, _오직_ "Origin"이 "Access-Control-Allow-Origin"에서 명시된 출처 중 하나인 경우에만 가능합니다.

(CORS는 추가적으로 GET과 POST 외의 복잡한 유형의 도메인 간 요청을 지원하기 위해 여기서는 논의되지 않는 _프리플라이트_ 요청의 개념을 포함합니다.)

Alloy에서, 우리는 CORS 요청을 두 개의 추가 필드 `origin`과 `allowedOrigins`를 가진 특별한 종류의 `XmlHttpRequest`로 모델링합니다:

```alloy
sig CorsRequest in XmlHttpRequest {
  -- "origin" header in request from client
  origin: Origin,
  -- "access-control-allow-origin" header in response from server
  allowedOrigins: set Origin
}{
  from in Script
}
```

그런 다음 유효한 CORS 요청을 구성하는 것이 무엇인지 설명하기 위해 Alloy 팩트 `corsRule`을 사용합니다:

```alloy
fact corsRule {
  all r: CorsRequest |
    -- the origin header of a CORS request matches the script context
    r.origin = origin[r.from.context.src] and
    -- the specified origin is one of the allowed origins
    r.origin in r.allowedOrigins
}
```

**분석:** CORS가 공격자가 신뢰할 수 있는 사이트의 보안을 손상시킬 수 있는 방식으로 오용될 수 있을까요? 요청받으면, Alloy 분석기는 `Confidentiality` 속성에 대한 간단한 반례를 반환합니다.

여기서 캘린더 애플리케이션의 개발자는 CORS 메커니즘을 사용하여 다른 애플리케이션들과 일부 리소스를 공유하기로 결정합니다. 불행히도, `CalendarServer`는 CORS 응답에서 `access-control-allow-origin` 헤더에 대해 (모든 출처 값들의 집합을 나타내는) `Origin`을 반환하도록 구성되어 있습니다. 결과적으로, `EvilDomain`을 포함하여 모든 출처의 스크립트가 `CalendarServer`에 사이트 간 요청을 만들고 그 응답을 읽는 것이 허용됩니다 (\aosafigref{500l.same-origin-policy.fig-cors}).

\aosafigure[240pt]{same-origin-policy-images/fig-cors.png}{CORS 반례}{500l.same-origin-policy.fig-cors}

이 예제는 개발자들이 CORS로 범하는 한 가지 일반적인 실수를 강조합니다: "access-control-allow-origin" 헤더의 값으로 와일드카드 값 "\*"를 사용하여, 모든 사이트가 서버의 리소스에 접근할 수 있게 하는 것입니다. 이 접근 패턴은 리소스가 공개적이고 누구나 접근 가능한 것으로 간주되는 경우에는 적절합니다. 하지만 많은 사이트들이 개인 리소스에 대해서도 "\*"를 기본값으로 사용하여, 부주의하게 악의적인 스크립트들이 CORS 요청을 통해 그것들에 접근할 수 있게 한다는 것이 밝혀졌습니다[^corsStudy].

왜 개발자가 와일드카드를 사용하려고 할까요? 허용된 출처들을 명시하는 것이 까다로울 수 있다는 것이 밝혀졌는데, 설계 시점에서 런타임에 어떤 출처들이 접근을 허가받아야 하는지 명확하지 않을 수 있기 때문입니다 (위에서 논의한 PostMessage 문제와 유사). 예를 들어, 서비스는 제3자 애플리케이션들이 동적으로 자신의 리소스를 구독할 수 있게 허용할 수 있습니다.

## 결론

이 장에서 우리는 Alloy라는 언어로 정책의 _모델_을 구축함으로써 SOP와 관련 메커니즘에 대한 명확한 이해를 제공하는 문서를 구성하기 시작했습니다. 우리의 SOP 모델은 전통적인 의미의 구현이 아니며, 다른 장들에서 보여진 산출물들과 달리 사용을 위해 배포될 수 없습니다. 대신, 우리는 "애자일 모델링"에 대한 우리 접근법의 핵심 요소들을 보여주고자 했습니다: (1) 시스템의 작고 추상적인 모델로 시작하여 필요에 따라 _점진적으로_ 세부 사항을 추가하기, (2) 시스템이 만족할 것으로 예상되는 _속성들_을 명시하기, (3) 시스템 설계의 잠재적 결함을 탐색하기 위해 _엄밀한 분석_을 적용하기. 물론 이 장은 SOP가 처음 도입된 훨씬 뒤에 작성되었지만, 이런 유형의 모델링이 시스템 설계의 초기 단계에서 수행된다면 잠재적으로 훨씬 더 유익할 것이라고 믿습니다.

SOP 외에도, Alloy는 네트워크 프로토콜, 시맨틱 웹, 바이트코드 보안부터 전자 투표와 의료 시스템에 이르기까지 서로 다른 도메인의 다양한 시스템들을 모델링하고 추론하는 데 사용되어 왔습니다. 이러한 많은 시스템들에 대해, Alloy의 분석은 경우에 따라서는 수년간 개발자들을 피해 온 설계 결함과 버그의 발견으로 이어졌습니다. 독자들에게 [Alloy 페이지](http://alloy.mit.edu)를 방문하여 자신이 좋아하는 시스템의 모델을 구축해보기를 권합니다!

[^postMessageStudy]: Sooel Son and Vitaly Shmatikov. *The Postman Always Rings Twice: Attacking and Defending postMessage in HTML5 Websites*. Network and Distributed System Security Symposium (NDSS), 2013.

[^corsStudy]: Sebastian Lekies, Martin Johns, and Walter Tighzert. *The State of the Cross-Domain Nation*. Web 2.0 Security and Privacy (W2SP), 2011.

## 부록: Alloy에서 모듈 재사용
\label{500l.sop.appendix}
이 장의 앞부분에서 언급한 바와 같이, Alloy는 모델링되는 시스템의 동작에 대해 어떤 가정도 하지 않습니다. 내장된 패러다임의 부재는 사용자가 기본 언어 구조의 작은 핵심을 사용하여 광범위한 모델링 관용구를 인코딩할 수 있게 해줍니다. 예를 들어, 우리는 시스템을 상태 기계로, 복잡한 불변량을 가진 데이터 모델로, 전역 클록을 가진 분산 이벤트 모델로, 또는 해당 문제에 가장 적합한 어떤 관용구로든 명시할 수 있습니다. 일반적으로 사용되는 관용구들은 제네릭 모듈로 포착되어 여러 시스템에서 재사용될 수 있습니다.

우리의 SOP 모델에서, 우리는 시스템을 하나 이상의 _호출_을 만들어 서로 통신하는 엔드포인트들의 집합으로 모델링합니다. _호출_이 상당히 일반적인 개념이므로, 우리는 그것의 설명을 별도의 Alloy 모듈에 캡슐화하여, 그것에 의존하는 다른 모듈들에서 가져올 수 있게 합니다 -- 프로그래밍 언어의 표준 라이브러리와 유사합니다:

```alloy
module call[T]
```

이 모듈 선언에서 `T`는 모듈이 가져올 때 제공되는 구체적인 타입으로 인스턴스화될 수 있는 타입 매개변수를 나타냅니다. 이 타입 매개변수가 어떻게 사용되는지 곧 보게 될 것입니다.

시스템 실행을 전역 시간 프레임에서 발생하는 것으로 설명하는 것이 종종 편리한데, 이를 통해 호출들이 서로 전후로 발생하는 것(또는 동시에)으로 말할 수 있습니다. 시간의 개념을 나타내기 위해, 우리는 `Time`이라는 새로운 시그니처를 도입합니다:

```alloy
open util/ordering[Time] as ord
sig Time {}
```

Alloy에서 `util/ordering`은 타입 매개변수에 전순서를 부과하는 내장 모듈이므로, `ordering[Time]`을 가져옴으로써 다른 전순서 집합들(예: 자연수)처럼 행동하는 `Time` 객체들의 집합을 얻습니다.

`Time`에 대해 절대적으로 특별한 것은 없다는 점에 주목하세요; 우리는 그것을 다른 방식으로 명명할 수 있었고(예를 들어, `Step`이나 `State`), 그것이 모델의 동작을 전혀 변경하지 않았을 것입니다. 우리가 여기서 하고 있는 것은 시스템 실행의 서로 다른 지점에서 필드의 내용을 나타내는 방법으로 관계의 추가 열을 사용하는 것입니다; 예를 들어, `Browser` 시그니처의 `cookies`. 이런 의미에서 `Time` 객체들은 일종의 색인으로 사용되는 도우미 객체들에 불과합니다.

각 호출은 두 시점 사이에 발생합니다—그것의 `start`와 `end` 시간—그리고 송신자(`from`으로 표현)와 수신자(`to`)와 연관됩니다:

```alloy
abstract sig Call { start, end: Time, from, to: T }
```

HTTP 요청에 대한 우리의 논의에서, 우리가 `Endpoint`를 타입 매개변수로 전달하여 모듈 `call`을 가져왔다는 것을 기억하세요. 결과적으로, 매개변수 타입 `T`는 `Endpoint`로 인스턴스화되고, 우리는 한 쌍의 송신자와 수신자 엔드포인트와 연관된 `Call` 객체들의 집합을 얻습니다. 모듈은 여러 번 가져올 수 있습니다; 예를 들어, 우리는 `UnixProcess`라는 시그니처를 선언하고, 한 Unix 프로세스에서 다른 프로세스로 보내지는 `Call` 객체들의 별개 집합을 얻기 위해 모듈 `call`을 인스턴스화할 수 있습니다.

