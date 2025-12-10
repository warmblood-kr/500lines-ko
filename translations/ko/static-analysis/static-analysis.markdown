title: 정적 분석
author: Leah Hanson
<markdown>
_Leah Hanson은 Hacker School의 자랑스러운 졸업생으로 사람들이 Julia를 배우는 것을 도와주는 일을 좋아합니다. 그녀는 [http://blog.leahhanson.us/](http://blog.leahhanson.us/)에서 블로그를 작성하고 [\@astrieanna](https://twitter.com/astrieanna)에서 트윗을 합니다._
</markdown>
## 개요

컴파일되지 않는 코드 부분에 빨간색 밑줄을 그어주는 고급 IDE를 사용해본 적이 있을 것입니다. 코드의 형식이나 스타일 문제를 확인하기 위해 린터(linter)를 실행해본 적도 있을 것입니다. 모든 경고를 켜놓고 매우 까다로운 모드로 컴파일러를 실행해본 적도 있을지 모릅니다. 이 모든 도구들은 정적 분석의 응용 사례들입니다.

정적 분석은 코드를 실행하지 않고도 문제를 검사하는 방법입니다. "정적(Static)"이라는 것은 실행 시간(runtime)이 아닌 컴파일 시간(compile time)을 의미하고, "분석(analysis)"이라는 것은 우리가 코드를 분석한다는 뜻입니다. 위에서 언급한 도구들을 사용했을 때 마법 같다고 느꼈을지도 모릅니다. 하지만 그런 도구들도 단순한 프로그램입니다&mdash;여러분과 같은 프로그래머가 작성한 소스 코드로 만들어진 것입니다. 이 장에서는 몇 가지 정적 분석 검사를 구현하는 방법에 대해 이야기하겠습니다. 이를 위해서는 검사가 무엇을 해야 하는지, 그리고 어떻게 해야 하는지를 알아야 합니다.

알아야 할 내용을 세 단계로 나누어 더 구체적으로 설명할 수 있습니다:

#### 1. 무엇을 검사할지 결정하기

프로그래밍 언어 사용자가 인식할 수 있는 용어로 해결하고자 하는 일반적인 문제를 설명할 수 있어야 합니다. 예를 들면:

- 철자가 틀린 변수명 찾기
- 병렬 코드에서 경쟁 상태(race condition) 찾기
- 구현되지 않은 함수 호출 찾기

#### 2. 정확히 어떻게 검사할지 결정하기

위에 나열한 작업들 중 하나를 친구에게 부탁할 수는 있지만, 컴퓨터에게 설명하기에는 충분히 구체적이지 않습니다. 예를 들어 "철자가 틀린 변수명"을 다루려면, 여기서 철자가 틀렸다는 것이 무엇을 의미하는지 결정해야 합니다. 한 가지 방법은 변수명이 사전에 있는 영어 단어들로 구성되어야 한다고 주장하는 것이고, 다른 방법은 한 번만 사용되는 변수(철자를 잘못 입력한 그 한 번)를 찾는 것입니다.

한 번만 사용되는 변수를 찾는다면, 변수 사용의 종류(값이 할당되는 것과 읽히는 것)와 어떤 코드가 경고를 발생시키거나 발생시키지 않을지에 대해 이야기할 수 있습니다.

#### 3. 구현 세부사항

이것은 실제로 코드를 작성하는 행위, 사용하는 라이브러리의 문서를 읽는 데 소요되는 시간, 그리고 분석을 작성하는 데 필요한 정보에 접근하는 방법을 알아내는 것을 다룹니다. 이는 코드 파일을 읽어들이고, 구조를 이해하기 위해 파싱하고, 그 구조에 대해 특정 검사를 수행하는 것을 포함할 수 있습니다.

이 장에서 구현된 개별 검사들 각각에 대해 이러한 단계들을 거쳐보겠습니다. 1단계는 분석하는 언어를 충분히 이해해서 그 언어 사용자들이 직면하는 문제들에 공감할 수 있어야 합니다. 이 장의 모든 코드는 Julia 코드를 분석하기 위해 작성된 Julia 코드입니다.

## Julia에 대한 매우 간단한 소개

Julia는 기술 컴퓨팅을 목표로 하는 젊은 언어입니다. 2012년 봄에 버전 0.1로 출시되었으며, 2015년 초 현재 버전 0.3에 도달했습니다. 일반적으로 Julia는 Python과 매우 비슷해 보이지만, 선택적인 타입 주석(type annotation)이 있고 객체 지향적인 요소는 없습니다. 대부분의 프로그래머가 Julia에서 새롭다고 느낄 기능은 다중 디스패치(multiple dispatch)인데, 이는 API 설계와 언어의 다른 설계 선택에 광범위한 영향을 미칩니다.

다음은 Julia 코드 예제입니다:

```julia
# A comment about increment
function increment(x::Int64)
  return x + 1
end

increment(5)
```

이 코드는 `x`라는 이름의 `Int64` 타입 인수 하나를 받는 `increment` 함수의 메서드를 정의합니다. 이 메서드는 `x + 1`의 값을 반환합니다. 그런 다음 새로 정의된 이 메서드를 값 `5`로 호출하며, 짐작하실 수 있듯이 이 함수 호출은 `6`으로 계산됩니다.

`Int64`는 메모리에서 64비트로 표현되는 부호 있는 정수를 값으로 하는 타입입니다. 컴퓨터에 64비트 프로세서가 있다면 하드웨어가 이해하는 정수들입니다. Julia의 타입은 메서드 디스패치에 영향을 주는 것 외에도 메모리에서 데이터의 표현을 정의합니다.

`increment`라는 이름은 많은 메서드를 가질 수 있는 제네릭 함수(generic function)를 참조합니다. 우리는 방금 그 중 하나의 메서드를 정의했습니다. 많은 언어에서 "함수(function)"와 "메서드(method)"라는 용어가 서로 바꿔서 사용되지만, Julia에서는 구별되는 의미를 갖습니다. "함수"를 메서드들의 이름 있는 집합으로, "메서드"를 특정 타입 시그니처에 대한 구체적인 구현으로 이해한다면 이 장을 더 잘 이해할 수 있을 것입니다.

`increment` 함수의 다른 메서드를 정의해보겠습니다:

```julia
# Increment x by y
function increment(x::Int64, y::Number)
  return x + y
end

increment(5) # => 6
increment(5,4) # => 9
```

이제 `increment` 함수는 두 개의 메서드를 갖게 되었습니다. Julia는 인수의 개수와 타입에 기반해서 주어진 호출에 대해 어떤 메서드를 실행할지 결정합니다. 이를 *동적 다중 디스패치(dynamic multiple dispatch)*라고 합니다:

- **동적(Dynamic)**: 런타임에 사용되는 값들의 타입에 기반하기 때문입니다.
- **다중(Multiple)**: 모든 인수들의 타입과 순서를 살펴보기 때문입니다.
- **디스패치(Dispatch)**: 이것이 함수 호출을 메서드 정의에 매칭시키는 방법이기 때문입니다.

이미 알고 있을 수 있는 언어들의 맥락에서 설명하자면, 객체 지향 언어들은 첫 번째 인수만 고려하므로 단일 디스패치(single dispatch)를 사용합니다. (`x.foo(y)`에서 첫 번째 인수는 `x`입니다.)

단일 디스패치와 다중 디스패치 모두 인수들의 타입에 기반합니다. 위의 `x::Int64`는 순전히 디스패치를 위한 타입 주석입니다. Julia의 동적 타입 시스템에서는 오류 없이 함수 안에서 `x`에 어떤 타입의 값이든 할당할 수 있습니다.

우리는 아직 "다중" 부분을 실제로 보지 못했지만, Julia에 대해 궁금하다면 스스로 찾아봐야 합니다. 우리는 첫 번째 검사로 넘어가야 합니다.

## 루프에서 변수 타입 검사하기

대부분의 프로그래밍 언어와 마찬가지로 Julia에서 매우 빠른 코드를 작성하려면 컴퓨터가 어떻게 작동하는지와 Julia가 어떻게 작동하는지를 이해해야 합니다. 컴파일러가 빠른 코드를 생성할 수 있도록 돕는 중요한 부분은 타입 안정적인(type-stable) 코드를 작성하는 것입니다. 이는 Julia와 JavaScript에서 중요하며, 다른 JIT 언어들에서도 도움이 됩니다. 컴파일러가 코드의 한 부분에서 변수가 항상 동일한 특정 타입을 포함할 것이라는 것을 알 수 있을 때, 컴파일러는 (올바르든 아니든) 그 변수에 대해 여러 가능한 타입이 있다고 믿는 경우보다 더 많은 최적화를 수행할 수 있습니다. 타입 안정성("단형성(monomorphism)"이라고도 함)이 JavaScript에서 왜 중요한지에 대해서는 [온라인](http://mrale.ph/blog/2015/01/11/whats-up-with-monomorphism.html)에서 더 읽을 수 있습니다.

### 이것이 왜 중요한가

`Int64`를 받아서 일정 양만큼 증가시키는 함수를 작성해보겠습니다. 숫자가 작으면(10 미만) 큰 숫자(50)만큼 증가시키고, 크면 0.5만큼만 증가시켜보겠습니다.

```julia
function increment(x::Int64)
  if x < 10
    x = x + 50
  else
    x = x + 0.5
  end
  return x
end
```

이 함수는 매우 간단해 보이지만, `x`의 타입이 불안정합니다. 두 개의 숫자를 선택했는데: 50은 `Int64`이고, 0.5는 `Float64`입니다. `x`의 값에 따라 이 둘 중 하나가 더해질 수 있습니다. 22와 같은 `Int64`를 0.5와 같은 `Float64`에 더하면 `Float64`(22.5)를 얻게 됩니다. 함수 내의 변수(`x`)의 타입이 함수에 대한 인수(`x`)의 값에 따라 변할 수 있기 때문에, 이 `increment` 메서드, 특히 변수 `x`는 타입 불안정입니다.

`Float64`는 64비트로 저장된 부동 소수점 값을 나타내는 타입입니다. C에서는 `double`이라고 불립니다. 이는 64비트 프로세서가 이해하는 부동 소수점 타입 중 하나입니다.

대부분의 효율성 문제와 마찬가지로, 이 문제는 루프 중에 발생할 때 더욱 두드러집니다. for 루프와 while 루프 안의 코드는 수없이 많이 실행되므로, 한 번 또는 두 번만 실행되는 코드를 빠르게 하는 것보다 그런 코드를 빠르게 만드는 것이 더 중요합니다. 따라서 우리의 첫 번째 검사는 루프 내부에서 불안정한 타입을 가지는 변수를 찾는 것입니다.

먼저 우리가 포착하고자 하는 것의 예제를 살펴보겠습니다. 두 개의 함수를 보겠습니다. 각각은 1부터 100까지의 수를 합하지만, 정수를 합하는 대신 각각을 2로 나눈 후 합합니다. 두 함수 모두 같은 답(2525.0)을 얻을 것이고, 둘 다 같은 타입(`Float64`)을 반환할 것입니다. 하지만 첫 번째 함수인 `unstable`은 타입 불안정성에 시달리지만, 두 번째 함수인 `stable`은 그렇지 않습니다.

```julia
function unstable()
  sum = 0
  for i=1:100
    sum += i/2
  end
  return sum
end
```

```julia
function stable()
  sum = 0.0
  for i=1:100
    sum += i/2
  end
  return sum
end
```

두 함수 간의 유일한 텍스트 차이는 `sum`의 초기화에 있습니다: `sum = 0` 대 `sum = 0.0`. Julia에서 `0`은 `Int64` 리터럴이고 `0.0`은 `Float64` 리터럴입니다. 이 작은 변화가 얼마나 큰 차이를 만들 수 있을까요?

Julia는 Just-In-Time(JIT) 컴파일되므로, 함수의 첫 실행은 이후 실행보다 더 오래 걸립니다. (첫 실행은 이러한 인수 타입들에 대해 함수를 컴파일하는 데 걸리는 시간을 포함합니다.) 함수들을 벤치마크할 때는 타이밍하기 전에 한 번 실행하거나 미리 컴파일해야 합니다.

```julia
julia> unstable()
2525.0

julia> stable()
2525.0

julia> @time unstable()
elapsed time: 9.517e-6 seconds (3248 bytes allocated)
2525.0

julia> @time stable()
elapsed time: 2.285e-6 seconds (64 bytes allocated)
2525.0
```

`@time` 매크로는 함수가 실행되는 데 걸린 시간과 실행 중에 할당된 바이트 수를 출력합니다. 할당된 바이트 수는 새로운 메모리가 필요할 때마다 증가하며, 가비지 컬렉터가 더 이상 사용되지 않는 메모리를 정리할 때 감소하지 않습니다. 이는 할당된 바이트가 메모리를 할당하고 관리하는 데 소요하는 시간과 관련이 있지만, 동시에 그 모든 메모리를 사용하고 있었다는 것을 의미하지는 않습니다.

`stable`과 `unstable`에 대한 확실한 수치를 얻으려면 루프를 훨씬 길게 만들거나 함수를 여러 번 실행해야 합니다. 하지만 `unstable`이 아마 더 느린 것 같습니다. 더 흥미롭게도, 할당된 바이트 수에서 큰 격차를 볼 수 있습니다. `unstable`은 약 3KB의 메모리를 할당한 반면 `stable`은 64바이트를 사용합니다.

`unstable`이 얼마나 간단한지 볼 수 있으므로, 이 할당이 루프에서 발생하고 있다고 추측할 수 있습니다. 이를 테스트하기 위해 루프를 더 길게 만들어서 할당이 그에 따라 증가하는지 볼 수 있습니다. 루프를 1부터 10000까지 돌리면 100배 더 많은 반복이 됩니다. 할당된 바이트 수도 약 100배 증가하여 약 300KB가 되는 것을 볼 것입니다.

```julia
function unstable()
  sum = 0
  for i=1:10000
    sum += i/2
  end
  return sum
end
```

함수를 재정의했으므로, 측정하기 전에 컴파일되도록 실행해야 합니다. 이제 더 많은 숫자를 합하므로 새 함수 정의에서 다르고 더 큰 답을 얻을 것으로 예상합니다.

```julia
julia> unstable()
2.50025e7

julia>@time unstable()
elapsed time: 0.000667613 seconds (320048 bytes allocated)
2.50025e7
```

새로운 `unstable`은 약 320KB를 할당했으며, 이는 할당이 루프에서 발생한다면 예상할 수 있는 것입니다. 여기서 무슨 일이 일어나고 있는지 설명하기 위해 Julia가 내부적으로 어떻게 작동하는지 살펴보겠습니다.

`unstable`과 `stable` 사이의 이런 차이는 `unstable`의 `sum`은 박싱(boxed)되어야 하지만 `stable`의 `sum`은 언박싱(unboxed)될 수 있기 때문에 발생합니다. 박싱된 값은 타입 태그와 값을 나타내는 실제 비트로 구성됩니다. 언박싱된 값은 실제 비트만 갖습니다. 하지만 타입 태그는 작으므로, 박싱된 값이 훨씬 더 많은 메모리를 할당하는 이유가 아닙니다.

차이는 컴파일러가 수행할 수 있는 최적화에서 나옵니다. 변수가 구체적이고 불변인 타입을 가질 때, 컴파일러는 함수 내부에서 이를 언박싱할 수 있습니다. 그렇지 않은 경우, 변수는 힙에 할당되어야 하고 가비지 컬렉터에 참여해야 합니다. 불변 타입은 Julia에 특정한 개념입니다. 불변 타입의 값은 변경될 수 없습니다.

불변 타입은 일반적으로 값들의 집합보다는 값을 나타내는 타입입니다. 예를 들어, `Int64`와 `Float64`를 포함한 대부분의 숫자 타입들은 불변입니다. (Julia의 숫자 타입들은 특별한 원시 타입이 아닌 일반적인 타입입니다. 제공된 것과 동일한 새로운 `MyInt64`를 정의할 수도 있습니다.) 불변 타입은 수정될 수 없기 때문에, 변경하고 싶을 때마다 새로운 복사본을 만들어야 합니다. 예를 들어 `4 + 6`는 결과를 담을 새로운 `Int64`를 만들어야 합니다. 반면 가변 타입의 멤버들은 제자리에서 업데이트될 수 있습니다. 이는 변경을 위해 전체를 복사할 필요가 없다는 뜻입니다.

`x = x + 2`가 메모리를 할당한다는 아이디어는 아마 꽤 이상하게 들릴 것입니다. `Int64` 값을 불변으로 만들어서 이런 기본 연산을 느리게 만드는 이유가 뭘까요? 여기서 컴파일러 최적화가 등장합니다. 불변 타입을 사용하는 것이 (보통은) 속도를 늦추지 않습니다. `x`가 안정적이고 구체적인 타입(`Int64`와 같은)을 가진다면, 컴파일러는 자유롭게 `x`를 스택에 할당하고 `x`를 제자리에서 변형할 수 있습니다. 문제는 `x`가 불안정한 타입을 가질 때만 발생합니다(따라서 컴파일러가 얼마나 크거나 어떤 타입일지 모릅니다). `x`가 박싱되어 힙에 있게 되면, 컴파일러는 다른 코드 조각이 그 값을 사용하지 않는다고 완전히 확신할 수 없어서 편집할 수 없습니다.

`stable`의 `sum`은 구체적인 타입(`Float64`)을 가지므로, 컴파일러는 이를 함수 내에서 언박싱된 상태로 로컬에 저장하고 그 값을 변형할 수 있다는 것을 압니다. `sum`은 힙에 할당되지 않으며 `i/2`를 더할 때마다 새로운 복사본을 만들 필요가 없습니다.

`unstable`의 `sum`은 구체적인 타입을 가지지 않으므로, 컴파일러는 이를 힙에 할당합니다. sum을 수정할 때마다 힙에 새로운 값을 할당합니다. 힙에 값을 할당하고 (`sum`의 값을 읽고 싶을 때마다 검색하는) 모든 이런 시간은 비쌉니다.

`0` 대 `0.0`을 사용하는 것은 특히 Julia를 처음 접할 때 하기 쉬운 실수입니다. 루프에서 사용되는 변수가 타입 안정적인지 자동으로 확인하는 것은 프로그래머가 성능에 중요한 코드 섹션에서 변수들의 타입이 무엇인지에 대한 더 많은 통찰을 얻는 데 도움이 됩니다.

### 구현 세부사항

루프 내부에서 어떤 변수들이 사용되는지 알아내야 하고, 그 변수들의 타입을 찾아야 합니다. 그리고 나서 이를 사람이 읽을 수 있는 형식으로 출력하는 방법을 결정해야 합니다.

* 루프를 어떻게 찾을 것인가?
* 루프에서 변수들을 어떻게 찾을 것인가?
* 변수의 타입을 어떻게 찾을 것인가?
* 결과를 어떻게 출력할 것인가?
* 타입이 불안정한지 어떻게 알 수 있는가?

이 전체 노력이 마지막 질문에 달려 있으므로, 마지막 질문부터 다루겠습니다. 우리는 불안정한 함수를 살펴보고 프로그래머로서 불안정한 변수를 식별하는 방법을 보았지만, 우리의 프로그램이 이를 찾도록 해야 합니다. 이는 값이 변할 수 있는 변수를 찾기 위해 함수를 시뮬레이션해야 하는 것처럼 들리며&mdash;상당한 작업이 필요할 것 같습니다. 다행히 Julia의 타입 추론은 이미 타입을 결정하기 위해 함수의 실행을 추적합니다.

`unstable`의 `sum`의 타입은 `Union(Float64,Int64)`입니다. 이는 `UnionType`으로, 변수가 타입 값들의 집합 중 어느 것이든 가질 수 있다는 것을 나타내는 특별한 종류의 타입입니다. `Union(Float64,Int64)` 타입의 변수는 `Int64` 또는 `Float64` 타입의 값을 가질 수 있습니다. 값은 그 타입들 중 하나만 가질 수 있습니다. `UnionType`은 임의 개수의 타입을 결합합니다(예: `UnionType(Float64, Int64, Int32)`은 세 개의 타입을 결합합니다). 우리가 찾고자 하는 것은 루프 내부의 `UnionType`된 변수들입니다.

코드를 대표적인 구조로 파싱하는 것은 복잡한 일이고, 언어가 성장함에 따라 더욱 복잡해집니다. 이 장에서는 컴파일러가 사용하는 내부 데이터 구조에 의존할 것입니다. 이는 파일을 읽거나 파싱하는 것에 대해 걱정할 필요가 없다는 것을 의미하지만, 우리가 제어하지 않으며 때로는 어색하거나 보기 흉하게 느껴지는 데이터 구조와 작업해야 한다는 것을 의미합니다.

코드를 직접 파싱하지 않아도 되는 모든 작업을 절약하는 것 외에도, 컴파일러가 사용하는 동일한 데이터 구조로 작업한다는 것은 우리의 검사가 컴파일러의 이해에 대한 정확한 평가에 기반할 것이라는 의미입니다&mdash;이는 우리의 검사가 코드가 실제로 실행되는 방식과 일치할 것임을 의미합니다.

Julia 코드에서 Julia 코드를 검사하는 이런 과정을 내성(introspection)이라고 합니다. 당신이나 제가 내성할 때, 우리는 어떻게 그리고 왜 우리가 생각하고 느끼는지에 대해 생각하고 있습니다. 코드가 내성할 때는 같은 언어의 코드(아마도 자신의 코드)의 표현이나 실행 속성을 검사합니다. 코드의 내성이 검사된 코드를 수정하는 것까지 확장될 때, 이를 메타프로그래밍(프로그램을 작성하거나 수정하는 프로그램)이라고 합니다.

#### Julia에서의 내성

Julia는 내성을 쉽게 만들어줍니다. 컴파일러가 무엇을 생각하고 있는지 볼 수 있게 해주는 네 가지 내장 함수가 있습니다: `code_lowered`, `code_typed`, `code_llvm`, `code_native`. 이들은 컴파일 과정에서 어떤 단계의 출력인지에 따라 순서대로 나열된 것입니다. 첫 번째는 우리가 입력하는 코드에 가장 가깝고 마지막 것은 CPU가 실행하는 것에 가장 가깝습니다. 이 장에서는 최적화되고 타입이 추론된 추상 구문 트리(AST)를 제공하는 `code_typed`에 집중하겠습니다.

`code_typed`는 두 개의 인수를 받습니다: 관심 있는 함수와 인수 타입들의 튜플입니다. 예를 들어, 두 개의 `Int64`로 호출될 때 함수 `foo`의 AST를 보고 싶다면, `code_typed(foo, (Int64,Int64))`를 호출할 것입니다.

```julia
function foo(x,y)
  z = x + y
  return 2 * z
end

code_typed(foo,(Int64,Int64))
```

이것은 `code_typed`가 반환할 구조입니다:
```
1-element Array{Any,1}:
:($(Expr(:lambda, {:x,:y}, {{:z},{{:x,Int64,0},{:y,Int64,0},{:z,Int64,18}},{}},
 :(begin  # none, line 2:
        z = (top(box))(Int64,(top(add_int))(x::Int64,y::Int64))::Int64 # line 3:
        return (top(box))(Int64,(top(mul_int))(2,z::Int64))::Int64
    end::Int64))))
```

이것은 `Array`입니다. 이는 `code_typed`가 여러 일치하는 메서드를 반환할 수 있게 해줍니다. 함수와 인수 타입의 일부 조합은 어떤 메서드가 호출되어야 하는지를 완전히 결정하지 못할 수 있습니다. 예를 들어, (`Int64` 대신에) `Any`와 같은 타입을 전달할 수 있습니다. `Any`는 타입 계층의 최상위에 있는 타입입니다. 모든 타입은 `Any`의 하위타입입니다(`Any` 자체를 포함해서). 인수 타입의 튜플에 `Any`를 포함하고 여러 일치하는 메서드가 있다면, `code_typed`에서 나온 `Array`는 하나 이상의 요소를 가질 것입니다. 일치하는 메서드당 하나씩의 요소를 가질 것입니다.

더 쉽게 이야기하기 위해 예제 `Expr`을 꺼내보겠습니다.
```julia
julia> e = code_typed(foo,(Int64,Int64))[1]
:($(Expr(:lambda, {:x,:y}, {{:z},{{:x,Int64,0},{:y,Int64,0},{:z,Int64,18}},{}},
 :(begin  # none, line 2:
        z = (top(box))(Int64,(top(add_int))(x::Int64,y::Int64))::Int64 # line 3:
        return (top(box))(Int64,(top(mul_int))(2,z::Int64))::Int64
    end::Int64))))
```

우리가 관심 있는 구조는 `Array` 안에 있습니다. 그것은 `Expr`입니다. Julia는 AST를 나타내기 위해 `Expr`(expression의 줄임말)을 사용합니다. (추상 구문 트리는 컴파일러가 코드의 의미에 대해 생각하는 방식입니다. 초등학교에서 문장을 다이어그램으로 그려야 했던 것과 비슷합니다.) 우리가 받은 `Expr`은 하나의 메서드를 나타냅니다. 이는 (메서드에 나타나는 변수들에 대한) 메타데이터와 메서드의 본체를 구성하는 표현식들을 가집니다.

이제 `e`에 대해 몇 가지 질문을 할 수 있습니다.

어떤 Julia 값이나 타입에서도 작동하는 `names` 함수를 사용해서 `Expr`이 어떤 속성을 가지는지 물을 수 있습니다. 이는 그 타입(또는 값의 타입)에 의해 정의된 이름들의 `Array`를 반환합니다.

```julia
julia> names(e)
3-element Array{Symbol,1}:
 :head
 :args
 :typ
```

방금 `e`가 어떤 이름들을 가지는지 물었고, 이제 각 이름이 어떤 값에 대응되는지 물을 수 있습니다. `Expr`은 세 가지 속성을 가집니다: `head`, `typ`, `args`.

```julia
julia> e.head
:lambda

julia> e.typ
Any

julia> e.args
3-element Array{Any,1}:
 {:x,:y}
 {{:z},{{:x,Int64,0},{:y,Int64,0},{:z,Int64,18}},{}}
 :(begin  # none, line 2:
        z = (top(box))(Int64,(top(add_int))(x::Int64,y::Int64))::Int64 # line 3:
        return (top(box))(Int64,(top(mul_int))(2,z::Int64))::Int64
    end::Int64)
```

방금 몇 가지 값들이 출력되는 것을 보았지만, 그것들이 무엇을 의미하고 어떻게 사용되는지에 대해서는 많이 알려주지 않습니다.

- `head`는 이것이 어떤 종류의 표현식인지 알려줍니다. 일반적으로 Julia에서는 이를 위해 별도의 타입을 사용하겠지만, `Expr`은 파서에서 사용되는 구조를 모델링하는 타입입니다. 파서는 Scheme의 방언으로 작성되었으며, 모든 것을 중첩된 리스트로 구조화합니다. `head`는 나머지 `Expr`이 어떻게 구성되어 있고 어떤 종류의 표현식을 나타내는지 알려줍니다.
- `typ`는 표현식의 추론된 반환 타입입니다. 어떤 표현식을 평가할 때 어떤 값이 결과로 나옵니다. `typ`는 표현식이 평가될 값의 타입입니다. 거의 모든 `Expr`에서 이 값은 `Any`일 것입니다(모든 가능한 타입이 `Any`의 하위타입이므로 항상 올바릅니다). 타입이 추론된 메서드의 `body`와 그 안의 대부분 표현식들만이 `typ`을 더 구체적인 것으로 설정할 것입니다. (`type`은 키워드이므로 이 필드는 그 단어를 이름으로 사용할 수 없습니다.)
- `args`는 `Expr`의 가장 복잡한 부분입니다. 그 구조는 `head`의 값에 따라 달라집니다. 항상 `Array{Any}`(타입이 없는 배열)이지만, 그 외에는 구조가 변합니다.

메서드를 나타내는 `Expr`에서는 `e.args`에 세 개의 요소가 있을 것입니다:

```julia
julia> e.args[1] # names of arguments as symbols
2-element Array{Any,1}:
 :x
 :y
```

심볼(Symbol)은 변수, 상수, 함수, 모듈의 이름을 나타내는 특별한 타입입니다. 프로그램 구성요소의 이름을 구체적으로 나타내기 때문에 문자열과는 다른 타입입니다.

```julia
julia> e.args[2] # three lists of variable metadata
3-element Array{Any,1}:
 {:z}
 {{:x,Int64,0},{:y,Int64,0},{:z,Int64,18}}
 {}
```

위의 첫 번째 리스트는 모든 지역 변수의 이름을 포함합니다. 여기서는 하나(`z`)만 있습니다. 두 번째 리스트는 메서드의 각 변수와 인수에 대한 튜플을 포함합니다. 각 튜플은 변수명, 변수의 추론된 타입, 그리고 숫자를 가집니다. 숫자는 변수가 어떻게 사용되는지에 대한 정보를 (사람보다는) 기계 친화적인 방식으로 전달합니다. 마지막 리스트는 캡처된 변수 이름들입니다. 이 예제에서는 비어있습니다.

```julia
julia> e.args[3] # the body of the method
:(begin  # none, line 2:
        z = (top(box))(Int64,(top(add_int))(x::Int64,y::Int64))::Int64 # line 3:
        return (top(box))(Int64,(top(mul_int))(2,z::Int64))::Int64
    end::Int64)
```

처음 두 `args` 요소는 세 번째에 대한 메타데이터입니다. 메타데이터는 매우 흥미롭지만, 지금 당장은 필요하지 않습니다. 중요한 부분은 세 번째 요소인 메서드의 본체입니다. 이것도 또 다른 `Expr`입니다.

```julia
julia> body = e.args[3]
:(begin  # none, line 2:
        z = (top(box))(Int64,(top(add_int))(x::Int64,y::Int64))::Int64 # line 3:
        return (top(box))(Int64,(top(mul_int))(2,z::Int64))::Int64
    end::Int64)

julia> body.head
:body
```

이 `Expr`은 메서드의 본체이므로 head가 `:body`입니다.

```julia
julia> body.typ
Int64
```

`typ`는 메서드의 추론된 반환 타입입니다.

```julia
julia> body.args
4-element Array{Any,1}:
 :( # none, line 2:)
 :(z = (top(box))(Int64,(top(add_int))(x::Int64,y::Int64))::Int64)
 :( # line 3:)
 :(return (top(box))(Int64,(top(mul_int))(2,z::Int64))::Int64)
```

`args`는 표현식들의 리스트를 담고 있습니다: 메서드 본체의 표현식들의 리스트입니다. 줄 번호 주석들(예: `:( # line 3:)`)이 몇 개 있지만, 본체의 대부분은 `z`의 값을 설정(`z = x + y`)하고 `2 * z`를 반환하는 것입니다. 이런 연산들이 `Int64` 전용 내장 함수들로 대체된 것을 주목하세요. `top(function-name)`은 내장 함수를 나타냅니다. Julia에서가 아닌 Julia의 코드 생성에서 구현된 것입니다.

아직 루프가 어떻게 생겼는지 보지 못했으므로, 시도해봅시다.

```julia
julia> function lloop(x)
         for x = 1:100
           x *= 2
         end
       end
lloop (generic function with 1 method)

julia> code_typed(lloop, (Int,))[1].args[3]
:(begin  # none, line 2:
        #s120 = $(Expr(:new, UnitRange{Int64}, 1, :(((top(getfield))(Intrinsics,
         :select_value))((top(sle_int))(1,100)::Bool,100,(top(box))(Int64,(top(
         sub_int))(1,1))::Int64)::Int64)))::UnitRange{Int64}
        #s119 = (top(getfield))(#s120::UnitRange{Int64},:start)::Int64        unless
         (top(box))(Bool,(top(not_int))(#s119::Int64 === (top(box))(Int64,(top(
         add_int))((top(getfield))
         (#s120::UnitRange{Int64},:stop)::Int64,1))::Int64::Bool))::Bool goto 1
        2:
        _var0 = #s119::Int64
        _var1 = (top(box))(Int64,(top(add_int))(#s119::Int64,1))::Int64
        x = _var0::Int64
        #s119 = _var1::Int64 # line 3:
        x = (top(box))(Int64,(top(mul_int))(x::Int64,2))::Int64
        3:
        unless (top(box))(Bool,(top(not_int))((top(box))(Bool,(top(not_int))
         (#s119::Int64 === (top(box))(Int64,(top(add_int))((top(getfield))(
         #s120::UnitRange{Int64},:stop)::Int64,1))::Int64::Bool))::Bool))::Bool
         goto 2
        1:         0:
        return
    end::Nothing)
```

본체에 for나 while 루프가 없다는 것을 눈치챌 것입니다. 컴파일러가 우리가 작성한 코드를 CPU가 이해하는 이진 명령어로 변환할 때, 인간에게는 유용하지만 CPU가 이해하지 못하는 기능들(루프 같은)은 제거됩니다. 루프는 `label`과 `goto` 표현식으로 다시 작성되었습니다. `goto`는 숫자를 가지고 있으며, 각 `label`도 숫자를 가집니다. `goto`는 같은 숫자를 가진 `label`로 점프합니다.

#### 루프 감지 및 추출

뒤로 점프하는 `goto` 표현식을 찾아서 루프를 찾을 것입니다.

레이블과 고토를 찾고, 어떤 것들이 일치하는지 알아내야 합니다. 먼저 전체 구현을 보여드리겠습니다. 코드 덩어리 다음에, 이를 분해해서 조각들을 살펴보겠습니다.

```julia
# 메서드 본체에서 루프를 감지하려는 함수
# 하나 이상의 루프 내부에 있는 라인들을 반환
function loopcontents(e::Expr)
  b = body(e)
  loops = Int[]
  nesting = 0
  lines = {}
  for i in 1:length(b)
    if typeof(b[i]) == LabelNode
      l = b[i].label
      jumpback = findnext(x-> (typeof(x) == GotoNode && x.label == l)
                              || (Base.is_expr(x,:gotoifnot) && x.args[end] == l),
                          b, i)
      if jumpback != 0
        push!(loops,jumpback)
        nesting += 1
      end
    end
    if nesting > 0
      push!(lines,(i,b[i]))
    end

    if typeof(b[i]) == GotoNode && in(i,loops)
      splice!(loops,findfirst(loops,i))
      nesting -= 1
    end
  end
  lines
end
```

이제 조각별로 설명하겠습니다:

```julia
b = body(e)
```

메서드 본체의 모든 표현식을 `Array`로 가져오는 것부터 시작합니다. `body`는 제가 이미 구현한 함수입니다:

```julia
  # 메서드의 본체를 반환.
  # 메서드를 나타내는 Expr을 받아서,
  # Vector{Expr}을 반환.
  function body(e::Expr)
    return e.args[3].args
  end
```

그리고 나서:

```julia
  loops = Int[]
  nesting = 0
  lines = {}
```

`loops`는 루프인 고토가 발생하는 레이블 줄 번호들의 `Array`입니다. `nesting`은 현재 우리가 안에 있는 루프의 개수를 나타냅니다. `lines`는 (인덱스, `Expr`) 튜플들의 `Array`입니다.

```julia
  for i in 1:length(b)
    if typeof(b[i]) == LabelNode
      l = b[i].label
      jumpback = findnext(
        x-> (typeof(x) == GotoNode && x.label == l)
            || (Base.is_expr(x,:gotoifnot) && x.args[end] == l),
        b, i)
      if jumpback != 0
        push!(loops,jumpback)
        nesting += 1
      end
    end
```

`e`의 본체에서 각 표현식을 살펴봅니다. 레이블이라면, 이 레이블로 점프하는 고토가 있는지 (그리고 현재 인덱스 이후에 발생하는지) 확인합니다. `findnext`의 결과가 0보다 크면, 그런 고토 노드가 존재하므로 `loops`(현재 우리가 안에 있는 루프들의 `Array`)에 추가하고 `nesting` 레벨을 증가시킵니다.

```julia
    if nesting > 0
      push!(lines,(i,b[i]))
    end
```

현재 루프 안에 있다면, 현재 줄을 반환할 줄들의 배열에 추가합니다.

```julia
    if typeof(b[i]) == GotoNode && in(i,loops)
      splice!(loops,findfirst(loops,i))
      nesting -= 1
    end
  end
  lines
end
```

`GotoNode`에 있다면, 루프의 끝인지 확인합니다. 그렇다면 `loops`에서 엔트리를 제거하고 nesting 레벨을 감소시킵니다.

이 함수의 결과는 `lines` 배열로, (인덱스, 값) 튜플들의 배열입니다. 이는 배열의 각 값이 메서드 본체 `Expr`의 본체에 대한 인덱스와 그 인덱스의 값을 가진다는 의미입니다. `lines`의 각 요소는 루프 안에서 발생한 표현식입니다.

#### 변수 찾기 및 타입 지정

루프 내부에 있는 `Expr`들을 반환하는 `loopcontents` 함수를 방금 완성했습니다. 다음 함수는 `loosetypes`로, `Expr`들의 리스트를 받아서 느슨하게 타입이 지정된 변수들의 리스트를 반환합니다. 나중에 `loopcontents`의 출력을 `loosetypes`에 전달할 것입니다.

루프 안에서 발생한 각 표현식에서, `loosetypes`는 심볼들과 그와 연관된 타입들의 발생을 검색합니다. 변수 사용은 AST에서 `SymbolNode`로 나타납니다. `SymbolNode`들은 변수의 이름과 추론된 타입을 담고 있습니다.

`loopcontents`가 수집한 각 표현식이 `SymbolNode`인지 확인하기만 하면 되는 것은 아닙니다. 문제는 각 `Expr`이 하나 이상의 `Expr`을 포함할 수 있고, 각 `Expr`이 하나 이상의 `SymbolNode`를 포함할 수 있다는 것입니다. 이는 중첩된 `Expr`들을 꺼내야 한다는 의미이므로, 각각에서 `SymbolNode`들을 찾을 수 있습니다.

```julia
# given `lr`, a Vector of expressions (Expr + literals, etc)
# try to find all occurrences of a variables in `lr`
# and determine their types
function loosetypes(lr::Vector)
  symbols = SymbolNode[]
  for (i,e) in lr
    if typeof(e) == Expr
      es = copy(e.args)
      while !isempty(es)
        e1 = pop!(es)
        if typeof(e1) == Expr
          append!(es,e1.args)
        elseif typeof(e1) == SymbolNode
          push!(symbols,e1)
        end
      end
    end
  end
  loose_types = SymbolNode[]
  for symnode in symbols
    if !isleaftype(symnode.typ) && typeof(symnode.typ) == UnionType
      push!(loose_types, symnode)
    end
  end
  return loose_types
end
```


```julia
  symbols = SymbolNode[]
  for (i,e) in lr
    if typeof(e) == Expr
      es = copy(e.args)
      while !isempty(es)
        e1 = pop!(es)
        if typeof(e1) == Expr
          append!(es,e1.args)
        elseif typeof(e1) == SymbolNode
          push!(symbols,e1)
        end
      end
    end
  end
```

The while loop goes through the guts of all the `Expr`s, recursively. Every time the loop finds a `SymbolNode`, it adds it to the vector `symbols`.

```julia
  loose_types = SymbolNode[]
  for symnode in symbols
    if !isleaftype(symnode.typ) && typeof(symnode.typ) == UnionType
      push!(loose_types, symnode)
    end
  end
  return loose_types
end
```
Now we have a list of variables and their types, so it's easy to check if a type is loose. `loosetypes` does that by looking for a specific kind of non-concrete type, a `UnionType`. We get a lot more "failing" results when we consider all non-concrete types to be "failing". This is because we're evaluating each method with its annotated argument types, which are likely to be abstract.

### Making This Usable

Now that we can do the check on an expression, we should make it easier to call on a user's code. We'll create two ways to call `checklooptypes`:

1. On a whole function; this will check each method of the given function.

2. On an expression; this will work if the user extracts the results of `code_typed` themselves.

```julia
## for a given Function, run checklooptypes on each Method
function checklooptypes(f::Callable;kwargs...)
  lrs = LoopResult[]
  for e in code_typed(f)
    lr = checklooptypes(e)
    if length(lr.lines) > 0 push!(lrs,lr) end
  end
  LoopResults(f.env.name,lrs)
end

# for an Expr representing a Method,
# check that the type of each variable used in a loop
# has a concrete type
checklooptypes(e::Expr;kwargs...) = 
 LoopResult(MethodSignature(e),loosetypes(loopcontents(e)))
```

We can see both options work about the same for a function with one method:

```julia
julia> using TypeCheck

julia> function foo(x::Int)
         s = 0
         for i = 1:x
           s += i/2
         end
         return s
       end
foo (generic function with 1 method)

julia> checklooptypes(foo)
foo(Int64)::Union(Int64,Float64)
	s::Union(Int64,Float64)
	s::Union(Int64,Float64)


julia> checklooptypes(code_typed(foo,(Int,))[1])
(Int64)::Union(Int64,Float64)
	s::Union(Int64,Float64)
	s::Union(Int64,Float64)
```

#### Pretty Printing
I've skipped an implementation detail here: how did we get the results to print out to the REPL?

First, I made some new types. `LoopResults` is the result of checking a whole function; it has the function name and the results for each method. `LoopResult` is the result of checking one method; it has the argument types and the loosely typed variables.

The `checklooptypes` function returns a `LoopResults`. This type has a function called `show` defined for it. The REPL calls `display` on values it wants to display; `display` will then call our `show` implementation.

This code is important for making this static analysis usable, but it is not doing static analysis. You should use the preferred method for pretty-printing types and output in your implementation language; this is just how it's done in Julia.

```julia
type LoopResult
  msig::MethodSignature
  lines::Vector{SymbolNode}
  LoopResult(ms::MethodSignature,ls::Vector{SymbolNode}) = new(ms,unique(ls))
end

function Base.show(io::IO, x::LoopResult)
  display(x.msig)
  for snode in x.lines
    println(io,"\t",string(snode.name),"::",string(snode.typ))
  end
end

type LoopResults
  name::Symbol
  methods::Vector{LoopResult}
end

function Base.show(io::IO, x::LoopResults)
  for lr in x.methods
    print(io,string(x.name))
    display(lr)
  end
end
```


## 미사용 변수 찾기

프로그램을 입력할 때 때로는 변수명을 잘못 입력합니다. 프로그램은 이전에 올바르게 철자를 입력한 같은 변수를 의도했다는 것을 알 수 없습니다. 프로그램은 한 번만 사용된 변수를 보지만, 여러분은 철자가 틀린 변수명을 볼 수 있습니다. 변수 선언을 요구하는 언어들은 자연스럽게 이런 철자 오류를 잡아내지만, 많은 동적 언어들은 선언을 요구하지 않아서 이를 잡아내기 위해 추가적인 분석 계층이 필요합니다.

한 번만 사용되거나&mdash;한 가지 방식으로만 사용되는 변수를 찾아서 철자가 틀린 변수명(과 다른 미사용 변수들)을 찾을 수 있습니다.

다음은 철자가 틀린 이름 하나가 있는 작은 코드 예제입니다.

```julia
function foo(variable_name::Int)
  sum = 0
  for i=1:variable_name
    sum += variable_name
  end
  variable_nme = sum
  return variable_name
end
```

이런 종류의 실수는 코드를 실행할 때만 발견되는 문제를 일으킬 수 있습니다. 각 변수명을 한 번만 잘못 철자한다고 가정해보겠습니다. 변수 사용을 쓰기와 읽기로 분리할 수 있습니다. 철자 오류가 쓰기라면(예: `worng = 5`), 오류가 발생하지 않을 것입니다. 단지 조용히 잘못된 변수에 값을 넣을 뿐이고&mdash;버그를 찾는 것이 좌절스러울 수 있습니다. 철자 오류가 읽기라면(예: `right = worng + 2`), 코드가 실행될 때 런타임 오류를 얻을 것입니다. 이 오류를 더 빨리 찾을 수 있도록 정적 경고를 원하지만, 문제를 보기 위해 코드를 실행할 때까지 여전히 기다려야 합니다.

코드가 더 길고 복잡해질수록 실수를 발견하기가 더 어려워집니다&mdash;정적 분석의 도움이 없다면요.

### 좌변과 우변

"읽기"와 "쓰기" 사용에 대해 이야기하는 다른 방법은 이를 "우변(right-hand side, RHS)"과 "좌변(left-hand side, LHS)" 사용이라고 부르는 것입니다. 이는 변수가 `=` 기호에 상대적으로 어디에 있는지를 가리킵니다.

다음은 `x`의 몇 가지 사용법입니다:

* 좌변:
    * `x = 2`
    * `x = y + 22`
    * `x = x + y + 2`
    * `x += 2` (이는 `x = x + 2`로 역설탕화됩니다)
* 우변:
    * `y = x + 22`
    * `x = x + y + 2`
    * `x += 2` (이는 `x = x + 2`로 역설탕화됩니다)
    * `2 * x`
    * `x`

`x = x + y + 2`와 `x += 2` 같은 표현식들은 `x`가 `=` 기호의 양쪽에 나타나므로 두 섹션에 모두 나타난다는 것을 주목하세요.

### 단일 사용 변수 찾기

찾아야 할 두 가지 경우가 있습니다:

1. 한 번만 사용되는 변수들.
2. LHS에서만 사용되거나 RHS에서만 사용되는 변수들.

모든 변수 사용을 찾을 것이지만, 두 경우를 모두 다루기 위해 LHS와 RHS 사용을 별도로 살펴볼 것입니다.

#### Finding LHS Usages

To be on the LHS, a variable needs to have an `=` sign to be to the left of. This means we can look for `=` signs in the AST, and then look to the left of them to find the relevant variable.

In the AST, an `=` is an `Expr` with the head `:(=)`. (The parentheses are there to make it clear that this is the symbol for `=` and not another operator, `:=`.) The first value in `args` will be the variable name on its LHS. Because we're looking at an AST that the compiler has already cleaned up, there will (nearly) always be just a single symbol to the left of our `=` sign.

Let's see what that means in code:
```julia
julia> :(x = 5)
:(x = 5)

julia> :(x = 5).head
:(=)

julia> :(x = 5).args
2-element Array{Any,1}:
  :x
 5  

julia> :(x = 5).args[1]
:x
```

Below is the full implementation, followed by an explanation.

```julia
# Return a list of all variables used on the left-hand-side of assignment (=)
#
# Arguments:
#   e: an Expr representing a Method, as from code_typed
#
# Returns:
#   a Set{Symbol}, where each element appears on the LHS of an assignment in e.
#
function find_lhs_variables(e::Expr)
  output = Set{Symbol}()
  for ex in body(e)
    if Base.is_expr(ex,:(=))
      push!(output,ex.args[1])
    end
  end
  return output
end
```

```julia
  output = Set{Symbol}()
```
    
We have a set of Symbols; those are variables names we've found on the LHS.

```julia
  for ex in body(e)
    if Base.is_expr(ex,:(=))
      push!(output,ex.args[1])
    end
  end
```
We aren't digging deeper into the expressions, because the `code_typed` AST is pretty flat; loops and ifs have been converted to flat statements with gotos for control flow. There won't be any assignments hiding inside function calls' arguments. This code will fail if anything more than a symbol is on the left of the equal sign. This misses two specific edge cases: array accesses (like `a[5]`, which will be represented as a `:ref` expression) and properties (like `a.head`, which will be represented as a `:.` expression). These will still always have the relevant symbol as the first value in their `args`, it might just be buried a bit (as in `a.property.name.head.other_property`). This code doesn’t handle those cases, but a couple lines of code inside the `if` statement could fix that.

```julia
      push!(output,ex.args[1])
```
When we find a LHS variable usage, we `push!` the variable name into the `Set`. The `Set` will make sure that we only have one copy of each name.

#### Finding RHS usages

To find all the other variable usages, we also need to look at each `Expr`. This is a bit more involved, because we care about basically all the `Expr`s, not just the `:(=)` ones and because we have to dig into nested `Expr`s (to handle nested function calls).

Here is the full implementation, with explanation following.
```julia
# Given an Expression, finds variables used in it (on right-hand-side)
#
# Arguments: e: an Expr
#
# Returns: a Set{Symbol}, where each e is used in a rhs expression in e
#
function find_rhs_variables(e::Expr)
  output = Set{Symbol}()

  if e.head == :lambda
    for ex in body(e)
      union!(output,find_rhs_variables(ex))
    end
  elseif e.head == :(=)
    for ex in e.args[2:end]  # skip lhs
      union!(output,find_rhs_variables(ex))
    end
  elseif e.head == :return
    output = find_rhs_variables(e.args[1])
  elseif e.head == :call
    start = 2  # skip function name
    e.args[1] == TopNode(:box) && (start = 3)  # skip type name
    for ex in e.args[start:end]
      union!(output,find_rhs_variables(ex))
    end
  elseif e.head == :if
   for ex in e.args # want to check condition, too
     union!(output,find_rhs_variables(ex))
   end
  elseif e.head == :(::)
    output = find_rhs_variables(e.args[1])
  end

  return output
end
```

The main structure of this function is a large if-else statement, where each case handles a different head-symbol.

```julia
  output = Set{Symbol}()
```

`output` is the set of variable names, which we will return at the end of the function. Since we only care about the fact that each of these variables has be read at least once, using a `Set` frees us from worrying about the uniqueness of each name.

```julia
  if e.head == :lambda
    for ex in body(e)
      union!(output,find_rhs_variables(ex))
    end
```

This is the first condition in the if-else statement. A `:lambda` represents the body of a function. We recurse on the body of the definition, which should get all the RHS variable usages in the definition.

```julia
  elseif e.head == :(=)
    for ex in e.args[2:end]  # skip lhs
      union!(output,find_rhs_variables(ex))
    end
```

If the head is `:(=)`, then the expression is an assignment. We skip the first element of `args` because that's the variable being assigned to. For each of the remaining expressions, we recursively find the RHS variables and add them to our set.

```julia
  elseif e.head == :return
    output = find_rhs_variables(e.args[1])
```

If this is a return statement, then the first element of `args` is the expression whose value is returned; we'll add any variables in there into our set.

```julia
  elseif e.head == :call
    # skip function name
    for ex in e.args[2:end]
      union!(output,find_rhs_variables(ex))
    end
```

For function calls, we want to get all variables used in all the arguments to the call. We skip the function name, which is the first element of `args`.

```julia
  elseif e.head == :if
   for ex in e.args # want to check condition, too
     union!(output,find_rhs_variables(ex))
   end
```
An `Expr` representing an if statement has the `head` value `:if`. We want to get variable usages from all the expressions in the body of the if statement, so we recurse on each element of `args`.

```julia
  elseif e.head == :(::)
    output = find_rhs_variables(e.args[1])
  end
```

The `:(::)` operator is used to add type annotations. The first argument is the expression or variable being annotated; we check for variable usages in the annotated expression.

```julia
  return output
```

At the end of the function, we return the set of RHS variable usages.


위의 메서드를 단순화하는 코드가 조금 더 있습니다. 위 버전은 `Expr`만 처리하지만, 재귀적으로 전달되는 일부 값은 `Expr`이 아닐 수 있으므로, 다른 가능한 타입을 적절히 처리하기 위한 몇 가지 메서드가 더 필요합니다.

```julia
# Recursive Base Cases, to simplify control flow in the Expr version
find_rhs_variables(a) = Set{Symbol}()  # unhandled, should be immediate val e.g. Int
find_rhs_variables(s::Symbol) = Set{Symbol}([s])
find_rhs_variables(s::SymbolNode) = Set{Symbol}([s.name])
```

#### Putting It Together

Now that we have the two functions defined above, we can use them together to find variables that are either only read from or only written to. The function that finds them will be called `unused_locals`.

```julia
function unused_locals(e::Expr)
  lhs = find_lhs_variables(e)
  rhs = find_rhs_variables(e)
  setdiff(lhs,rhs)
end
```

`unused_locals` will return a set of variable names. It's easy to write a function that determines whether the output of `unused_locals` counts as a "pass" or not. If the set is empty, the method passes. If all the methods of a function pass, then the function passes. The function `check_locals` below implements this logic.

```julia
check_locals(f::Callable) = all([check_locals(e) for e in code_typed(f)])
check_locals(e::Expr) = isempty(unused_locals(e))
```

## 결론
Julia 코드에 대해 두 가지 정적 분석을 수행했습니다&mdash;하나는 타입에 기반하고 다른 하나는 변수 사용에 기반합니다.

정적 타입 언어들은 이미 우리의 타입 기반 분석이 수행한 종류의 작업을 합니다. 추가적인 타입 기반 정적 분석은 주로 동적 타입 언어에서 유용합니다. Python, Ruby, Lisp를 포함한 언어들을 위한 정적 타입 추론 시스템을 구축하는 (대부분 연구) 프로젝트들이 있었습니다. 이런 시스템들은 보통 선택적 타입 주석 주변에 구축됩니다. 원할 때는 정적 타입을 가질 수 있고, 원하지 않을 때는 동적 타이핑으로 되돌아갈 수 있습니다. 이는 기존 코드베이스에 일부 정적 타이핑을 통합하는 데 특히 도움이 됩니다.

우리의 변수 사용 검사와 같은 타입에 기반하지 않는 검사들은 동적 타입과 정적 타입 언어 모두에 적용 가능합니다. 하지만 C++와 Java와 같은 많은 정적 타입 언어들은 변수를 선언하도록 요구하며, 이미 우리가 만든 것과 같은 기본 경고를 제공합니다. 여전히 작성할 수 있는 사용자 정의 검사들이 있습니다. 예를 들어, 프로젝트의 스타일 가이드에 특정한 검사들이나 보안 정책에 기반한 추가 안전 예방조치들입니다.

Julia는 정적 분석을 가능하게 하는 훌륭한 도구들을 가지고 있지만, 혼자만 그런 것은 아닙니다. 물론 Lisp는 코드가 중첩된 리스트의 데이터 구조라는 것으로 유명하므로, AST에 접근하기가 쉬운 편입니다. Java도 AST를 노출하지만, AST가 Lisp의 것보다 훨씬 복잡합니다. 일부 언어나 언어 도구 체인들은 단순한 사용자들이 내부 표현을 둘러보는 것을 허용하도록 설계되지 않았습니다. 오픈소스 도구 체인(특히 잘 주석이 달린 것들)의 경우, 한 가지 옵션은 AST에 접근할 수 있게 해주는 훅을 환경에 추가하는 것입니다.

그것이 작동하지 않는 경우, 최종 대안은 직접 파서를 작성하는 것입니다. 이는 가능한 한 피해야 합니다. 대부분의 프로그래밍 언어의 전체 문법을 다루는 것은 많은 작업이며, 언어에 새로운 기능이 추가될 때마다 직접 업데이트해야 합니다(업스트림에서 자동으로 업데이트를 받는 대신). 하고 싶은 검사에 따라, 몇 줄만 파싱하거나 언어 기능의 하위 집합만 파싱해서 해결할 수 있을지도 모릅니다. 이는 자신만의 파서를 작성하는 비용을 크게 줄일 것입니다.

정적 분석 도구가 어떻게 작성되는지에 대한 새로운 이해가 코드에 사용하는 도구들을 이해하는 데 도움이 되고, 아마도 자신만의 도구를 작성하도록 영감을 주기를 바랍니다.
