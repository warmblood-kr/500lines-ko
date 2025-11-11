title: 합의를 통한 클러스터링
author: Dustin J. Mitchell
<markdown>
_Dustin은 Mozilla의 오픈 소스 소프트웨어 개발자이자 릴리스 엔지니어입니다.
Puppet의 호스트 구성 시스템, Flask 기반 웹 프레임워크, 방화벽 구성을 위한 단위 테스트,
Twisted Python의 지속적 통합 프레임워크 등 다양한 프로젝트에서 작업했습니다. GitHub에서 [\@djmitche](http://github.com/djmitche)로,
이메일로는 [dustin@mozilla.com](mailto:dustin@mozilla.com)에서 찾을 수 있습니다._
</markdown>
## 소개

이 장에서는 신뢰성 있는 분산 컴퓨팅을 지원하도록 설계된 네트워크 프로토콜의 구현을 살펴보겠습니다.
네트워크 프로토콜은 올바르게 구현하기 어려울 수 있으므로, 버그를 최소화하고 남은 몇 가지 버그를 포착하고 수정하는 기법을 살펴보겠습니다.
신뢰성 있는 소프트웨어 구축에도 특별한 개발 및 디버깅 기법이 필요합니다.

## 동기 부여 예제

이 장의 초점은 프로토콜 구현에 있지만, 동기 부여 예제로 간단한 은행 계좌 관리 서비스를 생각해보겠습니다.
이 서비스에서 각 계좌는 현재 잔액을 가지며 계좌 번호로 식별됩니다.
사용자는 "입금", "이체", "잔액 조회"와 같은 작업을 요청하여 계좌에 접근합니다.
"이체" 작업은 두 계좌(출금 계좌와 입금 계좌)에서 동시에 작동하며, 출금 계좌의 잔액이 부족할 경우 거부되어야 합니다.

서비스가 단일 서버에서 호스팅된다면 구현하기 쉽습니다: 이체 작업이 병렬로 실행되지 않도록 잠금을 사용하고, 해당 메서드에서 출금 계좌의 잔액을 확인하면 됩니다.
그러나 은행은 중요한 계좌 잔액을 단일 서버에 의존할 수 없습니다.
대신 서비스는 여러 서버에 *분산*되어 있으며, 각 서버는 정확히 동일한 코드의 별도 인스턴스를 실행합니다.
사용자는 작업을 수행하기 위해 어떤 서버든 연결할 수 있습니다.

분산 처리의 단순한 구현에서는 각 서버가 모든 계좌 잔액의 로컬 복사본을 유지합니다.
서버는 수신된 모든 작업을 처리하고, 계좌 잔액 업데이트를 다른 서버에 전송합니다.
그러나 이 접근법은 심각한 실패 모드를 도입합니다: 두 서버가 동시에 같은 계좌에 대한 작업을 처리한다면, 어떤 새로운 계좌 잔액이 올바른 것일까요?
서버들이 잔액 대신 작업을 서로 공유한다고 해도, 계좌에서 동시에 두 번의 이체가 발생하면 계좌가 마이너스가 될 수 있습니다.

근본적으로 이러한 실패는 서버가 로컬 상태가 다른 서버의 상태와 일치하는지 먼저 확인하지 않고 로컬 상태를 사용하여 작업을 수행할 때 발생합니다.
예를 들어, 서버 A가 계좌 101에서 계좌 202로의 이체 작업을 받았는데, 서버 B가 이미 계좌 101의 전체 잔액을 계좌 202로 이체하는 다른 작업을 처리했지만 아직 서버 A에 알리지 않은 상황을 상상해보세요.
서버 A의 로컬 상태는 서버 B의 상태와 다르므로, 서버 A는 잘못되게 이체를 완료하도록 허용하며, 그 결과 계좌 101이 마이너스가 됩니다.

## 분산 상태 머신

이러한 문제를 피하는 기법을 "분산 상태 머신"이라고 합니다.
아이디어는 각 서버가 정확히 동일한 입력에 대해 정확히 동일한 결정적 상태 머신을 실행한다는 것입니다.
상태 머신의 특성상 각 서버는 정확히 동일한 출력을 보게 됩니다.
"이체"나 "잔액 조회"와 같은 작업과 그 매개변수(계좌 번호 및 금액)가 상태 머신의 입력을 나타냅니다.

이 애플리케이션의 상태 머신은 간단합니다:

```python
    def execute_operation(state, operation):
        if operation.name == 'deposit':
            if not verify_signature(operation.deposit_signature):
                return state, False
            state.accounts[operation.destination_account] += operation.amount
            return state, True
        elif operation.name == 'transfer':
            if state.accounts[operation.source_account] < operation.amount:
                return state, False
            state.accounts[operation.source_account] -= operation.amount
            state.accounts[operation.destination_account] += operation.amount
            return state, True
        elif operation.name == 'get-balance':
            return state, state.accounts[operation.account]
```

"잔액 조회" 작업을 실행해도 상태를 수정하지 않지만, 여전히 상태 전환으로 구현됩니다.
이는 반환된 잔액이 서버 클러스터의 최신 정보이며, 단일 서버의 (오래된 가능성이 있는) 로컬 상태를 기반으로 하지 않음을 보장합니다.

이것은 컴퓨터 과학 과정에서 배우는 일반적인 상태 머신과 다르게 보일 수 있습니다.
레이블이 붙은 전환을 가진 유한한 명명된 상태 집합이 아니라, 이 머신의 상태는 계좌 잔액의 모음이므로 무한한 가능한 상태가 있습니다.
그러나 결정적 상태 머신의 일반적인 규칙이 적용됩니다: 동일한 상태에서 시작하여 동일한 작업을 처리하면 항상 동일한 출력을 생성합니다.

따라서 분산 상태 머신 기법은 각 호스트에서 동일한 작업이 발생하도록 보장합니다.
그러나 모든 서버가 상태 머신의 입력에 대해 동의하도록 보장하는 문제가 남아 있습니다.
이것은 *합의*의 문제이며, 우리는 Paxos 알고리즘의 파생을 사용하여 이를 해결할 것입니다.

## Paxos를 통한 합의

Paxos는 Leslie Lamport가 1990년에 처음 제출하고 1998년에 결국 출판된 "The Part-Time Parliament"[^parttime]라는 제목의 상상력 넘치는 논문에서 설명되었습니다.
Lamport의 논문은 우리가 여기서 다룰 것보다 훨씬 더 자세한 내용을 담고 있으며, 재미있게 읽을 수 있습니다.
이 장 끝의 참고 문헌은 우리가 이 구현에서 적응한 알고리즘의 일부 확장을 설명합니다.

Paxos의 가장 간단한 형태는 서버 집합이 영원히 하나의 값에 동의하는 방법을 제공합니다.
Multi-Paxos는 이 기반 위에 구축되어 번호가 매겨진 사실 시퀀스에 대해 한 번에 하나씩 동의합니다.
분산 상태 머신을 구현하기 위해, 우리는 Multi-Paxos를 사용하여 각 상태 머신 입력에 동의하고 순서대로 실행합니다.

[^parttime]: L. Lamport, "The Part-Time Parliament," ACM Transactions on Computer Systems, 16(2):133–169, May 1998.

### Simple Paxos

"Simple Paxos"부터 시작해보겠습니다. 이는 Synod 프로토콜이라고도 불리며, 절대 변경될 수 없는 단일 값에 대해 합의하는 방법을 제공합니다.
Paxos라는 이름은 "The Part-Time Parliament"의 신화적인 섬에서 유래되었으며, 여기서 입법자들이 Lamport가 Synod 프로토콜이라고 명명한 과정을 통해 법안에 투표합니다.

이 알고리즘은 아래에서 보겠지만 더 복잡한 알고리즘의 구성 요소입니다.
이 예제에서 우리가 합의할 단일 값은 가상의 은행에서 처리된 첫 번째 거래입니다.
은행은 매일 거래를 처리하지만, 첫 번째 거래는 한 번만 발생하고 절대 변경되지 않으므로 Simple Paxos를 사용하여 그 세부 사항에 대해 합의할 수 있습니다.

프로토콜은 일련의 투표로 작동하며, 각 투표는 제안자(proposer)라고 불리는 클러스터의 단일 구성원이 주도합니다.
각 투표는 정수와 제안자의 신원을 기반으로 한 고유한 투표 번호를 가집니다.
제안자의 목표는 수락자(acceptor) 역할을 하는 클러스터 구성원의 과반수가 자신의 값을 수락하도록 하는 것이지만, 다른 값이 이미 결정되지 않았을 때만 가능합니다.

\aosafigure[240pt]{cluster-images/ballot.png}{투표}{500l.cluster.ballot}

투표는 제안자가 투표 번호 *N*과 함께 ``Prepare`` 메시지를 수락자들에게 보내고 과반수로부터 응답을 기다리는 것으로 시작됩니다(\aosafigref{500l.cluster.ballot}).

``Prepare`` 메시지는 *N*보다 작은 가장 높은 투표 번호를 가진 수락된 값(있다면)에 대한 요청입니다.
수락자들은 이미 수락한 값(있다면)을 포함하는 ``Promise``로 응답하며, 앞으로 *N*보다 작은 번호의 투표는 수락하지 않겠다고 약속합니다.
수락자가 이미 더 큰 투표 번호에 대해 약속을 한 경우, ``Promise``에 해당 번호를 포함하여 제안자가 선점되었음을 나타냅니다.
이 경우 투표는 끝나지만, 제안자는 다른 투표에서(그리고 더 큰 투표 번호로) 다시 시도할 수 있습니다.

제안자가 수락자의 과반수로부터 응답을 받으면, 투표 번호와 값을 포함하는 ``Accept`` 메시지를 모든 수락자에게 보냅니다.
제안자가 어떤 수락자로부터도 기존 값을 받지 않았다면, 자신이 원하는 값을 보냅니다.
그렇지 않으면, 가장 높은 번호의 약속에서 온 값을 보냅니다.

약속을 위반하지 않는 한, 각 수락자는 ``Accept`` 메시지의 값을 수락된 것으로 기록하고 ``Accepted`` 메시지로 응답합니다.
제안자가 수락자의 과반수로부터 자신의 투표 번호를 들었을 때 투표가 완료되고 값이 결정됩니다.

예제로 돌아가서, 처음에는 다른 값이 수락되지 않았으므로 수락자들은 모두 값이 없는 ``Promise``를 보내고, 제안자는 다음과 같은 자신의 값을 포함하는 ``Accept``를 보냅니다:

```python
    operation(name='deposit', amount=100.00, destination_account='Mike DiBernardo')
```

나중에 다른 제안자가 더 낮은 투표 번호와 다른 작업(예: ``'Dustin J. Mitchell'`` 계좌로의 이체)으로 투표를 시작하면, 수락자들은 단순히 이를 수락하지 않습니다.
해당 투표가 더 큰 투표 번호를 가진다면, 수락자들의 ``Promise``는 Michael의 $100.00 입금 작업에 대해 제안자에게 알려주고, 제안자는 Dustin으로의 이체 대신 해당 값을 ``Accept`` 메시지에서 보낼 것입니다.
새로운 투표는 수락되지만, 첫 번째 투표와 동일한 값을 위해서입니다.

실제로 이 프로토콜은 투표가 겹치거나, 메시지가 지연되거나, 소수의 수락자가 실패하더라도 두 개의 서로 다른 값이 결정되는 것을 절대 허용하지 않습니다.

여러 제안자가 동시에 투표를 할 때, 어떤 투표도 수락되지 않기 쉽습니다.
그러면 두 제안자 모두 다시 제안하고, 하나가 이기기를 바라지만, 타이밍이 정확히 맞으면 교착 상태가 무한정 계속될 수 있습니다.

다음 일련의 사건을 고려해보세요:

* 제안자 A가 투표 번호 1에 대해 ``Prepare``/``Promise`` 단계를 수행합니다.
* 제안자 A가 자신의 제안을 수락받기 전에, 제안자 B가 투표 번호 2에 대해 \newline ``Prepare``/``Promise`` 단계를 수행합니다.
* 제안자 A가 마침내 투표 번호 1로 ``Accept``를 보낼 때, 수락자들은 이미 투표 번호 2를 약속했기 때문에 이를 거부합니다.
* 제안자 A가 제안자 B가 자신의 ``Accept`` 메시지를 보내기 전에 즉시 더 높은 투표 번호 (3)로 ``Prepare``를 보내어 반응합니다.
* 제안자 B의 후속 ``Accept``는 거부되고, 과정이 반복됩니다.

불운한 타이밍으로는 -- 메시지를 보내고 응답을 받는 시간이 긴 장거리 연결에서 더 일반적인데 -- 이러한 교착 상태가 여러 라운드 동안 계속될 수 있습니다.

### Multi-Paxos


단일 정적 값에 대한 합의에 도달하는 것은 그 자체로는 특별히 유용하지 않습니다.
은행 계좌 서비스와 같은 클러스터링된 시스템은 시간에 따라 변화하는 특정 상태(계좌 잔액)에 대해 동의하고자 합니다.
우리는 각 작업에 대해 Paxos를 사용하여 합의하며, 이를 상태 머신 전환으로 취급합니다.

Multi-Paxos는 실질적으로 일련의 Simple Paxos 인스턴스들(슬롯)로 각각이 순차적으로 번호가 매겨집니다.
각 상태 전환은 "슬롯 번호"를 부여받으며, 클러스터의 각 구성원은 엄격한 숫자 순서로 전환을 실행합니다.
클러스터의 상태를 변경하기 위해(예를 들어, 이체 작업을 처리하기 위해), 우리는 다음 슬롯에서 해당 작업에 대한 합의를 달성하려고 합니다.
구체적으로 말하면, 이는 각 메시지에 슬롯 번호를 추가하는 것을 의미하며, 모든 프로토콜 상태가 슬롯별로 추적됩니다.

각 슬롯에 대해 최소 두 번의 라운드 트립을 가진 Paxos를 실행하는 것은 너무 느릴 것입니다.
Multi-Paxos는 모든 슬롯에 대해 동일한 투표 번호 집합을 사용하고, 모든 슬롯에 대해 ``Prepare``/``Promise`` 단계를 한 번에 수행하여 최적화합니다.

### Paxos Made Pretty Hard

실용적인 소프트웨어에서 Multi-Paxos를 구현하는 것은 악명 높게 어렵기로 유명하며, Lamport의 "Paxos Made Simple"을 조롱하는 "Paxos Made Practical"과 같은 제목의 여러 논문들이 등장했습니다.

첫째, 위에서 설명한 다중 제안자 문제는 각 클러스터 구성원이 각 슬롯에서 자신의 상태 머신 작업을 결정받으려고 시도할 때 바쁜 환경에서 문제가 될 수 있습니다.
해결책은 각 슬롯에 대한 투표를 제출할 책임이 있는 "리더"를 선출하는 것입니다.
그러면 다른 모든 클러스터 노드들은 실행을 위해 새로운 작업들을 리더에게 보냅니다.
따라서 단 하나의 리더만 있는 정상 작동에서는 투표 충돌이 발생하지 않습니다.

``Prepare``/``Promise`` 단계는 일종의 리더 선출로 기능할 수 있습니다: 가장 최근에 약속된 투표 번호를 소유한 클러스터 구성원이 리더로 간주됩니다.
그러면 리더는 첫 번째 단계를 반복하지 않고 직접 ``Accept``/``Accepted`` 단계를 실행할 수 있습니다.
아래에서 보겠지만, 리더 선출은 실제로 꽤 복잡합니다.

Simple Paxos는 클러스터가 상충하는 결정에 도달하지 않을 것을 보장하지만, 어떤 결정이 내려질 것을 보장할 수는 없습니다.
예를 들어, 초기 ``Prepare`` 메시지가 손실되어 수락자들에게 도달하지 않으면, 제안자는 절대 도착하지 않을 ``Promise`` 메시지를 기다릴 것입니다.
이를 해결하기 위해서는 신중하게 조율된 재전송이 필요합니다: 결국 진전을 이루기에는 충분하지만, 클러스터가 패킷 폭풍에 스스로를 매장할 정도로 많지는 않은 수준으로요.

또 다른 문제는 결정의 전파입니다.
``Decision`` 메시지의 간단한 브로드캐스트가 정상적인 경우에는 이를 처리할 수 있습니다.
그러나 메시지가 손실되면, 노드는 결정에 대해 영구적으로 무지한 상태로 남아 있을 수 있고 이후 슬롯들에 대한 상태 머신 전환을 적용할 수 없게 됩니다.
따라서 구현에서는 결정된 제안들에 대한 정보를 공유하는 어떤 메커니즘이 필요합니다.

분산 상태 머신의 사용은 또 다른 흥미로운 도전을 제시합니다: 시작입니다.
새로운 노드가 시작될 때, 클러스터의 기존 상태를 따라잡아야 합니다.
첫 번째 슬롯부터 모든 슬롯의 결정들을 따라잡음으로써 그렇게 할 수 있지만, 성숙한 클러스터에서는 이것이 수백만 개의 슬롯을 포함할 수 있습니다.
더욱이, 새로운 클러스터를 초기화하는 어떤 방법이 필요합니다.

하지만 이론과 알고리즘에 대한 이야기는 충분합니다 -- 코드를 살펴보겠습니다.

## Cluster 소개

이 장의 *Cluster* 라이브러리는 Multi-Paxos의 간단한 형태를 구현합니다.
이것은 더 큰 애플리케이션에 합의 서비스를 제공하는 라이브러리로 설계되었습니다.

이 라이브러리의 사용자들은 그 정확성에 의존할 것이므로, 명세와의 대응을 볼 수 있고 -- 그리고 테스트할 수 있도록 -- 코드를 구조화하는 것이 중요합니다.
복잡한 프로토콜은 복잡한 실패를 나타낼 수 있으므로, 드물게 발생하는 실패를 재현하고 디버깅하는 지원을 구축할 것입니다.

이 장의 구현은 개념 증명 코드입니다: 핵심 개념이 실용적임을 보여주기에는 충분하지만, 프로덕션에서 사용하기 위해 필요한 모든 일상적인 장비는 없습니다.
코드는 그러한 장비가 핵심 구현에 최소한의 변경으로 나중에 추가될 수 있도록 구조화되어 있습니다.

시작해보겠습니다.

### 타입과 상수

Cluster의 프로토콜은 15개의 서로 다른 메시지 타입을 사용하며, 각각은 Python [``namedtuple``](https://docs.python.org/3/library/collections.html)로 정의됩니다.

```python
    Accepted = namedtuple('Accepted', ['slot', 'ballot_num'])
    Accept = namedtuple('Accept', ['slot', 'ballot_num', 'proposal'])
    Decision = namedtuple('Decision', ['slot', 'proposal'])
    Invoked = namedtuple('Invoked', ['client_id', 'output'])
    Invoke = namedtuple('Invoke', ['caller', 'client_id', 'input_value'])
    Join = namedtuple('Join', [])
    Active = namedtuple('Active', [])
    Prepare = namedtuple('Prepare', ['ballot_num'])
    Promise = namedtuple('Promise', ['ballot_num', 'accepted_proposals'])
    Propose = namedtuple('Propose', ['slot', 'proposal'])
    Welcome = namedtuple('Welcome', ['state', 'slot', 'decisions'])
    Decided = namedtuple('Decided', ['slot'])
    Preempted = namedtuple('Preempted', ['slot', 'preempted_by'])
    Adopted = namedtuple('Adopted', ['ballot_num', 'accepted_proposals'])
    Accepting = namedtuple('Accepting', ['leader'])
```    


각 메시지 타입을 설명하기 위해 명명된 튜플을 사용하는 것은 코드를 깔끔하게 유지하고 일부 간단한 오류를 피하는 데 도움이 됩니다.
명명된 튜플 생성자는 정확히 올바른 속성이 주어지지 않으면 예외를 발생시켜 오타를 명백하게 만듭니다.
튜플들은 로그 메시지에서 자신을 깔끔하게 포맷하며, 추가 보너스로 딕셔너리만큼 많은 메모리를 사용하지 않습니다.

메시지를 생성하는 것은 자연스럽게 읽힙니다:

```python
    msg = Accepted(slot=10, ballot_num=30)
```

그리고 해당 메시지의 필드들은 최소한의 추가 타이핑으로 접근할 수 있습니다:

```python
    got_ballot_num = msg.ballot_num
```

이어지는 섹션들에서 이러한 메시지들이 무엇을 의미하는지 살펴볼 것입니다.
코드는 또한 몇 가지 상수들을 도입하는데, 대부분은 다양한 메시지들에 대한 타임아웃을 정의합니다:

```python
    JOIN_RETRANSMIT = 0.7
    CATCHUP_INTERVAL = 0.6
    ACCEPT_RETRANSMIT = 1.0
    PREPARE_RETRANSMIT = 1.0
    INVOKE_RETRANSMIT = 0.5
    LEADER_TIMEOUT = 1.0
    NULL_BALLOT = Ballot(-1, -1)  # sorts before all real ballots
    NOOP_PROPOSAL = Proposal(None, None, None)  # no-op to fill otherwise empty slots
```

마지막으로, Cluster는 프로토콜 설명에 대응하도록 명명된 두 개의 데이터 타입을 사용합니다:

```python
    Proposal = namedtuple('Proposal', ['caller', 'client_id', 'input'])
    Ballot = namedtuple('Ballot', ['n', 'leader'])
```

### 컴포넌트 모델

인간은 활성 메모리에 보관할 수 있는 것에 제한이 있습니다.
전체 Cluster 구현에 대해 한 번에 추론할 수 없습니다 -- 그것은 너무 많아서 세부 사항을 놓치기 쉽습니다.
비슷한 이유로, 크고 단일체적인 코드베이스는 테스트하기 어렵습니다: 테스트 케이스는 많은 움직이는 부분들을 조작해야 하고 취약하여, 코드의 거의 모든 변경에 실패합니다.

테스트 가능성을 장려하고 코드를 읽기 쉽게 유지하기 위해, 우리는 Cluster를 프로토콜에서 설명된 역할들에 대응하는 소수의 클래스들로 나눕니다.
각각은 ``Role``의 서브클래스입니다.

```python
class Role(object):

    def __init__(self, node):
        self.node = node
        self.node.register(self)
        self.running = True
        self.logger = node.logger.getChild(type(self).__name__)

    def set_timer(self, seconds, callback):
        return self.node.network.set_timer(self.node.address, seconds,
                                           lambda: self.running and callback())

    def stop(self):
        self.running = False
        self.node.unregister(self)
```

클러스터 노드가 갖는 역할들은 네트워크상의 단일 노드를 나타내는 ``Node`` 클래스에 의해 함께 묶입니다.
실행이 진행됨에 따라 역할들이 노드에 추가되고 제거됩니다.
노드에 도착하는 메시지들은 모든 활성 역할들에게 중계되며, ``do_`` 접두사가 붙은 메시지 타입 이름의 메서드를 호출합니다.
이러한 ``do_`` 메서드들은 쉬운 접근을 위해 메시지의 속성들을 키워드 인수로 받습니다.
``Node`` 클래스는 또한 편의를 위해 ``send`` 메서드를 제공하며, ``functools.partial``을 사용하여 ``Network`` 클래스의 동일한 메서드들에 일부 인수를 공급합니다.

```python

class Node(object):
    unique_ids = itertools.count()

    def __init__(self, network, address):
        self.network = network
        self.address = address or 'N%d' % self.unique_ids.next()
        self.logger = SimTimeLogger(
            logging.getLogger(self.address), {'network': self.network})
        self.logger.info('starting')
        self.roles = []
        self.send = functools.partial(self.network.send, self)

    def register(self, roles):
        self.roles.append(roles)

    def unregister(self, roles):
        self.roles.remove(roles)

    def receive(self, sender, message):
        handler_name = 'do_%s' % type(message).__name__

        for comp in self.roles[:]:
            if not hasattr(comp, handler_name):
                continue
            comp.logger.debug("received %s from %s", message, sender)
            fn = getattr(comp, handler_name)
            fn(sender=sender, **message._asdict())
    
```

### 애플리케이션 인터페이스

애플리케이션은 각 클러스터 구성원에서 ``Member`` 객체를 생성하고 시작하며, 애플리케이션별 상태 머신과 피어 목록을 제공합니다.
멤버 객체는 기존 클러스터에 참여하는 경우 노드에 부트스트랩 역할을, 새로운 클러스터를 생성하는 경우 시드를 추가합니다.
그런 다음 별도의 스레드에서 프로토콜을(``Network.run``을 통해) 실행합니다.

애플리케이션은 상태 전환에 대한 제안을 시작하는 ``invoke`` 메서드를 통해 클러스터와 상호작용합니다.
해당 제안이 결정되고 상태 머신이 실행되면, ``invoke``는 머신의 출력을 반환합니다.
이 메서드는 프로토콜 스레드로부터의 결과를 기다리기 위해 간단한 동기화된 `Queue`를 사용합니다.


```python
class Member(object):

    def __init__(self, state_machine, network, peers, seed=None,
                 seed_cls=Seed, bootstrap_cls=Bootstrap):
        self.network = network
        self.node = network.new_node()
        if seed is not None:
            self.startup_role = seed_cls(self.node, initial_state=seed, peers=peers,
                                      execute_fn=state_machine)
        else:
            self.startup_role = bootstrap_cls(self.node,
                                      execute_fn=state_machine, peers=peers)
        self.requester = None

    def start(self):
        self.startup_role.start()
        self.thread = threading.Thread(target=self.network.run)
        self.thread.start()

    def invoke(self, input_value, request_cls=Requester):
        assert self.requester is None
        q = Queue.Queue()
        self.requester = request_cls(self.node, input_value, q.put)
        self.requester.start()
        output = q.get()
        self.requester = None
        return output
```

### 역할 클래스들

라이브러리의 각 역할 클래스들을 하나씩 살펴보겠습니다.

#### Acceptor

``Acceptor``는 프로토콜에서 수락자 역할을 구현하므로, 가장 최근의 약속을 나타내는 투표 번호와 각 슬롯에 대한 수락된 제안들의 집합을 저장해야 합니다.
그런 다음 프로토콜에 따라 ``Prepare``와 ``Accept`` 메시지에 응답합니다.
결과는 프로토콜과 비교하기 쉬운 짧은 클래스입니다.

수락자들에게 있어 Multi-Paxos는 메시지에 슬롯 번호가 추가된 것을 제외하고는 Simple Paxos와 매우 유사해 보입니다.

```python
class Acceptor(Role):

    def __init__(self, node):
        super(Acceptor, self).__init__(node)
        self.ballot_num = NULL_BALLOT
        self.accepted_proposals = {}  # {slot: (ballot_num, proposal)}

    def do_Prepare(self, sender, ballot_num):
        if ballot_num > self.ballot_num:
            self.ballot_num = ballot_num
            # we've heard from a scout, so it might be the next leader
            self.node.send([self.node.address], Accepting(leader=sender))

        self.node.send([sender], Promise(
            ballot_num=self.ballot_num, 
            accepted_proposals=self.accepted_proposals
        ))

    def do_Accept(self, sender, ballot_num, slot, proposal):
        if ballot_num >= self.ballot_num:
            self.ballot_num = ballot_num
            acc = self.accepted_proposals
            if slot not in acc or acc[slot][0] < ballot_num:
                acc[slot] = (ballot_num, proposal)

        self.node.send([sender], Accepted(
            slot=slot, ballot_num=self.ballot_num))

```

#### Replica
\label{sec.cluster.replica}

``Replica`` 클래스는 몇 가지 밀접하게 관련된 책임을 갖고 있어 가장 복잡한 역할 클래스입니다:

* 새로운 제안 만들기;
* 제안이 결정될 때 로컬 상태 머신 호출하기;
* 현재 리더 추적하기; 그리고
* 새로 시작된 노드들을 클러스터에 추가하기.

복제본은 클라이언트로부터의 ``Invoke`` 메시지에 응답하여 새로운 제안을 생성하며, 사용되지 않은 것으로 믿는 슬롯을 선택하고 현재 리더에게 ``Propose`` 메시지를 보냅니다(\aosafigref{500l.cluster.replica}.)
더욱이, 선택된 슬롯에 대한 합의가 다른 제안에 대한 것이라면, 복제본은 새로운 슬롯으로 다시 제안해야 합니다.

\aosafigure[240pt]{cluster-images/replica.png}{Replica Role Control Flow}{500l.cluster.replica}

``Decision`` 메시지들은 클러스터가 합의에 도달한 슬롯들을 나타냅니다.
여기서, 복제본들은 새로운 결정을 저장한 다음, 결정되지 않은 슬롯에 도달할 때까지 상태 머신을 실행합니다.
복제본들은 클러스터가 합의한 *결정된* 슬롯과 로컬 상태 머신이 처리한 *커밋된* 슬롯을 구별합니다.
슬롯들이 순서에 맞지 않게 결정될 때, 커밋된 제안들은 다음 슬롯이 결정되기를 기다리며 뒤처질 수 있습니다.
슬롯이 커밋될 때, 각 복제본은 작업의 결과와 함께 요청자에게 ``Invoked`` 메시지를 다시 보냅니다.

일부 상황에서는 슬롯이 활성 제안도 없고 결정도 없을 가능성이 있습니다.
상태 머신은 슬롯들을 하나씩 실행해야 하므로, 클러스터는 슬롯을 채우기 위해 무언가에 대한 합의에 도달해야 합니다.
이러한 가능성으로부터 보호하기 위해, 복제본들은 슬롯을 따라잡을 때마다 "no-op" 제안을 만듭니다.
그러한 제안이 최종적으로 결정되면, 상태 머신은 해당 슬롯에 대해 아무것도 하지 않습니다.

마찬가지로, 같은 제안이 두 번 결정될 가능성도 있습니다.
복제본은 그러한 중복 제안들에 대해 상태 머신 호출을 건너뛰고, 해당 슬롯에 대해 전환을 수행하지 않습니다.

복제본들은 ``Propose`` 메시지를 보내기 위해 어떤 노드가 활성 리더인지 알아야 합니다.
나중에 보겠지만, 이를 올바르게 하기 위해서는 놀라울 정도로 많은 미묘함이 필요합니다.
각 복제본은 세 가지 정보 소스를 사용하여 활성 리더를 추적합니다.

리더 역할이 활성화될 때, 같은 노드의 복제본에게 ``Adopted`` 메시지를 보냅니다(\aosafigref{500l.cluster.adopted}.)

\aosafigure[240pt]{cluster-images/adopted.png}{Adopted}{500l.cluster.adopted}

수락자 역할이 새로운 리더에게 ``Promise``를 보낼 때, 자신의 로컬 복제본에게 ``Accepting`` 메시지를 보냅니다(\aosafigref{500l.cluster.accepting}.)

\aosafigure[240pt]{cluster-images/accepting.png}{Accepting}{500l.cluster.accepting}

활성 리더는 하트비트로 ``Active`` 메시지를 보냅니다(\aosafigref{500l.cluster.active}.) ``LEADER_TIMEOUT``이 만료되기 전에 그러한 메시지가 도착하지 않으면, 복제본은 리더가 죽었다고 가정하고 다음 리더로 넘어갑니다. 이 경우, 모든 복제본이 *같은* 새로운 리더를 선택하는 것이 중요하며, 이를 위해 구성원들을 정렬하고 목록에서 다음 것을 선택합니다.

\aosafigure[240pt]{cluster-images/active.png}{Active}{500l.cluster.active}

마지막으로, 노드가 네트워크에 참여할 때, 부트스트랩 역할이 ``Join`` 메시지를 보냅니다(\aosafigref{500l.cluster.bootstrap}.) 복제본은 가장 최근의 상태를 포함하는 ``Welcome`` 메시지로 응답하여, 새로운 노드가 빠르게 속도를 맞출 수 있도록 합니다.

\aosafigure[240pt]{cluster-images/bootstrap.png}{Bootstrap}{500l.cluster.bootstrap}

```python
class Replica(Role):

    def __init__(self, node, execute_fn, state, slot, decisions, peers):
        super(Replica, self).__init__(node)
        self.execute_fn = execute_fn
        self.state = state
        self.slot = slot
        self.decisions = decisions
        self.peers = peers
        self.proposals = {}
        # next slot num for a proposal (may lead slot)
        self.next_slot = slot
        self.latest_leader = None
        self.latest_leader_timeout = None

    # making proposals

    def do_Invoke(self, sender, caller, client_id, input_value):
        proposal = Proposal(caller, client_id, input_value)
        slot = next((s for s, p in self.proposals.iteritems() if p == proposal), None)
        # propose, or re-propose if this proposal already has a slot
        self.propose(proposal, slot)

    def propose(self, proposal, slot=None):
        """Send (or resend, if slot is specified) a proposal to the leader"""
        if not slot:
            slot, self.next_slot = self.next_slot, self.next_slot + 1
        self.proposals[slot] = proposal
        # find a leader we think is working - either the latest we know of, or
        # ourselves (which may trigger a scout to make us the leader)
        leader = self.latest_leader or self.node.address
        self.logger.info(
            "proposing %s at slot %d to leader %s" % (proposal, slot, leader))
        self.node.send([leader], Propose(slot=slot, proposal=proposal))

    # handling decided proposals

    def do_Decision(self, sender, slot, proposal):
        assert not self.decisions.get(self.slot, None), \
                "next slot to commit is already decided"
        if slot in self.decisions:
            assert self.decisions[slot] == proposal, \
                "slot %d already decided with %r!" % (slot, self.decisions[slot])
            return
        self.decisions[slot] = proposal
        self.next_slot = max(self.next_slot, slot + 1)

        # re-propose our proposal in a new slot if it lost its slot and wasn't a no-op
        our_proposal = self.proposals.get(slot)
        if (our_proposal is not None and 
            our_proposal != proposal and our_proposal.caller):
            self.propose(our_proposal)

        # execute any pending, decided proposals
        while True:
            commit_proposal = self.decisions.get(self.slot)
            if not commit_proposal:
                break  # not decided yet
            commit_slot, self.slot = self.slot, self.slot + 1

            self.commit(commit_slot, commit_proposal)

    def commit(self, slot, proposal):
        """Actually commit a proposal that is decided and in sequence"""
        decided_proposals = [p for s, p in self.decisions.iteritems() if s < slot]
        if proposal in decided_proposals:
            self.logger.info(
                "not committing duplicate proposal %r, slot %d", proposal, slot)
            return  # duplicate

        self.logger.info("committing %r at slot %d" % (proposal, slot))
        if proposal.caller is not None:
            # perform a client operation
            self.state, output = self.execute_fn(self.state, proposal.input)
            self.node.send([proposal.caller], 
                Invoked(client_id=proposal.client_id, output=output))

    # tracking the leader

    def do_Adopted(self, sender, ballot_num, accepted_proposals):
        self.latest_leader = self.node.address
        self.leader_alive()

    def do_Accepting(self, sender, leader):
        self.latest_leader = leader
        self.leader_alive()

    def do_Active(self, sender):
        if sender != self.latest_leader:
            return
        self.leader_alive()

    def leader_alive(self):
        if self.latest_leader_timeout:
            self.latest_leader_timeout.cancel()

        def reset_leader():
            idx = self.peers.index(self.latest_leader)
            self.latest_leader = self.peers[(idx + 1) % len(self.peers)]
            self.logger.debug("leader timed out; tring the next one, %s", 
                self.latest_leader)
        self.latest_leader_timeout = self.set_timer(LEADER_TIMEOUT, reset_leader)

    # adding new cluster members

    def do_Join(self, sender):
        if sender in self.peers:
            self.node.send([sender], Welcome(
                state=self.state, slot=self.slot, decisions=self.decisions))
```

#### Leader, Scout, 그리고 Commander

리더의 주요 작업은 새로운 투표를 요청하는 ``Propose`` 메시지를 받아서 결정을 생성하는 것입니다.
리더는 프로토콜의 ``Prepare``/``Promise`` 부분을 성공적으로 수행했을 때 "활성" 상태가 됩니다.
활성 리더는 ``Propose``에 응답하여 즉시 ``Accept`` 메시지를 보낼 수 있습니다.

역할별 클래스 모델을 유지하면서, 리더는 프로토콜의 각 부분을 수행하기 위해 스카우트와 커맨더 역할에게 위임합니다.

```python
class Leader(Role):

    def __init__(self, node, peers, commander_cls=Commander, scout_cls=Scout):
        super(Leader, self).__init__(node)
        self.ballot_num = Ballot(0, node.address)
        self.active = False
        self.proposals = {}
        self.commander_cls = commander_cls
        self.scout_cls = scout_cls
        self.scouting = False
        self.peers = peers

    def start(self):
        # reminder others we're active before LEADER_TIMEOUT expires
        def active():
            if self.active:
                self.node.send(self.peers, Active())
            self.set_timer(LEADER_TIMEOUT / 2.0, active)
        active()

    def spawn_scout(self):
        assert not self.scouting
        self.scouting = True
        self.scout_cls(self.node, self.ballot_num, self.peers).start()

    def do_Adopted(self, sender, ballot_num, accepted_proposals):
        self.scouting = False
        self.proposals.update(accepted_proposals)
        # note that we don't re-spawn commanders here; if there are undecided
        # proposals, the replicas will re-propose
        self.logger.info("leader becoming active")
        self.active = True

    def spawn_commander(self, ballot_num, slot):
        proposal = self.proposals[slot]
        self.commander_cls(self.node, ballot_num, slot, proposal, self.peers).start()

    def do_Preempted(self, sender, slot, preempted_by):
        if not slot:  # from the scout
            self.scouting = False
        self.logger.info("leader preempted by %s", preempted_by.leader)
        self.active = False
        self.ballot_num = Ballot((preempted_by or self.ballot_num).n + 1, 
                                 self.ballot_num.leader)

    def do_Propose(self, sender, slot, proposal):
        if slot not in self.proposals:
            if self.active:
                self.proposals[slot] = proposal
                self.logger.info("spawning commander for slot %d" % (slot,))
                self.spawn_commander(self.ballot_num, slot)
            else:
                if not self.scouting:
                    self.logger.info("got PROPOSE when not active - scouting")
                    self.spawn_scout()
                else:
                    self.logger.info("got PROPOSE while scouting; ignored")
        else:
            self.logger.info("got PROPOSE for a slot already being proposed")
```

리더는 비활성 상태일 때 ``Propose``를 받아서 활성화되고 싶을 때 스카우트 역할을 생성합니다(\aosafigref{500l.cluster.leaderscout}.)
스카우트는 ``Prepare`` 메시지를 보내고(필요하면 재전송하고), 피어의 과반수로부터 응답을 듣거나 선점당할 때까지 ``Promise`` 응답을 수집합니다.
각각 ``Adopted`` 또는 ``Preempted``로 리더에게 다시 통신합니다. \newpage

\aosafigure[240pt]{cluster-images/leaderscout.png}{Scout}{500l.cluster.leaderscout}

```python
class Scout(Role):

    def __init__(self, node, ballot_num, peers):
        super(Scout, self).__init__(node)
        self.ballot_num = ballot_num
        self.accepted_proposals = {}
        self.acceptors = set([])
        self.peers = peers
        self.quorum = len(peers) / 2 + 1
        self.retransmit_timer = None

    def start(self):
        self.logger.info("scout starting")
        self.send_prepare()

    def send_prepare(self):
        self.node.send(self.peers, Prepare(ballot_num=self.ballot_num))
        self.retransmit_timer = self.set_timer(PREPARE_RETRANSMIT, self.send_prepare)

    def update_accepted(self, accepted_proposals):
        acc = self.accepted_proposals
        for slot, (ballot_num, proposal) in accepted_proposals.iteritems():
            if slot not in acc or acc[slot][0] < ballot_num:
                acc[slot] = (ballot_num, proposal)

    def do_Promise(self, sender, ballot_num, accepted_proposals):
        if ballot_num == self.ballot_num:
            self.logger.info("got matching promise; need %d" % self.quorum)
            self.update_accepted(accepted_proposals)
            self.acceptors.add(sender)
            if len(self.acceptors) >= self.quorum:
                # strip the ballot numbers from self.accepted_proposals, now that it
                # represents a majority
                accepted_proposals = \ 
                    dict((s, p) for s, (b, p) in self.accepted_proposals.iteritems())
                # We're adopted; note that this does *not* mean that no other
                # leader is active.  # Any such conflicts will be handled by the
                # commanders.
                self.node.send([self.node.address],
                    Adopted(ballot_num=ballot_num, 
                            accepted_proposals=accepted_proposals))
                self.stop()
        else:
            # this acceptor has promised another leader a higher ballot number,
            # so we've lost
            self.node.send([self.node.address], 
                Preempted(slot=None, preempted_by=ballot_num))
            self.stop()
```

리더는 활성 제안이 있는 각 슬롯에 대해 커맨더 역할을 생성합니다(\aosafigref{500l.cluster.leadercommander}.)
스카우트와 마찬가지로, 커맨더는 ``Accept`` 메시지를 보내고 재전송하며, 수락자의 과반수가 ``Accepted``로 응답하거나 선점에 대한 소식을 기다립니다.
제안이 수락되면, 커맨더는 모든 노드에 ``Decision`` 메시지를 브로드캐스트합니다.
리더에게 ``Decided`` 또는 ``Preempted``로 응답합니다.

\aosafigure[240pt]{cluster-images/leadercommander.png}{Commander}{500l.cluster.leadercommander}

```python
class Commander(Role):

    def __init__(self, node, ballot_num, slot, proposal, peers):
        super(Commander, self).__init__(node)
        self.ballot_num = ballot_num
        self.slot = slot
        self.proposal = proposal
        self.acceptors = set([])
        self.peers = peers
        self.quorum = len(peers) / 2 + 1

    def start(self):
        self.node.send(set(self.peers) - self.acceptors, Accept(
            slot=self.slot, ballot_num=self.ballot_num, proposal=self.proposal))
        self.set_timer(ACCEPT_RETRANSMIT, self.start)

    def finished(self, ballot_num, preempted):
        if preempted:
            self.node.send([self.node.address], 
                           Preempted(slot=self.slot, preempted_by=ballot_num))
        else:
            self.node.send([self.node.address], 
                           Decided(slot=self.slot))
        self.stop()

    def do_Accepted(self, sender, slot, ballot_num):
        if slot != self.slot:
            return
        if ballot_num == self.ballot_num:
            self.acceptors.add(sender)
            if len(self.acceptors) < self.quorum:
                return
            self.node.send(self.peers, Decision(
                           slot=self.slot, proposal=self.proposal))
            self.finished(ballot_num, False)
        else:
            self.finished(ballot_num, True)
```

여담으로, 개발 중에 놀라울 정도로 미묘한 버그가 여기서 나타났습니다.
당시, 네트워크 시뮬레이터는 노드 내의 메시지에서도 패킷 손실을 도입했습니다.
*모든* ``Decision`` 메시지가 손실되었을 때, 프로토콜은 진행될 수 없었습니다.
복제본은 계속해서 ``Propose`` 메시지를 재전송했지만, 리더는 해당 슬롯에 대한 제안을 이미 갖고 있어 이를 무시했습니다.
복제본의 따라잡기 과정은 어떤 복제본도 결정에 대해 들은 적이 없어 결과를 찾을 수 없었습니다.
해결책은 실제 네트워크 스택의 경우와 같이 로컬 메시지가 항상 전달되도록 보장하는 것이었습니다.


#### Bootstrap

노드가 클러스터에 참여할 때, 참여하기 전에 현재 클러스터 상태를 결정해야 합니다.
부트스트랩 역할은 ``Welcome``을 받을 때까지 각 피어에게 차례로 ``Join`` 메시지를 보내어 이를 처리합니다.
Bootstrap의 통신 다이어그램은 위의 \aosasecref{sec.cluster.replica}에 표시되어 있습니다.

구현의 초기 버전은 각 노드를 완전한 역할 집합(복제본, 리더, 수락자)으로 시작했으며, 각각은 ``Welcome`` 메시지의 정보를 기다리는 "시작" 단계에서 시작했습니다.
이는 초기화 논리를 모든 역할에 분산시켜, 각각에 대한 별도의 테스팅이 필요했습니다.
최종 설계는 시작이 완료되면 부트스트랩 역할이 다른 각 역할을 노드에 추가하고, 초기 상태를 그들의 생성자에게 전달하도록 합니다.

```python
class Bootstrap(Role):

    def __init__(self, node, peers, execute_fn,
                 replica_cls=Replica, acceptor_cls=Acceptor, leader_cls=Leader,
                 commander_cls=Commander, scout_cls=Scout):
        super(Bootstrap, self).__init__(node)
        self.execute_fn = execute_fn
        self.peers = peers
        self.peers_cycle = itertools.cycle(peers)
        self.replica_cls = replica_cls
        self.acceptor_cls = acceptor_cls
        self.leader_cls = leader_cls
        self.commander_cls = commander_cls
        self.scout_cls = scout_cls

    def start(self):
        self.join()

    def join(self):
        self.node.send([next(self.peers_cycle)], Join())
        self.set_timer(JOIN_RETRANSMIT, self.join)

    def do_Welcome(self, sender, state, slot, decisions):
        self.acceptor_cls(self.node)
        self.replica_cls(self.node, execute_fn=self.execute_fn, peers=self.peers,
                         state=state, slot=slot, decisions=decisions)
        self.leader_cls(self.node, peers=self.peers, commander_cls=self.commander_cls,
                        scout_cls=self.scout_cls).start()
        self.stop()
```

#### Seed

정상적인 작동에서, 노드가 클러스터에 참여할 때, 클러스터가 이미 실행되고 있고 적어도 하나의 노드가 ``Join`` 메시지에 응답할 의지가 있다고 예상합니다.
하지만 클러스터는 어떻게 시작될까요?
한 가지 옵션은 부트스트랩 역할이 다른 모든 노드에 연결을 시도한 후, 자신이 클러스터의 첫 번째라고 결정하는 것입니다.
하지만 이것은 두 가지 문제가 있습니다.
첫째, 큰 클러스터의 경우 각 ``Join``이 타임아웃되는 동안 긴 대기를 의미합니다.
더 중요하게는, 네트워크 분할 상황에서 새로운 노드가 다른 어떤 노드와도 연결할 수 없어 새로운 클러스터를 시작할 수도 있습니다.

네트워크 분할은 클러스터링된 애플리케이션에 가장 도전적인 실패 사례입니다.
네트워크 분할에서는 모든 클러스터 구성원이 살아있지만, 일부 구성원들 간의 통신이 실패합니다.
예를 들어, 베를린과 타이페이의 노드들을 가진 클러스터를 연결하는 네트워크 링크가 실패하면, 네트워크가 분할됩니다.
분할 중에 클러스터의 두 부분이 모두 계속 작동한다면, 네트워크 링크가 복원된 후 부분들을 다시 연결하는 것은 도전적일 수 있습니다.
Multi-Paxos의 경우, 복구된 네트워크는 같은 슬롯 번호에 대해 다른 결정을 가진 두 개의 클러스터를 호스팅하게 될 것입니다.

이러한 결과를 피하기 위해, 새로운 클러스터 생성은 사용자가 지정하는 작업입니다.
클러스터에서 정확히 하나의 노드가 시드 역할을 실행하고, 다른 노드들은 평상시와 같이 부트스트랩을 실행합니다.
시드는 피어의 과반수로부터 ``Join`` 메시지를 받을 때까지 기다린 다음, 상태 머신의 초기 상태와 빈 결정 집합을 가진 ``Welcome``을 보냅니다.
그런 다음 시드 역할은 자신을 중지하고 새로 시드된 클러스터에 참여하기 위해 부트스트랩 역할을 시작합니다.

시드는 부트스트랩/복제본 상호작용의 ``Join``/``Welcome`` 부분을 모방하므로, 그 통신 다이어그램은 복제본 역할과 동일합니다.

```python
class Seed(Role):

    def __init__(self, node, initial_state, execute_fn, peers, 
                 bootstrap_cls=Bootstrap):
        super(Seed, self).__init__(node)
        self.initial_state = initial_state
        self.execute_fn = execute_fn
        self.peers = peers
        self.bootstrap_cls = bootstrap_cls
        self.seen_peers = set([])
        self.exit_timer = None

    def do_Join(self, sender):
        self.seen_peers.add(sender)
        if len(self.seen_peers) <= len(self.peers) / 2:
            return

        # cluster is ready - welcome everyone
        self.node.send(list(self.seen_peers), Welcome(
            state=self.initial_state, slot=1, decisions={}))

        # stick around for long enough that we don't hear any new JOINs from
        # the newly formed cluster
        if self.exit_timer:
            self.exit_timer.cancel()
        self.exit_timer = self.set_timer(JOIN_RETRANSMIT * 2, self.finish)

    def finish(self):
        # bootstrap this node into the cluster we just seeded
        bs = self.bootstrap_cls(self.node, 
                                peers=self.peers, execute_fn=self.execute_fn)
        bs.start()
        self.stop()
```

#### Requester

요청자 역할은 분산 상태 머신에 대한 요청을 관리합니다.
역할 클래스는 단순히 해당하는 ``Invoked``를 받을 때까지 로컬 복제본에 ``Invoke`` 메시지를 보냅니다.
이 역할의 통신 다이어그램은 위의 "Replica" 섹션을 참조하세요.

```python
class Requester(Role):

    client_ids = itertools.count(start=100000)

    def __init__(self, node, n, callback):
        super(Requester, self).__init__(node)
        self.client_id = self.client_ids.next()
        self.n = n
        self.output = None
        self.callback = callback

    def start(self):
        self.node.send([self.node.address], 
                       Invoke(caller=self.node.address, 
                              client_id=self.client_id, input_value=self.n))
        self.invoke_timer = self.set_timer(INVOKE_RETRANSMIT, self.start)

    def do_Invoked(self, sender, client_id, output):
        if client_id != self.client_id:
            return
        self.logger.debug("received output %r" % (output,))
        self.invoke_timer.cancel()
        self.callback(output)
        self.stop()
```

### 요약

요약하자면, 클러스터의 역할들은 다음과 같습니다:

 * Acceptor -- 약속을 하고 제안을 수락
 * Replica -- 분산 상태 머신 관리: 제안 제출, 결정 커밋, 요청자에게 응답
 * Leader -- Multi-Paxos 알고리즘의 라운드를 이끔
 * Scout -- 리더를 위해 Multi-Paxos 알고리즘의 ``Prepare``/``Promise`` 부분을 수행
 * Commander -- 리더를 위해 Multi-Paxos 알고리즘의 ``Accept``/``Accepted`` 부분을 수행
 * Bootstrap -- 기존 클러스터에 새로운 노드를 도입
 * Seed -- 새로운 클러스터를 생성
 * Requester -- 분산 상태 머신 작업을 요청

Cluster가 작동하기 위해 필요한 한 가지 더 있는 장비가 있습니다: 모든 노드들이 통신하는 네트워크입니다.

네트워크
-------

모든 네트워크 프로토콜은 메시지를 보내고 받을 수 있는 능력과 미래의 시점에 함수를 호출하는 수단이 필요합니다.

``Network`` 클래스는 이러한 기능들을 가진 간단한 시뮬레이션된 네트워크를 제공하며, 패킷 손실과 메시지 전파 지연도 시뮬레이션합니다.

타이머들은 Python의 `heapq` 모듈을 사용하여 처리되며, 다음 이벤트의 효율적인 선택을 허용합니다.
타이머 설정은 ``Timer`` 객체를 힙에 푸시하는 것을 포함합니다.
힙에서 항목을 제거하는 것은 비효율적이므로, 취소된 타이머들은 제자리에 남겨두되 취소되었다고 표시됩니다.

메시지 전송은 타이머 기능을 사용하여 각 노드에서 랜덤 시뮬레이션된 지연을 사용해 메시지의 나중 전달을 스케줄합니다.
우리는 다시 ``functools.partial``을 사용하여 적절한 인수와 함께 목적지 노드의 ``receive`` 메서드에 대한 미래 호출을 설정합니다.

시뮬레이션 실행은 단순히 힙에서 타이머를 팝하고, 취소되지 않았으며 목적지 노드가 여전히 활성 상태라면 실행하는 것을 포함합니다.

```python 
class Timer(object):

    def __init__(self, expires, address, callback):
        self.expires = expires
        self.address = address
        self.callback = callback
        self.cancelled = False

    def __cmp__(self, other):
        return cmp(self.expires, other.expires)

    def cancel(self):
        self.cancelled = True


class Network(object):
    PROP_DELAY = 0.03
    PROP_JITTER = 0.02
    DROP_PROB = 0.05

    def __init__(self, seed):
        self.nodes = {}
        self.rnd = random.Random(seed)
        self.timers = []
        self.now = 1000.0

    def new_node(self, address=None):
        node = Node(self, address=address)
        self.nodes[node.address] = node
        return node

    def run(self):
        while self.timers:
            next_timer = self.timers[0]
            if next_timer.expires > self.now:
                self.now = next_timer.expires
            heapq.heappop(self.timers)
            if next_timer.cancelled:
                continue
            if not next_timer.address or next_timer.address in self.nodes:
                next_timer.callback()

    def stop(self):
        self.timers = []

    def set_timer(self, address, seconds, callback):
        timer = Timer(self.now + seconds, address, callback)
        heapq.heappush(self.timers, timer)
        return timer

    def send(self, sender, destinations, message):
        sender.logger.debug("sending %s to %s", message, destinations)
        # avoid aliasing by making a closure containing distinct deep copy of
        # message for each dest
        def sendto(dest, message):
            if dest == sender.address:
                # reliably deliver local messages with no delay
                self.set_timer(sender.address, 0,  
                               lambda: sender.receive(sender.address, message))
            elif self.rnd.uniform(0, 1.0) > self.DROP_PROB:
                delay = self.PROP_DELAY + self.rnd.uniform(-self.PROP_JITTER, 
                                                           self.PROP_JITTER)
                self.set_timer(dest, delay, 
                               functools.partial(self.nodes[dest].receive, 
                                                 sender.address, message))
        for dest in (d for d in destinations if d in self.nodes):
            sendto(dest, copy.deepcopy(message))
```

이 구현에는 포함되지 않았지만, 컴포넌트 모델은 다른 컴포넌트에 변경 없이 실제 네트워크에서 실제 서버들 간에 통신하는 실제 네트워크 구현으로 교체할 수 있게 해줍니다.
테스팅과 디버깅은 시뮬레이션된 네트워크를 사용하여 이루어질 수 있으며, 라이브러리의 프로덕션 사용은 실제 네트워크 하드웨어에서 작동합니다.

디버깅 지원
-----------------

이와 같은 복잡한 시스템을 개발할 때, 버그는 간단한 ``NameError``와 같은 사소한 것에서 몇 분간의 (시뮬레이션된) 프로토콜 작동 후에만 나타나는 모호한 실패로 빠르게 전환됩니다.
이러한 버그를 추적하는 것은 오류가 명백해진 지점에서 역으로 작업하는 것을 포함합니다.
대화형 디버거들은 시간상 앞으로만 나아갈 수 있으므로 여기서는 쓸모없습니다.

Cluster에서 가장 중요한 디버깅 기능은 *결정론적* 시뮬레이터입니다.
실제 네트워크와 달리, 랜덤 수 생성기에 동일한 시드가 주어지면 매 실행에서 정확히 같은 방식으로 행동할 것입니다.
이는 코드에 추가적인 디버깅 확인이나 출력을 추가하고 시뮬레이션을 다시 실행하여 같은 실패를 더 자세히 볼 수 있음을 의미합니다.

물론, 그 세부 사항 대부분은 클러스터의 노드들에 의해 교환되는 메시지에 있으므로, 그것들은 전체적으로 자동으로 로그됩니다.
해당 로깅은 메시지를 보내거나 받는 역할 클래스와 ``SimTimeLogger`` 클래스를 통해 주입된 시뮬레이션된 타임스탬프를 포함합니다.

```python
class SimTimeLogger(logging.LoggerAdapter):

    def process(self, msg, kwargs):
        return "T=%.3f %s" % (self.extra['network'].now, msg), kwargs

    def getChild(self, name):
        return self.__class__(self.logger.getChild(name),
                              {'network': self.extra['network']})
```

이와 같은 복원력 있는 프로토콜은 버그가 트리거된 후에도 종종 오랫동안 실행될 수 있습니다.
예를 들어, 개발 중에 데이터 별칭 오류가 모든 복제본들이 동일한 ``decisions`` 딕셔너리를 공유하도록 했습니다.
이는 하나의 노드에서 결정이 처리되면, 다른 모든 노드들이 그것을 이미 결정된 것으로 본다는 것을 의미했습니다.
이 심각한 버그에도 불구하고, 클러스터는 교착 상태에 빠지기 전에 여러 트랜잭션에 대해 정확한 결과를 생성했습니다.

어서션은 이런 종류의 오류를 일찍 잡기 위한 중요한 도구입니다.
어서션은 알고리즘 설계의 불변량을 포함해야 하지만, 코드가 우리가 예상하는 대로 작동하지 않을 때, 우리의 기대를 어서션하는 것은 일이 잘못되는 곳을 보는 좋은 방법입니다.

```python
    assert not self.decisions.get(self.slot, None), \
            "next slot to commit is already decided"
    if slot in self.decisions:
        assert self.decisions[slot] == proposal, \
            "slot %d already decided with %r!" % (slot, self.decisions[slot])
```

코드를 읽는 동안 우리가 하는 올바른 가정들을 식별하는 것은 디버깅 기술의 일부입니다.
``Replica.do_Decision``의 이 코드에서, 문제는 커밋할 다음 슬롯에 대한 ``Decision``이 이미 ``self.decisions``에 있어서 무시되고 있다는 것이었습니다.
위반되고 있던 기본 가정은 커밋될 다음 슬롯이 아직 결정되지 않았다는 것이었습니다.
``do_Decision`` 시작에서 이를 어서션하는 것이 결함을 식별하고 빠르게 수정으로 이끌었습니다.
마찬가지로, 다른 버그들은 동일한 슬롯에서 다른 제안들이 결정되는 경우들로 이어졌습니다 -- 심각한 오류입니다.

프로토콜 개발 중에 많은 다른 어서션들이 추가되었지만, 공간의 관점에서 몇 개만 남겨두었습니다.

테스팅
-------

지난 10년 동안 어느 시점에, 테스트 없이 코딩하는 것이 마침내 안전벨트 없이 운전하는 것만큼 미친 일이 되었습니다.
테스트가 없는 코드는 아마도 부정확하며, 행동이 변경되었는지 확인할 방법 없이 코드를 수정하는 것은 위험합니다.

테스팅은 코드가 테스트 가능성을 위해 조직화되었을 때 가장 효과적입니다.
이 영역에는 몇 가지 활발한 사상학파가 있지만, 우리가 택한 접근법은 코드를 격리된 상태에서 테스트할 수 있는 작고 최소한으로 연결된 단위들로 나누는 것입니다.
이는 각 역할이 특정 목적을 갖고 다른 것들로부터 격리되어 작동할 수 있어 압축적이고 자급자족적인 클래스를 만드는 역할 모델과 잘 맞습니다.

Cluster는 그 격리를 최대화하도록 작성되었습니다: 새로운 역할을 생성하는 것을 제외하고는 역할들 간의 모든 통신이 메시지를 통해 이루어집니다.
따라서 대부분의 경우, 역할들은 메시지를 보내고 그들의 응답을 관찰함으로써 테스트될 수 있습니다.

#### 단위 테스팅

Cluster의 단위 테스트들은 간단하고 짧습니다:

```python
class Tests(utils.ComponentTestCase):
    def test_propose_active(self):
        """A PROPOSE received while active spawns a commander."""
        self.activate_leader()
        self.node.fake_message(Propose(slot=10, proposal=PROPOSAL1))
        self.assertCommanderStarted(Ballot(0, 'F999'), 10, PROPOSAL1)
```

이 메서드는 단일 유닛(``Leader`` 클래스)의 단일 행동(커맨더 생성)을 테스트합니다.
잘 알려진 "준비, 작동, 어서션" 패턴을 따릅니다: 활성 리더를 설정하고, 메시지를 보내고, 결과를 확인합니다.

#### 의존성 주입

우리는 새로운 역할들의 생성을 처리하기 위해 "의존성 주입"이라 불리는 기법을 사용합니다.
네트워크에 다른 역할들을 추가하는 각 역할 클래스는 실제 클래스들을 기본값으로 하는 클래스 객체들의 목록을 생성자 인수로 받습니다.
예를 들어, ``Leader``의 생성자는 다음과 같습니다:

```python
class Leader(Role):
    def __init__(self, node, peers, commander_cls=Commander, scout_cls=Scout):
        super(Leader, self).__init__(node)
        self.ballot_num = Ballot(0, node.address)
        self.active = False
        self.proposals = {}
        self.commander_cls = commander_cls
        self.scout_cls = scout_cls
        self.scouting = False
        self.peers = peers
```

``spawn_scout`` 메서드는 (그리고 유사하게, ``spawn_commander``) ``self.scout_cls``로 새로운 역할 객체를 생성합니다:

```python
class Leader(Role):
    def spawn_scout(self):
        assert not self.scouting
        self.scouting = True
        self.scout_cls(self.node, self.ballot_num, self.peers).start()
```

이 기법의 마법은 테스팅에서 ``Leader``에게 가짜 클래스들을 줄 수 있어 ``Scout``와 ``Commander``로부터 분리되어 테스트될 수 있다는 것입니다.

#### 인터페이스 정확성

작은 단위에 집중하는 것의 한 가지 함정은 단위들 간의 인터페이스를 테스트하지 않는다는 것입니다.
예를 들어, 수락자 역할에 대한 단위 테스트는 ``Promise`` 메시지의 ``accepted`` 속성 형식을 확인하고, 스카우트 역할에 대한 단위 테스트는 해당 속성에 대해 잘 형식화된 값을 제공합니다.
하지만 두 테스트 모두 그 형식들이 일치하는지 확인하지는 않습니다.

이 문제를 해결하는 한 가지 접근법은 인터페이스를 자체 강제하도록 만드는 것입니다.
Cluster에서 명명된 튜플과 키워드 인수의 사용은 메시지 속성에 대한 모든 불일치를 피합니다.
역할 클래스들 간의 유일한 상호작용이 메시지를 통해 이루어지기 때문에, 이는 인터페이스의 큰 부분을 다룹니다.

``accepted_proposals`` 형식과 같은 특정 문제들의 경우, 실제 데이터와 테스트 데이터 모두 동일한 함수(이 경우 ``verifyPromiseAccepted``)를 사용하여 검증할 수 있습니다.
수락자에 대한 테스트는 이 메서드를 사용하여 반환된 각 ``Promise``를 검증하고, 스카우트에 대한 테스트는 이를 사용하여 모든 가짜 ``Promise``를 검증합니다.

#### 통합 테스팅

인터페이스 문제와 설계 오류에 대한 최후의 방벽은 통합 테스팅입니다.
통합 테스트는 여러 단위를 함께 조립하고 그들의 결합된 효과를 테스트합니다.
우리의 경우, 이는 여러 노드의 네트워크를 구축하고, 여기에 몇 가지 요청을 주입하고, 결과를 검증하는 것을 의미합니다.
단위 테스팅에서 발견되지 않은 인터페이스 문제가 있다면, 통합 테스트가 빠르게 실패하도록 해야 합니다.

프로토콜이 노드 실패를 우아하게 처리하도록 설계되었기 때문에, 활성 리더의 적절하지 않은 실패를 포함하여 몇 가지 실패 시나리오도 테스트합니다.

통합 테스트는 잘 격리되지 않기 때문에 단위 테스트보다 작성하기 어렵습니다.
Cluster의 경우, 이는 실패한 리더를 테스트할 때 가장 명확하게 나타나는데, 어떤 노드든 활성 리더가 될 수 있기 때문입니다.
결정론적 네트워크를 사용하더라도, 하나의 메시지 변경이 랜덤 수 생성기의 상태를 바꾸어 나중 이벤트들을 예측할 수 없게 변경시킵니다.
예상되는 리더를 하드코딩하는 대신, 테스트 코드는 각 리더의 내부 상태를 파헤쳐서 자신이 활성 상태라고 믿는 것을 찾아야 합니다.

#### 퍼즈 테스팅

복원력 있는 코드를 테스트하는 것은 매우 어렵습니다: 자신의 버그에 대해서도 복원력이 있을 가능성이 높으므로, 통합 테스트도 매우 심각한 버그조차 발견하지 못할 수 있습니다.
또한 모든 가능한 실패 모드에 대한 테스트를 상상하고 구성하는 것도 어렵습니다.

이런 종류의 문제에 대한 일반적인 접근법은 "퍼즈 테스팅"입니다: 무언가가 깨질 때까지 무작위로 변하는 입력으로 코드를 반복적으로 실행하는 것입니다.
무언가가 실제로 *깨질* 때, 모든 디버깅 지원이 중요해집니다: 실패를 재현할 수 없고 로깅 정보가 버그를 찾기에 충분하지 않다면 수정할 수 없습니다!

개발 중에 클러스터에 대한 수동 퍼즈 테스팅을 수행했지만, 완전한 퍼즈 테스팅 인프라는 이 프로젝트의 범위를 벗어납니다.

## 권력 투쟁


많은 활성 리더가 있는 클러스터는 매우 시끄러운 곳으로, 스카우트들이 수락자들에게 계속 증가하는 투표 번호를 보내지만 어떤 투표도 결정되지 않습니다.
활성 리더가 없는 클러스터는 조용하지만, 똑같이 기능하지 않습니다.
클러스터가 거의 항상 정확히 하나의 리더에 동의하도록 구현의 균형을 맞추는 것은 놀랍도록 어렵습니다.

싸우는 리더들을 피하는 것은 충분히 쉽습니다: 선점당할 때, 리더는 그냥 새로운 비활성 상태를 받아들입니다.
하지만 이는 활성 리더가 없는 경우로 쉽게 이어지므로, 비활성 리더는 ``Propose`` 메시지를 받을 때마다 활성 상태가 되려고 시도할 것입니다.

전체 클러스터가 어떤 구성원이 활성 리더인지에 동의하지 않으면 문제가 됩니다: 서로 다른 복제본들이 서로 다른 리더들에게 ``Propose`` 메시지를 보내어 스카우트들이 싸우게 됩니다.
따라서 리더 선출이 빠르게 결정되고, 모든 클러스터 구성원들이 가능한 한 빠르게 결과를 알아내는 것이 중요합니다.

Cluster는 가능한 한 빠르게 리더 변경을 감지함으로써 이를 처리합니다: 수락자가 ``Promise``를 보낼 때, 약속받은 구성원이 다음 리더가 될 가능성이 높습니다.
실패는 하트비트 프로토콜로 감지됩니다.

## 추가 확장

물론 이 구현을 확장하고 개선할 수 있는 많은 방법들이 있습니다.

### 따라잡기

"순수한" Multi-Paxos에서 메시지를 받지 못한 노드들은 클러스터의 나머지보다 많은 슬롯 뒤처질 수 있습니다.
분산 상태 머신의 상태가 상태 머신 전환을 통해서만 접근되는 한, 이 설계는 기능적입니다.
상태에서 읽기 위해, 클라이언트는 실제로 상태를 변경하지는 않지만 원하는 값을 반환하는 상태 머신 전환을 요청합니다.
이 전환은 클러스터 전체에서 실행되어, 제안된 슬롯에서의 상태를 기반으로 모든 곳에서 동일한 값을 반환하도록 보장합니다.

최적의 경우에도, 이는 단지 값을 읽기 위해 여러 라운드 트립이 필요하여 느립니다.
분산 객체 저장소가 모든 객체 접근에 대해 그런 요청을 한다면, 그 성능은 형편없을 것입니다.
하지만 요청을 받는 노드가 뒤처져 있을 때, 그 노드는 성공적인 제안을 하기 전에 클러스터의 나머지를 따라잡아야 하므로 요청 지연이 훨씬 큽니다.

간단한 해결책은 가십 스타일 프로토콜을 구현하는 것으로, 각 복제본이 주기적으로 다른 복제본들에게 연락하여 자신이 알고 있는 가장 높은 슬롯을 공유하고 알려지지 않은 슬롯에 대한 정보를 요청합니다.
그러면 ``Decision`` 메시지가 손실되더라도, 복제본은 피어 중 하나로부터 결정에 대해 빠르게 알아낼 것입니다.

### 일관된 메모리 사용량

클러스터 관리 라이브러리는 신뢰할 수 없는 구성 요소들이 있는 상황에서 신뢰성을 제공합니다.
자체적인 신뢰성 없음을 추가해서는 안 됩니다.
불행히도, Cluster는 계속 증가하는 메모리 사용량과 메시지 크기로 인해 실패하지 않고 오래 실행되지 않을 것입니다.

프로토콜 정의에서 수락자와 복제본은 프로토콜의 "메모리"를 형성하므로, 모든 것을 기억해야 합니다.
이 클래스들은 아마도 뒤처진 복제본이나 리더로부터 오래된 슬롯에 대한 요청을 언제 받을지 결코 알지 못합니다.
따라서 정확성을 유지하기 위해, 클러스터가 시작된 이후 모든 결정의 목록을 보관합니다.
더 나쁜 것은, 이런 결정들이 ``Welcome`` 메시지에서 복제본들 간에 전송되어, 오래 살아있는 클러스터에서 이런 메시지들을 거대하게 만든다는 것입니다.

이 문제를 해결하는 한 가지 기법은 각 노드의 상태를 주기적으로 "체크포인트"하여, 제한된 수의 결정에 대한 정보를 손에 두는 것입니다.
체크포인트까지의 모든 슬롯을 커밋하지 않을 정도로 뒤처진 노드들은 클러스터를 떠나고 다시 참여함으로써 스스로를 "재설정"해야 합니다.

#### 영구 저장소

클러스터 구성원의 소수가 실패하는 것은 괜찮지만, 수락자가 자신이 수락한 값이나 한 약속을 "잊어버리는" 것은 괜찮지 않습니다.

불행히도, 이것이 클러스터 구성원이 실패하고 재시작할 때 정확히 일어나는 일입니다: 새로 초기화된 Acceptor 인스턴스는 선임자가 한 약속에 대한 기록이 없습니다.
문제는 새로 시작된 인스턴스가 기존 것의 자리를 차지한다는 것입니다.

이 문제를 해결하는 두 가지 방법이 있습니다.
더 간단한 해결책은 수락자 상태를 디스크에 쓰고 시작 시 그 상태를 다시 읽는 것을 포함합니다.
더 복잡한 해결책은 실패한 클러스터 구성원들을 클러스터에서 제거하고, 새로운 구성원들이 클러스터에 추가되도록 요구하는 것입니다.
클러스터 구성원의 이런 종류의 동적 조정을 "뷰 변경"이라고 합니다.

#### 뷰 변경

운영 엔지니어들은 부하와 가용성 요구사항을 충족하기 위해 클러스터 크기를 조정할 수 있어야 합니다.
간단한 테스트 프로젝트는 하나가 실패해도 영향 없이 최소 3개 노드의 클러스터로 시작할 수 있습니다.
하지만 그 프로젝트가 "라이브"로 갈 때, 추가 부하로 인해 더 큰 클러스터가 필요할 것입니다.

현재 작성된 Cluster는 전체 클러스터를 재시작하지 않고는 클러스터의 피어 집합을 변경할 수 없습니다.
이상적으로는, 클러스터가 상태 머신 전환에 대해서와 마찬가지로 자신의 구성원에 대한 합의를 유지할 수 있어야 합니다.
이는 클러스터 구성원의 집합(*뷰*)이 특별한 뷰 변경 제안에 의해 변경될 수 있음을 의미합니다.
하지만 Paxos 알고리즘은 클러스터 구성원에 대한 보편적 합의에 의존하므로, 각 슬롯에 대한 뷰를 정의해야 합니다.

Lamport는 "Paxos Made Simple"의 마지막 문단에서 이 문제를 다룹니다:

> 합의 알고리즘의 인스턴스 $i+\alpha$를 실행하는 서버 집합이 $i$번째 상태 머신 명령 실행 후의 상태에 의해 지정되도록 함으로써 리더가 $\alpha$ 명령을 앞서 갈 수 있게 할 수 있습니다.  (Lamport, 2001)

아이디어는 Paxos의 각 인스턴스(슬롯)가 $\alpha$ 슬롯 이전의 뷰를 사용한다는 것입니다.
이는 클러스터가 한 번에 최대 $\alpha$ 슬롯에서 작업할 수 있게 하므로, $\alpha$의 매우 작은 값은 동시성을 제한하는 반면, $\alpha$의 매우 큰 값은 뷰 변경이 효과를 나타내는 것을 느리게 만듭니다.

이 구현의 초기 초안에서(git 히스토리에 충실히 보존되어 있습니다!), 뷰 변경 지원을 구현했습니다(3 대신 $\alpha$를 사용하여).
이 겉보기에 간단한 변경은 엄청난 복잡성을 도입했습니다:

* 마지막 $\alpha$ 커밋된 슬롯들 각각에 대한 뷰를 추적하고 이를 새 노드들과 올바르게 공유하기,
* 사용할 수 있는 슬롯이 없는 제안들을 무시하기,
* 실패한 노드들을 감지하기,
* 여러 경쟁하는 뷰 변경들을 적절히 직렬화하기, 그리고
* 리더와 복제본 간 뷰 정보를 소통하기.

그 결과는 이 책에는 너무 컸습니다! \newpage

## 참고문헌

원래 Paxos 논문과 Lamport의 후속작 "Paxos Made Simple"[^simple] 외에도, 우리의 구현은 여러 다른 자료들로부터 얻은 정보를 바탕으로 확장을 추가했습니다. 역할 이름들은 "Paxos Made Moderately Complex"[^complex]에서 가져왔습니다. "Paxos Made Live"[^live]는 특히 스냅샷과 관련하여 도움이 되었고, ["Paxos Made Practical"](http://www.scs.stanford.edu/~dm/home/papers/paxos.pdf)은 뷰 변경을 설명했습니다(여기서 설명한 유형은 아니지만). Liskov의 "From Viewstamped Replication to Byzantine Fault Tolerance"[^tolerance]는 뷰 변경에 대한 또 다른 관점을 제공했습니다. 마지막으로, [Stack Overflow 토론](http://stackoverflow.com/questions/21353312/in-part-time-parliament-why-does-using-the-membership-from-decree-n-3-work-to)은 구성원이 시스템에 어떻게 추가되고 제거되는지 배우는 데 도움이 되었습니다.

[^simple]: L. Lamport, "Paxos Made Simple," ACM SIGACT News (Distributed Computing Column) 32, 4 (Whole Number 121, December 2001) 51-58.
[^complex]: R. Van Renesse and D. Altinbuken, "Paxos Made Moderately Complex," ACM Comp. Survey 47, 3, Article 42 (Feb. 2015)
[^live]: T. Chandra, R. Griesemer, and J. Redstone, "Paxos Made Live - An Engineering Perspective," Proceedings of the twenty-sixth annual ACM symposium on Principles of distributed computing (PODC '07). ACM, New York, NY, USA, 398-407. 
[^tolerance]: B. Liskov, "From Viewstamped Replication to Byzantine Fault Tolerance," In *Replication*, Springer-Verlag, Berlin, Heidelberg 121-149 (2010)
