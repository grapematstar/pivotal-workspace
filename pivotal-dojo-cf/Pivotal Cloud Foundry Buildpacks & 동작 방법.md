
#  Pivotal Cloud Foundry Buildpacks & 동작 방법
 
- Pivotal Cloud Foundry 상에서 Application Runtime & Framework를 구성하는 Buildpack과 Pivotal Cloud Foundry에서의 동작 방법을 기술한다.
- Modify Last Version 2.6

## 1. Buildpacks
- Pivotal Cloud Foundry는 Application에 대한 Framework와 Runtime 환경을 제공 한다.
- Application을 Push하게 되면 자동적으로 Pivotal Cloud Foundry가 Buildpack을 감지하여 Dependency 다운로드 & Application을 실행 한다.
- Pivotal Cloud Foundry에서 제공하는 Default Buildpack의 종류는 아래와 같다.
	- staticfile_buildpack
	- java_buildpack
	- php_buildpack
	- go_buildpack
	- nodejs_buildpack
	- ruby_buildpack
	- dotnet_core_buildpack
	- binary_buildpack
	- python_buildpack

- Pivotal Cloud Foundry에서 플랫폼 관리자에게 추가적으로 제공하는 Buildpack의 종류는 아래와 같다.
	- nginx_buildpack
	- R_buildpack
	- HWC(ASP .NET)_buildpack

## 2. Buildpack & Pivotal Cloud Foundry의 동작
- Buildpack에는 bin 디렉토리 아래에 5가지의 script를 포함 할 수 있다. 해당 script는 buildpack의 종류 별 다를 수 있다.
	- bin/detect
	- bin/supply
	- bin/finalize
	- bin/release
	- bin/compile

### 2.1. bin/detect
- detect Script는 Application에 Buildpack을 적용 할지에 대한 여부를 결정 한다, $ cf push 중 buildpack 사용에 대한 정보를 입력하지 않았을 경우 실행한다.
- Buildpack이 Application과 호환이 되면 Status Code 0을 출력하고, Buildpack의 name/version 등 기타 정보를 출력 한다.

```
#!/usr/bin/env ruby

gemfile_path = File.join ARGV[0], "Gemfile"

if File.exist?(gemfile_path)
  puts "Ruby"
  exit 0
else
  exit 1
end
```

### 2.2. bin/supply
- Application에 대한 ruby, bundle, nginx 실행 파일 등을 설치 하고, 특정 Directory에 위치 시키며 Buildpack에 대한 처리를 직접적으로 실행 시킨다.
- 개발자가 cf push를 할 경우 Buildpack이 실행하는 명령줄을 보여준다.
- supply script는 아래의 4가지의 Argument를 사용하여 Buildpack을 실행한다.
	- build: Application의 Director로써 conf, build 대상 Package 들이 들어가 있는 Directory
	- cache: Buildpack 실행 Process 중 캐시를 저장하는 Directory
	- deps: Buildpack이 제공하는 Dependency가 들어가는 공간
	- index: Buildpack의 Index로써 0번 부터 시작하는 Directory

```
#!/usr/bin/env ruby

#sync output

$stdout.sync = true

build_path = ARGV[0]
cache_path = ARGV[1]
deps_path = ARGV[2]
index = ARGV[3]

install_ruby

private

def install_ruby
  puts "Installing Ruby"

  # !!! build tasks go here !!!
  # download ruby 
  # install ruby
end
```

### 2.3. bin/finalize
- Buildpack 동작의 마지막에 실행  되고 Buidlpack의 Dependency를 다운로드하여 Application 실행을 준비한다.
- finalize script는 개발자가 cf push 중 실행 결과를 명령줄에 제공 한다.
- finalize script는 아래의 4가지의 Argument를 사용하여 Buildpack을 실행한다.
	- build: Application의 Director로써 conf, build 대상 Package 들이 들어가 있는 Directory
	- cache: Buildpack 실행 Process 중 캐시를 저장하는 Directory
	- deps: Buildpack이 제공하는 Dependency가 들어가는 공간
	- index: Buildpack의 Index로써 0번 부터 시작하는 Directory
