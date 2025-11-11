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

Sphinx는 무엇을 할까요? 

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
위 논의에서 다룬 세 개의 마크업 텍스트는
각각 대응하는 HTML 출력 파일을 생성합니다.
이러한 관계를 표현하는 가장 자연스러운 방법은
상자와 화살표의 집합 —
또는 수학적 용어로 *노드*와 *엣지* —
로 *그래프*를 형성하는 것입니다 (\aosafigref{500l.contingent.graph}).

\aosafigure[180pt]{contingent-images/figure1.png}{Three files generated by parsing three input texts.}{500l.contingent.graph}

프로그래머가 빌드 시스템 작성에 도전하는
모든 언어는 노드와 엣지로 구성된
이런 그래프를 표현할 수 있는
다양한 데이터 구조를 제공합니다.

Python에서는 이런 그래프를 어떻게 표현할 수 있을까요?

Python 언어는 네 가지 범용 데이터 구조에
언어 문법에서 직접 지원을 제공함으로써 우선순위를 부여합니다.
소스 코드에 리터럴 표현을 간단히 입력하는 것만으로
이 네 가지 주요 데이터 구조의 새 인스턴스를 생성할 수 있으며,
네 가지 타입 객체는 가져오기 없이 사용할 수 있는
내장 심볼로 제공됩니다.

**튜플**은 이질적인 데이터를 보관하는 데 사용되는
읽기 전용 시퀀스입니다 —
튜플의 각 슬롯은 일반적으로 서로 다른 의미를 갖습니다.
여기서 튜플은 호스트명과 포트 번호를 함께 보관하며,
요소들이 재정렬되면 의미를 잃게 됩니다:

```python
('dropbox.com', 443)
```

**리스트**는 동질적인 데이터를 보관하는 데 사용되는
변경 가능한 시퀀스입니다 —
각 항목은 일반적으로 동료와 동일한 구조와 의미를 갖습니다.
리스트는 데이터의 원래 입력 순서를 보존하는 데 사용할 수도 있고,
재배열하거나 정렬하여
새롭고 더 유용한 순서를 설정할 수도 있습니다.

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

**딕셔너리**는 키로 접근할 수 있는 값을 저장하기 위한
연관 데이터 구조입니다.
딕셔너리는 튜플과 리스트처럼 자동 정수 인덱싱을 사용하는 대신
각 값이 색인화되는 키를 프로그래머가 선택할 수 있게 해줍니다.
조회는 해시 테이블에 의해 지원되므로,
딕셔너리에 십여 개의 키가 있든 백만 개의 키가 있든
딕셔너리 키 조회는 동일한 속도로 실행됩니다.

```python
{'ssh': 22, 'telnet': 23, 'domain': 53, 'http': 80}
```

Python의 유연성의 핵심은
이 네 가지 데이터 구조가 조합 가능하다는 것입니다.
프로그래머는 이들을 서로 임의로 중첩하여
기반이 되는 튜플, 리스트, 집합, 딕셔너리의
단순한 규칙과 문법을 유지하면서도
더 복잡한 데이터 저장소를 만들 수 있습니다.

우리의 각 그래프 엣지가 최소한
원점 노드와 목적지 노드를 알아야 한다는 점에서,
가장 간단한 표현은 튜플이 될 것입니다.
\aosafigref{500l.contingent.graph}에서 상단 엣지는 다음과 같을 것입니다:

```python
    ('tutorial.rst', 'tutorial.html')
```

여러 엣지를 어떻게 저장할 수 있을까요?
초기 충동으로는 모든 엣지 튜플을
단순히 리스트에 던져 넣을 수도 있지만,
그것은 단점이 있을 것입니다.
A list is careful to maintain order,
but it is not meaningful to talk about an absolute order
for the edges in a graph.
그리고 리스트는 정확히 동일한 엣지의 여러 사본을
완벽하게 기꺼이 보관하게 되지만,
우리는 `tutorial.rst`와 `tutorial.html` 사이에
단일 화살표만 그어질 수 있기를 원합니다.
The correct choice is thus the set,
which would have us represent \aosafigref{500l.contingent.graph} as:

