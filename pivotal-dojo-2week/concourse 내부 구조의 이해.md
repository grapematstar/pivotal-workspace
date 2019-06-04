## 1. Concourse 내부 구조의 이해

### 1.1. Concourse의 특징

![concouse][concouse-image-0]

-  Go로 작성한 CI/CD 자동화 시스템으로 단순한 Build 부터 복잡한 배포까지 모든 종류의 Pipeline을 구성하여 관리를 할 수 있게하는 Tool
-  모든 Pipeline이 Container 내에서 실행 되고 자체 이미지를 사용하여 코드/패키지에 대한 종속성을 완벽하게 제어
- Resources라는 Pipeline의 섹션을 사용하여 별도의 플러그인이 필요 없이 git/s3/docker image/smtp 같은 다양한 추상화 한 Resource를 사용할 수 있다.
- 작은 단위로 설정하여 복잡성을 줄이고 빠르게 릴리즈 배포가 가능하게 한다.

### 1.2. Concourse vs CI/CD Tools

- Concourse의 다른 기타 CI/CD Tool의 차이점은 시각화/logging으로 Debug가 쉽고
- Pipeline의 계층구조가 복잡하지 않아 구성하기 쉬움
- 별도의 플러그인 사용 없이 구성하여 항상 일정한 결과 값을 가져올수 있음
- Resource 대한 버전 관리가 가능하여 언제든 결과 값을 Rollback 가능

### 1.3. Concourse 구성 요소

- Task: isolated 환경(Docker Image)에서 스크립트 실행 종속 자원이있는 환경 가장 작은 실행 단위 Property
- Job: 수행할 작업에 대한 동작 방법을 결정하여 Pipeline을 시각화 하는 Property 
- Resource: Pipeline의 모든 input/output을 설정하며 특정 위치의 resource에 대한 버전 관리를 가능하게 함

### 1.4. Concourse 아키텍처

![concouse_아키텍쳐][concouse-image-1]

- ATC: Concourse의 Web  UI와 fly 명령어/REST API 요청을 처리하는 Component
- TSA:  동작하는 Container의 SSH(2222) PORT 수신을 대기하며 internal/exteranl worker를 등록하여 ATC에 사용 할 수 있는 worker 정보를 전달, 지속적으로 ping을 체크하여 불안전한 worker 상태를 확인하는 Component
- Beacon: TSA와 join 하여 extenal worker의 2222 port와 handshake 하여 ATC와 통신이 가능하게 하며 Pipeline 생성 시 Worker의 준비 완료 상태를 전달하는 Component
- Garden: 실제 Container 환경이 실행 되는 Component, 7777 PORT로 컨테이너 API 요청을 수신한다.
- Baggageclaim: ATC가 Container에 Mount 된 Disk Resource 또는 Volume를 관리하기위한 7788 PORT API를 제공하는 Component
- Postgres: Pipeline의 정보와 Build 로그를 저장하고 있는 Component

### 1.5. fly CLI
- Concourse의 모든 관리 Command Line으로 Pipeline의 Life Cycle를 관리한다.

