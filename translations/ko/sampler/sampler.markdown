title: 기각 샘플러
author: Jessica B. Hamrick
<markdown>
_Jess는 UC 버클리의 박사과정 학생으로, 머신러닝의 확률 모델과 인지과학의 행동 실험을 결합하여 인간의 인지 과정을 연구하고 있습니다. 여가 시간에는 IPython과 Jupyter의 핵심 기여자로 활동하고 있습니다. MIT에서 컴퓨터과학 학사 및 석사 학위를 받았습니다._
</markdown>
## 소개

컴퓨터과학과 공학에서는 방정식으로 해결할 수 없는 문제에
자주 마주치게 됩니다. 이런 문제들은 보통 복잡한 시스템이나
노이즈가 있는 입력, 또는 이 둘 모두를 포함하고 있습니다.
정확한 분석적 해결책이 존재하지 않는 실세계 문제들의 몇 가지
예시를 살펴보겠습니다:

1. 비행기의 컴퓨터 모델을 만들었고, 다양한 기상 조건에서
   비행기가 얼마나 잘 견딜 수 있는지 결정하고 싶은 경우.

2. 제안된 공장에서 나오는 화학 물질 유출이
   지하수 확산 모델을 기반으로 근처 주민들의 급수에
   영향을 미칠지 판단하고 싶은 경우.

3. 카메라로부터 노이즈가 있는 이미지를 촬영하는 로봇이 있고,
   그 이미지들이 묘사하는 객체의 3차원 구조를
   복원하고 싶은 경우.

4. 체스에서 특정 수를 둘 때 승리할 가능성이 얼마나 되는지
   계산하고 싶은 경우.

이런 유형의 문제들은 정확하게 해결할 수는 없지만,
*몬테카를로 샘플링* 기법이라고 알려진 방법들을 사용하여
근사해를 구할 수 있는 경우가 많습니다. 몬테카를로 방법에서
핵심 아이디어는 많은 *샘플*을 취하는 것으로, 이를 통해
해답을 추정할 수 있게 됩니다.[^note]

[^note]: 이 장에서는 통계학과 확률론에 대한 어느 정도의
친숙함을 가정합니다.


### 샘플링이란 무엇인가?

*샘플링*이라는 용어는 어떤 확률 분포로부터 무작위 값들을
생성하는 것을 의미합니다. 예를 들어, 6면 주사위를 굴려서 얻는
값은 샘플입니다. 섞인 카드 더미의 맨 위에서 뽑는 카드는
샘플입니다. 다트가 보드에 맞는 위치 역시 샘플입니다. 이런
다양한 샘플들 사이의 유일한 차이점은 서로 다른 *확률 분포*에서
생성된다는 것입니다. 주사위의 경우, 분포는 6개의 값에 동일한
가중치를 부여합니다. 카드의 경우, 분포는 52개의 값에 동일한
가중치를 부여합니다. 다트 보드의 경우, 분포는 원형 영역에
가중치를 부여합니다(다트 실력에 따라 균등하게 분포되지 않을 수도
있지만요).

보통 샘플을 사용하고 싶은 방식은 두 가지가 있습니다. 첫 번째는
단순히 나중에 사용할 무작위 값을 생성하는 것입니다. 예를 들어,
컴퓨터 포커 게임에서 무작위로 카드를 뽑는 것이 그렇습니다. 두
번째로 샘플이 사용되는 방식은 추정을 위한 것입니다. 예를 들어,
친구가 속임수 주사위를 사용한다고 의심된다면, 주사위를 여러 번
굴려서 어떤 숫자가 예상보다 더 자주 나오는지 보고 싶을 것입니다.
또는 위의 비행기 예시에서처럼 가능성의 범위를 특성화하고 싶을
수도 있습니다. 날씨는 상당히 혼돈적인 시스템으로, 비행기가 특정
기상 상황에서 살아남을지를 *정확히* 계산하는 것은 불가능합니다.
대신, 다양한 기상 조건에서 비행기의 행동을 여러 번
시뮬레이션함으로써 어떤 조건에서 비행기가 실패할 가능성이 가장
높은지 알 수 있게 됩니다.

### 샘플과 확률을 이용한 프로그래밍

컴퓨터과학의 대부분의 응용 분야와 마찬가지로, 샘플과 확률을
이용해 프로그래밍할 때 코드의 전반적인 깔끔함, 일관성, 정확성에
영향을 미치는 설계 결정을 내릴 수 있습니다. 이 장에서는 컴퓨터
게임에서 무작위 아이템을 샘플링하는 간단한 예시를 살펴볼
것입니다. 특히, 확률 작업에 특화된 설계 결정들에 초점을 맞출
것인데, 여기에는 샘플링과 확률 평가를 위한 함수들, 로그를 이용한
작업, 재현성 허용, 그리고 샘플 생성 과정을 특정 응용 분야로부터
분리하는 것이 포함됩니다.

#### 표기법에 대한 간단한 부연설명

$p(x)$와 같은 수학적 표기법을 사용하여 $p$가 무작위 변수의 값
$x$에 대한 *확률 밀도 함수*(PDF) 또는 *확률 질량 함수*(PMF)임을
나타낼 것입니다. PDF는 $\int_{-\infty}^\infty p(x)\ \mathrm{d}x=1$을
만족하는 *연속* 함수 $p(x)$인 반면, PMF는 $\sum_{x\in \mathbb{Z}} p(x)=1$을
만족하는 *이산* 함수 $p(x)$입니다. 여기서 $\mathbb{Z}$는 모든
정수의 집합입니다.

다트 보드의 경우 확률 분포는 연속 PDF가 될 것이고, 주사위의
경우 확률 분포는 이산 PMF가 될 것입니다. 두 경우 모두 모든
$x$에 대해 $p(x) \geq 0$입니다. 즉, 확률은 음이 아닌 값이어야
합니다.

