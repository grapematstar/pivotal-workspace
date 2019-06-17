# Pivotal Cloud Foundry Private Repo Docker Push

## 1. Director, PAS Docker Image 사용 설정

### 1.1 OpsMan Director에 Docker Image 사용 설정 방법
```
Opsmanager 화면에서 Director 설정 -> Security에 대해 Private Docker Repo에 대한 신뢰하는 인증서를 추가
Private Docker Repo 인증서 위치는 harbor 화면의 Administration/Configuration 화면에서 Registry Root Certificate를 다운로드 할수 있다.

# 신뢰하는 인증서를 추가하지 않으면 $ cf push 중 509 SSL 관련 에러가 발생한다.
```
### 1.2 OpsMan PAS에 Docker Image 사용 설정 방법
```
Opsmanager 화면에서 Application Container 화면의 Private Docker Insecure Registry Whitelist에 Docker Repo의 Whitelist IP:PORT, HOSTNAME:PORT를 지정해준다.

# 해당 Whitelist를 추가 하지 않으면 $ cf push 중 insecure 관련 에러가 발생한다.
# 해당 Whitelist에 :PORT를 지정하지 않으면 $ cf push 중 default port 80을 물고들어가 harbor를 찾을수 없는 에러가 발생한다.
```
### 1.3 Diego cell에 Docker flag 사용 설정
```
 docker image를 사용하기 위해서는 cf push가 Docker container를 지원하는지에 대한 flag를 설정
 $ cf enable-feature-flag diego_docker
 
 # 해당 flag가 disable되어 있으면 $ cf push 중 diego_docker 권한에 대해 에러가 발생한다.
```
## 2. Docker Image Push

### 2.1 docker image build & tag & push
```
Docker Image 사용 설정이 완료되면 사용할 Docker Image에 대해 Build 하고 Pirvate Docker Repo에 대해 Tag를 지정하여 Docker Repo에 push 한다.

$ docker build --no-cache -t {$image_name:$image_version}
$ docker tag ${image_name:$image_version} ${docker_repo_image_name:docker_repo_image_version}
$ docker push ${docker_repo_image_name:docker_repo_image_version}
```
### 2.2 CF Push Docker Image
```
Private Docker Repo에 Image가 정상적으로 올라갔으면 해당 Image의 Pull 주소를 복사하고 cf push 한다.
$ $ CF_DOCKER_PASSWORD=YOUR-PASSWORD cf push APP-NAME --docker-image REPO/IMAGE:TAG --docker-username USER

# CF_DOCKER_PASSWORD는 별도의 cf push cli의 option이 없어 앞에 환경변수 형태로 사용한다.
# CF_DOCKER_PASSWORD를 지정하지 않으면 harbor에 대한 권한 에러가 발생한다.
```
## 3. Docker & PAS
```
$ cf push를 하게 되면 별도의 apps.YOUR_DOMAIN.com 형식으로 기본 80/443의 도메인으로 접근이 가능하다.
하지만 Docker APP이 사용하는 Port가 80/443이 아닌 다른 Port를 사용하게 되면 어플리케이션을 통해 접근 시 502 bad gateway error가 발생하게 된다.
502 bad gateway error가 발생하는 이유는 Docker Image를 Build 할 때 DockerFile에 expose
를 통해 Port에 대한 노출이 없어서 에러가 발생

expose port를 docker image에 할당하면 cf domain으로 접근 시 해당 expose port가 동적으로 할당되어 접근한다고 한다.

# 확인사항
Docker Image에 다른 Application이 2개의 Port를 가지고 Web 또는 Rest 통신을 할 경우 TCP Router를 통해 사용이 가능한지에 대한 검증이 필요하다.
```
