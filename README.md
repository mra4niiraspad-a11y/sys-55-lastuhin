# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки» - Ластухин Никита Sys-55


### Задание 1
Запустите два simple python сервера на своей виртуальной машине на разных портах
Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке
Настройте балансировку Round-robin на 4 уровне.
На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.



```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

listen stats
    bind :8889
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats realm Haproxy\ Statistics
    stats auth admin:password

frontend web_frontend
    bind :8080
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk GET /
    server s1 127.0.0.1:8888 check
    server s2 127.0.0.1:9999 check


```

(![Балансировка]((https://github.com/mra4niiraspad-a11y/sys-55-lastuhin/blob/main/S.png))
([Сервер haproxy](https://psv4.userapi.com/s/v1/d2/1jq4Ziex9RT1CCl9Oe1yL8IpebfOBXkHLnE3xV3M0H632aGFMpkpRdKi7eDxLWbCA7TM7__1F9y8NituMOGLkgdwHjq4ZYLQmq9v8mXq6pohMAtzvlGCBrguR2nlnal5tnlQre5KJxIj/d.png))

---

### Задание 2
Запустите три simple python сервера на своей виртуальной машине на разных портах
Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.


```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

listen stats
    bind :8889
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats realm Haproxy\ Statistics
    stats auth admin:password

frontend web_frontend
    bind :80
    acl host_example hdr(host) -i example.local
    use_backend weighted_servers if host_example
    default_backend no_balancing

backend weighted_servers
    mode http
    balance weight roundrobin
    option httpchk GET /
    server s1 127.0.0.1:8888 weight 2 check
    server s2 127.0.0.1:9999 weight 3 check
    server s3 127.0.0.1:7777 weight 4 check

backend no_balancing
    mode http
    errorfile 503 /etc/haproxy/errors/503.http


```
([Балансировка](https://psv4.userapi.com/s/v1/d2/Lg0piQBH26HJxeiJQA2u4b0M-WI120tJMZL-PqOsVZ-f9DXgHCSj-u6UZ9_cHi06wNZlm-wJsMrIdejW2qMzyaZizJdzwHkzyZZ5leFGQ6YAAmVueu-iGD_iRW02MrSgkDM-plKqItru/B.png))
([Сервер статистики](https://psv4.userapi.com/s/v1/d2/GDLSboDnbpTb24bOTyAAXUzkBjpBBJUtib2k33juen5afdgccIBu9UmqsLCeVf5L6anGQBD8NcglAuOLYQTD-rFsPkoeyiLZZYGvZSHXyKy_6Lh_ipch8O48aeKBd4U9NftTyBvpEKzC/V.png))
([Ошибка без настроек](https://psv4.userapi.com/s/v1/d2/Yw-YXf2LdWJLEiw46iJfv5nv7VFyzMLalY9DZnZCCasFUMaxlvt-ODsXS9Z9FKmQ2MfqEhoW8B6eyFFynJo2TUg5har0rAHOnmYAzs2aWN4S80Va4wA1xDQPhHHfzQO8v9DQhXlPCTnh/A.png))
---

