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

Let's call this `test_repo`:

```bash
$ mkdir test_repo 
$ cd test_repo 
$ git init
```

This will be our master repository. This is where developers check in
their code, so our CI should pull this repository and check for commits,
then run tests. The thing that checks for new commits is the repository
observer.

The repository observer works by checking commits, so we need at least
one commit in the master repository. Let’s commit our example tests so
we have some tests to run.

Copy the tests folder from this code base to `test_repo` and commit it:

```bash
$ cp -r /this/directory/tests /path/to/test_repo/ 
$ cd /path/to/test\_repo 
$ git add tests/ 
$ git commit -m ”add tests”
```

Now you have a commit in the master repository.

The repo observer component will need its own clone of the code, so it
can detect when a new commit is made. Let's create a clone of our master
repository, and call it `test_repo_clone_obs`:

```bash
$ git clone /path/to/test_repo test_repo_clone_obs
```


The test runner will also need its own clone of the code, so it can
checkout the repository at a given commit and run the tests. Let's
create another clone of our master repository, and call it
`test_repo_clone_runner`:

```bash
$ git clone /path/to/test_repo test_repo_clone_runner
```

## The Components

### The Repository Observer (`repo_observer.py`)

The repository observer monitors a repository and notifies the
dispatcher when a new commit is seen. In order to work with all version
control systems (since not all VCSs have built-in notification systems),
this repository observer is written to periodically check the repository
for new commits instead of relying on the VCS to notify it that changes
have been made.

The observer will poll the repository periodically, and when a change is
seen, it will tell the dispatcher the newest commit ID to run tests
against. The observer checks for new commits by finding the current
commit ID in its repository, then updates the repository, and lastly, it
finds the latest commit ID and compares them. For the purposes of this
example, the observer will only dispatch tests against the latest
commit. This means that if two commits are made between a periodic
check, the observer will only run tests against the latest commit.
Usually, a CI system will detect all commits since the last tested
commit, and will dispatch test runners for each new commit, but I have
modified this assumption for simplicity.

The observer must know which repository to observe. We previously
created a clone of our repository at `/path/to/test_repo_clone_obs`.
The observer will use this clone to detect changes. To allow the
repository observer to use this clone, we pass it the path when we
invoke the `repo_observer.py` file. The repository observer will use this
clone to pull from the main repository.

We must also give the observer the dispatcher's address, so the observer
may send it messages. When you start the repository observer, you can
pass in the dispatcher's server address using the `--dispatcher-server`
command line argument. If you do not pass it in, it will assume the
default address of `localhost:8888`.

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

Once the repository observer file is invoked, it starts the `poll()`
function. This function parses the command line arguments, and then
kicks off an infinite while loop. The while loop is used to periodically
check the repository for changes. The first thing it does is call the
`update_repo.sh` Bash script [^bash].

[^bash]: Bash is used because we need to check file
existence, create files, and use Git, and a shell script is the most
direct and easy way to achieve this. Alternatively, there are
cross-platform Python packages you can use; for example, Python's `os`
built-in module can be used for accessing the file system, and GitPython
can be used for Git access, but they perform actions in a more
roundabout way.

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

The `update_repo.sh` file is used to identify any new commits and let the
repository observer know. It does this by noting what commit ID we are
currently aware of, then pulls the repository, and checks the latest
commit ID. If they match, no changes are made, so the repository
observer doesn't need to do anything, but if there is a difference in
the commit ID, then we know a new commit has been made. In this case,
`update_repo.sh` will create a file called `.commit_id` with the latest
commit ID stored in it.

A step-by-step breakdown of `update_repo.sh` is as follows. First, the
script sources the `run_or_fail.sh` file, which provides the
`run_or_fail` helper method used by all our shell scripts. This method
is used to run the given command, or fail with the given error message.

```bash 
#!/bin/bash

source run_or_fail.sh 
```