```python
    {('tutorial.rst', 'tutorial.html'),
     ('index.rst', 'index.html'),
     ('api.rst', 'api.html')}
```

이것은 모든 엣지에 대한 빠른 반복,
단일 엣지에 대한 빠른 삽입 및 삭제 연산,
그리고 특정 엣지가 존재하는지 확인하는 빠른 방법을 허용할 것입니다.

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

우리의 튜플 집합은 이러한 질문들 중
어느 것도 쉽게 답할 수 있게 해주지 않습니다.
만약 `api.html`과 그래프의 나머지 부분 사이의
관계를 알아야 한다면,
`api.html` 노드에서 시작하거나 끝나는 엣지들을 찾으면서
전체 집합을 순회해야 할 것입니다.

Python의 딕셔너리와 같은 연관 데이터 구조는
특정 노드로부터의 모든 엣지를 직접 조회할 수 있게 하여
이러한 작업들을 더 쉽게 만들어줄 것입니다:

```python
    {'tutorial.rst': {('tutorial.rst', 'tutorial.html')},
     'tutorial.html': {('tutorial.rst', 'tutorial.html')},
     'index.rst': {('index.rst', 'index.html')},
     'index.html': {('index.rst', 'index.html')},
     'api.rst': {('api.rst', 'api.html')},
     'api.html': {('api.rst', 'api.html')}}
```

특정 노드의 엣지들을 조회하는 것은 이제 매우 빠르겠지만,
모든 엣지를 두 번 저장해야 하는 비용이 따릅니다:
한 번은 들어오는 엣지의 집합에,
그리고 한 번은 나가는 엣지의 집합에.
하지만 각 집합의 엣지들은 어떤 것이 들어오는 것이고
어떤 것이 나가는 것인지 확인하기 위해
수동으로 검토되어야 합니다.
또한 엣지 집합에서 노드를
반복해서 언급하는 것은 약간 중복적입니다.

이 두 가지 반대 의견에 대한 해결책은
들어오는 엣지와 나가는 엣지를
각각의 별도 데이터 구조에 배치하는 것입니다.
이것은 또한 노드가 관련된 모든 엣지에 대해
그 노드를 반복해서 언급해야 하는 부담에서
우리를 해방시켜 줄 것입니다.

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

`outgoing`이 Python 문법으로 직접
앞서 \aosafigref{500l.contingent.graph}에서 그렸던 것을
정확히 표현한다는 점에 주목하세요:
왼쪽의 소스 문서들이
빌드 시스템에 의해 오른쪽의
출력 문서들로 변환될 것입니다.
이 간단한 예제에서는 각 소스가 하나의 출력만을 가리키며 —
모든 출력 집합이 하나의 요소만을 갖습니다 —
하지만 단일 입력 노드가
여러 다운스트림 결과를 갖는 예제를 곧 보게 될 것입니다.

이 딕셔너리-집합 데이터 구조의 모든 엣지는
실제로 두 번 표현됩니다.
한 번은 한 노드에서 나가는 엣지로
(`tutorial.rst` → `tutorial.html`)
그리고 다시 한 번은 다른 노드로 들어오는 엣지로
(`tutorial.html` ← `tutorial.rst`) 표현됩니다.
이 두 표현은 정확히 동일한 관계를 포착하지만,
엣지의 양 끝에 있는 두 노드의
반대 관점에서 표현합니다.
하지만 이런 중복성에 대한 대가로,
이 데이터 구조는 Contingent가 필요로 하는 빠른 조회를 지원합니다.

## 클래스의 적절한 사용

