title: 지속적 통합 시스템
author: Malini Das
<markdown>
_Malini Das는 빠르지만 안전한 개발과 교차 기능적 문제 해결에 열정적인 소프트웨어 엔지니어입니다. Mozilla에서 도구 엔지니어로 일했으며 현재 Twitch에서 기술을 연마하고 있습니다. Malini를 [Twitter](https://twitter.com/malinidas)나 그녀의 [블로그](http://malinidas.com/)에서 팔로우하세요._
</markdown>
## 지속적 통합 시스템이란 무엇인가?

소프트웨어를 개발할 때, 우리는 새로운 기능이나 버그 수정이 안전하고 예상대로 작동하는지 검증할 수 있기를 원합니다. 이를 위해 코드에 대해 테스트를 실행합니다. 때로는 개발자들이 자신의 변경사항이 안전한지 확인하기 위해 로컬에서 테스트를 실행하지만, 소프트웨어가 실행되는 모든 시스템에서 코드를 테스트할 시간이 없을 수도 있습니다. 또한, 점점 더 많은 테스트가 추가됨에 따라 로컬에서만이라도 테스트를 실행하는 데 필요한 시간이 현실적이지 않게 됩니다. 이러한 이유로 지속적 통합 시스템이 만들어졌습니다.

지속적 통합(CI) 시스템은 새로운 코드를 테스트하는 데 전용으로 사용되는 시스템입니다. 코드 저장소에 커밋이 이루어지면, 지속적 통합 시스템은 이 커밋이 기존 테스트를 깨뜨리지 않는지 검증할 책임이 있습니다. 이를 위해 시스템은 새로운 변경사항을 가져오고, 테스트를 실행하며, 결과를 보고할 수 있어야 합니다. 다른 모든 시스템과 마찬가지로, 장애에 대한 저항력도 갖추어야 합니다. 즉, 시스템의 일부가 실패하더라도 그 지점에서 복구하여 계속 진행할 수 있어야 합니다.

이 테스트 시스템은 또한 부하를 잘 처리해야 하므로, 테스트를 실행할 수 있는 속도보다 빠르게 커밋이 이루어지는 경우에도 합리적인 시간 내에 테스트 결과를 얻을 수 있어야 합니다. 이는 테스트 작업을 분산하고 병렬화함으로써 달성할 수 있습니다. 이 프로젝트는 확장성을 고려해 설계된 작고 기본적인 분산 지속적 통합 시스템을 보여줄 것입니다.

## 프로젝트 제약사항 및 참고사항

이 프로젝트는 테스트해야 할 코드의 저장소로 Git을 사용합니다. 표준 소스 코드 관리 호출만 사용될 예정이므로, Git에 익숙하지 않더라도 svn이나 Mercurial 같은 다른 버전 관리 시스템(VCS)에 익숙하다면 따라올 수 있습니다. 

코드 길이와 유닛테스트의 제약으로 인해, 테스트 탐색을 단순화했습니다. 저장소 내에서 `tests`라는 이름의 디렉터리에 있는 테스트*만* 실행할 것입니다.

지속적 통합 시스템은 일반적으로 웹 서버에 호스팅되어 있고 CI의 파일 시스템에 로컬로 있지 않은 마스터 저장소를 모니터링합니다. 우리 예제의 경우, 원격 저장소 대신 로컬 저장소를 사용할 것입니다.

지속적 통합 시스템은 고정된 정규 스케줄에 따라 실행될 필요가 없습니다. 몇 번의 커밋마다 또는 커밋별로 실행하도록 할 수도 있습니다. 우리 예제의 경우, CI 시스템은 주기적으로 실행될 것입니다. 즉, 5초 주기로 변경사항을 확인하도록 설정된 경우, 5초 주기 후에 이루어진 가장 최근 커밋에 대해 테스트를 실행할 것입니다. 해당 시간 간격 내에 이루어진 모든 커밋을 테스트하지 않고, 가장 최근 커밋만 테스트합니다.

이 CI 시스템은 저장소의 변경사항을 주기적으로 확인하도록 설계되었습니다. 실제 CI 시스템에서는 호스팅된 저장소에 의해 저장소 옵저버가 알림을 받을 수도 있습니다. 예를 들어, GitHub은 URL로 알림을 보내는 "포스트 커밋 훅"을 제공합니다. 이 모델을 따르면, 저장소 옵저버는 해당 알림에 응답하기 위해 그 URL에서 호스팅되는 웹 서버에 의해 호출될 것입니다. 이를 로컬로 모델링하기에는 복잡하므로, 우리는 저장소 옵저버가 알림을 받는 대신 변경사항을 확인하는 옵저버 모델을 사용하고 있습니다.

CI 시스템에는 또한 리포터 측면이 있는데, 여기서 테스트 실행기는 결과를 사람들이 볼 수 있도록 하는 컴포넌트에 보고합니다. 아마도 웹페이지에서 말이죠. 단순화를 위해, 이 프로젝트는 테스트 결과를 수집하여 디스패처 프로세스에 로컬인 파일 시스템에 파일로 저장합니다.

이 CI 시스템이 사용하는 아키텍처는 여러 가능성 중 하나일 뿐이라는 점을 유의하세요. 이 접근 방식은 우리의 사례 연구를 세 가지 주요 컴포넌트로 단순화하기 위해 선택되었습니다.

## 소개

지속적 통합 시스템의 기본 구조는 세 가지 컴포넌트로 구성됩니다: 옵저버(observer), 테스트 작업 디스패처(test job dispatcher), 그리고 테스트 실행기(test runner)입니다. 옵저버는 저장소를 감시합니다. 커밋이 이루어졌음을 알아차리면, 작업 디스패처에게 알립니다. 그러면 작업 디스패처는 테스트 실행기를 찾아 테스트할 커밋 번호를 제공합니다.

CI 시스템을 설계하는 방법은 여러 가지가 있습니다. 옵저버, 디스패처, 그리고 실행기를 단일 머신의 같은 프로세스로 할 수도 있습니다. 이 접근 방식은 부하 처리가 없기 때문에 매우 제한적입니다. 따라서 CI 시스템이 처리할 수 있는 것보다 더 많은 변경사항이 저장소에 추가되면, 큰 백로그가 누적됩니다. 이 접근 방식은 또한 전혀 장애 허용성이 없습니다. 실행 중인 컴퓨터가 실패하거나 정전이 발생하면, 백업 시스템이 없으므로 어떤 테스트도 실행되지 않습니다. 이상적인 시스템은 요청되는 만큼 많은 테스트 작업을 처리할 수 있고, 머신이 다운될 때 최선을 다해 보상하는 시스템일 것입니다.

장애 허용적이고 부하를 견딜 수 있는 CI 시스템을 구축하기 위해, 이 프로젝트에서는 이러한 각 컴포넌트가 각자의 프로세스입니다. 이렇게 하면 각 프로세스가 다른 것들과 독립적이 되고, 각 프로세스의 여러 인스턴스를 실행할 수 있게 됩니다. 이는 동시에 실행되어야 할 테스트 작업이 하나 이상 있을 때 유용합니다. 그러면 여러 테스트 실행기를 병렬로 생성하여 필요한 만큼 많은 작업을 실행할 수 있고, 대기 중인 테스트의 백로그가 누적되는 것을 방지할 수 있습니다.

이 프로젝트에서는 이러한 컴포넌트들이 별도의 프로세스로 실행될 뿐만 아니라, 소켓을 통해 통신하므로 각 프로세스를 별도의 네트워크 머신에서 실행할 수 있습니다. 각 컴포넌트에는 고유한 호스트/포트 주소가 할당되며, 각 프로세스는 할당된 주소에 메시지를 게시함으로써 다른 프로세스들과 통신할 수 있습니다.

이 설계는 분산 아키텍처를 가능하게 함으로써 하드웨어 장애를 즉석에서 처리할 수 있게 해줍니다. 옵저버를 한 머신에서, 테스트 작업 디스패처를 다른 머신에서, 테스트 실행기들을 또 다른 머신에서 실행할 수 있으며, 이들 모두가 네트워크를 통해 서로 통신할 수 있습니다. 이러한 머신 중 하나가 다운되면, 네트워크에 새 머신을 스케줄링할 수 있으므로 시스템이 장애 안전(fail-safe)이 됩니다.

이 프로젝트는 자동 복구 코드를 포함하지 않습니다. 이는 분산 시스템의 아키텍처에 의존하기 때문입니다. 하지만 실제로 CI 시스템들은 페일오버 이중화(failover redundancy)를 가질 수 있도록 이와 같은 분산 환경에서 실행됩니다(즉, 프로세스가 실행 중이던 머신 중 하나가 고장나면 대기 머신으로 폴백할 수 있습니다).

이 프로젝트의 목적상, 이러한 각 프로세스는 로컬에서 그리고 서로 다른 로컬 포트에서 수동으로 시작될 것입니다.

### 이 프로젝트의 파일들

이 프로젝트는 이러한 각 컴포넌트에 대한 Python 파일을 포함합니다: 저장소 옵저버(`repo_observer.py`), 테스트 작업 디스패처(`dispatcher.py`), 그리고 테스트 실행기(`test_runner.py`)입니다. 이 세 프로세스는 각각 소켓을 사용해서 서로 통신하며, 정보 전송에 사용되는 코드가 모든 프로세스에서 공유되므로, 이를 포함하는 helpers.py 파일이 있습니다. 각 프로세스는 파일에 코드를 복제하는 대신 여기서 통신 함수를 가져옵니다.

이러한 프로세스들에서 사용되는 bash 스크립트 파일들도 있습니다. 이 스크립트 파일들은 os나 subprocess 같은 Python의 운영체제 수준 모듈을 계속 사용하는 것보다 더 쉬운 방법으로 bash와 git 명령어를 실행하기 위해 사용됩니다.

마지막으로, CI 시스템이 실행할 두 개의 예제 테스트를 포함하는 tests 디렉터리가 있습니다. 하나의 테스트는 통과하고, 다른 하나는 실패할 것입니다.

### 초기 설정

이 CI 시스템은 분산 시스템에서 작동할 준비가 되어 있지만, 네트워크 관련 문제에 부딪힐 위험을 추가하지 않으면서 CI 시스템이 어떻게 작동하는지 파악할 수 있도록 우선 한 대의 컴퓨터에서 로컬로 모든 것을 실행해 보겠습니다. 분산 환경에서 실행하고 싶다면, 각 컴포넌트를 각자의 머신에서 실행할 수 있습니다.

지속적 통합 시스템은 코드 저장소의 변경사항을 감지하여 테스트를 실행하므로, 시작하려면 우리 CI 시스템이 모니터링할 저장소를 설정해야 합니다.

이것을 `test_repo`라고 부르겠습니다:

```bash
$ mkdir test_repo
$ cd test_repo
$ git init
```

이것이 우리의 마스터 저장소가 됩니다. 개발자들이 코드를 체크인하는 곳이므로, CI는 이 저장소를 풀하고 커밋을 확인한 다음 테스트를 실행해야 합니다. 새로운 커밋을 확인하는 것이 바로 저장소 옵저버입니다.

저장소 옵저버는 커밋을 확인하는 방식으로 작동하므로, 마스터 저장소에 최소한 하나의 커밋이 필요합니다. 실행할 테스트를 갖기 위해 예제 테스트를 커밋해보겠습니다.

이 코드베이스에서 tests 폴더를 `test_repo`로 복사하고 커밋합니다:

```bash
$ cp -r /this/directory/tests /path/to/test_repo/
$ cd /path/to/test\_repo
$ git add tests/
$ git commit -m "add tests"
```

이제 마스터 저장소에 커밋이 하나 있게 되었습니다.

저장소 옵저버 컴포넌트는 새로운 커밋이 만들어졌을 때를 감지할 수 있도록 자체적으로 코드의 클론이 필요합니다. 마스터 저장소의 클론을 만들고 이를 `test_repo_clone_obs`라고 부르겠습니다:

```bash
$ git clone /path/to/test_repo test_repo_clone_obs
```

테스트 실행기도 주어진 커밋에서 저장소를 체크아웃하고 테스트를 실행할 수 있도록 자체적으로 코드의 클론이 필요합니다. 마스터 저장소의 또 다른 클론을 만들고 이를 `test_repo_clone_runner`라고 부르겠습니다:

```bash
$ git clone /path/to/test_repo test_repo_clone_runner
```

## 컴포넌트들

### 저장소 옵저버 (`repo_observer.py`)

저장소 옵저버는 저장소를 모니터링하고 새로운 커밋이 발견되면 디스패처에게 알립니다. 모든 버전 관리 시스템과 작동하기 위해 (모든 VCS가 내장 알림 시스템을 갖고 있지 않기 때문에), 이 저장소 옵저버는 VCS가 변경사항이 만들어졌음을 알려주는 것에 의존하는 대신 주기적으로 저장소에서 새로운 커밋을 확인하도록 작성되었습니다.

옵저버는 저장소를 주기적으로 폴링하며, 변경사항이 발견되면 테스트를 실행할 가장 최신 커밋 ID를 디스패처에게 알려줍니다. 옵저버는 저장소의 현재 커밋 ID를 찾고, 저장소를 업데이트한 다음, 마지막으로 최신 커밋 ID를 찾아 비교함으로써 새로운 커밋을 확인합니다. 이 예제의 목적상, 옵저버는 가장 최신 커밋에 대해서만 테스트를 디스패치할 것입니다. 이는 주기적 확인 사이에 두 개의 커밋이 만들어지면, 옵저버는 가장 최신 커밋에 대해서만 테스트를 실행한다는 의미입니다. 일반적으로 CI 시스템은 마지막으로 테스트된 커밋 이후의 모든 커밋을 감지하고 각 새로운 커밋에 대해 테스트 실행기를 디스패치하지만, 단순화를 위해 이 가정을 수정했습니다.

옵저버는 어떤 저장소를 관찰할지 알아야 합니다. 우리는 이전에 `/path/to/test_repo_clone_obs`에 저장소의 클론을 만들었습니다. 옵저버는 이 클론을 사용하여 변경사항을 감지할 것입니다. 저장소 옵저버가 이 클론을 사용할 수 있도록, `repo_observer.py` 파일을 호출할 때 경로를 전달합니다. 저장소 옵저버는 이 클론을 사용하여 메인 저장소에서 풀할 것입니다.

또한 옵저버가 메시지를 보낼 수 있도록 디스패처의 주소를 제공해야 합니다. 저장소 옵저버를 시작할 때, `--dispatcher-server` 명령줄 인수를 사용하여 디스패처의 서버 주소를 전달할 수 있습니다. 전달하지 않으면, 기본 주소인 `localhost:8888`을 가정합니다.

```python 
def poll():
    parser = argparse.ArgumentParser()
    parser.add_argument("--dispatcher-server",
                        help="dispatcher host:port, " \
                        "by default it uses localhost:8888",
                        default="localhost:8888",
                        action="store")
    parser.add_argument("repo", metavar="REPO", type=str,
                        help="path to the repository this will observe")
    args = parser.parse_args()
    dispatcher_host, dispatcher_port = args.dispatcher_server.split(":")
```

저장소 옵저버 파일이 호출되면, `poll()` 함수를 시작합니다. 이 함수는 명령줄 인수를 파싱하고, 무한 while 루프를 시작합니다. while 루프는 저장소의 변경사항을 주기적으로 확인하는 데 사용됩니다. 가장 먼저 하는 일은 `update_repo.sh` Bash 스크립트를 호출하는 것입니다[^bash].

[^bash]: 파일 존재 여부 확인, 파일 생성, Git 사용이 필요하기 때문에 Bash를 사용하며, 셸 스크립트가 이를 달성하는 가장 직접적이고 쉬운 방법입니다. 또는 크로스 플랫폼 Python 패키지를 사용할 수도 있습니다; 예를 들어, 파일 시스템 접근을 위해 Python의 내장 `os` 모듈을 사용하고, Git 접근을 위해 GitPython을 사용할 수 있지만, 이들은 더 우회적인 방식으로 작업을 수행합니다.

```python
    while True:
        try:
            # call the bash script that will update the repo and check
            # for changes. If there's a change, it will drop a .commit_id file
            # with the latest commit in the current working directory
            subprocess.check_output(["./update_repo.sh", args.repo])
        except subprocess.CalledProcessError as e:
            raise Exception("Could not update and check repository. " +
                            "Reason: %s" % e.output)
```

`update_repo.sh` 파일은 새로운 커밋을 식별하고 저장소 옵저버에게 알리는 데 사용됩니다. 현재 우리가 알고 있는 커밋 ID를 기록하고, 저장소를 풀하고, 최신 커밋 ID를 확인함으로써 이를 수행합니다. 만약 일치한다면 변경사항이 없으므로 저장소 옵저버는 아무것도 할 필요가 없지만, 커밋 ID에 차이가 있다면 새로운 커밋이 만들어졌다는 것을 알 수 있습니다. 이 경우, `update_repo.sh`는 최신 커밋 ID가 저장된 `.commit_id`라는 파일을 생성합니다.

`update_repo.sh`의 단계별 분석은 다음과 같습니다. 먼저, 스크립트는 모든 셸 스크립트에서 사용되는 `run_or_fail` 헬퍼 메소드를 제공하는 `run_or_fail.sh` 파일을 소스합니다. 이 메소드는 주어진 명령을 실행하거나 주어진 오류 메시지와 함께 실패하는 데 사용됩니다.

```bash 
#!/bin/bash

source run_or_fail.sh 
```

다음으로, 스크립트는 `.commit_id`라는 파일을 제거하려고 시도합니다. `updaterepo.sh`는 `repo_observer.py` 파일에 의해 무한히 호출되므로, 이전에 새로운 커밋이 있었다면 `.commit_id`가 생성되었지만 이미 테스트한 커밋을 담고 있습니다. 따라서 그 파일을 제거하고, 새로운 커밋이 발견된 경우에만 새로운 파일을 생성하려고 합니다.

```bash
bash rm -f .commit_id
```

파일을 제거한 후 (존재했다면), 관찰하고 있는 저장소가 존재하는지 확인하고, 어떤 것이 동기화가 맞지 않게 했을 경우에 대비해 가장 최근 커밋으로 리셋합니다.

```bash
run_or_fail "Repository folder not found!" pushd $1 1> /dev/null
run_or_fail "Could not reset git" git reset --hard HEAD
```

그런 다음 git log를 호출하고 출력을 파싱하여 가장 최근 커밋 ID를 찾습니다.

```bash
COMMIT=$(run_or_fail "Could not call 'git log' on repository" git log -n1)
if [ $? != 0 ]; then
  echo "Could not call 'git log' on repository"
  exit 1
fi
COMMIT_ID=`echo $COMMIT | awk '{ print $2 }'`
```

그런 다음 저장소를 풀하여 최근 변경사항을 가져오고, 가장 최근 커밋 ID를 얻습니다.

```bash
run_or_fail "Could not pull from repository" git pull
COMMIT=$(run_or_fail "Could not call 'git log' on repository" git log -n1)
if [ $? != 0 ]; then
  echo "Could not call 'git log' on repository"
  exit 1
fi
NEW_COMMIT_ID=`echo $COMMIT | awk '{ print $2 }'`
```

마지막으로, 커밋 ID가 이전 ID와 일치하지 않으면 확인할 새로운 커밋이 있다는 것을 알 수 있으므로, 스크립트는 최신 커밋 ID를 .commit\_id 파일에 저장합니다.

```bash
# if the id changed, then write it to a file
if [ $NEW_COMMIT_ID != $COMMIT_ID ]; then
  popd 1> /dev/null
  echo $NEW_COMMIT_ID > .commit_id
fi
```

`repo_observer.py`에서 `update_repo.sh`가 실행을 마치면, 저장소 옵저버는 `.commit_id` 파일의 존재를 확인합니다. 파일이 존재한다면, 새로운 커밋이 있다는 것을 알 수 있고, 테스트를 시작할 수 있도록 디스패처에게 알려야 합니다. 저장소 옵저버는 디스패처 서버에 연결하고 'status' 요청을 보내 디스패처 서버의 상태를 확인하여, 문제가 없는지, 그리고 지시를 받을 준비가 되어 있는지 확인합니다.

```python
        if os.path.isfile(".commit_id"):
            try:
                response = helpers.communicate(dispatcher_host,
                                               int(dispatcher_port),
                                               "status")
            except socket.error as e:
                raise Exception("Could not communicate with dispatcher server: %s" % e)
```

"OK"로 응답하면, 저장소 옵저버는 `.commit_id` 파일을 열고, 최신 커밋 ID를 읽어 `dispatch:<commit ID>` 요청을 사용하여 그 ID를 디스패처에게 보냅니다. 그런 다음 5초 동안 잠시 정지하고 프로세스를 반복합니다. 도중에 문제가 발생하면 5초 후에 다시 시도합니다.

```python
            if response == "OK":
                commit = ""
                with open(".commit_id", "r") as f:
                    commit = f.readline()
                response = helpers.communicate(dispatcher_host,
                                               int(dispatcher_port),
                                               "dispatch:%s" % commit)
                if response != "OK":
                    raise Exception("Could not dispatch the test: %s" %
                    response)
                print "dispatched!"
            else:
                raise Exception("Could not dispatch the test: %s" %
                response)
        time.sleep(5)
```

저장소 옵저버는 `KeyboardInterrupt` (Ctrl+c)를 통해 프로세스를 종료하거나 kill 신호를 보내기 전까지 이 프로세스를 영원히 반복할 것입니다.

### 디스패처 (`dispatcher.py`)

디스패처는 테스트 작업을 위임하는 데 사용되는 별도의 서비스입니다. 테스트 실행기와 저장소 옵저버로부터의 요청을 포트에서 수신합니다. 테스트 실행기들이 자신을 등록할 수 있게 해주며, 저장소 옵저버로부터 커밋 ID가 주어지면 새로운 커밋에 대해 테스트 실행기를 디스패치합니다. 또한 테스트 실행기들의 문제를 우아하게 처리하고, 문제가 발생하면 커밋 ID를 새로운 테스트 실행기로 재분배합니다.

`dispatch.py`가 실행되면, `serve` 함수가 호출됩니다. 먼저 디스패처의 호스트와 포트를 지정할 수 있는 인수를 파싱합니다:

```python
def serve():
    parser = argparse.ArgumentParser()
    parser.add_argument("--host",
                        help="dispatcher's host, by default it uses localhost",
                        default="localhost",
                        action="store")
    parser.add_argument("--port",
                        help="dispatcher's port, by default it uses 8888",
                        default=8888,
                        action="store")
    args = parser.parse_args()
```

이것은 디스패처 서버와 두 개의 다른 스레드를 시작합니다. 하나의 스레드는 `runner_checker` 함수를 실행하고, 다른 하나는 `redistribute` 함수를 실행합니다.

```python
    server = ThreadingTCPServer((args.host, int(args.port)), DispatcherHandler)
    print `serving on %s:%s` % (args.host, int(args.port))

    ...

    runner_heartbeat = threading.Thread(target=runner_checker, args=(server,))
    redistributor = threading.Thread(target=redistribute, args=(server,))
    try:
        runner_heartbeat.start()
        redistributor.start()
        # Activate the server; this will keep running until you
        # interrupt the program with Ctrl+C or Cmd+C
        server.serve_forever()
    except (KeyboardInterrupt, Exception):
        # if any exception occurs, kill the thread
        server.dead = True
        runner_heartbeat.join()
        redistributor.join()

```

`runner_checker` 함수는 각 등록된 테스트 실행기에 주기적으로 핑을 보내 여전히 응답하는지 확인합니다. 응답하지 않으면, 해당 실행기는 풀에서 제거되고 그 커밋 ID는 사용 가능한 다음 실행기로 디스패치됩니다. 함수는 커밋 ID를 `pending_commits` 변수에 로깅합니다.

```python
    def runner_checker(server):
        def manage_commit_lists(runner):
            for commit, assigned_runner in server.dispatched_commits.iteritems():
                if assigned_runner == runner:
                    del server.dispatched_commits[commit]
                    server.pending_commits.append(commit)
                    break
            server.runners.remove(runner)
        while not server.dead:
            time.sleep(1)
            for runner in server.runners:
                s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                try:
                    response = helpers.communicate(runner["host"],
                                                   int(runner["port"]),
                                                   "ping")
                    if response != "pong":
                        print "removing runner %s" % runner
                        manage_commit_lists(runner)
                except socket.error as e:
                    manage_commit_lists(runner)
```

`redistribute` 함수는 `pending_commits`에 로깅된 커밋 ID를 디스패치하는 데 사용됩니다. `redistribute`가 실행되면, `pending_commits`에 커밋 ID가 있는지 확인합니다. 있다면, 커밋 ID와 함께 `dispatch_tests` 함수를 호출합니다.

```python
    def redistribute(server):
        while not server.dead:
            for commit in server.pending_commits:
                print "running redistribute"
                print server.pending_commits
                dispatch_tests(server, commit)
                time.sleep(5)
```

`dispatch_tests` 함수는 등록된 실행기 풀에서 사용 가능한 테스트 실행기를 찾는 데 사용됩니다. 사용 가능한 실행기가 있으면, 커밋 ID와 함께 runtest 메시지를 보냅니다. 현재 사용 가능한 실행기가 없다면, 2초를 기다리고 이 과정을 반복합니다. 디스패치되면, 어떤 테스트 실행기가 어떤 커밋 ID를 테스트하고 있는지 `dispatched_commits` 변수에 로깅합니다. 커밋 ID가 `pending_commits` 변수에 있다면, `dispatch_tests`는 이미 성공적으로 재디스패치되었으므로 이를 제거합니다.

```python
def dispatch_tests(server, commit_id):
    # NOTE: usually we don't run this forever
    while True:
        print "trying to dispatch to runners"
        for runner in server.runners:
            response = helpers.communicate(runner["host"],
                                           int(runner["port"]),
                                           "runtest:%s" % commit_id)
            if response == "OK":
                print "adding id %s" % commit_id
                server.dispatched_commits[commit_id] = runner
                if commit_id in server.pending_commits:
                    server.pending_commits.remove(commit_id)
                return
        time.sleep(2)
```

디스패처 서버는 표준 라이브러리의 일부인 매우 간단한 서버인 `SocketServer` 모듈을 사용합니다. `SocketServer` 모듈에는 네 가지 기본 서버 타입이 있습니다: `TCP`, `UDP`, `UnixStreamServer`, `UnixDatagramServer`. UDP는 이를 보장하지 않으므로, 서버 간의 지속적이고 순서가 보장된 데이터 스트림을 확실히 하기 위해 TCP 기반 소켓 서버를 사용할 것입니다.

`SocketServer`에서 제공하는 기본 `TCPServer`는 한 번에 하나의 요청만 처리할 수 있으므로, 디스패처가 테스트 실행기로부터의 연결과 통신하고 있을 때 저장소 옵저버로부터 새로운 연결이 들어오는 경우를 처리할 수 없습니다. 이런 일이 발생하면, 저장소 옵저버는 첫 번째 연결이 완료되고 연결 해제될 때까지 기다려야 서비스를 받을 수 있습니다. 디스패처 서버가 모든 테스트 실행기와 저장소 옵저버와 직접적이고 신속하게 통신할 수 있어야 하므로, 이는 우리의 경우에 이상적이지 않습니다.

디스패처 서버가 동시 연결을 처리할 수 있도록, 기본 `SocketServer`에 스레딩 기능을 추가하는 `ThreadingTCPServer` 사용자 정의 클래스를 사용합니다. 이는 디스패처가 연결 요청을 받을 때마다 해당 연결만을 위한 새로운 프로세스를 생성한다는 의미입니다. 이렇게 하면 디스패처가 동시에 여러 요청을 처리할 수 있습니다.

```python
class ThreadingTCPServer(SocketServer.ThreadingMixIn, SocketServer.TCPServer):
    runners = [] # Keeps track of test runner pool
    dead = False # Indicate to other threads that we are no longer running
    dispatched_commits = {} # Keeps track of commits we dispatched
    pending_commits = [] # Keeps track of commits we have yet to dispatch
```

디스패처 서버는 각 요청에 대한 핸들러를 정의함으로써 작동합니다. 이는 `SocketServer`의 `BaseRequestHandler`를 상속받는 `DispatcherHandler` 클래스로 정의됩니다. 이 기본 클래스는 연결이 요청될 때마다 호출될 handle 함수만 정의하면 됩니다. `DispatcherHandler`에서 정의된 handle 함수는 우리의 사용자 정의 핸들러이며, 각 연결에서 호출됩니다. 들어오는 연결 요청을 살펴보고 (`self.request`가 요청 정보를 담고 있음), 어떤 명령이 요청되고 있는지 파싱합니다.

```python
class DispatcherHandler(SocketServer.BaseRequestHandler):
    """
    The RequestHandler class for our dispatcher.
    This will dispatch test runners against the incoming commit
    and handle their requests and test results
    """
    command_re = re.compile(r"(\w+)(:.+)*")
    BUF_SIZE = 1024
    def handle(self):
        self.data = self.request.recv(self.BUF_SIZE).strip()
        command_groups = self.command_re.match(self.data)
        if not command_groups:
            self.request.sendall("Invalid command")
            return
        command = command_groups.group(1)
```

네 가지 명령을 처리합니다: `status`, `register`, `dispatch`, `results`.
`status`는 디스패처 서버가 가동되고 실행 중인지 확인하는 데 사용됩니다.

```python
        if command == "status":
            print "in status"
            self.request.sendall("OK")
```

디스패처가 유용한 작업을 수행하려면, 최소한 하나의 테스트 실행기가 등록되어 있어야 합니다. host:port 쌍에 register가 호출되면, 나중에 테스트를 실행할 커밋 ID를 제공해야 할 때 실행기와 통신할 수 있도록 실행기의 정보를 리스트에 저장합니다 (`ThreadingTCPServer` 객체에 연결된 runners 객체).

```python
        elif command == "register":
            # Add this test runner to our pool
            print "register"
            address = command_groups.group(2)
            host, port = re.findall(r":(\w*)", address)
            runner = {"host": host, "port":port}
            self.server.runners.append(runner)
            self.request.sendall("OK")
```

`dispatch`는 저장소 옵저버가 커밋에 대해 테스트 실행기를 디스패치하는 데 사용됩니다. 이 명령의 형식은 `dispatch:<commit ID>`입니다. 디스패처는 이 메시지에서 커밋 ID를 파싱하고 테스트 실행기로 보냅니다.

```python
        elif command == "dispatch":
            print "going to dispatch"
            commit_id = command_groups.group(2)[1:]
            if not self.server.runners:
                self.request.sendall("No runners are registered")
            else:
                # The coordinator can trust us to dispatch the test
                self.request.sendall("OK")
                dispatch_tests(self.server, commit_id)
```

`results`는 테스트 실행기가 완료된 테스트 실행의 결과를 보고하는 데 사용됩니다. 이 명령의 형식은 `results:<commit ID>:<length of results data in bytes>:<results>`입니다. `<commit ID>`는 테스트가 어떤 커밋 ID에 대해 실행되었는지 식별하는 데 사용됩니다. `<length of results data in bytes>`는 결과 데이터에 얼마나 큰 버퍼가 필요한지 파악하는 데 사용됩니다. 마지막으로, `<results>`는 실제 결과 출력을 담고 있습니다.

```python
        elif command == "results":
            print "got test results"
            results = command_groups.group(2)[1:]
            results = results.split(":")
            commit_id = results[0]
            length_msg = int(results[1])
            # 3 is the number of ":" in the sent command
            remaining_buffer = self.BUF_SIZE - \
                (len(command) + len(commit_id) + len(results[1]) + 3)
            if length_msg > remaining_buffer:
                self.data += self.request.recv(length_msg - remaining_buffer).strip()
            del self.server.dispatched_commits[commit_id]
            if not os.path.exists("test_results"):
                os.makedirs("test_results")
            with open("test_results/%s" % commit_id, "w") as f:
                data = self.data.split(":")[3:]
                data = "\n".join(data)
                f.write(data)
            self.request.sendall("OK")
```

### 테스트 실행기 (`test_runner.py`)

테스트 실행기는 주어진 커밋 ID에 대해 테스트를 실행하고 결과를 보고하는 책임이 있습니다. 실행할 커밋 ID를 제공하고 테스트 결과를 받을 디스패처 서버와만 통신합니다.

`test_runner.py` 파일이 호출되면, 테스트 실행기 서버를 시작하고 `dispatcher_checker` 함수를 실행하는 스레드도 시작하는 `serve` 함수를 호출합니다. 이 시작 과정은 `repo_observer.py`와 `dispatcher.py`에서 설명된 것과 매우 유사하므로, 여기서는 설명을 생략합니다.

`dispatcher_checker` 함수는 디스패처 서버에 5초마다 핑을 보내 여전히 가동되고 실행 중인지 확인합니다. 이는 리소스 관리에 중요합니다. 디스패처가 다운되면, 작업을 제공하거나 보고할 디스패처가 없으면 의미있는 작업을 수행할 수 없으므로 테스트 실행기가 종료됩니다.

```python
    def dispatcher_checker(server):
        while not server.dead:
            time.sleep(5)
            if (time.time() - server.last_communication) > 10:
                try:
                    response = helpers.communicate(
                                       server.dispatcher_server["host"],
                                       int(server.dispatcher_server["port"]),
                                       "status")
                    if response != "OK":
                        print "Dispatcher is no longer functional"
                        server.shutdown()
                        return
                except socket.error as e:
                    print "Can't communicate with dispatcher: %s" % e
                    server.shutdown()
                    return
```

테스트 실행기는 디스패처 서버와 같은 `ThreadingTCPServer`입니다. 디스패처가 실행할 커밋 ID를 제공할 뿐만 아니라 테스트를 실행하는 동안 여전히 가동 중인지 확인하기 위해 주기적으로 실행기에 핑을 보낼 것이므로 스레딩이 필요합니다.

```python
class ThreadingTCPServer(SocketServer.ThreadingMixIn, SocketServer.TCPServer):
    dispatcher_server = None # Holds the dispatcher server host/port information
    last_communication = None # Keeps track of last communication from dispatcher
    busy = False # Status flag
    dead = False # Status flag
```

통신 흐름은 디스패처가 실행기에게 실행할 커밋 ID를 받아들이도록 요청하는 것으로 시작됩니다. 테스트 실행기가 작업을 실행할 준비가 되어 있다면, 디스패처 서버에 승인으로 응답하고, 디스패처는 연결을 종료합니다. 테스트 실행기 서버가 테스트를 실행하면서 동시에 디스패처로부터 더 많은 요청을 받을 수 있도록, 요청된 테스트 작업을 새로운 스레드에서 시작합니다.

이는 디스패처 서버가 요청을 하고 (이 경우 핑) 응답을 기대할 때, 테스트 실행기가 자체 스레드에서 테스트를 실행하느라 바쁜 동안 별도의 스레드에서 수행된다는 의미입니다. 이렇게 하면 테스트 실행기 서버가 동시에 여러 작업을 처리할 수 있습니다. 이런 스레드 설계 대신, 디스패처 서버가 각 테스트 실행기와의 연결을 유지하는 것도 가능하지만, 이는 디스패처 서버의 메모리 요구사항을 증가시키고 우발적인 연결 끊김과 같은 네트워크 문제에 취약합니다.

테스트 실행기 서버는 디스패처로부터 두 메시지에 응답합니다. 첫 번째는 `ping`으로, 디스패처 서버가 실행기가 여전히 활성 상태인지 확인하는 데 사용됩니다.

```python
class TestHandler(SocketServer.BaseRequestHandler):
    ...

    def handle(self):
        ....
        if command == "ping":
            print "pinged"
            self.server.last_communication = time.time()
            self.request.sendall("pong")
```

두 번째는 `runtest`로, `runtest:<commit ID>` 형식의 메시지를 받으며, 주어진 커밋에 대한 테스트를 시작하는 데 사용됩니다. runtest가 호출되면, 테스트 실행기는 이미 테스트를 실행 중인지 확인하고, 그렇다면 디스패처에게 `BUSY` 응답을 반환합니다. 사용 가능하다면, 서버에 `OK` 메시지로 응답하고, 상태를 바쁨으로 설정하며 `run_tests` 함수를 실행합니다.

```python
        elif command == "runtest":
            print "got runtest command: am I busy? %s" % self.server.busy
            if self.server.busy:
                self.request.sendall("BUSY")
            else:
                self.request.sendall("OK")
                print "running"
                commit_id = command_groups.group(2)[1:]
                self.server.busy = True
                self.run_tests(commit_id,
                               self.server.repo_folder)
                self.server.busy = False

```

이 함수는 저장소를 주어진 커밋 ID로 업데이트하는 셸 스크립트 `test_runner_script.sh`를 호출합니다. 스크립트가 반환되면, 저장소 업데이트가 성공했다면 unittest를 사용하여 테스트를 실행하고 결과를 파일에 수집합니다. 테스트 실행이 완료되면, 테스트 실행기는 결과 파일을 읽어 results 메시지로 디스패처에게 보냅니다.

```python
    def run_tests(self, commit_id, repo_folder):
        # update repo
        output = subprocess.check_output(["./test_runner_script.sh",
                                        repo_folder, commit_id])
        print output
        # run the tests
        test_folder = os.path.join(repo_folder, "tests")
        suite = unittest.TestLoader().discover(test_folder)
        result_file = open("results", "w")
        unittest.TextTestRunner(result_file).run(suite)
        result_file.close()
        result_file = open("results", "r")
        # give the dispatcher the results
        output = result_file.read()
        helpers.communicate(self.server.dispatcher_server["host"],
                            int(self.server.dispatcher_server["port"]),
                            "results:%s:%s:%s" % (commit_id, len(output), output))
```

Here's `test_runner_script.sh`:

```bash
#!/bin/bash
REPO=$1
COMMIT=$2
source run_or_fail.sh
run_or_fail "Repository folder not found" pushd "$REPO" 1> /dev/null
run_or_fail "Could not clean repository" git clean -d -f -x
run_or_fail "Could not call git pull" git pull
run_or_fail "Could not update to given commit hash" git reset --hard "$COMMIT"
```

`test_runner.py`를 실행하려면, 테스트를 실행할 저장소의 클론을 가리켜야 합니다. 이 경우, 이전에 생성한 `/path/to/test_repo test_repo_clone_runner` 클론을 인수로 사용할 수 있습니다. 기본적으로, `test_runner.py`는 8900-9000 범위의 포트를 사용하여 localhost에서 자체 서버를 시작하고, `localhost:8888`에서 디스패처 서버에 연결하려고 시도합니다. 이러한 값을 변경하기 위해 선택적 인수를 전달할 수 있습니다. `--host`와 `--port` 인수는 테스트 실행기 서버를 실행할 특정 주소를 지정하는 데 사용되며, `--dispatcher-server` 인수는 디스패처의 주소를 지정합니다.

### 제어 흐름 다이어그램

\aosafigref{500l.ci.controlflow}는 이 시스템의 개요 다이어그램입니다. 이 다이어그램은 세 개의 파일 (`repo_observer.py`, `dispatcher.py`, `test_runner.py`)이 모두 이미 실행 중이라고 가정하고, 새로운 커밋이 만들어졌을 때 각 프로세스가 취하는 동작을 설명합니다.

\aosafigure[360pt]{ci-images/diagram.png}{제어 흐름}{500l.ci.controlflow}

### 코드 실행

각 프로세스에 대해 세 개의 서로 다른 터미널 셸을 사용하여 이 간단한 CI 시스템을 로컬에서 실행할 수 있습니다. 포트 8888에서 실행되는 디스패처를 먼저 시작합니다:

```bash
$ python dispatcher.py
```

새로운 셸에서, (디스패처에 자체 등록할 수 있도록) 테스트 실행기를 시작합니다:

```bash
$ python test_runner.py <path/to/test_repo_clone_runner>
```

테스트 실행기는 8900-9000 범위에서 자체 포트를 할당할 것입니다. 원하는 만큼 많은 테스트 실행기를 실행할 수 있습니다.

마지막으로, 또 다른 새로운 셸에서, 저장소 옵저버를 시작해보겠습니다:

```bash
$ python repo_observer.py --dispatcher-server=localhost:8888 <path/to/repo_clone_obs>
```

이제 모든 것이 설정되었으므로, 몇 가지 테스트를 트리거해보겠습니다! 이를 위해서는 새로운 커밋을 만들어야 합니다. 마스터 저장소로 이동하여 임의의 변경을 만드세요:

```bash
$ cd /path/to/test_repo
$ touch new_file
$ git add new_file
$ git commit -m"new file" new_file
```

그러면 `repo_observer.py`가 새로운 커밋이 있다는 것을 인식하고 디스패처에게 알릴 것입니다. 각각의 셸에서 출력을 볼 수 있으므로 모니터링할 수 있습니다. 디스패처가 테스트 결과를 받으면, 커밋 ID를 파일 이름으로 사용하여 이 코드베이스의 `test_results/` 폴더에 저장합니다.

## 오류 처리

이 CI 시스템은 몇 가지 간단한 오류 처리를 포함합니다.

`test_runner.py` 프로세스를 종료하면, `dispatcher.py`가 실행기를 더 이상 사용할 수 없다는 것을 파악하고 풀에서 제거합니다.

머신 크래시나 네트워크 장애를 시뮬레이션하기 위해 테스트 실행기를 종료할 수도 있습니다. 그렇게 하면, 디스패처가 실행기가 다운되었다는 것을 인식하고 풀에 사용 가능한 다른 테스트 실행기가 있다면 그 작업을 줄 것이고, 또는 새로운 테스트 실행기가 풀에 자체 등록할 때까지 기다릴 것입니다.

디스패처를 종료하면, 저장소 옵저버가 다운되었다는 것을 파악하고 예외를 던집니다. 테스트 실행기들도 이를 인지하고 종료됩니다.

## 결론

관심사를 각자의 프로세스로 분리함으로써, 분산 지속적 통합 시스템의 기초를 구축할 수 있었습니다. 프로세스들이 소켓 요청을 통해 서로 통신하므로, 시스템을 여러 머신에 분산시킬 수 있어 시스템을 더 안정적이고 확장 가능하게 만드는 데 도움이 됩니다.

CI 시스템이 지금은 상당히 간단하므로, 훨씬 더 기능적이 되도록 직접 확장할 수 있습니다. 다음은 개선을 위한 몇 가지 제안입니다:

### 커밋별 테스트 실행

현재 시스템은 새로운 커밋이 실행되는지 주기적으로 확인하고 가장 최근 커밋을 실행합니다. 이는 각 커밋을 테스트하도록 개선되어야 합니다. 이를 위해, 주기적 검사기를 수정하여 마지막으로 테스트된 커밋과 최신 커밋 사이의 로그에 있는 각 커밋에 대해 테스트 실행을 디스패치할 수 있습니다.

### 더 똑똑한 테스트 실행기

테스트 실행기가 디스패처가 응답하지 않는다는 것을 감지하면, 실행을 중단합니다. 이는 테스트 실행기가 테스트를 실행하는 도중에도 발생합니다! 테스트 실행기가 디스패처가 온라인으로 돌아올 때까지 일정 시간 동안 (또는 리소스 관리에 신경 쓰지 않는다면 무기한으로) 기다리는 것이 더 나을 것입니다. 이 경우, 테스트 실행기가 적극적으로 테스트를 실행하는 동안 디스패처가 다운되면, 종료하는 대신 테스트를 완료하고 디스패처가 온라인으로 돌아올 때까지 기다리며, 결과를 보고할 것입니다. 이렇게 하면 테스트 실행기가 기울인 노력을 낭비하지 않고, 커밋당 한 번만 테스트를 실행하게 됩니다.

### 실제 보고

실제 CI 시스템에서는, 테스트 결과를 수집하고, 사람들이 검토할 수 있는 곳에 게시하며, 실패나 다른 주목할 만한 이벤트가 발생할 때 관심 있는 당사자들에게 알리는 리포터 서비스에 테스트 결과를 보고할 것입니다. 디스패처가 결과를 수집하는 대신, 보고된 결과를 얻는 새로운 프로세스를 생성함으로써 우리의 간단한 CI 시스템을 확장할 수 있습니다. 이 새로운 프로세스는 웹 서버가 될 수 있고 (또는 웹 서버에 연결할 수 있음), 결과를 온라인에 게시할 수 있으며, 메일 서버를 사용하여 구독자들에게 테스트 실패를 알릴 수 있습니다.

### 테스트 실행기 관리자

지금은 테스트 실행기를 시작하기 위해 `test_runner.py` 파일을 수동으로 실행해야 합니다. 대신, 디스패처로부터의 테스트 요청의 현재 부하를 평가하고 그에 따라 활성 테스트 실행기의 수를 조정하는 테스트 실행기 관리자 프로세스를 만들 수 있습니다. 이 프로세스는 runtest 메시지를 받고 각 요청에 대해 테스트 실행기 프로세스를 시작하며, 부하가 감소할 때 사용되지 않는 프로세스를 종료할 것입니다.

이러한 제안을 사용하면, 이 간단한 CI 시스템을 더 견고하고 장애 허용적으로 만들 수 있고, 웹 기반 테스트 리포터와 같은 다른 시스템과 통합할 수 있습니다.

지속적 통합 시스템이 달성할 수 있는 유연성 수준을 보고 싶다면, Java로 작성된 매우 견고한 오픈 소스 CI 시스템인 [Jenkins](<http://jenkins-ci.org/>)를 살펴보기를 권합니다. 플러그인을 사용하여 확장할 수 있는 기본 CI 시스템을 제공합니다. [GitHub를 통해](<https://github.com/jenkinsci/jenkins/>) 소스 코드에도 접근할 수 있습니다. 또 다른 추천 프로젝트는 Ruby로 작성된 [Travis CI](<https://travis-ci.org/>)이며, 그 소스 코드도 [GitHub를 통해](<https://github.com/travis-ci/travis-ci>) 이용할 수 있습니다.

이는 CI 시스템이 어떻게 작동하는지, 그리고 직접 시스템을 구축하는 방법을 이해하는 연습이었습니다. 이제 신뢰할 수 있는 분산 시스템을 만드는 데 필요한 것에 대한 더 확고한 이해를 갖게 되었으며, 이 지식을 사용하여 더 복잡한 솔루션을 개발할 수 있습니다.
