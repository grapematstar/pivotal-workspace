
## 1. Ops Manager 소개

- Ops Manager 기능 개요
	- Ops Manager는 Bosh Director를 바탕으로 Bosh Config/Bosh Stemcell Upload/Bosh Release Upload/Bosh Manifest Deploy 등을 실행 한다.
	- Apply Change를 통해 설치 하는 Product Tile에 대한 로그를 실시간으로 제공 한다.
	- Ops Manager는 PCF/PMetric/PCC/Redis/Mysql/HealthWatch 등 Product Tile에 대한 Config 설정을 관리하고 설치에 필요 한 Stemcell OS를  내부적으로 관리하는 한다.
	- 설치 한 Product Tile의 간략한 VM Spec의 Status(CPU/Memory/Disk 사용률, AZ 등)와 사용한 Credential 정보를 확인 할 수있다.

### 1.1. Ops Manager Install Dash Board 소개

![Opsmanager Install Dashboard Image][opsman-image-0]

- A: Import Product = Pivtotal Netowork에서 다운로드 한 Ops Manager에 호환이 되는 Product Tile을 추가,  F의 Setting에서 External API Access를 Pivotal Network로 설정하면 자동으로 Ops Manager 버전에 맞는 Product Tile들이 나타난다.
- B: Delete All Unused Products = 사용하지 않는 모든 Product Tile을 삭제한다.
- C: Installation Dashboard = Ops Manager Main Dash 화면
- D: Stemcell Library = 각 각의 Product Tile 설치 할 Stemcell OS가 존재하고 설치에 필요한 Stemcell Version을 fix 할수 있다.
- E: Change Log = Bosh Manifest Deploy를 실행 한 결과에 대해 Log를 확인 할 수 있다.
- F: User Account Menu = 사용자 설정 메뉴,  Logout, Ops Manager Config를 변경 할 수 있다.
- G: Revert = 현재 변경 한 Product Tile 들의 Config 들을 이 전 성공한 Deploy Version으로 돌아갈수 있다.
- H: Review Pending Changes = Apply Change(Bosh Manifest Deploy)를 대기하고 있는 Product Tile의 목록이 보여지며 Product Tile의 Check를 활성/비활성화하여 Apply Change 할 대상을 정할 수 있다.
- I: Orange Bar = Product Tile에 대해 Apply Change 전 필요한 Config가 더 필요 할 경우 나타난다.
- J: Missing Stemcell Link = 특정 Product Tile을 배포 할 때 필요한 Stemcell이 D: Stemcell Library에 존재하지 않거나 호환하는 Stemcell이 없을 경우 발생 한다.
- K: API Docs = Ops Manager 자체의 API 사용 가이드가 나타난다.

### 1.2. Ops Manager Config Setting

- 아래의 Image에서 Ops Manager의 자체 설정 정보를 변경 할 수 있다.

![Opsmanager Install Dashboard Image][opsman-image-1]

#### 1.2.1. Decryption Passphrase

![Opsmanager Install Dashboard Image][opsman-image-2]

- Ops Manager 초기 설치 시 설정 한 Decryption Passphrase를 수정 할 수 있다.

#### 1.2.2. Change Authentication Method to SAML
- Identity Provider 변경 방법
	- Current Decryption Passphrase: 현재 사용중인 Decryption Passphrase 입력
	- SAML IDP Metadata: SAML IDP의 전체 URL 또는 XML Metadata 입력
	- BOSH IDP Metadata: BOSH IDP의 전체 URL 또는 XML Metadata 입력
	- SAML Admin Group: 모든 Ops Manager Administrator가 있는 SAML 그룹의 이름
	- Groups Attribute: SAML 서버를 구성한 그룹 속성 태그 이름
	- Provision an admin client in the Bosh UAA: BOSH UAA에서 admin client르  Provision 할경우 선택
	
#### 1.2.3. SSL Certificate

![Opsmanager Install Dashboard Image][opsman-image-3]