위의 Python 데이터 구조 논의에서
클래스가 없다는 점에 놀랐을 수도 있습니다.
결국 클래스는 애플리케이션을 구조화하는
빈번한 메커니즘이며,
지지자와 반대자들 사이에서 열띤 논쟁의
주제가 되는 것도 그에 못지않게 빈번합니다.
클래스는 한때 매우 중요하게 여겨져서
전체 교육 커리큘럼이 클래스를 중심으로 설계되었고,
대부분의 인기 있는 프로그래밍 언어들이
클래스를 정의하고 사용하기 위한 전용 문법을 포함하고 있습니다.

하지만 클래스는 종종 데이터 구조 설계의 문제와
직교한다는 것이 밝혀졌습니다.
완전히 다른 데이터 모델링 패러다임을 제공하는 대신,
클래스는 우리가 이미 봤던 데이터 구조들을 단순히 반복합니다:

* 클래스 인스턴스는 딕셔너리로 *구현*됩니다.
* 클래스 인스턴스는 변경 가능한 튜플처럼 *사용*됩니다.

클래스는 더 예쁜 문법을 통한 키 조회를 제공하여,
`graph["incoming"]` 대신
`graph.incoming`이라고 말할 수 있게 해줍니다.
하지만 실제로, 클래스 인스턴스는 범용 키-값 저장소로
거의 사용되지 않습니다.
대신, 구현 세부사항이 일관되고 기억하기 쉬운 인터페이스 뒤에
캡슐화된 상태로
속성 이름에 따라 관련되지만 이질적인 데이터를
정리하는 데 사용됩니다.

따라서 호스트명과 포트 번호를 튜플에 함께 넣고
어느 것이 먼저이고 어느 것이 두 번째인지 기억해야 하는 대신,
각 인스턴스가 `host`와 `port` 속성을 갖는
`Address` 클래스를 생성합니다.
그런 다음 익명 튜플을 사용했을 곳에서
`Address` 객체를 전달할 수 있습니다.
코드는 읽기 쉽고 쓰기도 쉬워집니다.
하지만 클래스 인스턴스를 사용한다고 해서
위에서 데이터 설계를 할 때 직면했던 질문들이
실제로 바뀌는 것은 아닙니다;
단지 더 예쁘고 덜 익명인 컨테이너를 제공할 뿐입니다.

따라서 클래스의 진정한 가치는
데이터 설계의 과학을 바꾸는 것이 아닙니다.
클래스의 가치는
프로그램의 나머지 부분으로부터 데이터 설계를
*숨길* 수 있게 해주는 것입니다!

성공적인 애플리케이션 설계는
Python이 제공하는 강력한 내장 데이터 구조를 활용하면서
동시에 한 번에 머릿속에 담아야 하는
세부사항의 양을 최소화하는 능력에 달려 있습니다.
클래스는 이런 명백한 딜레마를 해결하는 메커니즘을 제공합니다:
효과적으로 사용될 때, 클래스는 시스템의 전체 설계 중
일부 작은 부분집합 주위에 파사드를 제공합니다.
한 부분집합 — 예를 들어 `Graph` — 내에서 작업할 때,
인터페이스만 기억할 수 있다면
다른 부분집합들의 구현 세부사항은 잊을 수 있습니다.
이런 방식으로, 프로그래머들은 시스템을 작성하는 과정에서
여러 추상화 수준 사이를 탐색하게 되며,
때로는 특정 서브시스템의 구체적인 데이터 모델과
구현 세부사항을 다루고,
때로는 인터페이스를 통해 고수준 개념들을 연결합니다.

예를 들어, 외부에서는
코드가 단순히 새로운 `Graph` 인스턴스를 요청할 수 있습니다:

```python
>>> from contingent import graphlib
>>> g = graphlib.Graph()
```

