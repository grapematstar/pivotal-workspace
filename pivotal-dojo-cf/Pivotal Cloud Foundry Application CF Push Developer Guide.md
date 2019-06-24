#  Pivotal Cloud Foundry Application CF Push Developer Guide

- cf CLI를 사용하여 Pivotal Cloud Foundry에 개발자 Application을 배포하는 방법에 대해 설명 한다.

- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Pivotal Cloud Foundry가 구축 되어 있어야 한다.
	- cf CLI가 설치 되어 있어 있고 Pivotal Cloud Foundry Target이 잡혀있어야 한다.
- Modify Last Version 2.5

## 1. Default CF Push 

```
$ cf push APP-NAME -p SPRING-BOOT.jar
```
- Application 명은 PAS 환경 내에 Unique 해야 하고, 만약 Application 명을 동일 하게 할 경우 Route를 변경해야 한다. 
- Application 명에 _를 사용할 경우 자동으로 -으로 변경되어 Push 된다.

## 2. Default CF Route

- Application Route는 URL이 실행되는 경로이다. PAS는 Application의 hostname과 domain에서 Application의 접속 경로를 생성 한다.
- 예를 들어 apps.example.com이라는 App Domain으로 PAS에서 실행되는 my-app-1234라는 App은 기본적으로 URL https://my-app-1234.apps.example.com에서 실행 된다.


## 3. Example CF Push
```
 $ cf push my-app-1234 -p SPRING-BOOT.jar
    Creating app my-app-1234 in org example-org / space development as a.user@shared-domain.example.com...
    OK # development space에 user로 인하여 App Instance가 생성 된다.

    Creating route my-app-1234.shared-domain.example.com... 
    OK # Route가 생성 된다.

    Binding my-app-1234.shared-domain.example.com to my-app-1234...
    OK # Route와 도메인이 Bind 된다.

    Uploading my-app-1234...
    Uploading app: 560.1K, 9 files
    OK Application의 소스 코드가 Diego Brain Component에 업로드 된다.

    Starting app my-app-1234 in org example-org / space development as a.user@shared-domain.example.com...
    -----> Downloaded app package (552K)
    OK
    -----> Using Ruby version: ruby-1.9.3
    -----> Installing dependencies using Bundler version 1.3.2 # Buildpack Package를 설치 한다.
           Running: bundle install --without development:test --path
             vendor/bundle --binstubs vendor/bundle/bin --deployment
           Installing rack (1.5.1)
           Installing rack-protection (1.3.2)
           Installing tilt (1.3.3)
           Installing sinatra (1.3.4)
           Using bundler (1.3.2)
           Updating files in vendor/cache
           Your bundle is complete! It was installed into ./vendor/bundle
           Cleaning up the bundler cache.
    -----> Uploading droplet (23M) # Droplet 정보를 Diego Cell에 업로드하고 Garden Container를 실행 시킨다.

    1 of 1 instances running # Instance running 상태를 확인 한다.
    App started

    Showing health and status for app my-app-1234 in org example-org / space development as a.user@shared-domain.example.com...
    OK

    requested state: started
    instances: 1/1
    usage: 1G x 1 instances
    urls: my-app-1234.shared-domain.example.com

         state     since                    cpu    memory        disk
    #0   running   2018-06-24 05:07:18 PM   0.0%   18.5M of 1G   52.5M of 1G
```

## 4. Customize Basic App Settings

- Name: 일련의 영숫자를 앱 이름으로 사용할 수 있다.
- Instance: Application의 배포하는 Instance수를 조정 할 수 있다. 여러개를 지정 할 경우 Update 시 Downtime이 줄어 든다, Pivotal의 권고 Instance 수는 2개 이상 이다.
- Memory Limit: Application이 사용 할 수 있는 Memory의 제한 값을 설정 할 수 있다, 일정 시간 동안 해당 Memory를 초과 할 경우 PAS는 자동으로 해당 Application을 재 시작한다.
- Start Command: PAS가 인스턴스를 시작 하기 전에 실행 하는 명령으로 Buildpack의 Framework별로 다른 명령어를 실행 한다. 

## 5. Customize the Route

- 사용자 정의의 host name을 사용 할 경우 $ cf push -n {host 명}을 사용 한다.
- 사용자 정의의 domain을 사용 할 겨웅 $ cf push -d {domain 명}을 사용 한다.
- Application 접속 Route의 고유성을 보장하려면 cf push --random-route를 사용 한다.

## 6. Limit the Upload Files

- 기본적으로 PAS는 Application 소스 코드 중 .git, .svn 등과 같은 Version 관리 파일을 제외한 모든 파일을 PAS에 업로드 한다. Application의 Size가 클 경우 .cfignore 파일을 통하여 특정 파일에 대한 업로드를 하지 않고 Application을 실행 할 수 있다.
- .cfignore의 파일 위치는 $ cf push가 실행되는 위치에 저장되어 있어야 한다.

## 7. Configure App 초기화
- 해당 내용은 Java Buildpack은 지원하지 않는다.
- $ cf push를 통해 구성한 Application에 대한 사용자 정의의 Script를 실행 할 수 있다. $ cf push가 실행되는 경로에서 .profile 파일을 생성하고 Script를 구성 한다. 해당 Script는 Application이 동작하기 전에 실행 된다.

## 8. Custom Push the App
- 




