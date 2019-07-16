#  Pivotal Cloud Foundry Application Design for the Cloud

- Cloud Foundry 상의 Cloud Application 설계에 대한 가이드 라인, 고려 사항에 대해 작성 하였다.
- Modify Last Version 2.6

## 1. Avoid Writing to the Local File System
- Pivotal Cloud Foundry에서 실행하는 Application은 아래의 이유로 인하여 Local System에 직접 File을 Write 하지 않아야 한다.

### 1.1. Local file system storage 수명
- Pivotal Cloud Foundry에서 실행하는 Application이 Crash, Stop 상태로 변하게 되면 Application이 Start되고 변경한 Local Disk를 Platform에서 회수하고 Crash, Stop에서 Restage/Restart시 새로운 Local Disk로 Application이 실행 된다.

### 1.2. 동일한 Application의 복수 Instance 사이의 Local File System 공유 하지 않음
- Pivotal Cloud Foundry에서 실행하는 Application은 복수개의 Instance를 가지고 있고 해당 Instance는 각 각의 격리된 Container 환경에서 작동하게 된다. 하나의 Application Instance에서 작성한 File을 다른 Instance에서 관리 및 공유 할 수 없으며 일관적인 File System 상태여야한다.
- Application Instance 사이의 공유 방법으로는 Database, Blobstore, S3, NFS, Redis 같은 외부 Service를 통하여 공유한다.

## 2. Cookies Accessible across Apps
- Pivotal Cloud Foundry의 Application은 Shard Domain을 통하여 서로 다른 Application의 Cookie를 공유 할 수 있다.
- example.com이라는 Shared Domain이 존재하고, test1-app.shared-domain.example.com App에 저장한Cookie를 test2-app.shared-domain.example.com App에서 접근 할 수 있다.

## 3. Port Considerations
- Pivotal Cloud Foundry의 Application은 Domain 기븐의 HTTP/HTTPS(80/443) Port로의 요청만을 허용한다.
- 특정 Log Stream 같은 기능에서 Web Socket TCP Port를 사용 할 경우 플랫폼 관리자는 별도의 LB 구성 및 Port를 443이 아닌 구성으로 변경해야 한다.

## 4. Cf Push 중 필요하지 않은 File 제거
- Pivotal Cloud Foundry는 $ cf push 명령어를 통해 Application을 배포 할 시 Project의 Directory에 있는 모든 파일들을 Application Instance에 Upload 한다, 예를들어 .git, .manifest.yml, .svn 등이 있고 해당 파일들을 .cfignore 파일에 기술하여 $ cf push 시 App Directory에 해당 파일들이 Upload 되지 않게 한다.

## 5. Instances to Increase Availability
- Pivotal Cloud Foundry에서 실행하는 Application을 Singleton 구성을 하게 될 경우 Application Upgrade 시 Stop -> Restart 또는 Stop -> Restage 단계를 거치게 되는데 해당 단계 동안 Diego Cell VM에서는 Application을 종료하고, 종료가 완료되면 Random의 Diego Cell VM에서 재기동하게 된다. 이때 Singleton일 경우에는 해당 Application을 일시적으로 사용 할 수 없게 된다.
- Pivotal Cloud Foundry를 HA 구성 하였을 때 IaaS 환경 하나의 Zone이 다운로드 되어 Application이 존재하는 Diego Cell에서 에러가 발생 했 을 경우 해당 1~2 분 동안 Application를 사용 할 수 없을 수 있다. 
