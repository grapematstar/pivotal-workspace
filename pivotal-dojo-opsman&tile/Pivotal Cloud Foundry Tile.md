##  Pivotal Cloud Foundry Tile

- PAS Tile은 Pivotal Cloud Foundry를 설치하는 Tile으로 Pivotal Cloud Foundry 설치에 대한 전반적인 Domain, Network, Component, Errand 등의 Config 정보를 설정 할 수있다.
- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
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

[Configure Domains][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html)

- System/Application Domain 정보를 입력 한다.
	- ex) sys.example.com, apps.sys.example.com
- Pivotal은 System Domain과 Application Domain을 Single Wildcard Certificate로 사용 할 수 있도록 권고 한다.


#### 1.3. Configure Networking
- Router IPs, HAProxy IPs: vSphere 환경에서는 HA Proxy VM사용 유무에 따라 Router IP의 설정 방법이 달라진다.
	- HAProxy를 사용하지 않을 경우
		- HA를 구성하기 위해 2개 이상의 Router IP를 입력 한다.
		- Pivotal Cloud Foundry 사용 Doamin에 대한 요청을 Router IP에 Load Balancer 구성 한다.
	- HAProxy를 사용할 경우
		- HA를 구성하기 위해 2개 이상의 Ha Proxy IP를 입력 한다.
		- Pivotal Cloud Foundry 사용 Doamin에 대한 요청을 Ha Proxy IP에 Load Balancer 구성 한다.
- SSH Proxy IPs: SSH 프록시 IP에서 Diego Brain의 IP 주소를 추가하면 SSH에 포트 2222의 응용 프로그램 컨테이너에 대한 요청을 해당 Brain IP로 받는다.
- TCP Router IPs: TCP Router를 사용하게 되면 Application 경로의 Port를 80/443아닌 다수의 TCP Port Range로 설정 할 수있으며 해당 기능을 사용하기 위해 특정 Config를 추가 해야 한다.
- Certificates and Private Keys for HAProxy and Router: HaProxy 와 Go Router에 대한 TSL Certificate/Key 생성 또는 입력
- Certificate Authorities Trusted by Router and HAProxy: HaProxy 와 Go Router에 Trusted Certificate를 추가 할 경우 입력
- Minimum version of TLS supported by HAProxy and Router: HAProxy와 Gorouter에서 사용하는 TLS의 Version을 선택 한다.
- Logging of Client IPs in CF Router:  Router에서 Client IP에 대한 Log의 노출 정보를 확인
- Configure support for the X-Forwarded-Client-Cert header. This header can be used by applications to verify the requester via mutual TLS. The option you should select depends upon where you will be terminating the TLS connection for the first time: TLS 통신이 처음 종료되는 위치를 기반으로 XFCC (x-forwarded-client-cert) HTTP 헤더를 처리 선택
- HAProxy behavior for Client Certificate Validation: HAProxy에서 Client Certificate 를 처리하도록 구성선택
- Router behavior for Client Certificate Validation: Router에서 Client Certificate 를 처리하도록 구성 선택
- HAProxy support for HSTS: HAProxy가 HSTS(HTTP Strict Transport Security)를 제공 유무 선택
- Disable SSL certificate verification for this environment: Pivotal Cloud Foundry에 대한 SSL certificate 확인을 시작/중지 선택, 자체 인증서를 사용 할 경우 해당 기능을 비활성화 할 수 있다.
- Disable HTTP on HAProxy and Router: HA Proxy와 Router에서 Http(80)을 수신 할 것인지 선택
- Disable insecure cookies on the Router: Router에서 생성한 Cookie에 대한 보안 Flag 사용 선택
- Routers reject requests for Isolation Segments: Router가 Isolation Segment를 reject 할 것인지 선택
- Enable support for PROXY protocol in CF Router:  Router가 각 요청에 대해 PROXY가 존재 하는지 확인한다.(성능 저하가 될 수 있음)
- Choose whether to enable route services. Route services enable you to proxy requests to your app over TLS to arbitrary URLs before hitting your app: Route Service를 사용 할 것인지 선택
 - Enable Zipkin tracing headers on the Router: Router에서 요청에 대한 Zipkin(분산 로그 추적) Header를 사용 할 것인지 선택
 - Enable Router to write access logs locally: Router의 Access Log를 Local에 저장 할 건지 선택, Enable 할 경우 Local Disk가 Pull이 나지 않도록 관리 해야 한다.
 - Enable Keepalive Connections for Router: Router와 Clinet 대한 연결을 Keepalive 할 것인지 선택, HTTP의 응답을 반환하고 TCP 통신은 Keepalive
 - Max Connections Per Backend:  Router의 각 Instance에 대한 Back End 당 최대  TCP 연결 , 실제 Application의 연결 수를 제한 할 수 있다. 0이면 무제한
 - Router Timeout to Backends: Router와 Application, PAS Component 끼리의 통신 대기 시간, 업로드 량이 큰 Application을 사용 할 경우 해당 대기 시간을 늘려 준다.
 - Frontend Idle Timeout for Router and HAProxy: Gorouter 또는 HAProxy 로의 연결이 너무 일찍 종료되는 것을 방지
 - Load Balancer Unhealthy Threshold: Router VM이 shutting down시 외부 Load Balancer의 연결을 대기하는 시간
 - Load Balancer Healthy Threshold: PAS 배포 시 Router VM이 외부 Load Balancer 에 연결 할때 까지 대기 하는 시간
 - HTTP Headers to Log: Router Access Log에 HTTP Header 주석을 입력
 - HAProxy Request Max Buffer Size:  Ha Proxy 요청에 대한 Buffer 크기(Byte)
 - HAProxy Protected Domains: Trusted CIDRs과 함께 알 수 없는 출처의 요청에 대해 Protected 되는 Domain의 주소, ",로 구분"
 - HAProxy Trusted CIDRs: Protected Domains에서 허용하는 PAS의 CIDR 입력
 - Loggregator Port: Default 443 PORT의 Loggregator Component PORT 변경
 - Container Network Interface Plugin
	 - Silk: PAS 기본 Container Network Instance(CNI) 사용 할 경우 설정
	 - External: vSphere의  PAS 용 VMware NSX-T Container Plugin을 배포 할때 설정