확률 분포로 하고 싶은 일은 두 가지가 있습니다. 값(또는 위치)
$x$가 주어졌을 때, 그 위치에서의 확률 밀도(또는 질량)가 얼마인지
*평가*하고 싶을 수 있습니다. 수학적 표기법으로는 이를 $p(x)$
(값 $x$에서의 확률 밀도)로 쓸 것입니다.

PDF 또는 PMF가 주어졌을 때, 분포에 비례하는 방식으로 값 $x$를
*샘플링*하고 싶을 수도 있습니다(확률이 더 높은 곳에서 샘플을 얻을
가능성이 더 높도록). 수학적 표기법으로는 $x$가 $p$에 비례하여
샘플링됨을 나타내기 위해 $x\sim p$로 씁니다.

## 마법 아이템 샘플링

확률을 이용한 프로그래밍의 다양한 설계 결정들을 보여주는 간단한
예시로, 롤플레잉 게임(RPG)을 작성한다고 상상해 봅시다. 몬스터가
무작위로 드롭하는 마법 아이템의 보너스 스탯을 생성하는 방법이
필요합니다. 아이템이 제공하고자 하는 최대 보너스를 +5로 정하고,
높은 보너스일수록 낮은 보너스보다 확률이 낮다고 결정할 수
있습니다. $B$를 보너스 값에 대한 무작위 변수라고 하면:

$$
p(B=\mathrm{+1}) = 0.55\\
p(B=\mathrm{+2}) = 0.25\\
p(B=\mathrm{+3}) = 0.12\\
p(B=\mathrm{+4}) = 0.06\\
p(B=\mathrm{+5}) = 0.02
$$

또한 보너스가 분산되어야 하는 6개의 스탯(민첩, 체질, 힘, 지능,
지혜, 매력)이 있다고 명시할 수 있습니다. 따라서 +5 보너스를 가진
아이템은 그 포인트들이 여러 스탯에 분산될 수도 있고(예: 지혜 +2,
지능 +3), 하나의 스탯에 집중될 수도 있습니다(예: 매력 +5).

이 분포에서 어떻게 무작위로 샘플링할까요? 가장 쉬운 방법은 아마
먼저 전체 아이템 보너스를 샘플링한 다음, 보너스가 스탯들에
분산되는 방식을 샘플링하는 것일 것입니다. 다행히도, 보너스의 확률
분포와 그것이 분산되는 방식 모두 *다항 분포*의 사례입니다.

## 다항 분포

다항 분포는 여러 가능한 결과가 있고, 각 결과가 발생할 확률을
특성화하고 싶을 때 사용됩니다. 다항 분포를 설명하는 데 사용되는
고전적인 예시는 *공과 항아리*입니다. 아이디어는 다양한 색의 공이
들어있는 항아리가 있다는 것입니다(예를 들어, 빨간색 30%, 파란색
20%, 녹색 50%). 공을 하나 빼서 색을 기록하고, 다시 항아리에
넣은 다음, 이를 여러 번 반복합니다. 이 경우, *결과*는 특정 색의
공을 뽑는 것에 해당하고, 각 결과의 확률은 그 색의 공의
비율에 해당합니다(예: 파란 공을 뽑는 결과의 경우, 확률은
$p(\mathrm{blue})=0.20$입니다). 그러면 다항 분포는 여러 공을 뽑을 때
가능한 결과의 조합들을 설명하는 데 사용됩니다(예: 녹색 둘과
파란색 하나).

이 섹션의 코드는 `multinomial.py` 파일에 있습니다.

### `MultinomialDistribution` 클래스

일반적으로, 분포에는 두 가지 사용 사례가 있습니다: 그 분포로부터
*샘플링*하고 싶을 수도 있고, 그 분포의 PMF나 PDF 하에서 샘플(또는
샘플들)의 *확률을 평가*하고 싶을 수도 있습니다. 이 두 함수를
수행하는 데 필요한 실제 계산은 상당히 다르지만, 공통된 정보 조각에
의존합니다: 분포의 *매개변수*가 무엇인지입니다. 다항 분포의
경우, 매개변수는 사건 확률들 $p$입니다(위의 항아리 예시에서
다양한 색 공들의 비율에 해당합니다).

가장 간단한 해결책은 동일한 매개변수를 받지만 서로 독립적인 두
함수를 만드는 것일 것입니다. 하지만 저는 보통 분포를 표현하기
위해 클래스를 사용하는 것을 선택합니다. 그렇게 하는 데에는 여러
장점이 있습니다:

1. 클래스를 생성할 때 매개변수를 한 번만 전달하면 됩니다.
2. 분포에 대해 알고 싶어할 수 있는 추가 속성들이 있습니다: 평균,
   분산, 도함수 등. 공통 객체에서 작동하는 함수가 몇 개라도 있게
   되면, 동일한 매개변수를 여러 다른 함수에 전달하는 것보다 클래스를
   사용하는 것이 훨씬 더 편리합니다.
3. 매개변수 값들이 유효한지 확인하는 것이 보통 좋은 생각입니다
   (예를 들어, 다항 분포의 경우 사건 확률들의 벡터 $p$는 1의 합을
   가져야 합니다). 함수가 호출될 때마다 이 확인을 하는 것보다
   클래스의 생성자에서 한 번 하는 것이 훨씬 더 효율적입니다.
4. 때때로 PMF나 PDF를 계산하는 것은 (매개변수가 주어진) 상수 값들을
   계산하는 것을 포함합니다. 클래스를 사용하면, PMF나 PDF 함수가
   호출될 때마다 계산해야 하는 것보다 생성자에서 이런 상수들을
   미리 계산할 수 있습니다.

실제로, 이것은 `scipy.stats` 모듈에 위치한 SciPy 자체의 분포들을
포함해 많은 통계 패키지들이 작동하는 방식입니다. 하지만 우리는
다른 SciPy 함수들을 사용하고 있지만, 설명을 위해서 그리고 SciPy에
현재 다항 분포가 없기 때문에 그들의 확률 분포는 사용하지 않습니다.

