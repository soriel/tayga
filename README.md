# NAT64 Gateway for MikroTik using Docker (Tayga)

Комплексное решение NAT64 для MikroTik на базе Docker контейнера с Tayga, позволяющее IPv6-клиентам взаимодействовать с IPv4-ресурсами.

## Содержание
1. [Поддерживаемые архитектуры](#поддерживаемые-архитектуры)
2. [Сборка контейнеров](#сборка-контейнеров)
3. [Настройка MikroTik](#настройка-mikrotik)
4. [Принцип работы](#принцип-работы)
5. [Устранение неисправностей](#устранение-неисправностей)
6. [English Version](#english-version)

---

## Поддерживаемые архитектуры
- ARM (`tayga-arm-2`)
- ARM64 (`tayga-arm64-2`)
- AMD64 (`tayga-amd64-2`)

## Сборка контейнеров

### Требования
- Установленный Docker
- Пакеты для multiarch сборки:
  ```bash
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
Подготовка окружения
bash
# Установка QEMU для multiarch
docker run --privileged --rm tonistiigi/binfmt --install all
Сборка образов
Выберите команду для вашей архитектуры:

bash
# Для ARM
docker buildx build --no-cache --platform arm --output=type=docker -t tayga-arm-2 .

# Для ARM64
docker buildx build --no-cache --platform arm64 --output=type=docker -t tayga-arm64-2 .

# Для AMD64
docker buildx build --no-cache --platform amd64 --output=type=docker -t tayga-amd64-2 .
Экспорт образов
bash
docker save tayga-arm-2 > /tmp/tayga-arm-2.tar
# Аналогично для других архитектур
Настройка MikroTik
Настройка интерфейсов
ros
/interface veth
add address=172.17.0.2/24,2001:db8:0:ffff::2/64 gateway=172.17.0.1 gateway6=2001:db8:0:ffff::1 name=veth1

/interface bridge add name=containers
/interface bridge port add bridge=containers interface=veth1
/ip address add address=172.17.0.1/24 interface=containers network=172.17.0.0
Конфигурация контейнера
ros
/container
add envlist=tayga interface=veth1 logging=yes workdir=/ file=tayga-arm-1.tar
/container envs
add key=TAYGA_CONF_IPV4_ADDR name=tayga value=172.18.20.1
add key=TAYGA_CONF_DYNAMIC_POOL name=tayga value=172.18.0.0/20
add key=TAYGA_CONF_PREFIX name=tayga value=2001:db8:1:ffff::/96
add key=TAYGA_IPV6_ADDR name=tayga value=2001:db8:0:ffff::2
Маршрутизация
ros
/ipv6 route 
add disabled=no dst-address=2001:db8:1:ffff::/96 gateway=2001:db8:0:ffff::2 routing-table=main suppress-hw-offload=no

/ipv6 address
add address=2001:db8:0:ffff::1/64 advertise=no interface=containers

/ip route 
add disabled=no distance=1 dst-address=172.18.0.0/20 gateway=172.17.0.2 routing-table=main scope=30 suppress-hw-offload=no target-scope=10
NAT
ros
/ip firewall nat
add action=masquerade chain=srcnat src-address=172.18.0.0/20
Принцип работы
Контейнер запускает демон Tayga NAT64

Создается bridge-интерфейс для сетевого взаимодействия

Настраивается маршрутизация IPv4/IPv6 между контейнером и MikroTik

Конфигурируется NAT для исходящего IPv4-трафика от IPv6-клиентов

Устранение неисправностей
Проверка работы контейнера: /container print

Просмотр логов: /container logs [ID_контейнера] 0

Проверка маршрутов на MikroTik

Тестирование подключения с IPv6-клиентов к IPv4-ресурсам

English Version
Supported Architectures
ARM (tayga-arm-2)

ARM64 (tayga-arm64-2)

AMD64 (tayga-amd64-2)

Container Build
bash
# Install prerequisites
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Setup QEMU for multiarch
docker run --privileged --rm tonistiigi/binfmt --install all

# Build for ARM
docker buildx build --no-cache --platform arm --output=type=docker -t tayga-arm-2 .

# Export image
docker save tayga-arm-2 > /tmp/tayga-arm-2.tar
Configuration Variables
Variable	Default Value	Description
TAYGA_CONF_DATA_DIR	/var/db/tayga	Tayga data directory
TAYGA_CONF_DIR	/usr/local/etc	Configuration directory
TAYGA_CONF_IPV4_ADDR	172.18.0.100	NAT64 IPv4 address
TAYGA_CONF_PREFIX	2001:db8:64:ff9b::/96	NAT64 IPv6 prefix
TAYGA_CONF_DYNAMIC_POOL	172.18.0.128/25	IPv4 dynamic pool
