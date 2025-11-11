title:  Dagoba: 인메모리 그래프 데이터베이스
author: Dann Toliver
<markdown>
_[Dann](https://twitter.com/dann)은 프로그래밍 언어, 데이터베이스, 분산 시스템, 똑똑하고 친근한 인간들의 커뮤니티, 그리고 두 살짜리 딸과 함께 만드는 조랑말 성 같은 것들을 만드는 것을 즐긴다._
</markdown>
## 프롤로그

> "우리가 무엇이든 홀로 떼어내려고 시도하면, 그것이 끊을 수 없는 수천 개의 보이지 않는 끈으로 우주의 모든 것과 단단히 결속되어 있음을 발견하게 된다."
> &mdash;존 뮤어(John Muir)

> "세상 끝까지 나아가 자신이 아닌 것, 신, 태양, 셰익스피어, 상업 여행자를 횡단한 것이, 실제로 자신을 횡단한 후에는 그 자신이 된다."
> &mdash;제임스 조이스(James Joyce)

\noindent 오래전, 세상이 아직 젊었을 때, 모든 데이터는 한 줄로 줄지어 행복하게 걸어다녔다. 데이터가 울타리를 넘게 하고 싶다면, 그저 그 경로에 울타리를 놓기만 하면 각 데이터가 차례대로 넘어갔다. 펀치카드가 들어가고, 펀치카드가 나왔다. 삶은 쉬웠고 프로그래밍은 미풍이었다.

그 다음 임의 접근 혁명이 왔고, 데이터는 언덕 곳곳에서 자유롭게 풀을 뜯었다. 데이터를 몰아가는 것이 심각한 관심사가 되었다: 언제든지 어떤 데이터에든 접근할 수 있다면, 다음에 어느 것을 선택해야 할지 어떻게 알 수 있을까? 항목들 사이의 링크를 형성하여 데이터를 둘러싸는 기법들이 개발되었고[^items], 연결 어셈블리를 통해 단위들의 그룹을 대형으로 정렬했다. 데이터를 질의한다는 것은 양 한 마리를 선택하고 그것과 연결된 모든 것을 함께 끌고 오는 의미였다.

나중에 프로그래머들은 이 전통에서 벗어나, 데이터가 어떻게 집계될 것인지에 대한 일련의 규칙을 강요했다[^relationaltheory]. 서로 다른 데이터를 직접적으로 묶는 대신, 내용별로 클러스터를 만들어 데이터를 한 입 크기의 조각들로 분해하고, 우리에 모아서 이름표를 달았다. 질의는 선언적으로 제기되어, 부분적으로 분해된 데이터 조각들(관계형주의자들이 "정규"라고 부르는 상태)을 축적하여 프로그래머에게 반환되는 프랑켄컬렉션을 만들었다.

기록된 역사의 대부분 동안 이 관계형 모델이 최고로 군림했다. 그 지배력은 두 차례의 주요 언어 전쟁과 무수한 소규모 전투를 거치면서도 도전받지 않았다. 비효율성, 서투름, 확장성 부족이라는 작은 대가만 치르면 모델에서 요구할 수 있는 모든 것을 제공했다. 오랜 시간 동안 그것은 프로그래머들이 기꺼이 치를 의향이 있는 대가였다. 그러다 인터넷이 등장했다.

분산 혁명이 모든 것을 다시 바꿨다. 데이터는 공간적 제약에서 벗어나 머신에서 머신으로 돌아다녔다. CAP를 휘두르는 이론가들이 관계형 독점을 깨뜨려, 새로운 목축 기법들의 문을 열었다&mdash;그 중 일부는 임의 접근 데이터를 길들이려는 초기 시도들로 거슬러 올라간다. 우리는 이 중 하나인 그래프 데이터베이스라고 알려진 스타일을 살펴볼 것이다.

[^items]: 가장 초기의 데이터베이스 설계 중 하나는 계층형 모델이었는데, 이는 항목들을 트리 형태의 계층으로 그룹화했으며 여전히 고속 트랜잭션 처리 시스템인 IBM의 IMS 제품의 기반으로 사용되고 있다. 그 영향은 XML, 파일 시스템, 지리 정보 저장에서도 볼 수 있다. 찰스 배흐만(Charles Bachmann)이 발명하고 CODASYL이 표준화한 네트워크 모델은 다중 부모를 허용하여 트리 대신 DAG를 형성함으로써 계층형 모델을 일반화했다. 이러한 항해형 데이터베이스 모델들은 1960년대에 유행했고 1980년대 성능 향상으로 관계형 데이터베이스가 사용 가능해질 때까지 지배적이었다.

[^relationaltheory]: 에드거 F. 코드(Edgar F. Codd)는 IBM에서 일하면서 관계형 데이터베이스 이론을 개발했지만, 빅 블루(IBM)는 관계형 데이터베이스가 IMS의 판매를 잠식할 것을 우려했다. IBM은 결국 System R이라는 연구 프로토타입을 구축했지만, 이는 코드의 원래 알파 언어 대신 SEQUEL이라는 새로운 비관계형 언어를 기반으로 했다. SEQUEL 언어는 래리 엘리슨(Larry Ellison)이 출시 전 컨퍼런스 논문을 기반으로 Oracle Database에서 복사했고, 상표 분쟁을 피하기 위해 이름을 SQL로 바꿨다.


## 첫 번째 시도

이 챕터에서 우리는 그래프 데이터베이스를 구축할 것이다[^dagoba]. 구축하면서 문제 공간을 탐색하고, 설계 결정에 대한 여러 솔루션을 생성하며, 그 솔루션들을 비교하여 서로 간의 트레이드오프를 이해하고, 마지막으로 우리 시스템에 적합한 솔루션을 선택할 것이다. 코드 간결성에 평소보다 높은 우선순위가 주어지지만, 그 과정은 태고로부터 소프트웨어 전문가들이 사용해 온 것을 그대로 따를 것이다. 이 챕터의 목적은 이 과정을 가르치는 것이다. 그리고 그래프 데이터베이스를 구축하는 것이다[^purpose].

[^dagoba]: 이 데이터베이스는 방향성 비순환 그래프(Directed Acyclic Graphs, DAGs)를 관리하는 라이브러리로 시작되었다. "Dagoba"라는 이름은 원래 늪지대의 가상 행성에 대한 오마주로 끝에 무음 'h'가 붙을 예정이었지만, 어느 날 초콜릿 바 뒷면을 읽다가 h가 없는 버전이 사물 간의 연결을 조용히 사색하는 장소를 의미한다는 것을 발견했는데, 이것이 더욱 적합해 보였다.

[^purpose]: 이 챕터의 두 가지 목적은 이 과정을 가르치는 것, 그래프 데이터베이스를 구축하는 것, 그리고 재미있게 하는 것이다.

그래프 데이터베이스를 사용하면 흥미로운 문제들을 우아한 방식으로 해결할 수 있다. 그래프는 사물 간의 연결을 탐색하기에 매우 자연스러운 데이터 구조다. 이런 의미에서 그래프는 정점의 집합과 간선의 집합이다; 다시 말해, 선으로 연결된 점들의 무리다. 그리고 데이터베이스는? "데이터 베이스"는 데이터를 위한 요새 같은 것이다. 데이터를 넣고 데이터를 다시 꺼낸다.

그러면 그래프 데이터베이스로 어떤 종류의 문제를 해결할 수 있을까? 조상 나무를 추적하는 것을 좋아한다고 가정해보자: 부모, 조부모, 8촌 조카, 그런 류의 것들 말이다. "토르의 재종사촌은 누구인가?" 또는 "프레이야와 발키리들의 연결고리는 무엇인가?"와 같은 자연스럽고 우아한 질의를 할 수 있는 시스템을 개발하고 싶을 것이다.

이 데이터 구조에 대한 합리적인 스키마는 개체 테이블과 관계 테이블을 갖는 것이다. 토르의 부모에 대한 질의는 다음과 같을 것이다:

```sql
SELECT e.* FROM entities as e, relationships as r
WHERE r.out = "Thor" AND r.type = "parent" AND r.in = e.id
```

하지만 조부모로 확장하려면 어떻게 해야 할까? 서브쿼리를 하거나, SQL에 대한 다른 유형의 벤더 특화 확장을 사용해야 한다. 그리고 재종사촌에 이를 때쯤이면 *엄청나게 많은* SQL이 필요할 것이다.

우리가 쓰고 싶은 것은 무엇일까? 간결하면서도 유연한 것; 우리 질의를 자연스럽게 모델링하고 그와 유사한 다른 질의들로 확장되는 것. `second_cousins('Thor')`은 간결하지만 유연성을 제공하지 않는다. 위의 SQL은 유연하지만 간결성이 부족하다.

`Thor.parents.parents.parents.children.children.children`과 같은 것은 꽤 좋은 균형을 이룬다. 기본 요소들이 많은 유사한 질문을 할 수 있는 유연성을 제공하지만, 질의는 간결하고 자연스럽다. 이 특별한 표현은 1촌사촌과 형제자매를 포함하므로 너무 많은 결과를 제공하지만, 우리는 여기서 게슈탈트를 추구하고 있다.

이런 종류의 인터페이스를 제공하는 가장 간단한 것을 무엇을 만들 수 있을까? 관계형 스키마처럼 정점 목록과 간선 목록을 만들고, 몇 가지 도우미 함수를 구축할 수 있다. 다음과 같은 모습일 것이다:

```javascript
V = [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15 ]
E = [ [1,2], [1,3],  [2,4],  [2,5],  [3,6],  [3,7],  [4,8]
    , [4,9], [5,10], [5,11], [6,12], [6,13], [7,14], [7,15] ]

parents = function(vertices) {
  var accumulator = []
  for(var i=0; i < E.length; i++) {
    var edge = E[i]
    if(vertices.indexOf(edge[1]) !== -1)
      accumulator.push(edge[0])
  }
  return accumulator
}
```

위 함수의 본질은 목록을 반복하며, 각 항목에 대해 일부 코드를 평가하고 결과의 누적기를 구축하는 것이다. 하지만 루핑 구조가 불필요한 복잡성을 도입하기 때문에 가능한 한 명확하지는 않다.

이 목적을 위해 설계된 더 구체적인 루핑 구조가 있으면 좋을 것이다. 우연히 `reduce` 함수가 정확히 그런 일을 한다: 목록과 함수가 주어지면, 목록의 각 요소에 대해 함수를 평가하면서 각 평가 단계를 통해 누적기를 연결한다.

이런 더 함수형 스타일로 작성하면 질의가 더 짧고 명확해진다:

```javascript
parents  = (vertices) => E.reduce( (acc, [parent, child])
         => vertices.includes(child)  ? acc.concat(parent) : acc , [] )
children = (vertices) => E.reduce( (acc, [parent, child])
         => vertices.includes(parent) ? acc.concat(child)  : acc , [] )
```

정점 목록이 주어지면 간선들을 reduce하여, 간선의 자식이 입력 목록에 있으면 간선의 부모를 누적기에 추가한다. `children` 함수는 동일하지만, 간선의 자식을 추가할지 여부를 결정하기 위해 간선의 부모를 검사한다.

이 함수들은 유효한 JavaScript이지만, 이 글을 쓰는 시점에서 브라우저가 아직 구현하지 않은 몇 가지 기능을 사용한다. 오늘날 작동할 번역된 버전은 다음과 같다:

```javascript
parents  = function(x) { return E.reduce(
  function(acc, e) { return ~x.indexOf(e[1]) ? acc.concat(e[0]) : acc }, [] )}
children = function(x) { return E.reduce(
  function(acc, e) { return ~x.indexOf(e[0]) ? acc.concat(e[1]) : acc }, [] )}
```

이제 다음과 같이 말할 수 있다:

```javascript
    children(children(children(parents(parents(parents([8]))))))
```

거꾸로 읽히고 바보 같은 괄호에 길을 잃게 되지만, 그 외에는 우리가 원했던 것과 꽤 비슷하다. 잠시 시간을 내어 코드를 살펴보자. 개선할 방법을 찾을 수 있는가?

우리는 간선을 전역 변수로 다루고 있는데, 이는 이러한 도우미 함수를 사용하여 한 번에 하나의 데이터베이스만 가질 수 있다는 뜻이다. 꽤 제한적이다.

또한 정점을 전혀 사용하지 않고 있다. 이것이 우리에게 무엇을 말해주는가? 필요한 모든 것이 간선 배열에 있다는 것을 의미하며, 이 경우엔 그것이 맞다: 정점 값들이 스칼라이므로, 간선 배열에서 독립적으로 존재한다. "프레이야와 발키리들의 연결고리는 무엇인가?"와 같은 질문에 답하려면 정점에 더 많은 데이터를 추가해야 하는데, 이는 정점을 복합 값으로 만드는 것을 의미하고, 간선 배열이 정점의 값을 복사하는 대신 정점을 참조해야 한다는 뜻이다.

우리 간선에 대해서도 같은 것이 적용된다: 간선은 "입력" 정점과 "출력" 정점을 포함하지만[^vertexnote], 추가 정보를 우아하게 통합할 방법이 없다. "로키의 계부모는 몇 명이었는가?" 또는 "오딘이 토르가 태어나기 전에 몇 명의 자식을 가졌는가?"와 같은 질문에 답하려면 그것이 필요할 것이다.

두 선택기의 코드가 매우 유사해 보인다는 것을 알기 위해 눈을 찡그릴 필요도 없는데, 이는 그들이 나오는 더 깊은 추상화가 있을 수 있음을 시사한다.

다른 문제점이 보이는가?

[^vertexnote]: 우리가 간선을 정점 쌍으로 모델링하고 있음을 주목하라. 또한 배열을 사용하고 있기 때문에 그 쌍들이 순서가 있다는 점도 주목하라. 즉, 모든 간선이 시작 정점과 끝 정점을 가지는 *방향성 그래프*를 모델링하고 있다는 뜻이다. 우리의 "점과 선" 시각적 모델이 "점과 화살표" 모델이 된다.
  이는 간선의 방향을 추적해야 하기 때문에 모델에 복잡성을 추가하지만, "정점 3을 가리키는 정점은 어느 것인가?" 또는 "가장 많은 나가는 간선을 가진 정점은 어느 것인가?"와 같은 더 흥미로운 질문을 할 수 있게 해준다. 무방향 그래프를 모델링해야 한다면 방향성 그래프의 각 기존 간선에 대해 역방향 간선을 추가할 수 있다. 반대 방향으로 가는 것은 번거로울 수 있다: 무방향 그래프에서 방향성 그래프를 시뮬레이트하는 것 말이다. 그렇게 할 방법을 생각해볼 수 있는가?

## 더 나은 그래프 구축

우리가 발견한 몇 가지 문제를 해결해보자. 정점과 간선이 전역 구조라는 것은 한 번에 하나의 그래프로만 제한하지만, 더 많이 갖고 싶다. 이를 해결하려면 어떤 구조가 필요하다. 네임스페이스부터 시작해보자.

```javascript
Dagoba = {}                                     // the namespace
```

객체를 네임스페이스로 사용할 것이다. JavaScript의 객체는 대부분 순서가 없는 키/값 쌍의 집합일 뿐이다. JavaScript에서는 선택할 수 있는 기본 데이터 구조가 네 개뿐이므로, 이것을 많이 사용할 것이다. (파티에서 사람들에게 물어볼 재미있는 질문은 "JavaScript의 네 가지 기본 데이터 구조는 무엇인가?"이다)

이제 그래프가 몇 개 필요하다. 클래식 OOP 패턴을 사용하여 이를 구축할 수 있지만, JavaScript는 프로토타입 상속을 제공하므로 프로토타입 객체&mdash;우리는 `Dagoba.G`라고 부를 것이다&mdash;를 구축하고 팩토리 함수를 사용하여 그것의 복사본을 인스턴스화할 수 있다. 이 접근법의 장점은 생성 과정을 단일 클래스 생성자에 바인딩하는 대신 팩토리에서 다양한 유형의 객체를 반환할 수 있다는 것이다. 따라서 무료로 약간의 추가 유연성을 얻는다.

```javascript
Dagoba.G = {}                                   // the prototype

Dagoba.graph = function(V, E) {                 // the factory
  var graph = Object.create( Dagoba.G )

  graph.edges       = []                        // fresh copies so they're not shared
  graph.vertices    = []
  graph.vertexIndex = {}                        // a lookup optimization

  graph.autoid = 1                              // an auto-incrementing ID counter

  if(Array.isArray(V)) graph.addVertices(V)     // arrays only, because you wouldn't
  if(Array.isArray(E)) graph.addEdges(E)        //   call this with singular V and E

  return graph
}
```

두 개의 선택적 인수를 받을 것이다: 정점 목록과 간선 목록. JavaScript는 매개변수에 대해 상당히 느슨하므로, 모든 명명된 매개변수는 선택사항이며 제공되지 않으면 기본적으로 `undefined`가 된다[^optionalparams]. 그래프를 구축하기 전에 정점과 간선을 가지고 있어서 V와 E 매개변수를 사용하는 경우가 많지만, 생성 시에는 그것들을 갖지 않고 프로그래밍 방식으로 그래프를 구축하는 것도 일반적이다[^graphbuilding].

[^optionalparams]: 또한 다른 방향으로도 느슨하다: 모든 함수는 가변 인자이며, 모든 인수는 배열과 거의 비슷하지만 완전히 같지는 않은 `arguments` 객체를 통해 위치로 접근할 수 있다. ("가변 인자"는 함수가 불확정한 아리티를 가진다고 말하는 멋진 방식이다. "함수가 불확정한 아리티를 가진다"는 것은 가변 개수의 변수를 받는다고 말하는 멋진 방식이다.)

[^graphbuilding]: 여기서 `Array.isArray` 검사는 우리의 두 가지 다른 사용 사례를 구별하기 위한 것이지만, 일반적으로 우리는 쓰레기통 대신 아키텍처에 집중하기 위해 프로덕션 코드에서 기대할 만한 많은 검증을 하지 않을 것이다.

그런 다음 프로토타입의 모든 장점과 단점이 없는 새 객체를 생성한다. 간선을 위한 완전히 새로운 배열(다른 기본 JS 데이터 구조 중 하나), 정점을 위한 또 다른 배열, `vertexIndex`라는 새 객체, 그리고 ID 카운터를 구축한다&mdash;후자 둘에 대해서는 나중에 더 설명한다. (생각해보자: 왜 이것들을 프로토타입에 그냥 넣을 수 없을까?)

그런 다음 팩토리 내부에서 `addVertices`와 `addEdges`를 호출하므로, 이제 그것들을 정의해보자.

```javascript
Dagoba.G.addVertices = function(vs) { vs.forEach(this.addVertex.bind(this)) }
Dagoba.G.addEdges    = function(es) { es.forEach(this.addEdge  .bind(this)) }
```

좋아, 그건 너무 쉬웠다&mdash;우리는 그냥 `addVertex`와 `addEdge`에게 일을 넘기고 있다. 이제 그것들도 정의해야 한다.

```javascript
Dagoba.G.addVertex = function(vertex) {         // accepts a vertex-like object
  if(!vertex._id)
    vertex._id = this.autoid++
  else if(this.findVertexById(vertex._id))
    return Dagoba.error('A vertex with that ID already exists')

  this.vertices.push(vertex)
  this.vertexIndex[vertex._id] = vertex         // a fancy index thing
  vertex._out = []; vertex._in = []             // placeholders for edge pointers
  return vertex._id
}
```

정점이 아직 `_id` 속성을 갖지 않으면 autoid를 사용하여 하나를 할당한다.[^autoid] `_id`가 이미 그래프의 정점에 존재한다면 새 정점을 거부한다. 잠깐, 그런 일이 언제 일어날까? 그리고 정점이 정확히 무엇인가?

[^autoid]: 왜 여기서 `this.vertices.length`를 그냥 사용할 수 없을까?

전통적인 객체지향 시스템에서는 모든 정점이 인스턴스가 될 정점 클래스를 찾을 것으로 예상한다. 우리는 다른 접근법을 취하여 `_id`, `_in`, `_out` 세 속성을 포함하는 모든 객체를 정점으로 간주할 것이다. 왜 그럴까? 궁극적으로는 Dagoba가 호스트 애플리케이션과 어떤 데이터를 공유할지 제어할 수 있게 하는 것으로 귀결된다.

`addVertex` 함수 내부에서 `Dagoba.Vertex` 인스턴스를 생성한다면, 우리의 내부 데이터는 호스트 애플리케이션과 절대 공유되지 않을 것이다. `addVertex` 함수의 인수로 `Dagoba.Vertex` 인스턴스를 받는다면, 호스트 애플리케이션이 그 정점 객체에 대한 포인터를 유지하고 런타임에 조작하여 우리의 불변 조건을 깨뜨릴 수 있다.

따라서 정점 인스턴스 객체를 생성한다면, 제공된 데이터를 항상 새 객체로 복사할 것인지&mdash;공간 사용량을 잠재적으로 두 배로 늘릴 수 있는&mdash;아니면 호스트 애플리케이션이 데이터베이스 객체에 자유롭게 접근할 수 있도록 허용할 것인지를 미리 결정해야 한다. 여기에는 성능과 보호 사이의 긴장이 있으며, 올바른 균형은 구체적인 사용 사례에 따라 달라진다.

정점 속성에 대한 덕 타이핑을 사용하면 들어오는 데이터를 깊이 복사하거나[^deepcopying] 정점으로 직접 사용하여[^vertexdecision] 런타임에 그 결정을 내릴 수 있다. 안전성과 성능의 균형을 잡는 책임을 항상 사용자의 손에 맡기고 싶지는 않지만, 이 두 가지 사용 사례가 너무 크게 갈라지기 때문에 추가적인 유연성이 중요하다.

이제 새 정점을 얻었으니 그래프의 정점 목록에 추가하고, `_id`로 효율적인 조회를 위해 `vertexIndex`에 추가하고, 두 가지 추가 속성인 `_out`과 `_in`을 추가할 것이다. 둘 다 간선 목록이 될 것이다[^edgelistadt].

[^deepcopying]: 깊은 복사로 인한 공간 누수에 직면했을 때 해결책은 종종 경로 복사 영속 데이터 구조를 사용하는 것인데, 이는 $\log{}N$ 추가 공간만으로 변경 없는 수정을 허용한다. 하지만 문제는 남아있다: 호스트 애플리케이션이 정점 데이터에 대한 포인터를 유지한다면 우리가 데이터베이스에 부과하는 제약과 관계없이 언제든지 그 데이터를 변경할 수 있다. 유일한 실용적 해결책은 정점을 깊이 복사하는 것인데, 이는 공간 사용량을 두 배로 늘린다. Dagoba의 원래 사용 사례는 호스트 애플리케이션에서 불변으로 처리되는 정점들을 포함하여 이 문제를 피할 수 있지만, 사용자 측에서 일정한 규율이 필요하다.

[^vertexdecision]: Dagoba 수준 구성 매개변수, 그래프 특정 구성, 또는 어떤 유형의 휴리스틱을 기반으로 이 결정을 내릴 수 있다.

[^edgelistadt]: 우리는 푸시 및 반복 연산이 필요한 추상 데이터 구조를 가리키기 위해 *목록*이라는 용어를 사용한다. 목록 추상화가 요구하는 API를 충족하기 위해 JavaScript의 "배열" 구체적 데이터 구조를 사용한다. 기술적으로 "간선 목록"과 "간선 배열" 모두 올바르므로, 주어진 순간에 어떤 것을 사용하는지는 맥락에 따라 달라진다: `.length` 속성과 같은 JavaScript 배열의 특정 세부사항에 의존한다면 "간선 배열"이라고 말할 것이다. 그렇지 않으면 어떤 목록 구현이든 충분하다는 표시로 "간선 목록"이라고 말한다.

```javascript
Dagoba.G.addEdge = function(edge) {             // accepts an edge-like object
  edge._in  = this.findVertexById(edge._in)
  edge._out = this.findVertexById(edge._out)

  if(!(edge._in && edge._out))
    return Dagoba.error("That edge's " + (edge._in ? 'out' : 'in')
                                       + " vertex wasn't found")

  edge._out._out.push(edge)                     // edge's out vertex's out edges
  edge._in._in.push(edge)                       // vice versa

  this.edges.push(edge)
}
```

먼저 간선이 연결하는 두 정점을 모두 찾은 다음, 어느 쪽 정점이 누락되었다면 간선을 거부한다. 거부 시 오류를 로깅하기 위해 도우미 함수를 사용할 것이다. 모든 오류가 이 도우미 함수를 통해 흐르므로 애플리케이션별로 동작을 재정의할 수 있다. 나중에 `onError` 핸들러를 등록할 수 있도록 확장하여, 호스트 애플리케이션이 도우미를 덮어쓰지 않고 자체 콜백을 연결할 수 있게 할 수 있다. 필요한 유연성 수준에 따라 그래프별, 애플리케이션별, 또는 둘 다에서 그런 핸들러를 등록할 수 있도록 허용할 수 있다.

```javascript
Dagoba.error = function(msg) {
  console.log(msg)
  return false
}
```

그런 다음 새 간선을 두 정점의 간선 목록에 추가할 것이다: 간선의 출력 정점의 출력 쪽 간선 목록과 입력 정점의 입력 쪽 간선 목록에.

그리고 그것이 지금 필요한 그래프 구조의 전부다!


## 질의의 등장

이 시스템에는 실제로 두 부분만 있다: 그래프를 보유하는 부분과 그래프에 대한 질문에 답하는 부분. 그래프를 보유하는 부분은 우리가 본 바와 같이 꽤 간단하다. 질의 부분은 조금 더 까다롭다.

이전과 같이 프로토타입과 질의 팩토리로 시작할 것이다.

```javascript
Dagoba.Q = {}

Dagoba.query = function(graph) {                // factory
  var query = Object.create( Dagoba.Q )

  query.   graph = graph                        // the graph itself
  query.   state = []                           // state for each step
  query. program = []                           // list of steps to take
  query.gremlins = []                           // gremlins for each step

  return query
}
```

이제 몇 가지 친구들을 소개할 좋은 때다.

*프로그램*은 일련의 *단계*들이다. 각 단계는 파이프라인의 파이프와 같다&mdash;데이터 조각이 한 끝으로 들어와서 어떤 방식으로 변환되고 다른 끝으로 나간다. 우리의 파이프라인은 완전히 그렇게 작동하지는 않지만, 좋은 1차 근사다.

프로그램의 각 단계는 *상태*를 가질 수 있으며, `query.state`는 `query.program`의 단계 목록과 색인이 상관관계가 있는 단계별 상태 목록이다.

*그렘린*은 우리의 명령을 수행하며 그래프를 여행하는 생물이다. 그렘린이 데이터베이스에서 발견되는 것은 놀라운 일일 수 있지만, 그들의 유산은 Tinkerpop의 [Blueprints](http://euranova.eu/upl_docs/publications/an-empirical-comparison-of-graph-databases.pdf)와 [Gremlin 및 Pacer 질의 언어](http://edbt.org/Proceedings/2013-Genova/papers/workshops/a29-holzschuher.pdf)로 거슬러 올라간다. 그들은 어디에 있었는지 기억하고 흥미로운 질문에 대한 답을 찾을 수 있게 해준다.

토르의 재종사촌에 대해 답하고 싶었던 질문을 기억하는가? 우리는 `Thor.parents.parents.parents.children.children.children`이 그것을 표현하는 꽤 좋은 방법이라고 결정했다. 각 `parents` 또는 `children` 인스턴스는 우리 프로그램의 단계다. 그 단계들 각각은 그 단계의 연산을 수행하는 함수인 *파이프타입*에 대한 참조를 포함한다.

우리의 실제 시스템에서 그 질의는 다음과 같을 것이다:

```javascript
    g.v('Thor').out().out().out().in().in().in()
```

각 단계는 함수 호출이므로 *인수*를 받을 수 있다. 인터프리터는 단계의 인수를 단계의 파이프타입 함수에 전달하므로, `g.v('Thor').out(2, 3)` 질의에서 `out` 파이프타입 함수는 첫 번째 매개변수로 `[2, 3]`을 받을 것이다.

질의에 단계를 추가하는 방법이 필요하다. 이를 위한 도우미 함수가 여기 있다:

```javascript
Dagoba.Q.add = function(pipetype, args) { // add a new step to the query
  var step = [pipetype, args]
  this.program.push(step)                 // step is a pair of pipetype and its args
  return this
}
```

각 단계는 파이프타입 함수와 그 함수에 적용할 인수를 결합한 복합 엔티티다. 튜플을 사용하는 대신 이 단계에서 둘을 부분적으로 적용된 함수로 결합할 수도 있지만[^tupleadt], 그러면 나중에 도움이 될 일부 내성적 힘을 잃을 것이다.

[^tupleadt]: 튜플은 또 다른 추상 데이터 구조로&mdash;목록보다 더 제약이 있는 것이다. 특히 튜플은 고정된 크기를 가진다: 이 경우 우리는 2-튜플(데이터 구조 연구자들의 기술 용어로는 "쌍"이라고도 알려진)을 사용하고 있다. 필요한 가장 제약적인 추상 데이터 구조에 대한 용어를 사용하는 것은 미래의 구현자들을 위한 배려다.

그래프에서 새 질의를 생성하는 작은 질의 초기화자 세트를 사용할 것이다. 여기 우리 예제 대부분을 시작하는 하나가 있다: `v` 메소드. 새 질의를 구축한 다음 우리의 `add` 도우미를 사용하여 초기 질의 프로그램을 채운다. 이것은 곧 살펴볼 `vertex` 파이프타입을 사용한다.

```javascript
Dagoba.G.v = function() {                       // query initializer: g.v() -> query
  var query = Dagoba.query(this)
  query.add('vertex', [].slice.call(arguments)) // add a step to our program
  return query
}
```

`[].slice.call(arguments)`는 "이 함수의 인수 배열을 주세요"라는 JS 용법이라는 점에 주목하라. `arguments`가 많은 상황에서 배열처럼 행동하므로 이미 배열이라고 가정하는 것은 용서받을 만하지만, 현대 JavaScript 배열에서 활용하는 기능 중 많은 부분이 부족하다.

## 성급함의 문제

파이프타입 자체를 살펴보기 전에 실행 전략의 흥미진진한 세계로 우회할 것이다. 주로 두 학파가 있다: 성급한 비버라고도 알려진 Call By Value 파벌은 함수가 적용되기 전에 모든 인수가 평가되어야 한다고 엄격하게 주장한다. 그들의 대립 파벌인 Call By Needian들은 무언가를 하기 전까지 가능한 한 마지막 순간까지 미루는 것에 만족한다&mdash;한 마디로, 그들은 게으르다.

엄격한 언어인 JavaScript는 호출되는 대로 각 단계를 처리할 것이다. 그러면 우리는 `g.v('Thor').out().in()`의 평가가 먼저 토르 정점을 찾고, 나가는 간선으로 연결된 모든 정점을 찾은 다음, 그 정점들 각각에서 마지막으로 들어오는 간선으로 연결된 모든 정점을 반환할 것이라고 예상할 것이다.

비엄격 언어에서는 같은 결과를 얻을 것이다&mdash;실행 전략이 여기서는 큰 차이를 만들지 않는다. 하지만 몇 가지 호출을 더 추가한다면 어떨까? 토르가 얼마나 잘 연결되어 있는지를 고려하면, 우리의 `g.v('Thor').out().out().out().in().in().in()` 질의는 많은 결과를 생성할 수 있다&mdash;실제로, 정점 목록을 고유한 결과로 제한하지 않기 때문에 전체 그래프에 있는 정점보다 훨씬 많은 결과를 생성할 수 있다.

아마 몇 개의 고유한 결과만 얻는데 관심이 있을 것이므로, 질의를 약간 바꿔보자: `g.v('Thor').out().out().out().in().in().in().unique().take(10)`. 이제 우리 질의는 최대 10개의 결과를 생성한다. 하지만 이것을 성급하게 평가한다면 어떻게 될까? 처음 10개만 반환하기 전에 여전히 수십억 개의 결과를 구축해야 할 것이다.

모든 그래프 데이터베이스는 가능한 한 적은 작업을 수행하는 메커니즘을 지원해야 하며, 대부분은 그렇게 하기 위해 어떤 형태의 비엄격 평가를 선택한다. 우리가 자체 인터프리터를 구축하고 있으므로 프로그램의 지연 평가가 가능하지만, 몇 가지 결과에 대처해야 할 수도 있다.


## 평가 전략이 우리의 정신적 모델에 미치는 파급효과

지금까지 평가에 대한 우리의 정신적 모델은 매우 간단했다:

- 정점 집합 요청
- 반환된 집합을 파이프에 입력으로 전달
- 필요에 따라 반복

사용자들을 위해 그 모델을 유지하고 싶다. 추론하기가 더 쉽기 때문이다. 하지만 우리가 본 바와 같이 구현에는 더 이상 그 모델을 사용할 수 없다. 사용자가 실제 구현과 다른 모델로 생각하게 하는 것은 많은 고통의 원천이다. 누수되는 추상화는 이것의 소규모 버전이다; 대규모에서는 좌절, 인지 부조화, 분노 종료로 이어질 수 있다.

하지만 우리의 경우는 이런 속임수에 대해 거의 최적이다: 질의에 대한 답은 실행 모델과 관계없이 동일할 것이다. 유일한 차이는 성능이다. 트레이드오프는 모든 사용자가 시스템을 사용하기 전에 더 복잡한 모델을 학습하게 하거나, 질의 성능을 더 잘 추론하기 위해 사용자의 일부를 간단한 모델에서 복잡한 모델로 전환하도록 강요하는 것 사이에 있다.

이 결정과 씨름할 때 고려할 몇 가지 요소는:

- 간단한 모델 대 더 복잡한 모델을 학습하는 상대적 인지적 어려움;
- 간단한 모델을 먼저 사용한 후 복잡한 모델로 발전하는 것과 간단한 모델을 건너뛰고 복잡한 모델만 학습하는 것에 의해 부과되는 추가적인 인지 부하;
- 전환이 필요한 사용자의 부분집합, 그들의 비례적 크기, 인지적 가용성, 가용 시간 등의 관점에서.

우리의 경우 이 트레이드오프는 의미가 있다. 대부분의 용도에서 질의는 사용자가 질의 구조를 최적화하거나 더 깊은 모델을 학습하는 것을 걱정할 필요가 없을 만큼 충분히 빠르게 결과를 반환할 것이다. 그렇게 할 사람들은 대용량 데이터셋에 대해 고급 질의를 작성하는 사용자들이며, 그들은 또한 새로운 모델로 전환하기에 가장 잘 갖춰진 사용자들일 가능성이 높다. 또한, 더 복잡한 모델을 학습하기 전에 간단한 모델을 사용함으로써 부과되는 어려움의 증가가 작을 것이라는 희망이 있다.

우리는 곧 이 새로운 모델에 대해 더 자세히 다룰 것이지만, 그동안 다음 섹션에서 염두에 둘 몇 가지 하이라이트가 있다:

- 각 파이프는 결과 집합이 아니라 한 번에 하나의 결과를 반환한다. 질의를 평가하는 동안 각 파이프가 여러 번 활성화될 수 있다.
- 읽기/쓰기 헤드가 다음에 활성화될 파이프를 제어한다. 헤드는 파이프라인의 끝에서 시작하고, 그 움직임은 현재 활성 파이프의 결과에 의해 지시된다.
- 그 결과는 앞서 언급한 그렘린 중 하나일 수 있다. 각 그렘린은 잠재적인 질의 결과를 나타내며, 파이프를 통해 상태를 함께 운반한다. 그렘린은 헤드가 오른쪽으로 이동하게 한다.
- 파이프는 'pull'의 결과를 반환할 수 있는데, 이는 헤드에게 입력이 필요하다는 신호를 보내고 헤드를 오른쪽으로 이동시킨다.
- 'done' 결과는 헤드에게 이전의 것들이 다시 활성화될 필요가 없다고 말하고, 헤드를 왼쪽으로 이동시킨다.


## 파이프타입

파이프타입은 우리 시스템의 핵심을 구성한다. 각각이 어떻게 작동하는지 이해하면, 인터프리터에서 어떻게 호출되고 함께 순서화되는지 이해하기 위한 더 나은 기반을 갖게 될 것이다.

파이프타입을 놓을 장소와 새로운 것을 추가하는 방법을 만드는 것부터 시작할 것이다.

```javascript
Dagoba.Pipetypes = {}

Dagoba.addPipetype = function(name, fun) {              // adds a chainable method
  Dagoba.Pipetypes[name] = fun
  Dagoba.Q[name] = function() {
    return this.add(name, [].slice.apply(arguments)) }  // capture pipetype and args
}
```

파이프타입의 함수가 파이프타입 목록에 추가되고, 그 다음 새로운 메소드가 질의 객체에 추가된다. 모든 파이프타입은 해당하는 질의 메소드를 가져야 한다. 그 메소드는 인수와 함께 질의 프로그램에 새로운 단계를 추가한다.

`g.v('Thor').out('parent').in('parent')`를 평가할 때 `v` 호출은 질의 객체를 반환하고, `out` 호출은 새로운 단계를 추가하고 질의 객체를 반환하며, `in` 호출도 같은 일을 한다. 이것이 우리의 메소드 체이닝 API를 가능하게 하는 것이다.

같은 이름으로 새로운 파이프타입을 추가하면 기존 것을 교체한다는 점에 주목하라. 이는 기존 파이프타입의 런타임 수정을 허용한다. 이 결정의 비용은 무엇인가? 대안은 무엇인가?

```javascript
Dagoba.getPipetype = function(name) {
  var pipetype = Dagoba.Pipetypes[name]                 // a pipetype is a function

  if(!pipetype)
    Dagoba.error('Unrecognized pipetype: ' + name)

  return pipetype || Dagoba.fauxPipetype
}
```

파이프타입을 찾을 수 없다면, 오류를 생성하고 기본 파이프타입을 반환하는데, 이는 빈 도관과 같이 작동한다: 한쪽에서 메시지가 들어오면 다른 쪽으로 전달된다.

```javascript
Dagoba.fauxPipetype = function(_, _, maybe_gremlin) {   // pass the result upstream
  return maybe_gremlin || 'pull'                        // or send a pull downstream
}
```

저 밑줄들이 보이는가? 우리는 함수에서 사용하지 않을 매개변수들을 표시하기 위해 그것들을 사용한다. 대부분의 다른 파이프타입들은 세 매개변수를 모두 사용하고, 세 매개변수 이름을 모두 갖는다. 이는 특정 파이프타입이 어떤 매개변수에 의존하는지 한눈에 구별할 수 있게 해준다.

이 밑줄 기법은 주석들을 보기 좋게 정렬시키기 때문에도 중요하다. 아니, 진짜로. 만약 프로그램이 ["사람이 읽기 위해 쓰여야 하고, 기계가 실행하는 것은 부수적이어야 한다"](https://mitpress.mit.edu/sicp/front/node3.html)면, 코드를 예쁘게 만드는 것이 우리의 주된 관심사여야 한다는 것이 즉시 따라온다.


#### Vertex

우리가 만나는 대부분의 파이프타입들은 그렘린을 받아서 더 많은 그렘린들을 생성하지만, 이 특별한 파이프타입은 그냥 문자열로부터 그렘린들을 생성한다. 정점 ID가 주어지면 단일 새 그렘린을 반환한다. 질의가 주어지면 일치하는 모든 정점들을 찾고, 그것들을 모두 처리할 때까지 한 번에 하나씩 새 그렘린을 내놓는다.

```javascript
Dagoba.addPipetype('vertex', function(graph, args, gremlin, state) {
  if(!state.vertices)
    state.vertices = graph.findVertices(args)       // state initialization

  if(!state.vertices.length)                        // all done
    return 'done'

  var vertex = state.vertices.pop()                 // OPT: requires vertex cloning
  return Dagoba.makeGremlin(vertex, gremlin.state)  // gremlins from as/back queries
})
```

먼저 일치하는 정점들을 이미 수집했는지 확인하고, 그렇지 않다면 일부를 찾으려고 시도한다. 정점들이 있다면, 하나를 꺼내서 그 정점에 앉은 새 그렘린을 반환한다. 각 그렘린은 자신만의 상태를 가져다닐 수 있는데, 그것이 어디에 있었고 그래프를 여행하면서 본 흥미로운 것들의 일지와 같다. 이 단계에 입력으로 그렘린을 받는다면, 나가는 그렘린을 위해 그것의 일지를 복사할 것이다.

여기서 state 인수를 직접 변경하고 있으며, 되돌려 전달하지 않는다는 점에 주목하라. 대안은 그렘린이나 신호 대신 객체를 반환하고, 그런 식으로 상태를 되돌려 전달하는 것이다. 그것은 우리의 반환값을 복잡하게 만들고, 추가적인 가비지를 생성한다[^garbage]. JS가 다중 반환값을 허용한다면 이 옵션을 더 우아하게 만들 것이다.

[^garbage]: Very short lived garbage though, which is the second best kind.

하지만 여전히 변경들을 처리하는 방법을 찾아야 하는데, 호출 지점이 원래 변수에 대한 참조를 유지하고 있기 때문이다. 만약 특정 참조가 "고유한" 것인지&mdash;즉, 그 객체에 대한 유일한 참조인지&mdash;를 결정할 수 있는 방법이 있다면 어떨까?

참조가 고유하다는 것을 알면, 비싼 copy-on-write 방식이나 복잡한 영속 데이터 구조를 피하면서 불변성의 이점을 얻을 수 있다. 참조가 하나뿐이면 객체가 변경되었는지 아니면 우리가 요청한 변경사항이 담긴 새 객체가 반환되었는지 구별할 수 없다: "관찰된 불변성"이 유지된다[^obsimmutability].

[^obsimmutability]: Two references to the same mutable data structure act like a pair of walkie-talkies, allowing whoever holds them to communicate directly. Those walkie-talkies can be passed around from function to function, and cloned to create a whole lot of walkie-talkies. This completely subverts the natural communication channels your code already possesses. In a system with no concurrency you can sometimes get away with it, but introduce multithreading or asynchronous behavior and all that walkie-talkie squawking can become a real drag.

이를 결정하는 몇 가지 일반적인 방법이 있다: 정적 타입 시스템에서는 고유성 타입[^uniquenesstypes]을 사용하여 각 객체가 컴파일 타임에 하나의 참조만 가진다는 것을 보장할 수 있다. 참조 카운터[^referencecounter]가 있다면&mdash;심지어 간단한 2비트 스티키 카운터라도&mdash;런타임에 객체가 하나의 참조만 가진다는 것을 알 수 있고 그 지식을 우리에게 유리하게 사용할 수 있다.

[^uniquenesstypes]: Uniqueness types were dusted off in the Clean language, and have a non-linear relationship with linear types, which are themselves a subtype of substructural types.

[^referencecounter]: Most modern JS runtimes employ generational garbage collectors, and the language is intentionally kept at arm's length from the engine's memory management to curtail a source of programmatic non-determinism.

JavaScript는 이러한 기능들 중 어느 것도 갖지 않지만, 우리가 정말, 정말 규율을 지킨다면 거의 같은 효과를 얻을 수 있다. 그리고 우리는 그렇게 할 것이다. 지금은. 


#### In-N-Out

그래프를 걷는 것은 버거를 주문하는 것만큼 쉽다. 이 두 줄이 우리를 위해 `in`과 `out` 파이프타입을 설정해 준다.

\newpage 

```javascript
Dagoba.addPipetype('out', Dagoba.simpleTraversal('out'))
Dagoba.addPipetype('in',  Dagoba.simpleTraversal('in'))
```

`simpleTraversal` 함수는 그렘린을 입력으로 받고, 질의될 때마다 새 그렘린을 생성하는 파이프타입 핸들러를 반환한다. 그 그렘린들이 소진되면, 이전 단계로부터 새 그렘린을 얻기 위해 'pull' 요청을 되돌려 보낸다.

```javascript
Dagoba.simpleTraversal = function(dir) {
  var find_method = dir == 'out' ? 'findOutEdges' : 'findInEdges'
  var edge_list   = dir == 'out' ? '_in' : '_out'

  return function(graph, args, gremlin, state) {
    if(!gremlin && (!state.edges || !state.edges.length))     // query initialization
      return 'pull'

    if(!state.edges || !state.edges.length) {                 // state initialization
      state.gremlin = gremlin
      state.edges = graph[find_method](gremlin.vertex)        // get matching edges
                         .filter(Dagoba.filterEdges(args[0]))
    }

    if(!state.edges.length)                                   // nothing more to do
      return 'pull'

    var vertex = state.edges.pop()[edge_list]                 // use up an edge
    return Dagoba.gotoVertex(state.gremlin, vertex)
  }
}
```

처음 몇 줄은 in 버전과 out 버전 사이의 차이점을 처리한다. 그러면 우리가 방금 본 vertex 파이프타입과 매우 비슷해 보이는 파이프타입 함수를 반환할 준비가 된다. 이것은 약간 놀라운데, 이것은 그렘린을 받아들이는 반면 vertex 파이프타입은 *무에서* 그렘린들을 생성하기 때문이다.

그럼에도 불구하고 여기서 같은 리듬이 반복되는 것을 볼 수 있는데, 질의 초기화 단계가 추가되었다. 그렘린이 없고 사용 가능한 간선이 떨어졌다면 pull한다. 그렘린이 있지만 아직 상태를 설정하지 않았다면 적절한 방향으로 가는 간선들을 찾아서 우리 상태에 추가한다. 그렘린이 있지만 현재 정점에 적절한 간선이 없다면 pull한다. 그리고 마지막으로 간선 하나를 꺼내서 그것이 가리키는 정점에서 새로 복제된 그렘린을 반환한다.

이 코드를 훑어보면 세 절 각각에서 `!state.edges.length`가 반복되는 것을 볼 수 있다. 그 조건문들의 복잡성을 줄이기 위해 이것을 리팩토링하고 싶은 유혹이 든다. 우리가 그렇게 하는 것을 막는 두 가지 문제가 있다.

하나는 비교적 사소한 것이다: 세 번째 `!state.edges.length`는 처음 두 개와 다른 의미인데, 두 번째와 세 번째 조건문 사이에서 `state.edges`가 변경되기 때문이다. 이는 실제로 우리가 리팩토링하도록 권장하는데, 단일 함수 내에서 같은 레이블이 두 가지 다른 의미를 갖는 것은 보통 이상적이지 않기 때문이다.

두 번째는 더 심각하다. 이것이 우리가 쓰고 있는 유일한 파이프타입 함수가 아니며, 질의 초기화 및/또는 상태 초기화의 이런 아이디어들이 계속해서 반복되는 것을 보게 될 것이다. 코드를 작성할 때 구조적 품질과 비구조적 품질 사이에는 항상 균형 맞추기가 있다. 너무 많은 구조는 보일러플레이트와 추상화 복잡성에서 높은 비용을 지불하게 한다. 너무 적은 구조는 모든 배관 세부사항을 머릿속에 유지해야 한다.

이 경우, 대략 12개 정도의 파이프타입으로는, 올바른 선택은 각 파이프타입 함수를 가능한 한 비슷하게 스타일링하고, 구성 요소들을 주석으로 레이블하는 것으로 보인다. 그래서 우리는 이 특별한 파이프타입을 리팩토링하려는 충동을 억제한다. 그렇게 하는 것이 균일성을 감소시킬 것이기 때문이다. 하지만 우리는 또한 질의 초기화, 상태 초기화 등을 위한 공식적인 구조적 추상화를 설계하려는 욕구도 억제한다. 수백 개의 파이프타입이 있다면 후자의 선택이 아마도 올바른 것일 것이다: 추상화의 복잡성 비용은 일정하지만, 이익은 단위 수에 따라 선형적으로 누적되기 때문이다. 그렇게 많은 움직이는 조각들을 다룰 때는, 그들 사이의 규칙성을 강제할 수 있는 모든 것이 도움이 된다.


#### Property

지금까지 본 세 가지 파이프타입을 기반으로 한 예제 질의를 생각해보기 위해 잠깐 멈춰보자. 이렇게 토르의 조부모를 요청할 수 있다[^runnote]: 

[^runnote]: The `run()` at the end of the query invokes the interpreter and returns results.

```javascript
g.v('Thor').out('parent').out('parent').run()
``` 
하지만 그들의 이름을 원한다면 어떨까? 그것의 끝에 map을 붙일 수 있다:

```javascript
g.v('Thor').out('parent').out('parent').run()
 .map(function(vertex) {return vertex.name})
```

하지만 이것은 충분히 일반적인 연산이라서 우리는 다음과 같은 것을 쓰는 것을 선호할 것이다:

```javascript
g.v('Thor').out('parent').out('parent').property('name').run()
```

게다가 이 방식에서는 property 파이프가 나중에 추가되는 것이 아니라 질의의 통합적인 부분이다. 곧 보게 되겠지만 이것은 몇 가지 흥미로운 이점이 있다.

```javascript
Dagoba.addPipetype('property', function(graph, args, gremlin, state) {
  if(!gremlin) return 'pull'                                  // query initialization
  gremlin.result = gremlin.vertex[args[0]]
  return gremlin.result == null ? false : gremlin             // false for bad props
})
```

여기서 우리의 질의 초기화는 간단하다: 그렘린이 없으면 pull한다. 그렘린이 있으면, 그것의 result를 속성의 값으로 설정한다. 그러면 그렘린은 계속 진행할 수 있다. 마지막 파이프를 통과하면 그것의 result가 수집되어 질의로부터 반환될 것이다. 모든 그렘린이 `result` 속성을 갖지는 않는다. 그렇지 않은 것들은 가장 최근에 방문한 정점을 반환한다.

속성이 존재하지 않으면 그렘린 대신 `false`를 반환한다는 점에 주목하라. 그래서 property 파이프도 필터의 한 유형으로 작동한다. 이것의 용도를 생각해볼 수 있는가? 이 설계 결정의 트레이드오프는 무엇인가?


#### Unique

토르의 조부모들의 손자들&mdash;그의 사촌들, 형제자매들, 그리고 그 자신&mdash;을 모두 수집하고 싶다면, `g.v('Thor').in().in().out().out().run()`과 같은 질의를 할 수 있다. 하지만 그것은 많은 중복을 줄 것이다. 실제로 토르 자신의 복사본이 최소 4개는 있을 것이다. (더 많을 수 있는 때를 생각해볼 수 있는가?)

이를 해결하기 위해 우리는 'unique'라는 새로운 파이프타입을 도입한다. 우리의 새로운 질의는 손자들과 일대일 대응으로 출력을 생성한다:

```javascript
    g.v('Thor').in().in().out().out().unique().run()
```

The pipetype implementation:

```javascript
Dagoba.addPipetype('unique', function(graph, args, gremlin, state) {
  if(!gremlin) return 'pull'                                  // query initialization
  if(state[gremlin.vertex._id]) return 'pull'                 // reject repeats
  state[gremlin.vertex._id] = true
  return gremlin
})
```

unique 파이프는 순전히 필터다: 그렘린을 변경 없이 통과시키거나 이전 파이프에서 새로운 그렘린을 pull하려고 시도한다.

그렘린을 수집하려고 시도함으로써 초기화한다. 그렘린의 현재 정점이 우리 캐시에 있다면, 이전에 본 것이므로 새로운 것을 수집하려고 시도한다. 그렇지 않으면, 그렘린의 현재 정점을 캐시에 추가하고 통과시킨다. 아주 쉽다.

#### Filter

우리는 필터링의 두 가지 간단한 방법을 보았지만, 때로는 더 정교한 제약이 필요하다. 체중이 키보다 큰 토르의 모든 형제자매를 찾고 싶다면 어떨까[^weight]? 이 질의가 우리에게 답을 줄 것이다:

[^weight]: With weight in skippund and height in fathoms, naturally. Depending on the density of Asgardian flesh this may return many results, or none at all. (Or just Volstagg, if we're allowing Shakespeare by way of Jack Kirby into our pantheon.)

```javascript
g.v('Thor').out().in().unique()
 .filter(function(asgardian) { return asgardian.weight > asgardian.height })
 .run()
```

토르의 형제자매 중 누가 라그나뢰크에서 살아남는지 알고 싶다면 filter에 객체를 전달할 수 있다:

```javascript
g.v('Thor').out().in().unique().filter({survives: true}).run()
```

작동 방식은 다음과 같다:

```javascript
Dagoba.addPipetype('filter', function(graph, args, gremlin, state) {
  if(!gremlin) return 'pull'                                  // query initialization

  if(typeof args[0] == 'object')                              // filter by object
    return Dagoba.objectFilter(gremlin.vertex, args[0])
         ? gremlin : 'pull'

  if(typeof args[0] != 'function') {
    Dagoba.error('Filter is not a function: ' + args[0])
    return gremlin                                            // keep things moving
  }

  if(!args[0](gremlin.vertex, gremlin)) return 'pull'         // gremlin fails filter
  return gremlin
})
```

필터의 첫 번째 인수가 객체나 함수가 아니라면 오류를 발생시키고, 그렘린을 통과시킨다. 잠깐 멈춰서 대안들을 생각해보자. 왜 오류가 발생하면 질의를 계속하기로 결정할까?

이 오류가 발생할 수 있는 두 가지 이유가 있다. 첫 번째는 프로그래머가 REPL이나 코드에서 직접 질의를 입력하는 것과 관련이 있다. 실행될 때, 그 질의는 결과를 생성하고, 프로그래머가 관찰할 수 있는 오류도 생성한다. 프로그래머는 그 다음 생성된 결과 집합을 더 필터링하기 위해 오류를 수정한다. 대안적으로, 시스템은 오류만 표시하고 결과를 생성하지 않을 수 있으며, 모든 오류를 수정하면 결과가 표시될 수 있다.

두 번째 가능성은 필터가 런타임에 동적으로 적용되는 것이다. 이것이 훨씬 더 중요한 경우인데, 질의를 호출하는 사람이 반드시 질의 코드의 작성자는 아니기 때문이다. 이것이 웹상에 있으므로, 우리의 기본 규칙은 항상 결과를 보여주고, 절대 중단시키지 않는 것이다. 보통은 상처에 굴복하여 사용자에게 끔찍한 오류 메시지를 제시하는 것보다 문제에 직면해서 굳건히 계속 진행하는 것이 좋다.

너무 많은 결과를 보여주는 것보다 너무 적은 결과를 보여주는 것이 나은 경우에는, `Dagoba.error`를 오버라이드하여 오류를 던질 수 있으며, 이로써 자연스러운 제어 흐름을 우회할 수 있다.


#### Take

우리는 항상 모든 결과를 한 번에 원하지는 않는다. 때로는 소수의 결과만 필요하다; 토르의 동시대인들을 12명 원한다고 하면, 원시 소 아우둠블라까지 완전히 거슬러 올라간다:

```javascript
g.v('Thor').out().out().out().out().in().in().in().in().unique().take(12).run()
```

`take` 파이프가 없다면 그 질의는 실행하는 데 상당한 시간이 걸릴 수 있지만, 지연 평가 전략 덕분에 `take` 파이프가 있는 질의는 매우 효율적이다.

때로는 한 번에 하나씩만 원한다: 결과를 처리하고, 작업하고, 그 다음에 다른 하나를 가져온다. 이 파이프타입은 그것도 할 수 있게 해준다.

```javascript
q = g.v('Auðumbla').in().in().in().property('name').take(1)

q.run() // ['Odin']
q.run() // ['Vili']
q.run() // ['Vé']
q.run() // []
```

우리 질의는 비동기 환경에서 작동할 수 있어서, 필요에 따라 더 많은 결과를 수집할 수 있다. 결과가 떨어지면, 빈 배열이 반환된다.


```javascript
Dagoba.addPipetype('take', function(graph, args, gremlin, state) {
  state.taken = state.taken || 0                              // state initialization

  if(state.taken == args[0]) {
    state.taken = 0
    return 'done'                                             // all done
  }

  if(!gremlin) return 'pull'                                  // query initialization
  state.taken++
  return gremlin
})
```

이미 존재하지 않으면 `state.taken`을 0으로 초기화한다. JavaScript에는 암시적 강제 변환이 있지만, `undefined`를 `NaN`으로 강제 변환하므로, 여기서는 명시적이어야 한다[^explicit].

[^explicit]: Some would argue it's best to be explicit all the time. Others would argue that a good system for implicits makes for more concise, readable code, with less boilerplate and a smaller surface area for bugs. One thing we can all agree on is that making effective use of JavaScript's implicit coercion requires memorizing a lot of non-intuitive special cases, making it a minefield for the uninitiated.

그러면 `state.taken`이 `args[0]`에 도달하면 'done'을 반환하여, 우리 이전의 파이프들을 차단한다. 또한 `state.taken` 카운터를 재설정하여, 나중에 질의를 반복할 수 있게 한다.

`take(0)`과 `take()`의 경우를 처리하기 위해 질의 초기화 전에 그 두 단계를 수행한다[^takereturn]. 그 다음 카운터를 증가시키고 그렘린을 반환한다.

[^takereturn]: What would you expect each of those to return? What do they actually return?


#### As

다음 네 개의 파이프타입은 그룹으로 작동하여 더 고급 질의를 가능하게 한다. 이것은 그냥 현재 정점에 레이블을 붙일 수 있게 해준다. 다음 두 파이프타입에서 그 레이블을 사용할 것이다.

```javascript
Dagoba.addPipetype('as', function(graph, args, gremlin, state) {
  if(!gremlin) return 'pull'                                  // query initialization
  gremlin.state.as = gremlin.state.as || {}                   // init the 'as' state
  gremlin.state.as[args[0]] = gremlin.vertex                  // set label to vertex
  return gremlin
})
```

질의를 초기화한 후, 그렘린의 로컬 상태가 `as` 매개변수를 갖도록 보장한다. 그 다음 그 매개변수의 속성을 그렘린의 현재 정점으로 설정한다.

#### Merge

정점들에 레이블을 붙이고 나면 merge를 사용하여 그것들을 추출할 수 있다. 토르의 부모, 조부모, 증조부모를 원한다면 다음과 같이 할 수 있다:

```javascript
g.v('Thor').out().as('parent').out().as('grandparent').out().as('great-grandparent')
           .merge('parent', 'grandparent', 'great-grandparent').run()
```

Here's the merge pipetype:

```javascript
Dagoba.addPipetype('merge', function(graph, args, gremlin, state) {
  if(!state.vertices && !gremlin) return 'pull'               // query initialization

  if(!state.vertices || !state.vertices.length) {             // state initialization
    var obj = (gremlin.state||{}).as || {}
    state.vertices = args.map(function(id) {return obj[id]}).filter(Boolean)
  }

  if(!state.vertices.length) return 'pull'                    // done with this batch

  var vertex = state.vertices.pop()
  return Dagoba.makeGremlin(vertex, gremlin.state)
})
```

각 인수를 매핑하여, 그렘린의 레이블이 붙은 정점 목록에서 그것을 찾는다. 찾으면, 그 정점으로 그렘린을 복제한다. 이 파이프에 도달한 그렘린들만 merge에 포함된다는 점에 주목하라&mdash;토르의 어머니의 부모들이 그래프에 없다면, 그녀는 결과 집합에 있지 않을 것이다.


#### Except

"토르가 아닌 토르의 모든 형제자매를 주세요"라고 말하고 싶은 경우들을 이미 보았다. 필터로 그것을 할 수 있다:

```javascript
g.v('Thor').out().in().unique()
           .filter(function(asgardian) {return asgardian._id != 'Thor'}).run()
```

`as`와 `except`로는 더 직관적이다:

```javascript
g.v('Thor').as('me').out().in().except('me').unique().run()
```

하지만 필터링하기 어려운 질의들도 있다. 토르의 삼촌과 이모를 원한다면 어떨까? 그의 부모들을 어떻게 필터링해서 제외할까? `as`와 `except`로는 쉽다[^unexpectedresults]:

```javascript
g.v('Thor').out().as('parent').out().in().except('parent').unique().run()
```

[^unexpectedresults]: There are certain conditions under which this particular query might yield unexpected results. Can you think of any? How could you modify it to handle those cases?

```javascript
Dagoba.addPipetype('except', function(graph, args, gremlin, state) {
  if(!gremlin) return 'pull'                                  // query initialization
  if(gremlin.vertex == gremlin.state.as[args[0]]) return 'pull'
  return gremlin
})
```

여기서 현재 정점이 이전에 저장한 정점과 같은지 확인한다. 같다면, 그것을 건너뛴다.


#### Back

우리가 물을 수 있는 질문들 중 일부는 그래프를 더 깊이 확인하는 것을 포함하는데, 답이 긍정적이면 나중에 시작점으로 돌아온다. Fjörgynn의 딸들 중 누가 Bestla의 아들들 중 하나와 자녀를 낳았는지 알고 싶다고 가정해보자.

```javascript
g.v('Fjörgynn').in().as('me')       // first gremlin's state.as is Frigg
 .in()                              // first gremlin's vertex is now Baldr
 .out().out()                       // clone that gremlin for each grandparent
 .filter({_id: 'Bestla'})           // keep only the gremlin on grandparent Bestla
 .back('me').unique().run()         // jump gremlin's vertex back to Frigg and exit
```

`back`의 정의는 다음과 같다:

```javascript
Dagoba.addPipetype('back', function(graph, args, gremlin, state) {
  if(!gremlin) return 'pull'                                  // query initialization
  return Dagoba.gotoVertex(gremlin, gremlin.state.as[args[0]])
})
```

여기서 모든 실제 작업을 수행하기 위해 `Dagoba.gotoVertex` 도우미 함수를 사용한다. 이제 그것과 다른 몇 가지 도우미들을 살펴보자.


## Helpers

위의 파이프타입들은 작업을 수행하기 위해 몇 가지 도우미들에 의존한다. 인터프리터로 들어가기 전에 그것들을 빠르게 살펴보자.

#### Gremlins

그렘린들은 단순한 생물이다: 현재 정점과 일부 로컬 상태를 가진다. 그래서 새로운 것을 만들려면 그 두 가지를 가진 객체를 만들기만 하면 된다.

```javascript
Dagoba.makeGremlin = function(vertex, state) {
  return {vertex: vertex, state: state || {} }
}
```

이 정의에 따르면 vertex 속성과 state 속성을 가진 모든 객체가 그렘린이므로, 그냥 생성자를 인라인할 수도 있지만, 함수로 감싸는 것은 모든 그렘린에 새로운 속성을 한 곳에서 추가할 수 있게 해준다.

`back` 파이프타입과 `simpleTraversal` 함수에서 본 것처럼, 기존 그렘린을 가져와서 새로운 정점으로 보낼 수도 있다.

```javascript
Dagoba.gotoVertex = function(gremlin, vertex) {               // clone the gremlin
  return Dagoba.makeGremlin(vertex, gremlin.state)
}
```

이 함수가 실제로는 완전히 새로운 그렘린을 반환한다는 점에 주목하라: 우리가 원하는 목적지로 보내진, 이전 것의 복제본이다. 이는 그렘린이 한 정점에 앉아 있으면서 그것의 복제본들이 많은 다른 정점들을 탐색하기 위해 보내진다는 뜻이다. 이것이 정확히 `simpleTraversal`에서 일어나는 일이다.

가능한 향상의 예로, 그렘린이 방문하는 모든 정점을 추적하기 위한 약간의 상태를 추가하고, 그런 경로들을 활용하기 위한 새로운 파이프타입들을 추가할 수 있다.


#### Finding

`vertex` 파이프타입은 질의를 시작할 초기 정점들의 집합을 수집하기 위해 `findVertices` 함수를 사용한다.

```javascript
Dagoba.G.findVertices = function(args) {                      // vertex finder helper
  if(typeof args[0] == 'object')
    return this.searchVertices(args[0])
  else if(args.length == 0)
    return this.vertices.slice()                              // OPT: slice is costly
  else
    return this.findVerticesByIds(args)
}
```

이 함수는 인수들을 목록으로 받는다. 첫 번째가 객체라면 그것을 `searchVertices`에 전달하여, 다음과 같은 질의를 허용한다:

```javascript
  g.v({_id:'Thor'}).run()
  g.v({species: 'Aesir'}).run()
```

그렇지 않고, 인수들이 있다면 `g.v('Thor', 'Odin').run()`과 같은 질의를 처리하는 `findVerticesByIds`에 전달된다.

인수가 전혀 없다면, 질의는 `g.v().run()`처럼 보인다. 이는 대형 그래프에서 자주 하고 싶은 일이 아니며, 특히 반환하기 전에 정점 목록을 슬라이싱하고 있기 때문이다. 일부 호출 지점에서 작업하면서 항목들을 꺼내어 반환된 목록을 직접 조작하기 때문에 슬라이싱한다. 호출 지점에서 복제하거나 그런 조작을 피함으로써 이 사용 사례를 최적화할 수 있다. (꺼내는 대신 상태에 카운터를 유지할 수도 있다.)

```javascript
Dagoba.G.findVerticesByIds = function(ids) {
  if(ids.length == 1) {
    var maybe_vertex = this.findVertexById(ids[0])            // maybe it's a vertex
    return maybe_vertex ? [maybe_vertex] : []                 // or maybe it isn't
  }

  return ids.map( this.findVertexById.bind(this) ).filter(Boolean)
}

Dagoba.G.findVertexById = function(vertex_id) {
  return this.vertexIndex[vertex_id]
}
```

여기서 `vertexIndex`의 사용에 주목하라. 그 인덱스 없이는 ID가 일치하는지 결정하기 위해 목록의 각 정점을 한 번에 하나씩 살펴봐야 할 것이다&mdash;상수 시간 연산을 선형 시간 연산으로 바꾸고, 그것에 직접 의존하는 모든 $O(n)$ 연산을 $O(n^2)$ 연산으로 만든다.

```javascript
Dagoba.G.searchVertices = function(filter) {        // match on filter's properties
  return this.vertices.filter(function(vertex) {
    return Dagoba.objectFilter(vertex, filter)
  })
}
```

`searchVertices` 함수는 그래프의 모든 정점에 대해 `objectFilter` 도우미를 사용한다. 다음 섹션에서 `objectFilter`를 살펴보겠지만, 그 동안 정점들을 지연적으로 검색하는 방법을 생각해볼 수 있는가?


#### Filtering

`simpleTraversal`이 만나는 간선들에 대해 필터링 함수를 사용하는 것을 보았다. 간단한 함수이지만, 우리 목적에는 충분히 강력하다.

```javascript
Dagoba.filterEdges = function(filter) {
  return function(edge) {
    if(!filter)                                 // no filter: everything is valid
      return true

    if(typeof filter == 'string')               // string filter: label must match
      return edge._label == filter

    if(Array.isArray(filter))                   // array filter: must contain label
      return !!~filter.indexOf(edge._label)

    return Dagoba.objectFilter(edge, filter)    // object filter: check edge keys
  }
}
```

첫 번째 경우는 필터가 전혀 없는 것이다: `g.v('Odin').in().run()`은 오딘으로의 모든 간선을 순회한다.

두 번째 경우는 간선의 레이블로 필터링한다: `g.v('Odin').in('parent').run()`은 'parent' 레이블을 가진 간선들을 순회한다.

세 번째 경우는 레이블의 배열을 받는다: `g.v('Odin').in(['parent', 'spouse']).run()`은 parent와 spouse 간선을 모두 순회한다.

그리고 네 번째 경우는 이전에 본 objectFilter 함수를 사용한다:

```javascript
Dagoba.objectFilter = function(thing, filter) {
  for(var key in filter)
    if(thing[key] !== filter[key])
      return false

  return true
}
```

이를 통해 필터 객체를 사용하여 간선을 질의할 수 있다:

```javascript
`g.v('Odin').in({_label: 'spouse', order: 2}).run()`    // finds Odin's second wife
```


## 인터프리터의 본질

우리는 서사의 산 정상에 도달했고, 우리의 상을 받을 준비가 되었다: 인터프리터. 코드는 실제로 꽤 간결하지만, 모델에는 약간의 미묘함이 있다.

앞서 프로그램들을 파이프라인과 비교했고, 그것은 질의 작성을 위한 좋은 정신적 모델이다. 하지만 보았듯이, 실제 구현을 위해서는 다른 모델이 필요하다. 그 모델은 파이프라인보다는 튜링 머신에 더 가깝다: 특정 단계 위에 앉는 읽기/쓰기 헤드가 있다. 그것은 단계를 "읽고", "상태"를 바꾸고, 왼쪽이나 오른쪽으로 이동한다.

단계를 읽는다는 것은 파이프타입 함수를 평가한다는 뜻이다. 위에서 본 바와 같이, 그 함수들 각각은 입력으로 전체 그래프, 자신의 인수들, 어쩌면 그렘린, 그리고 자신의 로컬 상태를 받는다. 출력으로는 그렘린, false, 또는 'pull'이나 'done'의 신호를 제공한다. 이 출력이 우리의 준-튜링 머신이 머신의 상태를 변경하기 위해 읽는 것이다.

그 상태는 단지 두 개의 변수로 구성된다: 'done'인 단계들을 기록하는 것 하나와 질의의 `results`를 기록하는 또 다른 하나. 그것들이 업데이트되고, 그 다음 머신 헤드가 이동하거나 질의가 끝나고 결과가 반환된다.

이제 우리 머신의 모든 상태를 설명했다. 빈 상태로 시작하는 결과 목록을 갖게 될 것이다:

```javascript
  var results = []
```

첫 번째 단계 뒤에서 시작하는 마지막 'done' 단계의 인덱스:

```javascript
  var done = -1
```

가장 최근 단계의 출력을 저장할 장소가 필요한데, 그것은 그렘린일 수도 있고&mdash;아무것도 아닐 수도 있으므로&mdash;`maybe_gremlin`이라고 부를 것이다:

```javascript
  var maybe_gremlin = false
```

그리고 마지막으로 읽기/쓰기 헤드의 위치를 나타내는 프로그램 카운터가 필요하다.

```javascript
  var pc = this.program.length - 1
```

그런데... 잠깐. 어떻게 지연적으로 만들 것인가[^getlazy]? 성급한 것에서 지연 시스템을 구축하는 전통적인 방법은 함수 호출의 매개변수들을 평가하는 대신 "썽크"로 저장하는 것이다. 썽크를 평가되지 않은 표현식이라고 생각할 수 있다. 일급 함수와 클로저를 가진 JS에서는, 함수와 그 인수들을 인수를 받지 않는 새로운 익명 함수로 감싸서 썽크를 생성할 수 있다:

[^getlazy]: Technically we need to implement an interpreter with non-strict semantics, which means it will only evaluate when forced to do so. Lazy evaluation is a technique used for implementing non-strictness. It's a bit lazy of us to conflate the two, so we will only disambiguate when forced to do so.

```javascript
function sum() {
  return [].slice.call(arguments).reduce(function(acc, n) { return acc + (n|0) }, 0)
}

function thunk_of_sum_1_2_3() { return sum(1, 2, 3) }

function thunker(fun, args) {
  return function() {return fun.apply(fun, args)}
}

function thunk_wrapper(fun) {
  return function() {
    return thunker.apply(null, [fun].concat([[].slice.call(arguments)]))
  }
}

sum(1, 2, 3)              // -> 6
thunk_of_sum_1_2_3()      // -> 6
thunker(sum, [1, 2, 3])() // -> 6

var sum2 = thunk_wrapper(sum)
var thunk = sum2(1, 2, 3)
thunk()                   // -> 6
```

None of the thunks are invoked until one is actually needed, which usually implies some type of output is required: in our case the result of a query. Each time the interpreter encounters a new function call, we wrap it in a thunk. Recall our original formulation of a query: `children(children(children(parents(parents(parents([8]))))))`. Each of those layers would be a thunk, wrapped up like an onion.

There are a couple of tradeoffs with this approach: one is that spatial performance becomes more difficult to reason about, because of the potentially vast thunk graphs that can be created. Another is that our program is now expressed as a single thunk, and we can't do much with it at that point.

This second point isn't usually an issue, because of the phase separation between when our compiler runs its optimizations and when all the thunking occurs at runtime. In our case we don't have that advantage: because we're using method chaining to implement a fluent interface [^fluentinterface] if we also use thunks to achieve laziness we would thunk each new method as it is called, which means by the time we get to `run()` we have only a thunk as our input, and no way to optimize our query.

[^fluentinterface]: Method chaining lets us write `g.v('Thor').in().out().run()` instead of the six lines of non-fluent JS required to accomplish the same thing.

Interestingly, our fluent interface hides another difference between our query language and regular programming languages. The query `g.v('Thor').in().out().run()` could be rewritten as `run(out(in(v(g, 'Thor'))))` if we weren't using method chaining. In JS we would first process `g` and `'Thor'`, then `v`, then `in`, `out` and `run`, working from the inside out. In a language with non-strict semantics we would work from the outside in, processing each consecutive nested layer of arguments only as needed.

So if we start evaluating our query at the end of the statement, with `run`, and work our way back to `v('Thor')`, calculating results only as needed, then we've effectively achieved non-strictness. The secret is in the linearity of our queries. Branches complicate the process graph and also introduce opportunities for duplicate calls, which require memoization to avoid wasted work. The simplicity of our query language means we can implement an equally simple interpreter based on our linear read/write head model.

In addition to allowing runtime optimizations, this style has many other benefits related to the ease of instrumentation: history, reversibility, stepwise debugging, query statistics. All these are easy to add dynamically because we control the interpreter and have left it as a virtual machine evaluator instead of reducing the program to a single thunk.


## Interpreter, Unveiled

```javascript
Dagoba.Q.run = function() {                 // a machine for query processing

  var max = this.program.length - 1         // index of the last step in the program
  var maybe_gremlin = false                 // a gremlin, a signal string, or false
  var results = []                          // results for this particular run
  var done = -1                             // behindwhich things have finished
  var pc = max                              // our program counter

  var step, state, pipetype

  while(done < max) {
    var ts = this.state
    step = this.program[pc]                 // step is a pair of pipetype and args
    state = (ts[pc] = ts[pc] || {})         // this step's state must be an object
    pipetype = Dagoba.getPipetype(step[0])  // a pipetype is just a function
```

Here `max` is just a constant, and `step`, `state`, and `pipetype` cache information about the current step. We've entered the driver loop, and we won't stop until the last step is done.

```javascript
    maybe_gremlin = pipetype(this.graph, step[1], maybe_gremlin, state)
```

Calling the step's pipetype function with its arguments.

```javascript
    if(maybe_gremlin == 'pull') {           // 'pull' means the pipe wants more input
      maybe_gremlin = false
      if(pc-1 > done) {
        pc--                                // try the previous pipe
        continue
      } else {
        done = pc                           // previous pipe is done, so we are too
      }
    }
```

To handle the 'pull' case we first set `maybe_gremlin` [^maybegremlin] to false. We're overloading our 'maybe' here by using it as a channel to pass the 'pull' and 'done' signals, but once one of those signals is sucked out we go back to thinking of this as a proper 'maybe'.

[^maybegremlin]: We call it `maybe_gremlin` to remind ourselves that it could be a gremlin, or it could be something else. Also because originally it was either a gremlin or Nothing.

If the step before us isn't 'done' [^stepnotdone] we'll move the head backward and try again. Otherwise, we mark ourselves as 'done' and let the head naturally fall forward.

[^stepnotdone]: Recall that done starts at -1, so the first step's predecessor is always done.

```javascript
    if(maybe_gremlin == 'done') {           // 'done' tells us the pipe is finished
      maybe_gremlin = false
      done = pc
    }
```

Handling the 'done' case is even easier: set `maybe_gremlin` to false and mark this step as 'done'.

```javascript
    pc++                                    // move on to the next pipe

    if(pc > max) {
      if(maybe_gremlin)
        results.push(maybe_gremlin)         // a gremlin popped out of the pipeline
      maybe_gremlin = false
      pc--                                  // take a step back
    }
  }
```

We're done with the current step, and we've moved the head to the next one. If we're at the end of the program and `maybe_gremlin` contains a gremlin, we'll add it to the results, set `maybe_gremlin` to false and move the head back to the last step in the program.

This is also the initialization state, since `pc` starts as `max`. So we start here and work our way back, and end up here again at least once for each final result the query returns.

```javascript
  results = results.map(function(gremlin) { // return projected results, or vertices
    return gremlin.result != null
         ? gremlin.result : gremlin.vertex } )

  return results
}
```

We're out of the driver loop now: the query has ended, the results are in, and we just need to process and return them. If any gremlin has its result set we'll return that, otherwise we'll return the gremlin's final vertex. Are there other things we might want to return? What are the tradeoffs here?


## Query Transformers

Now we have a nice compact interpreter for our query programs, but we're still missing something. Every modern DBMS comes with a query optimizer as an essential part of the system. For non-relational databases, optimizing our query plan rarely yields the exponential speedups seen in their relational cousins [^dboptimize], but it's still an important aspect of database design.

[^dboptimize]: Or, more pointedly, a poorly phrased query is less likely to yield exponential slowdowns. As an end-user of an RDBMS the aesthetics of query quality can often be quite opaque.

What's the simplest thing we could do that could reasonably be called a query optimizer? Well, we could write little functions for transforming our query programs before we run them. We'll pass a program in as input and get a different program back out as output.

```javascript
Dagoba.T = []                               // transformers (more than meets the eye)

Dagoba.addTransformer = function(fun, priority) {
  if(typeof fun != 'function')
    return Dagoba.error('Invalid transformer function')

  for(var i = 0; i < Dagoba.T.length; i++)  // OPT: binary search
    if(priority > Dagoba.T[i].priority) break

  Dagoba.T.splice(i, 0, {priority: priority, fun: fun})
}
```

Now we can add query transformers to our system. A query transformer is a function that accepts a program and returns a program, plus a priority level. Higher priority transformers are placed closer to the front of the list. We're ensuring `fun` is a function, because we're going to evaluate it later [^paramdomain].

[^paramdomain]: Note that we're keeping the domain of the priority parameter open, so it can be an integer, a rational, a negative number, or even things like Infinity or NaN.

We'll assume there won't be an enormous number of transformer additions, and walk the list linearly to add a new one. We'll leave a note in case this assumption turns out to be false&mdash;a binary search is much more time-optimal for long lists, but adds a little complexity and doesn't really speed up short lists.

To run these transformers we're going to inject a single line of code in to the top of our interpreter:

```javascript
Dagoba.Q.run = function() {                     // our virtual machine for querying
  this.program = Dagoba.transform(this.program) // activate the transformers
```

We'll use that to call this function, which just passes our program through each transformer in turn:

```javascript
Dagoba.transform = function(program) {
  return Dagoba.T.reduce(function(acc, transformer) {
    return transformer.fun(acc)
  }, program)
}
```

Up until this point, our engine has traded simplicity for performance, but one of the nice things about this strategy is that it leaves doors open for global optimizations that may have been unavailable if we had opted to optimize locally as we designed the system.

Optimizing a program can often increase complexity and reduce the elegance of the system, making it harder to reason about and maintain. Breaking abstraction barriers for performance gains is one of the more egregious forms of optimization, but even something seemingly innocuous like embedding performance-oriented code into business logic makes maintenance more difficult.

In light of that, this type of "orthogonal optimization" is particularly appealing. We can add optimizers in modules or even user code, instead of having them tightly coupled to the engine. We can test them in isolation, or in groups, and with the addition of generative testing we could even automate that process, ensuring that our available optimizers play nicely together.

We can also use this transformer system to add new functionality unrelated to optimization. Let's look at a case of that now.


## Aliases

Making a query like `g.v('Thor').out().in()` is quite compact, but is this Thor's siblings or his mates? Neither interpretation is fully satisfying. It'd be nicer to say what mean: either `g.v('Thor').parents().children()` or `g.v('Thor').children().parents()`.

We can use query transformers to make aliases with just a couple of extra helper functions:

```javascript
Dagoba.addAlias = function(newname, oldname, defaults) {
  defaults = defaults || []                     // default arguments for the alias
  Dagoba.addTransformer(function(program) {
    return program.map(function(step) {
      if(step[0] != newname) return step
      return [oldname, Dagoba.extend(step[1], defaults)]
    })
    }, 100)                                     // 100 because aliases run early

  Dagoba.addPipetype(newname, function() {})
}
```

We're adding a new name for an existing step, so we'll need to create a query transformer that converts the new name to the old name whenever it's encountered. We'll also need to add the new name as a method on the main query object, so it can be pulled into the query program.

If we could capture missing method calls and route them to a handler function then we might be able to run this transformer with a lower priority, but there's currently no way to do that. Instead we will run it with a high priority of 100 so the aliased methods are added before they are invoked.

We call another helper to merge the incoming step's arguments with the alias's default arguments. If the incoming step is missing an argument then we'll use the alias's argument for that slot.

```javascript
Dagoba.extend = function(list, defaults) {
  return Object.keys(defaults).reduce(function(acc, key) {
    if(typeof list[key] != 'undefined') return acc
    acc[key] = defaults[key]
    return acc
  }, list)
}
```

Now we can make those aliases we wanted:

```javascript
Dagoba.addAlias('parents', 'out')
Dagoba.addAlias('children', 'in')
```

We can also start to specialize our data model a little more, by labeling each edge between a parent and child as a 'parent' edge. Then our aliases would look like this:

```javascript
Dagoba.addAlias('parents', 'out', ['parent'])
Dagoba.addAlias('children', 'in', ['parent'])
```

Now we can add edges for spouses, step-parents, or even jilted ex-lovers. If we enhance our `addAlias` function we can introduce new aliases for grandparents, siblings, or even cousins:

```javascript
Dagoba.addAlias('grandparents', [ ['out', 'parent'], ['out', 'parent']])
Dagoba.addAlias('siblings',     [ ['as', 'me'], ['out', 'parent']
                                , ['in', 'parent'], ['except', 'me']])
Dagoba.addAlias('cousins',      [ ['out', 'parent'], ['as', 'folks']
                                , ['out', 'parent'], ['in', 'parent']
                                , ['except', 'folks'], ['in', 'parent']
                                , ['unique']])
```

That `cousins` alias is kind of cumbersome. Maybe we could expand our `addAlias` function to allow ourselves to use other aliases in our aliases, and call it like this:

```javascript
Dagoba.addAlias('cousins',      [ 'parents', ['as', 'folks']
                                , 'parents', 'children'
                                , ['except', 'folks'], 'children', 'unique'])
```

Now instead of

```javascript
g.v('Forseti').parents().as('parents').parents().children()
                        .except('parents').children().unique()
```

we can just say `g.v('Forseti').cousins()`.

We've introduced a bit of a pickle, though: while our `addAlias` function is resolving an alias it also has to resolve other aliases. What if `parents` called some other alias, and while we were resolving `cousins` we had to stop to resolve `parents` and then resolve its aliases and so on? What if one of `parents` aliases ultimately called `cousins`?

This brings us in to the realm of dependency resolution[^dependencyresolution], a core component of modern package managers. There are a lot of fancy tricks for choosing ideal versions, tree shaking, general optimizations and the like, but the basic idea is fairly simple. We're going to make a graph of all the dependencies and their relationships, and then try to find a way to line up the vertices while making all the arrows go from left to right. If we can, then this particular sorting of the vertices is called a 'topological ordering', and we've proven that our dependency graph has no cycles: it is a Directed Acyclic Graph (DAG). If we fail to do so then our graph has at least one cycle.

[^dependencyresolution]: You can learn more about dependency resolution in the Contingent chapter of this book.

On the other hand, we expect that our queries will generally be rather short (100 steps would be a very long query) and that we'll have a reasonably low number of transformers. Instead of fiddling around with DAGs and dependency management we could return 'true' from the transform function if anything changed, and then run it until it stops being productive. This requires each transformer to be idempotent, but that's a useful property for transformers to have. What are the pros and cons of these two pathways?


## Performance

All production graph databases share a particular performance characteristic: graph traversal queries are constant time with respect to total graph size [^ifadjacency]. In a non-graph database, asking for the list of someone's friends can require time proportional to the number of entries, because in the naive worst-case you have to look at every entry. This means if a query over ten entries takes a millisecond, then a query over ten million entries will take almost two weeks. Your friend list would arrive faster if sent by Pony Express [^ponyexpress]!

[^ifadjacency]: The fancy term for this is "index-free adjacency".

[^ponyexpress]: Though only in operation for 18 months due to the arrival of the transcontinental telegraph and the outbreak of the American Civil War, the Pony Express is still remembered today for delivering mail coast to coast in just ten days.

To alleviate this dismal performance most databases index over oft-queried fields, which turns an $O(n)$ search into an $O(log n)$ search. This gives considerably better search performance, but at the cost of some write performance and a lot of space&mdash;indices can easily double the size of a database. Careful balancing of the space/time tradeoffs of indices is part of the perpetual tuning process for most databases.

Graph databases sidestep this issue by making direct connections between vertices and edges, so graph traversals are just pointer jumps; no need to scan through every item, no need for indices, no extra work at all. Now finding your friends has the same price regardless of the total number of people in the graph, with no additional space cost or write time cost. One downside to this approach is that the pointers work best when the whole graph is in memory on the same machine. Effectively sharding a graph database across multiple machines is still an active area of research [^graphdbsharding].

[^graphdbsharding]: Sharding a graph database requires partitioning the graph. [Optimal graph partitioning is NP-hard](http://dl.acm.org/citation.cfm?doid=1007912.1007931), even for simple graphs like trees and grids, and good approximations also have exponential [asymptotic complexity](http://arxiv.org/pdf/1311.3144v2.pdf).

We can see this at work in the microcosm of Dagoba if we replace the functions for finding edges. Here's a naive version that searches through all the edges in linear time. It's similar to our very first implementation, but uses all the structures we've since built.

```javascript
Dagoba.G.findInEdges  = function(vertex) {
  return this.edges.filter(function(edge) {return edge._in._id  == vertex._id} )
}
Dagoba.G.findOutEdges = function(vertex) {
  return this.edges.filter(function(edge) {return edge._out._id == vertex._id} )
}
```

We can add an index for edges, which gets us most of the way there with small graphs but has all the classic indexing issues for large ones.

```javascript
Dagoba.G.findInEdges  = function(vertex) { return this.inEdgeIndex [vertex._id] }
Dagoba.G.findOutEdges = function(vertex) { return this.outEdgeIndex[vertex._id] }
```

And here we have our old friends back again: pure, sweet index-free adjacency.

```javascript
Dagoba.G.findInEdges  = function(vertex) { return vertex._in  }
Dagoba.G.findOutEdges = function(vertex) { return vertex._out }
```

Run these yourself to experience the graph database difference [^jslistfilter].

[^jslistfilter]: In modern JavaScript engines filtering a list is quite fast&mdash;for small graphs the naive version can actually be faster than the index-free version due to the underlying data structures and the way the code is JIT compiled. Try it with different sizes of graphs to see how the two approaches scale.


## Serialization

Having a graph in memory is great, but how do we get it there in the first place? We saw that our graph constructor can take a list of vertices and edges and create a graph for us, but once the graph has been built how do we get the vertices and edges back out?

Our natural inclination is to do something like `JSON.stringify(graph)`, which produces the terribly helpful error "TypeError: Converting circular structure to JSON". During the graph construction process the vertices were linked to their edges, and the edges are all linked to their vertices, so now everything refers to everything else. So how can we extract our nice neat lists again? JSON replacer functions to the rescue.

The `JSON.stringify` function takes a value to stringify, but it also takes two additional parameters: a replacer function and a whitespace number [^protip]. The replacer allows you to customize how the stringification proceeds.

[^protip]: Pro tip: Given a deep tree `deep_tree`, running `JSON.stringify(deep_tree, 0, 2)` in the JS console is a quick way to make it human readable.

We need to treat the vertices and edges a bit differently, so we're going to manually merge the two sides into a single JSON string.

```javascript
Dagoba.jsonify = function(graph) {
  return '{"V":' + JSON.stringify(graph.vertices, Dagoba.cleanVertex)
       + ',"E":' + JSON.stringify(graph.edges,    Dagoba.cleanEdge)
       + '}'
}
```

And these are the replacers for vertices and edges.

```javascript
Dagoba.cleanVertex = function(key, value) {
  return (key == '_in' || key == '_out') ? undefined : value
}

Dagoba.cleanEdge = function(key, value) {
  return (key == '_in' || key == '_out') ? value._id : value
}
```

The only difference between them is what they do when a cycle is about to be formed: for vertices, we skip the edge list entirely. For edges, we replace each vertex with its ID. That gets rid of all the cycles we created while building the graph.

We're manually manipulating JSON in `Dagoba.jsonify`, which generally isn't recommended as the JSON format is rather persnickety. Even in a dose this small it's easy to miss something and hard to visually confirm correctness.

We could merge the two replacer functions into a single function, and use that new replacer function over the whole graph by doing `JSON.stringify(graph, my_cool_replacer)`. This frees us from having to manually massage the JSON output, but the resulting code may be quite a bit messier. Try it yourself and see if you can come up with a well-factored solution that avoids hand-coded JSON. (Bonus points if it fits in a tweet.)


## Persistence

Persistence is usually one of the trickier parts of a database: disks are relatively safe but slow. Batching writes, making them atomic, journaling&mdash;these are difficult to make both fast and correct.

Fortunately, we're building an *in-memory* database, so we don't have to worry about any of that! We may, though, occasionally want to save a copy of the database locally for fast restart on page load. We can use the serializer we just built to do exactly that. First let's wrap it in a helper function:

```javascript
Dagoba.G.toString = function() { return Dagoba.jsonify(this) }
```

In JavaScript an object's `toString` function is called whenever that object is coerced into a string. So if `g` is a graph, then `g+''` will be the graph's serialized JSON string.

The `fromString` function isn't part of the language specification, but it's handy to have around.

```javascript
Dagoba.fromString = function(str) {             // another graph constructor
  var obj = JSON.parse(str)                     // this can throw
  return Dagoba.graph(obj.V, obj.E)
}
```

Now we'll use those in our persistence functions. The `toString` function is hiding&mdash;can you spot it?

```javascript
Dagoba.persist = function(graph, name) {
  name = name || 'graph'
  localStorage.setItem('DAGOBA::'+name, graph)
}

Dagoba.depersist = function (name) {
  name = 'DAGOBA::' + (name || 'graph')
  var flatgraph = localStorage.getItem(name)
  return Dagoba.fromString(flatgraph)
}
```

We preface the name with a faux namespace to avoid polluting the `localStorage` properties of the domain, as it can get quite crowded in there. There's also usually a low storage limit, so for larger graphs we'd probably want to use a Blob of some sort.

There are also potential issues if multiple browser windows from the same domain are persisting and depersisting simultaneously. The `localStorage` space is shared between those windows, and they're potentially on different event loops, so there's the possibility of one carelessly overwriting the work of another. The spec says there should be a mutex required for read/write access to `localStorage`, but it's inconsistently implemented between different browsers, and even with it a simple implementation like ours could still encounter issues.

If we wanted our persistence implementation to be multi-window–concurrency aware, then we could make use of the storage events that are fired when `localStorage` is changed to update our local graph accordingly.


## Updates

Our `out` pipetype copies the vertex's out-going edges and pops one off each time it needs one. Building that new data structure takes time and space, and pushes more work on to the memory manager. We could have instead used the vertex's out-going edge list directly, keeping track of our place with a counter variable. Can you think of a problem with that approach?

If someone deletes an edge we've visited while we're in the middle of a query, that would change the size of our edge list, and we'd then skip an edge because our counter would be off. To solve this we could lock the vertices involved in our query, but then we'd either lose our capacity to regularly update the graph, or the ability to have long-lived query objects responding to requests for more results on-demand. Even though we're in a single-threaded event loop, our queries can span multiple asynchronous re-entries, which means concurrency concerns like this are a very real problem.

So we'll pay the performance price to copy the edge list. There's still a problem, though, in that long-lived queries may not see a completely consistent chronology. We will traverse every edge belonging to a vertex at the moment we visit it, but we visit vertices at different clock times during our query. Suppose we save a query like `var q = g.v('Odin').children().children().take(2)` and then call `q.run()` to gather two of Odin's grandchildren. Some time later we need to pull another two grandchildren, so we call `q.run()` again. If Odin has had a new grandchild in the intervening time, we may or may not see it, depending on whether the parent vertex was visited the first time we ran the query.

One way to fix this non-determinism is to change the update handlers to add versioning to the data. We'll then change the driver loop to pass the graph's current version in to the query, so we're always seeing a consistent view of the world as it existed when the query was first initialized. Adding versioning to our database also opens the door to true transactions, and automated rollback/retries in an STM-like fashion.


## Future Directions

We saw one way of gathering ancestors earlier:

```javascript
g.v('Thor').out().as('parent')
           .out().as('grandparent')
           .out().as('great-grandparent')
           .merge(['parent', 'grandparent', 'great-grandparent'])
           .run()
```

This is pretty clumsy, and doesn't scale well&mdash;what if we wanted six layers of ancestors? Or to look through an arbitrary number of ancestors until we found what we wanted?

It'd be nice if we could say something like this instead:

```javascript
g.v('Thor').out().all().times(3).run()
```

What we'd like to get out of this is something like the query above&mdash;maybe:

```javascript
g.v('Thor').out().as('a')
           .out().as('b')
           .out().as('c')
           .merge(['a', 'b', 'c'])
           .run()`
```

after the query transformers have all run. We could run the `times` transformer first, to produce:

```javascript
    g.v('Thor').out().all().out().all().out().all().run()
```

Then run the `all` transformer and have it transform each `all` into a uniquely labeled `as`, and put a `merge` after the last `as`.

There are a few problems with this, though. For one, this `as`/`merge` technique only works if every pathway is present in the graph: if we're missing an entry for one of Thor's great-grandparents then we will skip valid entries. For another, what happens if we want to do this to just part of a query and not the whole thing? What if there are multiple `all`s?

To solve that first problem we're going to have to treat `all`s as something more than just as/merge. We need each parent gremlin to actually skip the intervening steps. We can think of this as a kind of teleportation&mdash;jumping from one part of the pipeline directly to another&mdash;or we can think of it as a certain kind of branching pipeline, but either way it complicates our model somewhat. Another approach would be to think of the gremlin as passing through the intervening pipes in a sort of suspended animation, until awoken by a special pipe. Scoping the suspending/unsuspending pipes may be tricky, however.

The next two problems are easier. To modify just part of a query we'll wrap that portion in special start/end steps, like `g.v('Thor').out().start().in().out().end().times(4).run()`. Actually, if the interpreter knows about these special pipetypes we don't need the end step, because the end of a sequence is always a special pipetype. We'll call these special pipetypes "adverbs", because they modify regular pipetypes like adverbs modify verbs.

To handle multiple `all`s we need to run all `all` transformers twice: once before `times`, to mark all `all`s uniquely, and again after `times` to re-mark all marked `all`s uniquely.

There's still the issue of searching through an unbounded number of ancestors&mdash;for example, how do we find out which of Ymir's descendants are scheduled to survive Ragnarök? We could make individual queries like `g.v('Ymir').in().filter({survives: true})` and <latex>\newline</latex> `g.v('Ymir').in().in().in().in().filter({survives: true})`, and manually collect the results ourselves, but that's pretty awful.

We'd like to use an adverb like this:

```javascript
g.v('Ymir').in().filter({survives: true}).every()
```

which would work like `all`+`times` but without enforcing a limit. We may want to impose a particular strategy on the traversal, though, like a stolid BFS or YOLO DFS, so <latex>\newline</latex> `g.v('Ymir').in().filter({survives: true}).bfs()` would be more flexible. Phrasing it this way allows us to state complicated queries like "check for Ragnarök survivors, skipping every other generation" in a straightforward fashion: `g.v('Ymir').in().filter({survives: true}).in().bfs()`.


## Wrapping Up

So what have we learned? Graph databases are great for storing interconnected [^sortainterconnected] data that you plan to query via graph traversals. Adding non-strict semantics allows for a fluent interface over queries you could never express in an eager system for performance reasons, and allows you to cross async boundaries. Time makes things complicated, and time from multiple perspectives (i.e., concurrency) makes things very complicated, so whenever we can avoid introducing a temporal dependency (e.g., state, observable effects, etc.) we make reasoning about our system easier. Building in a simple, decoupled and painfully unoptimized style leaves the door open for global optimizations later on, and using a driver loop allows for orthogonal optimizations&mdash;each without introducing the brittleness and complexity that is the hallmark of most optimization techniques.

That last point can't be overstated: keep it simple. Eschew optimization in favor of simplicity. Work hard to achieve simplicity by finding the right model. Explore many possibilities. The chapters in this book provide ample evidence that highly non-trivial applications can have a small, tight kernel. Once you find that kernel for the application you are building, fight to keep complexity from polluting it. Build hooks for attaching additional functionality, and maintain your abstraction barriers at all costs. Using these techniques well is not easy, but they can give you leverage over otherwise intractable problems.

[^sortainterconnected]: Not *too* interconnected, though&mdash;you'd like the number of edges to grow in direct proportion to the number of vertices. In other words, the average number of edges connected to a vertex shouldn't vary with the size of the graph. Most systems we'd consider putting in a graph database already have this property: if Loki had 100,000 additional grandchildren the degree of the Thor vertex wouldn't increase.


### Acknowledgements

Many thanks are due to Amy Brown, Michael DiBernardo, Colin Lupton, Scott Rostrup, Michael Russo, Erin Toliver, and Leo Zovic for their invaluable contributions to this chapter.
