title: DBDB: Dog Bed Database
author: Taavi Burns
<markdown>
_[Countermeasure](http://www.countermeasuremusic.com)의 최신 베이스(때로는 테너) 연주자인 Taavi는 틀을 깨뜨리려고 노력한다... 때로는 그 존재를 무시하기도 한다. 이는 그의 커리어에서 다양한 직장을 거쳐온 것에서도 확실히 드러난다: IBM(C와 Perl), FreshBooks(모든 것), Points.com(Python), 그리고 현재 PagerDuty(Scala)에서 일하고 있다. 그 외에도 Brompton 접이식 자전거를 타고 돌아다니지 않을 때는 아들과 함께 마인크래프트를 플레이하거나 아내와 함께 파쿠르(또는 암벽등반이나 다른 모험)에 참여하는 모습을 볼 수 있다. 그는 대륙식 뜨개질을 한다._
</markdown>
## 소개

DBDB(Dog Bed Database)는 간단한 키/값 데이터베이스를 구현한 파이썬 라이브러리다.
키와 값을 연결하고,
그 연결 관계를 나중에 검색할 수 있도록 디스크에 저장한다.

DBDB는 컴퓨터 충돌과
오류 상황에서도 데이터를 보존하는 것을 목표로 한다.
또한 모든 데이터를 한 번에 RAM에 보관하지 않아
RAM보다 많은 데이터를 저장할 수 있다.


## 메모리

내가 정말로 버그에 막혔던 첫 번째 순간을 기억한다. BASIC 프로그램을
다 작성하고 실행했을 때, 화면에 이상한 반짝이는 픽셀들이 나타나고
프로그램이 일찍 중단되었다. 코드를 다시 확인해보니,
프로그램의 마지막 몇 줄이 사라져 있었다.

어머니의 친구 중 한 명이 프로그래밍을 할 줄 알아서 전화를 걸었다.
몇 분 동안 대화한 후, 문제를 발견했다: 프로그램이 너무 커서
비디오 메모리를 침범한 것이었다. 화면을 지우는 것이 프로그램을 잘라냈고,
반짝임은 Applesoft BASIC이 프로그램 상태를 프로그램 끝 바로 너머의
RAM에 저장하는 방식의 부작용이었다.

그 순간부터, 나는 메모리 할당에 관심을 갖게 되었다.
포인터에 대해 배우고 malloc으로 메모리를 할당하는 방법을 배웠다.
내 데이터 구조가 메모리에 어떻게 배치되는지 배웠다. 그리고
데이터 구조를 변경할 때 매우, 매우 조심하는 법을 배웠다.

몇 년 후, Erlang이라는 프로세스 지향 언어에 대해 읽으면서,
프로세스 간에 메시지를 보낼 때 실제로 데이터를 복사할 필요가 없다는 것을 알게 되었는데,
모든 것이 불변이었기 때문이다. 그 후 Clojure에서 불변 데이터 구조를 발견했고,
정말로 이해하기 시작했다.

2013년에 CouchDB에 대해 읽었을 때, 나는 그냥 미소를 지으며 고개를 끄덕였다.
복잡한 데이터가 변경될 때 이를 관리하는 구조와 메커니즘을
인식했기 때문이다.

나는 불변 데이터를 중심으로 구축된 시스템을 설계할 수 있다는 것을 배웠다.

그러고 나서 책 챕터를 쓰기로 동의했다.

CouchDB의 핵심 데이터 저장 개념들을
(내가 이해한 바로는)
설명하는 것이 재미있을 것이라고 생각했다.

트리를 제자리에서 변경하는 이진 트리
알고리즘을 작성하려고 시도하면서, 상황이 얼마나 복잡해지는지에 좌절했다.
엣지 케이스의 수와 트리의 한 부분 변경이 다른 부분에 미치는 영향을
추론하려는 것이 머리를 아프게 했다.
이 모든 것을 어떻게 설명해야 할지 전혀 감이 오지 않았다.

배운 교훈을 기억하며, 불변 이진 트리를
갱신하는 재귀 알고리즘을 살짝 들여다보니
상대적으로 간단한 것으로 밝혀졌다.

다시 한 번, 변하지 않는 것들에 대해 추론하는 것이 더 쉽다는 것을 배웠다.

이것이 이야기의 시작이다.


## 왜 흥미로운가?

대부분의 프로젝트는 어떤 종류의 데이터베이스를 필요로 한다.
정말로 직접 만들면 안 된다;
JSON을 디스크에 쓰는 것만으로도 당신을 괴롭힐
많은 엣지 케이스가 있다:

* 파일 시스템 공간이 부족하면 어떻게 될까?
* 저장하는 동안 노트북 배터리가 떨어지면 어떻게 될까?
* 데이터 크기가 사용 가능한 메모리를 초과하면 어떻게 될까?
  (현대 데스크톱 컴퓨터의 대부분 애플리케이션에서는 가능성이 낮다&hellip; 하지만
  모바일 디바이스나 서버 사이드 웹 애플리케이션에서는 가능성이 높다.)

그러나 데이터베이스가 이 모든 문제를 어떻게 처리하는지 _이해하고_ 싶다면,
직접 만들어보는 것이 좋은 아이디어가 될 수 있다.

여기서 논의하는 기법과 개념들은
실패에 직면했을 때 합리적이고 예측 가능한 동작이 필요한
모든 문제에 적용할 수 있을 것이다.

실패에 대해 이야기해보자...


## 실패의 특성화

데이터베이스는 종종 ACID 속성을
얼마나 밀접하게 준수하는지로 특성화된다:
원자성, 일관성, 격리성, 그리고 내구성.

DBDB의 갱신은 원자적이고 내구적이며,
이 두 속성은 챕터 뒷부분에서 설명된다.
DBDB는 일관성 보장을 제공하지 않으며
저장되는 데이터에 대한 제약이 없기 때문이다.
격리성 또한 마찬가지로 구현되지 않았다.

<latex>
물론 애플리케이션 코드는 자체적인 일관성 보장을 부과할 수 있지만,
적절한 격리성은 트랜잭션 매니저를 필요로 한다. 여기서는 시도하지 않을 것이다;
하지만 CircleDB 챕터(\aosachapref{s:functionalDB})에서 트랜잭션 관리에 대해
더 배울 수 있다.
</latex>

<markdown>
물론 애플리케이션 코드는 자체적인 일관성 보장을 부과할 수 있지만,
적절한 격리성은 트랜잭션 매니저를 필요로 한다. 여기서는 시도하지 않을 것이다;
하지만 [CircleDB 챕터](http://aosabook.org/en/500L/an-archaeology-inspired-database.html)에서 트랜잭션 관리에 대해 더 배울 수 있다.
</markdown>

또한 생각해야 할 다른 시스템 유지보수 문제들이 있다.
이 구현에서는 오래된 데이터가 회수되지 않아서,
반복적인 갱신이
(같은 키에 대해서라도)
결국 모든 디스크 공간을 소비하게 될 것이다. (곧 이것이 왜 그런지 발견하게 될 것이다.)
[PostgreSQL](http://www.postgresql.org/)은 이 회수를 "vacuuming"이라고 부르며
(이것은 오래된 행 공간을 재사용 가능하게 만든다),
[CouchDB](http://couchdb.apache.org/)는 이것을 "compaction"이라고 부른다
(데이터의 "살아있는" 부분을 새 파일로 다시 쓰고,
원자적으로 이전 파일 위로 이동시킨다).

DBDB는 컴팩션 기능을 추가하도록 향상될 수 있지만,
독자를 위한 연습 문제로 남겨둔다[^bonus].

[^bonus]: 보너스 기능: 컴팩션된 트리 구조가 균형을 이루도록 보장할 수 있는가?
이는 시간이 지남에 따라 성능을 유지하는 데 도움이 된다.


## DBDB의 아키텍처

DBDB는 "이것을 디스크 어딘가에 넣어라"의 관심사를
(데이터가 파일에서 어떻게 배치되는지; 물리 레이어)
데이터의 논리적 구조와
(이 예제에서는 이진 트리; 논리 레이어)
키/값 저장소의 내용으로부터
(키 `a`와 값 `foo`의 연관; 공개 API)
분리한다.

많은 데이터베이스가 논리적 측면과 물리적 측면을 분리하는데
각각의 대체 구현을 제공하여 다른 성능 특성을 얻는 것이 종종 유용하기 때문이다.
예를 들어, DB2의 SMS(파일 시스템의 파일) 대 DMS(raw 블록 디바이스) 테이블스페이스,
또는 MySQL의 [대체 엔진 구현들](http://dev.mysql.com/doc/refman/5.7/en/storage-engines.html).

## 설계 발견하기

이 책의 대부분 챕터들은 프로그램이 시작부터 완성까지
어떻게 만들어졌는지를 설명한다. 그러나 이것은 우리 대부분이
작업하고 있는 코드와 상호작용하는 방식이 아니다.
우리는 대부분 다른 사람들이 작성한 코드를 발견하고,
그것을 수정하거나 확장하여 다른 일을 하도록 하는 방법을 알아낸다.

이 챕터에서는 DBDB가 완성된 프로젝트라고 가정하고,
그것이 어떻게 작동하는지 배우기 위해 살펴볼 것이다. 먼저 전체
프로젝트의 구조를 탐색해보자.

### 조직 단위

단위들은 여기서 최종 사용자로부터의 거리에 따라 정렬된다; 즉, 첫 번째 모듈은
이 프로그램의 사용자가 가장 많이 알아야 할 가능성이 있는 것이고,
마지막은 그들이 거의 상호작용하지 않아야 하는 것이다.

* ``tool.py``는
    터미널 창에서
    데이터베이스를 탐색하기 위한
    명령행 도구를 정의한다.

* ``interface.py``는
    구체적인 ``BinaryTree`` 구현을 사용하여
    파이썬 딕셔너리 API를 구현하는
    클래스(``DBDB``)를 정의한다.
    이것은 파이썬 프로그램 내에서 DBDB를 사용하는 방법이다.

* ``logical.py``는
    논리 레이어를 정의한다.
    이것은 키/값 저장소에 대한 추상 인터페이스다.

    - ``LogicalBase``는 논리적 갱신을 위한 API를
        (get, set, commit 같은)
        제공하고 갱신 자체를 구현하기 위해
        구체적인 서브클래스에 위임한다.
        또한 저장소 잠금과
        내부 노드 역참조를 관리한다.

    - ``ValueRef``는 데이터베이스에 저장된
        이진 블롭을 참조하는
        파이썬 객체다.
        이 간접 참조는 전체 데이터 저장소를
        한 번에 메모리로 로딩하는 것을 피할 수 있게 해준다.

* ``binary_tree.py``는
    논리 인터페이스 아래에서
    구체적인 이진 트리 알고리즘을 정의한다.

    - ``BinaryTree``는 키/값 쌍을 가져오고, 삽입하고, 삭제하는
        메서드를 가진
        이진 트리의 구체적인 구현을 제공한다.
        ``BinaryTree``는 불변 트리를 나타낸다;
        갱신은 기존 트리와 공통 구조를 공유하는
        새로운 트리를 반환함으로써 수행된다.

    - ``BinaryNode``는 이진 트리의 노드를 구현한다.

    - ``BinaryNodeRef``는 ``BinaryNode``를
        직렬화하고 역직렬화하는 방법을 아는
        특화된 ``ValueRef``다.

* ``physical.py``는
    물리 레이어를 정의한다.
    ``Storage`` 클래스는
    지속적인, (대부분) 추가 전용 레코드 저장소를 제공한다.

이러한 모듈들은 각 클래스에 단일 책임을 부여하려는
시도에서 자랐다.
다시 말해서,
각 클래스는 변경할 이유가 하나만 있어야 한다.


### 값 읽기

가장 간단한 케이스부터 시작하겠다: 데이터베이스에서 값을 읽기. ``example.db``에서 키 ``foo``와 연관된 값을 가져오려고 할 때
무엇이 일어나는지 보자:

```bash
$ python -m dbdb.tool example.db get foo
```

이것은 ``dbdb.tool`` 모듈에서 ``main()`` 함수를 실행한다:
```python
# dbdb/tool.py
def main(argv):
    if not (4 <= len(argv) <= 5):
        usage()
        return BAD_ARGS
    dbname, verb, key, value = (argv[1:] + [None])[:4]
    if verb not in {'get', 'set', 'delete'}:
        usage()
        return BAD_VERB
    db = dbdb.connect(dbname)          # CONNECT
    try:
        if verb == 'get':
            sys.stdout.write(db[key])  # GET VALUE
        elif verb == 'set':
            db[key] = value
            db.commit()
        else:
            del db[key]
            db.commit()
    except KeyError:
        print("Key not found", file=sys.stderr)
        return BAD_KEY
    return OK
```

``connect()`` 함수는
데이터베이스 파일을 열고
(생성할 수도 있지만,
덮어쓰지는 않는다)
``DBDB`` 인스턴스를 반환한다:
```python
# dbdb/__init__.py
def connect(dbname):
    try:
        f = open(dbname, 'r+b')
    except IOError:
        fd = os.open(dbname, os.O_RDWR | os.O_CREAT)
        f = os.fdopen(fd, 'r+b')
    return DBDB(f)
```

```python
# dbdb/interface.py
class DBDB(object):

    def __init__(self, f):
        self._storage = Storage(f)
        self._tree = BinaryTree(self._storage)
```

`DBDB`가 `Storage` 인스턴스에 대한 참조를 갖고 있지만,
`self._tree`와도 그 참조를 공유한다는 것을 바로 볼 수 있다. 왜 그럴까? `self._tree`가
스스로 저장소에 대한 접근을 관리할 수 없을까?

어떤 객체가 리소스를 "소유"하는가의 질문은 설계에서 종종 중요한데,
어떤 변경이 안전하지 않을 수 있는지에 대한 힌트를 주기 때문이다.
계속 진행하면서 그 질문을 염두에 두자.

DBDB 인스턴스를 얻으면, ``key``에서 값을 가져오는 것은
딕셔너리 조회(``db[key]``)를 통해 이루어지며, 이것은 파이썬 인터프리터가
``DBDB.__getitem__()``을 호출하게 한다.
```python
# dbdb/interface.py
class DBDB(object):
# ...
    def __getitem__(self, key):
        self._assert_not_closed()
        return self._tree.get(key)

    def _assert_not_closed(self):
        if self._storage.closed:
            raise ValueError('Database closed.')
```

``__getitem__()``는 `_assert_not_closed`를 호출하여
데이터베이스가 여전히 열려 있는지 확인한다. 아하! 여기서 `DBDB`가
우리의 `Storage` 인스턴스에 직접 접근해야 하는 이유를 적어도 하나는 본다: 전제 조건을 강제하기 위해서다.
(이 설계에 동의하는가? 우리가 이것을 할 수 있는 다른 방법을 생각해볼 수 있는가?)

그러면 DBDB는 ``LogicalBase``에서 제공하는
``_tree.get()``을 호출하여 내부 ``_tree``에서 ``key``와 연관된 값을 검색한다:

```python
# dbdb/logical.py
class LogicalBase(object):
# ...
    def get(self, key):
        if not self._storage.locked:
            self._refresh_tree_ref()
        return self._get(self._follow(self._tree_ref), key)
```

``get()``은 저장소가 잠겨 있는지 확인한다. 여기에 왜 잠금이 있을 수 있는지
100% 확실하지는 않지만, 아마도 작성자들이 데이터에 대한 접근을 직렬화할 수 있도록
존재할 것이라고 추측할 수 있다. 저장소가 잠겨 있지 않으면 무엇이 일어날까?

```python
# dbdb/logical.py
class LogicalBase(object):
# ...
def _refresh_tree_ref(self):
        self._tree_ref = self.node_ref_class(
            address=self._storage.get_root_address())
```

`_refresh_tree_ref`는 현재 디스크에 있는 것으로 트리의 데이터 "뷰"를 재설정하여,
완전히 최신인 읽기를 수행할 수 있게 해준다.

읽기를 시도할 때 저장소가 잠겨 _있다면_ 어떻게 될까? 이것은 다른 프로세스가
아마도 지금 우리가 읽고자 하는 데이터를 변경하고 있다는 것을 의미한다; 우리의 읽기는
데이터의 현재 상태와 최신이 아닐 가능성이 높다. 이것은
일반적으로 "더티 읽기"라고 알려져 있다. 이 패턴은 많은 읽기 프로세스들이
차단을 걱정하지 않고 데이터에 접근할 수 있게 해주지만, 약간 구식일 수 있다는 비용이 있다.

지금은 실제로 데이터를 어떻게 검색하는지 살펴보자:
```python
# dbdb/binary_tree.py
class BinaryTree(LogicalBase):
# ...
    def _get(self, node, key):
        while node is not None:
            if key < node.key:
                node = self._follow(node.left_ref)
            elif node.key < key:
                node = self._follow(node.right_ref)
            else:
                return self._follow(node.value_ref)
        raise KeyError
```
이것은 참조를 따라 노드로 가는 표준 이진 트리 검색이다. ``BinaryTree`` 문서를 읽어보면
``Node``와 ``NodeRef``가 값 객체라는 것을 안다:
그것들은 불변이고 내용이 절대 변하지 않는다.
``Node``는 연관된 키와 값,
그리고 왼쪽과 오른쪽 자식과 함께
생성된다.
이러한 연관들도 절대 변하지 않는다.
전체 ``BinaryTree``의 내용은 루트 노드가 교체될 때만
시각적으로 변한다.
이것은 우리가 검색을 수행하는 동안 우리 트리의 내용이
변경되는 것을 걱정할 필요가 없다는 것을 의미한다.

연관된 값이 발견되면,
사용자의 데이터를 정확히 보존하기 위해
추가 개행을 추가하지 않고
``main()``에 의해 ``stdout``에 쓰인다.


#### 삽입과 갱신

이제 ``example.db``에서 키 ``foo``를 값 ``bar``로 설정할 것이다:
```bash
$ python -m dbdb.tool example.db set foo bar
```

다시, 이것은 ``dbdb.tool`` 모듈에서 ``main()`` 함수를 실행한다. 우리가
이 코드를 전에 본 적이 있으므로, 중요한 부분들만 강조할 것이다:
```python
# dbdb/tool.py
def main(argv):
    ...
    db = dbdb.connect(dbname)          # CONNECT
    try:
        ...
        elif verb == 'set':
            db[key] = value            # SET VALUE
            db.commit()                # COMMIT
        ...
    except KeyError:
        ...
```

이번에는 ``db[key] = value``로 값을 설정하는데
이것은 ``DBDB.__setitem__()``을 호출한다.
```python
# dbdb/interface.py
class DBDB(object):
# ...
    def __setitem__(self, key, value):
        self._assert_not_closed()
        return self._tree.set(key, value)
```

``__setitem__``은 데이터베이스가 여전히 열려 있는지 확인하고
그 다음 ``_tree.set()``을 호출하여 내부 ``_tree``에서
``key``에서 ``value``로의 연관을 저장한다.

``_tree.set()``은 ``LogicalBase``에서 제공된다:
```python
# dbdb/logical.py
class LogicalBase(object):
# ...
    def set(self, key, value):
        if self._storage.lock():
            self._refresh_tree_ref()
        self._tree_ref = self._insert(
            self._follow(self._tree_ref), key, self.value_ref_class(value))
```

``set()``은 먼저 저장소 잠금을 확인한다:

```python
# dbdb/storage.py
class Storage(object):
    ...
    def lock(self):
        if not self.locked:
            portalocker.lock(self._f, portalocker.LOCK_EX)
            self.locked = True
            return True
        else:
            return False
```

여기서 주목할 두 가지 중요한 사항이 있다:

 - 우리의 잠금은 [portalocker](https://pypi.python.org/pypi/portalocker)라는
   타사 파일 잠금 라이브러리에서 제공된다.
 - `lock()`은 데이터베이스가 이미 잠겨 있으면 `False`를 반환하고,
   그렇지 않으면 `True`를 반환한다.

`_tree.set()`으로 돌아가면, 왜 애초에 `lock()`의 반환 값을 확인했는지
이제 이해할 수 있다: 가장 최근의 루트 노드 참조를 위해
`_refresh_tree_ref`를 호출할 수 있게 해주어
마지막으로 디스크에서 트리를 새로 고친 이후 다른 프로세스가
만들었을 수 있는 갱신을 잃지 않도록 한다.
그런 다음 루트 트리 노드를
삽입된(또는 갱신된) 키/값을 포함하는 새 트리로 교체한다.

트리를 삽입하거나 갱신하는 것은 노드를 변경하지 않는데,
``_insert()``가 새 트리를 반환하기 때문이다.
새 트리는 메모리와 실행 시간을 절약하기 위해
이전 트리와 변경되지 않은 부분을 공유한다.
이것을 재귀적으로 구현하는 것이 자연스럽다:
```python
# dbdb/binary_tree.py
class BinaryTree(LogicalBase):
# ...
    def _insert(self, node, key, value_ref):
        if node is None:
            new_node = BinaryNode(
                self.node_ref_class(), key, value_ref, self.node_ref_class(), 1)
        elif key < node.key:
            new_node = BinaryNode.from_node(
                node,
                left_ref=self._insert(
                    self._follow(node.left_ref), key, value_ref))
        elif node.key < key:
            new_node = BinaryNode.from_node(
                node,
                right_ref=self._insert(
                    self._follow(node.right_ref), key, value_ref))
        else:
            new_node = BinaryNode.from_node(node, value_ref=value_ref)
        return self.node_ref_class(referent=new_node)
```

어떻게 우리가 항상 새 노드를
(``NodeRef``로 감싼) 반환하는지 주목하라.
새 서브트리를 가리키도록 노드를 갱신하는 대신,
변경되지 않은 서브트리를 공유하는 새 노드를 만든다.
이것이 이 이진 트리를 불변 데이터 구조로 만드는 것이다.

여기서 이상한 것을 알아챘을 수도 있다: 우리는 아직 디스크의
어떤 것도 실제로 변경하지 않았다. 우리가 한 것은 트리 노드들을
이동시켜 디스크상 데이터의 뷰를 조작한 것뿐이다.

실제로 이러한 변경사항을 디스크에 쓰기 위해서는, 이 섹션의 시작에서
`tool.py`의 `set` 연산의 두 번째 부분으로 본 `commit()`에 대한 명시적인 호출이 필요하다.

커밋은 메모리에 있는 모든 더티 상태를 쓰고,
그런 다음 트리의 새 루트 노드의 디스크 주소를 저장하는 것을 포함한다.

API부터 시작하여:
```python
# dbdb/interface.py
class DBDB(object):
# ...
    def commit(self):
        self._assert_not_closed()
        self._tree.commit()
```

``_tree.commit()``의 구현은 ``LogicalBase``에서 나온다:
```python
# dbdb/logical.py
class LogicalBase(object)
# ...
    def commit(self):
        self._tree_ref.store(self._storage)
        self._storage.commit_root_address(self._tree_ref.address)
```

모든 ``NodeRef``는 먼저 자식들에게 ``prepare_to_store()``를 통해
직렬화하도록 요청함으로써 스스로를 디스크에 직렬화하는 방법을 안다:
```python
# dbdb/logical.py
class ValueRef(object):
# ...
    def store(self, storage):
        if self._referent is not None and not self._address:
            self.prepare_to_store(storage)
            self._address = storage.write(self.referent_to_string(self._referent))
```

``LogicalBase``의 ``self._tree_ref``는 실제로 이 경우에
``BinaryNodeRef``(``ValueRef``의 서브클래스)이므로,
``prepare_to_store()``의 구체적인 구현은:
```python
# dbdb/binary_tree.py
class BinaryNodeRef(ValueRef):
    def prepare_to_store(self, storage):
        if self._referent:
            self._referent.store_refs(storage)
```

문제의 ``BinaryNode``, ``_referent``는
자신의 참조들에게 스스로를 저장하도록 요청한다:
```python
# dbdb/binary_tree.py
class BinaryNode(object):
# ...
    def store_refs(self, storage):
        self.value_ref.store(storage)
        self.left_ref.store(storage)
        self.right_ref.store(storage)
```

이것은 쓰이지 않은 변경사항이 있는(즉, ``_address``가 없는) 모든 ``NodeRef``에 대해
끝까지 재귀한다.

이제 우리는 다시 `ValueRef`의 `store` 메서드에서 스택 위로 돌아왔다.
``store()``의 마지막 단계는 이 노드를 직렬화하고
저장소 주소를 저장하는 것이다:
```python
# dbdb/logical.py
class ValueRef(object):
# ...
    def store(self, storage):
        if self._referent is not None and not self._address:
            self.prepare_to_store(storage)
            self._address = storage.write(self.referent_to_string(self._referent))
```

이 시점에서
``NodeRef``의 ``_referent``는 자신의 모든 참조들에 대해 사용 가능한 주소를 갖는 것이 보장되므로,
이 노드를 나타내는 바이트스트링을 생성하여 직렬화한다:
```python
# dbdb/binary_tree.py
class BinaryNodeRef(ValueRef):
# ...
    @staticmethod
    def referent_to_string(referent):
        return pickle.dumps({
            'left': referent.left_ref.address,
            'key': referent.key,
            'value': referent.value_ref.address,
            'right': referent.right_ref.address,
            'length': referent.length,
        })
```

``store()`` 메서드에서 주소를 갱신하는 것은
기술적으로 ``ValueRef``의 변경이다.
사용자에게 보이는 값에 아무런 영향을 주지 않기 때문에,
우리는 이것을 불변으로 여길 수 있다.

루트 ``_tree_ref``에 대한 ``store()``가 완료되면
(``LogicalBase.commit()``에서),
모든 데이터가 디스크에 쓰였다는 것을 안다.
이제 다음을 호출하여 루트 주소를 커밋할 수 있다:
```python
# dbdb/physical.py
class Storage(object):
# ...
    def commit_root_address(self, root_address):
        self.lock()
        self._f.flush()
        self._seek_superblock()
        self._write_integer(root_address)
        self._f.flush()
        self.unlock()
```

파일 핸들이 플러시되었는지 확인하고
(OS가 모든 데이터를 SSD 같은 안정적인 저장소에 저장하기를 원한다는 것을 알도록)
루트 노드의 주소를 쓴다.
디스크 주소를 섹터 경계에 저장하기 때문에 이 마지막 쓰기가 원자적이라는 것을 안다.
이것은 파일의 가장 첫 번째 것이므로,
섹터 크기에 관계없이 참이고,
단일 섹터 디스크 쓰기는 디스크 하드웨어에 의해 원자적임이 보장된다.

루트 노드 주소가 (새 것의 일부와 옛 것의 일부가 아니라)
옛 값 또는 새 값을 갖기 때문에,
다른 프로세스들은 잠금을 얻지 않고도 데이터베이스에서 읽을 수 있다.
외부 프로세스는 옛 트리 또는 새 트리를 볼 수 있지만,
두 개의 혼합은 절대 볼 수 없다.
이러한 방식으로, 커밋은 원자적이다.

루트 노드 주소를 쓰기 전에
새 데이터를 디스크에 쓰고 ``fsync`` 시스템 콜[^fsync]을 호출하기 때문에,
커밋되지 않은 데이터는 도달할 수 없다.
반대로, 루트 노드 주소가 갱신되면,
그것이 참조하는 모든 데이터도 디스크에 있다는 것을 안다.
이러한 방식으로, 커밋은 또한 내구적이다.

[^fsync]: 파일 디스크립터에서 ``fsync``를 호출하는 것은
   운영체제와 하드 드라이브(또는 SSD)에게
   버퍼된 모든 데이터를 즉시 쓰라고 요청한다.
   운영체제와 드라이브는 성능을 향상시키기 위해
   보통 모든 것을 즉시 쓰지 않는다.

완료되었다!


### NodeRef가 메모리를 절약하는 방법

전체 트리 구조를 동시에 메모리에 유지하는 것을 피하기 위해,
논리적 노드가 디스크에서 읽힐 때
왼쪽과 오른쪽 자식의 디스크 주소가
(값과 함께)
메모리로 로드된다.
자식들과 그들의 값에 접근하는 것은
데이터를 역참조("실제로 얻기")하기 위해 ``NodeRef.get()``에 대한
하나의 추가 함수 호출이 필요하다.

``NodeRef``를 구성하는 데 필요한 모든 것은 주소다:

    +---------+
    | NodeRef |
    | ------- |
    | addr=3  |
    | get()   |
    +---------+

그것에 ``get()``을 호출하면 구체적인 노드를
그 노드의 참조들과 함께 ``NodeRef``들로 반환할 것이다:

    +---------+     +---------+     +---------+
    | NodeRef |     | Node    |     | NodeRef |
    | ------- |     | ------- | +-> | ------- |
    | addr=3  |     | key=A   | |   | addr=1  |
    | get() ------> | value=B | |   +---------+
    +---------+     | left  ----+
                    | right ----+   +---------+
                    +---------+ |   | NodeRef |
                                +-> | ------- |
                                    | addr=2  |
                                    +---------+

트리에 대한 변경이 커밋되지 않을 때,
그것들은 루트에서 변경된 잎으로의 참조와 함께
메모리에 존재한다.
변경사항들은 아직 디스크에 저장되지 않았으므로,
변경된 노드들은 구체적인 키와 값을 포함하고
디스크 주소는 없다.
쓰기를 하는 프로세스는 커밋되지 않은 변경사항을 볼 수 있고
커밋을 발행하기 전에 더 많은 변경을 할 수 있는데,
``NodeRef.get()``이 가지고 있으면 커밋되지 않은 값을 반환할 것이기 때문이다;
API를 통해 접근할 때 커밋된 데이터와 커밋되지 않은 데이터 사이에는
차이가 없다.
새 루트 노드 주소가 디스크에 쓰일 때까지 변경사항이 보이지 않기 때문에
모든 갱신은 다른 읽기 프로세스들에게 원자적으로 나타날 것이다.
동시 갱신은 디스크의 잠금 파일에 의해 차단된다.
잠금은 첫 번째 갱신에서 획득되고, 커밋 후에 해제된다.


### 독자를 위한 연습 문제

DBDB는 많은 프로세스가 차단 없이 동시에 같은 데이터베이스를 읽을 수 있게 해준다;
트레이드오프는 읽기 프로세스가 때때로 오래된 데이터를 검색할 수 있다는 것이다. 일부 데이터를 일관되게 읽을 수 있어야 한다면 어떨까? 일반적인 사용 사례는
값을 읽고 그 값에 기반하여 갱신하는 것이다. `DBDB`에서 이를 수행하는 메서드를
어떻게 작성하겠는가? 이 기능을 제공하기 위해 어떤 트레이드오프를 감수해야 할까?

데이터 저장소를 갱신하는 데 사용되는 알고리즘은
``interface.py``의 ``BinaryTree`` 문자열을 바꿈으로써
완전히 교체될 수 있다.
데이터 저장소는 성능을 향상시키기 위해
B-트리, B+ 트리, 그리고 다른 것들과 같은
더 복잡한 유형의 검색 트리를 사용하는 경향이 있다.
균형 이진 트리가
(이것은 균형이 잡혀있지 않지만)
값을 찾기 위해 $O(log_2(n))$ 무작위 노드 읽기를 수행해야 하는 반면,
B+ 트리는 각 노드가 단지 2개가 아닌 32가지 방법으로 분할되기 때문에
예를 들어 $O(log_{32}(n))$와 같이 훨씬 적게 필요하다.
40억 개의 항목을 살펴보는 것이
$log_2(2^{32}) = 32$에서 $log_{32}(2^{32}) \approx 6.4$ 조회로 바뀌므로
이것은 실제로 큰 차이를 만든다.
각 조회는 무작위 접근이며,
이것은 회전하는 플래터가 있는 하드 디스크에 대해 엄청나게 비싸다.
SSD는 지연 시간에 도움이 되지만, I/O의 절약은 여전히 유효하다.

기본적으로, 값들은 바이트를 값으로 기대하는
(`Storage`에 직접 전달되는) `ValueRef`에 의해 저장된다.
이진 트리 노드들 자체는
단지 `ValueRef`의 서브클래스다.
<a href="http://json.org">json</a> 또는 <a href="http://msgpack.org">msgpack</a>을 통해 더 풍부한 데이터를 저장하는 것은 자신만의 것을 작성하고
그것을 `value_ref_class`로 설정하는 문제다.
`BinaryNodeRef`는 데이터를 직렬화하기 위해
[pickle](https://docs.python.org/3.4/library/pickle.html)을 사용하는 예다.

데이터베이스 컴팩션은 또 다른 흥미로운 연습 문제다.
컴팩션은 진행하면서 것들을 쓰는 트리의 중위-중앙값 순회를 통해 수행될 수 있다.
모든 데이터 조각을 찾기 위해 순회되는 것이 트리 노드들이므로
트리 노드들이 모두 함께 가는 것이 아마도 최고일 것이다.
가능한 한 많은 중간 노드를 디스크 섹터에
패킹하는 것은 적어도 컴팩션 직후에는
읽기 성능을 향상시킬 것이다.
이것을 직접 구현하려고 시도한다면
여기에는 일부 미묘함이 있다
(예를 들어, 메모리 사용).
그리고 기억하라:
항상 성능 향상을 전후로 벤치마크하라!
종종 결과에 놀랄 것이다.

### 패턴과 원칙

구현이 아닌 인터페이스를 테스트하라.
DBDB를 개발하는 일부로서,
나는 그것을 어떻게 사용할 수 있기를 원하는지 설명하는
여러 테스트를 작성했다.
첫 번째 테스트들은 데이터베이스의 인메모리 버전에 대해 실행되었고,
그런 다음 DBDB를 확장하여 디스크에 지속시켰고,
심지어 나중에는 NodeRef의 개념을 추가했다.
대부분의 테스트들은 변경할 필요가 없었으며,
이것은 일들이 여전히 작동한다는 확신을 주었다.

단일 책임 원칙을 존중하라.
클래스들은 기껏해야 변경할 이유가 하나 있어야 한다.
DBDB의 경우 엄격히는 그렇지 않지만,
오직 국소적인 변경만 필요한
여러 확장 방법이 있다.
기능을 추가하면서 리팩터링하는 것이 즐거웠다!


### 요약

DBDB는 간단한 보장을 하는 간단한 데이터베이스이지만,
일들은 여전히 빠르게 복잡해졌다. 이 복잡성을 관리하기 위해 내가 한 가장 중요한 일은
불변 데이터 구조로 표면상 변경 가능한 객체를 구현하는 것이었다. 추적할 수 있는 것보다 더 많은 엣지 케이스가 있는 것 같은
까다로운 문제 한가운데서 자신을 발견하는 다음 번에 이 기법을 고려하기를 권한다. 
