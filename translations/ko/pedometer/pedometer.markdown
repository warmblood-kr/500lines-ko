title: 현실 세계의 만보계
author: Dessy Daskalov
<markdown>
_Dessy는 직업적으로는 엔지니어이고, 열정적으로는 기업가이며, 마음으로는 개발자입니다. 현재 [Nudge Rewards](http://nudgerewards.com/)의 CTO이자 공동창업자입니다. 팀과 함께 제품을 구축하느라 바쁘지 않을 때는, 다른 사람들에게 코딩을 가르치거나, 토론토 기술 이벤트에 참석하거나 주최하며, [dessydaskalov.com](http://www.dessydaskalov.com/)과 [\@dess_e](https://twitter.com/dess_e)에서 온라인 활동을 하고 있습니다._
</markdown>
## 완벽한 세계

많은 소프트웨어 엔지니어들이 자신의 교육 과정을 되돌아보면, 매우 완벽한 세계에 살았던 즐거움을 기억할 것입니다. 우리는 이상화된 영역에서 잘 정의된 문제들을 해결하는 방법을 배웠습니다.

그리고 나서 우리는 모든 복잡성과 도전 과제가 있는 현실 세계로 던져졌습니다. 현실은 지저분하지만, 그래서 훨씬 더 흥미진진합니다. 온갖 특이한 점들을 가진 실제 문제를 해결할 수 있다면, 사람들에게 진정으로 도움이 되는 소프트웨어를 구축할 수 있습니다.

이 장에서는 표면적으로는 단순해 보이지만, 현실 세계와 실제 사람들이 개입하면 매우 빠르게 복잡해지는 문제를 살펴보겠습니다.

우리는 함께 기본적인 만보계를 구축할 것입니다. 만보계의 이론을 논의하고 코드 밖에서 걸음 수 카운팅 솔루션을 만드는 것부터 시작하겠습니다. 그다음 우리의 솔루션을 코드로 구현할 것입니다. 마지막으로, 사용자가 작업할 수 있는 친근한 인터페이스를 제공하기 위해 코드에 웹 레이어를 추가할 것입니다.

소매를 걷어붙이고 현실 세계의 문제를 풀어낼 준비를 해봅시다.

## 만보계 이론

모바일 기기의 등장은 우리 일상생활에서 점점 더 많은 데이터를 수집하려는 경향을 가져왔습니다. 많은 사람들이 수집하는 데이터 중 하나는 일정 기간 동안 걸은 걸음 수입니다. 이 데이터는 건강 추적, 스포츠 이벤트 훈련에 사용되거나, 데이터 수집과 분석에 열중하는 사람들에게는 단순한 재미를 위해 사용될 수 있습니다. 걸음 수는 만보계를 사용하여 카운트할 수 있으며, 만보계는 종종 하드웨어 가속도계의 데이터를 입력으로 사용합니다.

### 가속도계란 무엇인가?

가속도계는 $x$, $y$, $z$ 방향의 가속도를 측정하는 하드웨어입니다. 많은 사람들이 어디를 가든지 가속도계를 휴대하고 다닙니다. 현재 시장에 나와 있는 거의 모든 스마트폰에 내장되어 있기 때문입니다. $x$, $y$, $z$ 방향은 휴대폰을 기준으로 상대적입니다.

가속도계는 3차원 공간에서 *신호*를 반환합니다. 신호는 시간에 걸쳐 기록된 데이터 포인트의 집합입니다. 신호의 각 구성 요소는 $x$, $y$, 또는 $z$ 방향 중 하나의 가속도를 나타내는 시계열입니다. 시계열의 각 점은 특정 시점에서 해당 방향의 가속도입니다. 가속도는 중력가속도 또는 *g*의 단위로 측정됩니다. 1 *g*는 9.8 $m/s^2$로, 지구에서 중력으로 인한 평균 가속도와 같습니다.

\aosafigref{500l.pedometer.accelerationtotal}는 세 개의 시계열을 가진 가속도계 신호의 예를 보여줍니다.

\aosafigure[333pt]{pedometer-images/acceleration-total.png}{Example acceleration signal}{500l.pedometer.accelerationtotal}

가속도계의 *샘플링 속도*는 종종 조정할 수 있으며, 초당 측정 횟수를 결정합니다. 예를 들어, 샘플링 속도가 100인 가속도계는 매초마다 $x$, $y$, $z$ 시계열 각각에 대해 100개의 데이터 포인트를 반환합니다.

### 걷기에 대해 이야기해보자

사람이 걸을 때, 각 걸음마다 약간씩 튀어 오릅니다. 당신에게서 멀어져 가는 사람의 머리 꼭대기를 지켜보세요. 머리, 몸통, 엉덩이가 부드러운 튕기는 동작으로 동기화되어 있습니다. 사람들이 그리 멀리 튀지는 않지만, 1~2센티미터 정도만 움직여도, 이것은 사람의 걷기 가속도 신호에서 가장 명확하고, 가장 일정하며, 가장 인식하기 쉬운 부분 중 하나입니다.

사람은 각 걸음마다 수직 방향으로 위아래로 튕깁니다. 지구(또는 우주에 떠 있는 다른 큰 질량덩어리)에서 걷고 있다면, 이 튕기는 움직임은 편리하게도 중력과 같은 방향입니다.

우리는 가속도계를 사용하여 위아래 튕기는 움직임을 카운트함으로써 걸음을 세려고 합니다. 휴대폰은 어느 방향으로든 회전할 수 있기 때문에, 아래쪽이 어느 방향인지 알기 위해 중력을 사용할 것입니다. **만보계는 중력 방향의 튕기는 횟수를 카운트하여 걸음을 셀 수 있습니다.**

가속도계가 장착된 스마트폰을 셔츠 주머니에 넣고 걷는 사람을 살펴봅시다(\aosafigref{500l.pedometer.walk1}).

\aosafigure[240pt]{pedometer-images/walk-1.png}{Walking}{500l.pedometer.walk1}

단순화를 위해, 이 사람은:

* $z$ 방향으로 걷고 있고;
* 각 걸음마다 $y$ 방향으로 튕기며;
* 전체 걷기 동안 휴대폰을 같은 방향으로 유지한다고

가정하겠습니다.

우리의 완벽한 세계에서, 걸음 튕김으로 인한 가속도는 $y$ 방향에서 완벽한 사인파를 형성할 것입니다. 사인파의 각 피크는 정확히 하나의 걸음입니다. 걸음 카운팅은 이러한 완벽한 피크들을 세는 문제가 됩니다.

아, 이런 텍스트에서만 경험할 수 있는 완벽한 세계의 즐거움이여! 걱정하지 마세요. 상황이 곧 조금 더 지저분해지고, 훨씬 더 흥미진진해질 것입니다. 우리 세계에 조금 더 현실을 추가해봅시다.

### 완벽한 세계도 자연의 기본 힘들을 가지고 있다

중력은 중력 방향으로 가속도를 발생시키며, 이를 중력 가속도라고 합니다. 이 가속도는 항상 존재하고, 이 장의 목적상 9.8 $m/s^2$로 일정하기 때문에 독특합니다.

스마트폰이 화면을 위로 향한 채 테이블 위에 놓여 있다고 가정해봅시다. 이 방향에서, 우리의 좌표계는 음의 $z$ 방향이 중력이 작용하는 방향이 되도록 설정되어 있습니다. 중력은 우리 휴대폰을 음의 $z$ 방향으로 끌어당기므로, 우리의 가속도계는 *완전히 정지해 있을 때조차* 음의 $z$ 방향으로 9.8 $m/s^2$의 가속도를 기록할 것입니다. 이 방향에서 우리 휴대폰의 가속도계 데이터는 \aosafigref{500l.pedometer.accelerationtotalphonestill}에 나와 있습니다.

\aosafigure[333pt]{pedometer-images/acceleration-total-phone-still.png}{Example accelerometer data at rest}{500l.pedometer.accelerationtotalphonestill}

$x(t)$와 $y(t)$는 0에서 일정하게 유지되는 반면, $z(t)$는 -1 *g*에서 일정함을 주목하세요. 우리의 가속도계는 중력 가속도를 포함한 모든 가속도를 기록합니다.

각 시계열은 해당 방향의 *전체 가속도*를 측정합니다. 전체 가속도는 *사용자 가속도*와 *중력 가속도*의 합입니다.

사용자 가속도는 사용자의 움직임으로 인한 기기의 가속도이며, 휴대폰이 완전히 정지해 있을 때는 0에서 일정합니다. 그러나 사용자가 기기와 함께 움직일 때, 사람이 일정한 가속도로 움직이기는 어렵기 때문에 사용자 가속도는 거의 일정하지 않습니다.

\aosafigure[240pt]{pedometer-images/component-signals-2.png}{Component signals}{500l.pedometer.componentsignals}

걸음을 세기 위해서는, 중력 방향으로 사용자가 만든 튕김에 관심이 있습니다. 즉, 3차원 가속도 신호에서 **중력 방향의 사용자 가속도**를 설명하는 1차원 시계열을 분리하는 데 관심이 있습니다(\aosafigref{500l.pedometer.componentsignals}).

우리의 간단한 예에서, 중력 가속도는 $x(t)$와 $z(t)$에서는 0이고 $y(t)$에서는 9.8 $m/s^2$로 일정합니다. 따라서 전체 가속도 플롯에서, $x(t)$와 $z(t)$는 0 주위에서 변동하는 반면 $y(t)$는 -1 *g* 주위에서 변동합니다. 사용자 가속도 플롯에서는 중력 가속도를 제거했기 때문에 세 시계열 모두 0 주위에서 변동하는 것을 알 수 있습니다. $y_{u}(t)$의 명백한 피크들을 주목하세요. 이것들은 걸음 튕김 때문입니다! 마지막 플롯에서, 중력 가속도 $y_{g}(t)$는 -1 *g*에서 일정하고, $x_{g}(t)$와 $z_{g}(t)$는 0에서 일정합니다.

따라서 우리 예에서, 우리가 관심 있는 중력 방향의 1차원 사용자 가속도 시계열은 $y_{u}(t)$입니다. $y_{u}(t)$가 우리의 완벽한 사인파만큼 부드럽지는 않지만, 피크들을 식별할 수 있고, 이 피크들을 사용하여 걸음을 셀 수 있습니다. 지금까지는 좋습니다. 이제 우리 세계에 더욱 현실을 추가해봅시다.

### 사람은 복잡한 생명체이다

만약 사람이 휴대폰을 어깨에 멘 가방에 넣어 다니는데, 휴대폰이 더 이상한 위치에 있다면 어떨까요? 설상가상으로, 휴대폰이 걷는 도중에 가방 안에서 회전한다면 어떨까요(\aosafigref{500l.pedometer.walk2})?

\aosafigure[133pt]{pedometer-images/walk-2.png}{A more complicated walk}{500l.pedometer.walk2}

아야. 이제 세 구성 요소 모두가 0이 아닌 중력 가속도를 가지므로, 중력 방향의 사용자 가속도가 세 시계열 모두에 분산됩니다. 중력 방향의 사용자 가속도를 결정하려면, 먼저 중력이 어느 방향으로 작용하고 있는지 알아야 합니다. 이를 위해 세 시계열 각각에서 전체 가속도를 사용자 가속도 시계열과 중력 가속도 시계열로 분할해야 합니다(\aosafigref{500l.pedometer.component3}).

\aosafigure[240pt]{pedometer-images/component-signals-3.png}{More complicated component signals}{500l.pedometer.component3}

그러면 각 구성 요소에서 중력 방향에 있는 사용자 가속도 부분을 분리할 수 있어, 중력 방향의 사용자 가속도 시계열만을 얻을 수 있습니다.

이를 다음과 같이 두 단계로 정의해봅시다:

1. 전체 가속도를 사용자 가속도와 중력 가속도로 분할하기.
2. 중력 방향의 사용자 가속도 분리하기.

각 단계를 개별적으로 살펴보고, 수학자 모자를 써봅시다.

### 1. 전체 가속도를 사용자 가속도와 중력 가속도로 분할하기

*필터*라는 도구를 사용하여 전체 가속도 시계열을 사용자 가속도 시계열과 중력 가속도 시계열로 분할할 수 있습니다.

#### 저역 통과 필터와 고역 통과 필터
필터는 신호 처리에서 신호로부터 원하지 않는 구성 요소를 제거하는 데 사용되는 도구입니다.

*저역 통과 필터*는 저주파 신호는 통과시키면서 설정된 임계값보다 높은 신호는 감쇠시킵니다. 반대로, *고역 통과 필터*는 고주파 신호는 통과시키면서 설정된 임계값보다 낮은 신호는 감쇠시킵니다. 음악을 비유로 들면, 저역 통과 필터는 고음을 제거할 수 있고, 고역 통과 필터는 저음을 제거할 수 있습니다.

우리 상황에서, Hz로 측정되는 주파수는 가속도가 얼마나 빨리 변화하는지를 나타냅니다. 일정한 가속도는 0 Hz의 주파수를 가지는 반면, 일정하지 않은 가속도는 0이 아닌 주파수를 가집니다. 이는 우리의 일정한 중력 가속도가 0 Hz 신호인 반면, 사용자 가속도는 그렇지 않다는 것을 의미합니다.

각 구성 요소에 대해, 전체 가속도를 저역 통과 필터에 통과시키면 중력 가속도 시계열만 남게 됩니다. 그다음 전체 가속도에서 중력 가속도를 빼면, 사용자 가속도 시계열을 얻게 됩니다(\aosafigref{500l.pedometer.lowpass}).

\aosafigure[240pt]{pedometer-images/low-pass-filter-a.png}{A low-pass filter}{500l.pedometer.lowpass}

필터에는 수많은 종류가 있습니다. 우리가 사용할 것은 무한 임펄스 응답(IIR) 필터라고 불립니다. 우리가 IIR 필터를 선택한 이유는 낮은 오버헤드와 구현의 용이성 때문입니다. 우리가 선택한 IIR 필터는 다음 공식을 사용하여 구현됩니다:

$$
output_{i} = \alpha_{0}(input_{i}\beta_{0} + input_{i-1}\beta_{1} + input_{i-2}\beta_{2} - output_{i-1}\alpha_{1} - output_{i-2}\alpha_{2})
$$

디지털 필터의 설계는 이 장의 범위를 벗어나지만, 매우 짧은 맛보기 논의는 적절합니다. 이는 잘 연구된, 매력적인 주제로, 수많은 실용적 응용이 있습니다. 디지털 필터는 원하는 주파수나 주파수 범위를 취소하도록 설계할 수 있습니다. 공식의 $\alpha$와 $\beta$ 값은 차단 주파수와 보존하고자 하는 주파수 범위를 기반으로 설정된 계수입니다.

우리는 일정한 중력 가속도를 제외한 모든 주파수를 취소하고 싶으므로, 0.2 Hz보다 높은 주파수를 감쇠시키는 계수를 선택했습니다. 우리가 임계값을 0 Hz보다 약간 높게 설정했다는 것을 주목하세요. 중력은 실제로 0 Hz 가속도를 만들어내지만, 우리의 현실적이고 불완전한 세계는 현실적이고 불완전한 가속도계를 가지고 있으므로, 측정에서 약간의 오차 여지를 허용하고 있습니다.

#### 저역 통과 필터 구현하기

앞선 예를 사용하여 저역 통과 필터 구현을 살펴봅시다. 우리는 다음을 분할할 것입니다:

* $x(t)$를 $x_{g}(t)$와 $x_{u}(t)$로,
* $y(t)$를 $y_{g}(t)$와 $y_{u}(t)$로,
* $z(t)$를 $z_{g}(t)$와 $z_{u}(t)$로.

공식이 작업할 초기값을 가지도록 중력 가속도의 처음 두 값을 0으로 초기화할 것입니다.

$$x_{g}(0) = x_{g}(1) = y_{g}(0) = y_{g}(1) = z_{g}(0) = z_{g}(1) = 0$$

그다음 각 시계열에 대해 필터 공식을 구현할 것입니다.

$$x_{g}(t) = \alpha_{0}(x(t)\beta_{0} + x(t-1)\beta_{1} + x(t-2)\beta_{2} - x_{g}(t-1)\alpha_{1} - x_{g}(t-2)\alpha_{2})$$

$$y_{g}(t) = \alpha_{0}(y(t)\beta_{0} + y(t-1)\beta_{1} + y(t-2)\beta_{2} - y_{g}(t-1)\alpha_{1} - y_{g}(t-2)\alpha_{2})$$

$$z_{g}(t) = \alpha_{0}(z(t)\beta_{0} + z(t-1)\beta_{1} + z(t-2)\beta_{2} - z_{g}(t-1)\alpha_{1} - z_{g}(t-2)\alpha_{2})$$

저역 통과 필터링 후의 결과 시계열은 \aosafigref{500l.pedometer.accelerationgravitational}에 있습니다.

\aosafigure[333pt]{pedometer-images/acceleration-gravitational.png}{Gravitational acceleration}{500l.pedometer.accelerationgravitational}

$x_{g}(t)$와 $z_{g}(t)$는 0 주위에서 맴돌고, $y_{g}(t)$는 매우 빠르게 $-1g$로 떨어집니다. $y_{g}(t)$의 초기 0 값은 공식의 초기화에서 나온 것입니다.

이제 사용자 가속도를 계산하기 위해, 전체 가속도에서 중력 가속도를 뺄 수 있습니다:

$$
x_{u}(t) = x(t) - x_{g}(t)
$$
$$
y_{u}(t) = y(t) - y_{g}(t)
$$
$$
z_{u}(t) = z(t) - z_{g}(t)
$$

결과는 \aosafigref{500l.pedometer.accelerationuser}에서 볼 수 있는 시계열입니다. 우리는 성공적으로 전체 가속도를 사용자 가속도와 중력 가속도로 분할했습니다!

\aosafigure[333pt]{pedometer-images/acceleration-user.png}{Split acceleration}{500l.pedometer.accelerationuser}


### 2. 중력 방향의 사용자 가속도 분리하기

$x_{u}(t)$, $y_{u}(t)$, $z_{u}(t)$는 중력 방향의 움직임뿐만 아니라 사용자의 모든 움직임을 포함합니다. 여기서 우리의 목표는 중력 방향의 사용자 가속도를 나타내는 1차원 시계열을 얻는 것입니다. 이는 각 방향의 사용자 가속도 부분들을 포함할 것입니다.

시작해봅시다. 먼저, 선형대수학 101입니다. 수학자 모자를 아직 벗지 마세요!

#### 내적

좌표를 다룰 때, $x$, $y$, $z$ 좌표의 크기와 방향을 비교하는 데 사용되는 기본 도구 중 하나인 *내적*을 접하지 않고는 멀리 갈 수 없습니다.

내적은 우리를 3차원 공간에서 1차원 공간으로 이동시킵니다(\aosafigref{500l.pedometer.dotproduct}). 두 시계열, 즉 사용자 가속도와 중력 가속도의 내적을 구하면(둘 다 3차원 공간에 있음), 중력 방향의 사용자 가속도 부분을 나타내는 1차원 공간의 단일 시계열이 남게 됩니다. 우리는 이 새로운 시계열을 임의로 $a(t)$라고 부르겠습니다. 모든 중요한 시계열은 이름을 가질 자격이 있으니까요.

\aosafigure[333pt]{pedometer-images/dot-product-explanation.png}{The dot product}{500l.pedometer.dotproduct}


#### 내적 구현하기

우리는 공식 $a(t) = x_{u}(t)x_{g}(t) + y_{u}(t)y_{g}(t) + z_{u}(t)z_{g}(t)$를 사용하여 앞선 예에 대한 내적을 구현할 수 있으며, 이는 1차원 공간에서 $a(t)$를 남겨둡니다(\aosafigref{500l.pedometer.accelerationdotproduct}).

\aosafigure[333pt]{pedometer-images/acceleration-dotproduct.png}{Implementing the dot product}{500l.pedometer.accelerationdotproduct}

이제 $a(t)$에서 걸음이 어디에 있는지 시각적으로 선별할 수 있습니다. 내적은 매우 강력하면서도 아름답게 단순합니다.

### 현실 세계의 솔루션

겉보기에 단순한 문제가 현실 세계와 실제 사람들의 도전 과제를 던져 넣었을 때 얼마나 빠르게 복잡해졌는지 보았습니다. 그러나 우리는 걸음을 세는 것에 훨씬 더 가까워졌고, $a(t)$가 우리의 이상적인 사인파를 닮아가기 시작한 것을 볼 수 있습니다. 하지만 "어느 정도만" 시작한 것입니다. 우리는 여전히 지저분한 $a(t)$ 시계열을 더 부드럽게 만들어야 합니다. 현재 상태의 $a(t)$에는 네 가지 주요 문제(\aosafigref{500l.pedometer.problems})가 있습니다. 각각을 살펴봅시다.

\aosafigure[333pt]{pedometer-images/jumpy-slow-short-bumpy.png}{Jumpy, slow, short, bumpy}{500l.pedometer.problems}


#### 1. 지터링 피크

$a(t)$는 매우 "지터링"입니다. 휴대폰이 각 걸음마다 흔들릴 수 있어 우리 시계열에 고주파 구성 요소가 추가되기 때문입니다. 이 지터링을 노이즈라고 합니다. 수많은 데이터 세트를 연구함으로써, 걸음 가속도가 최대 5 Hz임을 결정했습니다. 5 Hz 이상의 모든 신호를 감쇠시키는 $\alpha$와 $\beta$를 선택하여 저역 통과 IIR 필터를 사용해 노이즈를 제거할 수 있습니다.

#### 2. 느린 피크

샘플링 속도가 100일 때, $a(t)$에 표시된 느린 피크는 1.5초에 걸쳐 있으며, 이는 걸음이 되기에는 너무 느립니다. 충분한 데이터 샘플을 연구한 결과, 우리가 할 수 있는 가장 느린 걸음이 1 Hz 주파수임을 결정했습니다. 더 느린 가속도는 저주파 구성 요소 때문이며, 1 Hz 미만의 모든 신호를 취소하도록 $\alpha$와 $\beta$를 설정하여 고역 통과 IIR 필터를 사용해 다시 제거할 수 있습니다.

#### 3. 짧은 피크

사람이 앱을 사용하거나 통화를 할 때, 가속도계는 중력 방향의 작은 움직임을 등록하여 우리 시계열에서 짧은 피크로 나타납니다. 최소 임계값을 설정하고 $a(t)$가 양의 방향으로 해당 임계값을 넘을 때마다 걸음을 카운트함으로써 이러한 짧은 피크들을 제거할 수 있습니다.

#### 4. 거친 피크

우리 만보계는 다양한 걸음걸이를 가진 많은 사람들을 수용해야 하므로, 사람과 걸음걸이의 큰 샘플 크기를 기반으로 최소 및 최대 걸음 주파수를 설정했습니다. 이는 때때로 약간 너무 많이 또는 너무 적게 필터링할 수 있음을 의미합니다. 보통은 상당히 부드러운 피크를 가지겠지만, 가끔씩 "더 거친" 피크를 얻을 수 있습니다. \aosafigref{500l.pedometer.problems}는 그러한 피크 중 하나를 확대한 것입니다.

임계값에서 거칠함이 발생할 때, 하나의 피크에 대해 너무 많은 걸음을 잘못 카운트할 수 있습니다. 이를 해결하기 위해 *히스테리시스*라는 방법을 사용할 것입니다. 히스테리시스는 과거 입력에 대한 출력의 의존성을 의미합니다. 양의 방향의 임계값 교차와 음의 방향의 0 교차를 카운트할 수 있습니다. 그다음 0 교차 후에 임계값 교차가 발생하는 걸음만 카운트하여, 각 걸음을 한 번만 카운트하도록 보장합니다.

#### 딱 적절한 피크

\aosafigure[333pt]{pedometer-images/acceleration-filtered.png}{Tweaked peaks}{500l.pedometer.accelerationfiltered}

\noindent 이 네 가지 시나리오를 고려함으로써, 지저분한 $a(t)$를 이상적인 사인파에 상당히 가깝게 만들어(\aosafigref{500l.pedometer.accelerationfiltered}) 걸음을 셀 수 있게 되었습니다.

### 요약

문제는 처음 보기에는 단순해 보였습니다. 그러나 현실 세계와 실제 사람들이 우리에게 몇 가지 예상치 못한 문제를 던져주었습니다. 우리가 어떻게 문제를 해결했는지 요약해봅시다:

1. 전체 가속도 $(x(t), y(t), z(t))$로 시작했습니다.
2. 저역 통과 필터를 사용하여 전체 가속도를 사용자 가속도와 중력 가속도, 즉 각각 $(x_{u}(t), y_{u}(t), z_{u}(t))$와 $(x_{g}(t), y_{g}(t), z_{g}(t))$로 분할했습니다.
3. $(x_{u}(t), y_{u}(t), z_{u}(t))$와 $(x_{g}(t), y_{g}(t), z_{g}(t))$의 내적을 구하여 중력 방향의 사용자 가속도 $a(t)$를 얻었습니다.
4. $a(t)$의 고주파 구성 요소를 제거하기 위해 저역 통과 필터를 다시 사용하여 노이즈를 제거했습니다.
5. $a(t)$의 저주파 구성 요소를 취소하기 위해 고역 통과 필터를 사용하여 느린 피크를 제거했습니다.
6. 짧은 피크를 무시하기 위해 임계값을 설정했습니다.
7. 거친 피크로 인한 걸음 중복 카운팅을 피하기 위해 히스테리시스를 사용했습니다.

훈련이나 학술 환경의 소프트웨어 개발자로서, 우리는 완벽한 신호를 받고 해당 신호에서 걸음을 카운트하는 코드를 작성하라는 요청을 받았을 수 있습니다. 그것은 흥미로운 코딩 도전이었을 수 있지만, 실제 상황에서 적용할 수 있는 것은 아니었을 것입니다. 현실에서는 중력과 사람들이 섞여들면서 문제가 조금 더 복잡하다는 것을 보았습니다. 우리는 복잡성을 해결하기 위해 수학적 도구를 사용했고, 현실 세계의 문제를 해결할 수 있었습니다. 이제 우리의 솔루션을 코드로 번역할 때입니다.

## 코드로 뛰어들기

이 장의 목표는 가속도계 데이터를 받아들이고, 데이터를 파싱, 처리, 분석하여 걸음 수, 이동 거리, 경과 시간을 반환하는 Ruby 웹 애플리케이션을 만드는 것입니다.

### 예비 작업

우리의 솔루션은 시계열을 여러 번 필터링해야 합니다. 프로그램 전체에 필터링 코드를 흩뿌리기보다는, 필터링을 담당하는 클래스를 만드는 것이 합리적입니다. 그러면 향상시키거나 수정해야 할 때, 그 하나의 클래스만 변경하면 됩니다. 이 전략을 *관심사의 분리*라고 하며, 프로그램을 각각 하나의 주요 관심사를 가진 구별되는 조각들로 나누는 것을 장려하는 일반적으로 사용되는 설계 원칙입니다. 이는 깨끗하고 유지 관리 가능하며 쉽게 확장 가능한 코드를 작성하는 아름다운 방법입니다. 이 장 전반에 걸쳐 이 아이디어를 여러 번 다시 살펴볼 것입니다.

논리적으로 `Filter` 클래스에 포함된 필터링 코드로 뛰어들어 봅시다.

```ruby
class Filter

  COEFFICIENTS_LOW_0_HZ = {
    alpha: [1, -1.979133761292768, 0.979521463540373],
    beta:  [0.000086384997973502, 0.000172769995947004, 0.000086384997973502]
  }
  COEFFICIENTS_LOW_5_HZ = {
    alpha: [1, -1.80898117793047, 0.827224480562408],
    beta:  [0.095465967120306, -0.172688631608676, 0.095465967120306]
  }
  COEFFICIENTS_HIGH_1_HZ = {
    alpha: [1, -1.905384612118461, 0.910092542787947],
    beta:  [0.953986986993339, -1.907503180919730, 0.953986986993339]
  }

  def self.low_0_hz(data)
    filter(data, COEFFICIENTS_LOW_0_HZ)
  end

  def self.low_5_hz(data)
    filter(data, COEFFICIENTS_LOW_5_HZ)
  end

  def self.high_1_hz(data)
    filter(data, COEFFICIENTS_HIGH_1_HZ)
  end

private

  def self.filter(data, coefficients)
    filtered_data = [0,0]
    (2..data.length-1).each do |i|
      filtered_data << coefficients[:alpha][0] *
                      (data[i]            * coefficients[:beta][0] +
                       data[i-1]          * coefficients[:beta][1] +
                       data[i-2]          * coefficients[:beta][2] -
                       filtered_data[i-1] * coefficients[:alpha][1] -
                       filtered_data[i-2] * coefficients[:alpha][2])
    end
    filtered_data
  end

end
```

프로그램이 시계열을 필터링해야 할 때마다, 필터링하고자 하는 데이터와 함께 `Filter`의 클래스 메서드 중 하나를 호출할 수 있습니다:

* `low_0_hz`는 0 Hz 근처의 신호를 저역 통과 필터링하는 데 사용됩니다
* `low_5_hz`는 5 Hz 이하의 신호를 저역 통과 필터링하는 데 사용됩니다
* `high_1_hz`는 1 Hz 이상의 신호를 고역 통과 필터링하는 데 사용됩니다

각 클래스 메서드는 IIR 필터를 구현하고 결과를 반환하는 `filter`를 호출합니다. 향후 더 많은 필터를 추가하고 싶다면, 이 하나의 클래스만 변경하면 됩니다. 모든 매직 넘버가 상단에 정의되어 있다는 점을 주목하세요. 이는 우리 클래스를 더 읽기 쉽고 이해하기 쉽게 만듭니다.

### 입력 형식

입력 데이터는 안드로이드 폰과 아이폰과 같은 모바일 기기에서 나옵니다. 오늘날 시장에 나와 있는 대부분의 모바일 폰에는 전체 가속도를 기록할 수 있는 가속도계가 내장되어 있습니다. 전체 가속도를 기록하는 입력 데이터 형식을 *결합 형식*이라고 부르겠습니다. 모든 기기는 아니지만 많은 기기가 사용자 가속도와 중력 가속도를 별도로 기록할 수도 있습니다. 이 형식을 *분리 형식*이라고 부르겠습니다. 분리 형식으로 데이터를 반환할 수 있는 기기는 필연적으로 결합 형식으로도 데이터를 반환할 수 있습니다. 그러나 그 반대는 항상 참이 아닙니다. 일부 기기는 결합 형식으로만 데이터를 기록할 수 있습니다. 결합 형식의 입력 데이터는 분리 형식으로 바꾸기 위해 저역 통과 필터를 통과해야 할 것입니다.

우리는 프로그램이 가속도계가 있는 시장의 모든 모바일 기기를 처리하기를 원하므로, 두 형식의 데이터를 모두 받아들여야 할 것입니다. 우리가 받아들일 두 형식을 개별적으로 살펴봅시다.

#### 결합 형식

결합 형식의 데이터는 시간에 걸친 $x$, $y$, $z$ 방향의 전체 가속도입니다. $x$, $y$, $z$ 값들은 쉼표로 구분되고, 단위 시간당 샘플들은 세미콜론으로 구분됩니다.

$$
x_1,y_1,z_1; \ldots x_n,y_n,z_n;
$$

#### 분리 형식

분리 형식은 시간에 걸친 $x$, $y$, $z$ 방향의 사용자 가속도와 중력 가속도를 반환합니다. 사용자 가속도 값들은 중력 가속도 값들과 파이프로 구분됩니다.

$$
x^{u}_1,y^{u}_1,z^{u}_1 \vert x^{g}_1,y^{g}_1,z^{g}_1; \ldots x^{u}_n,y^{u}_n,z^{u}_n \vert x^{g}_n,y^{g}_n,z^{g}_n;
$$

### 다중 입력 형식이 있지만 표준은 없다

다중 입력 형식을 다루는 것은 흔한 프로그래밍 문제입니다. 전체 프로그램이 두 형식 모두와 작동하기를 원한다면, 데이터를 다루는 모든 코드 조각이 두 형식을 모두 처리하는 방법을 알아야 할 것입니다. 이는 매우 빠르게 매우 지저분해질 수 있으며, 특히 세 번째(또는 네 번째, 다섯 번째, 또는 백 번째) 입력 형식이 추가되면 더욱 그렇습니다.

#### 표준 형식

이를 처리하는 가장 깨끗한 방법은 두 입력 형식을 가능한 한 빨리 표준 형식에 맞추어, 프로그램의 나머지 부분이 이 새로운 표준 형식과 작동하도록 하는 것입니다. 우리의 솔루션은 사용자 가속도와 중력 가속도를 별도로 작업해야 하므로, 표준 형식은 두 가속도로 분할되어야 할 것입니다(\aosafigref{500l.pedometer.standardformat}).

\aosafigure[240pt]{pedometer-images/standard-format.png}{Standard format}{500l.pedometer.standardformat}

표준 형식을 사용하면 각 요소가 특정 시점의 가속도를 나타내므로 시계열을 저장할 수 있습니다. 이를 배열의 배열의 배열로 정의했습니다. 이 양파를 껍질을 벗겨봅시다.

* 첫 번째 배열은 모든 데이터를 담기 위한 래퍼일 뿐입니다.
* 두 번째 배열 집합은 취해진 데이터 샘플당 하나의 배열을 포함합니다. 샘플링 속도가 100이고 10초 동안 데이터를 샘플링한다면, 이 두 번째 집합에 $100 * 10$, 즉 1000개의 배열이 있을 것입니다.
* 세 번째 배열 집합은 두 번째 집합 내에 포함된 배열 쌍입니다. 둘 다 $x$, $y$, $z$ 방향의 가속도 데이터를 포함하며; 첫 번째는 사용자 가속도를, 두 번째는 중력 가속도를 나타냅니다.

### 파이프라인

우리 시스템의 입력은 가속도계의 데이터, 걷기를 수행하는 사용자에 대한 정보(성별, 보폭 등), 그리고 시험 걷기 자체에 대한 정보(샘플링 속도, 실제로 걸은 걸음 수 등)가 될 것입니다. 우리 시스템은 신호 처리 솔루션을 적용하고, 계산된 걸음 수, 실제 걸음과 계산된 걸음 사이의 차이, 이동 거리, 경과 시간을 출력할 것입니다. 입력에서 출력까지의 전체 과정은 파이프라인으로 볼 수 있습니다(\aosafigref{500l.pedometer.pipeline}).

\aosafigure[240pt]{pedometer-images/pipeline.png}{The pipeline}{500l.pedometer.pipeline}

관심사의 분리 정신에 따라, 파이프라인의 각각의 구별되는 구성 요소&mdash;파싱, 처리, 분석&mdash;에 대한 코드를 개별적으로 작성할 것입니다.

### 파싱

가능한 한 빨리 데이터를 표준 형식으로 만들고 싶으므로, 두 개의 알려진 입력 형식을 취하고 이를 표준 출력 형식으로 변환할 수 있는 파서를 파이프라인의 첫 번째 구성 요소로 작성하는 것이 합리적입니다. 우리의 표준 형식은 사용자 가속도와 중력 가속도를 분리하므로, 데이터가 결합 형식이라면 파서가 먼저 저역 통과 필터를 통과시켜 표준 형식으로 변환해야 할 것입니다.

\aosafigure[240pt]{pedometer-images/input-data-workflow-1.png}{Initial workflow}{500l.pedometer.input1}

향후 다른 입력 형식을 추가해야 한다면, 수정해야 할 코드는 이 파서뿐입니다. 관심사를 다시 한 번 분리하여, 파싱을 처리할 `Parser` 클래스를 만들어봅시다.

```ruby
class Parser

  attr_reader :parsed_data

  def self.run(data)
    parser = Parser.new(data)
    parser.parse
    parser
  end

  def initialize(data)
    @data = data
  end

  def parse
    @parsed_data = @data.to_s.split(';').map { |x| x.split('|') }
                   .map { |x| x.map { |x| x.split(',').map(&:to_f) } }

    unless @parsed_data.map { |x| x.map(&:length).uniq }.uniq == [[3]]
      raise 'Bad Input. Ensure data is properly formatted.'
    end

    if @parsed_data.first.count == 1
      filtered_accl = @parsed_data.map(&:flatten).transpose.map do |total_accl|
        grav = Filter.low_0_hz(total_accl)
        user = total_accl.zip(grav).map { |a, b| a - b }
        [user, grav]
      end

      @parsed_data = @parsed_data.length.times.map do |i|
        user = filtered_accl.map(&:first).map { |elem| elem[i] }
        grav = filtered_accl.map(&:last).map { |elem| elem[i] }
        [user, grav]
      end
    end
  end

end
```

`Parser`는 클래스 레벨 `run` 메서드와 이니셜라이저를 가지고 있습니다. 이는 여러 번 사용할 패턴이므로 논의할 가치가 있습니다. 이니셜라이저는 일반적으로 객체를 설정하는 데 사용되어야 하며, 많은 작업을 수행해서는 안 됩니다. `Parser`의 이니셜라이저는 단순히 결합 또는 분리 형식의 `data`를 받아 인스턴스 변수 `@data`에 저장합니다. `parse` 인스턴스 메서드는 내부적으로 `@data`를 사용하고, 파싱의 무거운 작업을 수행하여 결과를 표준 형식으로 `@parsed_data`에 설정합니다. 우리의 경우, `parse`를 즉시 호출하지 않고 `Parser` 인스턴스를 인스턴스화해야 할 필요가 전혀 없습니다. 따라서 `Parser`의 인스턴스를 인스턴스화하고, 그것에 `parse`를 호출하며, 객체의 인스턴스를 반환하는 편리한 클래스 레벨 `run` 메서드를 추가합니다. 이제 입력 데이터를 `run`에 전달하여 `@parsed_data`가 이미 설정된 `Parser`의 인스턴스를 받을 것임을 알 수 있습니다.

열심히 작업하는 `parse` 메서드를 살펴봅시다. 과정의 첫 번째 단계는 문자열 데이터를 가져와 수치 데이터로 변환하여 배열의 배열의 배열을 만드는 것입니다. 익숙하게 들리나요? 다음으로 하는 일은 형식이 예상된 대로인지 확인하는 것입니다. 최내부 배열당 정확히 세 개의 요소가 없다면, 예외를 던집니다. 그렇지 않으면 계속 진행합니다.

이 단계에서 두 형식 간의 `@parsed_data` 차이를 주목하세요. *결합 형식*에서는 정확히 *하나의* 배열의 배열들을 포함합니다:

$$
[[[x_1, y_1, z_1]], \ldots [[x_n, y_n, z_n]]
$$

*분리 형식*에서는 정확히 *두 개의* 배열의 배열들을 포함합니다:

$$[[[x_{u}^1,y_{u}^1,z_{u}^1], [x_{g}^1,y_{g}^1,z_{g}^1]], ... [[x_{u}^n,y_{u}^n,z_{u}^n], [x_{g}^n,y_{g}^n,z_{g}^n]]]$$

분리 형식은 이 작업 후에 이미 우리가 원하는 표준 형식입니다. 놀랍습니다. 그러나 데이터가 결합되어 있다면(또는 동등하게, 분리 형식이 두 개를 가지는 곳에 정확히 하나의 배열을 가진다면), 두 개의 루프로 진행합니다. 첫 번째 루프는 `:low_0_hz` 타입으로 `Filter`를 사용하여 전체 가속도를 중력 가속도와 사용자 가속도로 분할하고, 두 번째 루프는 데이터를 표준 형식으로 재구성합니다.

`parse`는 결합 또는 분리 데이터로 시작했는지에 관계없이 표준 형식의 데이터를 보유하는 `@parsed_data`를 남겨둡니다. 다행입니다!

프로그램이 더 정교해짐에 따라, 개선할 수 있는 한 영역은 더 구체적인 오류 메시지로 예외를 던져 사용자의 삶을 더 쉽게 만드는 것입니다. 이를 통해 일반적인 입력 형식 문제를 더 빠르게 추적할 수 있습니다.

### 처리

우리가 정의한 솔루션을 기반으로, 걸음을 세기 전에 파싱된 데이터에 대해 코드가 몇 가지 일을 수행해야 합니다:

1. 내적을 사용하여 중력 방향의 움직임을 분리하기.
2. 저역 통과 필터에 이어 고역 통과 필터로 지터링(고주파) 및 느린(저주파) 피크를 제거하기.

짧고 거친 피크는 걸음 카운팅 중에 피함으로써 처리할 것입니다.

이제 데이터가 표준 형식으로 되어 있으므로, 걸음을 세기 위해 분석할 수 있는 상태로 만들기 위해 처리할 수 있습니다(\aosafigref{500l.pedometer.input2}).

\aosafigure[166pt]{pedometer-images/input-data-workflow-2.png}{Processing}{500l.pedometer.input2}

처리의 목적은 표준 형식의 데이터를 가져와 점진적으로 정리하여 이상적인 사인파에 가능한 한 가까운 상태로 만드는 것입니다. 내적을 구하는 것과 필터링이라는 두 가지 처리 작업은 상당히 구별되지만, 둘 다 데이터를 처리하기 위한 것이므로 `Processor`라는 하나의 클래스를 만들 것입니다.

```ruby
class Processor

  attr_reader :dot_product_data, :filtered_data

  def self.run(data)
    processor = Processor.new(data)
    processor.dot_product
    processor.filter
    processor
  end

  def initialize(data)
    @data = data
  end

  def dot_product
    @dot_product_data = @data.map do |x|
      x[0][0] * x[1][0] + x[0][1] * x[1][1] + x[0][2] * x[1][2]
    end
  end

  def filter
    @filtered_data = Filter.low_5_hz(@dot_product_data)
    @filtered_data = Filter.high_1_hz(@filtered_data)
  end

end
```

다시, `run`과 `initialize` 메서드 패턴을 봅니다. `run`은 두 개의 프로세서 메서드인 `dot_product`와 `filter`를 직접 호출합니다. 각 메서드는 두 가지 처리 작업 중 하나를 수행합니다. `dot_product`는 중력 방향의 움직임을 분리하고, `filter`는 지터링과 느린 피크를 제거하기 위해 저역 통과 및 고역 통과 필터를 순서대로 적용합니다.

### 만보계 기능

만보계를 사용하는 사람에 대한 정보가 제공된다면, 걸음 수보다 더 많은 것을 측정할 수 있습니다. 우리의 만보계는 **걸은 거리**와 **경과 시간**뿐만 아니라 **걸음 수**도 측정할 것입니다.

### 걸은 거리

모바일 만보계는 일반적으로 한 사람이 사용합니다. 걷기 중 이동한 거리는 걸음 수에 그 사람의 보폭 길이를 곱해서 계산됩니다. 보폭 길이를 알 수 없다면, 성별과 키 같은 선택적 사용자 정보를 사용하여 근사치를 구할 수 있습니다. 이 관련 정보를 캡슐화하는 `User` 클래스를 만들어봅시다.

```ruby
class User

  GENDER      = ['male', 'female']
  MULTIPLIERS = {'female' => 0.413, 'male' => 0.415}
  AVERAGES    = {'female' => 70.0,  'male' => 78.0}

  attr_reader :gender, :height, :stride

  def initialize(gender = nil, height = nil, stride = nil)
    @gender = gender.to_s.downcase unless gender.to_s.empty?
    @height = Float(height) unless height.to_s.empty?
    @stride = Float(stride) unless stride.to_s.empty?

    raise 'Invalid gender' if @gender && !GENDER.include?(@gender)
    raise 'Invalid height' if @height && (@height <= 0)
    raise 'Invalid stride' if @stride && (@stride <= 0)

    @stride ||= calculate_stride
  end

private

  def calculate_stride
    if gender && height
      MULTIPLIERS[@gender] * height
    elsif height
      height * (MULTIPLIERS.values.reduce(:+) / MULTIPLIERS.size)
    elsif gender
      AVERAGES[gender]
    else
      AVERAGES.values.reduce(:+) / AVERAGES.size
    end
  end

end
```

클래스 상단에서, 전체에 걸쳐 매직 넘버와 문자열을 하드코딩하는 것을 피하기 위해 상수를 정의합니다. 이 논의의 목적상, `MULTIPLIERS`와 `AVERAGES`의 값들이 다양한 사람들의 큰 샘플 크기로부터 결정되었다고 가정하겠습니다.

이니셜라이저는 `gender`, `height`, `stride`를 선택적 인수로 받습니다. 선택적 매개변수가 전달되면, 이니셜라이저는 일부 데이터 포맷팅 후 같은 이름의 인스턴스 변수를 설정합니다. 유효하지 않은 값에 대해서는 예외를 발생시킵니다.

모든 선택적 매개변수가 제공되더라도, 입력된 보폭이 우선합니다. 제공되지 않으면, `calculate_stride` 메서드가 사용자에게 가능한 한 가장 정확한 보폭 길이를 결정합니다. 이는 `if` 문으로 수행됩니다:

* 보폭 길이를 계산하는 가장 정확한 방법은 유효한 성별과 키가 있는 경우 사람의 키와 성별에 기반한 승수를 사용하는 것입니다.
* 사람의 키는 성별보다 보폭의 더 나은 예측 인자입니다. 키는 있지만 성별이 없다면, 키에 `MULTIPLIERS`의 두 값의 평균을 곱할 수 있습니다.
* 성별만 있다면, `AVERAGES`의 평균 보폭 길이를 사용할 수 있습니다.
* 마지막으로, 아무것도 없다면, `AVERAGES`의 두 값의 평균을 취하여 보폭으로 사용할 수 있습니다.

 `if` 문에서 아래로 갈수록 보폭 길이의 정확도가 떨어진다는 점을 주목하세요. 어떤 경우든, `User` 클래스는 가능한 한 최선으로 보폭 길이를 결정합니다.

### 경과 시간

이동에 소요된 시간은 우리 `Processor`의 `@parsed_data`에 있는 데이터 샘플 수를 기기의 샘플링 속도로 나누어 측정됩니다(있는 경우). 속도는 사용자보다는 시험 걷기 자체와 더 관련이 있고, `User` 클래스는 실제로 샘플링 속도를 알 필요가 없으므로, 매우 작은 `Trial` 클래스를 만들기에 좋은 때입니다.

```ruby
class Trial

  attr_reader :name, :rate, :steps

  def initialize(name, rate = nil, steps = nil)
    @name  = name.to_s.delete(' ')
    @rate  = Integer(rate.to_s) unless rate.to_s.empty?
    @steps = Integer(steps.to_s) unless steps.to_s.empty?

    raise 'Invalid name'  if @name.empty?
    raise 'Invalid rate'  if @rate && (@rate <= 0)
    raise 'Invalid steps' if @steps && (@steps < 0)
  end

end
```

`Trial`의 모든 속성 리더는 전달된 매개변수를 기반으로 이니셜라이저에서 설정됩니다:

* `name`은 특정 시험의 이름으로, 서로 다른 시험들을 구별하는 데 도움이 됩니다.
* `rate`는 시험 중 가속도계의 샘플링 속도입니다.
* `steps`는 실제로 걸은 걸음 수를 설정하는 데 사용되어, 사용자가 실제로 걸은 걸음과 우리 프로그램이 센 걸음 사이의 차이를 기록할 수 있습니다.

`User` 클래스와 마찬가지로, 일부 정보는 선택적입니다. 시험의 세부 정보가 있다면 입력할 기회가 주어집니다. 그러한 세부 정보가 없다면, 우리 프로그램은 이동에 소요된 시간과 같은 추가 결과 계산을 건너뜁니다. `User` 클래스와의 또 다른 유사점은 유효하지 않은 값의 방지입니다.

### 걸음 수

이제 코드로 걸음 카운팅 전략을 구현할 때입니다. 지금까지 우리는 중력 방향의 사용자 가속도를 나타내는 깨끗한 시계열인 `@filtered_data`를 포함하는 `Processor` 클래스를 가지고 있습니다. 또한 사용자와 시험에 대한 필요한 정보를 제공하는 클래스들도 있습니다. 빠진 것은 `User`와 `Trial`의 정보로 `@filtered_data`를 분석하고, 걸음을 세고, 거리를 측정하고, 시간을 측정하는 방법입니다.

프로그램의 분석 부분은 `Processor`의 데이터 조작과 다르고, `User`와 `Trial` 클래스의 정보 수집 및 집계와도 다릅니다. 이 데이터 분석을 수행할 `Analyzer`라는 새로운 클래스를 만들어봅시다.

```ruby
class Analyzer

  THRESHOLD = 0.09

  attr_reader :steps, :delta, :distance, :time

  def self.run(data, user, trial)
    analyzer = Analyzer.new(data, user, trial)
    analyzer.measure_steps
    analyzer.measure_delta
    analyzer.measure_distance
    analyzer.measure_time
    analyzer
  end

  def initialize(data, user, trial)
    @data  = data
    @user  = user
    @trial = trial
  end

  def measure_steps
    @steps = 0
    count_steps = true

    @data.each_with_index do |data, i|
      if (data >= THRESHOLD) && (@data[i-1] < THRESHOLD)
        next unless count_steps

        @steps += 1
        count_steps = false
      end

      count_steps = true if (data < 0) && (@data[i-1] >= 0)
    end
  end

  def measure_delta
    @delta = @steps - @trial.steps if @trial.steps
  end

  def measure_distance
    @distance = @user.stride * @steps
  end

  def measure_time
    @time = @data.count/@trial.rate if @trial.rate
  end

end
```

`Analyzer`에서 처음 하는 일은 짧은 피크를 걸음으로 카운트하는 것을 피하는 데 사용할 `THRESHOLD` 상수를 정의하는 것입니다. 이 논의의 목적상, 수많은 다양한 데이터 세트를 분석하여 그러한 데이터 세트의 가장 큰 수를 수용하는 임계값을 결정했다고 가정하겠습니다. 임계값은 결국 동적이 될 수 있고 계산된 걸음 대 실제 걸음을 기반으로 다양한 사용자에 따라 달라질 수 있습니다. 학습 알고리즘이라고 할 수 있겠죠.

`Analyzer`의 이니셜라이저는 `data` 매개변수와 `User` 및 `Trial`의 인스턴스를 받아, 인스턴스 변수 `@data`, `@user`, `@trial`을 전달된 매개변수로 설정합니다. `run` 메서드는 `measure_steps`, `measure_delta`, `measure_distance`, `measure_time`을 호출합니다. 각 메서드를 살펴봅시다.

#### `measure_steps`

드디어! 걸음 카운팅 앱의 걸음 카운팅 부분입니다. `measure_steps`에서 처음 하는 일은 두 변수를 초기화하는 것입니다:

* `@steps`는 걸음 수를 카운트하는 데 사용됩니다.
* `count_steps`는 특정 시점에서 걸음을 카운트할 수 있는지 결정하는 히스테리시스에 사용됩니다.

그다음 `@processor.filtered_data`를 반복합니다. 현재 값이 `THRESHOLD`보다 크거나 같고, 이전 값이 `THRESHOLD`보다 작다면, 양의 방향으로 임계값을 넘은 것이며, 이는 걸음을 나타낼 수 있습니다. `unless` 문은 `count_steps`가 `false`인 경우 다음 데이터 포인트로 건너뛰며, 이는 해당 피크에 대해 이미 걸음을 카운트했음을 나타냅니다. 그렇지 않다면, `@steps`를 1 증가시키고 `count_steps`를 `false`로 설정하여 해당 피크에 대해 더 이상 걸음이 카운트되지 않도록 합니다. 다음 `if` 문은 시계열이 음의 방향으로 $x$-축을 넘으면 `count_steps`를 true로 설정하고, 다음 피크로 이동합니다.

여기에 있습니다. 프로그램의 걸음 카운팅 부분입니다! `Processor` 클래스가 시계열을 정리하고 거짓 걸음 카운팅을 야기할 주파수들을 제거하는 많은 작업을 했으므로, 실제 걸음 카운팅 구현은 복잡하지 않습니다.

걷기에 대한 전체 시계열을 메모리에 저장한다는 점은 주목할 가치가 있습니다. 시험들은 모두 짧은 걷기이므로 현재로서는 문제가 되지 않지만, 결국 대량의 데이터가 있는 긴 걷기를 분석하고 싶을 것입니다. 이상적으로는 데이터를 스트리밍하여 시계열의 매우 작은 부분만 메모리에 저장하고 싶을 것입니다. 이를 염두에 두고, 현재 데이터 포인트와 그 이전 데이터 포인트만 필요하도록 하는 작업을 했습니다. 또한 불린 값을 사용하여 히스테리시스를 구현했으므로, 0에서 $x$-축을 넘었는지 확인하기 위해 시계열에서 뒤로 돌아볼 필요가 없습니다.

제품의 향후 반복 가능성을 고려하는 것과 모든 상상 가능한 제품 방향에 대해 솔루션을 과도하게 엔지니어링하는 것 사이에는 미묘한 균형이 있습니다. 이 경우, 가까운 미래에 더 긴 걷기를 처리해야 할 것이라고 가정하는 것이 합리적이며, 걸음 카운팅에서 이를 고려하는 비용은 상당히 낮습니다.

#### `measure_delta`

시험에서 걷기 중 실제로 걸은 걸음 수를 제공한다면, `measure_delta`는 계산된 걸음과 실제 걸음 사이의 차이를 반환할 것입니다.

#### `measure_distance`

거리는 사용자의 보폭에 걸음 수를 곱하여 측정됩니다. 거리는 걸음 수에 의존하므로, `measure_distance` 전에 `measure_steps`가 호출되어야 합니다.

#### `measure_time`

샘플링 속도가 있는 한, 시간은 `filtered_data`의 총 샘플 수를 샘플링 속도로 나누어 계산됩니다. 따라서 시간은 초 단위로 계산됩니다.

### 파이프라인으로 모두 연결하기

`Parser`, `Processor`, `Analyzer` 클래스들은 개별적으로도 유용하지만, 함께일 때 확실히 더 좋습니다. 우리 프로그램은 앞서 소개한 파이프라인을 실행하기 위해 종종 이들을 사용할 것입니다. 파이프라인이 자주 실행되어야 하므로, 이를 실행할 `Pipeline` 클래스를 만들 것입니다. \newpage

```ruby
class Pipeline

  attr_reader :data, :user, :trial, :parser, :processor, :analyzer

  def self.run(data, user, trial)
    pipeline = Pipeline.new(data, user, trial)
    pipeline.feed
    pipeline
  end

  def initialize(data, user, trial)
    @data  = data
    @user  = user
    @trial = trial
  end

  def feed
    @parser    = Parser.run(@data)
    @processor = Processor.run(@parser.parsed_data)
    @analyzer  = Analyzer.run(@processor.filtered_data, @user, @trial)
  end

end
```

이제 익숙한 `run` 패턴을 사용하고 `Pipeline`에 가속도계 데이터와 `User` 및 `Trial`의 인스턴스를 제공합니다. `feed` 메서드는 파이프라인을 구현하며, 이는 가속도계 데이터로 `Parser`를 실행하고, 그다음 파서의 파싱된 데이터를 사용하여 `Processor`를 실행하고, 마지막으로 프로세서의 필터링된 데이터를 사용하여 `Analyzer`를 실행하는 것을 수반합니다. `Pipeline`은 `@parser`, `@processor`, `@analyzer` 인스턴스 변수들을 유지하므로, 프로그램이 앱을 통한 표시 목적으로 그러한 객체들의 정보에 접근할 수 있습니다.

## 친근한 인터페이스 추가하기

우리 프로그램에서 가장 노동 집약적인 부분을 지나왔습니다. 다음으로, 사용자에게 즐거운 형식으로 데이터를 제시할 웹 앱을 구축할 것입니다. 웹 앱은 자연스럽게 데이터 처리와 데이터 표시를 분리합니다. 코드 전에 사용자의 관점에서 앱을 살펴봅시다.

### 사용자 시나리오

사용자가 `/uploads`로 이동하여 앱에 처음 들어왔을 때, 기존 데이터의 테이블과 가속도계 출력 파일 및 시험과 사용자 정보를 업로드하여 새 데이터를 제출하는 양식을 봅니다(\aosafigref{500l.pedometer.app1}).

\aosafigure[240pt]{pedometer-images/app1.png}{Upload view}{500l.pedometer.app1}

양식을 제출하면 데이터를 파일 시스템에 저장하고, 파싱, 처리, 분석한 후 테이블의 새 항목과 함께 `/uploads`로 다시 리디렉션됩니다.

항목의 **Detail** 링크를 클릭하면 사용자에게 \aosafigref{500l.pedometer.app3}의 다음 뷰가 제시됩니다.

\aosafigure[240pt]{pedometer-images/app3.png}{Detail view}{500l.pedometer.app3}

제시된 정보에는 업로드 양식을 통해 사용자가 입력한 값, 우리 프로그램이 계산한 값, 내적 연산 후의 시계열 그래프, 그리고 필터링 후의 시계열 그래프가 포함됩니다. 사용자는 *Back to Uploads* 링크를 사용하여 `/uploads`로 다시 이동할 수 있습니다.

위에서 설명한 기능이 기술적으로 우리에게 무엇을 의미하는지 살펴봅시다. 아직 가지고 있지 않은 두 가지 주요 구성 요소가 필요할 것입니다:

1. 사용자 입력 데이터를 저장하고 검색하는 방법.
2. 기본 인터페이스가 있는 웹 애플리케이션.

이 두 요구 사항을 각각 살펴봅시다.

### 1. 데이터 저장 및 검색

앱은 파일 시스템에 입력 데이터를 저장하고, 파일 시스템에서 데이터를 검색해야 합니다. 이를 위해 `Upload` 클래스를 만들 것입니다. 이 클래스는 파일 시스템만을 다루고 만보계의 구현과 직접 관련이 없으므로, 간결성을 위해 생략했지만, 기본 기능을 논의할 가치가 있습니다. `Upload` 클래스는 파일 시스템 접근 및 검색을 위한 세 가지 클래스 레벨 메서드를 가지고 있으며, 모두 `Upload`의 하나 이상의 인스턴스를 반환합니다:

* `create`는 사용자 및 시험 정보와 함께 파일을 받습니다. 사용자 및 시험 정보를 포함하도록 생성하는 파일명으로 파일을 파일 시스템에 저장합니다. `@file_path`, `@user`, `@trial` 인스턴스 변수들은 각각 파일 경로, 사용자 객체, 시험 객체에 대한 접근을 허용합니다.
* `find`는 파일 경로를 받아 `Upload`의 인스턴스를 반환합니다.
* `all`은 파일 시스템의 각 가속도계 데이터 파일에 대해 하나씩 `Upload` 인스턴스들의 배열을 반환합니다.

#### Upload에서의 관심사 분리

다시 한번, 우리는 프로그램에서 관심사를 분리하는 것이 현명했습니다. 저장 및 검색과 관련된 모든 코드는 `Upload` 클래스에 포함되어 있습니다. 애플리케이션이 성장하면서, 파일 시스템에 모든 것을 저장하기보다는 데이터베이스를 사용하고 싶을 가능성이 높습니다. 그때가 되면, `Upload` 클래스만 변경하면 됩니다. 이는 리팩터링을 간단하고 깨끗하게 만듭니다.

향후에는 `User`와 `Trial` 객체들을 데이터베이스에 저장할 수 있습니다. 그러면 `Upload`의 `create`, `find`, `all` 메서드들이 `User`와 `Trial`에도 관련될 것입니다. 이는 일반적인 데이터 저장 및 검색을 다루는 자체 클래스로 그것들을 리팩터링할 가능성이 높다는 것을 의미하며, `User`, `Trial`, `Upload` 클래스들 각각이 그 클래스로부터 상속받을 것입니다. 결국 그 클래스에 헬퍼 쿼리 메서드들을 추가하고, 거기에서 계속 구축해나갈 수 있습니다.

### 2. 웹 애플리케이션 구축

웹 앱은 수없이 많이 구축되어 왔으므로, 오픈 소스 커뮤니티의 중요한 작업을 활용하고 지루한 배관 작업을 대신해줄 기존 프레임워크를 사용할 것입니다. Sinatra 프레임워크가 바로 그런 일을 합니다. 이 도구의 표현으로는, Sinatra는 "Ruby에서 웹 애플리케이션을 빠르게 생성하기 위한 DSL"입니다. 완벽합니다.

웹 앱은 HTTP 요청에 응답해야 하므로, HTTP 메서드와 URL의 각 조합에 대한 라우트와 관련 코드 블록을 정의하는 파일이 필요할 것입니다. 이를 `pedometer.rb`라고 부르겠습니다.

```ruby
get '/uploads' do
  @error = "A #{params[:error]} error has occurred." if params[:error]
  @pipelines = Upload.all.inject([]) do |a, upload|
    a << Pipeline.run(File.read(upload.file_path), upload.user, upload.trial)
    a
  end

  erb :uploads
end

get '/upload/*' do |file_path|
  upload = Upload.find(file_path)
  @pipeline = Pipeline.run(File.read(file_path), upload.user, upload.trial)

  erb :upload
end

post '/create' do
  begin
    Upload.create(params[:data][:tempfile], params[:user], params[:trial])

    redirect '/uploads'
  rescue Exception => e
    redirect '/uploads?error=creation'
  end
end
```

`pedometer.rb`는 각 라우트에 대한 HTTP 요청에 앱이 응답할 수 있게 해줍니다. 각 라우트의 코드 블록은 `Upload`를 통해 파일 시스템에서 데이터를 검색하거나 파일 시스템에 데이터를 저장한 다음, 뷰를 렌더링하거나 리디렉션합니다. 인스턴스화된 인스턴스 변수들은 뷰에서 직접 사용될 것입니다. 뷰들은 단순히 데이터를 표시하고 앱의 초점이 아니므로, 이 장에서는 그것들을 위한 코드를 생략하겠습니다.

`pedometer.rb`의 각 라우트를 개별적으로 살펴봅시다.

#### `GET /uploads`

`http://localhost:4567/uploads`로 이동하면 앱에 HTTP GET 요청을 보내, `get '/uploads'` 코드를 트리거합니다. 코드는 파일 시스템의 모든 업로드에 대해 파이프라인을 실행하고 업로드 목록과 새 업로드를 제출할 양식을 표시하는 `uploads` 뷰를 렌더링합니다. 오류 매개변수가 포함되어 있다면, 오류 문자열이 생성되고 `uploads` 뷰가 오류를 표시할 것입니다.

#### `GET /upload/*`

각 업로드의 **Detail** 링크를 클릭하면 해당 업로드의 파일 경로와 함께 `/upload`로 HTTP GET을 보냅니다. 파이프라인이 실행되고 `upload` 뷰가 렌더링됩니다. 뷰는 HighCharts라는 JavaScript 라이브러리를 사용하여 생성된 차트를 포함하여 업로드의 세부 정보를 표시합니다.

#### `POST /create`

마지막 라우트인 `create`로의 HTTP POST는 사용자가 `uploads` 뷰에서 양식을 제출할 때 호출됩니다. 코드 블록은 `params` 해시를 사용하여 사용자가 양식을 통해 입력한 값들을 가져와 새 `Upload`를 생성하고, `/uploads`로 다시 리디렉션합니다. 생성 과정에서 오류가 발생하면, `/uploads`로의 리디렉션에는 사용자에게 문제가 발생했음을 알리는 오류 매개변수가 포함됩니다.

## 완전히 기능하는 앱

좋습니다! 우리는 진정한 적용 가능성을 가진 완전히 기능하는 앱을 구축했습니다.

현실 세계는 복잡하고 정교한 도전 과제들을 제시합니다. 소프트웨어는 최소한의 자원으로 이러한 도전 과제들을 대규모로 해결할 수 있는 독특한 능력을 가지고 있습니다. 소프트웨어 엔지니어로서, 우리는 집, 커뮤니티, 세상에서 긍정적인 변화를 만들 수 있는 힘을 가지고 있습니다. 학문적이든 아니든 우리의 훈련은 고립된, 잘 정의된 문제들을 해결하는 코드를 작성할 수 있는 문제 해결 능력을 갖추게 해주었을 것입니다. 우리가 성장하고 기술을 연마해나가면서, 우리 세상의 지저분한 현실들과 얽혀있는 실용적인 문제들을 해결하도록 그 훈련을 확장하는 것은 우리에게 달려 있습니다. 이 장이 실제 문제를 작고 다룰 수 있는 부분들로 나누고, 솔루션을 구축하기 위한 아름답고, 깨끗하고, 확장 가능한 코드를 작성하는 맛을 제공했기를 바랍니다.

끝없이 흥미진진한 세상에서 흥미로운 문제들을 해결하는 것을 위해 건배합니다.
