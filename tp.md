# TP1 : Back to basics

# I. Gather informations

**Première étape : récupération d'infos sur le système.**  


Commandes à utiliser (ou pas) : `ip`, `ss`, `nmcli`, `cat`, `dig`, `firewall-cmd`

* 🌞 récupérer une **liste des cartes réseau** avec leur nom, leur IP et leur adresse MAC

```bash
[centos8@localhost bin]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:53:5f:73 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 84421sec preferred_lft 84421sec
    inet6 fe80::4ad6:a897:e477:132f/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:5a:7d:3e brd ff:ff:ff:ff:ff:ff
    inet 10.1.0.10/24 brd 10.1.0.255 scope global dynamic noprefixroute enp0s8
       valid_lft 269sec preferred_lft 269sec
    inet6 fe80::4d7a:ff8c:a5c9:1ee7/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```



* 🌞 déterminer si les cartes réseaux ont récupéré une **IP en DHCP** ou non

enp0s3 :

```bash
[centos8@localhost lib]$ sudo nmcli -f DHCP4 con show enp0s3
DHCP4.OPTION[1]:                        domain_name = auvence.co
DHCP4.OPTION[2]:                        domain_name_servers = 10.33.10.20 10.33.10.2 8.8.8.8 8.8.4.4
DHCP4.OPTION[3]:                        expiry = 1569587988
DHCP4.OPTION[4]:                        ip_address = 10.0.2.15
DHCP4.OPTION[5]:                        requested_broadcast_address = 1
DHCP4.OPTION[6]:                        requested_dhcp_server_identifier = 1
DHCP4.OPTION[7]:                        requested_domain_name = 1
DHCP4.OPTION[8]:                        requested_domain_name_servers = 1
DHCP4.OPTION[9]:                        requested_domain_search = 1
DHCP4.OPTION[10]:                       requested_host_name = 1
DHCP4.OPTION[11]:                       requested_interface_mtu = 1
DHCP4.OPTION[12]:                       requested_ms_classless_static_routes = 1
DHCP4.OPTION[13]:                       requested_nis_domain = 1
DHCP4.OPTION[14]:                       requested_nis_servers = 1
DHCP4.OPTION[15]:                       requested_ntp_servers = 1
DHCP4.OPTION[16]:                       requested_rfc3442_classless_static_routes = 1
DHCP4.OPTION[17]:                       requested_routers = 1
DHCP4.OPTION[18]:                       requested_static_routes = 1
DHCP4.OPTION[19]:                       requested_subnet_mask = 1
DHCP4.OPTION[20]:                       requested_time_offset = 1
DHCP4.OPTION[21]:                       requested_wpad = 1
DHCP4.OPTION[22]:                       routers = 10.0.2.2
DHCP4.OPTION[23]:                       subnet_mask = 255.255.255.0
```
enp0s8 :
```bash
[centos8@localhost lib]$ sudo nmcli -f DHCP4 con show Wired\ connection\ 1
DHCP4.OPTION[1]:                        expiry = 1569505935
DHCP4.OPTION[2]:                        ip_address = 10.1.0.10
DHCP4.OPTION[3]:                        requested_broadcast_address = 1
DHCP4.OPTION[4]:                        requested_dhcp_server_identifier = 1
DHCP4.OPTION[5]:                        requested_domain_name = 1
DHCP4.OPTION[6]:                        requested_domain_name_servers = 1
DHCP4.OPTION[7]:                        requested_domain_search = 1
DHCP4.OPTION[8]:                        requested_host_name = 1
DHCP4.OPTION[9]:                        requested_interface_mtu = 1
DHCP4.OPTION[10]:                       requested_ms_classless_static_routes = 1
DHCP4.OPTION[11]:                       requested_nis_domain = 1
DHCP4.OPTION[12]:                       requested_nis_servers = 1
DHCP4.OPTION[13]:                       requested_ntp_servers = 1
DHCP4.OPTION[14]:                       requested_rfc3442_classless_static_routes = 1
DHCP4.OPTION[15]:                       requested_routers = 1
DHCP4.OPTION[16]:                       requested_static_routes = 1
DHCP4.OPTION[17]:                       requested_subnet_mask = 1
DHCP4.OPTION[18]:                       requested_time_offset = 1
DHCP4.OPTION[19]:                       requested_wpad = 1
DHCP4.OPTION[20]:                       subnet_mask = 255.255.255.0
```
Je dirai que la enp0s3 récupère en DHCP car il possède ``domain_name_servers`` avec des IP et que enp0s8 non.
* 🌞 afficher la **table de routage** de la machine et sa **table ARP**

