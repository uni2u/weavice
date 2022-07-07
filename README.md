# weavice (weave service)
weavice [wi:vis] is a Cloud-Native network service.

## Cloud-Native Security Function Chaining (CNSFC by [TSNLab](https://www.tsnlab.com/home))


## Cloud-Native Network Policy Validation (CNNPV)

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

