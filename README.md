# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки» - Ластухин Никита Sys-55


### Задание 1
Запустите два simple python сервера на своей виртуальной машине на разных портах
Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке
Настройте балансировку Round-robin на 4 уровне.
На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.



```
global
    daemon
    maxconn 4000
    log 127.0.0.1 local0
    stats socket /run/haproxy/stats mode 660 level admin

defaults
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s
    log global
    option httplog


frontend web_tcp
    bind 127.0.0.1:8080
    default_backend app_servers

backend app_servers
    balance roundrobin
    option httpchk GET / HTTP/1.1
    server s1 127.0.0.1:9001 check
    server s2 127.0.0.1:9002 check

listen stats
    bind 127.0.0.1:8889
    mode http
    stats enable
    stats uri /
    stats realm Haproxy\ Statistics
    stats auth admin:password
    stats refresh 5s

```
([Балансировка при запросах](https://psv4.userapi.com/s/v1/d2/uw4LmdtK4dQ7yxJh0MhWhNALOWcL7tpsc0WYBY1dhui1XokYoZIhHysEZ02zrCy_-Z9QmxIcvwSCrqymzqcaydm3qlPRhrkxP4UhlhTSaKUGmkcwYbhqEaNIwB8utmNWczkE13Eyhxaj/S.png))
([Сервер haproxy](https://psv4.userapi.com/s/v1/d2/1jq4Ziex9RT1CCl9Oe1yL8IpebfOBXkHLnE3xV3M0H632aGFMpkpRdKi7eDxLWbCA7TM7__1F9y8NituMOGLkgdwHjq4ZYLQmq9v8mXq6pohMAtzvlGCBrguR2nlnal5tnlQre5KJxIj/d.png))

---

### Задание 2
Запустите три simple python сервера на своей виртуальной машине на разных портах
Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.


```
global
    daemon
    maxconn 8000
    log 127.0.0.1 local0 info
    stats socket /run/haproxy/stats mode 660 level admin
    spread-checks 3

defaults
    mode http
    option redispatch 1
    option forwardfor
    retries 3
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms
    log global
    option httplog
    option dontlognull

frontend web_tcp
    bind 127.0.0.1:8080
    option tcplog
    default_backend app_servers

backend app_servers
    balance roundrobin
    option http-server-close
    option httpchk HEAD / HTTP/1.1\r\nHost:\ localhost
    http-check expect status 200
    default-server inter 3s fall 2 rise 2
    server s1 127.0.0.1:9001 weight 1 check port 9001
    server s2 127.0.0.1:9002 weight 1 check port 9002

listen stats
    bind 127.0.0.1:8889
    mode http
    stats enable
    stats uri /stats
    stats realm HAProxy\ Statistics
    stats show-legends
    stats hide-version
    stats admin if TRUE
    stats refresh 2s
    stats auth admin:password

```
([Балансировка](https://psv4.userapi.com/s/v1/d2/Lg0piQBH26HJxeiJQA2u4b0M-WI120tJMZL-PqOsVZ-f9DXgHCSj-u6UZ9_cHi06wNZlm-wJsMrIdejW2qMzyaZizJdzwHkzyZZ5leFGQ6YAAmVueu-iGD_iRW02MrSgkDM-plKqItru/B.png))
([Сервер статистики](https://psv4.userapi.com/s/v1/d2/GDLSboDnbpTb24bOTyAAXUzkBjpBBJUtib2k33juen5afdgccIBu9UmqsLCeVf5L6anGQBD8NcglAuOLYQTD-rFsPkoeyiLZZYGvZSHXyKy_6Lh_ipch8O48aeKBd4U9NftTyBvpEKzC/V.png))
([Ошибка без настроек](https://psv4.userapi.com/s/v1/d2/Yw-YXf2LdWJLEiw46iJfv5nv7VFyzMLalY9DZnZCCasFUMaxlvt-ODsXS9Z9FKmQ2MfqEhoW8B6eyFFynJo2TUg5har0rAHOnmYAzs2aWN4S80Va4wA1xDQPhHHfzQO8v9DQhXlPCTnh/A.png))
---

