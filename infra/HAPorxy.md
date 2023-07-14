# HAProxy

하드웨어 기반의 L4/L7 스위치을 대체하기 위한 오픈소스 소프트웨어 솔루션으로 TCP 및 HTTP 기반 애플리케이션을 위한 고가용성, 로드 밸런싱 및 프록시 기능을 제공하는 매우 빠르고 안정적인 무료 Reverse 프록시

## RAbbitMQ 클러스터를 위한 로드 밸런싱
HAProxy 설정
```bash
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon


defaults
        log     global
        mode    tcp
        option  tcplog
        option  dontlognull
        retries 3
        option redispatch
        timeout connect 1000
        timeout client  1000
        timeout server  1000

frontend ft_web
    bind *:1883
    default_backend bk_web

backend bk_web
    balance roundrobin
    server mq1 172.28.5.0:1883 check inter 1000
    server mq2 172.28.5.18:1883 check inter 1000
    server mq3 172.28.4.212:1883 check inter 1000
```

frontend 즉, client의 1883포트로 MQTT프로토콜 형태의 데이터를 받아 백엔드의 RabbitMQ에 전달하기 위한 설정이다. 

RabbitMQ 클러스터의 노드들이 살아있는지 확인하는 헬스 체크의 주기를 1초로 설정하여, 노드가 다운되면 빠르게 인지하여 로드밸런싱의 대상에서 제외하도록 했다.

그럼에도, 헬스체크 때는 살아 있었으나 실제 데이터 전송 시에는 노드가 다운되어 있을 수 있기 때문에, 실패 시 재시도하는 `retries` 옵션과 실패 시 다른 노드로 전달하는 `redispatch` 옵션을 주었다.

이렇게 하고 0.01초 마다 데이터를 전송하고 노드를 내렸을 때, 유실 없이 MQ에 데이터가 저장되는 것을 확인할 수 있었다.

다만, HAPRoxy 마저도 100% 안전할거란 보장이 없기 때문에 이중화를 해야 한다.

HAPRoxy 서버를 하나 더 만들고 VIP(Virtual IP)를 할당하여 main Node에 둔다. 이후, keepalived를 설치하여 HAProxy 노드 끼리 서로 헬스 체크를 하고, main Node가 다운된 경우 sub Node에 VIP를 이전하도록 구성할 수 있다.

다만, AWS에서는 VIP할당이 안되는듯 하다... ELB를 쓰라는건가..