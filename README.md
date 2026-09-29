[NetSecure Lab.md](https://github.com/user-attachments/files/32808691/NetSecure.Lab.md)
# NetSecure Lab

Учебный pet-проект по сетям и безопасности Cisco: корпоративная сеть в **Cisco Packet Tracer** и инструмент аудита конфигураций VLAN/ACL.

> Проект создаётся пошагово в учебных целях. Каждое изменение сначала проверяется в Packet Tracer, затем фиксируется в конфигурации и при необходимости добавляется в автоматизированный аудитор.

## Цели проекта

- спроектировать сегментированную корпоративную сеть;
- изучить VLAN, trunk 802.1Q и Router-on-a-Stick;
- настроить DHCP, шлюзы и серверную зону;
- понять работу ACL и матрицы доступа;
- научиться находить уязвимые настройки Cisco IOS;
- разработать Python-инструмент для аудита `show running-config`;
- формировать отчёты с уровнем критичности и рекомендациями.

## Текущий статус

### Выполнено в Cisco Packet Tracer

- создан стенд центрального офиса;
- добавлены маршрутизатор `R1-EDGE`, коммутатор `SW1-CORE`, четыре ПК и сервер `SRV-APP`;
- настроены VLAN 10, 20, 30, 50, 60, 70, 80 и 999;
- настроен trunk между `SW1-CORE` и `R1-EDGE`;
- native VLAN изменена с VLAN 1 на VLAN 999;
- настроена маршрутизация между VLAN через Router-on-a-Stick;
- настроены DHCP-пулы для пользовательских VLAN;
- сервер получил статический адрес `10.10.50.10`;
- проверен доступ к серверу по ICMP и HTTP.

### Пока не выполнено

- ACL для ограничения гостевой сети;
- полная матрица доступа между VLAN;
- SSH и защищённый административный доступ;
- Port Security;
- отключение неиспользуемых портов;
- филиалы и динамическая маршрутизация;
- веб-интерфейс аудитора.

## Топология текущего стенда

```text
                         +----------------+
                         |    R1-EDGE     |
                         | Router 2911    |
                         |    Gi0/0       |
                         +-------+--------+
                                 |
                    802.1Q trunk | VLAN 10,20,30,50,60,70,80,999
                                 |
                         +-------+--------+
                         |    SW1-CORE    |
                         |   Switch 2960  |
                         +--+---+---+---+-+
                            |   |   |   |  |
                         Fa0/2 Fa0/3 Fa0/4 Fa0/5 Fa0/6
                            |   |   |   |  |
                         ADMIN ACC DEV GUEST SERVER
```

## План VLAN и адресации

| VLAN | Назначение | Сеть | Шлюз | Состояние |
|---:|---|---|---|---|
| 10 | ADMIN | `10.10.10.0/24` | `10.10.10.1` | используется |
| 20 | ACCOUNTING | `10.10.20.0/24` | `10.10.20.1` | используется |
| 30 | DEVELOPMENT | `10.10.30.0/24` | `10.10.30.1` | используется |
| 50 | SERVERS | `10.10.50.0/24` | `10.10.50.1` | используется |
| 60 | GUEST | `10.10.60.0/24` | `10.10.60.1` | используется |
| 70 | MANAGEMENT | `10.10.70.0/24` | `10.10.70.1` | подготовлена |
| 80 | DMZ | `10.10.80.0/24` | `10.10.80.1` | подготовлена |
| 999 | NATIVE_UNUSED | без IP | нет | native VLAN |

### Подключения коммутатора

| Порт | Устройство | VLAN | Адресация |
|---|---|---:|---|
| `Gi0/1` | `R1-EDGE` | trunk | 802.1Q |
| `Fa0/2` | `PC-ADMIN` | 10 | DHCP |
| `Fa0/3` | `PC-ACCOUNTING` | 20 | DHCP |
| `Fa0/4` | `PC-DEVELOPMENT` | 30 | DHCP |
| `Fa0/5` | `PC-GUEST` | 60 | DHCP |
| `Fa0/6` | `SRV-APP` | 50 | статический `10.10.50.10` |

## Сохранение лабораторной работы

### 1. Сохрани текущий Packet Tracer

В Cisco Packet Tracer выполни:

```text
File → Save As
```

Создай отдельную папку проекта и сохрани текущую версию под именем:

```text
NetSecure-Lab-03-Router-on-a-Stick.pkt
```

Не перезаписывай рабочую версию после каждого крупного изменения. Используй последовательные версии:

```text
NetSecure-Lab-01-Base.pkt
NetSecure-Lab-02-VLANs.pkt
NetSecure-Lab-03-Router-on-a-Stick.pkt
NetSecure-Lab-04-DHCP-Server.pkt
NetSecure-Lab-05-ACL.pkt
```

На текущем этапе следующую версию `NetSecure-Lab-05-ACL.pkt` пока создавать не нужно: ACL ещё не настроена.

### 2. Сохрани конфигурацию устройств внутри Packet Tracer

На `R1-EDGE` и `SW1-CORE` в CLI выполни:

```text
copy running-config startup-config
```

На вопрос:

```text
Destination filename [startup-config]?
```

нажми `Enter`.

`running-config` — текущая конфигурация в оперативной памяти. `startup-config` — конфигурация, которая будет загружена после перезапуска устройства.

### 3. Сделай резервные копии текстовых конфигураций

Позже для аудита понадобится текст команды:

```text
show running-config
```

В Packet Tracer можно выполнить её на каждом устройстве, скопировать вывод из CLI и сохранить в отдельные файлы:

```text
packet-tracer/configs/R1-EDGE-running-config.txt
packet-tracer/configs/SW1-CORE-running-config.txt
```

Сейчас это можно сделать после сохранения `.pkt`. Важно копировать именно `show running-config`, а не только отдельные команды.

## Быстрая проверка после открытия проекта

После повторного открытия `.pkt` проверь:

### На коммутаторе

```text
show vlan brief
show interfaces trunk
```

Ожидается:

- `Fa0/2` в VLAN 10;
- `Fa0/3` в VLAN 20;
- `Fa0/4` в VLAN 30;
- `Fa0/5` в VLAN 60;
- `Fa0/6` в VLAN 50;
- `Gi0/1` в режиме trunk;
- native VLAN `999`.

### На маршрутизаторе

```text
show ip interface brief
show ip route
show ip dhcp binding
```

Ожидается:

- подинтерфейсы `G0/0.10`, `.20`, `.30`, `.50`, `.60`, `.70`, `.80` имеют состояние `up/up`;
- сети `10.10.10.0/24`–`10.10.80.0/24` отображаются как directly connected;
- DHCP выдал адреса пользовательским ПК;
- сервер использует `10.10.50.10`.

## Инструмент аудита конфигураций

Аудитор находится в каталоге `auditor/`. Он анализирует текстовые конфигурации Cisco IOS и ищет типовые проблемы VLAN, trunk, ACL и управления.

Запуск:

```bash
cd netsecure-lab
python3 auditor/auditor.py examples/vulnerable_config.txt
```

JSON-отчёт:

```bash
python3 auditor/auditor.py \
  examples/vulnerable_config.txt \
  --format json \
  --output reports/vulnerable.json
```

Запуск тестов:

```bash
python3 -m pytest -q
```

## Структура проекта

```text
netsecure-lab/
├── README.md
├── docs/
│   └── 01-technical-task.md
├── examples/
│   └── vulnerable_config.txt
├── packet-tracer/
│   └── configs/
├── auditor/
│   ├── auditor.py
│   └── tests/
├── reports/
│   └── vulnerable.json
└── requirements-dev.txt
```

## Принцип аудита

Цикл разработки проекта:

```text
1. Спроектировать безопасную настройку
2. Собрать её в Packet Tracer
3. Получить show running-config
4. Проверить конфигурацию аудитором
5. Создать тестовую уязвимую версию
6. Убедиться, что аудитор находит проблему
7. Исправить конфигурацию
8. Сравнить результаты до и после
```

## Учебное предупреждение

Проект предназначен для учебной лаборатории и Cisco Packet Tracer. Конфигурации и правила необходимо дополнительно проверять перед применением на реальном оборудовании, поскольку версии Cisco IOS и поддерживаемый синтаксис могут отличаться.

## Лицензия

Учебный проект. Лицензия будет определена позже.


---

# NetSecure Lab — English Version

Educational networking and Cisco security pet project: a corporate network built in **Cisco Packet Tracer** and a configuration auditing tool for VLAN and ACL security.

> This project is developed step by step for educational purposes. Each configuration change is first tested in Packet Tracer, documented, and then added to the automated auditor when appropriate.

## Project goals

- design a segmented corporate network;
- study VLANs, 802.1Q trunks, and Router-on-a-Stick;
- configure DHCP, default gateways, and a server segment;
- understand ACL behavior and access-control matrices;
- detect vulnerable Cisco IOS configuration settings;
- develop a Python tool that audits `show running-config` output;
- generate reports with severity levels and remediation recommendations.

## Current status

### Completed in Cisco Packet Tracer

- created a headquarters lab topology;
- added router `R1-EDGE`, switch `SW1-CORE`, four PCs, and the `SRV-APP` server;
- configured VLANs 10, 20, 30, 50, 60, 70, 80, and 999;
- configured a trunk between `SW1-CORE` and `R1-EDGE`;
- changed the native VLAN from VLAN 1 to VLAN 999;
- configured inter-VLAN routing using Router-on-a-Stick;
- configured DHCP pools for user VLANs;
- assigned the server the static address `10.10.50.10`;
- verified ICMP and HTTP access to the server.

### Not implemented yet

- ACL for restricting guest network access;
- a complete inter-VLAN access matrix;
- SSH and secure administrative access;
- Port Security;
- shutdown of unused switch ports;
- branch offices and dynamic routing;
- a web interface for the auditor.

## Current topology

```text
                         +----------------+
                         |    R1-EDGE     |
                         | Router 2911    |
                         |    Gi0/0       |
                         +-------+--------+
                                 |
                    802.1Q trunk | VLAN 10,20,30,50,60,70,80,999
                                 |
                         +-------+--------+
                         |    SW1-CORE    |
                         |   Switch 2960  |
                         +--+---+---+---+-+
                            |   |   |   |  |
                         Fa0/2 Fa0/3 Fa0/4 Fa0/5 Fa0/6
                            |   |   |   |  |
                         ADMIN ACC DEV GUEST SERVER
```

## VLAN and IP addressing plan

| VLAN | Purpose | Network | Gateway | Status |
|---:|---|---|---|---|
| 10 | ADMIN | `10.10.10.0/24` | `10.10.10.1` | in use |
| 20 | ACCOUNTING | `10.10.20.0/24` | `10.10.20.1` | in use |
| 30 | DEVELOPMENT | `10.10.30.0/24` | `10.10.30.1` | in use |
| 50 | SERVERS | `10.10.50.0/24` | `10.10.50.1` | in use |
| 60 | GUEST | `10.10.60.0/24` | `10.10.60.1` | in use |
| 70 | MANAGEMENT | `10.10.70.0/24` | `10.10.70.1` | prepared |
| 80 | DMZ | `10.10.80.0/24` | `10.10.80.1` | prepared |
| 999 | NATIVE_UNUSED | no IP address | none | native VLAN |

### Switch port assignments

| Port | Device | VLAN | Addressing |
|---|---|---:|---|
| `Gi0/1` | `R1-EDGE` | trunk | 802.1Q |
| `Fa0/2` | `PC-ADMIN` | 10 | DHCP |
| `Fa0/3` | `PC-ACCOUNTING` | 20 | DHCP |
| `Fa0/4` | `PC-DEVELOPMENT` | 30 | DHCP |
| `Fa0/5` | `PC-GUEST` | 60 | DHCP |
| `Fa0/6` | `SRV-APP` | 50 | static `10.10.50.10` |

## Saving the lab

### 1. Save the Packet Tracer project

In Cisco Packet Tracer, select:

```text
File → Save As
```

Create a separate project folder and save the current version as:

```text
NetSecure-Lab-04-DHCP-Server.pkt
```

Do not overwrite the working version after every major change. Use versioned files:

```text
NetSecure-Lab-01-Base.pkt
NetSecure-Lab-02-VLANs.pkt
NetSecure-Lab-03-Router-on-a-Stick.pkt
NetSecure-Lab-04-DHCP-Server.pkt
NetSecure-Lab-05-ACL.pkt
```

The `NetSecure-Lab-05-ACL.pkt` version should not be created yet because ACLs have not been configured.

### 2. Save device configurations inside Packet Tracer

On both `R1-EDGE` and `SW1-CORE`, run:

```text
copy running-config startup-config
```

When Packet Tracer asks for:

```text
Destination filename [startup-config]?
```

press `Enter`.

`running-config` is the current configuration in RAM. `startup-config` is the saved configuration loaded after a device restart.

### 3. Export text configurations for the auditor

The auditor will analyze the output of:

```text
show running-config
```

On each device, run the command, copy the CLI output, and save it as:

```text
packet-tracer/configs/R1-EDGE-running-config.txt
packet-tracer/configs/SW1-CORE-running-config.txt
```

Save the complete output, from the configuration header through the final `end` line.

## Quick verification after reopening the project

### On the switch

```text
show vlan brief
show interfaces trunk
```

Expected results:

- `Fa0/2` belongs to VLAN 10;
- `Fa0/3` belongs to VLAN 20;
- `Fa0/4` belongs to VLAN 30;
- `Fa0/5` belongs to VLAN 60;
- `Fa0/6` belongs to VLAN 50;
- `Gi0/1` operates as a trunk;
- the native VLAN is `999`.

### On the router

```text
show ip interface brief
show ip route
show ip dhcp binding
```

Expected results:

- subinterfaces `G0/0.10`, `.20`, `.30`, `.50`, `.60`, `.70`, and `.80` are `up/up`;
- networks from `10.10.10.0/24` through `10.10.80.0/24` appear as directly connected;
- DHCP has assigned addresses to user PCs;
- the server uses `10.10.50.10`.

## Configuration auditing tool

The auditor is located in the `auditor/` directory. It analyzes Cisco IOS text configurations and detects common VLAN, trunk, ACL, and management-security problems.

Run the auditor:

```bash
cd netsecure-lab
python3 auditor/auditor.py examples/vulnerable_config.txt
```

Generate a JSON report:

```bash
python3 auditor/auditor.py \
  examples/vulnerable_config.txt \
  --format json \
  --output reports/vulnerable.json
```

Run automated tests:

```bash
python3 -m pytest -q
```

## Project structure

```text
netsecure-lab/
├── README.md
├── docs/
│   └── 01-technical-task.md
├── examples/
│   └── vulnerable_config.txt
├── packet-tracer/
│   └── configs/
├── auditor/
│   ├── auditor.py
│   └── tests/
├── reports/
│   └── vulnerable.json
└── requirements-dev.txt
```

## Audit workflow

```text
1. Design a secure configuration
2. Build it in Packet Tracer
3. Export show running-config
4. Run the configuration auditor
5. Create a deliberately vulnerable version
6. Confirm that the auditor detects the problem
7. Fix the configuration
8. Compare the results before and after remediation
```

## Educational disclaimer

This project is intended for an educational lab and Cisco Packet Tracer. Before using any configuration on real equipment, verify the commands against the relevant Cisco IOS version and the organization's security requirements.

## License

Educational project. The license will be defined later.