**table de routage :**
```bash
[centos8@localhost ~]$ ip route
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.10 metric 101
```

ligne 1 : Cette route est vers le réseau wan 10.0.2.2  
ligne 2 : 10.0.2.0/24 indique que le réseau est local, elle est utilisée pour une connexion locale  
La passerelle de cette route est à l IP 10.0.2.15 et cette IP est portée par enp0s3  
ligne 3 : 10.1.0.0/24 indique que le réseau est local, elle est utilisée pour une connexion locale
La passerelle de cette route est à l IP 10.1.0.10 et cette IP est portée par enp0s8


**table ARP :**
```bash
[centos8@localhost ~]$ arp -an
? (10.0.2.2) at 52:54:00:12:35:02 [ether] on enp0s3
? (10.1.0.1) at 0a:00:27:00:00:0c [ether] on enp0s8
```
ligne 1 : 10.0.2.2 est la passerelle du réseau 10.0.2.0/24  et 52:54:00:12:35:02 comme adresse mac portée par enp0s3  
ligne 2 : 10.1.0.1 est l'adresse IP de l'ordinateur sur le réseau et 0a:00:27:00:00:0c comme adresse mac portée par enp0s8


* 🌞 récupérer **la liste des ports en écoute** (*listening*) sur la machine (TCP et UDP)

```bash
[centos8@localhost ~]$ netstat -atu
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0     64 localhost.localdoma:ssh 10.1.0.1:50650          ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
udp        0      0 localhost.locald:bootpc 0.0.0.0:*         
udp        0      0 localhost.locald:bootpc 0.0.0.0:*         
udp        0      0 localhost:323           0.0.0.0:*         
udp6       0      0 localhost:323           [::]:*            
```

Le SSh écoute sur le port 22 de la machine.

* 🌞 récupérer **la liste des DNS utilisés par la machine**

```bash
[centos8@localhost etc]$ command nmcli device show enp0s3
GENERAL.DEVICE:                         enp0s3
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         08:00:27:53:5F:73
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     enp0s3
GENERAL.CON-PATH:                       /org/freedesktop/Netw>
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         10.0.2.15/24
IP4.GATEWAY:                            10.0.2.2
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh =>
IP4.ROUTE[2]:                           dst = 10.0.2.0/24, nh>
IP4.DNS[1]:                             10.33.10.20
IP4.DNS[2]:                             10.33.10.2
IP4.DNS[3]:                             8.8.8.8
IP4.DNS[4]:                             8.8.4.4
IP4.DOMAIN[1]:                          auvence.co
IP6.ADDRESS[1]:                         fe80::4ad6:a897:e477:>
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh =>
IP6.ROUTE[2]:                           dst = ff00::/8, nh = >
```

* effectuez une requête DNS afin de récupérer l'adresse IP associée au domaine `www.reddit.com` ~~(parce que c'est important d'avoir les bonnes adresses)~~
  ```bash 
  [centos8@localhost etc]$ ping www.redit.com
  PING www.reddit.com (192.254.236.136) 56(84) bytes of data.
  ```

  * 🌞 afficher **l'état actuel du firewall*

  ```bash
  [centos8@localhost etc]$ sudo firewall-cmd --list-all
  public (active)
    target: default
    icmp-block-inversion: no
    interfaces: enp0s3 enp0s8
    sources:
    services: cockpit dhcpv6-client ssh
    ports:
    protocols:
    masquerade: no
    forward-ports:
    source-ports:
    icmp-blocks:
    rich rules:
  ```
  * quelles interfaces sont filtrées ? enp0s3 et enp0s8
  * quel port TCP/UDP sont autorisés/filtrés ? ``cockpit``, ``dhcpv6`` et ``ssh`` qui est le port 22 TCP  


## II. Edit configuration

