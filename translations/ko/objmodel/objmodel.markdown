title: 간단한 객체 모델
author: Carl Friedrich Bolz
<markdown>
_Carl Friedrich Bolz는 King's College London의 연구원으로 동적 언어의 구현과 최적화 전반에 관심을 가지고 있습니다. 그는 PyPy/RPython의 핵심 개발자 중 한 명이며, Prolog, Racket, Smalltalk, PHP, Ruby의 구현 작업에 참여했습니다. 트위터 계정은 [\@cfbolz](https://twitter.com/cfbolz)입니다._
</markdown>
## 소개

객체지향 프로그래밍은 오늘날 사용되는 주요 프로그래밍 패러다임 중 하나로, 많은 언어들이 어떤 형태로든 객체지향 기능을 제공합니다. 표면적으로는 서로 다른 객체지향 프로그래밍 언어들이 프로그래머에게 제공하는 메커니즘이 매우 유사해 보이지만, 세부사항은 크게 다를 수 있습니다. 대부분의 언어에서 공통적으로 나타나는 특징은 객체의 존재와 어떤 형태의 상속 메커니즘입니다. 그러나 클래스는 모든 언어가 직접적으로 지원하는 기능은 아닙니다. 예를 들어 Self나 JavaScript 같은 프로토타입 기반 언어에서는 클래스의 개념이 존재하지 않고, 대신 객체들이 서로 직접 상속합니다.

서로 다른 객체 모델 간의 차이점을 이해하는 것은 흥미로울 수 있습니다. 이러한 차이점들은 종종 서로 다른 언어들 간의 가족적 유사성을 드러냅니다.
새로운 언어의 모델을 다른 언어들의 모델과 비교하여 맥락에서 이해하는 것은
새로운 모델을 빠르게 이해하고 프로그래밍 언어 설계 공간에 대한
더 나은 감각을 얻는 데 유용할 수 있습니다.

이 장에서는 일련의 매우 간단한 객체 모델의 구현을 탐구합니다. 간단한 인스턴스와 클래스, 그리고 인스턴스에서 메소드를 호출하는 기능으로 시작합니다. 이것은 Simula 67과 Smalltalk 같은 초기 객체지향 언어에서 확립된 "고전적인" 객체지향 접근법입니다. 이 모델은 단계별로 확장되며, 다음 두 단계에서는 서로 다른 언어 설계 선택지를 탐구하고, 마지막 단계에서는 객체 모델의 효율성을 개선합니다. 최종 모델은 실제 언어의 모델이 아니라 Python 객체 모델의 이상화되고 단순화된 버전입니다.

이 장에서 제시되는 객체 모델은 Python으로 구현됩니다. 코드는 Python 2.7과 3.4 모두에서 작동합니다. 동작과 설계 선택을 더 잘 이해하기 위해, 이 장에서는 객체 모델에 대한 테스트도 제시할 것입니다. 테스트는 py.test나 nose로 실행할 수 있습니다.

Python을 구현 언어로 선택한 것은 다소 비현실적입니다. "실제" VM은 일반적으로 C/C++ 같은 저수준 언어로 구현되며, 효율성을 위해 엔지니어링 세부사항에 많은 주의를 기울여야 합니다. 그러나 더 간단한 구현 언어를 사용하면 구현 세부사항에 얽매이지 않고 실제 동작 차이에 집중하기가 더 쉬워집니다.



## 메소드 기반 모델

우리가 시작할 객체 모델은 Smalltalk의 극도로 단순화된 버전입니다. Smalltalk은 1970년대 Xerox PARC의 Alan Kay 그룹에서 설계한 객체지향 프로그래밍 언어였습니다. 이 언어는 객체지향 프로그래밍을 대중화했으며, 오늘날 프로그래밍 언어에서 발견되는 많은 기능의 원천이 되었습니다. Smalltalk의 언어 설계 핵심 원칙 중 하나는 "모든 것이 객체"라는 것이었습니다. 오늘날 사용되는 Smalltalk의 가장 직접적인 후계자는 Ruby로, 더 C와 비슷한 문법을 사용하지만 Smalltalk의 객체 모델 대부분을 유지하고 있습니다.

이 섹션의 객체 모델은 클래스와 그 인스턴스들, 객체에 속성을 읽고 쓰는 기능, 객체에서 메소드를 호출하는 기능, 그리고 클래스가 다른 클래스의 서브클래스가 될 수 있는 기능을 가질 것입니다. 처음부터 클래스는 스스로 속성과 메소드를 가질 수 있는 완전히 평범한 객체가 될 것입니다.

용어에 관한 주의사항: 이 장에서 저는 "인스턴스"라는 단어를 "클래스가 아닌 객체"를 의미하는 데 사용하겠습니다.

시작하기에 좋은 접근법은 구현될 동작이 어떻게 되어야 하는지를 명시하는 테스트를 작성하는 것입니다. 이 장에서 제시되는 모든 테스트는 두 부분으로 구성될 것입니다. 첫 번째는 몇 개의 클래스를 정의하고 사용하는 일반적인 Python 코드로, Python 객체 모델의 점점 더 고급 기능을 활용합니다. 두 번째는 일반적인 Python 클래스 대신 이 장에서 구현할 객체 모델을 사용하는 해당 테스트입니다.