Next, the script tries to remove a file named `.commit_id`. Since
`updaterepo.sh` is called infinitely by the `repo_observer.py` file, if we
previously had a new commit, then `.commit_id` was created, but holds a
commit we already tested. Therefore, we want to remove that file, and
create a new one only if a new commit is found.

```bash
bash rm -f .commit_id 
```

After it removes the file (if it existed), it verifies that the
repository we are observing exists, and then resets it to the most
recent commit, in case anything caused it to get out of sync.

```bash
run_or_fail "Repository folder not found!" pushd $1 1> /dev/null
run_or_fail "Could not reset git" git reset --hard HEAD
```

It then calls git log and parses the output, looking for the most recent
commit ID.

```bash
COMMIT=$(run_or_fail "Could not call 'git log' on repository" git log -n1)
if [ $? != 0 ]; then
  echo "Could not call 'git log' on repository"
  exit 1
fi
COMMIT_ID=`echo $COMMIT | awk '{ print $2 }'`
```

Then it pulls the repository, getting any recent changes, then gets the
most recent commit ID.

```bash
run_or_fail "Could not pull from repository" git pull
COMMIT=$(run_or_fail "Could not call 'git log' on repository" git log -n1)
if [ $? != 0 ]; then
  echo "Could not call 'git log' on repository"
  exit 1
fi
NEW_COMMIT_ID=`echo $COMMIT | awk '{ print $2 }'`
```

Lastly, if the commit ID doesn't match the previous ID, then we know we
have new commits to check, so the script stores the latest commit ID in
a .commit\_id file.

```bash
# if the id changed, then write it to a file
if [ $NEW_COMMIT_ID != $COMMIT_ID ]; then
  popd 1> /dev/null
  echo $NEW_COMMIT_ID > .commit_id
fi
```

When `update_repo.sh` finishes running in `repo_observer.py`, the
repository observer checks for the existence of the `.commit_id` file. If
the file does exist, then we know we have a new commit, and we need to
notify the dispatcher so it can kick off the tests. The repository
observer will check the dispatcher server's status by connecting to it
and sending a 'status' request, to make sure there are no problems with
it, and to make sure it is ready for instruction.

```python
        if os.path.isfile(".commit_id"):
            try:
                response = helpers.communicate(dispatcher_host,
                                               int(dispatcher_port),
                                               "status")
            except socket.error as e:
                raise Exception("Could not communicate with dispatcher server: %s" % e)
```

If it responds with "OK", then the repository observer opens the
`.commit_id` file, reads the latest commit ID and sends that ID to the
dispatcher, using a `dispatch:<commit ID>` request. It will then sleep
for five seconds and repeat the process. We'll also try again in five
seconds if anything went wrong along the way.

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

The repository observer will repeat this process forever, until you kill
the process via a \newline `KeyboardInterrupt` (Ctrl+c), or by sending it a kill
signal.

### The Dispatcher (`dispatcher.py`)

The dispatcher is a separate service used to delegate testing tasks. It
listens on a port for requests from test runners and from the repository
observer. It allows test runners to register themselves, and when given
a commit ID from the repository observer, it will dispatch a test runner
against the new commit. It also gracefully handles any problems with the
test runners and will redistribute the commit ID to a new test runner if
anything goes wrong.

When `dispatch.py` is executed, the `serve` function is called. First it
parses the arguments that allow you to specify the dispatcher's host and
port:

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

This starts the dispatcher server, and two other threads. One thread
runs the `runner_checker` function, and other runs the `redistribute`
function.

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

The `runner_checker` function periodically pings each registered test
runner to make sure they are still responsive. If they become
unresponsive, then that runner will be removed from the pool and its
commit ID will be dispatched to the next available runner. The function
will log the commit ID in the `pending_commits` variable.

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

The `redistribute` function is used to dispatch the commit IDs logged in
`pending_commits`. When `redistribute` runs, it checks if there are any
commit IDs in `pending_commits`. If so, it calls the `dispatch_tests`
function with the commit ID.