- Ops Manager가 사용하는 UI/API의 트래픽에 대해 해당 SSL 인증서를 사용, 빈칸일 경우 자체적으로 생성한 인증서를 사용한다.

![Opsmanager Install Dashboard Image][opsman-image-4]

#### 1.2.4. External API Access
 -   Pivotal Network API의 Token을 입력하여 Ops Manager의 Version을 감지해 호환되는 Product Tile을 Install Dashboard에 보여준다.

#### 1.2.5. Proxy Settings
- Ops Manager에 연결할 때 http/https Proxy 서버를 사용할 경우 입력 한다.

#### 1.2.6. Custom Banner

![Opsmanager Install Dashboard Image][opsman-image-5]

- 운영자에게 보여질 Banner를 Text 형식으로 설정 한다. UI Banner는 Web에서 보여지며 SSH Banner는 Shell에 접속 했을 경우 나타난다. 

#### 1.2.7. Export Installation Settings
- Ops Manager에 설정 한 모든 installation  정보를 Export 한다. 실제 Ops Manager Backup 시 해당 기능의 API를 사용 한다.

#### 1.2.8. Syslog

![Opsmanager Install Dashboard Image][opsman-image-6]

- Ops Manager의 Syslog를 특정 로그 수집기(logstash, rsyslog) 등으로 보내준다.

#### 1.2.9. Advanced
- Download activity data: Ops Manager에 설치한 Product Tile 들의 배포 Manifest과 Database/Version 정보를 관리하는 데이터를 다운로드 한다.
- Download Root CA Cert: Ops Manager의 Root CA Cert를 다운로드 할 때 사용 한다.
- View diagnostic report: Ops Manager가 설치한 Product Tile들의 Deploy 정보를 확인 한다.
- Delete this Installation: Ops Manager 정보를 삭제 한다.

### 1.3. Ops Manager 주요 화면 & 기능 소개

#### 1.3.1. Review Pending Changes 화면

![Opsmanager Install Dashboard Image][opsman-image-7]

- A: Select All Products = Ops Manager에 등록한 모든 Product Tile들이 선택되고 Apply Change 시 선택 한 Product들이 배포/수정 배포 된다.
- B: Select Product = Apply Change를 실행 할 단일 Product Tile들에 대한 선택이 된다, Director Tile은 항상 Defaut로 선택되어 있다.
- C: Product Listing = 각 각의 Product Tile 들이 Apply Change가 가능한지에 대한 확인이 색상으로 표기 된다.
	-  Green: Apply Change를 할 준비가 완료 상태 
	-  Orange: Product Tile의 Config가 정상적으로 구성되지 않은 상태
	-  Red: Product Tile이 삭제 대기 상태
- D: Depends on = 대상 Product Tile의 종속성과 Version을 보여준다.
- E: Warnings = Product Listing의 Orange 색상의 Product Tile에 대한 필요한 Config 구성 요소를 나타낸다.
	- Missing stemcell: Stemcell Library에서 해당 Product Tile이 사용하는 Stemcell이 존재하지 않음.
	- Stemcell out of date: Stemcell Library에서 해당 Product Tile이 사용하는 Stemcell의 Version이 맞지 않음
	- Configuration is incomplete: Product Tile의 구성이 누락
	- Configuration is invalid: Product Tile의 구성 중 잘 못된 부분이 있음
- F: Changes = Apply Change 시 변경 될 부분에 대해 아래와 같은 요소로 나타난다.
	- Product Tile 삭제
	- Product Tile Version 업그레이드
	- Stemcell Version 업그레이드
- G: Errands = Product Tile을 Apply Change 할 때 Trigger로 실행 할 Errands를 설정 할 수있다.
- H: Apply Changes = Select 한 Product Tile을 전체 배포 한다.

#### 1.3.2. Adding and Deleting Products

![Opsmanager Install Dashboard Image][opsman-image-8]

- Ops Manager에서 사용 할 수있는 Product Tile은 Pivotal Network에서 다운로드 가능하다.