일반적인 Python 클래스를 사용하는 것과 우리의 객체 모델을 사용하는 것 사이의 매핑은 테스트에서 수동으로 이루어집니다. 예를 들어, Python에서 ``obj.attribute``를 쓰는 대신, 객체 모델에서는 ``obj.read_attr("attribute")`` 메소드를 사용할 것입니다. 이러한 매핑은 실제 언어 구현에서는 언어의 인터프리터나 컴파일러에 의해 수행될 것입니다.

이 장에서 추가적인 단순화는 객체 모델을 구현하는 코드와 객체에서 사용되는 메소드를 작성하는 데 사용되는 코드 사이에 명확한 구분을 두지 않는다는 것입니다. 실제 시스템에서는 이 둘이 종종 서로 다른 프로그래밍 언어로 구현될 것입니다.

객체 필드를 읽고 쓰는 간단한 테스트부터 시작해보겠습니다.

```python
def test_read_write_field():
    # Python code
    class A(object):
        pass
    obj = A()
    obj.a = 1
    assert obj.a == 1

    obj.b = 5
    assert obj.a == 1
    assert obj.b == 5

    obj.a = 2
    assert obj.a == 2
    assert obj.b == 5

    # Object model code
    A = Class(name="A", base_class=OBJECT, fields={}, metaclass=TYPE)
    obj = Instance(A)
    obj.write_attr("a", 1)
    assert obj.read_attr("a") == 1

    obj.write_attr("b", 5)
    assert obj.read_attr("a") == 1
    assert obj.read_attr("b") == 5

    obj.write_attr("a", 2)
    assert obj.read_attr("a") == 2
    assert obj.read_attr("b") == 5
```

이 테스트는 우리가 구현해야 하는 세 가지를 사용합니다.
``Class``와 ``Instance`` 클래스는 각각 우리 객체 모델의 클래스와
인스턴스를 나타냅니다. 클래스의 특별한 인스턴스 두 개가 있습니다: ``OBJECT``와 ``TYPE``입니다. ``OBJECT``는 Python의 ``object``에 해당하며 상속 계층구조의 최상위 기본 클래스입니다. ``TYPE``은 Python의 ``type``에 해당하며 모든 클래스의 타입입니다.

``Class``와 ``Instance``의 인스턴스로 어떤 작업이든 하기 위해, 이들은 여러 메소드를 노출하는 공유 기본 클래스 ``Base``를 상속하여 공유 인터페이스를 구현합니다:

```python
class Base(object):
    """ The base class that all of the object model classes inherit from. """

    def __init__(self, cls, fields):
        """ Every object has a class. """
        self.cls = cls
        self._fields = fields

    def read_attr(self, fieldname):
        """ read field 'fieldname' out of the object """
        return self._read_dict(fieldname)

    def write_attr(self, fieldname, value):
        """ write field 'fieldname' into the object """
        self._write_dict(fieldname, value)

    def isinstance(self, cls):
        """ return True if the object is an instance of class cls """
        return self.cls.issubclass(cls)

    def callmethod(self, methname, *args):
        """ call method 'methname' with arguments 'args' on object """
        meth = self.cls._read_from_class(methname)
        return meth(self, *args)

    def _read_dict(self, fieldname):
        """ read an field 'fieldname' out of the object's dict """
        return self._fields.get(fieldname, MISSING)

    def _write_dict(self, fieldname, value):
        """ write a field 'fieldname' into the object's dict """
        self._fields[fieldname] = value

MISSING = object()

```

``Base`` 클래스는 객체의 클래스를 저장하고 객체의 필드 값을 담는 딕셔너리를 구현합니다.
이제 ``Class``와 ``Instance``를 구현해야 합니다. ``Instance``의 생성자는
인스턴스화할 클래스를 받고 `fields` `dict`를 빈 딕셔너리로 초기화합니다.
그 외에는 ``Instance``는 추가 기능을 더하지 않는 ``Base``를 둘러싼 매우 얇은 서브클래스입니다.

``Class``의 생성자는 클래스의 이름, 기본 클래스, 클래스의 딕셔너리, 그리고 메타클래스를 받습니다.
클래스의 경우, 필드들은 객체 모델의 사용자에 의해 생성자에 전달됩니다. 클래스 생성자는 또한 기본 클래스를 받는데, 지금까지의 테스트에서는 필요하지 않지만 다음 섹션에서 사용할 것입니다.

```python
class Instance(Base):
    """Instance of a user-defined class. """

    def __init__(self, cls):
        assert isinstance(cls, Class)
        Base.__init__(self, cls, {})


class Class(Base):
    """ A User-defined class. """

    def __init__(self, name, base_class, fields, metaclass):
        Base.__init__(self, metaclass, fields)
        self.name = name
        self.base_class = base_class
```

클래스도 객체의 한 종류이므로, (간접적으로) ``Base``를 상속합니다. 따라서 클래스는 다른 클래스의 인스턴스가 되어야 합니다: 그것의 메타클래스입니다.

이제 첫 번째 테스트가 거의 통과합니다. 유일하게 빠진 부분은 ``Class``의 인스턴스인 기본 클래스 ``TYPE``과 ``OBJECT``의 정의입니다. 이들을 위해 우리는 상당히 복잡한 메타클래스 시스템을 가진 Smalltalk 모델에서 크게 벗어날 것입니다. 대신 Python이 채택한 ObjVlisp[^objvlisp]에서 소개된 모델을 사용할 것입니다.

[^objvlisp]: P. Cointe, "Metaclasses are first class: The ObjVlisp Model," SIGPLAN Not, vol. 22, no. 12, pp. 156–162, 1987.

