
#  Pivotal Cloud Foundry Operating TCP Routing

- Pivotal Cloud Foundry는 기본적으로 80/443 Domain 기반의 HTTP/HTTPS를 통하여 Application에 접근 한다. 하지만 TCP Routing을 Enable 시켜주면 모든 TCP Port를 통하여 Application에 접근 할 수 있다.
- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- Ops Manager에 Pivotal Cloud Foundry Tile이 존재 해야 한다.
- Modify Last Version 2.5


[https://docs.pivotal.io/pivotalcf/2-5/opsguide/tcp-routing-ert-config.html](https://docs.pivotal.io/pivotalcf/2-5/opsguide/tcp-routing-ert-config.html)

[https://docs.pivotal.io/pivotalcf/2-4/adminguide/enabling-tcp-routing.html](https://docs.pivotal.io/pivotalcf/2-4/adminguide/enabling-tcp-routing.html)

[https://docs.pivotal.io/pivotalcf/2-5/concepts/cf-routing-architecture.html](https://docs.pivotal.io/pivotalcf/2-5/concepts/cf-routing-architecture.html)
