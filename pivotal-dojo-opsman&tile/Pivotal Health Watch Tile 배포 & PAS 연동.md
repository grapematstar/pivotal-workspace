##  Pivotal Health Watch Tile 배포 & PAS 연동

- Pivotal Health Watch Tile은 Pivotal Health Watch를 설치하는 Tile으로 Pivotal Health Watch 설치에 대한 전반적인 Domain, Network, Component, Errand 등의 Config 정보를 설정 할 수있다.
- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- PAS가 구축되어 있어야 한다.
	- Ops Manager에 Pivotal Health Watch Tile이 존재 해야 한다.
- Modify Last Version 1.4

### 1. Pivotal Health Watch 소개

- PCF Healthwatch는 Bosh Director와 Pivotal Cloud Foundry의 현재 상태, 성능 및 용량을 시각화하여 Monitoring하고 Event Alert와 연동하여 임계치 한계시 사용자에서 E Mail Alarm을 제공하는 Service

#### 1.1. Pivotal Health Watch Architecture

- Pivotal Health Watch의 Ingestor Application이 Pivotal Cloud Foundry Loggregator Firehose의 Metric 정보를 수집 한다.
- Ingestor Application은 Metric 정보를 Redis에 전달하고 Woker Application이 Redis를 사용하여 Data를 집계하고 변환하여 Mysql에 저장 한다.
- Mysql에 변환된 데이터는 Pivotal Health UI에서 표기되거나 Aggregator App을 통해 추가로 변환하여 외부 Pivotal Cloud Foundry Firehose를 통하여 출력된다.

[Pivotal Health Watch Architecture][https://docs.pivotal.io/pcf-healthwatch/1-4/architecture.html](https://docs.pivotal.io/pcf-healthwatch/1-4/architecture.html)


### 2. Pivotal Health Watch Deploy

#### 2.1. Pivotal Health Watch Tile 다운로드 & 업로드
- Pivotal Network에 접속하여 Pivotal Health Watch Tile를 다운로드 한다.
- 다운로드 한 Pivotal Health Watch Tile를 Ops Manager에 Import 한다.

#### 2.1. Pivotal Health Watch Tile Config

##### 2.1.1. Assign AZs and Networks
 
- Pivotal Cloud Foundry를 배포 할 때 Singleton으로 구성 할 VM(BlobStore)과 여러 다중으로 구성하는 AZ를 선택 한다.
- Bosh Director Tile에서 설정한 Network를 선택 한다.
- AZ 또는 Network를 증설 할 경우 Bosh Director Tile의 정보를 수정하고 Apply Change 해야한다.
