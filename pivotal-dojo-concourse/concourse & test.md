#### 1. concourse(pipeline) & minio(s3) & gitlab(source reposistory) & harbor(docker repository) 사용 Platform Pipeline
  
  concource 
  
  1.1 Platform 실습 vm 복구 용 git hub 생성 jumpbox-bosh, minio, harbor
  1.2 https://bosh.io/releases/github.com/concourse/concourse-bosh-release?all=1 (concourse 설치)
      stemcell/release/manifest external download -> internal scp
      concourse deployment bosh manifest https://bosh.io/d/github.com/concourse/concourse-bosh-release?v=5.1.0
      
  issue 1: credhub <-> uaa 연결 시 도메인을 사용하면 dns를 구성해주지 않으면 알수 없는 에러가 발생한다.
    log: [2019-04-18 07:15:41+0000] Could not reach the UAA server
  
  issue 2: bosh의 credhub와 무관하게 추가적으로 deployment에 credhub를 연결 할 경우 버전 차이로 인하여 에러가 발생 할 수 있다.
  bosh와 설치 할 deployment가 사용 할 credhub release의 버전을 가게 해줘야 할 수 있다.
 
  pivotal automation: Coucourse와 Ops Manager를 사용하려면 om이라는 API VM이 필요 함
  
  apply-changes: env vcenter 정보와 Opsmager 정보를 바탕으로 pas/director를 설치 (git에서 config 파일을 저장해놔야함 concourse get 하기 위해)
  
  download-product: stemcell과 release 등을 외부 pivonet 또는 내부 s3에서 땡겨서 사용하기 위함
  
  docs: http://docs.pivotal.io/platform-automation/v2.1/index.html
  offline installation concept: https://docs.pivotal.io/pivotalcf/2-4/customizing/offline_installation.html
  
  issue: concouse 서버의 hosts, resolv.conf에 dns를 넣어도 worker vm 위의 컨테이너 영역에서는 등록이 안된다.
  
  
