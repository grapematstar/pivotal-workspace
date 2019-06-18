
#  Pivotal Cloud Foundry Operating TCP Routing

- Pivotal Cloud Foundry는 기본적으로 80/443 Domain 기반의 HTTP/HTTPS를 통하여 Application에 접근 한다. 하지만 TCP Routing을 Enable 시켜주면 다수의 TCP Port를 통하여 Application에 접근 할 수 있다.

- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- Ops Manager에 Pivotal Cloud Foundry Tile이 존재 해야 한다.
- Modify Last Version 2.5

## 1. Pivotal Cloud Foundry Router Architecture

- PAS Deployment Manifest의  Route_Registrar Property를 통하여 NATS Component를 통해  API, BlobStore, BBS, Cell, UAA, Log-Cache의 경로 Domain을 등록 한다.
- Diego BBS는 Silk Network Plugin을 통해 구성한 Application의 Back End 주소를 정하고 Cloud Controller의 Application Route 정보와 함께 Diego Cell Component의 Route_Emitter에 전달 한다.
- Route_Emitter는 전달 받은 Application의 경로가 Http/Https 일 경우 NATS Component를 통해 Application의 경로, IP, Port를 Go Router Component에 전달하고 TCP 일 경우 Routing API를 통하여 Application의 경로, IP, Port를 TCP Router Component에 전달한다.
- Go Router와 TCP Router는 Route_Emitter의 정보를 바탕으로 들어오는 Application 경로 요청을 Back End 주소에 Mapping 시킨다.

![router-architecture][tcp-router-image-1]

### 1.1. External Client Request 흐름

- External Client의 Application Route 요청을 받는다.
- CF CLI의 Domain의 Type, DNS Name Prefix를 통하여 HTTP, TCP Load Balancer로 요청을 보낸다.
- 이때 HTTP Load Balancer가 받는 Port는 80/443, TCP Load Balancer가 받는 Port는 사용 할 Application에 대한 모든 Port를 등록 해야 한다.
- 80/443 HTTP/HTTPS 요청은 Go Router로 1024 ~ 65535 TCP 요청은 TCP Router로 보내져 Mapping되어 있는 Application에 전달 된다.

![router-architecture][tcp-router-image-2]


## 2. Pivotal Cloud Foundry TCP Router 설정

- 기본적으로 Pivotal Cloud Foundry 배포 시 TCP Router 사용은 Disable 되어 있다.

### 2.1.  Pivotal Cloud Foundry Tile Config

#### 2.1.1. Pivotal Cloud Foundry Tile Networking Config
- TCP Router IPs: TCP Router의 IP 주소를 입력 한다.
- Enable TCP Routing을 선택하여 TCP Routing을 사용하고 1024 ~ 65535까지 사용 할 Port Range를 입력 한다.

#### 2.1.2. Pivotal Cloud Foundry Tile Resource Config
- TCP Router의 Instance 수를 증가 시킨다.

#### 2.1.3. Apply Change
- Ops Manager UI에서 Apply Change를 실행하여 변경 사항을 적용 한다.


### 2.2.  TCP Router 기본 사용 방법

- Apply Change가 완료 되면 아래 명령어를 통하여 TCP Router가 정상적으로 적용 됬는지 확인 한다.
```
$ cf curl /routing/v1/router_groups

# 결과 값
[
   {
      "guid": "d9175318-d332-492a-75c8-de5a9cf6c1aa",
      "name": "default-tcp",
      "type": "tcp", # tcp type 확인
      "reservable_ports": "1024-10000" # 적용한 port range 확인
   }
]
```

- TCP Router를 통하여 Application에 Bound 할 TCP Domain을 생성 한다.
```
  
$ cf router-groups

# 결과 값
Getting router groups as admin ...

name          type

default-tcp   tcp

$ cf create-shared-domain tcp.APPS-DOMAIN.com --router-group default-tcp
```

- TCP Domain을 통하여 Application 배포 방법
```
$ cf push my-app -p spring-music.jar -d tcp.example.com --random-route
```

- 해당 Router에 연동 한 Application의 Back End Port/IP 확인 방법
```
# uaac tcp_emitter client를 통하여 Token을 가져 온다.
$ uaac token client get tcp_emitter
Client secret:

# Token을 확인한다.
$ uaac context

$ curl api.MY-DOMAIN/routing/v1/tcp_routes -H "Authorization: bearer TOKEN"

# 결과 값
[{"router_group_guid":"f3518f7d-d8a0-4279-4e89-c058040d0000",   
"backend_port":60000,"backend_ip":"10.244.00.0","port":60000,"modification_tag":{"guid":"d4cc3bbe-c838-4857-7360-19f034440000",   
"index":1},"ttl":120}]
```

[tcp-router-image-1]:./images/tcp-router-1.png
[tcp-router-image-2]:./images/tcp-router-2.png
