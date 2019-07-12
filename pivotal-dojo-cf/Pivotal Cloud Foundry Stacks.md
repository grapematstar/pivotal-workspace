
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

- Pivotal Cloud Foundry에서 Application Push를 하게 되면 Stack의 directory 구조와 Buidpack을 사용하게 된다. Stack은 






