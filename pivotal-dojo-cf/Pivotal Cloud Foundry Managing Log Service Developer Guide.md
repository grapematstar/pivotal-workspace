
#  Pivotal Cloud Foundry Managing Log Service Developer Guide

- Pivotal Cloud Foundry의 Loggregator Component에서 외부 Log Service Instance로 Application Log를 Streaming하는 방법에 대해 기술한다.

- Modify Last Version 2.5

## 1. Streaming App Logs to Log Management Services
- Pivotal Cloud Foundry는 내부 Loggregator Component를 사용하여 모든 Application에 대한 요청과 Instance의 Log를 기록한다.
- 아래 명령어를 통해 Application의 Log를 Streaming한다.
```
$ cf logs YOUR-APP-NAME
Retrieving logs for app aaa in org system / space dev as admin...

   2019-06-27T17:39:06.31+0900 [API/0] OUT Updated app with guid 87e439bb-ebbd-4f3c-a3d1-2937bf12ea48 ({"buildpack"=>"timezone_buildpack", "disk_quota"=>1024, "health_check_http_endpoint"=>"", "health_check_type"=>"port", "instances"=>3, "memory"=>768, "name"=>"aaa", "space_guid"=>"f2bccb67-2a32-4053-8361-3df405d49eb2"})
   2019-06-27T17:39:06.75+0900 [API/0] OUT Uploading bits for app with guid 87e439bb-ebbd-4f3c-a3d1-2937bf12ea48
   2019-06-27T17:39:13.21+0900 [API/0] OUT Creating build for app with guid 87e439bb-ebbd-4f3c-a3d1-2937bf12ea48
   2019-06-27T17:39:13.29+0900 [API/0] OUT Updated app with guid 87e439bb-ebbd-4f3c-a3d1-2937bf12ea48 ({"state"=>"STARTED"})
   2019-06-27T17:39:13.33+0900 [STG/0] OUT Downloading timezone_buildpack...
   2019-06-27T17:39:13.33+0900 [STG/0] OUT Downloaded timezone_buildpack
   2019-06-27T17:39:13.33+0900 [STG/0] OUT Cell ec47e5b9-9ec5-4be0-8e05-2b01341ef7c4 creating container for instance e5c9aece-4fe4-4740-a366-ece85b123a84
   2019-06-27T17:39:13.62+0900 [STG/0] OUT Cell ec47e5b9-9ec5-4be0-8e05-2b01341ef7c4 successfully created container for instance e5c9aece-4fe4-4740-a366-ece85b123a84
   2019-06-27T17:39:14.08+0900 [STG/0] OUT Downloading app package...
   2019-06-27T17:39:15.65+0900 [STG/0] OUT Downloaded app package (42.3M)
   2019-06-27T17:39:16.93+0900 [STG/0] OUT -----> Java Buildpack 542d66c (offline) | https://github.com/cloudfoundry/java-buildpack.git#542d66c
   2019-06-27T17:39:16.99+0900 [STG/0] OUT -----> Downloading Jvmkill Agent 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/bionic/x86_64/jvmkill-1.16.0-RELEASE.so (found in cache)
   2019-06-27T17:39:16.99+0900 [STG/0] OUT -----> Downloading Open Jdk JRE 1.8.0_212 from https://java-buildpack.cloudfoundry.org/openjdk/bionic/x86_64/openjdk-jre-1.8.0_212-bionic.tar.gz (found in cache)
   2019-06-27T17:39:18.09+0900 [STG/0] OUT        Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
   2019-06-27T17:39:18.09+0900 [STG/0] OUT        JVM DNS caching disabled in lieu of BOSH DNS caching
   2019-06-27T17:39:18.09+0900 [STG/0] OUT -----> Downloading Open JDK Like Memory Calculator 3.13.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/bionic/x86_64/memory-calculator-3.13.0-RELEASE.tar.gz (found in cache)
   2019-06-27T17:39:19.07+0900 [STG/0] OUT        Loaded Classes: 18651, Threads: 250
   2019-06-27T17:39:19.09+0900 [STG/0] OUT -----> Downloading Client Certificate Mapper 1.8.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.8.0-RELEASE.jar (found in cache)
   2019-06-27T17:39:19.10+0900 [STG/0] OUT -----> Downloading Container Security Provider 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.16.0-RELEASE.jar (found in cache)
   2019-06-27T17:39:19.10+0900 [STG/0] OUT -----> Downloading Spring Auto Reconfiguration 2.7.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-2.7.0-RELEASE.jar (found in cache)
.... 생략
```

## 2.  Using Services from the Cloud Foundry Marketplace
- Pivotal Cloud Foundry Market Place를 이용하여 Service를 연동 할 경우 일반적인 Application Bind 명령어를 사용하여 연동 한다.
- 일반적인 Application Bind를 사용하기 위해선 Syslog를 통해 자동화 된 drain 기능을 구현하거나 존재하고 있는 Service를 배포해야 한다.

```
$ cf create-service SERVICE PLAN SERVICE-INSTANCE
$ cf bind-service YOUR-APP YOUR-LOG-STORE
```

## 3. Using Services Not Available in Your Marketplace
- Pivotal Cloud Foundry가 관리하는 Service가 아닐 경우 사용자 제공 Service를 생성하여 Application의 Log를 Streaming 할 수 있다.
- CF Drain Plugin을 사용하여 cf CLI를 통해 Application의 Log를 Streaming 한다.

### 3.1. Cf Drain 설치 방법
```
$ cf install-plugin -r CF-Community "drains"
$ cf install-plugin download/path/cf-drain-cli
```

### 3.2. 사용자 제공 Log Service 사용 방법

#### 3.2.1. Application Outbound 설정
- Application에 Service를 Bind 하여 Log를 보내기 위해서 External Service의 IP에 외부 통신이 가능한지 확인한다.
- syslog, syslog-tls 또는 https의 구성을 갖은 URL을 생성 한다.
	- example: 1.  syslog://logs.example.com:1234

#### 3.2.2. Create and Bind a User-Provided Service Instance
- Cf Drain을 통해 조직 또는 Application에 Syslog 연동
```
# Single App
$ cf drain APP-NAME SYSLOG-DRAIN-URL
# All Space App
$ cf drain-space --drain-name DRAIN-NAME --drain-url SYSLOG-DRAIN-URL --username USERNAME
```

- Service Instance를 통하여 연동
```
$ cf create-user-provided-service DRAIN-NAME -l SYSLOG-URL
$ cf bind-service YOUR-APP-NAME DRAIN-NAME
```