`Graph`가 어떻게 작동하는지에 대한 세부사항을 이해할 필요가 없습니다.
그래프를 단순히 사용하는 코드는
엣지를 추가하거나 다른 작업을 수행할 때처럼
그래프를 조작할 때
인터페이스 동사 — 메서드 호출 — 만을 봅니다:

```python
>>> g.add_edge('index.rst', 'index.html')
>>> g.add_edge('tutorial.rst', 'tutorial.html')
>>> g.add_edge('api.rst', 'api.html')
```

주의 깊은 독자들은 우리가 "노드"와 "엣지" 객체를
명시적으로 생성하지 않고 그래프에 엣지를 추가했으며,
이 초기 예제들에서 노드 자체가
단순히 문자열이라는 점을 알아차렸을 것입니다.
다른 언어와 전통에서 오는 사람들은
시스템의 모든 것에 대해 사용자 정의 클래스와
인터페이스를 보기를 기대했을 수도 있습니다:

```java
    Graph g = new ConcreteGraph();
    Node indexRstNode = new StringNode("index.rst");
    Node indexHtmlNode = new StringNode("index.html");
    Edge indexEdge = new DirectedEdge(indexRstNode, indexHtmlNode);
    g.addEdge(indexEdge);
```

Python 언어와 커뮤니티는 명시적으로 그리고 의도적으로
해결하고자 하는 문제의 모든 세부사항에 대해
사용자 정의 클래스를 만드는 대신
문제 해결에 간단하고 범용적인 데이터 구조를
사용하는 것을 강조합니다.
이것은 "파이썬다운" 솔루션 개념의 한 면입니다:
파이썬다운 솔루션은 문법적 오버헤드를 최소화하고
Python의 강력한 내장 도구와
광범위한 표준 라이브러리를 활용하려고 노력합니다.

이러한 고려사항들을 염두에 두고,
데이터 구조와 클래스 인터페이스 간의 상호작용을 보기 위해
`Graph` 클래스로 돌아가서
그 설계와 구현을 살펴봅시다.
새로운 `Graph` 인스턴스가 생성될 때,
이전 섹션에서 설명한 로직을 사용하여 엣지를 저장하기 위한
딕셔너리 쌍이 이미 구축되어 있습니다:

```python
class Graph:
    """A directed graph of the relationships among build tasks."""

    def __init__(self):
        self._inputs_of = defaultdict(set)
        self._consequences_of = defaultdict(set)
```

속성 이름 `_inputs_of`와 `_consequences_of` 앞의
선행 밑줄은 해당 속성이 비공개임을
나타내는 Python 커뮤니티의 일반적인 관례입니다.
이 관례는 프로그래머들이 공간과 시간을 통해
서로에게 메시지와 경고를 전달하는 방법 중 하나로
커뮤니티가 제안하는 것입니다.
공개 및 내부 객체 속성 간의 차이를
신호할 필요성을 인식하여,
커뮤니티는 미래의 우리 자신을 포함한
다른 프로그래머들에게 해당 속성이
클래스의 보이지 않는 내부 기계장치의 일부로
가장 잘 취급된다는 것을 나타내는
간결하고 상당히 일관된 지표로
단일 선행 밑줄을 채택했습니다.

표준 딕셔너리 대신 `defaultdict`를 사용하는 이유는 무엇일까요?
딕셔너리를 다른 데이터 구조와 결합할 때
일반적인 문제는 누락된 키를 처리하는 것입니다.
일반적인 딕셔너리에서는
존재하지 않는 키를 검색하면 `KeyError`가 발생합니다:

```python
>>> consequences_of = {}
>>> consequences_of['index.rst'].add('index.html')
Traceback (most recent call last):
     ...
KeyError: 'index.rst'
```

일반적인 딕셔너리를 사용하려면 이 특정한 경우를 처리하기 위해
코드 전체에 특별한 검사가 필요하며,
예를 들어 새로운 엣지를 추가할 때가 그렇습니다:

```python
    # Special case to handle “we have not seen this task yet”:

    if input_task not in self._consequences_of:
        self._consequences_of[input_task] = set()

    self._consequences_of[input_task].add(consequence_task)
```

이러한 필요성이 너무 일반적이어서 Python은 특별한 유틸리티인
`defaultdict`를 포함하고 있으며, 이는 부재하는 키들에 대한
값을 반환하는 함수를 제공할 수 있게 해줍니다.
`Graph`가 아직 보지 못한 엣지에 대해 물어볼 때,
예외 대신 빈 `set`을 돌려받게 될 것입니다:

```python
>>> from collections import defaultdict
>>> consequences_of = defaultdict(set)
>>> consequences_of['api.rst']
set()
```

이런 방식으로 구현을 구조화한다는 것은
각 키의 첫 번째 사용이
특정 키가 두 번째 및 후속 사용되는 것과
동일하게 보일 수 있음을 의미합니다:

```python
>>> consequences_of['index.rst'].add('index.html')
>>> 'index.html' in consequences_of['index.rst']
True
```

이러한 기술들을 고려할 때, 앞서 \aosafigref{500l.contingent.graph}에 대한
그래프를 구축하는 데 사용했던
`add_edge`의 구현을 살펴봅시다.

```python
    def add_edge(self, input_task, consequence_task):
        """Add an edge: `consequence_task` uses the output of `input_task`."""
        self._consequences_of[input_task].add(consequence_task)
        self._inputs_of[consequence_task].add(input_task)
```

이 메서드는 각각의 새로운 엣지에 대해
하나가 아니라 두 개의 저장 단계가 필요하다는 사실을 숨겨서
양 방향으로 그것에 대해 알 수 있게 합니다.
그리고 `add_edge()`가 어느 노드가 이전에 본 적이 있는지
알지도 못하고 신경 쓰지도 않는다는 점에 주목하세요.
입력과 결과 데이터 구조가 각각 `defaultdict(set)`이기 때문에,
`add_edge()` 메서드는 노드의 새로움에 대해
행복하게 무지한 상태를 유지합니다 —
`defaultdict`가 즉석에서 새로운 `set` 객체를 생성하여
차이점을 처리해줍니다.
위에서 봤듯이, `defaultdict`를 사용하지 않았다면
`add_edge()`는 세 배나 길어졌을 것입니다.
더 중요한 것은, 결과 코드를 이해하고 추론하기가
더 어려워졌을 것입니다.
이 구현은 파이썬다운 문제 접근법을 보여줍니다:
간단하고, 직접적이며, 간결합니다.

호출자들에게는 우리의 데이터 구조를 순회하는 방법을 배우지 않고도
모든 엣지를 방문할 수 있는 간단한 방법이 제공되어야 합니다:

```python
    def edges(self):
        """Return all edges as ``(input_task, consequence_task)`` tuples."""
        return [(a, b) for a in self.sorted(self._consequences_of)
                       for b in self.sorted(self._consequences_of[a])]
```

`Graph.sorted()` 메서드는
사용자에게 안정적인 출력 순서를 제공할 수 있는
자연스러운 정렬 순서(예: 알파벳순)로
노드들을 정렬하려고 시도합니다.

이 순회 메서드를 사용하여
앞서 세 번의 "add" 메서드 호출을 따라
`g`가 이제 \aosafigref{500l.contingent.graph}에서 봤던
같은 그래프를 표현한다는 것을 알 수 있습니다.

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

`index.html`은 어떨까요?

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
`index.html`이 튜토리얼의 제목이 필요하다는 것을 어떻게 알 수 있을까요?
그리고 이 의존성은 어떻게 해결될까요?

Our intuitive notion of these ideas
served when we were constructing consequences graphs by hand,
but unfortunately computers are not terribly intuitive,
so we will need to be more precise about what we want.

