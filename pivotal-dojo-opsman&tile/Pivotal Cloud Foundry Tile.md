##  Pivotal Cloud Foundry Tile

- PAS Tile은 Pivotal Cloud Foundry를 설치하는 Tile으로 Pivotal Cloud Foundry 설치에 대한 전반적인 Domain, Network, Component, Errand 등의 Config 정보를 설정 할 수있다.
- 전제 조건
	- vShpere 환경이 구축되어 있어야 한다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- Ops Manager에 Pivotal Cloud Foundry Tile이 존재 해야 한다.
- Modify Last Version 2.5
 
### 1. Pivotal Cloud Foundry Tile Config 구성

#### 1.1. AZ and Network Assignments Config
- Pivotal Cloud Foundry를 배포 할 때 Singleton으로 구성 할 VM(BlobStore)과 여러 다중으로 구성하는 AZ를 선택 한다.
- Bosh Director Tile에서 설정한 Network를 선택 한다.
- AZ 또는 Network를 증설 할 경우 Bosh Director Tile의 정보를 수정하고 Apply Change 해야한다.

#### 1.2. Configure Domains
- System/Application Domain 정보를 입력 한다.
	- ex) sys.example.com, apps.sys.example.com
- Pivotal은 System Domain과 Application Domain을 Single Wildcard Certificate로 사용 할 수 있도록 권고 한다.

[Configure Domains][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html)

#### 1.3. Configure Networking
- Router IPs, HAProxy IPs: vSphere 환경에서는 HA Proxy VM사용 유무에 따라 Router IP의 설정 방법이 달라진다.
	- HAProxy를 사용하지 않을 경우
		- HA를 구성하기 위해 2개 이상의 Router IP를 입력 한다.
		- Pivotal Cloud Foundry 사용 Doamin에 대한 요청을 Router IP에 Load Balancer 구성 한다.
	- HAProxy를 사용할 경우
		- HA를 구성하기 위해 2개 이상의 Ha Proxy IP를 입력 한다.
		- 




