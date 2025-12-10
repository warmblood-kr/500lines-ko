title: 플로 샵 스케줄러
author: Dr. Christian Muise
<markdown>
_[Dr. Christian Muise](http://haz.ca)는 [MIT CSAIL](http://www.csail.mit.edu/)의 [MERS 그룹](http://groups.csail.mit.edu/mers/)에서 연구원으로 활동하고 있습니다. 그는 AI, 데이터 기반 프로젝트, 매핑, 그래프 이론, 데이터 시각화뿐만 아니라 켈트 음악, 조각, 축구, 커피 등 다양한 분야에 관심을 가지고 있습니다._
</markdown>
## 플로 샵 스케줄러
*플로 샵 스케줄링*은 오퍼레이션 리서치에서 가장 도전적이고 잘 연구된 문제 중 하나입니다. 많은 까다로운 최적화 문제와 마찬가지로, 실용적인 크기의 문제에서는 최적해를 찾는 것이 불가능합니다. 이 장에서는 *지역 탐색*이라는 기법을 사용하는 플로 샵 스케줄링 솔버의 구현을 살펴보겠습니다. 지역 탐색은 최적해를 찾을 수 없을 때 "충분히 좋은" 해를 찾을 수 있게 해줍니다. 솔버는 주어진 시간 동안 문제에 대한 새로운 해를 계속 찾으려고 시도하고, 마지막에 찾은 해 중 최고의 것을 반환합니다.

지역 탐색의 기본 아이디어는 조금 더 나을 수 있는 유사한 해들을 고려하여 기존 해를 휴리스틱하게 개선하는 것입니다. 솔버는 다양한 전략을 사용하여 (1) 유사한 해를 찾으려 시도하고, (2) 다음으로 탐색할 가능성이 있는 해를 선택합니다. 구현은 Python으로 작성되었으며 외부 요구사항이 없습니다. Python의 덜 알려진 기능을 활용하여, 솔버는 어떤 전략이 잘 작동하는지에 따라 해결 과정에서 탐색 전략을 동적으로 변경합니다.

먼저 플로 샵 스케줄링 문제와 지역 탐색 기법에 대한 배경 자료를 제공합니다. 그 다음 일반적인 솔버 코드와 우리가 사용하는 다양한 휴리스틱 및 근방 선택 전략을 자세히 살펴보겠습니다. 다음으로 솔버가 모든 것을 연결하기 위해 사용하는 동적 전략 선택을 고려합니다. 마지막으로 프로젝트 요약과 구현 과정에서 배운 교훈으로 마무리합니다.


## 배경
### 플로 샵 스케줄링
플로 샵 스케줄링 문제는 작업을 완료하는 데 걸리는 총 시간을 최소화하도록 작업을 스케줄링하기 위해 작업의 다양한 태스크에 대한 처리 시간을 결정해야 하는 최적화 문제입니다. 예를 들어, 자동차의 각 부분이 서로 다른 기계에서 순서대로 완성되는 조립 라인을 가진 자동차 제조업체를 생각해 보십시오. 주문마다 사용자 정의 요구사항이 있을 수 있어서, 예를 들어 차체 도색 작업이 자동차마다 달라질 수 있습니다. 우리 예제에서 각 자동차는 새로운 *작업*이고 자동차의 각 부분을 *태스크*라고 합니다. 모든 작업은 완료해야 할 동일한 태스크 순서를 갖습니다.

플로 샵 스케줄링의 목표는 모든 작업의 모든 태스크를 완료까지 처리하는 데 걸리는 총 시간을 최소화하는 것입니다. (일반적으로 이 총 시간을 *메이크스팬*이라고 합니다.) 이 문제는 많은 응용 분야를 가지고 있지만, 생산 시설 최적화와 가장 밀접한 관련이 있습니다.

모든 플로 샵 문제는 $n$개의 기계와 $m$개의 작업으로 구성됩니다. 우리 자동차 예제에서는 자동차를 작업할 $n$개의 스테이션과 총 $m$대의 자동차를 만들어야 합니다. 각 작업은 정확히 $n$개의 태스크로 구성되며, 작업의 $i$번째 태스크는 기계 $i$를 사용해야 하고 미리 정해진 처리 시간이 필요하다고 가정할 수 있습니다: $p(j,i)$는 작업 $j$의 $i$번째 태스크의 처리 시간입니다. 또한, 주어진 작업의 태스크 순서는 사용 가능한 기계의 순서를 따라야 합니다; 주어진 작업에서 태스크 $i$는 태스크 $i+1$이 시작되기 전에 완료되어야 합니다. 우리 자동차 예제에서는 프레임이 조립되기 전에 자동차 도색을 시작하고 싶지 않을 것입니다. 마지막 제약 조건은 두 개의 태스크가 동시에 하나의 기계에서 처리될 수 없다는 것입니다.

작업 내 태스크의 순서가 미리 정해져 있기 때문에, 플로 샵 스케줄링 문제의 해는 작업들의 순열로 나타낼 수 있습니다. 기계에서 처리되는 작업의 순서는 모든 기계에서 동일하며, 순열이 주어지면 작업 $j$의 기계 $i$에 대한 태스크는 다음 두 가능성 중 늦은 것으로 스케줄됩니다:

1. 작업 $j-1$의 기계 $i$에 대한 태스크의 완료 시간 (즉, 동일한 기계에서의 가장 최근 태스크), 또는

2. 작업 $j$의 기계 $i-1$에 대한 태스크의 완료 시간 (즉, 동일한 작업에서의 가장 최근 태스크)

이 두 값 중 최댓값을 선택하기 때문에, 기계 $i$ 또는 작업 $j$에 대한 유휴 시간이 생성됩니다. 궁극적으로 최소화하고자 하는 것이 바로 이 유휴 시간인데, 이는 총 메이크스팬을 더 크게 만들기 때문입니다.

문제의 단순한 형태로 인해, 작업의 모든 순열이 유효한 해이며, 최적해는 *어떤* 순열에 대응됩니다. 따라서 작업의 순열을 변경하고 해당 메이크스팬을 측정하여 개선된 해를 찾습니다. 이하에서는 작업의 순열을 *후보*라고 합니다.

두 개의 작업과 두 개의 기계가 있는 간단한 예를 생각해 보겠습니다. 첫 번째 작업에는 각각 1분과 2분이 걸리는 태스크 $\mathbf{A}$와 $\mathbf{B}$가 있습니다. 두 번째 작업에는 각각 2분과 1분이 걸리는 태스크 $\mathbf{C}$와 $\mathbf{D}$가 있습니다. $\mathbf{A}$는 $\mathbf{B}$ 앞에 와야 하고 $\mathbf{C}$는 $\mathbf{D}$ 앞에 와야 한다는 점을 기억하십시오. 두 개의 작업이 있으므로 고려해야 할 순열은 두 개뿐입니다. 작업 2를 작업 1보다 먼저 순서를 정하면 메이크스팬은 5입니다 (\aosafigref{500l.flowshop.example1}); 반면에 작업 1을 작업 2보다 먼저 순서를 정하면 메이크스팬은 4에 불과합니다 (\aosafigref{500l.flowshop.example2}).

\aosafigure[240pt]{flow-shop-images/example1.png}{플로 샵 예제 1}{500l.flowshop.example1}

\aosafigure[240pt]{flow-shop-images/example2.png}{플로 샵 예제 2}{500l.flowshop.example2}

어떤 태스크도 더 일찍 시작할 여지가 없다는 점에 주목하십시오. 좋은 순열의 기본 원칙은 기계가 처리할 태스크 없이 남겨지는 시간을 최소화하는 것입니다.

### 지역 탐색
지역 탐색은 최적해를 계산하기 너무 어려울 때 최적화 문제를 해결하는 전략입니다. 직관적으로, 꽤 좋아 보이는 해에서 더 나아 보이는 또 다른 해로 이동합니다. 가능한 모든 해를 다음에 집중할 후보로 고려하는 대신, *근방*이라고 알려진 것을 정의합니다: 현재 해와 유사하다고 여겨지는 해들의 집합입니다. 작업의 모든 순열이 유효한 해이므로, 작업들을 섞는 모든 메커니즘을 지역 탐색 절차로 볼 수 있습니다 (실제로 아래에서 하는 일이 바로 이것입니다).

지역 탐색을 공식적으로 사용하려면 몇 가지 질문에 답해야 합니다:

1. 어떤 해로 시작해야 할까요?
2. 해가 주어졌을 때, 고려해야 할 이웃하는 해들은 무엇인가요?
3. 후보 이웃들의 집합이 주어졌을 때, 다음으로 이동을 고려해야 할 것은 어떤 것인가요?

다음 세 섹션에서 이러한 질문들을 순서대로 다룹니다.


## 일반 솔버
이 섹션에서는 플로 샵 스케줄러를 위한 일반적인 프레임워크를 제공합니다. 시작하기 위해 필요한 Python 임포트와 솔버 설정이 있습니다:

```python
import sys, os, time, random

from functools import partial
from collections import namedtuple
from itertools import product

import neighbourhood as neigh
import heuristics as heur

##############
## 설정 ##
##############
TIME_LIMIT = 300.0 # Time (in seconds) to run the solver
TIME_INCREMENT = 13.0 # Time (in seconds) in between heuristic measurements
DEBUG_SWITCH = False # Displays intermediate heuristic info when True
MAX_LNS_NEIGHBOURHOODS = 1000 # Maximum number of neighbours to explore in LNS
```

더 자세히 설명해야 할 두 가지 설정이 있습니다. `TIME_INCREMENT` 설정은 동적 전략 선택의 일부로 사용되고, `MAX_LNS_NEIGHBOURHOODS` 설정은 근방 선택 전략의 일부로 사용됩니다. 둘 다 아래에서 더 자세히 설명됩니다.

이러한 설정들은 명령줄 매개변수로 사용자에게 노출될 수 있지만, 현 단계에서는 대신 입력 데이터를 프로그램의 매개변수로 제공합니다. 입력 문제는 Taillard 벤치마크 세트의 문제로, 플로 샵 스케줄링을 위한 표준 형식으로 되어 있다고 가정합니다. 다음 코드는 솔버 파일의 `__main__` 메서드로 사용되며, 프로그램에 입력된 매개변수의 수에 따라 적절한 함수를 호출합니다:

```python
if __name__ == '__main__':

    if len(sys.argv) == 2:
        data = parse_problem(sys.argv[1], 0)
    elif len(sys.argv) == 3:
        data = parse_problem(sys.argv[1], int(sys.argv[2]))
    else:
        print "\nUsage: python flow.py <Taillard problem file> [<instance number>]\n"
        sys.exit(0)

    (perm, ms) = solve(data)
    print_solution(data, perm)
```

Taillard 문제 파일의 파싱에 대해서는 곧 설명하겠습니다. (파일들은 [온라인에서 이용 가능합니다](http://mistic.heig-vd.ch/taillard/problemes.dir/ordonnancement.dir/ordonnancement.html).)

`solve` 메서드는 `data` 변수가 각 작업의 활동 지속 시간을 포함하는 정수 리스트이길 기대합니다. `solve` 메서드는 전역 전략 세트를 초기화하는 것으로 시작합니다 (아래에서 설명될 예정). 핵심은 각 전략에 대한 통계를 유지하기 위해 `strat_*` 변수들을 사용한다는 것입니다. 이는 해결 과정에서 전략을 동적으로 선택하는 데 도움이 됩니다.

```python
def solve(data):
    """Solves an instance of the flow shop scheduling problem"""

    # We initialize the strategies here to avoid cyclic import issues
    initialize_strategies()
    global STRATEGIES

    # Record the following for each strategy:
    #  improvements: The amount a solution was improved by this strategy
    #  time_spent: The amount of time spent on the strategy
    #  weights: The weights that correspond to how good a strategy is
    #  usage: The number of times we use a strategy
    strat_improvements = {strategy: 0 for strategy in STRATEGIES}
    strat_time_spent = {strategy: 0 for strategy in STRATEGIES}
    strat_weights = {strategy: 1 for strategy in STRATEGIES}
    strat_usage = {strategy: 0 for strategy in STRATEGIES}
```

플로 샵 스케줄링 문제의 매력적인 특징 중 하나는 *모든* 순열이 유효한 해라는 것이고, 적어도 하나는 최적의 메이크스팬을 가질 것이라는 점입니다 (많은 것들이 끔찍한 메이크스팬을 가지겠지만). 다행히도 이는 하나의 순열에서 다른 순열로 갈 때 실행 가능한 해의 공간 내에 머물러 있는지 확인하는 것을 생략할 수 있게 해줍니다&mdash;모든 것이 실행 가능합니다!

하지만 순열의 공간에서 지역 탐색을 시작하려면 초기 순열이 있어야 합니다. 간단하게 하기 위해, 작업 리스트를 무작위로 섞어서 지역 탐색의 시드를 만듭니다:

```python
    # Start with a random permutation of the jobs
    perm = range(len(data))
    random.shuffle(perm)
```

다음으로, 지금까지 찾은 최고의 순열을 추적할 수 있는 변수들과 출력 제공을 위한 타이밍 정보를 초기화합니다. \newpage

```python
    # Keep track of the best solution
    best_make = makespan(data, perm)
    best_perm = perm
    res = best_make

    # Maintain statistics and timing for the iterations
    iteration = 0
    time_limit = time.time() + TIME_LIMIT
    time_last_switch = time.time()

    time_delta = TIME_LIMIT / 10
    checkpoint = time.time() + time_delta
    percent_complete = 10

    print "\nSolving..."
```

이것은 지역 탐색 솔버이므로, 시간 제한에 도달하지 않는 한 계속해서 해를 개선하려고 시도합니다. 솔버의 진행 상황을 나타내는 출력을 제공하고 계산한 반복 횟수를 추적합니다:

```python
    while time.time() < time_limit:

        if time.time() > checkpoint:
            print " %d %%" % percent_complete
            percent_complete += 10
            checkpoint += time_delta

        iteration += 1
```

전략이 어떻게 선택되는지는 아래에서 설명하겠지만, 지금은 전략이 `neighbourhood` 함수와 `heuristic` 함수를 제공한다는 것만 알면 충분합니다. 전자는 고려할 *다음 후보들*의 집합을 제공하고 후자는 그 집합에서 *최고의 후보*를 선택합니다. 이러한 함수들로부터 새로운 순열(`perm`)과 새로운 메이크스팬 결과(`res`)를 얻습니다:

```python
        # Heuristically choose the best strategy
        strategy = pick_strategy(STRATEGIES, strat_weights)

        old_val = res
        old_time = time.time()

        # Use the current strategy's heuristic to pick the next permutation from
        # the set of candidates generated by the strategy's neighbourhood
        candidates = strategy.neighbourhood(data, perm)
        perm = strategy.heuristic(data, candidates)
        res = makespan(data, perm)
```

메이크스팬을 계산하는 코드는 매우 간단합니다: 최종 작업이 완료되는 시점을 평가하여 순열로부터 계산할 수 있습니다. `compile_solution`이 어떻게 작동하는지는 아래에서 보겠지만, 지금은 2차원 배열이 반환되고 `[-1][-1]`의 요소가 스케줄에서 최종 작업의 시작 시간에 대응된다는 것만 알면 충분합니다:

```python
def makespan(data, perm):
    """Computes the makespan of the provided solution"""
    return compile_solution(data, perm)[-1][-1] + data[perm[-1]][-1]
```

전략 선택을 돕기 위해 (1) 전략이 해를 얼마나 개선했는지, (2) 전략이 정보를 계산하는 데 얼마나 많은 시간을 소비했는지, (3) 전략이 몇 번 사용되었는지에 대한 통계를 유지합니다. 더 나은 해를 우연히 발견하면 최고 순열에 대한 변수들도 업데이트합니다:

```python
        # Record the statistics on how the strategy did
        strat_improvements[strategy] += res - old_val
        strat_time_spent[strategy] += time.time() - old_time
        strat_usage[strategy] += 1

        if res < best_make:
            best_make = res
            best_perm = perm[:]
```

일정한 간격으로 전략 사용에 대한 통계가 업데이트됩니다. 가독성을 위해 관련 스니펫을 제거했으며 아래에서 코드를 자세히 설명합니다. 마지막 단계로, while 루프가 완료되면 (즉, 시간 제한에 도달하면) 해결 과정에 대한 통계를 출력하고 최고의 순열을 메이크스팬과 함께 반환합니다:

```python
    print " %d %%\n" % percent_complete
    print "\nWent through %d iterations." % iteration

    print "\n(usage) Strategy:"
    results = sorted([(strat_weights[STRATEGIES[i]], i)
                      for i in range(len(STRATEGIES))], reverse=True)
    for (w, i) in results:
        print "(%d) \t%s" % (strat_usage[STRATEGIES[i]], STRATEGIES[i].name)

    return (best_perm, best_make)
```


### 문제 파싱
파싱 절차의 입력으로, 입력을 찾을 수 있는 파일 이름과 사용해야 할 예제 번호를 제공합니다. (각 파일에는 여러 인스턴스가 포함되어 있습니다.)

```python
def parse_problem(filename, k=1):
    """Parse the kth instance of a Taillard problem file

    The Taillard problem files are a standard benchmark set for the problem
    of flow shop scheduling. 

    print "\nParsing..."
```

파일을 읽어들이고 각 문제 인스턴스를 구분하는 라인을 식별하는 것으로 파싱을 시작합니다:

```python
    with open(filename, 'r') as f:
        # Identify the string that separates instances
        problem_line = ('/number of jobs, number of machines, initial seed, '
                        'upper bound and lower bound :/')

        # Strip spaces and newline characters from every line
        lines = map(str.strip, f.readlines())
```

올바른 인스턴스를 찾기 쉽게 하기 위해, 라인들이 '/' 문자로 구분될 것이라고 가정합니다. 이를 통해 모든 인스턴스의 상단에 나타나는 공통 문자열을 기반으로 파일을 분할할 수 있으며, 첫 번째 라인의 시작 부분에 '/' 문자를 추가하면 선택한 인스턴스에 관계없이 아래의 문자열 처리가 올바르게 작동합니다. 또한 파일에서 찾은 인스턴스 컬렉션을 고려할 때 제공된 인스턴스 번호가 범위를 벗어나는 경우를 감지합니다.

```python
        # We prep the first line for later
        lines[0] = '/' + lines[0]

        # We also know '/' does not appear in the files, so we can use it as
        #  a separator to find the right lines for the kth problem instance
        try:
            lines = '/'.join(lines).split(problem_line)[k].split('/')[2:]
        except IndexError:
            max_instances = len('/'.join(lines).split(problem_line)) - 1
            print "\nError: Instance must be within 1 and %d\n" % max_instances
            sys.exit(0)
```

데이터를 직접 파싱하여 각 태스크의 처리 시간을 정수로 변환하고 리스트에 저장합니다. 마지막으로 데이터를 zip하여 행과 열을 뒤집어서 위의 해결 코드에서 기대하는 형식을 존중하도록 합니다. (`data`의 모든 항목은 특정 작업에 대응되어야 합니다.)

```python
        # Split every line based on spaces and convert each item to an int
        data = [map(int, line.split()) for line in lines]

    # We return the zipped data to rotate the rows and columns, making each
    #  item in data the durations of tasks for a particular job
    return zip(*data)
```


### 해 컴파일하기
플로 샵 스케줄링 문제의 해는 모든 작업의 각 태스크에 대한 정확한 타이밍으로 구성됩니다. 작업들의 순열로 해를 암묵적으로 나타내기 때문에, 순열을 정확한 시간으로 변환하는 `compile_solution` 함수를 도입합니다. 입력으로 이 함수는 문제에 대한 데이터(모든 태스크의 지속 시간을 제공)와 작업들의 순열을 받습니다.

함수는 각 태스크의 시작 시간을 저장하는 데 사용되는 데이터 구조를 초기화하는 것으로 시작하고, 그 다음 순열의 첫 번째 작업에서 태스크들을 포함시킵니다.

```python
def compile_solution(data, perm):
    """Compiles a scheduling on the machines given a permutation of jobs"""

    num_machines = len(data[0])

    # Note that using [[]] * m would be incorrect, as it would simply
    #  copy the same list m times (as opposed to creating m distinct lists).
    machine_times = [[] for _ in range(num_machines)]

    # Assign the initial job to the machines
    machine_times[0].append(0)
    for mach in range(1,num_machines):
        # Start the next task in the job when the previous finishes
        machine_times[mach].append(machine_times[mach-1][0] +
                                   data[perm[0]][mach-1])
```

그 다음 나머지 작업들에 대한 모든 태스크들을 추가합니다. 작업의 첫 번째 태스크는 항상 이전 작업의 첫 번째 태스크가 완료되는 즉시 시작됩니다. 나머지 태스크들의 경우, 가능한 한 빨리 작업을 스케줄링합니다: 동일한 작업의 이전 태스크 완료 시간과 동일한 기계에서의 이전 태스크 완료 시간 중 최댓값입니다.

```python
    # Assign the remaining jobs
    for i in range(1, len(perm)):

        # The first machine never contains any idle time
        job = perm[i]
        machine_times[0].append(machine_times[0][-1] + data[perm[i-1]][0])

        # 나머지 기계들의 경우, 시작 시간은 작업의 이전 태스크가 완료된 시점과
        # 현재 기계가 이전 작업의 태스크를 완료한 시점 중 최댓값입니다.
        for mach in range(1, num_machines):
            machine_times[mach].append(max(
                machine_times[mach-1][i] + data[perm[i]][mach-1],
                machine_times[mach][i-1] + data[perm[i-1]][mach]))

    return machine_times
```
### 해 출력하기
해결 과정이 완료되면, 프로그램은 해에 대한 정보를 컴팩트한 형태로 출력합니다. 모든 작업의 모든 태스크에 대한 정확한 타이밍을 제공하는 대신, 다음과 같은 정보들을 출력합니다:

1. 최고의 메이크스팬을 산출한 작업들의 순열
2. 순열의 계산된 메이크스팬
3. 모든 기계의 시작 시간, 완료 시간, 유휴 시간
4. 모든 작업의 시작 시간, 완료 시간, 유휴 시간

작업 또는 기계의 시작 시간은 작업 또는 기계에서 첫 번째 태스크의 시작에 대응됩니다. 마찬가지로, 작업 또는 기계의 완료 시간은 작업 또는 기계에서 마지막 태스크의 종료에 대응됩니다. 유휴 시간은 특정 작업 또는 기계에 대한 태스크들 사이의 여유 시간의 양입니다. 이상적으로는 유휴 시간의 양을 줄이고 싶은데, 이는 전체 프로세스 시간도 줄어든다는 것을 의미하기 때문입니다.

해를 컴파일하는 코드 (즉, 모든 태스크에 대한 시작 시간을 계산하는 코드)는 이미 논의되었고, 순열과 메이크스팬을 출력하는 것은 간단합니다:

```python
def print_solution(data, perm):
    """Prints statistics on the computed solution"""

    sol = compile_solution(data, perm)

    print "\nPermutation: %s\n" % str([i+1 for i in perm])

    print "Makespan: %d\n" % makespan(data, perm)
```

다음으로, Python의 문자열 서식 기능을 사용하여 각 기계와 작업의 시작, 종료 및 유휴 시간 표를 출력합니다. 작업의 유휴 시간은 작업이 시작된 시점부터 완료까지의 시간에서 작업의 각 태스크에 대한 처리 시간의 합을 뺀 것입니다. 기계의 유휴 시간도 비슷한 방식으로 계산합니다.

```python
    row_format ="{:>15}" * 4
    print row_format.format('Machine', 'Start Time', 'Finish Time', 'Idle Time')
    for mach in range(len(data[0])):
        finish_time = sol[mach][-1] + data[perm[-1]][mach]
        idle_time = (finish_time - sol[mach][0]) - sum([job[mach] for job in data])
        print row_format.format(mach+1, sol[mach][0], finish_time, idle_time)

    results = []
    for i in range(len(data)):
        finish_time = sol[-1][i] + data[perm[i]][-1]
        idle_time = (finish_time - sol[0][i]) - sum([time for time in data[perm[i]]])
        results.append((perm[i]+1, sol[0][i], finish_time, idle_time))

    print "\n"
    print row_format.format('Job', 'Start Time', 'Finish Time', 'Idle Time')
    for r in sorted(results):
        print row_format.format(*r)

    print "\n\nNote: Idle time does not include initial or final wait time.\n"
```


## 근방

지역 탐색의 기본 아이디어는 한 해에서 근처의 다른 해들로 *지역적으로* 이동하는 것입니다. 주어진 해의 *근방*을 그것과 지역적으로 가까운 다른 해들이라고 부릅니다. 이 섹션에서는 복잡성이 증가하는 네 가지 잠재적 근방을 자세히 설명합니다.

첫 번째 근방은 주어진 수의 무작위 순열을 생성합니다. 이 근방은 우리가 시작하는 해조차 고려하지 않으므로 "근방"이라는 용어가 엄밀하지 않습니다. 하지만 탐색에 일부 무작위성을 포함하는 것은 탐색 공간의 탐험을 촉진하므로 좋은 관행입니다.
 
```python
def neighbours_random(data, perm, num = 1):
    # Returns <num> random job permutations, including the current one
    candidates = [perm]
    for i in range(num):
        candidate = perm[:]
        random.shuffle(candidate)
        candidates.append(candidate)
    return candidates
```

다음 근방에서는 순열에서 임의의 두 작업을 교환하는 것을 고려합니다. `itertools` 패키지의 `combinations` 함수를 사용하여 모든 인덱스 쌍을 쉽게 반복하고 각 인덱스에 위치한 작업들을 교환하는 데 대응하는 새로운 순열을 생성할 수 있습니다. 어떤 의미에서 이 근방은 우리가 시작한 것과 매우 유사한 순열들을 생성합니다.

```python
def neighbours_swap(data, perm):
    # Returns the permutations corresponding to swapping every pair of jobs
    candidates = [perm]
    for (i,j) in combinations(range(len(perm)), 2):
        candidate = perm[:]
        candidate[i], candidate[j] = candidate[j], candidate[i]
        candidates.append(candidate)
    return candidates
```

다음으로 고려하는 근방은 당면한 문제에 특정한 정보를 사용합니다. 가장 많은 유휴 시간을 가진 작업들을 찾고 가능한 모든 방법으로 그들을 교환하는 것을 고려합니다. 고려할 작업의 수인 `size` 값을 받습니다: 가장 유휴한 `size`개의 작업들입니다. 프로세스의 첫 번째 단계는 순열의 모든 작업에 대한 유휴 시간을 계산하는 것입니다:

```python
def neighbours_idle(data, perm, size=4):
    # Returns the permutations of the <size> most idle jobs
    candidates = [perm]

    # Compute the idle time for each job
    sol = flow.compile_solution(data, perm)
    results = []

    for i in range(len(data)):
        finish_time = sol[-1][i] + data[perm[i]][-1]
        idle_time = (finish_time - sol[0][i]) - sum([t for t in data[perm[i]]])
        results.append((idle_time, i))
```

다음으로, 가장 많은 유휴 시간을 가진 `size`개의 작업들의 리스트를 계산합니다.

```python
    # Take the <size> most idle jobs
    subset = [job_ind for (idle, job_ind) in reversed(sorted(results))][:size]
```

마지막으로, 식별한 가장 유휴한 작업들의 모든 순열을 고려하여 근방을 구성합니다. 순열을 찾기 위해 `itertools` 패키지의 `permutations` 함수를 사용합니다.

```python
    # Enumerate the permutations of the idle jobs
    for ordering in permutations(subset):
        candidate = perm[:]
        for i in range(len(ordering)):
            candidate[subset[i]] = perm[ordering[i]]
        candidates.append(candidate)

    return candidates
```

마지막으로 고려하는 근방은 일반적으로 *Large Neighbourhood Search* (LNS)라고 불립니다. 직관적으로, LNS는 현재 순열의 작은 부분집합들을 독립적으로 고려하여 작동합니다&mdash;작업 부분집합의 최고 순열을 찾는 것은 LNS 근방에 대한 단일 후보를 제공합니다. 특정 크기의 여러 (또는 모든) 부분집합에 대해 이 프로세스를 반복함으로써 근방의 후보 수를 늘릴 수 있습니다. 이웃의 수가 매우 빠르게 증가할 수 있으므로 `MAX_LNS_NEIGHBOURHOODS` 매개변수를 통해 고려되는 수를 제한합니다. LNS 계산의 첫 번째 단계는 `itertools` 패키지의 `combinations` 함수를 사용하여 교환을 고려할 작업 집합의 무작위 리스트를 계산하는 것입니다:

```python
def neighbours_LNS(data, perm, size = 2):
    # Returns the Large Neighbourhood Search neighbours
    candidates = [perm]

    # Bound the number of neighbourhoods in case there are too many jobs
    neighbourhoods = list(combinations(range(len(perm)), size))
    random.shuffle(neighbourhoods)
```

다음으로, 부분집합들을 반복하여 각각에서 작업들의 최고 순열을 찾습니다. 가장 유휴한 작업들의 모든 순열을 반복하는 유사한 코드를 위에서 보았습니다. 여기서 핵심 차이점은 부분집합에 대해서만 최고 순열을 기록한다는 것입니다. 더 큰 근방은 고려된 작업의 각 부분집합에 대해 하나의 순열을 선택하여 구성되기 때문입니다.

```python
    for subset in neighbourhoods[:flow.MAX_LNS_NEIGHBOURHOODS]:

        # Keep track of the best candidate for each neighbourhood
        best_make = flow.makespan(data, perm)
        best_perm = perm

        # Enumerate every permutation of the selected neighbourhood
        for ordering in permutations(subset):
            candidate = perm[:]
            for i in range(len(ordering)):
                candidate[subset[i]] = perm[ordering[i]]
            res = flow.makespan(data, candidate)
            if res < best_make:
                best_make = res
                best_perm = candidate

        # Record the best candidate as part of the larger neighbourhood
        candidates.append(best_perm)

    return candidates
```

만약 `size` 매개변수를 작업 수와 동일하게 설정한다면, 모든 순열이 고려되고 최고의 것이 선택될 것입니다. 하지만 실제로는 부분집합의 크기를 3 또는 4 정도로 제한해야 합니다; 그보다 큰 것은 `neighbours_LNS` 함수가 금지적인 양의 시간을 소요하게 할 것입니다.


## 휴리스틱

휴리스틱은 제공된 후보 집합에서 단일 후보 순열을 반환합니다. 휴리스틱은 어떤 후보가 선호될 수 있는지 평가하기 위해 문제 데이터에도 접근할 수 있습니다.

우리가 고려하는 첫 번째 휴리스틱은 `heur_random`입니다. 이 휴리스틱은 어떤 것이 선호될 수 있는지 평가하지 않고 리스트에서 후보를 무작위로 선택합니다:

```python
def heur_random(data, candidates):
    # Returns a random candidate choice
    return random.choice(candidates)
```

다음 휴리스틱인 `heur_hillclimbing`은 다른 극단을 사용합니다. 후보를 무작위로 선택하는 대신, 최고의 메이크스팬을 가진 후보를 선택합니다. `scores` 리스트는 `(make,perm)` 형태의 튜플들을 포함할 것이며, 여기서 `make`는 순열 `perm`에 대한 메이크스팬 값입니다. 이러한 리스트를 정렬하면 최고의 메이크스팬을 가진 튜플이 리스트의 시작 부분에 위치합니다; 이 튜플에서 순열을 반환합니다.

```python
def heur_hillclimbing(data, candidates):
    # Returns the best candidate in the list
    scores = [(flow.makespan(data, perm), perm) for perm in candidates]
    return sorted(scores)[0][1]
```

마지막 휴리스틱인 `heur_random_hillclimbing`은 위의 무작위 및 언덕 등반 휴리스틱을 모두 결합합니다. 지역 탐색을 수행할 때, 항상 무작위 후보나 심지어 최고의 후보를 선택하고 싶지 않을 수 있습니다. `heur_random_hillclimbing` 휴리스틱은 0.5의 확률로 최고의 후보를, 그 다음 0.25의 확률로 두 번째로 좋은 후보를 선택하는 식으로 "충분히 좋은" 해를 반환합니다. while 루프는 본질적으로 매 반복마다 동전을 던져 인덱스를 계속 증가시켜야 하는지 확인합니다 (리스트 크기에 제한이 있음). 선택된 최종 인덱스는 휴리스틱이 선택하는 후보에 대응됩니다.

```python
def heur_random_hillclimbing(data, candidates):
    # Returns a candidate with probability proportional to its rank in sorted quality
    scores = [(flow.makespan(data, perm), perm) for perm in candidates]
    i = 0
    while (random.random() < 0.5) and (i < len(scores) - 1):
        i += 1
    return sorted(scores)[i][1]
```

메이크스팬이 우리가 최적화하려는 기준이므로, 언덕 등반은 지역 탐색 과정을 더 나은 메이크스팬을 가진 해로 향하게 할 것입니다. 무작위성을 도입하면 매 단계마다 최고로 보이는 해로 맹목적으로 가는 대신 근방을 탐험할 수 있게 해줍니다.

## 동적 전략 선택
좋은 순열을 위한 지역 탐색의 핵심은 한 해에서 다른 해로 점프하기 위해 특정 휴리스틱과 근방 함수를 사용하는 것입니다. 어떻게 한 옵션 세트를 다른 것보다 선택할까요? 실제로 탐색 중에 전략을 바꾸는 것이 자주 효과적입니다. 우리가 사용하는 동적 전략 선택은 휴리스틱과 근방 함수의 조합들 사이를 전환하여 가장 잘 작동하는 전략들로 동적으로 이동하려고 시도할 것입니다. 우리에게 *전략*은 휴리스틱과 근방 함수의 특정 구성(매개변수 값 포함)입니다.

시작하기 위해, 우리 코드는 해결 중에 고려하고 싶은 전략의 범위를 구성합니다. 전략 초기화에서, `functools` 패키지의 `partial` 함수를 사용하여 각 근방의 매개변수를 부분적으로 할당합니다. 추가적으로, 휴리스틱 함수들의 리스트를 구성하고, 마지막으로 product 연산자를 사용하여 근방과 휴리스틱 함수의 모든 조합을 새로운 전략으로 추가합니다.

```python
################
## 전략 ##
#################################################
## 전략은 특정 구성이다
##  of neighbourhood generator (to compute
##  the next set of candidates) and heuristic
##  computation (to select the best candidate).
##

STRATEGIES = []

# Using a namedtuple is a little cleaner than using dictionaries.
#  E.g., strategy['name'] versus strategy.name
Strategy = namedtuple('Strategy', ['name', 'neighbourhood', 'heuristic'])

def initialize_strategies():

    global STRATEGIES

    # Define the neighbourhoods (and parameters) that we would like to use
    neighbourhoods = [
        ('Random Permutation', partial(neigh.neighbours_random, num=100)),
        ('Swapped Pairs', neigh.neighbours_swap),
        ('Large Neighbourhood Search (2)', partial(neigh.neighbours_LNS, size=2)),
        ('Large Neighbourhood Search (3)', partial(neigh.neighbours_LNS, size=3)),
        ('Idle Neighbourhood (3)', partial(neigh.neighbours_idle, size=3)),
        ('Idle Neighbourhood (4)', partial(neigh.neighbours_idle, size=4)),
        ('Idle Neighbourhood (5)', partial(neigh.neighbours_idle, size=5))
    ]

    # Define the heuristics that we would like to use
    heuristics = [
        ('Hill Climbing', heur.heur_hillclimbing),
        ('Random Selection', heur.heur_random),
        ('Biased Random Selection', heur.heur_random_hillclimbing)
    ]

    # Combine every neighbourhood and heuristic strategy
    for (n, h) in product(neighbourhoods, heuristics):
        STRATEGIES.append(Strategy("%s / %s" % (n[0], h[0]), n[1], h[1]))
```

전략들이 정의되면, 탐색 중에 반드시 단일 옵션을 고수하고 싶지는 않습니다. 대신, 전략들 중 하나를 무작위로 선택하지만, 전략이 얼마나 잘 수행되었는지에 기반하여 *선택에 가중치를 부여*합니다. 가중치는 아래에서 설명하지만, `pick_strategy` 함수의 경우 전략 리스트와 해당하는 상대적 가중치 리스트(어떤 수든 상관없음)만 필요합니다. 주어진 가중치로 무작위 전략을 선택하기 위해, 0과 모든 가중치의 합 사이에서 균등하게 수를 선택합니다. 그 후, $i$보다 작은 인덱스들의 모든 가중치의 합이 우리가 선택한 무작위 수보다 큰 가장 낮은 인덱스 $i$를 찾습니다. 때때로 *룰렛 휠 선택*이라고 불리는 이 기법은 무작위로 전략을 선택해주고 더 높은 가중치를 가진 전략들에게 더 많은 기회를 제공합니다.

```python
def pick_strategy(strategies, weights):
    # Picks a random strategy based on its weight: roulette wheel selection
    #  Rather than selecting a strategy entirely at random, we bias the
    #  random selection towards strategies that have worked well in the
    #  past (according to the weight value).
    total = sum([weights[strategy] for strategy in strategies])
    pick = random.uniform(0, total)
    count = weights[strategies[0]]

    i = 0
    while pick > count:
        count += weights[strategies[i+1]]
        i += 1

    return strategies[i]
```

남은 것은 해를 탐색하는 동안 가중치가 어떻게 증가되는지 설명하는 것입니다. 이는 솔버의 메인 while 루프에서 정기적인 시간 간격(`TIME_INCREMENT` 변수로 정의됨)으로 발생합니다:

```python

        # At regular intervals, switch the weighting on the strategies available.
        #  This way, the search can dynamically shift towards strategies that have
        #  proven more effective recently.
        if time.time() > time_last_switch + TIME_INCREMENT:

            time_last_switch = time.time()
```

`strat_improvements`가 전략이 만든 모든 개선의 합을 저장하는 반면 `strat_time_spent`는 마지막 간격 동안 전략이 주어진 시간을 저장한다는 것을 기억하십시오. 각 전략이 마지막 간격에서 얼마나 잘 수행되었는지의 지표를 얻기 위해 각 전략에 소요된 총 시간으로 만들어진 개선을 정규화합니다. 전략이 전혀 실행될 기회가 없었을 수 있으므로, 기본값으로 작은 양의 시간을 선택합니다.

```python
            # Normalize the improvements made by the time it takes to make them
            results = sorted([
                (float(strat_improvements[s]) / max(0.001, strat_time_spent[s]), s)
                for s in STRATEGIES])
```

이제 각 전략이 얼마나 잘 수행되었는지의 순위가 있으므로, 최고 전략의 가중치에 $k$를 추가하고 ($k$개의 전략이 있다고 가정), 다음으로 좋은 전략에 $k-1$을 추가하는 식입니다. 각 전략은 가중치가 증가할 것이고, 리스트에서 가장 나쁜 전략은 1만큼만 증가할 것입니다.

```python
            # Boost the weight for the successful strategies
            for i in range(len(STRATEGIES)):
                strat_weights[results[i][1]] += len(STRATEGIES) - i
```

추가 조치로, 사용되지 않은 모든 전략들을 인위적으로 끌어올립니다. 이는 전략을 완전히 잊지 않도록 하기 위해서입니다. 한 전략이 처음에는 성능이 나쁜 것처럼 보일 수 있지만, 나중에 탐색에서 매우 유용할 수 있습니다.

```python
                # Additionally boost the unused strategies to avoid starvation
                if results[i][0] == 0:
                    strat_weights[results[i][1]] += len(STRATEGIES)
```

마지막으로, (`DEBUG_SWITCH` 플래그가 설정된 경우) 전략 순위에 대한 일부 정보를 출력하고, 다음 간격을 위해 `strat_improvements`와 `strat_time_spent` 변수를 재설정합니다.

```python
            if DEBUG_SWITCH:
                print "\nComputing another switch..."
                print "Best: %s (%d)" % (results[0][1].name, results[0][0])
                print "Worst: %s (%d)" % (results[-1][1].name, results[-1][0])
                print results
                print sorted([strat_weights[STRATEGIES[i]] 
                              for i in range(len(STRATEGIES))])

            strat_improvements = {strategy: 0 for strategy in STRATEGIES}
            strat_time_spent = {strategy: 0 for strategy in STRATEGIES}
```

## 토론
이 장에서 우리는 플로 샵 스케줄링이라는 복잡한 최적화 문제를 해결하기 위해 상대적으로 적은 양의 코드로 무엇을 달성할 수 있는지 보았습니다. 플로 샵과 같은 대규모 최적화 문제에 대한 최고의 해를 찾는 것은 어려울 수 있습니다. 이런 경우, *충분히 좋은* 해를 계산하기 위해 지역 탐색과 같은 근사 기법으로 전환할 수 있습니다. 지역 탐색으로 한 해에서 다른 해로 이동하면서 좋은 품질의 해를 찾는 것을 목표로 할 수 있습니다.

지역 탐색의 일반적인 직관은 광범위한 문제에 적용될 수 있습니다. 우리는 (1) 한 후보 해에서 문제에 대한 관련 해들의 근방을 생성하는 것과 (2) 해들을 평가하고 비교하는 방법을 설정하는 것에 집중했습니다. 이 두 구성 요소를 갖춘 상태에서, 최고의 옵션이 단순히 계산하기 너무 어려울 때 가치 있는 해를 찾기 위해 지역 탐색 패러다임을 사용할 수 있습니다.

문제를 해결하기 위해 어떤 하나의 전략을 사용하는 대신, 해결 과정에서 전환하기 위해 전략을 동적으로 선택할 수 있는 방법을 보았습니다. 이 간단하고 강력한 기법은 프로그램에게 당면한 문제에 대한 부분 전략들을 혼합하고 매칭하는 능력을 제공하며, 개발자가 전략을 직접 맞춤화할 필요가 없다는 것을 의미하기도 합니다.
