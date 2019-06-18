
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
	- 적은 Network의 노출

### 1.1 System Access 노출 제한

- Pivotal Cloud Foundry 기본적인 구성에서는 거의 모든 Component 들이 Private Network(Vlan)으로 배포 되며 DMZ 영역에 Go Router의 Endpoint에 도달하는 Load Balancer,  Platform 관리를 위한 Jump box, OutBound를 위한 NAT VM 3가지가 필요하다, External과의 연결을 적은 수로 하여 보안 취약성을 최소화 한다.

[System Access][https://docs.pivotal.io/pivotalcf/2-5/concepts/security.html](https://docs.pivotal.io/pivotalcf/2-5/concepts/security.html)