**Deuxièmement : Modifier la configuration existante**

Commandes à utiliser (ou pas) : `vim`, `cat`, `nmcli`, `systemctl`, `firewall-cmd`

---

### 1. Configuration cartes réseau

**NB** : sur CentOS8, la gestion des cartes réseau a légèrement changé. Il existe un démon qui gère désormais tout ce qui est relatif au réseau : NetworkManager.  

Marche à suivre pour modifier la configuration d'une carte réseau :
* édition du fichier de configuration
  * `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
* refresh de NetworkManager ("Hey prend mes modifications en compte stp !")
  * `sudo nmcli connection reload` 
  * `sudo nmcli con reload` même chose, on peut abréger les commandes `nmcli`
  * `sudo nmcli c reload` même chose aussi
* restart de l'interface
  * `sudo ifdown enp0s8` puis `sudo ifup enp0s8`
  * **OU** `sudo nmcli con up enp0s8`

---

* 🌞 modifier la configuration de la carte réseau privée

```bash
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s8
```
nano :

```bash
DEVICE=enp0s8
BOOTPROTO=none
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=192.168.0.2
```

* 🌞 dans la VM définir une IP statique pour cette nouvelle carte

```bash
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s4
```
nano :

```bash
DEVICE=enp0s4
BOOTPROTO=none
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=192.168.0.4
```

### 2. Serveur SSH

* 🌞 modifier la configuration du système pour que le serveur SSH tourne sur le port 2222
  
  ```bash
  sudo nano /etc/ssh/sshd_config
  ...
  #Port 2222
  ...
  ```

* adapter la configuration du firewall (fermer l'ancien port, ouvrir le nouveau)
  ```bash
  sudo semanage port -a -t ssh_port_t -p tcp 2222
  ```
   

# III. Routage simple

Dans cette partie, vous allez remettre en place un routage statique simple. Vous êtes libres du choix de la techno (CentOS8, Cisco, autres. Vous pouvez utiliser GNS3). 

Vous devez reproduire la mini-archi suivante : 
```
                   +-------+
                   |Outside|
                   | world |
                   +---+---+
                       |
                       |
+-------+         +----+---+         +-------+
|       |   net1  |        |   net2  |       |
|  VM1  +---------+ Router +---------+  VM2  |
|       |         |        |         |       |
+-------+         +--------+         +-------+
```

* **Description**
  * Le routeur a trois interfaces, dont une qui permet de joindre l'extérieur (internet)
  * La `VM1` a une interface dans le réseau `net1`
  * La `VM2` a une interface dans le réseau `net2`
  * Les deux VMs peuvent joindre Internet en passant par le `Router`
* 🌞 **To Do** 
  * Tableau récapitulatif des IPs
  * Configuration (bref) de VM1 et VM2
  * Configuration routeur
  * Preuve que VM1 passe par le routeur pour joindre internet
  * Une (ou deux ? ;) ) capture(s) réseau ainsi que des explications qui mettent en évidence le routage effectué par le routeur

# IV. Autres applications et métrologie

Dans cette partie, on va jouer un peu avec de nouvelles commandes qui peuvent être utiles pour diagnostiquer un peu ce qu'il se passe niveau réseau.

## 1. Commandes

* jouer avec iftop
  * expliquer son utilisation et imaginer un cas où iftop peut être utile
   ```bash
   Ecouter une interface spécifique, voir le traffic.
   Avec un sudo iftop -F net/mask on peut voir le trafic d entrée et de sortie sur un réseau IPV4.
   ```

---
## 2. Cockpit

* 🌞 mettre en place cockpit sur la VM1
  * c'est quoi ? C'est un service web. Pour quoi faire ? Vous allez vite comprendre en le voyant.
  * `sudo dnf install -y cockpit`
  * `sudo systemctl start cockpit`
  * trouver (à l'aide d'une commande shell) sur quel port (TCP ou UDP) écoute Cockpit 
    ```bash
    info cockpit
    ```
    il écoute sur le port 9090 TCP

  * vérifier que le port est ouvert dans le firewall
    ```bash
    
    ```
* 🌞 explorer Cockpit, plus spécifiquement ce qui est en rapport avec le réseau
    ```bash

    ```