```python
    def redistribute(server):
        while not server.dead:
            for commit in server.pending_commits:
                print "running redistribute"
                print server.pending_commits
                dispatch_tests(server, commit)
                time.sleep(5)
```

The `dispatch_tests` function is used to find an available test runner
from the pool of registered runners. If one is available, it will send a
runtest message to it with the commit ID. If none are currently
available, it will wait two seconds and repeat this process. Once
dispatched, it logs which commit ID is being tested by which test runner
in the `dispatched_commits` variable. If the commit ID is in the
`pending_commits` variable, `dispatch_tests` will remove it since it has
already been successfully re-dispatched.

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

The dispatcher server uses the `SocketServer` module, which is a very simple
server that is part of the standard library. There are four basic server types
in the `SocketServer` module: `TCP`, `UDP`, `UnixStreamServer` and
`UnixDatagramServer`. We will be using a TCP-based socket server so we can
ensure continuous, ordered streams of data between servers, as UDP does not
ensure this.

The default `TCPServer` provided by `SocketServer` can only handle one
request at a time, so it cannot handle the case where the dispatcher is
talking to one connection, say from a test runner, and then a new
connection comes in, say from the repository observer. If this happens,
the repository observer would have to wait for the first connection to
complete and disconnect before it would be serviced. This is not ideal
for our case, since the dispatcher server must be able to directly and
swiftly communicate with all test runners and the repository observer.

In order for the dispatcher server to handle simultaneous connections,
it uses the \newline `ThreadingTCPServer` custom class, which adds threading
ability to the default `SocketServer`. This means that any time the
dispatcher receives a connection request, it spins off a new process
just for that connection. This allows the dispatcher to handle multiple
requests at the same time.

```python
class ThreadingTCPServer(SocketServer.ThreadingMixIn, SocketServer.TCPServer):
    runners = [] # Keeps track of test runner pool
    dead = False # Indicate to other threads that we are no longer running
    dispatched_commits = {} # Keeps track of commits we dispatched
    pending_commits = [] # Keeps track of commits we have yet to dispatch
```

The dispatcher server works by defining handlers for each request. This
is defined by the `DispatcherHandler` class, which inherits from
`SocketServer`'s `BaseRequestHandler`. This base class just needs us to
define the handle function, which will be invoked whenever a connection
is requested. The handle function defined in `DispatcherHandler` is our
custom handler, and it will be called on each connection. It looks at
the incoming connection request (`self.request` holds the request
information), and parses out what command is being requested of it.

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

It handles four commands: `status`, `register`, `dispatch`, and `results`.
`status` is used to check if the dispatcher server is up and running.

```python
        if command == "status":
            print "in status"
            self.request.sendall("OK")
```

In order for the dispatcher to do anything useful, it needs to have at
least one test runner registered. When register is called on a host:port
pair, it stores the runner's information in a list (the runners object
attached to the `ThreadingTCPServer` object) so it can communicate with
the runner later, when it needs to give it a commit ID to run tests
against.

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

`dispatch` is used by the repository observer to dispatch a test runner
against a commit. The format of this command is `dispatch:<commit ID>`.
The dispatcher parses out the commit ID from this message and sends it
to the test runner.

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

`results` is used by a test runner to report the results of a finished
test run. The format of this command is
`results:<commit ID>:<length of results data in bytes>:<results>`.
The `<commit ID>` is used to identify which commit ID the tests were run
against. The `<length of results data in bytes>` is used to figure out
how big a buffer is needed for the results data. Lastly, `<results>`
holds the actual result output.

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

### The Test Runner (`test_runner.py`)

The test runner is responsible for running tests against a given commit
ID and reporting the results. It communicates only with the dispatcher
server, which is responsible for giving it the commit IDs to run
against, and which will receive the test results.