```
$ fly help
Usage:
  fly [OPTIONS] <command>

Application Options:
  -t, --target=              Concourse target name
  -v, --version              Print the version of Fly and exit
      --verbose              Print API requests and responses
      --print-table-headers  Print table headers even for redirected output

Help Options:
  -h, --help                 Show this help message

Available commands:
  abort-build          Abort a build (aliases: ab)
  builds               List builds data (aliases: bs)
  check-resource       Check a resource (aliases: cr)
  check-resource-type  Check a resource-type (aliases: crt)
  checklist            Print a Checkfile of the given pipeline (aliases: cl)
  clear-task-cache     Clears cache from a task container (aliases: ctc)
  containers           Print the active containers (aliases: cs)
  curl                 curl the api (aliases: c)
  delete-target        Delete target (aliases: dtg)
  destroy-pipeline     Destroy a pipeline (aliases: dp)
  destroy-team         Destroy a team and delete all of its data (aliases: dt)
  edit-target          Edit a target (aliases: etg)
  execute              Execute a one-off build using local bits (aliases: e)
  expose-pipeline      Make a pipeline publicly viewable (aliases: ep)
  format-pipeline      Format a pipeline config (aliases: fp)
  get-pipeline         Get a pipeline's current configuration (aliases: gp)
  help                 Print this help message
  hide-pipeline        Hide a pipeline from the public (aliases: hp)
  hijack               Execute a command in a container (aliases: intercept, i)
  jobs                 List the jobs in the pipelines (aliases: js)
  land-worker          Land a worker (aliases: lw)
  login                Authenticate with the target (aliases: l)
  logout               Release authentication with the target (aliases: o)
  order-pipelines      Orders pipelines (aliases: op)
  pause-job            Pause a job (aliases: pj)
  pause-pipeline       Pause a pipeline (aliases: pp)
  pipelines            List the configured pipelines (aliases: ps)
  prune-worker         Prune a stalled, landing, landed, or retiring worker (aliases: pw)
  rename-pipeline      Rename a pipeline (aliases: rp)
  rename-team          Rename a team (aliases: rt)
  resource-versions    List the versions of a resource (aliases: rvs)
  resources            List the resources in the pipeline (aliases: rs)
  set-pipeline         Create or update a pipeline's configuration (aliases: sp)
  set-team             Create or modify a team to have the given credentials (aliases: st)
  status               Login status
  sync                 Download and replace the current fly from the target (aliases: s)
  targets              List saved targets (aliases: ts)
  teams                List the configured teams (aliases: t)
  trigger-job          Start a job in a pipeline (aliases: tj)
  unpause-job          Unpause a job (aliases: uj)
  unpause-pipeline     Un-pause a pipeline (aliases: up)
  userinfo             User information
  validate-pipeline    Validate a pipeline config (aliases: vp)
  volumes              List the active volumes (aliases: vs)
  watch                Stream a build's output (aliases: w)
  workers              List the registered workers (aliases: ws)
```

### 1.6. Concourse 구성 요소 상세

#### 1.6.1. Auth & Teams
```
Concoures는 내부적으로 많은 User 생성이 가능하다.
Pipeline, Bulid, UserInfo 등 모든 데이터들은 Team이 소유하고 있으며 namespace 형식으로 관리 한다.
Team에 연결 할 User의 정보는 Git/Cloud Foundry/LDAP/OAuth2/Bagic 등이 있으며 Team을 기준으로 권한을 부여 할 수 있다.
권한의 종류는 owner/member/pipeline-operator/viewer가 존재한다.

$ fly --target ${FLY_TARGET_NAME} login --team-name main \     --concourse-url https://ci.example.com
$ fly -t ${FLY_TARGET_NAME} teams
name
main
```
#### 1.6.2. Pipelines
```
Concourse Pipeline은 Resource와 Job의 값들이 합쳐진 Yaml 파일 형식의 결과이며 지속적으로 Resource의 버전을 감지한다.
Pipeline은 아래와 같은 구조를 가지고 있다.
- jobs: [job] -> Pipeline이 주기적으로 확인 할 Job 집합
- resources: [resource] -> Pileline이 주기적으로 확인 할 Resource 집합
- resource_types: [resource_type] -> Pipleline이 사용할 Resource 유형의 집합
- groups: [group] -> Web UI에서의 작업을 정리
```

#### 1.6.3. Resources
```
Resource는 Concourse Pipeline 내의 모든 외부 Input과 Output을 설정 할 수 있다.
Resource는 아래와 같은 구조를 가지고 있다.
- name: string -> job의 명칭으로 Pipeline에서 실행하는 실제 Task가 참조한다.
- type: string -> Pipeline에 정의한 resource type의 명칭
- icon: string -> Concourse Pipeline Web UI에서 resource 옆에 붙을 icon 설정
- source: object -> git, s3, docker repo 등 사용할 resource에 대한 uri, username 등을 정의
- version: object -> job이 사용 할 resource의 버전을 고정적으로 정의
- check_every: string -> 새 resource verion을 체크 할 time 정의 default는 1m
- tags: [string] -> git, s3, docker repo 등 사용할 resource에 대한 version/tag 정의
- public: boolean -> true일 경우 권한이 없는 사용자가 해당 Resource의 version에 대한 메타데이터를 확인 할 수 있다. default는 false
- webhook_token: string -> web hook과 연동 정보 정의
```
#### 1.6.4. Resource Type
```
Docker Image를 사용하여 커스터마이징이 이루어진 resource를 사용하는 기능
example) smtp, slack, k8s, terraform 등
Resource Type은 아래와 같은 구조를 가지고 있다.

- name: string -> resource type의 명칭으로 pipeline 내의 resource와 task의 image_resources에서 참조 된다.
- type: string -> Container Image를 제공하는 데 사용되는 자원의 유형 정의
- source: object -> Docker Image Repo의 정보를 정의
- params: object -> 해당 Resource를 가져올때 param이 넘어 간다.
- check_every: string -> 새 resource verion을 체크 할 time 정의 default는 1m
- tags: [string] -> Docker Image의 Tag 지정
- unique_version_history: boolean -> 글로벌 버전을 지정할것인지에 대한 true/false 정의
```

