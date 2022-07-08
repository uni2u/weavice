# weavice (weave service)
weavice [wi:vis] is a Cloud-Native network service.

## Cloud-Native Security Function Chaining (CNSFC by [TSNLab](https://www.tsnlab.com/home))
CNSFC는 Cloud-Native 환경에서 생성되는 마이크로서비스간 안전한 네트워킹을 위해 보안 서비스를 동적으로 연결한다.
SF (Security Function) 는 네트워킹 보안 서비스를 제공하는 서비스로서 Firewall, IPS, IDS 등을 의미하며 process, VM, container 등으로 제공된다.
SF 는 네트워킹중인 microservice 사이에 유연하게 적용될 수 있도록하며 이러한 모양을 Chaining 이라고 정의한다.
본 프로젝트에서 정의하는 SFC 는 기존 [Service Function Chaining](https://cloudify.co/blog/cloudify-orchestrates-service-function-chaining-at-mef-openstack-summit-nfv-tosca-orchestration-network-automation/) 과 동일하며, 네트워크 보안을 위한 Function 을 Chaining 하는 것을 의미한다.
```
+------------+   +------------+   []   +------------+   +------------+   +------------+
|microservice|===|microservice|   []   |microservice|===|microservice|===|microservice|
|     A      |   |     B      |   []   |     A      |   |  security  |   |     B      |
+------------+   +------------+   []   +------------+   +------------+   +------------+

---

+------------+        +------------+
|microservice|        |microservice|
|    IPS     |  +----->  Firewall  |
+--^------+--+  |     +--+---------+
   |      |     |        |
+--+------v--+  |        |     +------------+
|microservice|  |        |     |microservice|
|     A      +--+        +----->     B      |
+------------+                 +------------+
```
위 예와 같이 _microservice A_ 와 _microservice B_ 사이에 네트워크가 미리 구성된 상태에서 네트워크의 보안을 위해 _microservice security_ 서비스를 동적으로 추가/삭제할 수 있는 것을 의미한다.

마이크로서비스의 특성상 수많은 서비스들의 생성과 삭제가 반복되는 관계로 IP 가 동적으로 교체될 수 있다.
즉, 이미 구성된 마이크로서비스간 네트워킹이 지속되기 위해서는 _iptables_ 관리가 매우 중요하다.


## Cloud-Native Network Policy Validation (CNNPV by [ZENTO](http://www.zento.co.kr/index.html))

### docker case

#### What is `nsenter`?
docker는 namespace 를 기반으로 컨테이너 환경을 격리한다.
`nsenter` 는 `namespace enter` 의 약어로, 이를 활용하여 격리된 namespace 에 접근할 수 있다.

예) PID 13566 프로세스의 netstat 실행:

```
// `-t` 는 `target` 을 의미
$ sudo nsenter -t 13566 -n netstat
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
```

즉, 원하는 docker 의 프로세스 id 만 nsenter 의 타겟으로 넘겨주면 내부 작업상태를 모니터링 할 수 있다.

### docker process id check

```
docker inspect -f '{{.State.Pid}}' {container_id or name}

$ docker inspect -f '{{.State.Pid}}' cb2939r52s22
5858
```

### docker socket check

```
$ sudo nsenter -t 5858 -n netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-77-225.ec2.:45104 ESTABLISHED
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-77-225.ec2.:14804 TIME_WAIT
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-76-6:seclayer-tls TIME_WAIT
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-76-65.ec:plethora TIME_WAIT
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-77-225.ec2.:14830 TIME_WAIT
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-76-65.ec2.i:23284 ESTABLISHED
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-76-65.ec2.i:27948 ESTABLISHED
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-77-225.ec2.:14848 TIME_WAIT
tcp        0      0 ip-172-17-0-2.ec2.:webcache ip-10-100-77-225.ec2.:45544 ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node Path

```

