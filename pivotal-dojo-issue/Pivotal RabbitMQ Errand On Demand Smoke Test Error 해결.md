# Pivotal RabbitMQ Errand On Demand Smoke Test Error 해결
- 증상: Ops Manager에서 Rabbit MQ를 배포 했을 때 Install 단계는 정상적으로 넘어가지만 Running errand On Demand Instance Smoke Tests for RabbitMQ에서 에러 발생

## 1. 에러 내용
- Running errand On Demand Instance Smoke Tests for RabbitMQ Error, 등록한 RabbitMQ의 Ondemand Service Broker를 호출하여 Service Instance 생성을 호출하여 VM을 생성 할 떄 에러가 발생
- 에러 내용 1. Ops Manager Change Log
```
 status:    create failed  
          message:   Instance provisioning failed: There was a problem completing your request. Please contact your operations team providing the following information: service: p.rabbitmq, service-instance-guid: a4151730-a590-424c-8cf7-fcf47e1a3c09, broker-request-id: 90fc4208-7d22-4c43-b468-3a1c61dfd93f, task-id: 58613, operation: create  
          started:   2019-08-01T03:43:16Z  
          updated:   2019-08-01T03:48:28Z  
            
          There are no bound apps for this service.  
```
- 에러 내용 2. Bosh Task 로그
```
Task 58613 | 03:43:28 | Creating missing vms: rabbitmq-server/ff92ad78-b357-4ab1-8924-71731b91528b (0) (00:01:33)
Task 58613 | 03:45:03 | Updating instance rabbitmq-server: rabbitmq-server/ff92ad78-b357-4ab1-8924-71731b91528b (0) (canary) (00:02:56)
Task 58613 | 03:48:08 | Error: Action Failed get_task: Task 7bd04f3b-e186-4e45-72a2-5b29e9d7445d result: 1 of 1 post-deploy scripts failed. Failed Jobs: rabbitmq-server.
```

- 에러 내용 3. RabbitMQ VM Job Log
```
does not have the correct permissions for vhost
```

## 2. 에러 대처
### 2.1. vShpere의 Cluster Spce 확인 - 결과: Fail
- Rabbit MQ 설치 대상 vSphere Cluster의 Memory가 거의 꽉 차  on-demand-broker의 AZ를 변경하여 설치 진행

### 2.2. Broker의 Timeout 설정 - 결과: Fail
- on-demand-broker의 Broker Config에서 Timeout을 600초로 설정하여 진행

### 2.3. Bosh Director Tile Config의 Post Deploy Script 속성 비활성화 결과: Success

- Post Deploy 설명 문서 링크: https://bosh.io/docs/post-deploy/
- Post Deploy가 실행 될 때 아래와 같은 에러가 발생, failed의 검출로 실패한 것으로 추정 됨.

```
mesg: ttyname failed: Inappropriate ioctl for device
```