클래스의 생성자 코드는 다음과 같습니다:

```python
import numpy as np

class MultinomialDistribution(object):

    def __init__(self, p, rso=np.random):
        """Initialize the multinomial random variable.

        Parameters
        ----------
        p: numpy array of length `k`
            The event probabilities
        rso: numpy RandomState object (default: None)
            The random number generator

        """

        # Check that the probabilities sum to 1. If they don't, then
        # something is wrong! We use `np.isclose` rather than checking
        # for exact equality because in many cases, we won't have
        # exact equality due to floating-point error.
        if not np.isclose(np.sum(p), 1.0):
            raise ValueError("event probabilities do not sum to 1")

        # Store the parameters that were passed in
        self.p = p
        self.rso = rso

        # Precompute log probabilities, for use by the log-PMF, for
        # each element of `self.p` (the function `np.log` operates
        # elementwise over NumPy arrays, as well as on scalars.)
        self.logp = np.log(self.p)
```

이 클래스는 사건 확률들 $p$와 `rso`라고 불리는 변수를 인자로
받습니다. 먼저, 생성자는 매개변수들이 유효한지 확인합니다. 즉,
`p`가 1의 합을 갖는지 확인합니다. 그 다음 전달받은 인자들을
저장하고, 사건 확률들을 사용해서 사건 *로그* 확률들을 계산합니다.
(이것이 왜 필요한지는 조금 후에 다룰 것입니다). `rso` 객체는
나중에 무작위 숫자를 생성하는 데 사용할 것입니다. (이것이 무엇인지
조금 후에 더 이야기할 것입니다).

클래스의 나머지 부분을 다루기 전에, 생성자와 관련된 두 가지
사항을 살펴봅시다.

#### 설명적 변수명 대 수학적 변수명

보통 프로그래머들은 설명적인 변수명을 사용하도록 권장받습니다:
예를 들어, `x`와 `y`보다는 `independent_variable`과
`dependent_variable`이라는 이름을 사용하는 것이 더 나은 관례로
여겨집니다. 표준적인 경험 법칙은 한두 글자만으로 된 변수명을
절대 사용하지 않는 것입니다. 하지만 우리의
`MultinomialDistribution` 클래스의 생성자에서 `p`라는 변수명을
사용한다는 것을 알아차릴 것인데, 이는 일반적인 명명 규칙을
위반하는 것입니다.

그런 명명 규칙이 거의 모든 영역에 적용되어야 한다는 점에
동의하지만, 한 가지 예외가 있습니다: 수학입니다. 수학적 방정식을
코딩할 때의 어려움은 그 방정식들이 보통 한 글자로만 된 변수명을
갖는다는 것입니다: $x$, $y$, $\alpha$ 등. 따라서 이들을 코드로
직접 변환한다면, 가장 쉬운 변수명은 `x`, `y`, `alpha`일 것입니다.
분명히 이들은 가장 유익한 변수명은 아니지만(`x`라는 이름은 많은
정보를 전달하지 않습니다), 더 설명적인 변수명을 갖는 것 또한
코드와 방정식 사이를 전환하는 것을 더 어렵게 만들 수 있습니다.

방정식을 직접 구현하는 코드를 작성할 때는 방정식에서와 동일한
변수명을 사용해야 한다고 생각합니다. 이렇게 하면 코드의 어떤
부분이 방정식의 어떤 조각을 구현하는지 쉽게 알 수 있습니다.
물론 이는 코드를 독립적으로 이해하기 어렵게 만들 수 있으므로,
주석이 다양한 계산의 목표를 잘 설명하는 것이 특히 중요합니다.
방정식이 학술 논문에 나와 있다면, 주석에서 쉽게 찾아볼 수 있도록
방정식 번호를 참조해야 합니다.

#### NumPy 임포트하기

`numpy` 모듈을 `np`로 임포트했다는 것을 알아차렸을 것입니다.
이는 수치 계산 세계에서의 표준 관례인데, NumPy가 엄청나게 많은
유용한 함수들을 제공하기 때문이며, 이 중 많은 것들이 하나의
파일에서도 사용될 수 있습니다. 이 장의 간단한 예시들에서는
11개의 NumPy 함수만 사용하지만, 숫자는 훨씬 더 높을 수 있습니다:
프로젝트 전체에서 40개 정도의 서로 다른 NumPy 함수를 사용하는
것이 저에게는 드물지 않습니다!

NumPy를 임포트하는 몇 가지 선택사항이 있습니다. `from numpy import *`를
사용할 수도 있지만, 함수들이 어디서 왔는지 판단하기 어렵게
만들기 때문에 일반적으로 좋지 않은 스타일입니다. `from numpy import array, log, ...`로
함수들을 개별적으로 임포트할 수도 있지만, 이는 상당히 빨리
번거로워집니다. 그냥 `import numpy`를 사용할 수도 있지만, 이는
종종 코드를 읽기 훨씬 어렵게 만듭니다. 다음 두 예시 모두 읽기
어렵지만, `numpy`보다 `np`를 사용하는 것이 훨씬 더 명확합니다:

```python
>>> numpy.sqrt(numpy.sum(numpy.dot(numpy.array(a), numpy.array(b))))
>>> np.sqrt(np.sum(np.dot(np.array(a), np.array(b))))
```

### 다항 분포로부터 샘플링하기

다항 분포에서 샘플을 취하는 것은 실제로 상당히 간단한데,
NumPy가 이를 수행하는 함수 `np.random.multinomial`을
제공해주기 때문입니다[^multinomial].

[^multinomial]: NumPy는 많은 다양한 유형의 분포에서 샘플을 추출하는
함수들을 포함하고 있습니다. 전체 목록은 무작위 샘플링 모듈
`np.random`을 살펴보세요.