소스로부터 출력을 생성하는 데 필요한 단계는 무엇일까요?
이러한 단계들은 어떻게 정의되고 실행될까요?
그리고 Contingent는 이들 간의 연결을 어떻게 알 수 있을까요?

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
전체 문서 세트를 렌더링하면 어떻게 될까요?
Contingent가 전체 빌드 프로세스를 학습할 수 있을까요?

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

성공했습니다!
출력에서 볼 수 있듯이
우리의 변환이 소스 문서의 지시문을
문서 제목으로 대체했으며,
이는 Contingent가 문서를 빌드하는 데 필요한
다양한 작업들 간의 연결을
발견할 수 있었음을 나타냅니다.

\aosafigure[240pt]{contingent-images/figure4.png}{The complete set of relationships between our input files and our HTML outputs.}{500l.contingent.graph4}

`task` 래퍼 메커니즘을 통해
한 작업이 다른 작업을 호출하는 것을 관찰함으로써,
`Project`는 입력과 결과의 그래프를
자동으로 학습했습니다.
완전한 결과 그래프를 마음대로 사용할 수 있으므로,
Contingent는 어떤 작업의 입력이 변경되면
재빌드해야 할 모든 것들을 알고 있습니다.

## 결과 추적

초기 빌드가 완료되면,
Contingent는 입력 파일의 변경사항을 모니터링해야 합니다.
사용자가 새로운 편집을 마치고 "저장"을 실행하면,
`read()` 메서드와 그 결과들이 모두 호출되어야 합니다.

이는 그래프가 생성된 순서와는
반대 순서로 그래프를 탐색해야 합니다.
기억하시겠지만, 이 그래프는 API 레퍼런스에 대해
`render()`를 호출하고 그것이 `parse()`를 호출한 후
마지막에 `read()` 작업을 호출함으로써 구축되었습니다.
이제 우리는 반대 방향으로 진행합니다:
`read()`가 이제 새로운 콘텐츠를 반환할 것임을 알고 있으므로,
다운스트림에 어떤 결과들이 놓여 있는지 파악해야 합니다.

결과를 컴파일하는 프로세스는 재귀적입니다.
각 결과 자체가 그것에 의존하는 추가 작업들을 가질 수 있기 때문입니다.
그래프에 대한 반복 호출을 통해
이 재귀를 수동으로 수행할 수 있습니다.
(여기서 우리는 Python 프롬프트가 마지막에 표시된 값을
후속 표현식에서 사용하기 위해
`_`라는 이름으로 저장한다는 사실을 활용하고 있습니다.)

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

즉시 결과를 반복적으로 찾고
더 이상 결과가 없는 작업에 도달했을 때만 중단하는
이 재귀적 작업은 `Graph` 클래스의 메서드에 의해
직접 지원될 만큼 기본적인 그래프 연산입니다:

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

변경을 감지했을 때
재귀적 결과의 모든 작업을 주의깊게 다시 실행하면,
Contingent는 너무 적게 재빌드하는 것을 피할 수 있을 것입니다.
하지만 두 번째 과제는
너무 많이 재빌드하는 것을 피하는 것이었습니다.
\aosafigref{500l.contingent.graph4}를 다시 참조해보세요.
`tutorial.txt`가 변경될 때마다 세 개의 문서를
모두 재빌드하는 것을 피하고 싶습니다.
대부분의 편집은 아마도 제목에는 영향을 주지 않고
본문에만 영향을 줄 것이기 때문입니다.
어떻게 이를 달성할 수 있을까요?

해결책은 그래프 재계산을 캐싱에 의존하게 만드는 것입니다.
변경의 재귀적 결과를 통해 앞으로 진행할 때,
마지막보다 입력이 다른 작업들만 호출할 것입니다.

이 최적화에는 마지막 데이터 구조가 포함될 것입니다.
`Project`에 `_todo` 집합을 제공하여
적어도 하나의 입력 값이 변경되어
재실행이 필요한 모든 작업을
기억하도록 할 것입니다.
`_todo`에 있는 작업만이 구식이므로,
빌드 프로세스는 그곳에 나타나지 않는 한
모든 작업의 실행을 건너뛸 수 있습니다.

