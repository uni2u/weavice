# weavice (weave service)
weavice [wi:vis] is a Cloud Native network service.

## network policy validation

### docker socket check

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