이 함수가 이미 존재한다는 사실에도 불구하고, 우리가 내릴 수
있는 관련 설계 결정들이 몇 가지 있습니다.

#### 무작위 숫자 생성기 시드 설정하기

*무작위* 샘플을 추출하고 싶지만, 때때로 우리의 결과가
재현가능하기를 원합니다: 숫자들이 무작위로 보이지만, 프로그램을
다시 실행한다면 *동일한* "무작위" 숫자들의 시퀀스를 사용하기를
원할 수 있습니다.

이런 "재현가능한 무작위" 숫자들의 생성을 가능하게 하기 위해서,
샘플링 함수에게 무작위 숫자를 *어떻게* 생성할지 알려줘야 합니다.
이는 NumPy `RandomState` 객체를 사용함으로써 달성할 수 있는데,
이는 본질적으로 전달될 수 있는 무작위 숫자 생성기 객체입니다.
이는 `np.random`과 대부분 동일한 함수들을 갖고 있습니다;
차이점은 무작위 숫자가 어디서 오는지를 우리가 제어할 수 있다는
것입니다. 다음과 같이 생성합니다:

```python
>>> import numpy as np
>>> rso = np.random.RandomState(230489)
```

여기서 `RandomState` 생성자에 전달된 숫자는 무작위 숫자 생성기의
*시드*입니다. 동일한 시드로 인스턴스화하는 한, `RandomState`
객체는 동일한 순서로 동일한 "무작위" 숫자들을 생성할 것이며,
따라서 재현성을 보장합니다:

```python
>>> rso.rand()
0.5356709186237074
>>> rso.rand()
0.6190581888276206
>>> rso.rand()
0.23143573416770336
>>> rso.seed(230489)
>>> rso.rand()
0.5356709186237074
>>> rso.rand()
0.6190581888276206
```

Earlier, we saw that the constructor took an argument called `rso`.
This `rso` variable is a `RandomState` object that has already been
initialized. I like to make the `RandomState` object an optional
parameter: it is occasionally convenient to not be *forced* to use it,
but I do want to have the *option* of using it (which, if I were to
just use the `np.random` module, I would not be able to do).

So, if the `rso` variable is not given, then the constructor defaults
to `np.random.multinomial`. Otherwise, it uses the multinomial
sampler from the `RandomState` object itself[^rng].

[^rng]: The functions in `np.random` actually do rely on a random
number generator that we can control: NumPy's global random number
generator. You can set the global seed with `np.seed`. There's a
tradeoff to using the global generator vs. a local `RandomState`
object. If you use the global generator, then you don't have to pass
around a `RandomState` object everywhere. However, you also run the
risk of depending on some third party code that also uses the global
generator without your knowledge. If you use a local object, then it
is easier to find out whether there is nondeterminism coming from
somewhere other than your own code.

#### 매개변수란 무엇인가?

`np.random.multinomial`이나 `rso.multinomial` 중 어느 것을
사용할지 결정한 후에는, 샘플링은 단지 적절한 함수를 호출하는
문제입니다. 하지만 우리가 고려할 수 있는 다른 결정이 하나 더
있습니다: 무엇이 매개변수로 간주되는가?

앞서 결과 확률들 $p$가 다항 분포의 매개변수라고 말했습니다.
하지만 누구에게 물어보느냐에 따라, 사건의 수 $n$도 다항 분포의
매개변수가 될 *수* 있습니다. 그렇다면 왜 $n$을 생성자의 인자로
포함하지 않았을까요?

이 질문은 다항 분포에 상대적으로 특화되어 있지만, 실제로
확률 분포를 다룰 때 상당히 자주 나타나며, 답은 정말로 사용
사례에 달려 있습니다. 다항 분포의 경우, 사건의 수가 항상
동일하다고 가정할 수 있나요? 그렇다면 $n$을 생성자의 인자로
전달하는 것이 더 나을 수 있습니다. 그렇지 않다면, 객체 생성
시간에 $n$을 명시하도록 요구하는 것은 매우 제한적일 수 있고,
샘플을 뽑을 때마다 새로운 분포 객체를 만들어야 할 수도 있습니다!

저는 보통 코드에 의해 그렇게 제한받는 것을 좋아하지 않으므로,
생성자의 인자로 갖는 것보다는 `sample` 함수의 인자로 `n`을
갖기로 선택합니다. 대안적 해결책은 `n`을 생성자의 인자로
갖되, 완전히 새로운 객체를 만들지 않고도 `n`의 값을 변경할 수
있도록 하는 메소드들을 포함하는 것일 수 있습니다. 하지만
우리의 목적에는 이 해결책이 아마 과도할 것이므로, `sample`의
인자로만 갖는 것을 고수하겠습니다:

```python
def sample(self, n):
    """Samples draws of `n` events from a multinomial distribution with
    outcome probabilities `self.p`.

    Parameters
    ----------
    n: integer
        The number of total events

    Returns
    -------
    numpy array of length `k`
        The sampled number of occurrences for each outcome

    """
    x = self.rso.multinomial(n, self.p)
    return x
```

### 다항 PMF 평가하기

우리가 생성하는 마법 아이템들의 확률을 명시적으로 계산할 필요는
없지만, 분포의 확률 질량 함수(PMF)나 확률 밀도 함수(PDF)를
계산할 수 있는 함수를 작성하는 것은 거의 항상 좋은 생각입니다.
왜일까요?

한 가지 이유는 테스트에 사용할 수 있기 때문입니다: 많은 샘플을
with our sampling function, then they should approximate the exact PDF
or PMF. If after many samples the approximation is poor or obviously
wrong, then we know there is a bug in our code somewhere.

Another reason to implement the PMF or PDF is that frequently, you
will actually need it later down the line and simply don't realize it
initially. For example, we might want to classify our randomly
generated items as *common*, *uncommon*, and *rare*, depending on how
likely they are to be generated. To determine this, we need to be able to
compute the PMF.