ObjVlisp 모델에서 ``OBJECT``와 ``TYPE``은 서로 얽혀있습니다. ``OBJECT``는 모든 클래스의 기본 클래스로, 기본 클래스가 없습니다. ``TYPE``은 ``OBJECT``의 서브클래스입니다.
기본적으로 모든 클래스는 ``TYPE``의 인스턴스입니다. 특히 ``TYPE``과 ``OBJECT`` 모두 ``TYPE``의 인스턴스입니다. 그러나 프로그래머는 새로운 메타클래스를 만들기 위해 ``TYPE``을 서브클래스할 수도 있습니다:

```python
# set up the base hierarchy as in Python (the ObjVLisp model)
# the ultimate base class is OBJECT
OBJECT = Class(name="object", base_class=None, fields={}, metaclass=None)
# TYPE is a subclass of OBJECT
TYPE = Class(name="type", base_class=OBJECT, fields={}, metaclass=None)
# TYPE is an instance of itself
TYPE.cls = TYPE
# OBJECT is an instance of TYPE
OBJECT.cls = TYPE
```

새로운 메타클래스를 정의하기 위해서는 ``TYPE``을 서브클래스하는 것으로 충분합니다. 그러나 이 장의 나머지 부분에서는 그렇게 하지 않을 것입니다; 모든 클래스의 메타클래스로 단순히 항상 ``TYPE``을 사용할 것입니다.

\aosafigure[240pt]{objmodel-images/inheritance.png}{상속}{500l.objmodel.inheritance}

이제 첫 번째 테스트가 통과합니다. 두 번째 테스트는 클래스에서도 속성 읽기와 쓰기가 작동하는지 확인합니다. 작성하기 쉽고 즉시 통과합니다. \newpage

```python
def test_read_write_field_class():
    # classes are objects too
    # Python code
    class A(object):
        pass
    A.a = 1
    assert A.a == 1
    A.a = 6
    assert A.a == 6

    # Object model code
    A = Class(name="A", base_class=OBJECT, fields={"a": 1}, metaclass=TYPE)
    assert A.read_attr("a") == 1
    A.write_attr("a", 5)
    assert A.read_attr("a") == 5
```

### `isinstance` 검사


지금까지 우리는 객체가 클래스를 가진다는 사실을 활용하지 않았습니다. 다음 테스트는 ``isinstance`` 메커니즘을 구현합니다:

```python
def test_isinstance():
    # Python code
    class A(object):
        pass
    class B(A):
        pass
    b = B()
    assert isinstance(b, B)
    assert isinstance(b, A)
    assert isinstance(b, object)
    assert not isinstance(b, type)

    # Object model code
    A = Class(name="A", base_class=OBJECT, fields={}, metaclass=TYPE)
    B = Class(name="B", base_class=A, fields={}, metaclass=TYPE)
    b = Instance(B)
    assert b.isinstance(B)
    assert b.isinstance(A)
    assert b.isinstance(OBJECT)
    assert not b.isinstance(TYPE)
```

객체 ``obj``가 특정 클래스 ``cls``의 인스턴스인지 확인하려면, ``cls``가 ``obj``의 클래스의 슈퍼클래스인지, 또는 클래스 자체인지 확인하는 것으로 충분합니다.
클래스가 다른 클래스의 슈퍼클래스인지 확인하기 위해서는, 그 클래스의 슈퍼클래스 체인을 따라갑니다. 다른 클래스가 그 체인에서 발견되는 경우에만, 그것은 슈퍼클래스입니다. 클래스 자체를 포함하여 클래스의 슈퍼클래스 체인을
그 클래스의 "메소드 해결 순서(method resolution order)"라고 부릅니다. 이것은 쉽게 재귀적으로 계산할 수 있습니다:


```python
class Class(Base):
    ...

    def method_resolution_order(self):
        """ compute the method resolution order of the class """
        if self.base_class is None:
            return [self]
        else:
            return [self] + self.base_class.method_resolution_order()

    def issubclass(self, cls):
        """ is self a subclass of cls? """
        return cls in self.method_resolution_order()
```

이 코드로 테스트가 통과합니다.


### 메소드 호출

이 첫 번째 버전의 객체 모델에서 남은 누락된 기능은 객체에서 메소드를 호출하는 능력입니다. 이 장에서는 간단한 단일 상속 모델을 구현할 것입니다.

```python
def test_callmethod_simple():
    # Python code
    class A(object):
        def f(self):
            return self.x + 1
    obj = A()
    obj.x = 1
    assert obj.f() == 2

    class B(A):
        pass
    obj = B()
    obj.x = 1
    assert obj.f() == 2 # works on subclass too

    # Object model code
    def f_A(self):
        return self.read_attr("x") + 1
    A = Class(name="A", base_class=OBJECT, fields={"f": f_A}, metaclass=TYPE)
    obj = Instance(A)
    obj.write_attr("x", 1)
    assert obj.callmethod("f") == 2

    B = Class(name="B", base_class=A, fields={}, metaclass=TYPE)
    obj = Instance(B)
    obj.write_attr("x", 2)
    assert obj.callmethod("f") == 3
```

객체로 전송되는 메소드의 올바른 구현을 찾기 위해, 객체 클래스의 메소드 해결 순서를 따라갑니다. 메소드 해결 순서에 있는 클래스 중 하나의 딕셔너리에서 발견된 첫 번째 메소드가 호출됩니다:

```python
class Class(Base):
    ...

    def _read_from_class(self, methname):
        for cls in self.method_resolution_order():
            if methname in cls._fields:
                return cls._fields[methname]
        return MISSING

```