다시 말하지만, Python의 편리하고 통합된 설계는
이런 기능들을 매우 쉽게 코딩할 수 있게 합니다.
작업 객체들이 해시 가능하므로,
`_todo`는 단순히 작업 항목들을
식별자로 기억하는 집합이 될 수 있으며 —
작업이 두 번 나타나지 않는 것을 보장합니다 —
그리고 이전 실행의 반환 값들의 `_cache`는
작업을 키로 하는 딕셔너리가 될 수 있습니다.

더 정확히 말하면, 재빌드 단계는
`_todo`가 비어있지 않은 한 계속 반복해야 합니다.
각 루프 동안 다음을 수행해야 합니다:

* `recursive_consequences_of()`를 호출하고
  `_todo`에 나열된 모든 작업을 전달합니다.
  반환 값은 `_todo` 작업들 자체뿐만 아니라
  그들의 모든 다운스트림 작업들 —
  즉, 이번에 출력이 다르게 나온다면
  재실행이 필요할 수도 있는 모든 작업들의 목록이 됩니다.

* 목록의 각 작업에 대해
  그것이 `_todo`에 나열되어 있는지 확인합니다.
  그렇지 않다면 실행을 건너뛸 수 있습니다.
  왜냐하면 그것의 업스트림에서 우리가 다시 호출한 작업들 중
  어느 것도 그 작업의 재계산을 요구하는
  새로운 반환 값을 생성하지 않았기 때문입니다.

* 하지만 우리가 도달했을 때
  실제로 `_todo`에 나열된 작업에 대해서는
  재실행하고 반환 값을 다시 계산하도록 요청해야 합니다.
  작업 래퍼 함수가 이 반환 값이
  이전 캐시된 값과 일치하지 않음을 감지하면,
  재귀적 결과 목록에서 도달하기 전에
  그것의 다운스트림 작업들이 자동으로 `_todo`에 추가될 것입니다.

목록의 끝에 도달할 때까지
재실행이 필요할 수도 있는 모든 작업은
실제로 재실행되어야 합니다.
하지만 혹시 모르니 `_todo`를 확인하고
아직 비어있지 않다면 다시 시도할 것입니다.
매우 빠르게 변화하는 의존성 트리의 경우에도
이것은 빠르게 안정화되어야 합니다.
순환만이 —
예를 들어, 작업 *A*가 작업 *B*의 출력을 필요로 하는데
그 작업 *B* 자체가 작업 *A*의 출력을 필요로 하는 경우 —
빌더를 무한 루프에 빠뜨릴 수 있으며,
그리고 그들의 반환 값이 안정화되지 않는 경우에만 그렇습니다.
다행히도, 실제 세계의 빌드 작업은 일반적으로 순환이 없습니다.

예제를 통해 이 시스템의 동작을 추적해 봅시다.

`tutorial.txt`를 편집하여
제목과 본문 콘텐츠를 모두 변경한다고 가정해 봅시다.
우리의 `filesystem` 딕셔너리에서 값을 수정하여
이를 시뮬레이션할 수 있습니다:

```python
>>> filesystem['tutorial.txt'] = """
... The Coder Tutorial
... ------------------
... This is a new and improved
... introductory paragraph.
... """
```

이제 콘텐츠가 변경되었으므로,
주어진 작업과 인수에 대해 이전 캐시된 결과를
반환하려는 의지를 일시적으로 비활성화하는
`cache_off()` 컨텍스트 매니저를 사용하여
Project에게 `read()` 작업을 재실행하도록 요청할 수 있습니다:

```python
>>> with project.cache_off():
...     text = read('tutorial.txt')
```