Finally, in many cases, your particular use case will dictate that you
implement the PMF or PDF from the beginning, anyway.

#### The Multinomial PMF Equation

Formally, the multinomial distribution has the following equation:

$$
p(\mathbf{x}; \mathbf{p}) = \frac{(\sum_{i=1}^k x_i)!}{x_1!\cdots{}x_k!}p_1^{x_1}\cdots{}p_k^{x_k}
$$

\noindent where $\mathbf{x}=[x_1, \ldots{}, x_k]$ is a vector of length $k$
specifying the number of times each event happened, and
$\mathbf{p}=[p_1, \ldots{}, p_k]$ is a vector specifying the
probability of each event occurring. As mentioned above, the event
probabilities $\mathbf{p}$ are the *parameters* of the distribution.

The factorials in the equation above can actually be expressed using a
special function, $\Gamma$, called the *gamma function*. When we get
to writing the code, it will be more convenient and efficient to use
the gamma function rather than factorial, so we will rewrite the
equation using $\Gamma$:

$$
p(\mathbf{x}; \mathbf{p}) = \frac{\Gamma((\sum_{i=1}^k x_i)+1)}{\Gamma(x_1+1)\cdots{}\Gamma(x_k+1)}p_1^{x_1}\cdots{}p_k^{x_k}
$$

#### Working with Log Values

Before getting into the actual code needed to implement the equation
above, I want to emphasize one of the the most important design
decisions when writing code with probabilities: working with
log values. What this means is that rather than working directly with
probabilities $p(x)$, we should be working with *log*-probabilities,
$\log{p(x)}$. This is because probabilities can get very small very
quickly, resulting in underflow errors.

To motivate this, consider that probabilities must range between 0 and
1 (inclusive). NumPy has a useful function, `finfo`, that will tell us
the limits of floating point values for our system. For example, on a
64-bit machine, we see that the smallest usable positive number (given
by `tiny`) is:

```python
>>> import numpy as np
>>> np.finfo(float).tiny
2.2250738585072014e-308
```

While that may seem very small, it is not unusual to encounter
probabilities of this magnitude, or even smaller. Moreover, it is a
common operation to multiply probabilities, yet if we try to do this
with very small probabilities, we encounter underflow problems:

```python
>>> tiny = np.finfo(float).tiny
>>> # if we multiply numbers that are too small, we lose all precision
>>> tiny * tiny
0.0
```

However, taking the log can help alleviate this issue because we can
represent a much wider range of numbers with logarithms than we can
normally. Officially, log values range from $-\infty$ to zero. In
practice, they range from the `min` value returned by `finfo`,
which is the smallest number that can be represented, to zero. The
`min` value is *much* smaller than the log of the `tiny` value (which
would be our lower bound if we did not work in log space):

```python
>>> # this is our lower bound normally
>>> np.log(tiny)
-708.39641853226408
>>> # this is our lower bound when using logs
>>> np.finfo(float).min
-1.7976931348623157e+308
```

So, by working with log values, we can greatly expand our range of
representable numbers.  Moreover, we can perform multiplication with logs by
using addition, because $\log(x\cdot{}y) = \log(x) + \log(y)$. Thus, if we do
the multiplication above with logs, we do not have to worry (as much) about
loss of precision due to underflow:

```python
>>> # the result of multiplying small probabilities
>>> np.log(tiny * tiny)
-inf
>>> # the result of adding small log probabilities
>>> np.log(tiny) + np.log(tiny)
-1416.7928370645282
```

Of course, this solution is not a magic bullet. If we need to derive
the number from the logarithm (for example, to add probabilities,
rather than multiply them), then we are back to underflow:

```python
>>> tiny*tiny
0.0
>>> np.exp(np.log(tiny) + np.log(tiny))
0.0
```

Still, doing all our computations with logs can save a lot of
headache. We might be forced to lose that precision if we need to go
back to the original numbers, but we at least maintain *some* information about
the probabilities&mdash;enough to compare them, for example&mdash;that would
otherwise be lost.

#### Writing the PMF Code

Now that we have seen the importance of working with logs, we can
actually write our function to compute the log-PMF:

```python
def log_pmf(self, x):
    """Evaluates the log-probability mass function (log-PMF) of a
    multinomial with outcome probabilities `self.p` for a draw `x`.

    Parameters
    ----------
    x: numpy array of length `k`
        The number of occurrences of each outcome

    Returns
    -------
    The evaluated log-PMF for draw `x`

    """
    # Get the total number of events
    n = np.sum(x)

    # equivalent to log(n!)
    log_n_factorial = gammaln(n + 1)
    # equivalent to log(x1! * ... * xk!)
    sum_log_xi_factorial = np.sum(gammaln(x + 1))

    # If one of the values of self.p is 0, then the corresponding
    # value of self.logp will be -inf. If the corresponding value
    # of x is 0, then multiplying them together will give nan, but
    # we want it to just be 0.
    log_pi_xi = self.logp * x
    log_pi_xi[x == 0] = 0
    # equivalent to log(p1^x1 * ... * pk^xk)
    sum_log_pi_xi = np.sum(log_pi_xi)

    # Put it all together
    log_pmf = log_n_factorial - sum_log_xi_factorial + sum_log_pi_xi
    return log_pmf
```

For the most part, this is a straightforward implementation of the
equation above for the multinomial PMF. The `gammaln` function is from
`scipy.special`, and computes the log-gamma function,
$\log{\Gamma(x)}$. As mentioned above, it is more convenient to use
the gamma function rather than a factorial function; this is because
SciPy gives us a log-gamma function, but not a log-factorial function.
We could have computed a log factorial ourselves, using something like:

```python
log_n_factorial = np.sum(np.log(np.arange(1, n + 1)))
sum_log_xi_factorial = np.sum([np.sum(np.log(np.arange(1, i + 1))) for i in x])
```

but it is easier to understand, easier to code, and more
computationally efficient if we use the gamma function already built
in to SciPy.

