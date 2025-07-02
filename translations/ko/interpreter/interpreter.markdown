title: 파이썬으로 작성한 파이썬 인터프리터
author: Allison Kaptur
<markdown>
_앨리슨은 Dropbox의 엔지니어로, 세계에서 가장 큰 규모의 파이썬 클라이언트 네트워크 중 하나를 유지 관리하는 일을 담당하고 있습니다. Dropbox에 입사하기 전에는 뉴욕의 프로그래머를 위한 작가 수련회인 Recurse Center에서 진행자로 일했습니다. 그녀는 PyCon North America에서 파이썬 내부 구조에 대해 발표했으며 이상한 버그를 좋아합니다. [akaptur.com](http://akaptur.com)에서 블로그를 운영하고 있습니다._

_(이 장은 [중국어 간체](http://qingyunha.github.io/taotao/)로도 제공됩니다)_. 
</markdown>
## 소개

Byterun은 파이썬으로 구현된 파이썬 인터프리터입니다. Byterun을 작업하면서 파이썬 인터프리터의 핵심 구조가 500줄이라는 크기 제한에 쉽게 맞출 수 있다는 것을 발견하고 놀라며 기뻐했습니다. 이 장에서는 인터프리터의 구조를 살펴보고 더 깊이 탐구할 수 있는 충분한 맥락을 제공합니다. 목표는 인터프리터에 대해 알아야 할 모든 것을 설명하는 것이 아닙니다. 프로그래밍과 컴퓨터 과학의 많은 흥미로운 영역들처럼, 이 주제에 대한 깊은 이해를 발전시키는 데는 수년이 걸릴 수 있습니다.

Byterun은 Ned Batchelder와 제가 Paul Swartz의 작업을 기반으로 작성했습니다. 그 구조는 파이썬의 주요 구현체인 CPython과 유사하므로, Byterun을 이해하면 일반적인 인터프리터와 특히 CPython 인터프리터를 이해하는 데 도움이 됩니다. (어떤 파이썬을 사용하고 있는지 모른다면, 아마도 CPython일 것입니다.) 짧은 길이에도 불구하고 Byterun은 대부분의 간단한 파이썬 프로그램을 실행할 수 있습니다[^versions].

[^versions]: 이 장은 파이썬 3.5 이하 버전에서 생성된 바이트코드를 기준으로 합니다. 파이썬 3.6에서 바이트코드 명세에 일부 변경사항이 있었기 때문입니다. 

### 파이썬 인터프리터

시작하기 전에 "파이썬 인터프리터"가 무엇을 의미하는지 명확히 해보겠습니다. 파이썬을 논의할 때 "인터프리터"라는 단어는 다양한 방식으로 사용될 수 있습니다. 때로는 인터프리터가 파이썬 REPL, 즉 명령줄에서 `python`을 입력하면 나타나는 대화형 프롬프트를 가리키기도 합니다. 때로는 사람들이 "파이썬 인터프리터"를 "파이썬"과 거의 동일하게 사용하여 파이썬 코드를 처음부터 끝까지 실행하는 것을 말하기도 합니다. 이 장에서 "인터프리터"는 더 좁은 의미를 가집니다. 파이썬 프로그램을 실행하는 과정의 마지막 단계입니다.

인터프리터가 작업을 시작하기 전에 파이썬은 세 가지 다른 단계를 수행합니다: 어휘 분석(lexing), 구문 분석(parsing), 그리고 컴파일입니다. 이 단계들은 함께 프로그래머의 소스 코드를 텍스트 줄에서 인터프리터가 이해할 수 있는 명령어를 포함하는 구조화된 _코드 객체(code objects)_로 변환합니다. 인터프리터의 역할은 이러한 코드 객체를 가져와서 명령어를 따르는 것입니다.

파이썬 코드를 실행하는 과정에 컴파일이 포함된다는 것을 듣고 놀랄 수 있습니다. 파이썬은 종종 C나 Rust 같은 "컴파일된" 언어와 대조되는 Ruby나 Perl 같은 "인터프리터" 언어라고 불립니다. 하지만 이러한 용어는 보이는 것만큼 정확하지 않습니다. 파이썬을 포함한 대부분의 인터프리터 언어들은 실제로 컴파일 단계를 포함합니다. 파이썬이 "인터프리터" 언어라고 불리는 이유는 컴파일된 언어에 비해 컴파일 단계에서 상대적으로 적은 작업을 수행하고 (인터프리터가 상대적으로 더 많은 작업을 수행하기) 때문입니다. 이 장의 뒷부분에서 볼 수 있듯이, 파이썬 컴파일러는 C 컴파일러보다 프로그램의 동작에 대한 정보를 훨씬 적게 가지고 있습니다.

### 파이썬 파이썬 인터프리터

Byterun은 파이썬으로 작성된 파이썬 인터프리터입니다. 이것이 이상하게 보일 수 있지만, C로 C 컴파일러를 작성하는 것보다 더 이상한 것은 아닙니다. (실제로 널리 사용되는 C 컴파일러인 gcc는 C로 작성되었습니다.) 파이썬 인터프리터는 거의 모든 언어로 작성할 수 있습니다.

파이썬으로 파이썬 인터프리터를 작성하는 것은 장점과 단점을 모두 가지고 있습니다. 가장 큰 단점은 속도입니다. Byterun을 통해 코드를 실행하는 것은 인터프리터가 C로 작성되고 세심하게 최적화된 CPython에서 실행하는 것보다 훨씬 느립니다. 하지만 Byterun은 원래 학습 목적으로 설계되었으므로, 속도는 우리에게 중요하지 않습니다. 파이썬을 사용하는 가장 큰 장점은 파이썬 런타임의 나머지 부분, 특히 객체 시스템을 구현하지 않고도 인터프리터 *만을* 더 쉽게 구현할 수 있다는 것입니다. 예를 들어, Byterun은 클래스를 생성해야 할 때 "실제" 파이썬으로 되돌아갈 수 있습니다. 또 다른 장점은 Byterun이 이해하기 쉽다는 것입니다. 부분적으로는 많은 사람들이 읽기 쉽다고 여기는 고수준 언어(파이썬!)로 작성되었기 때문입니다. (우리는 또한 Byterun에서 인터프리터 최적화를 제외했습니다. 다시 한 번 속도보다 명확성과 단순성을 선호했습니다.)

## 인터프리터 구축하기

Byterun의 코드를 살펴보기 시작하기 전에, 인터프리터의 구조에 대한 상위 수준의 맥락이 필요합니다. 파이썬 인터프리터는 어떻게 작동할까요?

파이썬 인터프리터는 _가상 머신(virtual machine)_입니다. 이는 물리적 컴퓨터를 에뮬레이트하는 소프트웨어라는 의미입니다. 이 특별한 가상 머신은 스택 머신입니다. 여러 스택을 조작하여 연산을 수행합니다 (특정 메모리 위치에 쓰고 읽는 레지스터 머신과 대조됩니다).

파이썬 인터프리터는 _바이트코드 인터프리터(bytecode interpreter)_입니다. 그 입력은 _바이트코드(bytecode)_라고 불리는 명령어 집합입니다. 파이썬을 작성할 때, 어휘 분석기, 구문 분석기, 그리고 컴파일러가 인터프리터가 작업할 코드 객체를 생성합니다. 각 코드 객체는 실행될 명령어 집합(그것이 바이트코드입니다)과 인터프리터가 필요로 할 기타 정보를 포함합니다. 바이트코드는 파이썬 코드의 _중간 표현(intermediate representation)_입니다. 인터프리터가 이해할 수 있는 방식으로 여러분이 작성한 소스 코드를 표현합니다. 이는 어셈블리 언어가 C 코드와 하드웨어 사이의 중간 표현 역할을 하는 것과 유사합니다.

### 작은 인터프리터

이것을 구체적으로 만들기 위해, 매우 최소한의 인터프리터부터 시작해보겠습니다. 이 인터프리터는 숫자를 더하는 것만 할 수 있고, 단지 세 가지 명령어만 이해합니다. 실행할 수 있는 모든 코드는 이 세 가지 명령어의 서로 다른 조합으로 구성됩니다. 세 가지 명령어는 다음과 같습니다:

- `LOAD_VALUE`
- `ADD_TWO_VALUES`
- `PRINT_ANSWER`

이 장에서는 어휘 분석기, 구문 분석기, 컴파일러에 대해서는 다루지 않으므로, 명령어 집합이 어떻게 생성되는지는 중요하지 않습니다. `7 + 5`를 작성하면 컴파일러가 이 세 가지 명령어의 조합을 출력한다고 상상할 수 있습니다. 또는 적절한 컴파일러가 있다면, 동일한 명령어 조합으로 변환되는 Lisp 구문을 작성할 수도 있습니다. 인터프리터는 신경 쓰지 않습니다. 중요한 것은 우리의 인터프리터가 잘 구성된 명령어 배열을 받는다는 것입니다.

다음과 같이

```python
7 + 5
```

이 명령어 집합을 생성한다고 가정해봅시다:

```python
what_to_execute = {
    "instructions": [("LOAD_VALUE", 0),  # 첫 번째 숫자
                     ("LOAD_VALUE", 1),  # 두 번째 숫자
                     ("ADD_TWO_VALUES", None),
                     ("PRINT_ANSWER", None)],
    "numbers": [7, 5] }
```

파이썬 인터프리터는 _스택 머신_이므로, 두 숫자를 더하기 위해 스택을 조작해야 합니다 (\aosafigref{500l.interpreter.stackmachine}). 인터프리터는 첫 번째 명령어인 `LOAD_VALUE`를 실행하고 첫 번째 숫자를 스택에 푸시하는 것으로 시작합니다. 다음으로 두 번째 숫자를 스택에 푸시합니다. 세 번째 명령어인 `ADD_TWO_VALUES`에서는 두 숫자를 모두 팝하고, 더한 다음, 결과를 스택에 푸시합니다. 마지막으로 답을 스택에서 다시 팝하여 출력합니다.

\aosafigure[240pt]{interpreter-images/interpreter-stack.png}{스택 머신}{500l.interpreter.stackmachine}

`LOAD_VALUE` 명령어는 인터프리터에게 스택에 숫자를 푸시하라고 지시하지만, 명령어 자체만으로는 어떤 숫자인지 명시하지 않습니다. 각 명령어는 추가 정보가 필요합니다. 인터프리터에게 로드할 숫자를 어디서 찾을지 알려주는 것입니다. 따라서 우리의 명령어 집합은 두 부분으로 구성됩니다: 명령어 자체와 명령어가 필요로 할 상수 목록입니다. (파이썬에서 우리가 "명령어"라고 부르는 것은 바이트코드이고, 아래의 "실행할 것" 객체는 _코드 객체_입니다.)

왜 숫자를 명령어에 직접 넣지 않을까요? 숫자 대신 문자열을 더한다고 상상해보세요. 문자열은 임의로 클 수 있으므로 명령어와 함께 넣고 싶지 않을 것입니다. 이 설계는 또한 필요한 각 객체의 복사본을 하나만 가질 수 있다는 것을 의미합니다. 예를 들어 `7 + 7`을 더하려면 `"numbers"`는 단지 `[7]`일 수 있습니다.

`ADD_TWO_VALUES` 외에 다른 명령어가 왜 필요한지 궁금할 수 있습니다. 실제로 두 숫자를 더하는 간단한 경우에는 이 예제가 약간 억지스럽습니다. 하지만 이 명령어는 더 복잡한 프로그램을 위한 구성 요소입니다. 예를 들어, 지금까지 정의한 명령어만으로도 적절한 명령어 집합이 주어지면 이미 세 개의 값 또는 임의의 개수의 값을 더할 수 있습니다. 스택은 인터프리터의 상태를 추적하는 깔끔한 방법을 제공하며, 더 복잡해져도 지원할 것입니다.

이제 인터프리터 자체를 작성하기 시작해봅시다. 인터프리터 객체는 스택을 가지고 있으며, 이를 리스트로 표현하겠습니다. 객체는 또한 각 명령어를 실행하는 방법을 설명하는 메서드를 가지고 있습니다. 예를 들어, `LOAD_VALUE`의 경우 인터프리터는 값을 스택에 푸시합니다.

```python
class Interpreter:
    def __init__(self):
        self.stack = []

    def LOAD_VALUE(self, number):
        self.stack.append(number)

    def PRINT_ANSWER(self):
        answer = self.stack.pop()
        print(answer)

    def ADD_TWO_VALUES(self):
        first_num = self.stack.pop()
        second_num = self.stack.pop()
        total = first_num + second_num
        self.stack.append(total)
```

이 세 함수는 우리의 인터프리터가 이해하는 세 가지 명령어를 구현합니다. 인터프리터에는 한 가지가 더 필요합니다: 모든 것을 연결하고 실제로 실행하는 방법입니다. 이 메서드인 `run_code`는 위에서 정의한 `what_to_execute` 딕셔너리를 인수로 받습니다. 각 명령어를 반복하고, 해당 명령어에 인수가 있으면 처리한 다음, 인터프리터 객체의 해당 메서드를 호출합니다.

```python
    def run_code(self, what_to_execute):
        instructions = what_to_execute["instructions"]
        numbers = what_to_execute["numbers"]
        for each_step in instructions:
            instruction, argument = each_step
            if instruction == "LOAD_VALUE":
                number = numbers[argument]
                self.LOAD_VALUE(number)
            elif instruction == "ADD_TWO_VALUES":
                self.ADD_TWO_VALUES()
            elif instruction == "PRINT_ANSWER":
                self.PRINT_ANSWER()
```

테스트하기 위해 객체의 인스턴스를 생성한 다음 위에서 정의한 7 + 5를 더하는 명령어 집합으로 `run_code` 메서드를 호출할 수 있습니다.

```python
    interpreter = Interpreter()
    interpreter.run_code(what_to_execute)
```

예상대로 답을 출력합니다: 12.

이 인터프리터는 상당히 제한적이지만, 이 과정은 실제 파이썬 인터프리터가 숫자를 더하는 방법과 거의 정확히 동일합니다. 이 작은 예제에서도 주목할 몇 가지가 있습니다.

첫째, 일부 명령어는 인수가 필요합니다. 실제 파이썬 바이트코드에서는 약 절반의 명령어가 인수를 가집니다. 인수는 우리 예제에서처럼 명령어와 함께 패킹됩니다. _명령어_의 인수가 호출되는 메서드의 인수와 다르다는 점에 주목하세요.

둘째, `ADD_TWO_VALUES` 명령어는 어떤 인수도 필요로 하지 않았습니다. 대신 더해질 값들이 인터프리터의 스택에서 팝되었습니다. 이것이 스택 기반 인터프리터의 정의적 특징입니다.

유효한 명령어 집합이 주어지면 우리의 인터프리터를 전혀 변경하지 않고도 한 번에 두 개 이상의 숫자를 더할 수 있다는 것을 기억하세요. 아래 명령어 집합을 고려해보세요. 무엇이 일어날 것으로 예상하십니까? 친근한 컴파일러가 있다면, 이 명령어 집합을 생성하기 위해 어떤 코드를 작성할 수 있을까요? \newpage

```python
    what_to_execute = {
        "instructions": [("LOAD_VALUE", 0),
                         ("LOAD_VALUE", 1),
                         ("ADD_TWO_VALUES", None),
                         ("LOAD_VALUE", 2),
                         ("ADD_TWO_VALUES", None),
                         ("PRINT_ANSWER", None)],
        "numbers": [7, 5, 8] }
```

이 시점에서 이 구조가 어떻게 확장 가능한지 보기 시작할 수 있습니다: 인터프리터 객체에 더 많은 연산을 설명하는 메서드를 추가할 수 있습니다 (잘 구성된 명령어 집합을 제공할 컴파일러가 있는 한).

#### 변수

다음으로 인터프리터에 변수를 추가해보겠습니다. 변수는 변수의 값을 저장하는 명령어 `STORE_NAME`, 그것을 검색하는 명령어 `LOAD_NAME`, 그리고 변수 이름에서 값으로의 매핑이 필요합니다. 지금은 네임스페이스와 스코핑을 무시하므로 변수 매핑을 인터프리터 객체 자체에 저장할 수 있습니다. 마지막으로 `what_to_execute`가 상수 목록 외에도 변수 이름 목록을 가지고 있는지 확인해야 합니다.

```python
>>> def s():
...     a = 1
...     b = 2
...     print(a + b)
# a friendly compiler transforms `s` into:
    what_to_execute = {
        "instructions": [("LOAD_VALUE", 0),
                         ("STORE_NAME", 0),
                         ("LOAD_VALUE", 1),
                         ("STORE_NAME", 1),
                         ("LOAD_NAME", 0),
                         ("LOAD_NAME", 1),
                         ("ADD_TWO_VALUES", None),
                         ("PRINT_ANSWER", None)],
        "numbers": [1, 2],
        "names":   ["a", "b"] }
```

새로운 구현은 아래와 같습니다. 어떤 이름이 어떤 값에 바인드되는지 추적하기 위해 `__init__` 메서드에 `environment` 딕셔너리를 추가하겠습니다. 또한 `STORE_NAME`과 `LOAD_NAME`도 추가하겠습니다. 이 메서드들은 먼저 해당 변수 이름을 찾은 다음 딕셔너리를 사용하여 그 값을 저장하거나 검색합니다.

명령어의 인수는 이제 두 가지 다른 의미를 가질 수 있습니다: "numbers" 목록의 인덱스이거나 "names" 목록의 인덱스일 수 있습니다. 인터프리터는 실행 중인 명령어를 확인하여 어떤 것이어야 하는지 앍니다. 이 논리와 명령어의 인수가 무엇을 의미하는지의 매핑을 별도의 메서드로 분리하겠습니다. \newpage

```python
class Interpreter:
    def __init__(self):
        self.stack = []
        self.environment = {}

    def STORE_NAME(self, name):
        val = self.stack.pop()
        self.environment[name] = val

    def LOAD_NAME(self, name):
        val = self.environment[name]
        self.stack.append(val)

    def parse_argument(self, instruction, argument, what_to_execute):
        """ 각 명령어의 인수가 무엇을 의미하는지 이해합니다."""
        numbers = ["LOAD_VALUE"]
        names = ["LOAD_NAME", "STORE_NAME"]

        if instruction in numbers:
            argument = what_to_execute["numbers"][argument]
        elif instruction in names:
            argument = what_to_execute["names"][argument]

        return argument

    def run_code(self, what_to_execute):
        instructions = what_to_execute["instructions"]
        for each_step in instructions:
            instruction, argument = each_step
            argument = self.parse_argument(instruction, argument, what_to_execute)

            if instruction == "LOAD_VALUE":
                self.LOAD_VALUE(argument)
            elif instruction == "ADD_TWO_VALUES":
                self.ADD_TWO_VALUES()
            elif instruction == "PRINT_ANSWER":
                self.PRINT_ANSWER()
            elif instruction == "STORE_NAME":
                self.STORE_NAME(argument)
            elif instruction == "LOAD_NAME":
                self.LOAD_NAME(argument)
```

단지 다섯 개의 명령어만으로도 `run_code` 메서드가 지루해지기 시작합니다. 이 구조를 유지한다면 각 명령어에 대해 `if` 문의 분기가 하나씩 필요할 것입니다. 여기서 파이썬의 동적 메서드 검색을 활용할 수 있습니다. `FOO`라는 명령어를 실행하기 위해 항상 `FOO`라는 메서드를 정의할 것이므로, 큰 `if` 문을 사용하는 대신 파이썬의 `getattr` 함수를 사용하여 즉석에서 메서드를 찾을 수 있습니다. 그러면 `run_code` 메서드는 다음과 같이 보입니다: \newpage

```python
    def execute(self, what_to_execute):
        instructions = what_to_execute["instructions"]
        for each_step in instructions:
            instruction, argument = each_step
            argument = self.parse_argument(instruction, argument, what_to_execute)
            bytecode_method = getattr(self, instruction)
            if argument is None:
                bytecode_method()
            else:
                bytecode_method(argument)
```

## 실제 파이썬 바이트코드

이 시점에서 우리의 장난감 명령어 집합을 버리고 실제 파이썬 바이트코드로 전환하겠습니다. 바이트코드의 구조는 우리의 장난감 인터프리터의 장황한 명령어 집합과 비슷하지만, 각 명령어를 식별하기 위해 긴 이름 대신 하나의 바이트를 사용한다는 점이 다릅니다. 이 구조를 이해하기 위해 짧은 함수의 바이트코드를 살펴보겠습니다. 아래 예제를 고려해보세요:

```python
>>> def cond():
...     x = 3
...     if x < 5:
...         return 'yes'
...     else:
...         return 'no'
...
```

파이썬은 런타임에 많은 내부 사항을 노출하며, REPL에서 바로 접근할 수 있습니다. 함수 객체 `cond`에 대해 `cond.__code__`는 연결된 코드 객체이고, `cond.__code__.co_code`는 바이트코드입니다. 파이썬 코드를 작성할 때 이러한 속성들을 직접 사용할 좋은 이유는 거의 없지만, 온갖 장난을 칠 수 있게 해주고 내부를 이해하기 위해 살펴볼 수 있게 해줍니다.

```python
>>> cond.__code__.co_code  # the bytecode as raw bytes
b'd\x01\x00}\x00\x00|\x00\x00d\x02\x00k\x00\x00r\x16\x00d\x03\x00Sd\x04\x00Sd\x00
   \x00S'
>>> list(cond.__code__.co_code)  # the bytecode as numbers
[100, 1, 0, 125, 0, 0, 124, 0, 0, 100, 2, 0, 107, 0, 0, 114, 22, 0, 100, 3, 0, 83, 
 100, 4, 0, 83, 100, 0, 0, 83]
```

바이트코드를 그냥 출력하면 이해할 수 없어 보입니다. 알 수 있는 것은 단지 바이트들의 연속이라는 것뿐입니다. 다행히 이를 이해하는 데 사용할 수 있는 강력한 도구가 있습니다: 파이썬 표준 라이브러리의 `dis` 모듈입니다.

`dis`는 바이트코드 디어셈블러입니다. 디어셈블러는 어셈블리 코드나 바이트코드와 같이 머신을 위해 작성된 낮은 수준의 코드를 가져와서 사람이 읽을 수 있는 방식으로 출력합니다. `dis.dis`를 실행하면 전달된 바이트코드에 대한 설명을 출력합니다. \newpage

```python
>>> dis.dis(cond)
  2           0 LOAD_CONST               1 (3)
              3 STORE_FAST               0 (x)

  3           6 LOAD_FAST                0 (x)
              9 LOAD_CONST               2 (5)
             12 COMPARE_OP               0 (<)
             15 POP_JUMP_IF_FALSE       22

  4          18 LOAD_CONST               3 ('yes')
             21 RETURN_VALUE

  6     >>   22 LOAD_CONST               4 ('no')
             25 RETURN_VALUE
             26 LOAD_CONST               0 (None)
             29 RETURN_VALUE
```

이 모든 것이 무엇을 의미할까요? 첫 번째 명령어 `LOAD_CONST`를 예로 살펴보겠습니다. 첫 번째 열의 숫자(`2`)는 파이썬 소스 코드의 줄 번호를 보여줍니다. 두 번째 열은 바이트코드의 인덱스로, `LOAD_CONST` 명령어가 0번 위치에 나타난다는 것을 알려줍니다. 세 번째 열은 명령어 자체로, 사람이 읽을 수 있는 이름으로 매핑되었습니다. 네 번째 열이 있을 때는 해당 명령어의 인수입니다. 다섯 번째 열이 있을 때는 인수가 무엇을 의미하는지에 대한 힘트입니다.

이 바이트코드의 처음 몇 바이트를 고려해보세요: [100, 1, 0, 125, 0, 0]. 이 6개의 바이트는 인수와 함께 두 개의 명령어를 나타냅니다. 바이트에서 이해 가능한 문자열로의 매핑인 `dis.opname`을 사용하여 100과 125 명령어가 무엇에 매핑되는지 알아낼 수 있습니다:

```python
>>> dis.opname[100]
'LOAD_CONST'
>>> dis.opname[125]
'STORE_FAST'
```

두 번째와 세 번째 바이트(1, 0)는 `LOAD_CONST`의 인수이고, 다섯 번째와 여섯 번째 바이트(0, 0)는 `STORE_FAST`의 인수입니다. 우리의 장난감 예제에서처럼 `LOAD_CONST`는 로드할 상수를 어디서 찾을지 알아야 하고, `STORE_FAST`는 저장할 이름을 찾아야 합니다. (파이썬의 `LOAD_CONST`는 우리의 장난감 인터프리터의 `LOAD_VALUE`와 동일하고, `LOAD_FAST`는 `LOAD_NAME`과 동일합니다.) 따라서 이 6개의 바이트는 코드의 첫 번째 줄인 `x = 3`을 나타냅니다. (왜 각 인수에 2바이트를 사용할까요? 파이썬이 상수와 이름을 찾기 위해 2바이트 대신 1바이트만 사용한다면, 하나의 코드 객체에 연결된 이름/상수를 256개만 가질 수 있습니다. 2바이트를 사용하면 256의 제곱, 즉 65,536개까지 가질 수 있습니다.)

### 조건문과 루프

지금까지 인터프리터는 명령어를 하나씩 단계별로 실행하는 방식으로 코드를 실행했습니다. 이는 문제가 됩니다. 종종 특정 명령어를 여러 번 실행하거나 특정 조건에서 건너뛰고 싶을 때가 있기 때문입니다. 코드에서 루프와 if 문을 작성할 수 있게 하려면, 인터프리터가 명령어 집합 내에서 점프할 수 있어야 합니다. 어떤 면에서 파이썬은 바이트코드의 `GOTO` 문으로 루프와 조건문을 처리합니다! 함수 `cond`의 디어셈블리를 다시 살펴보세요: \newpage

```python
>>> dis.dis(cond)
  2           0 LOAD_CONST               1 (3)
              3 STORE_FAST               0 (x)

  3           6 LOAD_FAST                0 (x)
              9 LOAD_CONST               2 (5)
             12 COMPARE_OP               0 (<)
             15 POP_JUMP_IF_FALSE       22

  4          18 LOAD_CONST               3 ('yes')
             21 RETURN_VALUE

  6     >>   22 LOAD_CONST               4 ('no')
             25 RETURN_VALUE
             26 LOAD_CONST               0 (None)
             29 RETURN_VALUE
```

코드 3번 줄의 조건 `if x < 5`는 네 개의 명령어로 컴파일됩니다: `LOAD_FAST`, `LOAD_CONST`, `COMPARE_OP`, 그리고 `POP_JUMP_IF_FALSE`. `x < 5`는 `x`를 로드하고, 5를 로드하고, 두 값을 비교하는 코드를 생성합니다. `POP_JUMP_IF_FALSE` 명령어가 `if`를 구현하는 역할을 담당합니다. 이 명령어는 인터프리터의 스택에서 맨 위 값을 팝합니다. 값이 참이면 아무일도 일어나지 않습니다. (값은 "truthy"일 수 있습니다. 말 그대로 `True` 객체일 필요는 없습니다.) 값이 거짓이면 인터프리터는 다른 명령어로 점프합니다.

도착할 명령어를 점프 대상(jump target)이라고 하며, `POP_JUMP` 명령어의 인수로 제공됩니다. 여기서 점프 대상은 22입니다. 인덱스 22의 명령어는 6번 줄의 `LOAD_CONST`입니다. (`dis`는 점프 대상을 `>>`로 표시합니다.) `x < 5`의 결과가 False라면, 인터프리터는 4번 줄(`return "yes"`)\uc744 건너뛰고 바로 6번 줄(`return "no"`)\ub85c 점프합니다. 따라서 인터프리터는 점프 명령어를 사용하여 명령어 집합의 일부를 선택적으로 건너뛱니다.

파이썬 루프도 점프에 의존합니다. 아래 바이트코드에서 `while x < 5` 줄이 `if x < 10`과 거의 동일한 바이트코드를 생성한다는 점에 주목하세요. 두 경우 모두 비교가 계산되고 `POP_JUMP_IF_FALSE`가 다음에 실행될 명령어를 제어합니다. 4번 줄의 끝, 즉 루프 본문의 끝에서 `JUMP_ABSOLUTE` 명령어는 항상 인터프리터를 루프의 맨 위인 명령어 9로 다시 보냅니다. x < 5가 거짓이 되면 `POP_JUMP_IF_FALSE`는 인터프리터를 루프의 끝을 지나 명령어 34로 점프시킵니다. 

```python
>>> def loop():
...      x = 1
...      while x < 5:
...          x = x + 1
...      return x
...
>>> dis.dis(loop)
  2           0 LOAD_CONST               1 (1)
              3 STORE_FAST               0 (x)

  3           6 SETUP_LOOP              26 (to 35)
        >>    9 LOAD_FAST                0 (x)
             12 LOAD_CONST               2 (5)
             15 COMPARE_OP               0 (<)
             18 POP_JUMP_IF_FALSE       34

  4          21 LOAD_FAST                0 (x)
             24 LOAD_CONST               1 (1)
             27 BINARY_ADD
             28 STORE_FAST               0 (x)
             31 JUMP_ABSOLUTE            9
        >>   34 POP_BLOCK

  5     >>   35 LOAD_FAST                0 (x)
             38 RETURN_VALUE
```

### 바이트코드 탐구

여러분이 작성한 함수에 `dis.dis`를 실행해보시기 바랍니다. 탐구해볼 몇 가지 질문들입니다:

- 파이썬 인터프리터에게 for 루프와 while 루프의 차이점은 무엇인가요?
- 동일한 바이트코드를 생성하는 서로 다른 함수를 어떻게 작성할 수 있나요?
- `elif`는 어떻게 작동할까요? 리스트 컴프리헨션은 어떨까요?

## 프레임

지금까지 파이썬 가상 머신이 스택 머신이라는 것을 배웠습니다. 명령어를 단계별로 실행하고 점프하며, 스택에 값을 푸시하고 팝합니다. 하지만 우리의 사고 모델에는 아직 몇 가지 공백이 있습니다. 위의 예제들에서 마지막 명령어는 `RETURN_VALUE`이고, 이는 코드의 `return` 문에 해당합니다. 그런데 이 명령어는 어디로 돌아갈까요?

이 질문에 답하기 위해 복잡성의 층을 추가해야 합니다: 프레임입니다. 프레임은 코드 조각에 대한 정보와 맥락의 모음입니다. 프레임은 파이썬 코드가 실행되는 동안 즉석에서 생성되고 소멸됩니다. 각 함수 *호출*에 해당하는 하나의 프레임이 있습니다. 따라서 각 프레임은 하나의 코드 객체와 연결되어 있지만, 하나의 코드 객체는 여러 프레임을 가질 수 있습니다. 자기 자신을 10번 재귀적으로 호출하는 함수가 있다면, 11개의 프레임이 있을 것입니다. 각 재귀 레벨에 하나씩과 시작한 모듈에 하나입니다. 일반적으로 파이썬 프로그램의 각 스코프에 대해 하나의 프레임이 있습니다. 예를 들어, 각 모듈, 각 함수 호출, 그리고 각 클래스 정의는 프레임을 가집니다.

프레임은 _호출 스택(call stack)_에 존재하며, 이는 지금까지 논의해온 것과는 완전히 다른 스택입니다. (호출 스택은 여러분이 이미 가장 친숙한 스택입니다. 예외의 트레이스백에서 출력되는 것을 본 적이 있을 것입니다. "File 'program.py', line 10"으로 시작하는 트레이스백의 각 줄은 호출 스택의 하나의 프레임에 해당합니다.) 지금까지 살펴본 스택, 즉 인터프리터가 바이트코드를 실행하는 동안 조작하는 스택을 _데이터 스택(data stack)_이라고 하겠습니다. _블록 스택(block stack)_이라고 하는 세 번째 스택도 있습니다. 블록은 특정 종류의 제어 흐름, 특히 루핑과 예외 처리에 사용됩니다. 호출 스택의 각 프레임은 자체적으로 데이터 스택과 블록 스택을 가집니다.

예제로 이를 구체적으로 만들어보겠습니다. 파이썬 인터프리터가 현재 아래에서 3번으로 표시된 줄을 실행하고 있다고 가정해보세요. 인터프리터는 `foo` 호출의 중간에 있고, 이것이 다시 `bar`를 호출하고 있습니다. 다이어그램은 프레임의 호출 스택, 블록 스택, 그리고 데이터 스택의 도식적 모습을 보여줍니다. (이 코드는 REPL 세션처럼 작성되었으므로 먼저 필요한 함수들을 정의했습니다.) 우리가 관심 있는 순간에 인터프리터는 맨 아래에서 `foo()`를 실행하고 있으며, 이것이 `foo`의 본문에 도달한 다음 `bar`로 올라갑니다. \newpage

```python
>>> def bar(y):
...     z = y + 3     # <--- (3) ... and the interpreter is here.
...     return z
...
>>> def foo():
...     a = 1
...     b = 2
...     return a + bar(b) # <--- (2) ... which is returning a call to bar ...
...
>>> foo()             # <--- (1) We're in the middle of a call to foo ...
3
```

\aosafigure[240pt]{interpreter-images/interpreter-callstack.png}{호출 스택}{500l.interpreter.callstack}

이 시점에서 인터프리터는 `bar`에 대한 함수 호출 중간에 있습니다. 호출 스택에는 세 개의 프레임이 있습니다: 모듈 레벨에 하나, 함수 `foo`에 하나, 그리고 `bar`에 하나입니다 (\aosafigref{500l.interpreter.callstack}). `bar`가 리턴하면 그와 연결된 프레임이 호출 스택에서 팝되고 버려집니다.

바이트코드 명령어 `RETURN_VALUE`는 인터프리터에게 프레임 간에 값을 전달하라고 지시합니다. 먼저 호출 스택의 맨 위 프레임의 데이터 스택에서 맨 위 값을 팝합니다. 그런 다음 전체 프레임을 호출 스택에서 팝하고 버립니다. 마지막으로 그 값을 한 단계 아래의 프레임의 데이터 스택에 푸시합니다.

Ned Batchelder와 제가 Byterun을 작업할 때, 오랜 시간 동안 구현에 중대한 오류가 있었습니다. 각 프레임에 하나의 데이터 스택을 가지는 대신, 전체 가상 머신에 단 하나의 데이터 스택만 있었습니다. 우리는 수십 개의 작은 파이썬 코드 조각들로 구성된 테스트를 가지고 있었고, 이것들을 Byterun과 실제 파이썬 인터프리터에서 실행하여 두 인터프리터에서 동일한 일이 일어나는지 확인했습니다. 이 테스트들의 거의 모든 것이 통과하고 있었습니다. 작동시킬 수 없었던 유일한 것은 제너레이터였습니다. 마침내 CPython 코드를 더 주의 깊게 읽은 후, 우리는 실수를 깨달았습니다[^thanks]. 데이터 스택을 각 프레임으로 이동시킨 것이 문제를 해결했습니다.

[^thanks]: 이 버그에 대한 통찰력을 주신 Michael Arntzenius에게 감사드립니다. 

돌이켜보면 이 버그에서 파이썬의 얼마나 적은 부분이 각 프레임이 다른 데이터 스택을 가지는 데 의존하는지 놀라웠습니다. 파이썬 인터프리터의 거의 모든 연산이 데이터 스택을 주의 깊게 정리하므로, 프레임들이 동일한 스택을 공유한다는 사실은 문제가 되지 않았습니다. 위의 예제에서 `bar`가 실행을 마치면 데이터 스택을 비운 상태로 남겨둡니다. `foo`가 동일한 스택을 공유하더라도 값들은 더 아래에 있을 것입니다. 하지만 제너레이터의 경우, 핵심 기능은 프레임을 일시 중지하고, 다른 프레임으로 돌아간 다음, 나중에 제너레이터 프레임으로 다시 돌아와서 떠났던 것과 정확히 동일한 상태를 가지는 능력입니다. \newpage

## Byterun

이제 파이썬 인터프리터에 대한 충분한 맥락을 가지고 Byterun을 살펴보기 시작할 수 있습니다. 

Byterun에는 네 종류의 객체가 있습니다:

- `VirtualMachine` 클래스: 최고 수준의 구조, 특히 프레임의 호출 스택을 관리하고 명령어에서 연산으로의 매핑을 포함합니다. 이는 위의 `Intepreter` 객체의 더 복잡한 버전입니다.
- `Frame` 클래스: 모든 `Frame` 인스턴스는 하나의 코드 객체를 가지고 몇 가지 다른 필요한 상태 비트들, 특히 전역 및 지역 네임스페이스, 호출하는 프레임에 대한 참조, 그리고 마지막으로 실행된 바이트코드 명령어를 관리합니다.
- `Function` 클래스: 실제 파이썬 함수 대신 사용될 것입니다. 함수를 호출하는 것이 인터프리터에서 새로운 프레임을 생성한다는 것을 기억하세요. 우리는 새로운 프레임의 생성을 제어할 수 있도록 Function을 구현합니다.
- `Block` 클래스: 블록의 세 개 속성을 단순히 래핑합니다. (블록의 세부 사항들은 파이썬 인터프리터의 핵심이 아니므로 많은 시간을 할애하지는 않을 것이지만, Byterun이 실제 파이썬 코드를 실행할 수 있도록 여기에 포함되었습니다.)

### `VirtualMachine` 클래스

파이썬 인터프리터가 하나만 있기 때문에 프로그램이 실행될 때마다 `VirtualMachine`의 인스턴스는 하나만 생성됩니다. `VirtualMachine`은 호출 스택, 예외 상태, 그리고 프레임 간에 전달되는 동안의 반환 값들을 저장합니다. 코드를 실행하는 진입점은 컴파일된 코드 객체를 인수로 받는 `run_code` 메서드입니다. 이는 프레임을 설정하고 실행하는 것으로 시작합니다. 이 프레임은 다른 프레임들을 생성할 수 있습니다. 프로그램이 실행되면서 호출 스택은 커지고 줄어듭니다. 첫 번째 프레임이 결국 리턴하면 실행이 완료됩니다.

```python
class VirtualMachineError(Exception):
    pass

class VirtualMachine(object):
    def __init__(self):
        self.frames = []   # The call stack of frames.
        self.frame = None  # The current frame.
        self.return_value = None
        self.last_exception = None

    def run_code(self, code, global_names=None, local_names=None):
        """ 가상 머신을 사용하여 코드를 실행하는 진입점입니다."""
        frame = self.make_frame(code, global_names=global_names, 
                                local_names=local_names)
        self.run_frame(frame)

```

### `Frame` 클래스

다음으로 `Frame` 객체를 작성하겠습니다. 프레임은 메서드 없이 속성들의 모음입니다. 위에서 언급했듯이, 속성들에는 컴파일러가 생성한 코드 객체, 지역, 전역, 그리고 내장 네임스페이스, 이전 프레임에 대한 참조, 데이터 스택, 블록 스택, 그리고 마지막으로 실행된 명령어가 포함됩니다. (파이썬이 서로 다른 모듈에서 이 네임스페이스를 다르게 취급하기 때문에 내장 네임스페이스에 도달하기 위해 약간의 추가 작업을 해야 합니다. 이 세부 사항은 가상 머신에게는 중요하지 않습니다.)

```python
class Frame(object):
    def __init__(self, code_obj, global_names, local_names, prev_frame):
        self.code_obj = code_obj
        self.global_names = global_names
        self.local_names = local_names
        self.prev_frame = prev_frame
        self.stack = []
        if prev_frame:
            self.builtin_names = prev_frame.builtin_names
        else:
            self.builtin_names = local_names['__builtins__']
            if hasattr(self.builtin_names, '__dict__'):
                self.builtin_names = self.builtin_names.__dict__

        self.last_instruction = 0
        self.block_stack = []
```

다음으로 가상 머신에 프레임 조작을 추가하겠습니다. 프레임에 대한 세 개의 도우미 함수가 있습니다: 새로운 프레임을 생성하는 하나(새 프레임의 네임스페이스를 정리하는 역할), 그리고 프레임 스택에서 프레임을 푸시하고 팝하는 각각 하나씩입니다. 네 번째 함수인 `run_frame`은 프레임을 실행하는 주요 작업을 수행합니다. 곧 다시 돌아오겠습니다.

```python
class VirtualMachine(object):
    [... snip ...]

    # Frame manipulation
    def make_frame(self, code, callargs={}, global_names=None, local_names=None):
        if global_names is not None and local_names is not None:
            local_names = global_names
        elif self.frames:
            global_names = self.frame.global_names
            local_names = {}
        else:
            global_names = local_names = {
                '__builtins__': __builtins__,
                '__name__': '__main__',
                '__doc__': None,
                '__package__': None,
            }
        local_names.update(callargs)
        frame = Frame(code, global_names, local_names, self.frame)
        return frame

    def push_frame(self, frame):
        self.frames.append(frame)
        self.frame = frame

    def pop_frame(self):
        self.frames.pop()
        if self.frames:
            self.frame = self.frames[-1]
        else:
            self.frame = None

    def run_frame(self):
        pass
        # 곧 다시 돌아오겠습니다
```

### `Function` 클래스

`Function` 객체의 구현은 약간 복잡하며, 대부분의 세부 사항들은 인터프리터를 이해하는 데 중요하지 않습니다. 주목할 중요한 점은 함수를 호출하는 것, 즉 `__call__` 메서드를 호출하는 것이 새로운 `Frame` 객체를 생성하고 실행을 시작한다는 것입니다.

```python
class Function(object):
    """
    인터프리터가 기대하는 것들을 정의하여 현실적인 함수 객체를 생성합니다.
    """
    __slots__ = [
        'func_code', 'func_name', 'func_defaults', 'func_globals',
        'func_locals', 'func_dict', 'func_closure',
        '__name__', '__dict__', '__doc__',
        '_vm', '_func',
    ]

    def __init__(self, name, code, globs, defaults, closure, vm):
        """인터프리터를 이해하는 데 이것을 자세히 따라갈 필요는 없습니다."""
        self._vm = vm
        self.func_code = code
        self.func_name = self.__name__ = name or code.co_name
        self.func_defaults = tuple(defaults)
        self.func_globals = globs
        self.func_locals = self._vm.frame.f_locals
        self.__dict__ = {}
        self.func_closure = closure
        self.__doc__ = code.co_consts[0] if code.co_consts else None

        # 때로는 실제 파이썬 함수가 필요합니다. 이는 그를 위한 것입니다.
        kw = {
            'argdefs': self.func_defaults,
        }
        if closure:
            kw['closure'] = tuple(make_cell(0) for _ in closure)
        self._func = types.FunctionType(code, globs, **kw)

    def __call__(self, *args, **kwargs):
        """함수를 호출할 때 새로운 프레임을 만들고 실행합니다."""
        callargs = inspect.getcallargs(self._func, *args, **kwargs)
        # callargs를 사용하여 새 프레임에 전달할 인수: 값 매핑을 제공합니다.
        frame = self._vm.make_frame(
            self.func_code, callargs, self.func_globals, {}
        )
        return self._vm.run_frame(frame)

def make_cell(value):
    """실제 파이썬 클로저를 생성하고 셀을 가져옵니다."""
    # 이 복잡한 부분에 대한 도움을 주신 Alex Gaynor에게 감사드립니다.
    fn = (lambda x: lambda: x)(value)
    return fn.__closure__[0]
```

다음으로 `VirtualMachine` 객체로 돌아가서 데이터 스택 조작을 위한 몇 개의 도우미 메서드를 추가하겠습니다. 스택을 조작하는 바이트코드들은 항상 현재 프레임의 데이터 스택에서 작동합니다. 이는 `POP_TOP`, `LOAD_FAST`, 그리고 스택을 다루는 모든 다른 명령어들의 구현을 더 읽기 쉬만들 것입니다.

```python
class VirtualMachine(object):
    [... snip ...]

    # Data stack manipulation
    def top(self):
        return self.frame.stack[-1]

    def pop(self):
        return self.frame.stack.pop()

    def push(self, *vals):
        self.frame.stack.extend(vals)

    def popn(self, n):
        """값 스택에서 여러 개의 값을 팝합니다.
        가장 깊은 값이 먼저 오는 `n`개 값들의 리스트가 반환됩니다.
        """
        if n:
            ret = self.frame.stack[-n:]
            self.frame.stack[-n:] = []
            return ret
        else:
            return []
```

프레임을 실행하기 전에 두 개의 메서드가 더 필요합니다. 

첫 번째인 `parse_byte_and_args`는 바이트코드를 받아서 인수가 있는지 확인하고, 있다면 인수를 파싱합니다. 이 메서드는 마지막으로 실행된 명령어에 대한 참조인 프레임의 `last_instruction` 속성도 업데이트합니다. 하나의 명령어는 인수가 없으면 1바이트 길이이고, 인수가 있으면 3바이트 길이입니다. 마지막 두 바이트가 인수입니다. 각 명령어의 인수의 의미는 어떤 명령어인지에 따라 달라집니다. 예를 들어, 위에서 언급했듯이 `POP_JUMP_IF_FALSE`의 경우 명령어의 인수는 점프 대상입니다. `BUILD_LIST`의 경우는 리스트의 요소 수입니다. `LOAD_CONST`의 경우는 상수 목록의 인덱스입니다.

일부 명령어들은 인수로 단순한 숫자를 사용합니다. 다른 명령어들의 경우 가상 머신이 인수가 무엇을 의미하는지 알아내기 위해 약간의 작업을 해야 합니다. 표준 라이브러리의 `dis` 모듈은 어떤 인수가 어떤 의미를 가지는지 설명하는 컴닝시트를 제공하여 우리의 코드를 더 간결하게 만듭니다. 예를 들어, `dis.hasname` 목록은 `LOAD_NAME`, `IMPORT_NAME`, `LOAD_GLOBAL`, 그리고 아홉 개의 다른 명령어들의 인수가 동일한 의미를 가진다고 알려줍니다: 이러한 명령어들의 경우 인수는 코드 객체의 이름 목록에 대한 인덱스를 나타냅니다.

```python
class VirtualMachine(object):
    [... snip ...]

    def parse_byte_and_args(self):
        f = self.frame
        opoffset = f.last_instruction
        byteCode = f.code_obj.co_code[opoffset]
        f.last_instruction += 1
        byte_name = dis.opname[byteCode]
        if byteCode >= dis.HAVE_ARGUMENT:
            # index into the bytecode
            arg = f.code_obj.co_code[f.last_instruction:f.last_instruction+2]  
            f.last_instruction += 2   # advance the instruction pointer
            arg_val = arg[0] + (arg[1] * 256)
            if byteCode in dis.hasconst:   # Look up a constant
                arg = f.code_obj.co_consts[arg_val]
            elif byteCode in dis.hasname:  # Look up a name
                arg = f.code_obj.co_names[arg_val]
            elif byteCode in dis.haslocal: # Look up a local name
                arg = f.code_obj.co_varnames[arg_val]
            elif byteCode in dis.hasjrel:  # Calculate a relative jump
                arg = f.last_instruction + arg_val
            else:
                arg = arg_val
            argument = [arg]
        else:
            argument = []

        return byte_name, argument
```

다음 메서드는 `dispatch`로, 주어진 명령어에 대한 연산을 찾아서 실행합니다. CPython 인터프리터에서는 이 디스패치가 1,500줄에 걸친 거대한 switch 문으로 수행됩니다! 다행히 우리는 파이썬을 작성하고 있으므로 더 간결할 수 있습니다. 각 바이트 이름에 대한 메서드를 정의한 다음 `getattr`을 사용하여 찾아보겠습니다. 위의 장난감 인터프리터에서처럼 명령어가 `FOO_BAR`라는 이름이라면, 해당 메서드는 `byte_FOO_BAR`라는 이름이 될 것입니다. 지금은 이러한 메서드들의 내용을 블랙박스로 남겨두겠습니다. 각 바이트코드 메서드는 `None` 또는 `why`라고 불리는 문자열을 반환할 것입니다. 이는 경우에 따라 인터프리터가 필요로 하는 추가적인 상태 조각입니다. 개별 명령어 메서드들의 이러한 반환값들은 인터프리터 상태의 내부 지시자로만 사용됩니다. 프레임 실행으로부터의 반환값과 혼동하지 마세요.


```python
class VirtualMachine(object):
    [... snip ...]

    def dispatch(self, byte_name, argument):
        """ 바이트 이름으로 해당 메서드에 디스패치합니다.
        예외는 캐치되어 가상 머신에 설정됩니다."""

        # 나중에 블록 스택을 풀 때,
        # 왜 그렇게 하는지 추적해야 합니다.
        why = None
        try:
            bytecode_fn = getattr(self, 'byte_%s' % byte_name, None)
            if bytecode_fn is None:
                if byte_name.startswith('UNARY_'):
                    self.unaryOperator(byte_name[6:])
                elif byte_name.startswith('BINARY_'):
                    self.binaryOperator(byte_name[7:])
                else:
                    raise VirtualMachineError(
                        "unsupported bytecode type: %s" % byte_name
                    )
            else:
                why = bytecode_fn(*argument)
        except:
            # 연산을 실행하는 동안 발생한 예외를 처리합니다.
            self.last_exception = sys.exc_info()[:2] + (None,)
            why = 'exception'

        return why

    def run_frame(self, frame):
        """Run a frame until it returns (somehow).
        Exceptions are raised, the return value is returned.
        """
        self.push_frame(frame)
        while True:
            byte_name, arguments = self.parse_byte_and_args()

            why = self.dispatch(byte_name, arguments)

            # Deal with any block management we need to do
            while why and frame.block_stack:
                why = self.manage_block_stack(why)

            if why:
                break

        self.pop_frame()

        if why == 'exception':
            exc, val, tb = self.last_exception
            e = exc(val)
            e.__traceback__ = tb
            raise e

        return self.return_value
```

### The `Block` Class

각 바이트코드 명령어에 대한 메서드를 구현하기 전에 블록에 대해 간략히 논의하겠습니다. 블록은 특정 종류의 제어 흐름, 특히 예외 처리와 루핑에 사용됩니다. 블록은 연산이 완료될 때 데이터 스택이 적절한 상태에 있도록 하는 역할을 담당합니다. 예를 들어, 루프에서는 특별한 반복자 객체가 루프가 실행되는 동안 스택에 남아 있지만, 루프가 완료되면 팝됩니다. 인터프리터는 루프가 계속되고 있는지 아니면 완료되었는지 추적해야 합니다.

이 추가 정보를 추적하기 위해 인터프리터는 상태를 나타내는 플래그를 설정합니다. 우리는 이 플래그를 `why`라는 변수로 구현하며, 이는 `None`이거나 `"continue"`, `"break"`, `"exception"`, 또는 `"return"` 문자열 중 하나일 수 있습니다. 이는 블록 스택과 데이터 스택에 어떤 종류의 조작이 일어나야 하는지를 나타냅니다. 반복자 예제로 돌아가면, 블록 스택의 맨 위가 `loop` 블록이고 `why` 코드가 `continue`라면 반복자 객체는 데이터 스택에 남아 있어야 하지만, `why` 코드가 `break`라면 팝되어야 합니다.

블록 조작의 정확한 세부 사항들은 상당히 까다롭고, 이에 대해 더 시간을 할애하지는 않을 것이지만, 관심 있는 독자들은 주의 깊게 살펴보시기 바랍니다.

```python
Block = collections.namedtuple("Block", "type, handler, stack_height")

class VirtualMachine(object):
    [... snip ...]

    # Block stack manipulation
    def push_block(self, b_type, handler=None):
        stack_height = len(self.frame.stack)
        self.frame.block_stack.append(Block(b_type, handler, stack_height))

    def pop_block(self):
        return self.frame.block_stack.pop()

    def unwind_block(self, block):
        """Unwind the values on the data stack corresponding to a given block."""
        if block.type == 'except-handler':
            # The exception itself is on the stack as type, value, and traceback.
            offset = 3  
        else:
            offset = 0

        while len(self.frame.stack) > block.level + offset:
            self.pop()

        if block.type == 'except-handler':
            traceback, value, exctype = self.popn(3)
            self.last_exception = exctype, value, traceback

    def manage_block_stack(self, why):
        """ """
        frame = self.frame
        block = frame.block_stack[-1]
        if block.type == 'loop' and why == 'continue':
            self.jump(self.return_value)
            why = None
            return why

        self.pop_block()
        self.unwind_block(block)

        if block.type == 'loop' and why == 'break':
            why = None
            self.jump(block.handler)
            return why

        if (block.type in ['setup-except', 'finally'] and why == 'exception'):
            self.push_block('except-handler')
            exctype, value, tb = self.last_exception
            self.push(tb, value, exctype)
            self.push(tb, value, exctype) # yes, twice
            why = None
            self.jump(block.handler)
            return why

        elif block.type == 'finally':
            if why in ('return', 'continue'):
                self.push(self.return_value)

            self.push(why)

            why = None
            self.jump(block.handler)
            return why
        return why
```

## The Instructions

남은 것은 명령어들을 위한 수십 개의 메서드를 구현하는 것입니다. 실제 명령어들은 인터프리터의 가장 흥미롭지 않은 부분이므로, 여기서는 몇 개만 보여주지만 전체 구현은 [GitHub에서 이용 가능합니다](https://github.com/nedbat/byterun). (위에서 디스어셈블한 모든 코드 샘플을 실행하기에 충분한 명령어들이 여기에 포함되어 있습니다.)

```python
class VirtualMachine(object):
    [... snip ...]

    ## Stack manipulation

    def byte_LOAD_CONST(self, const):
        self.push(const)

    def byte_POP_TOP(self):
        self.pop()

    ## Names
    def byte_LOAD_NAME(self, name):
        frame = self.frame
        if name in frame.f_locals:
            val = frame.f_locals[name]
        elif name in frame.f_globals:
            val = frame.f_globals[name]
        elif name in frame.f_builtins:
            val = frame.f_builtins[name]
        else:
            raise NameError("name '%s' is not defined" % name)
        self.push(val)

    def byte_STORE_NAME(self, name):
        self.frame.f_locals[name] = self.pop()

    def byte_LOAD_FAST(self, name):
        if name in self.frame.f_locals:
            val = self.frame.f_locals[name]
        else:
            raise UnboundLocalError(
                "local variable '%s' referenced before assignment" % name
            )
        self.push(val)

    def byte_STORE_FAST(self, name):
        self.frame.f_locals[name] = self.pop()

    def byte_LOAD_GLOBAL(self, name):
        f = self.frame
        if name in f.f_globals:
            val = f.f_globals[name]
        elif name in f.f_builtins:
            val = f.f_builtins[name]
        else:
            raise NameError("global name '%s' is not defined" % name)
        self.push(val)

    ## Operators

    BINARY_OPERATORS = {
        'POWER':    pow,
        'MULTIPLY': operator.mul,
        'FLOOR_DIVIDE': operator.floordiv,
        'TRUE_DIVIDE':  operator.truediv,
        'MODULO':   operator.mod,
        'ADD':      operator.add,
        'SUBTRACT': operator.sub,
        'SUBSCR':   operator.getitem,
        'LSHIFT':   operator.lshift,
        'RSHIFT':   operator.rshift,
        'AND':      operator.and_,
        'XOR':      operator.xor,
        'OR':       operator.or_,
    }

    def binaryOperator(self, op):
        x, y = self.popn(2)
        self.push(self.BINARY_OPERATORS[op](x, y))

    COMPARE_OPERATORS = [
        operator.lt,
        operator.le,
        operator.eq,
        operator.ne,
        operator.gt,
        operator.ge,
        lambda x, y: x in y,
        lambda x, y: x not in y,
        lambda x, y: x is y,
        lambda x, y: x is not y,
        lambda x, y: issubclass(x, Exception) and issubclass(x, y),
    ]

    def byte_COMPARE_OP(self, opnum):
        x, y = self.popn(2)
        self.push(self.COMPARE_OPERATORS[opnum](x, y))

    ## Attributes and indexing

    def byte_LOAD_ATTR(self, attr):
        obj = self.pop()
        val = getattr(obj, attr)
        self.push(val)

    def byte_STORE_ATTR(self, name):
        val, obj = self.popn(2)
        setattr(obj, name, val)

    ## Building

    def byte_BUILD_LIST(self, count):
        elts = self.popn(count)
        self.push(elts)

    def byte_BUILD_MAP(self, size):
        self.push({})

    def byte_STORE_MAP(self):
        the_map, val, key = self.popn(3)
        the_map[key] = val
        self.push(the_map)

    def byte_LIST_APPEND(self, count):
        val = self.pop()
        the_list = self.frame.stack[-count] # peek
        the_list.append(val)

    ## Jumps

    def byte_JUMP_FORWARD(self, jump):
        self.jump(jump)

    def byte_JUMP_ABSOLUTE(self, jump):
        self.jump(jump)

    def byte_POP_JUMP_IF_TRUE(self, jump):
        val = self.pop()
        if val:
            self.jump(jump)

    def byte_POP_JUMP_IF_FALSE(self, jump):
        val = self.pop()
        if not val:
            self.jump(jump)

    ## Blocks

    def byte_SETUP_LOOP(self, dest):
        self.push_block('loop', dest)

    def byte_GET_ITER(self):
        self.push(iter(self.pop()))

    def byte_FOR_ITER(self, jump):
        iterobj = self.top()
        try:
            v = next(iterobj)
            self.push(v)
        except StopIteration:
            self.pop()
            self.jump(jump)

    def byte_BREAK_LOOP(self):
        return 'break'

    def byte_POP_BLOCK(self):
        self.pop_block()

    ## Functions

    def byte_MAKE_FUNCTION(self, argc):
        name = self.pop()
        code = self.pop()
        defaults = self.popn(argc)
        globs = self.frame.f_globals
        fn = Function(name, code, globs, defaults, None, self)
        self.push(fn)

    def byte_CALL_FUNCTION(self, arg):
        lenKw, lenPos = divmod(arg, 256) # KWargs not supported here
        posargs = self.popn(lenPos)

        func = self.pop()
        frame = self.frame
        retval = func(*posargs)
        self.push(retval)

    def byte_RETURN_VALUE(self):
        self.return_value = self.pop()
        return "return"
```

## Dynamic Typing: What the Compiler Doesn't Know

아마 들어본 적이 있는 한 가지는 파이썬이 "동적" 언어, 특히 "동적으로 타입이 지정되는" 언어라는 것입니다. 지금까지 우리가 한 작업은 이 설명에 어느 정도 빛을 비추어 줍니다.

이 맥락에서 "동적"이 의미하는 것 중 하나는 많은 작업이 런타임에 수행된다는 것입니다. 우리는 앞서 파이썬 컴파일러가 코드가 실제로 무엇을 하는지에 대한 정보를 많이 가지고 있지 않다는 것을 보았습니다. 예를 들어, 아래의 짧은 함수 `mod`를 고려해보세요. `mod`는 두 개의 인수를 받아서 첫 번째를 두 번째로 나눠 나머지를 반환합니다. 바이트코드에서 변수 `a`와 `b`가 로드되고, 그 다음 바이트코드 `BINARY_MODULO`가 모듈로 연산 자체를 수행하는 것을 볼 수 있습니다.

```python
>>> def mod(a, b):
...    return a % b
>>> dis.dis(mod)
  2           0 LOAD_FAST                0 (a)
              3 LOAD_FAST                1 (b)
              6 BINARY_MODULO
              7 RETURN_VALUE
>>> mod(19, 5)
4
```

19 `%` 5를 계산하면 4가 나옵니다. 놓라울 것도 없습니다. 다른 인수로 호출하면 어떻게 될까요?

```python
>>> mod("by%sde", "teco")
'bytecode'
```

What just happened? You've probably seen this syntax before, but in a different context: 

```
>>> print("by%sde" % "teco")
bytecode
```

문자열을 출력용으로 포맷하기 위해 `%` 기호를 사용하는 것은 `BINARY_MODULO` 명령어를 호출하는 것을 의미합니다. 이 명령어는 명령어가 실행될 때 스택의 맨 위 두 값을 모듈로 연산합니다. 그것들이 문자열이든, 정수이든, 아니면 여러분이 직접 정의한 클래스의 인스턴스이든 상관없이 말입니다. 바이트코드는 함수가 컴파일될 때 (사실상 정의될 때) 생성되었고, 다른 타입의 인수들과 함께 동일한 바이트코드가 사용됩니다.

파이썬 컴파일러는 바이트코드가 가질 효과에 대해 비교적 적게 앍니다. `BINARY_MODULO`가 작동하는 객체의 타입을 결정하고 그 타입에 맞는 올바른 일을 하는 것은 인터프리터의 몰입니다. 이것이 파이썬이 _동적으로 타입이 지정되는_ 언어로 설명되는 이유입니다: 실제로 실행할 때까지는 이 함수의 인수들의 타입을 모릅니다. 대조적으로 정적으로 타입이 지정되는 언어에서는 프로그래머가 컴파일러에게 인수들이 어떤 타입이 될 것인지 미리 알려주거나 (또는 컴파일러가 스스로 그것들을 알아냅니다).

The compiler's ignorance is one of the challenges to optimizing Python or analyzing it statically&mdash;just looking at the bytecode, without actually running the code, you don't know what each instruction will do! In fact, you could define a class that implements the `__mod__` method, and Python would invoke that method if you use `%` on your objects. So `BINARY_MODULO` could run any code at all!

다음 코드를 보면 `a % b`의 첫 번째 계산이 낭비적으로 보입니다.

```python
def mod(a,b):
    a % b
    return a %b
```

Unfortunately, a static analysis of this code&mdash;the kind of you can do without running it&mdash;can't be certain that the first `a % b` really does nothing. Calling `__mod__` with `%` might write to a file, or interact with another part of your program, or do literally anything else that's possible in Python. It's hard to optimize a function when you don't know what it does! In Russell Power and Alex Rubinsteyn's great paper "How fast can we make interpreted Python?", they note, "In the general absence of type information, each instruction must be treated as `INVOKE_ARBITRARY_METHOD`."

## 결론

Byterun은 CPython보다 이해하기 쉬운 간결한 파이썬 인터프리터입니다. Byterun은 CPython의 주요 구조적 세부 사항들을 복제합니다: 바이트코드라고 불리는 명령어 집합에서 작동하는 스택 기반 인터프리터입니다. 이러한 명령어들을 단계별로 실행하거나 점프하며, 데이터 스택에 푸시하고 팝합니다. 인터프리터는 함수와 제너레이터를 호출하고 리턴하면서 프레임들을 생성하고, 소멸하고, 그 사이를 점프합니다. Byterun은 실제 인터프리터의 한계도 공유합니다: 파이썬이 동적 타이핑을 사용하기 때문에 인터프리터는 런타임에 프로그램의 올바른 동작을 결정하기 위해 열심히 작업해야 합니다.

I encourage you to disassemble your own programs and to run them using Byterun. You'll quickly run into instructions that this shorter version of Byterun doesn't implement. The full implementation can be found at https://github.com/nedbat/byterun&mdash;or, by carefully reading the real CPython interpreter's `ceval.c`, you can implement it yourself!

## 감사인사

이 프로젝트를 시작하고 제 기여를 안내해주신 Ned Batchelder, 코드 디버깅과 산문 편집에 도움을 주신 Michael Arntzenius, 편집에 도움을 주신 Leta Montopoli, 그리고 지원과 관심을 보여주신 전체 Recurse Center 커뮤니티에 감사드립니다. 모든 오류는 제 것입니다.
