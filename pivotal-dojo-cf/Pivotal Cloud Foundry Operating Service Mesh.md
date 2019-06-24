
#  Pivotal Cloud Foundry Operating Service Mesh(Beta)

- Pivotal Cloud Foundry v2.5 이상에서 Service Mesh 신규 기능이 추가되었다.
- Service Mesh는 별도의 Istio Release를 추가 배치하여  Istio-Router를 VM을 통해 Micro Service에 대한 Traffic/Security/Observability를 제공 한다.
- Beta 버전이며 아직 Pivotal Cloud Foundry에서 사용 준비가 완벽하게 되지 않음. (his release is under heavy development and not yet ready for use)

- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- Pivotal Cloud Foundry가 구축 되어 있어야 한다.

- Modify Last Version Check 2.5

## 1. Pivotal Cloud Foundry Service Mesh의 구조

- Pivotal Cloud Foundry Service Mesh는 아래와 같은 구조를 가지고 있다.
	- Data Plane:  실제 Container의 Micro Service Application 옆에 Envoy(Proxy)를 배치하여 Application의 Traffic을 통제하는 영역이다.
		- Load Balancer가 Micro Service Application에 대한 요청을 Ingress Envoy로 보낸다.
		- Ingress Envoy는 다음 요청을 보낼 Micro Service Application를 선택하여 요청을 전달 한다.
	 
	- Controller Plane: Pilot를 통해 Data Plane에 위치하고 있는 Envoy가 Micro Service Application의 End Point를 제공해주고 Envoy를 Control, 배치하는 영역이다.
		- Cloud Controller, BBS를 통해 Micro Service Application의 Route와 Mapping 정보를 Copilot에 전달한다.
		- Copilot는  전달 받은 Route와 Mapping 정보를 가공하여 Pilot에게 노출한다.
		- Pilot는 해당 구성 정보를 바탕으로 Istio-Router의 Ingress Envoy를 배치 한다.
		
![service-mesh][servicemesh-image-1]


## 2.Pivotal Cloud Foundry Service Mesh가 사용하는 Istio의 기능

- 현재 Beta인 Pivotal Cloud Service Mesh는 Release가 4개 밖에 없고 PAS Tile의 Option이 TLS 기반의 인증서를 넣는 Properties 밖에 없어 아직은 Security 만을 지원한다고 사례 된다.
- Pivotal Cloud Service Mesh가 사용하는 Istio bosh release가 아닌 opensource Istio의 주요 기능은 아래와 같다.

### 2.1. Micro Service Application의 Version 기반의 Traffic 분할
 
![service-mesh][servicemesh-image-2]

- Traffic 분할은 버전별로 Traffic 양을 조절하요 신규 Micro Service Application을 배포할때, 기존 버전으로 95%의 트래픽을 보내고, 새 버전으로 5%의 트래픽만 보내서 테스트하는 것이 가능하다.


### 2.2. Micro Service Application의 Content 기반의 Traffic 분할

![service-mesh][servicemesh-image-3]

- Network Connection 통신의 패킷 기반으로 Traffic 분할이 가능 하다, HTTP 헤더의 User-agent를 설정하여 Micro Service Application에 대한 Traffic를 분할 할 수 있다.

### 2.3. Health Check 및 Service Discovery

![service-mesh][servicemesh-image-4]

- Pilot은 Micro Service Application 인스턴스가 많을 경우 자동으로 Load Balancer 해주며 주기적으로 Health Check 하여 이상이 있는 Micro Service Application는 자동으로 제거 한다.

### 2.4. Retry, Timeout, Circuit Breaker 지원
- Micro Service Application간의 Request가 실패 할 경우 재시도 횟수를 지정 할 수 있고, 요청에 대한 일정 시간 Response가 오지 않을 경우 Timeout을 통하여 Error 처리 또는 Timeout까지의 Request에 대한 리소스를 사용하지 않도록 Open/Close/Half Open의 Circuit Breaker를 지원 한다.

### 2.5. Security

![service-mesh][servicemesh-image-5]

- 기본적으로 Envoy를 통해서 통신하는 모든 Traffic을 자동으로 TLS를 이용해서 암호화한다. Micro Service Application 통신이 Default로 TLS 암호화 된다.
- Micro Service Application 대한 인증 (Authentication)을 통해 Network Traffic의 Role을 정할 수 있다.

### 2.6. Monitoring

![service-mesh][servicemesh-image-6]

- Istio는 Envoy Network Traffic을 모니터링함으로써, 서비스간에 호출 관계가 어떻게 되고, 서비스의 응답 시간, 처리량등의 다양한 Metric을 수집하여 모니터링할 수 있다.
-   수집한 Metric 정보를 바탕으로 Prometheus, StackDriver, Datadog 등과 연동이 가능하다.

## 3.Pivotal Cloud Foundry Service Mesh Deploy

### 3.1. Pivotal Cloud Foundry Tile Configure Networking - Service Mesh (Beta) 

- IP Addresses for Ingress Router: Istio Router의 Static IP를 입력하여하고 요청에 대한 Load Balancer를 구성해야 한다.
- Ingress Router TLS Keypairs: Istio Router의 Client와 TSL 통신을 하기 위해 Certificate와 Private Key를 생성한다.

### 3.2. Pivotal Cloud Foundry Tile Configure Resource Config

- Istio Control Instance를 증가시킨다. Control Plane의 인스턴스가 HA 구성을 하지 않을 경우 배포 시 Router 경로 등록이 중단 될 수 있다.
- Istio Router Instance를 증가시킨다.
- Route Syncer Instance를 증가시킨다.

### 3.3. Apply Change

- Ops Manager UI에서 Apply Change를 실행하여 Service Mesh 변경 사항을 배포 한다.


## 3. Istio Router Application Domain 접근

![service-mesh][servicemesh-image-7]

- Istio Router VM을 추가하고 *.mesh.YOUR-APPS-DOMAIN Domain을 통해 Application에 접근 한다.

[servicemesh-image-1]:./images/servicemesh-image-1.jpg
[servicemesh-image-2]:./images/servicemesh-image-2.png
[servicemesh-image-3]:./images/servicemesh-image-3.png
[servicemesh-image-4]:./images/servicemesh-image-4.png
[servicemesh-image-5]:./images/servicemesh-image-5.png
[servicemesh-image-6]:./images/servicemesh-image-6.png
[servicemesh-image-7]:./images/servicemesh-image-7.png