- DNS search domains: Container에서 사용 할 DNS Server Name (",로 구분")
- Database Connection Timeout: Policy 서버 및 Silk Database의 Client에 대한 연결 시간
- Select this option if you prefer to enable TCP Routing at a later time: TCP Router를 사용하지 않을 경우 선택
- Enable TCP Routing: TCP Router를 사용할 경우 선택하여 Port 지정
- Enable Dynamic Egress Enforcement: Dynamic Egress을 사용하면 Application이 외부 Service와 연동 할 때 재시작을 하지 않아도 된다 (Beta)
- Remove Specified HTTP Response Headers: Router가 모든 Application에서 반환 되는 Response에서 제거할 HTTP Header 목록

#### 1.4.  Configure Networking - Service Mesh (Beta) 
- Service Mesh는 Micro Service에 대한 Traffic 관리, Security를 제공한다.
- IP Addresses for Ingress Router: Istio Router의 Static IP를 입력하여하고 요청에 대한 Load Balancer를 구성해야 한다.
- Ingress Router TLS Keypairs: Istio Router의 Clinet와 TSL 통신을 하기 위해 Certificate와 Private Key를 생성한다.

#### 1.5.   Configure Application Containers 

[Configure Application Containers][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking)

- Enable Custom Buildpacks:  cf push 명령의 -b 옵션을 통해 사용자 정의 빌드 팩을 사용 할 것인지 선택
- Allow SSH access to app containers: HA Proxy또는 Load Balancer를 통하여 Default 2222번 Port를 바탕으로 Application에 대한 SSH 접속을 허용/금지 할 것인지 선택
- Enable SSH when an app is created: Application을 만들 때 SSH 접속을 허용
- Router application identity verification:  Router와 Application 사이의 ID 값을 확인하기 위해 양방향으로 할것인지, 한 방향으로 할것인지 선택
-  Private Docker Insecure Registry Whitelist: Private Docker Repo Domain에 대한 Insecure 설정, ",로 구분"
- Docker Images Disk-Cleanup Scheduling on Cell VMs:  실제 Application이 배치 되는 Cell VM에서 사용한 Docker Image에 대한 Cleanup 주기를 선택
- Max Inflight Container Starts: Diego Cell VM에서 실행 하는 최대 Application Conatiner 수
- Enabling NFSv3 volume services: Application 상에서 공유 파일 Access를 위해 NFS Volume을 bind 할 수 있다.
- Format of timestamps in Diego logs: Diego의 Log Time Stamp 형식 선택
- Default health check timeout: Application의 시작과 첫 정상 동작 사이의 시간