``Base`` 구현의 ``callmethod`` 코드와 함께, 이것은 테스트를 통과시킵니다.

인수를 가진 메소드가 제대로 작동하고 메소드 오버라이딩이 올바르게 구현되었는지 확인하기 위해, 다음의 약간 더 복잡한 테스트를 사용할 수 있으며, 이는 이미 통과합니다:

```python
def test_callmethod_subclassing_and_arguments():
    # Python code
    class A(object):
        def g(self, arg):
            return self.x + arg
    obj = A()
    obj.x = 1
    assert obj.g(4) == 5

    class B(A):
        def g(self, arg):
            return self.x + arg * 2
    obj = B()
    obj.x = 4
    assert obj.g(4) == 12

    # Object model code
    def g_A(self, arg):
        return self.read_attr("x") + arg
    A = Class(name="A", base_class=OBJECT, fields={"g": g_A}, metaclass=TYPE)
    obj = Instance(A)
    obj.write_attr("x", 1)
    assert obj.callmethod("g", 4) == 5

    def g_B(self, arg):
        return self.read_attr("x") + arg * 2
    B = Class(name="B", base_class=A, fields={"g": g_B}, metaclass=TYPE)
    obj = Instance(B)
    obj.write_attr("x", 4)
    assert obj.callmethod("g", 4) == 12
```





## 속성 기반 모델

우리 객체 모델의 가장 간단한 버전이 작동하므로, 이제 이를 변경할 방법을 생각해볼 수 있습니다. 이 섹션에서는 메소드 기반 모델과 속성 기반 모델 사이의 구분을 소개할 것입니다. 이것은 한편으로는 Smalltalk, Ruby, JavaScript와 다른 한편으로는 Python과 Lua 사이의 핵심 차이점 중 하나입니다.

메소드 기반 모델은 메소드 호출을 프로그램 실행의 원시 연산으로 가집니다:

```python
result = obj.f(arg1, arg2)
```

속성 기반 모델은 메소드 호출을 두 단계로 나눕니다: 속성을 찾고 결과를 호출하는 것:

```python
method = obj.f
result = method(arg1, arg2)
```

이 차이점은 다음 테스트에서 보여질 수 있습니다:

```python
def test_bound_method():
    # Python code
    class A(object):
        def f(self, a):
            return self.x + a + 1
    obj = A()
    obj.x = 2
    m = obj.f
    assert m(4) == 7

    class B(A):
        pass
    obj = B()
    obj.x = 1
    m = obj.f
    assert m(10) == 12 # works on subclass too

    # Object model code
    def f_A(self, a):
        return self.read_attr("x") + a + 1
    A = Class(name="A", base_class=OBJECT, fields={"f": f_A}, metaclass=TYPE)
    obj = Instance(A)
    obj.write_attr("x", 2)
    m = obj.read_attr("f")
    assert m(4) == 7

    B = Class(name="B", base_class=A, fields={}, metaclass=TYPE)
    obj = Instance(B)
    obj.write_attr("x", 1)
    m = obj.read_attr("f")
    assert m(10) == 12
```

설정은 메소드 호출에 대한 해당 테스트와 동일하지만, 메소드가 호출되는 방식이 다릅니다. 먼저, 메소드의 이름을 가진 속성이 객체에서 찾아집니다. 그 찾기 연산의 결과는 *바운드 메소드*로, 객체와 클래스에서 발견된 함수 모두를 캡슐화하는 객체입니다. 다음으로, 그 바운드 메소드가 호출 연산으로 호출됩니다[^attributenote].

[^attributenote]: 속성 기반 모델이 메소드 찾기와 호출 모두가 필요하기 때문에 개념적으로 더 복잡해 보입니다. 실제로는 무언가를 호출하는 것이 특별한 속성인 ``__call__``을 찾고 호출하는 것으로 정의되므로 개념적 단순함이 되찾아집니다. 하지만 이것은 이 장에서는 구현되지 않을 것입니다.)

이 동작을 구현하기 위해, ``Base.read_attr`` 구현을 변경해야 합니다. 속성이 딕셔너리에서 발견되지 않으면, 클래스에서 찾아집니다. 클래스에서 발견되고 그 속성이 호출 가능하다면, 바운드 메소드로 변환되어야 합니다. 바운드 메소드를 에뮬레이트하기 위해 단순히 클로저를 사용합니다. ``Base.read_attr``를 변경하는 것에 더하여, 모든 테스트가 여전히 통과하는지 확인하기 위해 ``Base.callmethod``도 메소드를 호출하는 새로운 접근법을 사용하도록 변경할 수 있습니다.

```python
class Base(object):
    ...
    def read_attr(self, fieldname):
        """ read field 'fieldname' out of the object """
        result = self._read_dict(fieldname)
        if result is not MISSING:
            return result
        result = self.cls._read_from_class(fieldname)
        if _is_bindable(result):
            return _make_boundmethod(result, self)
        if result is not MISSING:
            return result
        raise AttributeError(fieldname)

    def callmethod(self, methname, *args):
        """ call method 'methname' with arguments 'args' on object """
        meth = self.read_attr(methname)
        return meth(*args)

def _is_bindable(meth):
    return callable(meth)

def _make_boundmethod(meth, self):
    def bound(*args):
        return meth(self, *args)
    return bound

```

나머지 코드는 전혀 변경할 필요가 없습니다.


## 메타 객체 프로토콜

