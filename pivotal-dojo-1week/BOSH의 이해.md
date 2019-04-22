## BOSH의 내부구조

### 1.  BOSH 소개

#### 1-1 BOSH는 다양한 Public/Private Cloud 또는 local 환경에서의 수백가지의 VM 배포 시스템의 Release 엔지니어링, Deployment Deploy, Life Cycle 관리 및 모니터링을위한 Opensource 도구
![bosh-소개][bosh-image-0]

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
	- 플랫폼, Application의 스케일아웃/스케일업 지원
- Security
	- 무중단 패치
	- 네트워크 보안
	- 플랫폼 데이터 보안

