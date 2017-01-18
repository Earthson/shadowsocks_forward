##### ip_forward

```bash
sysctl -w net.ipv4.ip_forward=1
echo net.ipv4.ip_forward=1 >> /etc/sysctl.d/99-sysctl.conf
```

##### shadowsocks-client

```bash
pacman -Sy shadowsocks-libev mariadb
mkdir /etc/shadowsocks
echo '{
    "server":"host",
    "server_port":port,
    "local_address": "0.0.0.0",
    "local_port":local_port,
    "password":",
    "timeout":600,
    "method":"aes-256-cfb"
}
' > /etc/shadowsocks/redir.json
echo '{
    "server":"",
    "server_port":,
    "local_address": "0.0.0.0",
    "local_port":,
    "password":"",
    "timeout":600,
    "method":"aes-256-cfb"
}
' > /etc/shadowsocks/tun.json

systemctl enable shadowsocks-libev-redir@redir
systemctl enable shadowsocks-libev-tunneltun
systemctl start shadowsocks-libev-redir@redir.service
systemctl start shadowsocks-libev-tunnel@tun.service
systemctl status shadowsocks-libev-redir@redir.service 
systemctl status shadowsocks-libev-tunnel@tun.service
```

```bash
function fetch_china_ip() {
    curl http://ftp.apnic.net/stats/apnic/delegated-apnic-latest | awk -F '|' '/CN/&&/ipv4/ {print $4 "/" 32-log($5)/log(2)}'
}

fetch_china_ip > $HOME/ip.txt
```



```bash


iptables -t nat -N SHADOWSOCKS

iptables -t nat -A SHADOWSOCKS -d $(resolveip -s $HOST) -j RETURN

iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/24,192.168.0.0/16,0.0.0.0/8,10.0.0.0/8,172.16.0.0/12,224.0.0.0/4,100.64.0.0/10,240.0.0.0/4,169.254.0.0/16,192.0.0.0/24,192.0.2.0/24,192.88.99.0/24,198.18.0.0/15,255.255.255.255/32,198.51.100.0/24,203.0.113.0/24  -j RETURN  
iptables -t nat -A SHADOWSOCKS -d $(cat $HOME/ip.txt | awk -vORS=, '{ print $1 }' | sed 's/,$/\n/') -j RETURN

iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 10001
iptables -t nat -A SHADOWSOCKS -p udp -j REDIRECT --to-ports 10001

iptables -t nat -I PREROUTING -p tcp -j SHADOWSOCKS

#清空
iptables -t nat -F SHADOWSOCKS
```

