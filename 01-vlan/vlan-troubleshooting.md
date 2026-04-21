# Реальные проблемы с VLAN и их решение (из опыта в Ростелекоме)

## Проблема 1: Компьютер не получает IP из правильного VLAN

**Симптомы:**
- ПК подключён к порту, настроенному на VLAN 10
- IP-адрес приходит из VLAN 1 (default)

**Диагностика:**
show vlan brief
show interfaces status
show mac address-table interface fastEthernet 0/5

**Что проверяю:**
1. Порт точно в access mode? switchport mode access
2. VLAN назначен? switchport access vlan 10
3. Нет ли конфликта с voice VLAN?

**Решение:** переприменить конфиг порта

interface fastEthernet 0/5
 switchport mode access
 switchport access vlan 10
 shutdown
 no shutdown

---

## Проблема 2: Trunk не пропускает нужные VLAN

**Симптомы:**
- На коммутаторе настроен trunk
- Но хосты в VLAN 20 не видят друг друга через trunk

**Диагностика:**
show interfaces trunk
show interfaces trunk allowed-vlan

**Что проверяю:**
1. Порт действительно в trunk mode?
2. Разрешены ли нужные VLAN? switchport trunk allowed vlan add 20

**Решение:** добавить VLAN в разрешённые

interface fastEthernet 0/24
 switchport trunk allowed vlan add 20

---

## Проблема 3: Неожиданное появление MAC-адресов в другом VLAN

**Симптомы:**
- MAC-адрес ПК из VLAN 10 появляется в таблице VLAN 20

**Что это может значить:**
- Кто-то подключил неуправляемый коммутатор
- Проблема с конфигурацией port-security

**Диагностика:**
show mac address-table | include MAC-адрес
show mac address-table interface fastEthernet 0/5

**Мои действия из опыта в Ростелекоме:**
1. Найти порт, на котором появился чужой MAC
2. Проверить, не включён ли на порту режим trunk
3. Если порт в access mode — сменить VLAN на изолирующий (например, VLAN 999)
4. Включить port-security

---

## Что я запомнил

- Всегда проверяю show vlan brief после настройки
- Перед изменением порта делаю show running-config interface (порт)
- В Ростелекоме была политика: отключать неиспользуемые порты и переводить в dead VLAN
