# Реальные сценарии использования ACL

## Сценарий 1: Запретить доступ к серверу управления из гостевой сети

**Задача:** Гостевая WiFi сеть (192.168.50.0/24) не должна иметь доступ к серверу управления (192.168.1.10)

**Решение:**

access-list 101 deny ip 192.168.50.0 0.0.0.255 host 192.168.1.10
access-list 101 permit ip any any

interface vlan 50
 ip access-group 101 in

**Проверка:**
- Из гостевой сети: ping 192.168.1.10 -> таймаут
- Из корпоративной сети: пинг работает

---

## Сценарий 2: Ограничить SSH доступ к коммутаторам

**Задача:** Подключаться по SSH к коммутаторам можно только с IP-адресов инженеров (10.10.10.0/24)

**Решение:**

access-list 12 remark SSH allowed hosts
access-list 12 permit 10.10.10.0 0.0.0.255
access-list 12 deny any

line vty 0 4
 access-class 12 in
 transport input ssh

---

## Сценарий 3: Защита от подмены IP (анти-спуфинг)

**Задача:** В VLAN 10 (192.168.10.0/24) запретить трафик с чужими IP

**Решение:**

access-list 110 permit ip 192.168.10.0 0.0.0.255 any
access-list 110 deny ip any any

interface vlan 10
 ip access-group 110 in

**Как это работает:** Если кто-то подставит IP 8.8.8.8, трафик будет отброшен.

---

## Сценарий 4: Защита от DoS на маршрутизаторе

**Задача:** Ограничить ICMP (пинг) до 5 пакетов в секунду

**Решение:**

access-list 120 permit icmp any any

class-map match-all ICMP-TRAFFIC
 match access-group 120

policy-map ICMP-POLICY
 class ICMP-TRAFFIC
  police 5000 1000 conform-action transmit exceed-action drop

interface gigabitEthernet 0/0
 service-policy input ICMP-POLICY

---

## Что я вынес из практики

1. ACL применяются в порядке очереди — первое совпадение срабатывает
2. В конце всегда нужно правило permit any any, иначе всё заблокируется
3. Перед применением ACL на production лучше проверить в Packet Tracer
4. В Ростелекоме мы хранили все ACL в отдельном файле с комментариями
