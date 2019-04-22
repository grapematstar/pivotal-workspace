## BOSH의 내부구조

### 1.  BOSH 소개

#### 1-1 BOSH는 다양한 Public/Private Cloud 또는 local 환경에서의 수백가지의 VM 배포 시스템의 Release 엔지니어링, Deployment Deploy, Life Cycle 관리 및 모니터링을위한 Opensource 도구

![bosh-소개][bosh-image-0]

#bosh docs 참고 주소
https://ultimateguidetobosh.com/
http://bosh-docs.cfapps.io/
https://bosh.io/docs/dns/

- BOSH를 통해 관리가 가능한 Databases

  ![bosh-managed-databases][bosh-image-1]
- BOSH를 통해 관리가 가능한 Messaging Clusters
 
  ![bosh-managed-clusters][bosh-image-2]
  
- BOSH를 통해 관리가 가능한 CI/CD
   ![bosh-managed-ci/cd][bosh-image-3]

#### 1-2 BOSH를 사용해야하는 이유
- Speed
	 - 신속한 VM, Service, Application 배포
	 - 최신 버전의 플랫폼 Upgrade
	 - Upgrade의 소요 시간
- Stability & Scalability(안정성과 확장성)
	- 플랫폼, Application 고가용성
	- 플랫폼, Application의 scale up/scale out 지원
- Security
	- 무중단 패치
	- 네트워크 보안
	- 플랫폼 데이터 보안

#### 1-3 BOSH의 구성 요소 (VM설치 구성)

- Stemcell
	-  스템셀은 IaaS의 특정 패키징으로 포장 된 버전 관리 OS 이미지로 기본 OS 이미지의 구성을 가지고 있다. 스템셀에는 BOSH Agent를 포함하고 있으며 Agent를 통해 해당 bosh의 Blobstore로부터 Job을 배포 한다.
	
	![bosh-stemcell][bosh-image-4]
	
- Release
	- 릴리즈는 해당 소프트웨어나 패키지의 구성 정보와 시작과 중지 스크립트 등이 존재하며 소프트웨어를 빌드하고 실행 시키는데 필요한 Package와 Job의 모음이다.
	-   Packages에는 Package와 package 간 의존 관계 및 설치 스크립트 등이 존재 한다.
	-   Job에는 VM을 배포 시 적용 되는 설정 파일과 프로세스 시작/종료에 관한 스크립트 파일 등이 존재 한다.
	
	![bosh-stemcell][bosh-image-5]
	
- Deployment Manifest
	- 배포는 IaaS 환경 별 스템셀 이미지를 기반으로 다 수의 VM 생성 하는 것이며 특정 릴리즈의 Package, Job과 배포 Manifest 파일에 명시 되어 있는 Resource(VM의 Disk Size, 소프트웨어 정보, VM의 네트워크 정보 등)를 BOSH의 Director에 업로드를하고 Director를 통해 리소스를 할당 하여 IaaS에 VM 및 을 생성 한다.

	- BOSH 설치 Manifest 파일은 배포에 필요한 컴포넌트 및 속성 정보를 YAML 파일 형식으로 구성이 되어 있으며 크게 Release 정보를 설정 하는 Block, Network 정보를 설정하는 Block, Resource Pools 스템셀과 CPI 정보를 설정하는 Block, Disk Pool 사이즈를 설정하는 Block, Job 정보를 설정하는 Bloc, Property를설정하는 Block 등이 존재한다.

		※ 다음은 BOSH 설치 Manifest 파일의 Block 설명 이다.

		1.  Deployment Identification: 배포 명과 Director의 고유 UUID를 설정하는 Block
		2.  Releases Block: 배포 시 사용할 Release 명과 버전을 설정하는 Block
		3.  Networks Block: 배포 시 할당 할 IaaS의 네트워크 정보를 설정하는 Block
		4.  Resource Pools Block: BOSH가 설치 하고 관리 하는 VM의 속성 Block
		5.  Disk Pools Block: BOSH가 작성하고 관리하는 Disk Pool의 등록 정보를 설정하는 Block
		6.  Compilation Block: VM의 컴파일 속성을 설정하는 Block
		7.  Update Block: 배포 중에 BOSH가 작업 인스턴스를 업데이트하는 방법을 정의하는 Block
		8.  Jobs Block: Job에 대한 구성 및 자원을 설정하는 Block
		9.  Properties Block: 전역 속성과 일반화 된 구성 정보(config)를 설정 하는 Block

 #### 1-4 BOSH의 구성 요소 (Architecture)
![bosh-stemcell][bosh-image-6]

1.  Command Line Interface(CLI): Director와 상호작용을 위한 Command Line Interface
2.  Director: Director가 VM을 생성 또는 수정할 때 설정 정보를 레지스트리에 저장한다. 저장된 레지스트리 정보는 VM의 bootstrapping stage에서 이용된다.
3.  Database: Director가 사용하는 Postgres 데이터베이스로 Deployment시에 필요로하는 Stemcell / Release / Deployment의 메타 정보들을 저장한다.
4.  Message Bus(Nats): Message Bus는 Director와 Agent간 통신하기 위한 publish-subscribe 방식의 Message System으로 VM 모니터링과 특정 명령을 수행하기 위해 사용된다.
5.  Registry: VM 생성을 위한 설정 정보 저장
6.  Health Monitor: Health Monitor는 BOSH Agent로부터 클라우드의 상태정보들을 수집한다. 클라우드로부터 특정 Alert이 발생하면 Resurrector를 하거나 Notification Plug-in을 통해 Alert Message를 전송할 수도 있다.
7.  Agent:  BOSH를 통해 클라우드에 배포되는 모든 VM에 설치되고 Director로부터 특정 명령을 받고 수행하는 역할을 하게된다. Agent는 Director로부터 수신 받은 Job Specification(설치할 패키지 및 구성 방법)정보로부터 해당 VM에 Director의 지시대로 지정된 패키지를 설치하고, 필요한 구성정보를 설정하게 된다.

#### 1-5 BOSH의 구성 요소 (설치 링크)
- 스템셀 다운로드 주소
	[https://bosh.io/stemcells/](https://bosh.io/stemcells/)

- 릴리즈 다운로드 주소
	[https://bosh.io/releases/](https://bosh.io/releases/)

- Bosh Manifest 다운로드 주소
   [https://github.com/cloudfoundry/bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)