When the `test_runner.py` file is invoked, it calls the `serve` function
which starts the test runner server, and also starts a thread to run the
`dispatcher_checker` function. Since this startup process is very similar
to the ones described in `repo_observer.py` and `dispatcher.py`, we omit
the description here.

The `dispatcher_checker` function pings the dispatcher server every five
seconds to make sure it is still up and running. This is important for
resource management. If the dispatcher goes down, then the test runner
will shut down since it won't be able to do any meaningful work if there
is no dispatcher to give it work or to report to.

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

The test runner is a `ThreadingTCPServer`, like the dispatcher server. It
requires threading because not only will the dispatcher be giving it a
commit ID to run, but the dispatcher will be pinging the runner
periodically to verify that it is still up while it is running tests.

```python
class ThreadingTCPServer(SocketServer.ThreadingMixIn, SocketServer.TCPServer):
    dispatcher_server = None # Holds the dispatcher server host/port information
    last_communication = None # Keeps track of last communication from dispatcher
    busy = False # Status flag
    dead = False # Status flag
```

The communication flow starts with the dispatcher requesting that the
runner accept a commit ID to run. If the test runner is ready to run the
job, it responds with an acknowledgement to the dispatcher server, which
then closes the connection. In order for the test runner server to both
run tests and accept more requests from the dispatcher, it starts the
requested test job on a new thread.

This means that when the dispatcher server makes a request (a ping, in
this case) and expects a response, it will be done on a separate thread,
while the test runner is busy running tests on its own thread. This
allows the test runner server to handle multiple tasks simultaneously.
Instead of this threaded design, it is possible to have the dispatcher
server hold onto a connection with each test runner, but this would
increase the dispatcher server's memory needs, and is vulnerable to
network problems, like accidentally dropped connections.

The test runner server responds to two messages from the dispatcher.  The first
is `ping`, which is used by the dispatcher server to verify that the runner is
still active.

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

The second is `runtest`, which accepts messages of the form `runtest:<commit
ID>`, and is used to kick off tests on the given commit. When runtest is
called, the test runner will check to see if it is already running a test, and
if so, it will return a `BUSY` response to the dispatcher. If it is available,
it will respond to the server with an `OK` message, set its status as busy and
run its `run_tests` function.

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

This function calls the shell script `test_runner_script.sh`, which
updates the repository to the given commit ID. Once the script returns,
if it was successful at updating the repository we run the tests using
unittest and gather the results in a file. When the tests have finished
running, the test runner reads in the results file and sends it in a
results message to the dispatcher.

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

In order to run `test_runner.py`, you must point it to a clone of the
repository to run tests against. In this case, you can use the previously
created `/path/to/test_repo test_repo_clone_runner` clone as the argument. By
default, `test_runner.py` will start its own server on localhost using a port
in the range 8900-9000, and will try to connect to the dispatcher server at
`localhost:8888`. You may pass it optional arguments to change these values.
The `--host` and `--port` arguments are used to designate a specific address to
run the test runner server on, and the `--dispatcher-server` argument specifies
the address of the dispatcher.

### Control Flow Diagram

\aosafigref{500l.ci.controlflow} is an overview diagram of this system. This
diagram assumes that all three files \newline (`repo_observer.py`,
`dispatcher.py` and `test_runner.py`) are already running, and describes the
actions each process takes when a new commit is made.

\aosafigure[360pt]{ci-images/diagram.png}{Control Flow}{500l.ci.controlflow}

### Running the Code

We can run this simple CI system locally, using three different terminal shells
for each process. We start the dispatcher first, running on port 8888:

```bash
$ python dispatcher.py
```

In a new shell, we start the test runner (so it can register itself with
the dispatcher):

```bash
$ python test_runner.py <path/to/test_repo_clone_runner>
```

The test runner will assign itself its own port, in the range 8900-9000.
You may run as many test runners as you like.

