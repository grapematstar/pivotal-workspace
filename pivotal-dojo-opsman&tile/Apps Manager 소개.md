## 1. Apps Manager 소개

- Apps Manager 기능 개요
	- Apps Manager는 Pivotal Cloud Foundry의 API를 바탕으로 Org/Space관리와 ApplicationLife Cycle(start/stop/restage/service bind/scaling/Route 등) 을 관리 한다.
	- Apps Manager는 SMTP 서버 등을 통해 사용자를 초대 할 수 있으며 사용자에게 별도의 권한을 부여 할 수 있다.

### 1.1. Apps Manager 주요 화면 & 기능 소개

#### 1.1.1. Manage an Org & Space

- Pivotal Cloud Foundry 운영자는 개발자에게 특정 Org/Space를 할당하여 권한 및 사용량 할당, Service의 노출, Domain의 공유들을 지정 할 수 있다.
- Apps Manager의 Org/Space를 통해 관리하는 영역은 아래와 같다.

---

![apps-manager][appsman-image-1]

1. Manager Domains
	- Pivotal Cloud Foundry Domain을 통해 배포 한 Application에 대한 경로를 지정 할 수 있다.
	- Application을 배포하게 되면 {APPLICATION NAME}.{DOMAIN} 형식으로 경로가 생긴다.
	- Domain은 Private/Shared Domain으로 나눠지며 Private Domain는 해당 조직에서의 Http/Https만 지원한다.  Shared Domain은 모든 Pivotal Cloud Foundry의 조직에서 사용 가능하며 별도의 구성을 통하여 TCP(1024~65535)의 Port를 사용 할 수 있다.
---
---

![apps-manager][appsman-image-2]

2.  Manger Members
	- Apps Manager에서는 Org에 대한 사용자 초대 및 Org와 Space에 대한 권한을 설정 할 수 있다.
---
---

![apps-manager][appsman-image-3]

3. Setting
- Apps Manager에서는 Setting을 통해 Org/Space의 Name 수정과 Application 단의 Security Group, Metadata를 설정 할 수 있다.
---
---

![apps-manager][appsman-image-4]

4. Apps
- Apps Manager에서는 선택한 Org -> Space 화면에서 Application의 Status, Name, Instance 수, 사용 가능한 Memory 양, 마지막 Push 이후 시간, Application의 접속 경로들을 확인 할 수 있다.
---

---

![apps-manager][appsman-image-5]

5. Services
- Apps Manager의 Service 목록에는 Service, Name, Bound 한 Application 수 및 각 Service Instance의 Plan을 확인 할 수 있다.

---

---

![apps-manager][appsman-image-6]

6. Routes
- Apps Manager의 Routes 목록에는 Route와 Route에 Bound 한 Application을 확인 할 수 있다.
- 해당 Route를 클릭하면 Application으로 접속 한다.

---

#### 1.1.2. App Autoscaler를  사용하여 Application Scaling (Apps Manager Market Place Service Bind)

- Pivotal Cloud Foundry 배포 후 Push 한 Errand Autoscaler Application의 설정을 통하여Application을 Auto scale 할 수 있다.
- App Autoscaler는 Market Place로 등록되며 Apps Manager를 이용하거나 Cf CLI를 이용하여 설정 할 수 있다. 설정 방법은 아래와 같다.
	- CPU 사용량 같은 Metric를 통하여  Instance의 수를 조정하는 방법
	- Application의 Instance 수를 최소/최대로 구분하여 schedule 하여 조정하는 방법
	```
	App Autoscaler의 전제조건은 Metric 정보를 추출 하기위해
	Loggregator’s Log Cache API를 사용하여 Pivotal Cloud Foundry의
	Log Cache 설정을 Enable 해야한다.(추후 변경 될 수 있음)
	```

- App Autoscaler를 사용하기 위해서는 App Autoscaler 서비스 인스턴스를 만들어서  Auto scale 할 Application에 Bind 해줘야 한다.

- App Manager를 통하여 Market Place에 등록한 Service를 생성하고 Application에 Bind 하는 방법은 아래와 같다.
	1. Apps Manager Market Place 화면에서 접속한다.
	2. Market Place에 등록 되어 있는 App Autoscaler 선택 한다.
	3. App Autoscaler의 Plan을 선택하고 내용을 입력 한다.
		- Instance Name: Service Instance의 Name을 입력한다, Space에 고유한 이름이여여 한다.
		- Add to Space: Service Instance를 생성 할 공간을 선택 한다.
		- Bind to App: Bind 할 Application을 선택 한다.
	4. Space에 이미 만들어진 Service Instance가 있는 경우 (optional)
		- Space의 Service를 선택하고 "Bind App" 버튼을 클릭하여 Application을 Bind 한다.

![apps-manager][appsman-image-7]

- Space에서 Bind한 Application을 선택하고 Autoscaling를 가능으로 변경 -> "Manage Autoscaling"  버튼을 클릭 한다.

![apps-manager][appsman-image-8]

- Manage Autoscaling 화면에서 Config를 설정하여 자동으로 Application이 Autoscaling 되도록 설정 한다.

[apps-image-1]:./images/appsman-1.png
[apps-image-2]:./images/appsman-2.png
[apps-image-3]:./images/appsman-3.png
[apps-image-4]:./images/appsman-4.png
[apps-image-5]:./images/appsman-5.png
[apps-image-6]:./images/appsman-6.png
[apps-image-7]:./images/appsman-7.png
[apps-image-8]:./images/appsman-8.png