![Opsmanager Install Dashboard Image][opsman-image-9]

- Pivotal Network에서 다운로드 한 Product Tile을 Installation Dash board의 "import product" 버튼을 통해 Ops Manager에 넣는다.

![Opsmanager Install Dashboard Image][opsman-image-10]

- Ops Manager의 Version에 호환하는 Product Tile Version이 아닐 경우 호환하는 Version을 출력 한다.
- 녹색의 "+" 아이콘을 클릭하게 되면 Product Tile이 Ops Manager에 등록 된다.

![Opsmanager Install Dashboard Image][opsman-image-11]

- Product Tile이 주황색일 경우 설정해야 할 정보가 필요함을 의미
- Product Tile 구성을 완료하고 "Apply Change" 버튼을 클릭한다.

#### 1.3.3. Stemcell Library 화면

- Stemcell OS를 Ops Manager 화면에 등록 해 Product Tile에 적용 시킬수 있다.

![Opsmanager Install Dashboard Image][opsman-image-12]

- Pivotal Network 또는 bosh io 에서 Stemcell을 다운로드 한다.

![Opsmanager Install Dashboard Image][opsman-image-13]

- "Import Stemcells" 버튼을 클릭하여 다운로드 한 Stemcell을 Ops Manager에 Import 시킨다.

![Opsmanager Install Dashboard Image][opsman-image-14]

- Product Tile에서 사용할 Stemcell Version을 선택 한다.

#### 1.3.4. Ops Manager 사용자 생성
- Ops Manger는 사용자를 관리하기 위해 설치 시 Login Server인 UAA와 연동 되어 있다. UAA에 Ops Manager 계정을 추가 하기 위해 Ops Manager Shell에 접속하여 진행 한다.
```
# Ops Manager UAA 서버를 Targeting 한다.
$ uaac target https://OPS-MAN-FQDN/uaa

# uaac 명령어를 통해 사용자를 생성하기 위하여 uaac token을 받아온다.
$ uaac token owner get
Client ID: opsman
Client Secret: [Press Enter]
Username: Admin
Password: *******
 
Successfully fetched token via client credentials grant.
Target https://YOUR-OPSMANAGER-FQDN/uaa/

# uaa에 사용자를 등록한다.
$ uaac user add USER-NAME -p USER-PASSWORD --emails USER-EMAIL@EXAMPLE.COM
```

- 위 절차를 실행하게 되면 Role이 아무것도 존재하지 않은 사용자가 생성되고 Ops Manager UI에 접근하지 못한다. Ops Manager Role은 아래와 같다.
	- Ops Manager Administrator: Scope = opsman.admin (admin 권한)
	- Full Control: Scope = Scope = opsman.full_control (전체 통제 권한)
	- Restricted Control: Scope = opsman.restricted_control (부분적 통제 권한)
	- Full View: Scope = opsman.full_view (전체 읽기 권한)
	- Restricted View: Scope: opsman.restricted_view (부분적 읽기 권한)

- 아래 명령어를 통해 생성한 사용자에게 권한을 부여 한다.
```
$ uaac member add opsman.full_view admin1
success

# 읽기 권한만 할당한 사용자는 Ops Manager 화면이 Block 처리 됨을 확인한다.
```

[opsman-image-0]:./images/opsman1.png
[opsman-image-1]:./images/opsman2.png
[opsman-image-2]:./images/opsman3.png
[opsman-image-3]:./images/opsman4.png
[opsman-image-4]:./images/opsman5.png
[opsman-image-5]:./images/opsman6.png
[opsman-image-6]:./images/opsman7.png
[opsman-image-7]:./images/opsman8.png
[opsman-image-8]:./images/opsman9.PNG
[opsman-image-9]:./images/opsman10.png
[opsman-image-10]:./images/opsman11.png
[opsman-image-11]:./images/opsman12.png
[opsman-image-12]:./images/opsman13.PNG
[opsman-image-13]:./images/opsman14.png
[opsman-image-14]:./images/opsman15.png

