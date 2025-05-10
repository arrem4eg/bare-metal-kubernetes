```
                                   VIP:174.17.12.100
+---------------------------+                |               +---------------------------+
|  [haproxy1.artem4eg.cc]   | 174.17.12.194  | 174.17.12.198 |  [haproxy2.artem4eg.cc]   |
|        Keepalived#1       +---------------+----------------+       Keepalived#2        |
|                           |                                |                           |
+---------------------------+                                +---------------------------+
```

## haproxy1 
```bash
q@haproxy1:~$ sudo -i
root@haproxy1:~# hostnamectl set-hostname haproxy1
root@haproxy1:~# apt update
root@haproxy1:~# apt -y install keepalived
root@haproxy1:~# nano /etc/keepalived/keepalived.conf

global_defs {
    router_id haproxy1
}

vrrp_instance VRRP1 {
    state MASTER
    interface enp1s0
    virtual_router_id 101
    priority 200
    advert_int 1
    virtual_ipaddress {
        174.17.12.100/24
    }
}

root@haproxy1:~# systemctl restart keepalived
root@haproxy1:~# ip address show enp1s0
```

## haproxy2 
```bash
q@haproxy2:~$ sudo -i
root@haproxy2:~# hostnamectl set-hostname haproxy2
root@haproxy2:~# apt update
root@haproxy2:~# apt -y install keepalived
root@haproxy2:~# nano /etc/keepalived/keepalived.conf

global_defs {
    router_id haproxy2
}

vrrp_instance VRRP1 {
    state BACKUP
    interface enp1s0
    virtual_router_id 101
    priority 100
    advert_int 1
    virtual_ipaddress {
        174.17.12.100/24
    }
}

root@haproxy2:~# systemctl restart keepalived
root@haproxy2:~# ip address show enp1s0
```

## test 
```bash
root@haproxy1:~# reboot

root@haproxy2:~# ip address show enp1s0
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:3e:1e:43 brd ff:ff:ff:ff:ff:ff
    inet 174.17.12.198/24 metric 100 brd 174.17.12.255 scope global dynamic enp1s0
       valid_lft 1806sec preferred_lft 1806sec
    inet 174.17.12.100/24 scope global secondary enp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe3e:1e43/64 scope link 
       valid_lft forever preferred_lft forever
```