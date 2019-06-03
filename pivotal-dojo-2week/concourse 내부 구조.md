## 1. Concourse 내부 구조의 이해

### 1.1 Concourse의 특징

[Concourse_Image]

-  Go로 작성한 CI/CD 자동화 시스템으로 단순한 Build 부터 복잡한 배포까지 모든 종류의 Pipeline을 구성하여 관리를 할 수 있게하는 Tool
-  모든 Pipeline이 Container 내에서 실행 되고 자체 이미지를 사용하여 코드/패키지에 대한 종속성을 완벽하게 제어
- Resources라는 Pipeline의 섹션을 사용하여 별도의 플러그인이 필요 없이 git/s3/docker image/smtp 같은 다양한 추상화 한 Resource를 사용할 수 있다.
- 작은 단위로 설정하여 복잡성을 줄이고 빠르게 릴리즈 배포가 가능하게 한다.

### 1.2 Concourse vs CI/CD Tools

- Concourse의 다른 기타 CI/CD Tool의 차이점은 시각화/logging으로 Debug가 쉽고
- Pipeline의 계층구조가 복잡하지 않아 구성하기 쉬움
- 별도의 플러그인 사용 없이 구성하여 항상 일정한 결과 값을 가져올수 있음
- Resource 대한 버전 관리가 가능하여 언제든 결과 값을 Rollback 가능

### 1.3 Concourse 구성 요소

- Task: isolated 환경(Docker Image)에서 스크립트 실행 종속 자원이있는 환경 가장 작은 실행 단위 Property
- Job: 수행할 작업에 대한 동작 방법을 결정하여 Pipeline을 시각화 하는 Property 
- Resource: Pipeline의 모든 input/output을 설정하며 특정 위치의 resource에 대한 버전 관리를 가능하게 함

### 1.4 Concourse 아키텍처

[아키텍처_Image]

- ATC: Concourse의 Web  UI와 fly 명령어/REST API 요청을 처리하는 Component
- TSA:  동작하는 Container의 SSH(2222) PORT 수신을 대기하며 internal/exteranl worker를 등록하여 ATC에 사용 할 수 있는 worker 정보를 전달, 지속적으로 ping을 체크하여 불안전한 worker 상태를 확인하는 Component
- Beacon: TSA와 join 하여 extenal worker의 2222 port와 handshake 하여 ATC와 통신이 가능하게 하며 Pipeline 생성 시 Worker의 준비 완료 상태를 전달하는 Component
- Garden: 실제 Container 환경이 실행 되는 Component, 7777 PORT로 컨테이너 API 요청을 수신한다.
- Baggageclaim: ATC가 Container에 Mount 된 Disk Resource 또는 Volume를 관리하기위한 7788 PORT API를 제공하는 Component

### 1.5 FIY CLI

- Concourse의 모든 관리 Command Line으로 Pipeline의 Life Cycle를 관리한다.
