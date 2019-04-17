#### 1. PCF VM, App Log TimeZone 설정 문제
  - Bosh는 NTP 설정이 필요
  - VM은 Bosh Runtime config로 Release Post Script를 통해 수정 가능
  - APP은 CF Push 중 manifest env, jave env 설정 가능
  - ubuntu trust -> xenial로 변경해야 /etc/chorny가 생성되고 bosh director의 ntp가 적용 된다.

#### 2. 현재 2개의 데이터센터에서 IP 대역을 다름 vSphere SDN 할 경우 하나의 네트워크? 다른 데이터 센터의 네트워크 대역에서 사용이 가능하게 되는데 Application DR의 필요성?
  -PaaS-TA 플랫폼에서는 어떻게든 상관 없지만 Application에서의 설계 필요
  
#### 3. 