#### 1.6.5. Jobs
```
Plan을 통해 Concourse Pipeline의 동작 방식을 결정하며 Web UI의 시각화를 trigger, passed 등을 통해 Task의 실행 방법과 순서를 정의한다.
Jobs는 아래와 같은 구조를 가지고 있다.
- name: string -> job의 명칭
- old_name: string -> 이전 job의 명칭으로 정의하게되면 새 job이 이전 job을 상속 한다.
- plan: [step] -> Pipeline의 실행 순서를 정의 한다.
- serial: boolean -> true로 변경하게 되면 build가 한개씩 실행 된다. default는 false
- build_logs_to_retain: 빌드 로그의 보관 주기 정책 정의
	- days: number -> 빌드 로그의 보관 주기 정의
	- builds: number -> 저장할 빌드 로그의 빌드 수 정의
- serial_groups: [string] -> 지정한 태그에 대한 빌드가 직렬화한다.
- max_in_flight: integer -> 한번에 실행 할 Build의 최대 개수 정의
- public: boolean -> true일 경우 인증되지 않은 사용자가 빌드 로그를 볼 수 있다. defualt는 false
- disable_manual_trigger: boolean -> 수동 trigger 작업을 활성/비활성화 한다.
- on_success: step -> Job이 성공하면 실행하는 단계 정의
- on_failure: step -> Job이 실패하면 실행하는 단계 정의
- on_abort: step -> Job이 중단 될때 실행하는 단계 정의
- ensure: step -> Job이 종료되고 무조건 실행하는 단계 정의
```
#### 1.6.6. Tasks
```
Concourse Pipeline에서 구성 가능한 최소 단위 실행 단위이며. Job에서의 Input에서 Output까지 Success/fail을 결정한다.
Tasks는 아래와 같은 구조를 가지고 있다.
- platform: string -> Job을 실행 할 Platform 명 ex) linux/windows/darwin
- image_resource: resource -> resource image의 경로 지정
	- type: string  -> resource image의 type ex) docker-image 정의
	- source: object -> resource image의 Repo의 정보를 정의
	- params: object -> 해당 Resource를 가져올때 해당 param이 넘어 간다.
	- version: object -> 사용 할 resource의 버전 정보
- rootfs_uri: string -> Garden이 사용할 rootfs_uri 정보 정의
- inputs: [input]  
	- name: string -> input의 명칭 
	- path: string -> input이 사용할 경로, 경로를 지정하지 않으면 input 명칭 값으로 명시 된다. 경로는 job의 디렉토리와 연관이 된다.
	- optional: boolean -> true일 경우 task에서 해당 input을 사용하지 않아도 된다.
- outputs: [output]
	- name: string -> output의 명칭, 해당 경로에 있는 내용은 해당 명칭으로 jobs의 나머지 plan에서 사용 가능하다.
	- path: string -> output 디렉토리, 디렉토리를 지정하지 않으면 output 명칭 값으로 명시 된다.
- caches: [cache] -> Task 실행 중 공유 된 캐시 디렉토리 지정
	- path: string -> 
- run: run-config 
	- path: string -> Container 환경에서 실행 할 명령 파일이 있는 디렉토리
	- args: [string] -> Container 환경에서 실행 할 명령어 줄
	- dir: string -> 명령어를 실행 할 때 디렉토리로 설정 할 초기 작업 디렉토리
	- user: string -> Container 환경의 사용자를 지정 defalut: root
- params: {string: string} -> 환경 변수를 통해 job에 노출 할 key/value
```
[concouse-image-0]:./images/concourse-image-0.png
[concouse-image-1]:./images/concourse-image-1.png
