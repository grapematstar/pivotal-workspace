#  Pivotal Cloud Foundry Stacks
 
- Pivotal Cloud Foundry 상에서 가동하는 Application의 File System의 Stack에 대한 설명을 기술 하였다.
- Modify Last Version 2.6

## 1. Stacks
- Release(cf-linuxfs3.tgz 등) 형태로 Diego Cell VM에 배포가 되며 실제 Application 배포 시 사용하는 File System을 구축하는데 사용한다.
- Stack Package의 위치는 Diego Cell VM /var/vcap/packages 아래 경로에 위치 한다.
- File System의 예시로 Ubuntu일 경우 /etc /lib /bin /tmp /root /run /dev /sys 등의 구조를 지니고 있다.
- Pivotal Cloud Foundry에서 제공하는 Stack의 종류는 아래와 같다.
	- cflinuxfs3
	- windowsfs

- Pivotal Cloud Foundry에서 Application Push를 하게 되면 Stack의 directory 구조와 Buidpack을 사용하게 된다. Stack은 Buildpack에 종속되며 Buildpack을 생성 할 때 Stack을 지정하게 된다.
- Buildpack은 같은 Name은 가질 수 있지만 같은 Name과 Stack은 가질 수 없다.

```
$ cf buildpacks
Getting buildpacks...

buildpack                position   enabled   locked   filename                                             stack
staticfile_buildpack     1          true      false    staticfile_buildpack-cached-cflinuxfs2-v1.4.29.zip   cflinuxfs2
java_buildpack_offline   2          true      false    java-buildpack-offline-cflinuxfs2-v4.12.1.zip        cflinuxfs2
ruby_buildpack           3          true      false    ruby_buildpack-cached-cflinuxfs2-v1.7.21.zip         cflinuxfs2
. . .
```

## 2. Stack 사용 방법

### 2.1. Stack이 존재 하지 않는 Buildpack이 있을 경우

- Buildpack을 생성 시 Stack을 명시하지 않을 경우 $ cf Buildpack 명령어 출력 시 빈값으로 나타나게 되는데 자동으로 Pivotal Cloud Foundry는 cflinuxfs3 stack을 사용하게 된다.
- Buildpack에 Stack을 넣기 위해서는 아래 명령어를 실행 한다.
```
$ cf stacks
Getting stacks in org system / space system as admin...
OK

name            description
cflinuxfs2      Cloud Foundry Linux-based filesystem - Ubuntu Trusty 14.04 LTS
cflinuxfs3      Cloud Foundry Linux-based filesystem - Ubuntu Bionic 18.04 LTS
windows2012R2   Microsoft Windows / .Net 64 bit
windows2016     Microsoft Windows 2016
windows         Windows Server

$ cf update-buildpack BUILDPACK-NAME --assign-stack STACK_NAME
```

### 2.2. Application에 Bind한 Stack을 변경하는 방법
- Security를 위해 Pivotal Cloud Foundry 버전이 올라 갈 때 Stack의 Version도 같이 올라가게 된다, 현재 Pivotal Cloud Foundry 2.6의 Stack File System의 버전은 Ubuntu Bionic 18.04 LTS 이다.
- 일부 Python, Ruby, C의 Application은 다시 패키징을 해야 할 수 있다.

```
$ cf push MY-APP -s cflinuxfs3
        Using stack cflinuxfs3...
        OK
        Creating app MY-APP in org MY-ORG / space development as developer@example.com...
        OK
        ...
        requested state: started
        instances: 1/1
        usage: 1G x 1 instances
        urls: MY-APP.cfapps.io
        last uploaded: Wed Apr 8 23:40:57 UTC 2015
            state    since                    cpu    memory        disk
        #0  running  2015-04-08 04:41:54 PM   0.0%   57.3M of 1G   128.8M of 1G
```

### 2.3. Application에 Bind한 Stack을 Plugin을 통해 변경하는 방법

#### 2.3.1. pivotal Stack plugin 설치
- pivotal network에서 제공하는 stack plugin을 설치 한다.  [https://network.pivotal.io/products/buildpack-extensions/](https://network.pivotal.io/products/buildpack-extensions/)
```
$ tar xvzf PATH-TO-BINARY
$ cf install-plugin PATH-TO-BINARY
```
- Application이 사용하는 Stack 목록을 확인한다.
```
$ cf audit-stack

$ cf audit-stack
    first-org/development/first-app cflinuxfs2 
    first-org/staging/first-app cflinuxfs2
    first-org/production/first-app cflinuxfs2
    second-org/development/second-app cflinuxfs3
    second-org/staging/second-app cflinuxfs3
    second-org/production/second-app cflinuxfs3
    ...
```
- Application의 소스코드 변경 없이 Stack을 변경 한다.
- 변경 도중 조금의 Downtime이 발생하며 Blue-Green(Route) 방식을 사용하여 Application의  Downtime을 방지한다.
```
$ cf change-stack my-app cflinuxfs3
Attempting to change stack to cflinuxfs3 for my-app...
Starting app my-app in org pivotal-pubtools / space pivotalcf-staging as ljarzynski@pivotal.io...
Downloading staticfile_buildpack...


requested state: started
instances: 1/1
usage: 64M x 1 instance
urls: example.com
last uploaded: Thu Mar 28 17:44:46 UTC 2019
stack: cflinuxfs3
buildpack: staticfile_buildpack

     state     since                    cpu    memory        disk         details
#0   running   2019-04-02 03:18:57 PM   0.0%   8.2M of 64M   6.9M of 1G

Application my-app was successfully changed to Stack cflinuxfs3
```

## 3. Trusted System Certificates

- 인증서는 Diego Cell에서 실행되는 Linux 기반 Application에서 사용할 수 있다. Application에는 cflinuxfs3 Stack을 사용하는 buildpack 기반 Application의 /etc/ssl/certs 디렉토리에 Certificates가 추가 되고 Application 환경 변수 CF_SYSTEM_CERT_PATH를 통해 값을 확인 할 수 있다.