프로그램에 의해 직접 호출되는 "일반적인" 메소드 외에도, 많은 동적 언어들은 *특별한 메소드*들을 지원합니다. 이것들은 직접 호출되도록 의도되지 않았지만 객체 시스템에 의해 호출될 메소드들입니다. Python에서 이러한 특별한 메소드들은 보통 두 개의 밑줄로 시작하고 끝나는 이름을 가집니다; 예를 들어, ``__init__``입니다. 특별한 메소드들은 원시 연산들을 오버라이드하고 대신 커스텀 동작을 제공하는 데 사용될 수 있습니다. 따라서 이들은 객체 모델 메커니즘에게 특정한 일들을 정확히 어떻게 해야 하는지 알려주는 훅입니다. Python의 객체 모델은 [수십 개의 특별한 메소드](https://docs.python.org/2/reference/datamodel.html#special-method-names)를 가지고 있습니다.

메타 객체 프로토콜은 Smalltalk에 의해 도입되었지만, CLOS 같은 Common Lisp의 객체 시스템에서 훨씬 더 많이 사용되었습니다. 특별한 메소드들의 집합을 가리키는 *메타 객체 프로토콜*이라는 이름이 만들어진 곳도 바로 거기입니다[^kiczales].

[^kiczales]: G. Kiczales, J. des Rivieres, and D. G. Bobrow, The Art of the Metaobject Protocol. Cambridge, Mass: The MIT Press, 1991.

이 장에서 우리는 객체 모델에 이런 메타 훅 세 개를 추가할 것입니다. 이들은 속성을 읽고 쓸 때 정확히 무엇이 일어나는지를 세밀하게 조정하는 데 사용됩니다. 우리가 먼저 추가할 특별한 메소드는 ``__getattr__``와 ``__setattr__``로, 이들은 Python의 동명 메소드들의 동작을 밀접하게 따릅니다.


### 속성 읽기와 쓰기 커스터마이징

``__getattr__`` 메소드는 찾고 있는 속성이 일반적인 방법으로는 발견되지 않을 때; 즉, 인스턴스에서도 클래스에서도 발견되지 않을 때 객체 모델에 의해 호출됩니다. 이 메소드는 찾고 있는 속성의 이름을 인수로 받습니다. ``__getattr__`` 특별 메소드와 동등한 것이 초기 Smalltalk[^smalltalk] 시스템에서 ``doesNotUnderstand:``라는 이름으로 있었습니다.

[^smalltalk]: A. Goldberg, Smalltalk-80: The Language and its Implementation. Addison-Wesley, 1983, page 61.

``__setattr__``의 경우는 약간 다릅니다. 속성을 설정하는 것은 항상 그것을 생성하므로, \newline ``__setattr__``는 속성을 설정할 때 항상 호출됩니다. ``__setattr__`` 메소드가 항상 존재하는지 확인하기 위해, ``OBJECT`` 클래스는 ``__setattr__``의 정의를 가집니다. 이 기본 구현은 지금까지 속성 설정이 했던 일을 단순히 수행하는데, 그것은 속성을 객체의 딕셔너리에 쓰는 것입니다. 이것은 또한 사용자 정의 ``__setattr__``가 경우에 따라 기본 ``OBJECT.__setattr__``에 위임할 수 있게 만듭니다.

이 두 특별한 메소드에 대한 테스트는 다음과 같습니다:

```python
def test_getattr():
    # Python code
    class A(object):
        def __getattr__(self, name):
            if name == "fahrenheit":
                return self.celsius * 9. / 5. + 32
            raise AttributeError(name)

        def __setattr__(self, name, value):
            if name == "fahrenheit":
                self.celsius = (value - 32) * 5. / 9.
            else:
                # call the base implementation
                object.__setattr__(self, name, value)
    obj = A()
    obj.celsius = 30
    assert obj.fahrenheit == 86 # test __getattr__
    obj.celsius = 40
    assert obj.fahrenheit == 104

    obj.fahrenheit = 86 # test __setattr__
    assert obj.celsius == 30
    assert obj.fahrenheit == 86

    # Object model code
    def __getattr__(self, name):
        if name == "fahrenheit":
            return self.read_attr("celsius") * 9. / 5. + 32
        raise AttributeError(name)
    def __setattr__(self, name, value):
        if name == "fahrenheit":
            self.write_attr("celsius", (value - 32) * 5. / 9.)
        else:
            # call the base implementation
            OBJECT.read_attr("__setattr__")(self, name, value)

    A = Class(name="A", base_class=OBJECT,
              fields={"__getattr__": __getattr__, "__setattr__": __setattr__},
              metaclass=TYPE)
    obj = Instance(A)
    obj.write_attr("celsius", 30)
    assert obj.read_attr("fahrenheit") == 86 # test __getattr__
    obj.write_attr("celsius", 40)
    assert obj.read_attr("fahrenheit") == 104
    obj.write_attr("fahrenheit", 86) # test __setattr__
    assert obj.read_attr("celsius") == 30
    assert obj.read_attr("fahrenheit") == 86
```

이 테스트들을 통과시키기 위해, ``Base.read_attr``와 ``Base.write_attr`` 메소드를 변경해야 합니다:

``` python
class Base(object):
    ...

    def read_attr(self, fieldname):
        """ read field 'fieldname' out of the object """
        result = self._read_dict(fieldname)
        if result is not MISSING:
            return result
        result = self.cls._read_from_class(fieldname)
        if _is_bindable(result):
            return _make_boundmethod(result, self)
        if result is not MISSING:
            return result
        meth = self.cls._read_from_class("__getattr__")
        if meth is not MISSING:
            return meth(self, fieldname)
        raise AttributeError(fieldname)

    def write_attr(self, fieldname, value):
        """ write field 'fieldname' into the object """
        meth = self.cls._read_from_class("__setattr__")
        return meth(self, fieldname, value)
```

속성을 읽는 절차는 오류를 발생시키는 대신, 메소드가 존재한다면 ``__getattr__`` 메소드를 fieldname을 인수로 하여 호출하도록 변경됩니다. ``__getattr__`` (그리고 실제로 Python의 모든 특별한 메소드들)은 ``self.read_attr("__getattr__")``를 재귀적으로 호출하는 대신 클래스에서만 찾는다는 점에 주목하세요. 그 이유는 후자가 ``__getattr__``가 객체에 정의되지 않았을 경우 ``read_attr``의 무한 재귀를 초래할 것이기 때문입니다.

속성 쓰기는 ``__setattr__`` 메소드에 완전히 위임됩니다. 이것이 작동하게 하려면, ``OBJECT``는 기본 동작을 호출하는 ``__setattr__`` 메소드를 가져야 합니다, 다음과 같이:

```python
def OBJECT__setattr__(self, fieldname, value):
    self._write_dict(fieldname, value)
OBJECT = Class("object", None, {"__setattr__": OBJECT__setattr__}, None)
```

``OBJECT__setattr__``의 동작은 이전 ``write_attr``의 동작과 같습니다. 이러한 수정으로 새로운 테스트가 통과합니다.


### 디스크립터 프로토콜

서로 다른 온도 스케일 간의 자동 변환을 제공하는 위의 테스트는 작동했지만, ``__getattr__``와 ``__setattr__`` 메소드에서 속성 이름을 명시적으로 확인해야 하므로 작성하기 번거로웠습니다. 이를 해결하기 위해 *디스크립터 프로토콜*이 Python에 도입되었습니다.

``__getattr__``와 ``__setattr__``가 속성을 읽어오는 객체에서 호출되는 반면, 디스크립터 프로토콜은 객체로부터 속성을 가져온 *결과*에 대해 특별한 메소드를 호출합니다. 이것은 메소드를 객체에 바인딩하는 일반화로 볼 수 있습니다 – 그리고 실제로 메소드를 객체에 바인딩하는 것이 디스크립터 프로토콜을 사용하여 수행됩니다. 바운드 메소드 외에도, Python에서 디스크립터 프로토콜의 가장 중요한 사용 사례는 ``staticmethod``, ``classmethod``, ``property``의 구현입니다.

이 하위 섹션에서는 객체 바인딩을 다루는 디스크립터 프로토콜의 부분집합을 소개할 것입니다. 이것은 특별한 메소드 ``__get__``을 사용하여 수행되며, 예제 테스트로 가장 잘 설명됩니다:

```python
def test_get():
    # Python code
    class FahrenheitGetter(object):
        def __get__(self, inst, cls):
            return inst.celsius * 9. / 5. + 32

    class A(object):
        fahrenheit = FahrenheitGetter()
    obj = A()
    obj.celsius = 30
    assert obj.fahrenheit == 86

    # Object model code
    class FahrenheitGetter(object):
        def __get__(self, inst, cls):
            return inst.read_attr("celsius") * 9. / 5. + 32

    A = Class(name="A", base_class=OBJECT,
              fields={"fahrenheit": FahrenheitGetter()},
              metaclass=TYPE)
    obj = Instance(A)
    obj.write_attr("celsius", 30)
    assert obj.read_attr("fahrenheit") == 86
```

The ``__get__`` method is called on the ``FahrenheitGetter`` instance after that
has been looked up in the class of ``obj``. The arguments to ``__get__`` are the
instance where the lookup was done[^secondarg]. 

[^secondarg]: In Python the second argument is the class where the attribute was
found, though we will ignore that here.

Implementing this behaviour is easy. We simply need to change ``_is_bindable``
and \newline ``_make_boundmethod``:

```python
def _is_bindable(meth):
    return hasattr(meth, "__get__")

def _make_boundmethod(meth, self):
    return meth.__get__(self, None)
```

이것으로 테스트가 통과합니다. 바운드 메소드에 관한 이전 테스트들도 여전히 통과하는데, Python의 함수들이 바운드 메소드 객체를 반환하는 ``__get__`` 메소드를 가지고 있기 때문입니다.

실제로 디스크립터 프로토콜은 훨씬 더 복잡합니다. 또한 속성별로 속성 설정이 무엇을 의미하는지 오버라이드하기 위한 ``__set__``을 지원합니다. 또한 현재 구현은 몇 가지 모서리를 자르고 있습니다. ``_make_boundmethod``가 ``meth.read_attr("__get__")``를 사용하는 대신 구현 수준에서 ``__get__`` 메소드를 호출한다는 점에 주목하세요. 이것은 우리의 객체 모델이 객체 모델을 사용하는 표현을 갖는 대신 Python에서 함수와 따라서 메소드를 빌려오기 때문에 필요합니다. 더 완전한 객체 모델은 이 문제를 해결해야 할 것입니다.


## 인스턴스 최적화

객체 모델의 처음 세 변형들이 동작 변화에 관심을 가졌던 반면, 이 마지막 섹션에서는 동작에는 영향을 주지 않는 최적화를 살펴볼 것입니다. 이 최적화는 *맵*이라고 불리며 Self 프로그래밍 언어의 VM에서 개척되었습니다[^self]. 이것은 여전히 가장 중요한 객체 모델 최적화 중 하나입니다: PyPy와 V8 같은 모든 현대 JavaScript VM에서 사용됩니다 (여기서 이 최적화는 *숨겨진 클래스*라고 불립니다).

[^self]: C. Chambers, D. Ungar, and E. Lee, "An efficient implementation of
SELF, a dynamically-typed object-oriented language based on prototypes," in
OOPSLA, 1989, vol. 24.

최적화는 다음 관찰로부터 시작됩니다: 지금까지 구현된 객체 모델에서 모든 인스턴스는 자신의 속성들을 저장하기 위해 완전한 딕셔너리를 사용합니다. 딕셔너리는 해시 맵을 사용하여 구현되는데, 이는 많은 메모리를 차지합니다. 또한 같은 클래스의 인스턴스들의 딕셔너리는 일반적으로 같은 키들을 가집니다. 예를 들어, ``Point`` 클래스가 주어졌을 때, 모든 인스턴스 딕셔너리의 키는 ``"x"``와 ``"y"``일 가능성이 높습니다.

맵 최적화는 이 사실을 활용합니다. 모든 인스턴스의 딕셔너리를 효과적으로 두 부분으로 나눕니다. 같은 속성 이름 집합을 가진 모든 인스턴스 간에 공유될 수 있는 키들을 저장하는 부분(맵)이 있습니다. 그러면 인스턴스는 공유 맵에 대한 참조와 리스트에 있는 속성들의 값들만 저장하면 됩니다 (이는 딕셔너리보다 메모리에서 훨씬 더 컴팩트합니다). 맵은 속성 이름에서 그 리스트의 인덱스로의 매핑을 저장합니다.

그 동작의 간단한 테스트는 다음과 같습니다:

```python
def test_maps():
    # white box test inspecting the implementation
    Point = Class(name="Point", base_class=OBJECT, fields={}, metaclass=TYPE)
    p1 = Instance(Point)
    p1.write_attr("x", 1)
    p1.write_attr("y", 2)
    assert p1.storage == [1, 2]
    assert p1.map.attrs == {"x": 0, "y": 1}

    p2 = Instance(Point)
    p2.write_attr("x", 5)
    p2.write_attr("y", 6)
    assert p1.map is p2.map
    assert p2.storage == [5, 6]

    p1.write_attr("x", -1)
    p1.write_attr("y", -2)
    assert p1.map is p2.map
    assert p1.storage == [-1, -2]

    p3 = Instance(Point)
    p3.write_attr("x", 100)
    p3.write_attr("z", -343)
    assert p3.map is not p1.map
    assert p3.map.attrs == {"x": 0, "z": 1}
```

Note that this is a different flavour of test than the ones we've written
before. All previous tests just tested the behaviour of the classes via the
exposed interfaces. This test instead checks the implementation details of the
``Instance`` class by reading internal attributes and comparing them to
predefined values. Therefore this test can be called a *white-box* test.

The ``attrs`` attribute of the map of ``p1`` describes the layout of the
instance as having two attributes ``"x"`` and ``"y"`` which are stored at
position 0 and 1 of the ``storage`` of ``p1``. Making a second instance ``p2``
and adding to it the same attributes in the same order will make it end up with
the same map. If, on the other hand, a different attribute is added, the map can
of course not be shared.

The ``Map`` class looks like this:

```python
class Map(object):
    def __init__(self, attrs):
        self.attrs = attrs
        self.next_maps = {}

    def get_index(self, fieldname):
        return self.attrs.get(fieldname, -1)

    def next_map(self, fieldname):
        assert fieldname not in self.attrs
        if fieldname in self.next_maps:
            return self.next_maps[fieldname]
        attrs = self.attrs.copy()
        attrs[fieldname] = len(attrs)
        result = self.next_maps[fieldname] = Map(attrs)
        return result

EMPTY_MAP = Map({})
```

Maps have two methods, ``get_index`` and ``next_map``. The former is used to
find the index of an attribute name in the object's storage. The latter is used
when a new attribute is added to an object. In that case the object needs to use
a different map, which ``next_map`` computes. The method uses the ``next_maps``
dictionary to cache already created maps. That way, objects that have the same
layout also end up using the same ``Map`` object.

\aosafigure[166pt]{objmodel-images/maptransition.png}{맵 전이}{500l.objmodel.maptransition}

The ``Instance`` implementation that uses maps looks like this:

```python
class Instance(Base):
    """Instance of a user-defined class. """

    def __init__(self, cls):
        assert isinstance(cls, Class)
        Base.__init__(self, cls, None)
        self.map = EMPTY_MAP
        self.storage = []

    def _read_dict(self, fieldname):
        index = self.map.get_index(fieldname)
        if index == -1:
            return MISSING
        return self.storage[index]

    def _write_dict(self, fieldname, value):
        index = self.map.get_index(fieldname)
        if index != -1:
            self.storage[index] = value
        else:
            new_map = self.map.next_map(fieldname)
            self.storage.append(value)
            self.map = new_map
```

이제 클래스는 ``Instance``가 딕셔너리의 내용을 다른 방식으로 저장할 것이므로 fields 딕셔너리로 ``Base``에 ``None``을 전달합니다. 따라서 ``_read_dict``와 ``_write_dict`` 메소드를 오버라이드해야 합니다. 실제 구현에서는 ``Base`` 클래스가 더 이상 fields 딕셔너리 저장을 담당하지 않도록 리팩터링할 것이지만, 지금은 인스턴스가 거기에 ``None``을 저장하는 것으로 충분합니다.

새로 생성된 인스턴스는 속성이 없고 비어있는 저장소를 가진 ``EMPTY_MAP``을 사용하여 시작합니다. ``_read_dict``를 구현하기 위해, 인스턴스의 맵에 속성 이름의 인덱스를 요청합니다. 그러면 저장소 리스트의 해당 항목이 반환됩니다.

fields 딕셔너리에 쓰는 것은 두 가지 경우가 있습니다. 한편으로는 기존 속성의 값을 변경할 수 있습니다. 이것은 해당 인덱스의 저장소를 단순히 변경하여 수행됩니다. 다른 한편으로는, 속성이 아직 존재하지 않는다면, ``next_map`` 메소드를 사용하여 *맵 전환* (\aosafigref{500l.objmodel.maptransition})이 필요합니다. 새 속성의 값이 저장소 리스트에 추가됩니다.


이 최적화는 무엇을 달성할까요? 같은 레이아웃을 가진 많은 인스턴스가 있는 일반적인 경우에서 메모리 사용을 최적화합니다. 이것은 범용 최적화가 아닙니다: 완전히 다른 속성 집합을 가진 인스턴스를 생성하는 코드는 단순히 딕셔너리를 사용하는 것보다 더 큰 메모리 풋프린트를 가질 것입니다.

이것은 동적 언어를 최적화할 때 흔한 문제입니다. 모든 경우에서 더 빠르거나 더 적은 메모리를 사용하는 최적화를 찾는 것은 종종 불가능합니다. 실제로, 선택된 최적화들은 언어가 *일반적으로* 사용되는 방식에 적용되며, 극도로 동적인 기능을 사용하는 프로그램의 동작을 잠재적으로 악화시킬 수 있습니다.

맵의 또 다른 흥미로운 측면은 여기서는 메모리 사용만을 최적화하지만, 적시 컴파일(JIT) 컴파일러를 사용하는 실제 VM에서는 프로그램의 성능도 향상시킨다는 것입니다. 이를 달성하기 위해, JIT은 맵을 사용하여 속성 검색을 객체 저장소의 고정 오프셋에서의 검색으로 컴파일하여, 모든 딕셔너리 검색을 완전히 제거합니다[^lookups].

[^lookups]: 그것이 어떻게 작동하는지는 이 장의 범위를 벗어납니다. 몇 년 전에 쓴 논문에서 그것에 대한 합리적으로 읽기 쉬운 설명을 제공하려고 했습니다. 그것은 기본적으로 이 장의 것의 변형인 객체 모델을 사용합니다: C. F. Bolz, A. Cuni, M. Fijałkowski, M. Leuschel, S. Pedroni, and A. Rigo, "Runtime feedback in a meta-tracing JIT for efficient dynamic languages," in Proceedings of the 6th Workshop on Implementation, Compilation, Optimization of Object-Oriented Languages, Programs and Systems, New York, NY, USA, 2011, pp. 9:1–9:8.

## 잠재적 확장

우리의 객체 모델을 확장하고 다양한 언어 설계 선택지를 실험하는 것은 쉽습니다. 다음은 몇 가지 가능성입니다:

- 가장 쉬운 일은 추가적인 특별한 메소드들을 더하는 것입니다. 추가하기 쉽고 흥미로운 것들로는 ``__init__``, ``__getattribute__``, ``__set__``이 있습니다.

- 모델은 다중 상속을 지원하도록 매우 쉽게 확장될 수 있습니다. 이를 위해서는 모든 클래스가 기본 클래스들의 리스트를 갖게 됩니다. 그러면 ``Class.method_resolution_order`` 메소드가 메소드 검색을 지원하도록 변경되어야 합니다. 간단한 메소드 해결 순서는 중복 제거를 포함한 깊이 우선 탐색을 사용하여 계산될 수 있습니다. 더 복잡하지만 더 나은 것은 [C3 알고리즘](https://www.python.org/download/releases/2.3/mro/)으로, 다이아몬드 모양 다중 상속 계층구조의 기반에서 더 나은 처리를 추가하고 무의미한 상속 패턴을 거부합니다.

- 더 급진적인 변화는 클래스와 인스턴스 간의 구분을 제거하는 프로토타입 모델로 전환하는 것입니다.


## 결론

객체지향 프로그래밍 언어 설계의 핵심 측면 중 일부는 그 객체 모델의 세부사항입니다. 작은 객체 모델 프로토타입을 작성하는 것은 기존 언어의 내부 작동을 더 잘 이해하고 객체지향 언어의 설계 공간에 대한 통찰을 얻는 쉽고 재미있는 방법입니다. 객체 모델을 가지고 노는 것은 파싱과 코드 실행 같은 언어 구현의 더 지루한 부분들에 대해 걱정하지 않고 다양한 언어 설계 아이디어를 실험하는 좋은 방법입니다.

이러한 객체 모델들은 단순히 실험의 수단으로서가 아니라 실제로도 유용할 수 있습니다. 다른 언어에 임베드되고 사용될 수 있습니다. 이 접근법의 예는 흔합니다: GLib과 다른 Gnome 라이브러리에서 사용되는 C로 작성된 GObject 객체 모델; 또는 JavaScript의 다양한 클래스 시스템 구현들.

