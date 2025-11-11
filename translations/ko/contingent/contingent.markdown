title: Contingent: 완전히 동적인 빌드 시스템
author: Brandon Rhodes and Daniel Rocco
<markdown>
_Brandon Rhodes는 1990년대 말부터 Python을 사용하기 시작했으며, 17년 동안
아마추어 천문학자들을 위한 PyEphem 라이브러리를 유지보수해왔습니다. 그는
Dropbox에서 일하며, 기업 고객들을 위한 Python 프로그래밍 과정을 가르치고,
New England Wildflower Society의 "Go Botany" Django 사이트와 같은 프로젝트에
컨설팅을 제공했습니다. 또한 2016년과 2017년 PyCon 컨퍼런스의 의장을
맡았습니다. Brandon은 잘 작성된 코드는 문학의 한 형태이고, 아름답게 포맷된
코드는 그래픽 디자인의 작품이며, 올바른 코드는 가장 투명한 형태의 사고라고
믿습니다._

_Daniel Rocco는 Python, 커피, 크래프트 맥주, 스타우트, 객체 및 시스템 디자인,
버번, 교육, 나무, 그리고 라틴 기타를 사랑합니다. 생계를 위해 Python을 쓸 수
있다는 것에 감격하며, 항상 커뮤니티의 다른 사람들로부터 배울 기회를 찾고
지식을 공유함으로써 기여하려 합니다. 그는 PyAtl에서 입문 주제, 테스팅, 디자인,
그리고 새로운 기술들에 대해 자주 발표하며, 누군가가 참신하고 놀랍거나 아름다운
아이디어를 공유할 때 사람들의 눈에서 보이는 경이로움과 기쁨의 반짝임을 보는
것을 좋아합니다. Daniel은 미생물학자와 네 명의 미래의 로켓 과학자들과 함께
애틀랜타에 살고 있습니다._
</markdown>
## 소개

빌드 시스템은 오랫동안 컴퓨터 프로그래밍의
표준 도구였습니다.

ACM 소프트웨어 시스템 상을 수상한
표준 `make` 빌드 시스템은
1976년에 처음 개발되었습니다. 이를 재귀적으로 수행할 수 있게 해줍니다.
A program, for example, might depend upon an object file
which itself depends upon the corresponding source code:

```
    prog: main.o
            cc -o prog main.o

    main.o: main.c
            cc -C -o main.o main.c
```

`make`는 다음 번 실행 시 `main.c` 소스 코드 파일이
`main.o`보다 최근에 수정되었음을 발견하면,
`main.o` 객체 파일을 다시 빌드할 뿐만 아니라
`prog` 자체도 다시 빌드할 것입니다.

빌드 시스템은 전산학과 학부생들에게 주어지는
일반적인 한 학기 프로젝트입니다.
빌드 시스템이 거의 모든 소프트웨어 프로젝트에서 사용되기 때문만이 아니라,
이들의 구성이 방향 그래프와 관련된 기본적인 데이터 구조와
알고리즘을 포함하기 때문입니다
(이 장에서는 나중에 더 자세히 논의할 예정입니다).

빌드 시스템이 수십 년간 사용되고 실용화되어 왔음에도 불구하고,
완전히 범용적이 되어 가장 까다로운 요구사항까지 충족할 수 있게
되었으리라 기대할 수 있을 것입니다.
하지만 사실, 빌드 산출물 간의 일반적인 상호작용의 한 종류인
동적 상호 참조 문제는 대부분의 빌드 시스템에서 너무도 제대로 다루어지지 못하고 있어서,
이 장에서는 `make` 문제를 해결하는 데 사용되는
고전적인 솔루션과 데이터 구조를 재연할 뿐만 아니라,
그 솔루션을 훨씬 더 까다로운 영역으로 극적으로 확장하는 것에
영감을 받았습니다.

다시 말하자면, 문제는 상호 참조입니다.
상호 참조가 주로 어디서 나타나는가요?
텍스트 문서, 문서화, 그리고 인쇄된 책에서입니다! \newpage

## 문제: 문서 시스템 빌드하기

소스로부터 포맷된 문서를 재빌드하는 시스템들은
항상 너무 많은 작업을 하거나, 너무 적은 작업을 하는 것 같습니다.

이들은 사소한 편집에 반응하여
관련 없는 장들이 다시 파싱되고 재포맷되기를
기다리게 만들 때 너무 많은 작업을 합니다.
하지만 때로는 너무 적은 재빌드를 하여
일관성 없는 최종 결과물을 남기기도 합니다.

