

#  Pivotal Cloud Foundry Operating Security

- Pivotal Cloud Foundry에서는 Platform, Application 단의 여러 보안 구성이 가능 하다.

- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- Pivotal Cloud Foundry가 구축 되어 있어야 한다.
- Modify Last Version 2.5

## 1. Pivotal Cloud Foundry Security

- Pivotal Cloud Foundry Security에서 제공하는 Security는 아래와 같다.
	- Application과 Container Data의 분리
	- 내부 Component의 연결 암호화
	- 사용자 역할 권한 부여를 통해 Space Access 관리
	- Resource Starvation 상황의 서비스 공격 방지
	- 적은 수의 Network의 노출

### 1.1. System Access 노출 제한

- Pivotal Cloud Foundry 기본적인 구성에서는 거의 모든 Component 들이 Private Network(Vlan)으로 배포 되며 DMZ 영역에 Go Router의 Endpoint에 도달하는 Load Balancer,  Platform 관리를 위한 Jump box, OutBound를 위한 NAT VM 3가지가 필요하다, External과의 연결을 적은 수로 하여 보안 취약성을 최소화 한다.

[System Access][https://docs.pivotal.io/pivotalcf/2-5/concepts/security.html](https://docs.pivotal.io/pivotalcf/2-5/concepts/security.html)

### 1.2. Isolation Segments

- Isolation Segments를 통하여 하나의 PAS에서 Application을 격리 된 공간에 배포하도록 구성해 네트워크의 복잡성과 중복 관리 요소를 줄 일 수 있다.
- PAS 내의 ORG/SPACE에서 하나의 Isolation Segments을 사용하도록 구성하여 격리된 환경의 다른 ORG/SPACE에 영향을 받지 않으며 Service를 구축 할 수 있다.
- Cloud Controller는 placement_tag을 통하여 Isolation Segments를 구분하여 해당 Cell에 접근 가능 하도록 한다.

[Isolation Segments][https://docs.pivotal.io/pivotalcf/2-5/concepts/security.html](https://docs.pivotal.io/pivotalcf/2-5/concepts/security.html)

### 1.3. Pivotal Cloud Foundry 인증 & 권한 UAA

- UAA는 Pivotal Cloud Foundry 및 다양한 Deployment에 대한 사용자 ID Manager Service
- UAA는 OAuth2 인증 서버 역할을하며 Platform또는 Application 대한 Resource/API End Point를 요청하는 Access Token을 발급 한다.

### 1.4. ORG/SPACE에 대한 Access 권한 관리

- Pivotal Cloud Foundry 사용자가 Application을 관리하기 위해서는 cf CLI를 통하여 API Endpoint에 Target을 설정하고 UAA 사용자 Login을 해야하며 사용자는  Pivotal Cloud Foundry의 Org/Space에 대한 Access 권한이 반드시 존재해야 한다.
- 주요 Access 권한에 대한 Permission scope는 아래와 같다.
	- cloud_controller.read
	- cloud_controller.write
	- cloud_controller.admin
	- cloud_controller.global_auditor

### 1.5. Service Broker에 대한 Access 권한 관리

- Pivotal Cloud Foundry 내부 Component Cloud Controller를 통해 Market Place의 Service Broker를 등록 할 경우 관련한 모든 API에 대해 인증 절차를 하게 되고 권한이 없는 사용자 또는 사용자를 입력하지 않을 경우 해당 요청을 Reject 한다.

### 1.6. Software 취약점 관리
- Pivotal Cloud Foundry는 Release와 Stemcell을 사용하여 Software 취약점 관리한다.
- Release를 통하여 Code 부분의 취약점을 지속적으로 보완하며 Stemcell을 통하여 OS 레벨의 보안 패치가 이루어진다.

### 1.7. Application Security
- 


