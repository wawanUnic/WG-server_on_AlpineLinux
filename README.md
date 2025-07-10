# WG-server_on_AlpineLinux
wireguard-server_on_AlpineLinux

Процессор / память - VIA Eden Processor 1000MHz (32бит) / 0,5 Гб DDR2

Версия AlpineLinux / Версия ядра Linux - 3.21.3 / 6.12.13-0-lts

Настрока производится для VPN сети - 10.8.1.1/24

Порт/протокол обмена - UDP:2021

Работаем от root

### 1. Впишем новые репозитории для обновления пакетов
```
vi /etc/apk/repositories
	I
	http://dl-cdn.alpinelinux.org/alpine/v3.21/main
	http://dl-cdn.alpinelinux.org/alpine/v3.21/community
	Esc
	:wq
 ```

### 2. Обновим систему
```
apk update
apk upgrade
```

### 3. Впишем доп.папку для коммитов (эта папка по-умолчанию не включена в список для коммита)
```
lbu include /etc/init.d/
```

### 4. Установим пакеты для работы сервера wireguard
```
apk add wireguard-tools wireguard-tools-openrc iptables
```
mkdir -p /etc/wireguard
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey


/etc/wireguard/wg0.conf:
[Interface]
Address = 10.8.1.1/24
ListenPort = 2021
PrivateKey = <содержимое файла privatekey>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE


/etc/sysctl.conf:
net.ipv4.ip_forward = 1
sysctl -p


Создай символическую ссылку:
ln -s /etc/init.d/wg-quick /etc/init.d/wg-quick.wg0
автозарузка:
rc-update add wg-quick.wg0
управление:
rc-service wg-quick.wg0 start
rc-service wg-quick.wg0 stop
rc-service wg-quick.wg0 restart
rc-service wg-quick.wg0 status

wg show