#### 1.6. Configure Application Developer Controls

[Configure Application Developer Controls][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking)

- Maximum File Upload Size (MB) min= 1024, max= 2048: Application Upload의 최대 크기
- Default App Memory (MB) min: 64, max: 2048 : Application 배포 시 할당 하는 Default Memory
- Default App Memory Quota per Org (MB) min: 10240, max: 102400: Application을 배포 하는 Org의 총 Memory 할당량
- Maximum Disk Quota per App (MB) min: 512, max: 20480: Application 배포 시 할당 하는 최대 Disk Size
- Default Disk Quota per App (MB) min: 512, max: 20480: Application 배포 시 할당 하는 Default Disk Size 
- Default Service Instances Quota per Org min: 0, max: 1000: Org에 할당 할 수 있는 최대 Service Instance 수
- Staging Timeout (Seconds): Application이 Staging 할때 걸리는 대기 시간
- Internal Domain: Application이 내부 DNS 서비스 검색에 사용하는 Domain 입력
- Allow Space Developers to manage network policies: 자체 Network 정책을 설정 할 경우 선택

#### 1.7. Configure Application Security Groups
- Pivotal Cloud Foundry를 통해 배포 한 Application의 내부 Security Groups, X로 입력 할 경우 PAS에 전체 허용, 설치 후 CF CLI로수정 및  추가 구성 할 필요가 있다.

[Configure Application Security Groups][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking)

#### 1.8. Configure Authentication and Enterprise SSO
- 내부 UAA Component, External, SAML Identity Provider, LDAP을 통하여 사용자 인증 서버를 선택 할수 있다.

[Configure Authentication and Enterprise SSO][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking)

- Internal UAA
	- Minimum Password Length:  최소 암호의 길이 입력
	- Minimum Uppercase Characters Required for Password: 최소 대문자 길이 입력
	- Minimum Lowercase Characters Required for Password: 최소 소문자 길이 입려 
	- Minimum Numerical Digits Required for Password:  유효한 암호에 대한 최소 자리 수 입력
	- Minimum Special Characters Required for Password: 최소 특수 문자 길이 입력
	- Maximum Password Entry Attempts Allowed: 계정이 잠기기 전 최대 실패 허용 수

#### 1.9. Configure UAA
- Pivotal Cloud Foundry UAA 사용자 계정 및 인증 서버 서버를 구성 할수 있다.

