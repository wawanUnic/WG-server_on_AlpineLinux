# WG-server_on_AlpineLinux
wireguard-server_on_AlpineLinux

Processor / Memory - VIA Eden Processor 1000MHz (32бит) / 0,5 Гб DDR2

AlpineLinux Version / Linux Kernel Version - 3.21.3 / 6.12.13-0-lts

The setting is made for a VPN network - 10.8.1.1/24

Port/exchange protocol - UDP:2021

Working as root

### 1. Let's add new repositories for updating packages
```
vi /etc/apk/repositories
	I
	http://dl-cdn.alpinelinux.org/alpine/v3.21/main
	http://dl-cdn.alpinelinux.org/alpine/v3.21/community
	Esc
	:wq
 ```

### 2. Let's update the system
```
apk update
apk upgrade
```

### 3. Let's add an additional folder for commits (this folder is not included in the list for commit by default)
```
lbu include /etc/init.d/
```

### 4. Let's install packages for the wireguard server to work
```
apk add wireguard-tools wireguard-tools-openrc iptables
```

### 5. Let's create keys
```
mkdir -p /etc/wireguard
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

### 6. Create /etc/wireguard/wg0.conf:
```
[Interface]
Address = 10.8.1.1/24
ListenPort = 2021
PrivateKey = <содержимое файла privatekey>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

### 7. Enable packet redirection. Edit /etc/sysctl.conf:
```
net.ipv4.ip_forward = 1
sysctl -p
```

### 8. Create a symbolic link:
```
ln -s /etc/init.d/wg-quick /etc/init.d/wg-quick.wg0
```

### 9. Enable autoload:
```
rc-update add wg-quick.wg0
```

### 10. Control commands:
```
rc-service wg-quick.wg0 start
rc-service wg-quick.wg0 stop
rc-service wg-quick.wg0 restart
rc-service wg-quick.wg0 status
```

### 11. WireGuard Test:
```
wg show
```

### 12. To save a commit to Alpine's persistent memory, use the command
```
lbu commit
```

### 13. Restart and recheck
```
reboot
```