Lastly, in another new shell, let's start the repo observer:

```bash
$ python repo_observer.py --dispatcher-server=localhost:8888 <path/to/repo_clone_obs>
```

Now that everything is set up, let's trigger some tests! To do that,
we'll need to make a new commit. Go to your master repository and make
an arbitrary change:

```bash
$ cd /path/to/test_repo
$ touch new_file
$ git add new_file
$ git commit -m"new file" new_file
```

Then `repo_observer.py` will realize that there's a new commit and notify
the dispatcher. You can see the output in their respective shells, so
you can monitor them. Once the dispatcher receives the test results, it
stores them in a `test_results/` folder in this code base, using the
commit ID as the filename.

## Error Handling

This CI system includes some simple error handling.

If you kill the `test_runner.py` process, `dispatcher.py` will figure out
that the runner is no longer available and will remove it from the pool.

You can also kill the test runner, to simulate a machine crash or
network failure. If you do so, the dispatcher will realize the runner
went down and will give another test runner the job if one is available
in the pool, or will wait for a new test runner to register itself in
the pool.

If you kill the dispatcher, the repository observer will figure out it
went down and will throw an exception. The test runners will also
notice, and shut down.

## Conclusion

By separating concerns into their own processes, we were able to build
the fundamentals of a distributed continuous integration system. With
processes communicating with each other via socket requests, we are able
to distribute the system across multiple machines, helping to make our
system more reliable and scalable.

Since the CI system is quite simple now, you can extend it yourself to
be far more functional. Here are a few suggestions for improvements:

### Per-Commit Test Runs

The current system will periodically check to see if new commits are run
and will run the most recent commit. This should be improved to test
each commit. To do this, you can modify the periodic checker to dispatch
test runs for each commit in the log between the last-tested and the latest commit.

### Smarter Test Runners

If the test runner detects that the dispatcher is unresponsive, it stops
running. This happens even when the test runner is in the middle of
running tests! It would be better if the test runner waited for a period
of time (or indefinitely, if you do not care about resource management)
for the dispatcher to come back online. In this case, if the dispatcher
goes down while the test runner is actively running a test, instead of
shutting down it will complete the test and wait for the dispatcher to
come back online, and will report the results to it. This will ensure
that we don't waste any effort the test runner makes, and that we will
only run tests once per commit.

### Real Reporting

In a real CI system, you would have the test results report to a
reporter service which would gather the results, post them somewhere for
people to review, and notify a list of interested parties when a failure
or other notable event occurs. You can extend our simple CI system by
creating a new process to get the reported results, instead of the
dispatcher gathering the results. This new process could be a web server
(or can connect to a web server) which could post the results online,
and may use a mail server to alert subscribers to any test failures.

### Test Runner Manager

Right now, you have to manually launch the `test_runner.py` file to start
a test runner. Instead, you could create a test runner manager process
which would assess the current load of test requests from the dispatcher
and scale the number of active test runners accordingly. This process
will receive the runtest messages and will start a test runner process
for each request, and will kill unused processes when the load
decreases.

Using these suggestions, you can make this simple CI system more robust
and fault-tolerant, and you can integrate it with other systems, like a
web-based test reporter.

If you wish to see the level of flexibility continuous integration
systems can achieve, I recommend looking into
[Jenkins](<http://jenkins-ci.org/>), a very robust, open-source CI
system written in Java. It provides you with a basic CI system which you
can extend using plugins. You may also access its source code [through
GitHub](<https://github.com/jenkinsci/jenkins/>). Another recommended
project is [Travis CI](<https://travis-ci.org/>), which is written in
Ruby and whose source code is also available [through
GitHub](<https://github.com/travis-ci/travis-ci>).

This has been an exercise in understanding how CI systems work, and how
to build one yourself. You should now have a more solid understanding of
what is needed to make a reliable distributed system, and you can now
use this knowledge to develop more complex solutions.