[Configure UAA][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#networking)

- Choose the location of your UAA database: UAA Database를 Internal/External 사용 설정 할 수 있다.
- JWT Issuer URI:  JWT(JSON WEB TOKEN) Issuer Server URI을 통하여 UAA Token을 발급 받을 경우 URI를 입력
- SAML Service Provider Credentials: *.login.YOUR-SYSTEM-DOMAIN 과 연동이 되는 SAML 인증서 생성 및 입력
- SAML Service Provider Key Password: SAML Service Provider의 Passphrase 입력
- SAML Entity ID Override: 사용자 지정 SAML Entity ID 입력, Default는  login.YOUR-SYSTEM-DOMAIN
- Signature Algorithm: Signature 알고리즘 선택, Default: sha256
- Apps Manager Access Token Lifetime (in seconds): Apps Manager Access Token의 수명 주기 설정
- Cloud Foundry CLI Access Token Lifetime (in seconds): CF CLI Access Token의 수명 주기 설정
- Cloud Foundry CLI Refresh Token Lifetime (in seconds): CF CLI Refresh Token의 수명 주기 설정
- Global Login Session Max Timeout (in seconds):  전역 Login 세션 유효 최대 시간 설정
- Global Login Session Idle Timeout (in seconds): 전역 Login 세션 유효 최대 시간 초과 설정
	- Global Login Session 종류는 UAA를 사용하는 Apps Manager, PCF Metrics, and other web UI 또는 SSO를 사용하는 Application
- Customize Username Label (on login page):  UAA Login 화면에서 나오는 Username Text를 변경 
- Customize Password Label (on login page): UAA Login 화면에서 나오는 Password Text를 변경 

#### 1.10. Configure CredHub
- Pivotal Cloud Foundry의 내부 Component의 민감한 정보를 가지고 있는 Credhub 설정을 구성 할수 있다.

- Choose the location of your Credhub database: Credhub Database를 Internal/External 사용 설정 할 수 있다.
- Encryption Keys: CredHub 데이터를 암호화하고 해독하는 데 사용하는 Key 여러개를 지정 했을 경우 반드시 하나의 Primary Key를 선택해야 한다.
 -   Provider Partition: Encryption Key를 저장하기 위한 파티션 (참조  [https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html))
-   Provider Partition Password: Provider Partition Password (참조  [https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html))
-   Provider Client Certificate: Credhub가 HSM Client와 연결 할 때 사용하는 certificate
-   Provider Client Certificate Private Key: Credhub가 HSM Client와 연결 할 때 사용하는 Certificate
-   HSM Host Address: HSM Server 주소
-   HSM Port Address: HSM Server Port (Default HSM Server Port는 1792)
-   Partition Serial Number: Partition 번호
-   HSM Certificate: Credhub와 HSM 사이의 양방향 통신을 위한 mTLS Key
- Secure Service Instance Credentials: ????

#### 1.11. Configure System Databases
- Pivotal Cloud Foundry 운용에 있어 필요한 Database 정보를 Internal/External Database에 저장 할 것인지 선택
- External Database로 지정하게 되면 아래 목록에 있는 DB 정보의 Host/Port/Username/Password 등 을 입력해야 한다.
	-  `account`
	-   `app_usage_service`
	-   `autoscale`
	-   `credhub`
	-   `diego`
	-   `locket`
	-   `networkpolicyserver`
	-   `nfsvolume`
	-   `notifications`
	-   `routing`
	-   `silk`
	-   `uaa`

#### 1.12. Configure Internal MySQL
- Pivotal Cloud Foundry Tile에서 설정한 Mysql에 대한 Internal Config를 설정 할 수 있다.
- Replication canary time period: HA 구성 Mysql의 Read-Write 중 Write에 대한 Replication Canary가 실행 되는지에 대한 시간 설정 
- Replication canary read delay: 각 MysqlNode에서 데이터가 복제되는지 확인하기 전에 Canary가 대기하는 시간을 설정
- E-mail address (required): Cluster에서 Replication 문제가 발생할 경우 E-Mail 알람 설정
- Allow Command History: Mysql 명령 행 히스토리 파일을 생성 여부 설정
- Allow Remote Admin Access: 원격 호스트에서 Remote하여 접속 할 수 있는 여부 설정
- Cluster Probe Timeout: 새로운 Mysql Node가 Cluster를 검색 할 때 걸리는 시간 설정
- Max Connections: Mysql 접속 최대 연결 수
- Server Activity Logging:  Internal Mysql에 대한 Event Activity Logging 사용 여부 설정
- Load Balancer Healthy Threshold: Mysql Proxy Instance가 시작될 때까지 대기 할 시간
- Load Balancer Unhealthy Threshold min: 0: Instance shutting down 전에 Mysql Proxy가 연결을 계속 받아 들일 시간
- Prevent node auto re-join: Node 사이의 데이터 집합에 불일치가 있음을 알게되면 Mysql Database에 대한 모든 쓰기를 중지 여부 설정

#### 1.13. Configure File Storage
- Pivotal Cloud Foundry Tile에서 Cloud Controller가 사용하는 Blob 저장소 Config를 설정 할 수 있다.
- Max Valid Packages per App min: 1:  각 Application이 최근 저장한 Package의 수의 최대 값
- Max Staged Droplets per App min: 1:  각 Application이 최근  Staged 단계 중 저장한 Droplets 수의 최대 값 
- Configure your Cloud Controller's filesystem:  Blob 저장소의 Internal/External 위치 선택
	- Internal WebDAV: WebDAV 방식으로 Pivotal Cloud Foundry 내부 Instance를 Blob 저장소로 사용
	- External S3-Compatible File Store (if you want to use a service like S3 or Ceph): S3 Server를 Blob 저장소로 사용
	- External Google Cloud Storage with Access Key and Secret Key: Access Key/Secret Key를 통한 Google Cloud Storage를 Blob 저장소로 사용
	- External Google Cloud Storage with Service Account: Service Account를 통한 Google Cloud Storage를 Blob 저장소로 사용
	- External Azure Storage: Azure Storage Account를 통한 Azure Storage를 Blob 저장소 사용

#### 1.14. Configure System Logging

- Pivotal Cloud Foundry Tile에서 rsyslog를 구성하여 Flatform 구성 요소, Application Container Log를 External로 전달할 수 있도록 Config 설정이 가능 하다
 
[Configure System Logging][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#auth-config](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html#auth-config)

- Address: Syslog Server의 주소 입력
- Port: Syslog Server의 Port 번호 입력
- Transport Protocol: Log 전달 전송 Protocol 선택
- Encrypt syslog using TLS: Log를 전달 할 때 TSL 통신을 할 것 인지 선택
- Number of syslog drain addresses to retrieve: Cloud Controller이 검색 할 수 있는 Syslog Drain 주소가 있는 Application 수
- Include container metrics in SysLog Drains: Syslog Drain에 App Container Log를 포함 할 건지 선택
- Enable Cloud Controller security event logging: Cloud Controller의 Event Log를 Log Stream에 포함 시킬 건지 선택 (API End Point, User, Source 등 노출)
- Use TCP for file forwarding local transport: TCP를 통하여 Log를 전달 할 것인지 선택, Log의 유실은 막지만 성능에서 문제가 발생 할 수 있음.
- Don't Forward Debug Logs: Debug Log를 사용 할 것인지 선택
- Custom rsyslog Configuration: 사용자 정의 Rsyslog 규칙을 입력

#### 1.15. Configure Custom Branding and Apps Manager
- Pivotal Cloud Foundry Tile에서 Custom Logo, Background Color, Footer Text 등을 설정하여 사용자 Brand를 구성 할 수 있다.

[Configure Custom Branding and Apps Manager][[https://docs.pivotal.io/pivotalcf/2-5/opsguide/custom-branding.html](https://docs.pivotal.io/pivotalcf/2-5/opsguide/custom-branding.html))

- Company Name: 회사/조직 명 입력
- Accent Color: CSS Type 강조 색상 입력
- Main Logo (PNGs only): PNG Image의 Base64 인코딩 URL 문자열 입력
- Square Logo (PNGs only): PNG Image의 Base64 인코딩 URL 문자열 입력
- Favicon (PNGs only): favicon으로 사용할 PNG Image의 Base64 인코딩 URL 문자열을 입력
- Footer Text: Footer에 노출 할 Text 입력
- Classification Header/Footer Background Color: CSS Type Header/Footer의 Background색상 입력
- Classification Header/Footer Text Color:  CSS Type Header/Footer의 Text 색상 입력
- Classification Header Content: Header의 Content 입력
- Classification Footer Content: Footer의 Content 입력

#### 1.16. Apps Manager Config Page

[Apps Manager Config Page][https://docs.pivotal.io/pivotalcf/2-5/opsguide/custom-branding.html](https://docs.pivotal.io/pivotalcf/2-5/opsguide/custom-branding.html)

- Enable Invitations: Apps Manger에서 새로운 사용자를 초대 할 수있는 기능을 사용 할 것인지 선택
- Display Marketplace Service Plan Prices: Market Place의 Service Plan의 가격을 표시 할 것 인지 선택
- Supported currencies as JSON: Currencies Type을 Json 형태로 입력
- Product Name: Apps Manager Product의 표기 명 입력
- Marketplace Name: Apps Manager의 Market Place 표기 명 입력
- Customize Sidebar Links: 참고 할 Doc, Market Place, Tool의 URI 입력
- Apps Manager Memory Usage (MB): Apps Manager App의 Memory 사용량 입력
- Invitations Memory Usage (MB): Invitations App의 Memory 사용량 입력
- Apps Manager Polling Interval:  Apps Manager의 사용 요청에 대한 Cloud Controller 응답 시간을 설정 (Default로 사용하는 것을 권고 0-30)
- App Details Polling Interval: Apps Manager Polling Interval 간격을 통해 만족하는 결과가 이루어지지 않을 때 Cloud Controller의 Load를 줄임
- Multi-Foundation Configuration (BETA):  Apps Manager가 연동 할 Apps Manager Interface에서 여러 PCF 기반의 Space, Application 및 Service Instance를 관리 할 수 ​​있다.

#### 1.16. Apps Manager Config Page Configure Email Notifications
- Pivotal Cloud Foundry Tile에서 Email Notifications를 설정하여 SMTP를 사용하여 초대 및 확인을 Apps Manager User에게 보낼 수 있다.

[Apps Manager Config Page Configure Email Notifications][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html)

- From Email: Apps Manager 사용자에게 보낼 Mail 주소 입력
- SMTP Server Address: SMTP 서버 주소 입력
- SMTP Server Port: SMTP 서버 Port 입력
- SMTP Server Credentials: SMTP의 자격 증명
- SMTP Enable Automatic STARTTLS:  SMTP STARTTLS (TSL,SSL) 사용 여부 선택
- SMTP Authentication Mechanism: SMTP 인증 방법 선택
- SMTP CRAMMD5 secret: 인증 방법에 CRAMMD5를 선택한 경우 입력

#### 1.17. Configure the App Autoscaler
- Pivotal Cloud Foundry는 App Autoscaler 설정을 통하여 개발자가 배포한 Application을 Autoscale할 수 있도록 한다.

[Configure the App Autoscaler][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html)

- Autoscaler Instance Count: App Autoscaler Application의 Instance 수, HA 구성을 위해 3개일 필요가 있으며 split-brain를 피하기 위해 홀수로 설정 한다.
- Autoscaler API Instance Count: Autoscaler API Application의 Instance 수
- Metric Collection Interval:  App Autoscaler 가이 개발자가 배포한 Application의 수집하는 데이터의 Metric 수집 시간
- Scaling Interval: App Autoscaler가 개발자가 배포 한 Application의 크기를 조정을 평가하는 시간
- Verbose Logging: App Automoscaler가 개발자가 배포한 Application을 확장 한 구체적 Log를 배출
- Disable API Connection Pooling: Autoscaler API가 HTTP 통신을 사용 유무를 선택
- Enable Notifications:  App Autoscaler Event에 e-mail 알람을 받을지 선택

#### 1.18. Configure the Cloud Controller

- Pivotal Cloud Foundry에서는 실제 API 요청을 받는 Cloud Controller에 대한 Config 설정이 가능 하다.

[Configure the Cloud Controller][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html)

- Cloud Controller DB Encryption Key: Cloud Controller DB 암호화 키를 입력(Cloud Controller DB에 대해 PAS Backup을 통해 재설치 할 경우)
- Enabling CF API Rate Limiting will prevent API consumers from overwhelming the platform API servers. Limits are imposed on a per-user or per-client basis and reset on an hourly interval.: 운영자가 API를 사용 할 때 모든 API Endpoint, 비인증 요청 대한 Limit을 설정 할 수 있다.
- Database Connection Validation Timeout: Cloud Controller Database cc_db 연결에 대한 유효성 Check 시간
- Database Read Timeout: Cloud Controller Database cc_db Read 시간 초과를 초 단위로
- Type "X" to acknowledge that you have no applications running with cflinuxfs2. If you are upgrading from 2.4, please read this documentation closely for steps to verify all apps have been migrated: https://docs.pivotal.io/pivotalcf/2-5/upgrading/checklist.html#cflinuxfs3: cflinuxfs2에서 실행중인 응용 프로그램이 없다는 것을 확인하려면 X를 입력하고 cflinuxfs2를 사용한다면 cflinuxfs3으로 커스텀아이징 단계를 실행 해야 한다.
- Age in days after which completed tasks will be pruned from the Cloud Controller Database: 완료한 Task가 Cloud Controller Database cc_db에서 정리 되는 시간(days)

#### 1.19. Configure Smoke Tests
- Pivotal Cloud Foundry Tile에서 PAS Deploy, PAS Errand의 실행 후 간단한 TEST를 할 수 있는 Smoke Test의 설정이 가능하다.

[Configure Smoke Tests][https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/configure-pas.html) 

- Choose where to deploy applications when running the smoke tests: Smoke Test가 이루어질 공간을 선택한다. 사용자가 지정할 경우 해당 공간은 Smoke Test를 위해 생성 되었다가 완료 후 삭제 한다.

#### 1.20. Configure Advanced Features

- Pivotal Cloud Foundry Tile에서 PAS 설정에 대한 특정 고급 설정을 할 수 있다.
- Diego Cell Memory and Disk Overcommit: Diego Cell VM의 Memory와 Disk에 대한 Overcommit에 대한 Config 설정이 가능하다.
- Whitelist for non-RFC-1918 Private Networks: Pivotal Cloud Foundry의 내부 저장소 Blobstore(WebDev)가 다른 Pivotal Cloud Foundry Component와 통신 할 수 있도록 추가 구성 한다, CIDR 입력
- CF CLI Connection Timeout: Cloud Foundry 명령 줄 CF CLI에 대한 Timeout 시간을 높일 수 있다, Default 5초
- Enable SMB volume services: SMB Volume Service를 활성화하여 기존의 SMB 서버를 Application에 Bind 할 수 있도록 Service Broker를 생성한다, Default로 설정 시 Errand의 SMB Broker를 OFF 시키고 배포한다.
- BETA: Enable TLS for internal system database: Internal Database의 Clinet에 대해 TLS를 사용하도록 설정 한다.
- Database Connection Limits: Diego와 Application Container가 가질 수 있는 최대 Networking 수
- Disable Zero Downtime App Deployments: Application의 Rolling Update를 중지 시킬수 있다.
- The maximum number of items stored in Log Cache per source: source_id가 가질수 있는 Log Cache의 최대 수

#### 1.21.  Configure the Metric Registrar
- Pivotal Cloud Foundry Tile에서 Metric Registrar를 활성화하여 Component의 구조화 한 Log를 Metric으로 변환하여 Loggretor로 보낼 수 있다.
- Enable Metric Registrar: Metric Registrar를 사용 할 것 인지 확인
- Endpoint Scraping Interval: Metric Registrar가 사용자가 정의한 Metric API를 폴링하는 시간 설정
- Blacklisted Tags: Metric Registrar가 Metric 또는 Event Tag 값을 사용하지 못하도록 하기 위한 설정 값

#### 1.22. Configure Errands

- Pivotal Cloud Foundry Tile에서 PAS 설치 후 실행 하는 Errand, PAS 삭제 전 실행 하는 Errand에 대한 정보를 설정 할 수 있다.
- Smoke Test Errand: Smoke Test를 실행하면 Application Push/Scale/Delete, Org/Space Create/Delete을 수행 할 수 있는지 Test 한다.
- Usage Service Errand: Usage Service Errand는 Apps Manager에서 사용하기 위한 Dependency로 Application 형태로 배포 한다.
- Apps Manager: Apps Manager는 실제 Pivotal Cloud Foundry 관리 GUI Application을 배포 한다.
- Notifications Errand: Notifications Errand는 Pivotal Cloud Foundry 사용자가 e-mail을 통해 알람을 받기 위한 API를 Application으로 배포 한다.
- Notifications UI Errand: Notifications UI Errand는 사용자의 알림 구독을 관리 하기 위해 Application으로 배포하는 Dashboard
- App Autoscaler Errand: App Autoscaler Service Broker를 사용 하기 위해서 배포하는 Application
- NFS Broker Errand: Pivotal Cloud  Foundry에서 NFS Service Broker를 사용하기 위해서 배포하는 Application
- Metric Registrar Smoke Test Erand: 사용자 정의 Metric를 Access하여 Loggregator Metric으로 변환 할 수 있는지 Test 한다.
- SMB Broker Application Errand: SMB Broker Application Errand는 Pivotal Cloud  Foundry에서 SMB Service Broker를 사용하기 위해서 배포하는 Application

#### 1.23. Resource Config
- Pivotal Cloud Foundry Tile에서 PAS 설치 시 사용 할 VM에 대한 Spec를 설정 할 수 있다.
- 만약 External S3를 Blobstore로 구성한다면 Resource Config File Storage의 Instance를 0으로 구성 한다.
- 만약 UAA, System, and CredHub, Cloud Controller를 External Databases 구성 하였다면 Resource Config MySQL Proxy/MySQL Server/MySQL Monitor의 Instance를 0으로 구성 한다.
- 만약 TCP Router를 사용하지 않는다면 Resource Config TCP Router의 Instance를 0으로 구성 한다.
- 만약 HA Proxy를 사용하지 않늗나면 Resource Config HAProxy의 Instance를 0으로 구성 한다.