```
#!/usr/bin/env ruby

#sync output

$stdout.sync = true

build_path = ARGV[0]
cache_path = ARGV[1]
deps_path = ARGV[2]
index = ARGV[3]

setup_ruby

private

def setup_ruby
  puts "Configuring your app to use Ruby"

  # !!! build tasks go here !!!
  # setup ruby 
end
```
### 2.4. bin/compile (Deprecated)
- build directory를 사용하여 종속성을 저장하였던 script 지만 해당 기능이 supply script로 넘어가면서 더 이상 사용 하지 않음.

### 2.5. bin/release
- release는 Buildpack 동작이 완료 후 완성한 실행 정보, Conf 정보 들을 바탕으로 Build Directory에서 실행하는 Application 동작 Script


## 3. Droplet Filesystem
- 개발자의 Cf Push 명령어롤 통해 Application을 배포하게 되면 Buildpack Staging 단계에서 Container 내부의 /home/vcap Directory에 아래와 같은 구조를 생성한다.
```
app/
deps/
logs/
tmp/
staging_info.yml
```
- app Directory는 Buildpack의 build Directory를 포함한다.
- staging_info.yml은 Droplet에 저장된 Application의 Stating metadata를 포함한다.

## 4. Buildpack 검색
- Pivotal CLoud Foundry는 detection script를 사용하여 Application에 적합한 Buildpack을 검색하여 사용 한다.
- Buildpack의 검색 순서는 $ cf bulidpack 명령줄로 검색하였을때의 Index 순서이며, 1번 Index가 Application 배포에 적합 하지 않을 경우 2번 Index로 넘어간다.
- 만약 검색한 Builpack이 모두 적합 하지 않는다면 아래와 같은 에러를 발생한다.
```
None of the buildpacks detected a compatible application
Exit status 222
Staging failed: Exited with status 222

FAILED
NoAppDetectedError
```

## 5. Builpack과 Pivotal Cloud Foundry의 상호 작용 순서

### 5.1. Pivotal Cloud Foundry가 Buildpack을 검색 하지 않을 경우
- 개발자가 $ cf push 명령어를 통해 Application을 배포 할 떄-b 옵션을 사용, manifest.yml에 Buildpack을 명시 할 경우 Pivotal Cloud Foundry는 별도의 다른 Buildpack을 검색하지 않는다.
- Pivotal Cloud Foundry는 아래와 같은 절차를 수행 한다.
	- deps/index directory를 생성 한다.
	- builds, cache, deps를 파라미터로 사용하여 /bin/supply script를 실행 한다.
	- Buildpack이 사용하는 index의 Directory만을 사용하도록 설정 한다.
	- /bin/finalize script가 존재 하는 경우
		- build, cache, deps directory를 사용하여 finalize script를 실행한다.
	- /bin/finalize script가 존재 하지 않을 경우
		- build, cache directory를 사용하여 compile script를 실행한다.
	- /bin/release를 실행하여 conf, env 환경을 바탕으로 Application을 실행 시킨다.

### 5.1. Pivotal Cloud Foundry가 Buildpack을 검색 할 경우
- 개발자가 $ cf push 명령어를 통해 Application을 배포 할 떄-b 옵션을 사용하지 않음, manifest.yml에 Buildpack을 명시 하지 않았을 경우 Pivotal Cloud Foundry는 Buildpack 시퀀스를 통해 Application에 적합한 Buildpack을 검색 한다.
- Pivotal Cloud Foundry는 아래와 같은 절차를 수행 한다.
	- bin/detect script를 실행시켜 Buildpack의 시퀀스 순서로 detect script 결과가 0일 경우의 Buildpack을 채택한다.
	- /bin/finalize script가 존재 할 경우
		- /bin/supply script가 존재 하는 지 확인하고 존재한다면 build, cache, deps directory를 기준으로 해당  script를 실행 시킨다. 
		- build, cache, deps directory를 사용하여 finalize script를 실행한다.
	- /bin/finalize script가 존재 하지 않을 경우
		- build, cache directory를 사용하여 compile script를 실행한다.
- /bin/release를 실행하여 conf, env 환경을 바탕으로 Application을 실행 시킨다.