There is one edge case that we need to tackle: when one of
our probabilities is zero. When $p_i=0$, then $\log{p_i}=-\infty$.
This would be fine, except for the following behavior when infinity is
multiplied by zero:

```python
>>> # it's fine to multiply infinity by integers...
>>> -np.inf * 2.0
-inf
>>> # ...but things break when we try to multiply by zero
>>> -np.inf * 0.0
nan
```

`nan` means "not a number", and it is almost always a pain to deal
with, because most computations with `nan` result in another
`nan`. So, if we don't handle the case where $p_i=0$ and $x_i=0$, we
will end up with a `nan`. That will get summed with other numbers,
producing another `nan`, which is just not useful. To handle this, we
check specifically for the case when $x_i=0$, and set the resulting
$x_i\cdot{}\log(p_i)$ also to zero.

Let's return for a moment to our discussion of using logs. Even if we
really only need the PMF, and not the log-PMF, it is generally better
to *first* compute it with logs, and then exponentiate it if we
need to:

```python
def pmf(self, x):
    """Evaluates the probability mass function (PMF) of a multinomial
    with outcome probabilities `self.p` for a draw `x`.

    Parameters
    ----------
    x: numpy array of length `k`
        The number of occurrences of each outcome

    Returns
    -------
    The evaluated PMF for draw `x`

    """
    pmf = np.exp(self.log_pmf(x))
    return pmf
```

To further drive home the importance of working with logs,
we can look at an example with just the multinomial:

```python
>>> dist = MultinomialDistribution(np.array([0.25, 0.25, 0.25, 0.25]))
>>> dist.log_pmf(np.array([1000, 0, 0, 0])
-1386.2943611198905
>>> dist.log_pmf(np.array([999, 0, 0, 0])
-1384.9080667587707
```

In this case, we get *extremely* small probabilities (which, you will
notice, are much smaller than the `tiny` value we discussed
above). This is because the fraction in the PMF is huge: 1000
factorial can't even be computed due to overflow. But, the *log* of
the factorial can be:

```python
>>> from scipy.special import gamma, gammaln
>>> gamma(1000 + 1)
inf
>>> gammaln(1000 + 1)
5912.1281784881639
```

If we had tried to compute just the PMF using the `gamma` function, we
would have ended up with `gamma(1000 + 1) / gamma(1000 + 1)`, which
results in a `nan` value (even though we can see that it
should be 1). But, because we do the computation with logarithms, it's
not an issue and we don't need to worry about it!

## Sampling Magical Items, Revisited

Now that we have written our multinomial functions, we can put them to
work to generate our magical items. To do this, we will
create a class called `MagicItemDistribution`, located in the file
`rpg.py`:

```python
class MagicItemDistribution(object):

    # these are the names (and order) of the stats that all magical
    # items will have
    stats_names = ("dexterity", "constitution", "strength",
                   "intelligence", "wisdom", "charisma")

    def __init__(self, bonus_probs, stats_probs, rso=np.random):
        """Initialize a magic item distribution parameterized by `bonus_probs`
        and `stats_probs`.

        Parameters
        ----------
        bonus_probs: numpy array of length m
            The probabilities of the overall bonuses. Each index in
            the array corresponds to the bonus of that amount (e.g.,
            index 0 is +0, index 1 is +1, etc.)

        stats_probs: numpy array of length 6
            The probabilities of how the overall bonus is distributed
            among the different stats. `stats_probs[i]` corresponds to
            the probability of giving a bonus point to the ith stat;
            i.e., the value at `MagicItemDistribution.stats_names[i]`.

        rso: numpy RandomState object (default: np.random)
            The random number generator

        """
        # Create the multinomial distributions we'll be using
        self.bonus_dist = MultinomialDistribution(bonus_probs, rso=rso)
        self.stats_dist = MultinomialDistribution(stats_probs, rso=rso)
```

The constructor to our `MagicItemDistribution` class takes parameters for
the bonus probabilities, the stats probabilities, and the random
number generator. Even though we specified above what we wanted the
bonus probabilities to be, it is generally a good idea to encode
parameters as arguments that are passed in. This leaves open the
possibility of sampling items under different distributions. (For
example, maybe the bonus probabilities would change as the player's
level increases.) We encode the *names* of the stats as a class
variable, `stats_names`, though this could just as easily be another
parameter to the constructor.

As mentioned previously, there are two steps to sampling a magical
item: first sampling the overall bonus, and then sampling the
distribution of the bonus across the stats. As such, we code these
steps as two methods: `_sample_bonus` and `_sample_stats`:

```python
def _sample_bonus(self):
    """Sample a value of the overall bonus.

    Returns
    -------
    integer
        The overall bonus

    """
    # The bonus is essentially just a sample from a multinomial
    # distribution with n=1; i.e., only one event occurs.
    sample = self.bonus_dist.sample(1)

    # `sample` is an array of zeros and a single one at the
    # location corresponding to the bonus. We want to convert this
    # one into the actual value of the bonus.
    bonus = np.argmax(sample)
    return bonus

def _sample_stats(self):
    """Sample the overall bonus and how it is distributed across the
    different stats.

    Returns
    -------
    numpy array of length 6
        The number of bonus points for each stat

    """
    # First we need to sample the overall bonus
    bonus = self._sample_bonus()

    # Then, we use a different multinomial distribution to sample
    # how that bonus is distributed. The bonus corresponds to the
    # number of events.
    stats = self.stats_dist.sample(bonus)
    return stats
```

We *could* have made these a single method&mdash;especially since
`_sample_stats` is the only function that depends on
`_sample_bonus`&mdash;but I have chosen to keep them separate, both
because it makes the sampling routine easier to understand, and
because breaking it up into smaller pieces makes the code easier to
test.

You'll also notice that these methods are prefixed with an underscore,
indicating that they're not really meant to be used outside the
class. Instead, we provide the function `sample`:

```python
def sample(self):
    """Sample a random magical item.

    Returns
    -------
    dictionary
        The keys are the names of the stats, and the values are
        the bonus conferred to the corresponding stat.

    """
    stats = self._sample_stats()
    item_stats = dict(zip(self.stats_names, stats))
    return item_stats
```

The `sample` function does essentially the same thing as
`_sample_stats`, except that it returns a dictionary with the stats'
names as keys. This provides a clean and understandable interface for
sampling items&mdash;it is obvious which stats have how many bonus
points&mdash;but it also keeps open the option of using just
`_sample_stats` if one needs to take many samples and efficiency is
required.

We use a similar design for evaluating the probability of
items. Again, we expose high-level methods `pmf` and `log_pmf` which
take dictionaries of the form produced by `sample`:

```python
def log_pmf(self, item):
    """Compute the log probability of the given magical item.

    Parameters
    ----------
    item: dictionary
        The keys are the names of the stats, and the values are
        the bonuses conferred to the corresponding stat.

    Returns
    -------
    float
        The value corresponding to log(p(item))

    """
    # First pull out the bonus points for each stat, in the
    # correct order, then pass that to _stats_log_pmf.
    stats = np.array([item[stat] for stat in self.stats_names])
    log_pmf = self._stats_log_pmf(stats)
    return log_pmf

def pmf(self, item):
    """Compute the probability the given magical item.

    Parameters
    ----------
    item: dictionary
        The keys are the names of the stats, and the values are
        the bonus conferred to the corresponding stat.

    Returns
    -------
    float
        The value corresponding to p(item)

    """
    return np.exp(self.log_pmf(item))
```

These methods rely on `_stats_log_pmf`, which computes the
probability of the stats (but which takes an array rather than a
dictionary):

```python
def _stats_log_pmf(self, stats):
    """Evaluate the log-PMF for the given distribution of bonus points
    across the different stats.

    Parameters
    ----------
    stats: numpy array of length 6
        The distribution of bonus points across the stats

    Returns
    -------
    float
        The value corresponding to log(p(stats))

    """
    # There are never any leftover bonus points, so the sum of the
    # stats gives us the total bonus.
    total_bonus = np.sum(stats)

    # First calculate the probability of the total bonus
    logp_bonus = self._bonus_log_pmf(total_bonus)

    # Then calculate the probability of the stats
    logp_stats = self.stats_dist.log_pmf(stats)

    # Then multiply them together (using addition, because we are
    # working with logs)
    log_pmf = logp_bonus + logp_stats
    return log_pmf
```

The method `_stats_log_pmf`, in turn, relies on `_bonus_log_pmf`,
which computes the probability of the overall bonus:

```python
def _bonus_log_pmf(self, bonus):
    """Evaluate the log-PMF for the given bonus.

    Parameters
    ----------
    bonus: integer
        The total bonus.

    Returns
    -------
    float
        The value corresponding to log(p(bonus))

    """
    # Make sure the value that is passed in is within the
    # appropriate bounds
    if bonus < 0 or bonus >= len(self.bonus_dist.p):
        return -np.inf

    # Convert the scalar bonus value into a vector of event
    # occurrences
    x = np.zeros(len(self.bonus_dist.p))
    x[bonus] = 1

    return self.bonus_dist.log_pmf(x)
```

We can now create our distribution as follows:

```python
>>> import numpy as np
>>> from rpg import MagicItemDistribution
>>> bonus_probs = np.array([0.0, 0.55, 0.25, 0.12, 0.06, 0.02])
>>> stats_probs = np.ones(6) / 6.0
>>> rso = np.random.RandomState(234892)
>>> item_dist = MagicItemDistribution(bonus_probs, stats_probs, rso=rso)
```

Once created, we can use it to generate a few different items:

```
>>> item_dist.sample()
{'dexterity': 0, 'strength': 0, 'constitution': 0, 
 'intelligence': 0, 'wisdom': 0, 'charisma': 1}
>>> item_dist.sample()
{'dexterity': 0, 'strength': 0, 'constitution': 1, 
 'intelligence': 0, 'wisdom': 2, 'charisma': 0}
>>> item_dist.sample()
{'dexterity': 1, 'strength': 0, 'constitution': 1, 
 'intelligence': 0, 'wisdom': 0, 'charisma': 0}
```

And, if we want, we can evaluate the probability of a sampled item:

```
>>> item = item_dist.sample()
>>> item
{'dexterity': 0, 'strength': 0, 'constitution': 0, 
 'intelligence': 0, 'wisdom': 2, 'charisma': 0}
>>> item_dist.log_pmf(item)
-4.9698132995760007
>>> item_dist.pmf(item)
0.0069444444444444441
```

## Estimating Attack Damage

We've seen one application of sampling: generating
random items that monsters drop. I mentioned earlier that sampling can
also be used when you want to estimate something from the distribution
as a whole, and there are certainly cases in which we could use our
`MagicItemDistribution` to do this. For example, let's say that damage in
our RPG works by rolling some number of D12s (twelve-sided dice). The
player gets to roll one die by default, and then add dice according to
their strength bonus. So, for example, if they have a +2 strength
bonus, they can roll three dice. The damage dealt is then the sum of
the dice.

We might want to know how much damage a player might deal after
finding some number of weapons; e.g., as a factor in setting the
difficulty of monsters. Let's say that after collecting two items, we
want the player to be able to defeat monsters within three hits in
about 50% of the battles. How many hit points should the monster have?

One way to answer this question is through sampling. We can use the
following scheme:

1. Randomly pick a magic item.
2. Based on the item's bonuses, compute the number of dice that will
   be rolled when attacking.
3. Based on the number of dice that will be rolled, generate a sample
   for the damage inflicted over three hits.
4. Repeat steps 1-3 many times. This will result in an approximation
   to the distribution over damage.

### Implementing a Distribution Over Damage

The class `DamageDistribution` (also in `rpg.py`) shows an
implementation of this scheme:

```python
class DamageDistribution(object):

    def __init__(self, num_items, item_dist,
                 num_dice_sides=12, num_hits=1, rso=np.random):
        """Initialize a distribution over attack damage. This object can
        sample possible values for the attack damage dealt over
        `num_hits` hits when the player has `num_items` items, and
        where attack damage is computed by rolling dice with
        `num_dice_sides` sides.

        Parameters
        ----------
        num_items: int
            The number of items the player has.
        item_dist: MagicItemDistribution object
            The distribution over magic items.
        num_dice_sides: int (default: 12)
            The number of sides on each die.
        num_hits: int (default: 1)
            The number of hits across which we want to calculate damage.
        rso: numpy RandomState object (default: np.random)
            The random number generator

        """

        # This is an array of integers corresponding to the sides of a
        # single die.
        self.dice_sides = np.arange(1, num_dice_sides + 1)

        # Create a multinomial distribution corresponding to one of
        # these dice.  Each side has equal probabilities.
        self.dice_dist = MultinomialDistribution(
            np.ones(num_dice_sides) / float(num_dice_sides), rso=rso)

        self.num_hits = num_hits
        self.num_items = num_items
        self.item_dist = item_dist

    def sample(self):
        """Sample the attack damage.

        Returns
        -------
        int
            The sampled damage

        """
        # First, we need to randomly generate items (the number of
        # which was passed into the constructor).
        items = [self.item_dist.sample() for i in xrange(self.num_items)]

        # Based on the item stats (in particular, strength), compute
        # the number of dice we get to roll.
        num_dice = 1 + np.sum([item['strength'] for item in items])

        # Roll the dice and compute the resulting damage.
        dice_rolls = self.dice_dist.sample(self.num_hits * num_dice)
        damage = np.sum(self.dice_sides * dice_rolls)
        return damage
```

The constructor takes as arguments the number of sides the dice have,
how many hits we want to compute damage over, how many items the
player has, a distribution over magic items (of type
`MagicItemDistribution`) and a random state object. By default, we set
`num_dice_sides` to 12 because, while it is technically a parameter,
it is unlikely to change. Similarly, we set `num_hits` to 1 as a
default because a more likely use case is that we just want to take
one sample of the damage for a single hit.

We then implement the actual sampling logic in `sample`. (Note the
structural similarity to `MagicItemDistribution`.)  First, we
generate a set of possible magic items that the player has. Then, we
look at the strength stat of those items, and from that compute the
number of dice to roll. Finally, we roll the dice (again relying on
our trusty multinomial functions) and compute the damage from that.

#### What Happened to Evaluating Probabilities?

You may have noticed that we didn't include a `log_pmf` or `pmf`
function in our `DamageDistribution`. This is because we actually do
not know what the PMF should be! This would be the equation:

$$
\sum_{{item}_1, \ldots{}, {item}_m} p(\mathrm{damage} \vert \mathrm{item}_1,\ldots{},\mathrm{item}_m)p(\mathrm{item}_1)\cdots{}p(\mathrm{item}_m)
$$

What this equation says is that we would need to compute the
probability of every possible damage amount, given every possible set
of $m$ items. We actually *could* compute this through brute force,
but it wouldn't be pretty. This is actually a perfect example of a
case where we want to use sampling to approximate the solution to a
problem that we can't compute exactly (or which would be very
difficult to compute exactly). So, rather than having a method for the
PMF, we'll show in the next section how we can approximate the
distribution with many samples.

### Approximating the Distribution

Now we have the machinery to answer our question from earlier: If the
player has two items, and we want the player to be able to defeat the
monster within three hits 50% of the time, how many hit points should
the monster have?

First, we create our distribution object, using the same `item_dist`
and `rso` that we created earlier:

```python
>>> from rpg import DamageDistribution
>>> damage_dist = DamageDistribution(2, item_dist, num_hits=3, rso=rso)
```

Now we can draw a bunch of samples, and compute the 50th percentile 
(the damage value that is greater than 50% of the samples):

```python
>>> samples = np.array([damage_dist.sample() for i in xrange(100000)])
>>> samples.min()
3
>>> samples.max()
154
>>> np.percentile(samples, 50)
27.0
```

If we were to plot a histogram of how many samples we got for each
amount of damage, it would look something like \aosafigref{500l.sampler.damage}.

\aosafigure[180pt]{sampler-images/damage_distribution.png}{Damage Distribution}{500l.sampler.damage}

There is a pretty wide range of damage that the player could
potentially inflict, but it has a long tail: the 50th percentile is at
27 points, meaning that in half the samples, the player inflicted no
more than 27 points of damage. Thus, if we wanted to use this criteria
for setting monster difficulty, we would give them 27 hit points.
    
## Summary

In this chapter, we've seen how to write code for generating samples
from a non-standard probability distribution, and how to compute the
probabilities for those samples as well. In working through this
example, we've covered several design decisions that are applicable in
the general case:

1. Representing probability distributions using a class, and including
   functions both for sampling and for evaluating the PMF (or PDF).
2. Computing the PMF (or PDF) using logarithms.
3. Generating samples from a random number generator object to enable
   reproducible randomness.
4. Writing functions whose inputs/outputs are clear and understandable
   (e.g., using dictionaries as the output of
   `MagicItemDistribution.sample`) while still exposing the less clear
   but more efficient and purely numeric version of those functions
   <latex>\linebreak</latex> (e.g., `MagicItemDistribution._sample_stats`).

Additionally, we've seen how sampling from a probability distribution
can be useful both for producing single random values (e.g.,
generating a single magical item after defeating a monster) and for
computing information about a distribution that we would otherwise not
know (e.g., discovering how much damage a player with two items is
likely to deal). Almost every type of sampling you might encounter
falls under one of these two categories; the differences only have to
do with what distributions you are sampling from. The general
structure of the code&mdash;independent of those distributions&mdash;remains
the same.
