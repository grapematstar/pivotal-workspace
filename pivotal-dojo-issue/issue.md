#### 1. PCF VM, App Log TimeZone 설정 문제
  - Bosh는 NTP 설정이 필요
  - VM은 Bosh Runtime config로 Release Post Script를 통해 수정 가능
  - APP은 CF Push 중 manifest env, jave env 설정 가능
  - ubuntu trust -> xenial로 변경해야 /etc/chorny가 생성되고 bosh director의 ntp가 적용 된다.
  - 예) NTP 서버를 생성하여 NTP 서버는 한국 시간으로 -> bosh vm 들은 NTP로 동기화 (X)?

#### 2. 현재 2개의 데이터센터에서 IP 대역을 다름 vSphere SDN 할 경우 하나의 네트워크? 다른 데이터 센터의 네트워크 대역에서 사용이 가능하게 되는데 Application DR의 필요성?
  -PaaS-TA 플랫폼에서는 어떻게든 상관 없지만 Application에서의 설계 필요
  
#### 3. 내부 프린터 서버 어떻게 사용할 것인가 Docker Iamge로 Build하여 사용하도록 함

#### 4. BOSH와 Concourse Credhub의 버전 차이일 경우 버전에 대한 문제점을 어떻게 해결?
  - opensource와 pivotal concouse의 차이점은 라이센스를 통해 구매하면 같이 올라가는 방법으로 나옴
  - pivotal 라이센스 구매 후 업데이트를 할 경우 동시에 가능하도록 한다.
  
#### 5. bosh deploy yaml에 cpi_provider block, instance_groups block의 ntp 차이점은
  - Bosh VM의 NTP는 CPI Block에서 적용 Bosh가 설치 한 VM은 Director Block에서 적용 가능.

#### 6. pipeline으로 PCF 버전 업그레이드를 배포 시 일일이 config 파일에 대한 작성/어플리케이션을 재배포 & 테스트해야하는지,
권고 방법은 테스트 용으로 같은 환경을 똑같이 만들어서 배포 및 테스트를 해야한다.  
1. sendbox
2. dev
3. stg
4. prod 방식으로 4번 테스트?

#### 6. locale과 NTP를 맞춰 log 시간 같게 해야 한다.

#### 7. https:// ssl 공인인증서를 사용 할 때 apps-manager가 작동하지 않음, 사설인증서에는 가능한 Issue

#### 8. git hub가 유실되면 파이프라인이 사라지는지? Concourse 서버가 죽으면 파이프라인이 새로 구축 되는지
  - gp 후 받은 pipeline을 통해 복구가 가능
  - concourse 서버가 죽엇을 때 git hub를 통해서 복구 가능
  
#### 9. Ops Manager 버전 Upgrade시 바뀌는 계정 -> Ops Manager vm 삭제 후 업그레이드 해야 한다.

