새로운 튜토리얼 텍스트가 이제 캐시에 읽혔습니다.
얼마나 많은 다운스트림 작업이 재실행되어야 할까요?

이 질문에 답하는 데 도움을 주기 위해,
`Project` 클래스는 재빌드 과정에서
어떤 작업들이 실행되는지 알려주는
간단한 추적 기능을 지원합니다.
위의 `tutorial.txt` 변경이
본문과 제목 모두에 영향을 주므로,
다운스트림의 모든 것들이 재계산되어야 할 것입니다:

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

\aosafigref{500l.contingent.graph4}를 다시 살펴보면,
예상대로 이것이 `read('tutorial.txt')`의
즉시 또는 다운스트림 결과인
모든 작업임을 알 수 있습니다.

하지만 다시 편집하되
이번에는 제목을 그대로 둔다면 어떨까요?

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

이 작고 제한적인 변경은
다른 문서들에 아무런 영향을 주지 않아야 합니다.

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

Contingent가 문제 도메인의 모든 개념에
장황한 이름이 부여된 작은 클래스들의
숨막히는 숲이 될 수 있는 언어와
프로그래밍 방법론들이 존재합니다.

하지만 Python으로 Contingent를 프로그래밍할 때는
`TaskArgument`, `CachedResult`, `ConsequenceList`와 같은
수십 개의 가능한 클래스들의 생성을 건너뛰었습니다.
대신 범용 데이터 구조로 일반적인 문제를 해결하는
Python의 강력한 전통을 활용하여,
핵심 데이터 구조인 튜플, 리스트, 집합, 딕셔너리에서
작은 아이디어 집합을 반복적으로 사용하는
코드를 만들어냈습니다.

하지만 이것이 문제를 일으키지는 않을까요?

범용 데이터 구조는 그 본질상 익명입니다.
우리의 `project._cache`는 집합입니다.
`Graph` 내부의 모든 업스트림 및 다운스트림 노드 컬렉션도
마찬가지입니다.
범용 `set` 오류 메시지를 보고
오류에 대해 프로젝트 또는 그래프
구현 중 어디를 찾아야 할지 모르는
위험에 처해 있을까요?

사실, 우리는 위험에 처해 있지 않습니다!

캡슐화의 세심한 원칙 —
그래프의 집합을 `Graph` 코드만 건드리도록 하고,
프로젝트의 집합을 `Project` 코드만 건드리도록 하는 —
덕분에 프로젝트의 나중 단계에서 집합 연산이
오류를 반환하더라도 모호함은 결코 없을 것입니다.
오류 발생 순간의 가장 안쪽 실행 메서드의 이름이
반드시 실수와 관련된 정확한 클래스와 집합으로
우리를 안내할 것입니다.
데이터 구조 속성 앞에 관례적인 밑줄을 붙이고
클래스 외부의 코드에서 그것들을 건드리지 않도록
주의하는 한, 데이터 타입의 모든 가능한 응용에 대해
`set`의 서브클래스를 만들 필요는 없습니다.

Contingent는 시대를 획기한 *Design Patterns* 책에서 나온
파사드 패턴이 잘 설계된 Python 프로그램에
얼마나 중요한지를 보여줍니다.
Python 프로그램의 모든 데이터 구조와 데이터 조각이
자신만의 클래스를 갖게 되는 것은 아닙니다.
대신, 클래스는 코드에서 개념적 중심점에서
아껴서 사용됩니다. 여기서 의존성 그래프라는
아이디어와 같은 큰 아이디어가
그 밑에 놓인 단순한 범용 데이터 구조들의
세부사항을 숨기는 파사드로
포장될 수 있습니다.

파사드 외부의 코드는
필요한 큰 개념들과
수행하고자 하는 연산들을 명명합니다.
파사드 내부에서는
프로그래머가 Python 프로그래밍 언어의
작고 편리한 구성 요소들을 조작하여
연산이 일어나도록 만듭니다.