공식 Python 언어 문서와 Python 커뮤니티의
많은 다른 프로젝트에서 사용되는 문서 빌더인
[Sphinx](http://sphinx-doc.org/)를 생각해 보세요.
A Sphinx project’s `index.rst`
will usually include a table of contents:

```
   Table of Contents
   =================

   .. toctree::

      install.rst
      tutorial.rst
      api.rst
```

이 장 파일명 목록은
Sphinx가 `index.html` 출력 파일을 빌드할 때
언급된 세 개의 장 각각에 대한 링크를 포함하도록 지시합니다.
또한 각 장 내의 섹션들에 대한 링크도 포함할 것입니다.
마크업을 제거하면,
위의 제목으로부터 생성되는 텍스트는
and `toctree` command might be:

```
  Table of Contents

  • Installation

  • Newcomers Tutorial
      • Hello, World
      • Adding Logging

  • API Reference
      • Handy Functions
      • Obscure Classes
```

이 목차는, 보시다시피, 네 개의 다른 파일로부터
가져온 정보들이 혼합된 것입니다.
기본적인 순서와 구조는 `index.rst`에서 나오지만,
각 장과 섹션의 실제 제목들은
세 개의 장 소스 파일 자체에서 가져옵니다.

나중에 튜토리얼의 장 제목을 다시 생각해 보면 —
결국 "newcomer"라는 단어는 너무 구식으로 들리며,
마치 사용자들이 개척 시대 와이오밍에 방금 도착한 정착민인 것 같습니다 —
그러면 `tutorial.rst`의 첫 번째 줄을 편집하여
더 나은 것을 작성할 것입니다:

```
  -Newcomers Tutorial
  +Beginners Tutorial
   ==================

   Welcome to the tutorial!
   This text will take you through the basics of...
```

재빌드할 준비가 되었을 때,
Sphinx는 정확히 올바른 일을 수행할 것입니다!
튜토리얼 장 자체와 인덱스 모두를
재빌드할 것입니다.
(출력을 `cat`으로 파이프하면 Sphinx가
이러한 진행 상황 업데이트로 단일 줄을 반복적으로 덮어쓰는 대신
각 재빌드된 파일을 별도의 줄에 발표하게 됩니다.)

```
   $ make html | cat
   writing output... [ 50%] index
   writing output... [100%] tutorial
```

Sphinx가 두 문서 모두를 재빌드하도록 선택했기 때문에,
`tutorial.html`은 이제 상단에 새로운 제목을 표시할 뿐만 아니라,
출력 `index.html`도 목차에서 업데이트된 장 제목을
표시할 것입니다.
Sphinx는 출력이 일관되도록 모든 것을 재빌드했습니다.

`tutorial.rst`에 대한 편집이 더 사소하다면 어떨까요?

```
   Beginners Tutorial
   ==================

  -Welcome to the tutorial!
  +Welcome to our project tutorial!
   This text will take you through the basics of...
```

이 경우 `index.html`을 재빌드할 필요가 없습니다.
왜냐하면 문단 내부의 이런 사소한 편집은
목차의 어떤 정보도 변경하지 않기 때문입니다.
하지만 Sphinx는 처음에 나타났을지도 모르는 만큼
그렇게 영리하지 않은 것으로 밝혀졌습니다!
결과 내용이 정확히 동일할 것임에도 불구하고
`index.html`을 재빌드하는 중복적인 작업을
계속 진행할 것입니다.

```
   writing output... [ 50%] index
   writing output... [100%] tutorial
```

`index.html`의 "이전"과 "이후" 버전에 대해
`diff`를 실행하여 작은 편집이
첫 페이지에 영향을 미치지 않았음을 확인할 수 있습니다 —
그럼에도 Sphinx는 어쨌든 재빌드하는 동안 기다리게 만들었습니다.

컴파일하기 쉬운 작은 문서에 대해서는
추가 재빌드 노력을 눈치채지 못할 수도 있습니다.
하지만 길고 복잡하거나 플롯이나 애니메이션과 같은
멀티미디어 생성을 포함하는 문서에 대해
빈번한 조정과 편집을 수행할 때는
워크플로우의 지연이 상당해질 수 있습니다.
Sphinx는 적어도 단일 변경을 수행할 때
모든 장을 재빌드하지 않으려고 노력하고 있습니다 —
예를 들어, `tutorial.rst` 편집에 대응하여
`install.html`이나 `api.html`을 재빌드하지 않았습니다 —
하지만 필요한 것보다 더 많은 일을 수행하고 있습니다.

하지만 Sphinx가 더욱 나쁜 일을 하는 것으로 밝혀졌습니다:
때때로 너무 적게 작업하여,
사용자들이 알아차릴 수 있는 일관성 없는 출력을 남겨둡니다.

가장 간단한 실패 중 하나를 보려면,
먼저 API 문서의 상단에 상호 참조를 추가하세요:

```
   API Reference
   =============

  +Before reading this, try reading our :doc:`tutorial`!
  +
   The sections below list every function
   and every single class and method offered...
```

목차와 관련해서 항상 그렇듯이 신중하게,
Sphinx는 이 API 레퍼런스 문서와
프로젝트의 `index.html` 홈페이지 모두를 성실히 재빌드할 것입니다:

```
   writing output... [ 50%] api
   writing output... [100%] index
```

`api.html` 출력 파일에서 Sphinx가
튜토리얼 장의 매력적이고 사람이 읽을 수 있는 제목을
상호 참조의 앵커 태그에 포함시켰음을 확인할 수 있습니다:

```html
   <p>Before reading this, try reading our
   <a class="reference internal" href="tutorial.html">
     <em>Beginners Tutorial</em>
   </a>!</p>
```

이제 `tutorial.rst` 파일 상단의 제목을
다시 편집한다면 어떨까요?
*세 개의* 출력 파일을 무효화시킨 것입니다:

1. The title at the top of `tutorial.html` is now out of date,
   so the file needs to be rebuilt.

2. The table of contents in `index.html` still has the old title,
   so that document needs to be rebuilt.

3. The embedded cross reference in the first paragraph of `api.html`
   still has the old chapter title,
   and also needs to be rebuilt.

What does Sphinx do? 

```
   writing output... [ 50%] index
   writing output... [100%] tutorial
```

어라?

두 개의 파일만 재빌드되었고, 세 개가 아닙니다.
Sphinx는 문서를 올바르게 재빌드하지 못했습니다.

이제 HTML을 웹에 푸시하면,
사용자들은 `api.html` 상단의 상호 참조에서
이전 제목을 보게 되지만,
링크를 클릭하여 `tutorial.html` 자체로 이동하면
다른 제목 — 새로운 제목 —을 보게 됩니다.
이는 Sphinx가 지원하는 여러 종류의 상호 참조에서 발생할 수 있습니다:
장 제목, 섹션 제목, 문단,
클래스, 메서드, 그리고 함수들에서 말이죠.

## 빌드 시스템과 일관성

위에서 설명한 문제는 Sphinx만의 것이 아닙니다.
LaTeX와 같은 다른 문서 시스템들을 괴롭힐 뿐만 아니라,
자산들이 우연히 흥미로운 방식으로 상호 참조하게 되면,
유서 깊은 `make` 유틸리티로 단순히 컴파일 단계를
지시하려고 하는 프로젝트들까지도 괴롭힐 수 있습니다.

이 문제는 오래되고 보편적이므로,
그 해결책 역시 똑같이 오랜 계보를 갖고 있습니다:

```bash
   $ rm -r _build/
   $ make html
```

모든 출력을 제거하면,
완전한 재빌드가 보장됩니다!
일부 프로젝트들은 `rm` `-r`을 `clean`이라는 타겟으로 별칭을 만들어서
빠른 `make` `clean`만으로도 슬레이트를 깨끗이 지울 수 있게 합니다.

모든 중간 또는 출력 자산의 모든 사본을 제거함으로써,
강력한 `rm` `-r`은 아무것도 캐시되지 않은 상태로 —
오래된 제품으로 이어질 수 있는 이전 상태의 메모리 없이 —
빌드를 다시 시작하도록 강제할 수 있습니다.

하지만 더 나은 접근법을 개발할 수 있을까요?

빌드 시스템이 한 문서의 소스 코드에서
다른 문서의 텍스트로 전달되는
모든 장 제목, 모든 섹션 제목,
그리고 모든 상호 참조 구문을 감지하는
지속적인 프로세스라면 어떨까요?
단일 소스 파일의 변경 후 다른 문서들을
재빌드할지 여부에 대한 결정이
단순한 추측 대신 정확할 수 있고,
출력을 일관성 없는 상태로 남겨두는 대신
올바를 수 있을 것입니다.

그 결과는 오래된 정적 `make` 도구와 같지만
파일들이 빌드될 때 파일 간의 의존성을 학습하는 —
상호 참조가 추가, 업데이트, 삭제됨에 따라
의존성을 동적으로 추가하고 제거하는 — 시스템이 될 것입니다.

다음 섹션들에서 우리는 Python으로
Contingent라고 명명된 그러한 도구를
구축할 것입니다.
Contingent는 가능한 한 가장 적은 재빌드 단계를 수행하면서
동적 의존성이 존재하는 상황에서 정확성을 보장합니다.
어떤 문제 영역에도 적용할 수 있지만,
위에서 설명한 문제의 작은 버전에 대해 실행할 것입니다.

## 그래프를 만들기 위한 작업 연결

모든 빌드 시스템에는 입력과 출력을 연결하는 방법이 필요합니다.
The three markup texts in our discussion above,
for example,
each produce a corresponding HTML output file.
The most natural way to express these relationships
is as a collection of boxes and arrows —
or, in mathematical terminology, *nodes* and *edges* —
to form a *graph* (\aosafigref{500l.contingent.graph}).

\aosafigure[180pt]{contingent-images/figure1.png}{Three files generated by parsing three input texts.}{500l.contingent.graph}

Each language in which a programmer
might tackle writing a build system
will offer various data structures
with which such a graph of nodes and edges might be represented.

How could we represent such a graph in Python?

The Python language gives priority to four generic data structures
by giving them direct support in the language syntax.
You can create new instances of these big-four data structures
by simply typing their literal representation into your source code,
and their four type objects are available as built-in symbols
that can be used without being imported.

The **tuple** is a read-only sequence
used to hold heterogeneous data —
each slot in a tuple typically means something different.
Here, a tuple holds together a hostname and port number,
and would lose its meaning if the elements were re-ordered:

```python
('dropbox.com', 443)
```

The **list** is a mutable sequence
used to hold homogenous data —
each item usually has the same structure and meaning as its peers.
Lists can be used either to preserve data’s original input order,
or can be rearranged or sorted
to establish a new and more useful order.

```python
['C', 'Awk', 'TCL', 'Python', 'JavaScript']
```

**집합**은 순서를 보존하지 않습니다.
집합은 주어진 값이 추가되었는지 여부만을 기억하고,
몇 번 추가되었는지는 기억하지 않으므로,
데이터 스트림에서 중복을 제거하기 위한
기본 데이터 구조입니다.
예를 들어, 다음 두 집합은 각각 세 개의 요소를 가질 것입니다:

```python
{3, 4, 5}
{3, 4, 5, 4, 4, 3, 5, 4, 5, 3, 4, 5}
```

The **dict** is an associative data structure for storing values
accessible by a key.
Dicts let the programmer chose the key
by which each value is indexed,
instead of using automatic integer indexing as the tuple and list do.
The lookup is backed by a hash table,
which means that dict key lookup runs at the same speed
whether the dict has a dozen or a million keys.

```python
{'ssh': 22, 'telnet': 23, 'domain': 53, 'http': 80}
```

A key to Python’s flexibility
is that these four data structures are composable.
The programmer can arbitrarily nest them inside each other
to produce more complex data stores
whose rules and syntax remain the simple ones
of the underlying tuples, lists, sets, and dicts.

Given that each of our graph edges needs
to know at least its origin node and its destination node,
the simplest possible representation would be a tuple.
The top edge in \aosafigref{500l.contingent.graph} might look like:

```python
    ('tutorial.rst', 'tutorial.html')
```

How can we store several edges?
While our initial impulse might be
to simply throw all of our edge tuples into a list,
that would have disadvantages.
A list is careful to maintain order,
but it is not meaningful to talk about an absolute order
for the edges in a graph.
And a list would be perfectly happy to hold several copies
of exactly the same edge,
even though we only want it to be possible
to draw a single arrow between `tutorial.rst` and `tutorial.html`.
The correct choice is thus the set,
which would have us represent \aosafigref{500l.contingent.graph} as:

```python
    {('tutorial.rst', 'tutorial.html'),
     ('index.rst', 'index.html'),
     ('api.rst', 'api.html')}
```

This would allow quick iteration across all of our edges,
fast insert and delete operations for a single edge,
and a quick way to check whether a particular edge was present.

불행히도, 이것들이 우리에게 필요한 유일한 연산은 아닙니다.

Contingent와 같은 빌드 시스템은
주어진 노드와 그에 연결된 모든 노드들 사이의
관계를 이해해야 합니다.
예를 들어, `api.rst`가 변경될 때,
Contingent는 완전한 빌드를 보장하면서도
수행되는 작업을 최소화하기 위해
그 변경에 의해 영향을 받는 자산들(있다면)을
알아야 합니다.
"`api.rst`의 다운스트림에 있는 노드들은 무엇인가?"라는
질문에 답하기 위해서는
`api.rst`로부터의 *나가는* 엣지들을 검토해야 합니다.

하지만 의존성 그래프를 구축하려면
Contingent가 노드의 *입력*에 대해서도 관심을 가져야 합니다.
예를 들어, 빌드 시스템이 출력 문서 `tutorial.html`을
조립할 때 어떤 입력들이 사용되었을까요?
각 노드에 대한 입력을 관찰함으로써
Contingent는 `api.html`이 `api.rst`에 의존하지만
`tutorial.html`은 그렇지 않다는 것을 알 수 있습니다.
소스가 변경되고 재빌드가 발생하면,
Contingent는 변경된 각 노드의 들어오는 엣지들을
재구축하여 잠재적으로 오래된 엣지들을 제거하고
이번에는 작업이 어떤 자원들을 사용하는지 다시 학습합니다.

Our set-of-tuples does not make answering
either of these questions easy.
If we needed to know the relationship between `api.html`
and the rest of the graph,
we would need to traverse the entire set
looking for edges that start or end at the `api.html` node.

An associative data structure like Python's dict
would make these chores easier
by allowing direct lookup of all the edges from a particular node:

```python
    {'tutorial.rst': {('tutorial.rst', 'tutorial.html')},
     'tutorial.html': {('tutorial.rst', 'tutorial.html')},
     'index.rst': {('index.rst', 'index.html')},
     'index.html': {('index.rst', 'index.html')},
     'api.rst': {('api.rst', 'api.html')},
     'api.html': {('api.rst', 'api.html')}}
```

Looking up the edges of a particular node would now be blazingly fast,
at the cost of having to store every edge twice:
once in a set of incoming edges,
and once in a set of outgoing edges.
But the edges in each set would have to be examined manually
to see which are incoming and which are outgoing.
It is also slightly redundant to keep naming the node
over and over again in its set of edges.

The solution to both of these objections
is to place incoming and outgoing edges
in their own separate data structures,
which will also absolve us
of having to mention the node over and over again
for every one of the edges in which it is involved.

```python
    incoming = {
        'tutorial.html': {'tutorial.rst'},
        'index.html': {'index.rst'},
        'api.html': {'api.rst'},
        }

    outgoing = {
        'tutorial.rst': {'tutorial.html'},
        'index.rst': {'index.html'},
        'api.rst': {'api.html'},
        }
```

Notice that `outgoing` represents, directly in Python syntax,
exactly what we drew in \aosafigref{500l.contingent.graph} earlier:
the source documents on the left
will be transformed by the build system into the
output documents on the right.
For this simple example each source points to only one output —
all the output sets have only one element —
but we will see examples shortly where a single input node
has multiple downstream consequences.

Every edge in this dictionary-of-sets data structure
does get represented twice,
once as an outgoing edge from one node
(`tutorial.rst` → `tutorial.html`)
and again as an incoming edge to the other
(`tutorial.html` ← `tutorial.rst`).
These two representations capture precisely the same relationship,
just from the opposite perspectives of the two nodes
at either end of the edge.
But in return for this redundancy,
the data structure supports the fast lookup that Contingent needs.

## 클래스의 적절한 사용

You may have been surprised
by the absence of classes in the above discussion
of Python data structures.
After all, classes are a frequent mechanism for structuring applications
and a hardly less-frequent subject of heated debate
among their adherents and detractors.
Classes were once thought important enough that
entire educational curricula were designed around them,
and the majority of popular programming languages
include dedicated syntax for defining and using them.

But it turns out that classes are often orthogonal
to the question of data structure design.
Rather than offering us an entirely alternative data modeling paradigm,
classes simply repeat data structures that we have already seen:

* A class instance is *implemented* as a dict.
* A class instance is *used* like a mutable tuple.

The class offers key lookup through a prettier syntax,
where you get to say `graph.incoming`
instead of `graph["incoming"]`.
But, in practice, class instances are almost never used
as generic key-value stores.
Instead, they are used to organize related but heterogeneous data
by attribute name,
with implementation details encapsulated behind
a consistent and memorable interface.

So instead of putting a hostname and a port number together in a tuple
and having to remember which came first and which came second,
you create an `Address` class
whose instances each have a `host` and a `port` attribute.
You can then pass `Address` objects around
where otherwise you would have had anonymous tuples.
코드는 읽기 쉽고 쓰기도 쉬워집니다.
하지만 클래스 인스턴스를 사용한다고 해서
위에서 데이터 설계를 할 때 직면했던 질문들이
실제로 바뀌는 것은 아닙니다;
단지 더 예쁘고 덜 익명인 컨테이너를 제공할 뿐입니다.

The true value of classes, then,
is not that they change the science of data design.
The value of classes
is that they let you *hide* your data design from the rest of a program!

Successful application design
hinges upon our ability to exploit
the powerful built-in data structures Python offers us
while minimizing the volume of details we are required to
keep in our heads at any one time.
Classes provide the mechanism for resolving this apparent quandary:
used effectively, a class provides a facade
around some small subset of the system's overall design.
When working within one subset — a `Graph`, for example —
we can forget the implementation details of other subsets
as long as we can remember their interfaces.
In this way, programmers often find themselves navigating
among several levels of abstraction
in the course of writing a system,
now working with the specific data model and implementation details
for a particular subsystem,
now connecting higher-level concepts through their interfaces.

For example, from the outside,
code can simply ask for a new `Graph` instance:

```python
>>> from contingent import graphlib
>>> g = graphlib.Graph()
```

without needing to understand the details of how `Graph` works.
Code that is simply using the graph
sees only interface verbs — the method calls —
when manipulating a graph,
as when an edge is added or some other operation performed:

```python
>>> g.add_edge('index.rst', 'index.html')
>>> g.add_edge('tutorial.rst', 'tutorial.html')
>>> g.add_edge('api.rst', 'api.html')
```

Careful readers will have noticed that we added edges to our graph
without explicitly creating “node” and “edge” objects,
and that the nodes themselves in these early examples
are simply strings.
Coming from other languages and traditions,
one might have expected to see
user-defined classes and interfaces for everything in the system:

```java
    Graph g = new ConcreteGraph();
    Node indexRstNode = new StringNode("index.rst");
    Node indexHtmlNode = new StringNode("index.html");
    Edge indexEdge = new DirectedEdge(indexRstNode, indexHtmlNode);
    g.addEdge(indexEdge);
```

The Python language and community explicitly and intentionally emphasize
using simple, generic data structures to solve problems,
instead of creating custom classes for every minute detail
of the problem we want to tackle.
This is one facet of the notion of “Pythonic” solutions:
Pythonic solutions try to
minimize syntactic overhead
and leverage Python's powerful built-in tools
and extensive standard library.

With these considerations in mind,
let’s return to the `Graph` class,
examining its design and implementation to see
the interplay between data structures and class interfaces.
When a new `Graph` instance is constructed,
a pair of dictionaries has already been built
to store edges using the logic we outlined in the previous section:

```python
class Graph:
    """A directed graph of the relationships among build tasks."""

    def __init__(self):
        self._inputs_of = defaultdict(set)
        self._consequences_of = defaultdict(set)
```

The leading underscore
in front of the attribute names `_inputs_of` and `_consequences_of`
is a common convention in the Python community
to signal that an attribute is private.
This convention is one way the community suggests
that programmers pass messages and warnings
through space and time to each other.
Recognizing the need to signal differences between
public and internal object attributes,
the community adopted the single leading underscore
as a concise and fairly consistent indicator
to other programmers,
including our future selves,
that the attribute is best treated
as part of the invisible internal machinery of the class.

Why are we using a `defaultdict` instead of a standard dict?
A common problem when composing dicts
with other data structures is handling missing keys.
With a normal dict,
retrieving a key that does not exist raises a `KeyError`:

```python
>>> consequences_of = {}
>>> consequences_of['index.rst'].add('index.html')
Traceback (most recent call last):
     ...
KeyError: 'index.rst'
```

Using a normal dict requires special checks throughout the code
to handle this specific case, for example when adding a new edge:

```python
    # Special case to handle “we have not seen this task yet”:

    if input_task not in self._consequences_of:
        self._consequences_of[input_task] = set()

    self._consequences_of[input_task].add(consequence_task)
```

This need is so common that Python includes a special utility,
the `defaultdict`, which lets you provide a function
that returns a value for absent keys.
When we ask about an edge that the `Graph` hasn't yet seen,
we will get back an empty `set` instead of an exception:

```python
>>> from collections import defaultdict
>>> consequences_of = defaultdict(set)
>>> consequences_of['api.rst']
set()
```

Structuring our implementation this way means that
each key’s first use can look identical
to second and subsequent times that a particular key is used:

```python
>>> consequences_of['index.rst'].add('index.html')
>>> 'index.html' in consequences_of['index.rst']
True
```

Given these techniques, let’s examine the implementation
of `add_edge`, which we earlier used
to build the graph for \aosafigref{500l.contingent.graph}.

```python
    def add_edge(self, input_task, consequence_task):
        """Add an edge: `consequence_task` uses the output of `input_task`."""
        self._consequences_of[input_task].add(consequence_task)
        self._inputs_of[consequence_task].add(input_task)
```

This method hides the fact that two, not one,
storage steps are required for each new edge
so that we know about it in both directions.
And notice how `add_edge()` does not know or care
whether either node has been seen before.
Because the inputs and consequences data structures
are each a `defaultdict(set)`,
the `add_edge()` method remains blissfully ignorant
as to the novelty of a node —
the `defaultdict` takes care of the difference
by creating a new `set` object on the fly.
As we saw above, `add_edge()` would be
three times longer had we not used `defaultdict`.
More importantly, it would be more difficult
to understand and reason about the resulting code.
This implementation demonstrates a Pythonic
approach to problems: simple, direct, and concise.

Callers should also be given a simple way to visit every edge
without having to learn how to traverse our data structure:

```python
    def edges(self):
        """Return all edges as ``(input_task, consequence_task)`` tuples."""
        return [(a, b) for a in self.sorted(self._consequences_of)
                       for b in self.sorted(self._consequences_of[a])]
```

The `Graph.sorted()` method
makes an attempt to sort the nodes
in a natural sort order
(such as alphabetical)
that can provide a stable output order for the user.

By using this traversal method we can see that,
following our three “add” method calls earlier,
`g` now represents the same graph that we saw in \aosafigref{500l.contingent.graph}.

```python
>>> from pprint import pprint
>>> pprint(g.edges())
[('api.rst', 'api.html'),
 ('index.rst', 'index.html'),
 ('tutorial.rst', 'tutorial.html')]
```

Since we now have a real live Python object,
and not just a figure,
we can ask it interesting questions!
For example, when Contingent is building a blog from source files,
it will need to know things like “What depends on `api.rst`?” when
the content of `api.rst` changes:

```python
>>> g.immediate_consequences_of('api.rst')
['api.html']
```

This `Graph` is telling Contingent that,
when `api.rst` changes,
`api.html` is now stale and must be rebuilt.

How about `index.html`?

```python
>>> g.immediate_consequences_of('index.html')
[]
```

An empty list has been returned,
signalling that `index.html` is at the right edge of the graph
and so nothing further needs to be rebuilt if it changes.
This query can be expressed very simply
thanks to the work that has already gone in to laying out our data:

```python
    def immediate_consequences_of(self, task):
        """Return the tasks that use `task` as an input."""
        return self.sorted(self._consequences_of[task])
```

```python
 >>> from contingent.rendering import as_graphviz
 >>> open('figure1.dot', 'w').write(as_graphviz(g)) and None
```

\aosafigref{500l.contingent.graph} ignored one of the most important relationships
that we discovered in the opening section of our chapter:
the way that document titles appear in the table of contents.
이 세부 사항을 채워봅시다.
입력 파일을 파싱하여 생성되고
다른 루틴 중 하나에 전달되어야 하는
각 제목 문자열에 대한 노드를 만들 것입니다:

```python
>>> g.add_edge('api.rst', 'api-title')
>>> g.add_edge('api-title', 'index.html')
>>> g.add_edge('tutorial.rst', 'tutorial-title')
>>> g.add_edge('tutorial-title', 'index.html')
```

The result is a graph (\aosafigref{500l.contingent.graph2}) that could properly handle
rebuilding the table of contents that we discussed
in the opening of this chapter.

\aosafigure[240pt]{contingent-images/figure2.png}{Being prepared to rebuild `index.html` whenever any title that it mentions gets changed.}{500l.contingent.graph2}

This manual walk-through illustrates what we
will eventually have Contingent do for us:
the graph `g` captures the inputs and consequences
for the various artifacts in our project's documentation.

## 연결 학습

We now have a way for Contingent
to keep track of tasks and the relationships between them.
If we look more closely at \aosafigref{500l.contingent.graph2}, however,
we see that it is actually a little hand-wavy and vague:
*how* is `api.html` produced from `api.rst`?
How do we know that `index.html` needs the title from the tutorial?
And how is this dependency resolved?

Our intuitive notion of these ideas
served when we were constructing consequences graphs by hand,
but unfortunately computers are not terribly intuitive,
so we will need to be more precise about what we want.

What are the steps required to produce output from sources?
How are these steps defined and executed?
And how can Contingent know the connections between them?

Contingent에서 빌드 작업은 함수와 인수로 모델링됩니다.
함수는 특정 프로젝트가 수행하는 방법을
이해하는 동작을 정의합니다.
인수는 구체적인 내용을 제공합니다:
*어떤* 소스 문서를 읽어야 하는지,
*어떤* 블로그 제목이 필요한지 등입니다.
실행되는 동안,
이러한 함수들은 차례로 *다른* 작업 함수들을 호출하여,
답이 필요한 모든 인수를 전달할 수 있습니다.

To see how this works, we will actually now implement
the documentation builder described at the beginning of the chapter.
In order to prevent ourselves from wallowing around in a bog of details,
for this illustration we will work with
simplified input and output document formats.
Our input documents will consist of a title on the first line,
with the remainder of the text forming the body.
Cross references will simply be source file names
enclosed in backticks,
which on output are replaced with the title
from the corresponding document in the output.

Here is the content of our example
`index.txt`, `api.txt`, and `tutorial.txt`,
illustrating titles, document bodies, and cross-references
from our little document format:

```python
>>> index = """
... Table of Contents
... -----------------
... * `tutorial.txt`
... * `api.txt`
... """

>>> tutorial = """
... Beginners Tutorial
... ------------------
... Welcome to the tutorial!
... We hope you enjoy it.
... """

>>> api = """
... API Reference
... -------------
... You might want to read
... the `tutorial.txt` first.
... """
```

Now that we have some source material to work with,
what functions would a Contingent-based blog builder
need?

In the simple examples above,
the HTML output files proceed directly from the source,
but in a realistic system,
turning source into markup involves several steps:
reading the raw text from disk,
parsing the text to a convenient internal representation,
processing any directives the author may have specified,
resolving cross-references or other external dependencies
(such as include files),
and applying one or more view transformations
to convert the internal representation to its output form.

Contingent manages tasks by grouping them into a `Project`,
a sort of build system busybody
that injects itself into the middle of the build process,
noting every time one task talks to another
to construct a graph of the relationships between all the tasks.

```python
>>> from contingent.projectlib import Project, Task
>>> project = Project()
>>> task = project.task
```

A build system for the example given at the beginning of the chapter
might involve a few tasks.

우리의 `read()` 작업은 디스크에서 파일을 읽는 척할 것입니다.
실제로는 소스 텍스트를 변수로 정의했으므로,
파일명에서 해당 텍스트로
변환하는 것이 전부입니다.

```python
  >>> filesystem = {'index.txt': index,
  ...               'tutorial.txt': tutorial,
  ...               'api.txt': api}
  ...
  >>> @task
  ... def read(filename):
  ...     return filesystem[filename]
```

The `parse()` task interprets the raw text of the file contents
according to the specification of our document format.
Our format is very simple:
the title of the document appears on the first line,
and the rest of the content is considered the document's body.

```python
  >>> @task
  ... def parse(filename):
  ...     lines = read(filename).strip().splitlines()
  ...     title = lines[0]
  ...     body = '\n'.join(lines[2:])
  ...     return title, body
```

Because the format is so simple,
the parser is a little silly, 
but it illustrates the interpretive responsibilities
that parsers are required to carry out.
(Parsing in general is a very interesting subject
and many books have been written
either partially or completely about it.)
In a system like Sphinx,
the parser must understand the many markup tokens,
directives, and commands defined by the system,
transforming the input text into something
the rest of the system can work with.

Notice the connection point between
`parse()` and `read()` —
the first task in parsing is to pass the filename it has been given
to `read()`, which finds and returns the contents of that file.

The `title_of()` task, given a source file name,
returns the document's title:

```python
  >>> @task
  ... def title_of(filename):
  ...     title, body = parse(filename)
  ...     return title
```

This task nicely illustrates the
separation of responsibilities between
the parts of a document processing system.
The `title_of()` function works directly
from an in-memory representation of a document —
in this case, a tuple —
instead of taking it upon itself to re-parse
the entire document again just to find the title.
The `parse()` function alone produces the in-memory representation,
in accordance with the contract of the system specification,
and the rest of the blog builder processing functions
like `title_of()` simply use its output as their authority.

If you are coming from an orthodox object-oriented tradition,
this function-oriented design may look a little weird.
In an OO solution,
`parse()` would return some sort of `Document` object
that has `title_of()` as a method or property.
In fact, Sphinx works exactly this way:
its `Parser` subsystem produces a “Docutils document tree” object
for the other parts of the system to use.

Contingent is not opinionated
with regard to these differing design paradigms
and supports either approach equally well.
이 장에서는 단순하게 유지하겠습니다.

마지막 작업인
`render()`는
문서의 메모리 내 표현을
출력 형태로 변환합니다.
사실상 `parse()`의 역과정입니다.
Whereas `parse()` takes an input document
conforming to a specification
and converts it to an in-memory representation,
`render()` takes an in-memory representation
and produces an output document
conforming to some specification.

```python
  >>> import re
  >>>
  >>> LINK = '<a href="{}">{}</a>'
  >>> PAGE = '<h1>{}</h1>\n<p>\n{}\n<p>'
  >>>
  >>> def make_link(match):
  ...     filename = match.group(1)
  ...     return LINK.format(filename, title_of(filename))
  ...
  >>> @task
  ... def render(filename):
  ...     title, body = parse(filename)
  ...     body = re.sub(r'`([^`]+)`', make_link, body)
  ...     return PAGE.format(title, body)
```  

Here is an example run
that will invoke every stage of the above logic —
rendering `tutorial.txt` to produce its output:

```python
>>> print(render('tutorial.txt'))
<h1>Beginners Tutorial</h1>
<p>
튜토리얼에 오신 것을 환영합니다!
즐겁게 읽으시길 바랍니다.
<p>
```

\aosafigref{500l.contingent.graph3} illustrates the task graph
that transitively connects all the tasks
required to produce the output,
from reading the input file,
to parsing and transforming the document,
and rendering it:

\aosafigure[240pt]{contingent-images/figure3.png}{A task graph.}{500l.contingent.graph3}

It turns out that \aosafigref{500l.contingent.graph3} was not hand-drawn for this chapter,
but has been generated directly from Contingent!
Building this graph is possible for the `Project` object
because it maintains its own call stack,
similar to the stack of live execution frames
that Python maintains to remember which function to continue running
when the current one returns.

Every time a new task is invoked,
Contingent can assume that it has been called —
and that its output will be used —
by the task currently at the top of the stack.
Maintaining the stack will require that several extra steps
surround the invocation of a task *T*:

1. Push *T* onto the stack.
2. Execute *T*, letting it call any other tasks it needs.
3. Pop *T* off the stack.
4. Return its result.

To intercept task calls,
the `Project` leverages a key Python feature: *function decorators*.
A decorator is allowed to process or transform a function
at the moment that it is being defined.
The `Project.task` decorator uses this opportunity
to package every task inside another function, a *wrapper*,
which allows a clean separation of responsibilities
between the wrapper —
which will worry about graph and stack management
on behalf of the Project —
and our task functions that focus on document processing.
Here is what the `task` decorator boilerplate looks like:

```python
        from functools import wraps

        def task(function):
            @wraps(function)
            def wrapper(*args):
                # wrapper body, that will call function()
            return wrapper
```

이는 완전히 일반적인 Python 데코레이터 선언입니다.
그런 다음 함수를 생성하는 `def`
위에 `@` 문자 뒤에 이름을 적어서
함수에 적용할 수 있습니다:

```python
    @task
    def title_of(filename):
        title, body = parse(filename)
        return title
```

When this definition is complete,
the name `title_of` will refer
to the wrapped version of the function.
The wrapper can access the original version of the function
via the name `function`,
calling it at the appropriate time.
The body of the Contingent wrapper
runs something like this:

```python
    def task(function):
        @wraps(function)
        def wrapper(*args):
            task = Task(wrapper, args)
            if self.task_stack:
                self._graph.add_edge(task, self.task_stack[-1])
            self._graph.clear_inputs_of(task)
            self._task_stack.append(task)
            try:
                value = function(*args)
            finally:
                self._task_stack.pop()

            return value
        return wrapper
```

This wrapper performs several crucial maintenance steps:

1. Packages the task —
   a function plus its arguments —
   into a small object for convenience.
   The `wrapper` here names the wrapped version of the task function.

2. If this task has been invoked
   by a current task that is already underway,
   add an edge capturing the fact that
   this task is an input to the already-running task.

3. Forget whatever we might have learned last time about the task,
   since it might make new decisions this time —
   if the source text of the API guide no longer mentions the Tutorial,
   for example, then its `render()` will no longer ask
   for the `title_of()` the Tutorial document.

4. Push this task onto the top of the task stack
   in case it decides, in its turn, to invoke further tasks
   in the course of doing its work.

5. Invoke the task
   inside of a `try...finally` block
   that ensures we correctly remove the finished task from the stack,
   even if it dies by raising an exception.

6. Return the task’s return value,
   so that callers of this wrapper
   will not be able to tell that they have not simply invoked
   the plain task function itself.

Steps 4 and 5 maintain the task stack itself,
which is then used by step 2 to perform the consequences tracking
that is our whole reason for building a task stack in the first place.

Since each task gets surrounded by its own copy of the wrapper function,
the mere invocation and execution of the normal stack of tasks
will produce a graph of relationships as an invisible side effect.
That is why we were careful to use the wrapper
around each processing step that we defined:

```python
    @task
    def read(filename):
        # body of read

    @task
    def parse(filename):
        # body of parse

    @task
    def title_of(filename):
        # body of title_of

    @task
    def render(filename):
        # body of render
```

Thanks to these wrappers,
when we called `parse('tutorial.txt')`
the decorator learned
the connection between `parse` and `read`.
We can ask about the relationship by building another `Task` tuple
and asking what the consequences would be
if its output value changed:

```python
>>> task = Task(read, ('tutorial.txt',))
>>> print(task)
read('tutorial.txt')
>>> project._graph.immediate_consequences_of(task)
[parse('tutorial.txt')]
```

The consequence of re-reading the `tutorial.txt` file
and finding that its contents have changed
is that we need to re-execute the `parse()` routine for that document.
What happens if we render the entire set of documents?
Will Contingent be able to learn the entire build process?

```python
>>> for filename in 'index.txt', 'tutorial.txt', 'api.txt':
...     print(render(filename))
...     print('=' * 30)
...
<h1>Table of Contents</h1>
<p>
* <a href="tutorial.txt">Beginners Tutorial</a>
* <a href="api.txt">API Reference</a>
<p>
==============================
<h1>Beginners Tutorial</h1>
<p>
튜토리얼에 오신 것을 환영합니다!
즐겁게 읽으시길 바랍니다.
<p>
==============================
<h1>API Reference</h1>
<p>
You might want to read
the <a href="tutorial.txt">Beginners Tutorial</a> first.
<p>
==============================
```

It worked!
From the output, we can see that
our transform substituted the document titles
for the directives in our source documents,
indicating that Contingent was able to
discover the connections between the various tasks
needed to build our documents.

\aosafigure[240pt]{contingent-images/figure4.png}{The complete set of relationships between our input files and our HTML outputs.}{500l.contingent.graph4}

By watching one task invoke another
through the `task` wrapper machinery,
`Project` has automatically learned
the graph of inputs and consequences.
Since it has a complete consequences graph
at its disposal,
Contingent knows all the things to rebuild
if the inputs to any tasks change.

## 결과 추적

Once the initial build has run to completion,
Contingent는 입력 파일의 변경사항을 모니터링해야 합니다.
사용자가 새로운 편집을 마치고 "저장"을 실행하면,
`read()` 메서드와 그 결과들이 모두 호출되어야 합니다.

This will require us to walk the graph in the opposite order
from the one in which it was created.
It was built, you will recall, by calling
`render()` for the API Reference and having that call `parse()`
which finally invoked the `read()` task.
Now we go in the other direction:
we know that `read()` will now return new content,
and we need to figure out what consequences lie downstream.

The process of compiling consequences is a recursive one,
as each consequence can itself have further tasks that depended on it.
We could perform this recursion manually
through repeated calls to the graph.
(Note that we are here taking advantage
of the fact that the Python prompt saves the last value displayed
under the name `_` for use in the subsequent expression.)

```python
>>> task = Task(read, ('api.txt',))
>>> project._graph.immediate_consequences_of(task)
[parse('api.txt')]
>>> t1, = _
>>> project._graph.immediate_consequences_of(t1)
[render('api.txt'), title_of('api.txt')]
>>> t2, t3 = _
>>> project._graph.immediate_consequences_of(t2)
[]
>>> project._graph.immediate_consequences_of(t3)
[render('index.txt')]
>>> t4, = _
>>> project._graph.immediate_consequences_of(t4)
[]
```

This recursive task of looking repeatedly for immediate consequences
and only stopping when we arrive at tasks with no further consequences
is a basic enough graph operation that it is supported directly
by a method on the `Graph` class:

```python
>>> # Secretly adjust pprint to a narrower-than-usual width:
>>> _pprint = pprint
>>> pprint = lambda x: _pprint(x, width=40)
>>> pprint(project._graph.recursive_consequences_of([task]))
[parse('api.txt'),
 render('api.txt'),
 title_of('api.txt'),
 render('index.txt')]
```

사실, `recursive_consequences_of()`는 조금 영리합니다.
특정 작업이 여러 다른 작업들의 다운스트림 결과로
반복적으로 나타나면,
출력 목록에서 한 번만 언급하도록 주의하며,
그것의 입력인 작업들 이후에만 나타나도록
끝 부분으로 이동시킵니다.
이러한 지능은 위상 정렬의 고전적인 깊이 우선 구현에 의해
작동되며,
이는 숨겨진 재귀 도우미 함수를 통해
Python에서 작성하기 상당히 쉬운 알고리즘입니다.
자세한 내용은 `graphlib.py` 소스 코드를 확인해보세요.

If, upon detecting a change,
we are careful to re-run every task in the recursive consequences,
then Contingent will be able to avoid rebuilding too little.
Our second challenge, however,
was to avoid rebuilding too much.
\aosafigref{500l.contingent.graph4}를 다시 참조해보세요.
`tutorial.txt`가 변경될 때마다 세 개의 문서를
모두 재빌드하는 것을 피하고 싶습니다.
대부분의 편집은 아마도 제목에는 영향을 주지 않고
본문에만 영향을 줄 것이기 때문입니다.
어떻게 이를 달성할 수 있을까요?

해결책은 그래프 재계산을 캐싱에 의존하게 만드는 것입니다.
When stepping forward through the recursive consequences of a change,
we will only invoke tasks whose inputs are different than last time.

이 최적화에는 마지막 데이터 구조가 포함될 것입니다.
`Project`에 `_todo` 집합을 제공하여
적어도 하나의 입력 값이 변경되어
재실행이 필요한 모든 작업을
기억하도록 할 것입니다.
`_todo`에 있는 작업만이 구식이므로,
빌드 프로세스는 그곳에 나타나지 않는 한
모든 작업의 실행을 건너뛸 수 있습니다.

Again, Python’s convenient and unified design
makes these features very easy to code.
Because task objects are hashable,
`_todo` can simply be a set
that remembers task items by identity —
guaranteeing that a task never appears twice —
and the `_cache` of return values from previous runs
can be a dict with tasks as keys.

More precisely, the rebuild step must keep looping
as long as `_todo` is non-empty.
During each loop, it should:

* Call `recursive_consequences_of()`
  and pass in every task listed in `_todo`.
  The return value will be a list
  of not only the `_todo` tasks themselves,
  but also every task downstream of them —
  every task, in other words, that could possibly need re-execution
  if the outputs come out different this time.

* For each task in the list,
  check whether it is listed in `_todo`.
  If not, then we can skip running it,
  because none of the tasks that we have re-invoked upstream of it
  has produced a new return value
  that would require the task’s recomputation.

* But for any task that is indeed listed in `_todo`
  by the time we reach it,
  we need to ask it to re-run and re-compute its return value.
  If the task wrapper function detects that this return value
  does not match the old cached value,
  then its downstream tasks will be automatically added to `_todo`
  before we reach them in the list of recursive consequences.

By the time we reach the end of the list,
every task that could possibly need to be re-run
should in fact have been re-run.
But just in case, we will check `_todo`
and try again if it is not yet empty.
Even for very rapidly changing dependency trees,
this should quickly settle out.
Only a cycle —
where, for example, task *A* needs the output of task *B*
which itself needs the output of task *A* —
could keep the builder in an infinite loop,
and only if their return values never stabilize.
다행히도, 실제 세계의 빌드 작업은 일반적으로 순환이 없습니다.

예제를 통해 이 시스템의 동작을 추적해 봅시다.

Suppose you edit `tutorial.txt`
and change both the title and the body content.
We can simulate this by modifying the value
in our `filesystem` dict:

```python
>>> filesystem['tutorial.txt'] = """
... The Coder Tutorial
... ------------------
... This is a new and improved
... introductory paragraph.
... """
```

Now that the contents have changed,
we can ask the Project to re-run the `read()` task
by using its `cache_off()` context manager
that temporarily disables its willingness
to return its old cached result for a given task and argument:

```python
>>> with project.cache_off():
...     text = read('tutorial.txt')
```

새로운 튜토리얼 텍스트가 이제 캐시에 읽혔습니다.
얼마나 많은 다운스트림 작업이 재실행되어야 할까요?

To help us answer this question,
the `Project` class supports a simple tracing facility
that will tell us which tasks are executed in the course
of a rebuild.
Since the above change to `tutorial.txt`
affects both its body and its title,
everything downstream will need to be re-computed:

```python
>>> project.start_tracing()
>>> project.rebuild()
>>> print(project.stop_tracing())
calling parse('tutorial.txt')
calling render('tutorial.txt')
calling title_of('tutorial.txt')
calling render('api.txt')
calling render('index.txt')
```

Looking back at \aosafigref{500l.contingent.graph4},
you can see that, as expected,
this is every task that is an immediate or downstream consequence
of `read('tutorial.txt')`.

But what if we edit it again,
but this time leave the title the same?

```python
>>> filesystem['tutorial.txt'] = """
... The Coder Tutorial
... ------------------
... Welcome to the coder tutorial!
... It should be read top to bottom.
... """
>>> with project.cache_off():
...     text = read('tutorial.txt')
```

This small, limited change
should have no effect on the other documents.

```python
>>> project.start_tracing()
>>> project.rebuild()
>>> print(project.stop_tracing())
calling parse('tutorial.txt')
calling render('tutorial.txt')
calling title_of('tutorial.txt')
```

성공입니다!
한 개의 문서만 재빌드되었습니다.
`title_of()`가 새로운 입력 문서를 받았음에도 불구하고
동일한 값을 반환했다는 사실은 모든 추가
다운스트림 작업들이 변경으로부터 격리되어
재호출되지 않았음을 의미합니다.

## 결론

There exist languages and programming methodologies
under which Contingent would be a suffocating forest of tiny classes,
with verbose names given to every concept in the problem domain.

When programming Contingent in Python, however,
we skipped the creation of a dozen possible classes 
like `TaskArgument` and `CachedResult` and `ConsequenceList`.
We instead drew upon Python’s strong tradition
of solving generic problems with generic data structures,
resulting in code that repeatedly uses a small set of ideas
from the core data structures tuple, list, set, and dict.

But does this not cause a problem?

범용 데이터 구조는 그 본질상 익명입니다.
우리의 `project._cache`는 집합입니다.
`Graph` 내부의 모든 업스트림 및 다운스트림 노드 컬렉션도
마찬가지입니다.
범용 `set` 오류 메시지를 보고
오류에 대해 프로젝트 또는 그래프
구현 중 어디를 찾아야 할지 모르는
위험에 처해 있을까요?

In fact, we are not in danger!

Thanks to the careful discipline of encapsulation —
of only allowing `Graph` code to touch the graph’s sets,
and `Project` code to touch the project’s set —
there will never be ambiguity if a set operation
returns an error during a later phase of the project.
The name of the innermost executing method at the moment of the error
will necessarily direct us to exactly the class, and set,
involved in the mistake.
There is no need to create a subclass of `set`
for every possible application of the data type,
so long as we put that conventional underscore in front of data
structure attributes and then are careful not to touch them
from code outside of the class.

Contingent demonstrates how crucial the Facade pattern,
from the epochal *Design Patterns* book,
is for a well-designed Python program.
Not every data structure and fragment of data in a Python program
gets to be its own class.
Instead, classes are used sparingly,
at conceptual pivots in the code where a big idea —
like the idea of a dependency graph —
can be wrapped up into a Facade
that hides the details of the simple generic data structures
that lie beneath it.

Code outside of the Facade
names the big concepts that it needs
and the operations that it wants to perform.
Inside of the Facade,
the programmer manipulates the small and convenient moving parts
of the Python programming language to make the operations happen.
