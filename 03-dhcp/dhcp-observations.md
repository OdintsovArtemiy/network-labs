# Мои наблюдения за DHCP 

## Как я проверяю работу DHCP

### 1. Смотрю выдачу адресов

show ip dhcp binding

Пример вывода:
192.168.10.15    00:15:5D:01:02:03   Jan 15 2026 10:30 AM   Automatic
192.168.10.16    00:15:5D:01:02:04   Jan 15 2026 10:32 AM   Automatic

### 2. Проверяю пулы

show ip dhcp pool

### 3. Смотрю конфликты (если два устройства получили один IP)

show ip dhcp conflict

---


**Ситуация:** Пользователи жаловались, что иногда пропадает интернет на 1-2 минуты

**Диагностика:**
1. Посмотрел логи DHCP
2. Увидел множественные конфликты IP-адресов
3. Оказалось, что кто-то подключил свой роутер с включённым DHCP-сервером

**Решение:**
1. Нашёл порт, где появился чужой DHCP-сервер (show dhcp snooping binding)
2. Отключил порт и нашёл устройство
3. Включил DHCP Snooping на всех access-портах

---

## Что такое DHCP Snooping и зачем он нужен

DHCP Snooping — защита от поддельных DHCP-серверов

**Настройка:**

ip dhcp snooping
ip dhcp snooping vlan 10,20
interface fastEthernet 0/24
 ip dhcp snooping trust (доверенный порт, куда подключен настоящий DHCP-сервер)

interface fastEthernet 0/1
 ip dhcp snooping limit rate 10 (не больше 10 запросов в секунду)

**Результат:** Чужой DHCP-сервер не сможет раздавать IP в нашей сети

---

## Полезные команды для DHCP диагностики

debug ip dhcp server events
debug ip dhcp server packets

show ip dhcp snooping
show ip dhcp snooping binding
show ip dhcp statistics
