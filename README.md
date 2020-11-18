# otus-nfs

## Введение

Выполнение действий приведенных в данной методичке позволит познакомиться с настройкой клиента и сервера сетевой файловой системы NFS. Так же вы научитесь настраивать фаерволл для корректного функционирования NFS сервера и клиента.

Для выполнения потребуется следующее ПО:
- *VirtualBox* - среда виртуализации, позволяет создавать и выполнять виртуальные машины;
- *Vagrant* - ПО для создания и конфигурирования виртуальной среды. В данном случае в качестве среды виртуализации используется _VirtualBox_;
- *Git* - система контроля версий

## Документация по теме

- Документация http://wiki.linux-nfs.org/wiki/index.php/Main_Page
- Неплохая wiki от gentoo linux =) https://wiki.gentoo.org/wiki/Nfs-utils
- HOWTO http://nfs.sourceforge.net/nfs-howto/ar01s03.html
- Multi-machine Vagrantfile [documentation](https://www.vagrantup.com/docs/multi-machine)

## Установка необходимого ПО

### Vagrant

Установочные пакеты Vagrant доступны по следующей ссылке: https://www.vagrantup.com/downloads.html

Установка Vagrant описана в документации: https://learn.hashicorp.com/tutorials/vagrant/getting-started-install

Например, для установки vagrant версии 2.2.13 на debian-based дистрибутив, нужно выполнить следующие действия:
```shell
curl -O https://releases.hashicorp.com/vagrant/2.2.13/vagrant_2.2.13_x86_64.deb && \
sudo dpkg -i vagrant_2.2.13_x86_64.deb
```

После успешного окончания, vagrant будет установлен.

### VirtualBox

Подробнее об установке VirtualBox можно узнать из официальной документации: https://www.virtualbox.org/manual/UserManual.html#installation

Для установки VirtualBox из репозитория на debian-based дистрибутиве, нужно выполнить следующие действия:
```shell
apt install virtualbox
```

После успешного окончания, virtualbox будет установлен.

## Запуск тестового окружения

Для запуска тестового окружения необходимо установить необходимые приложения, описанные выше. Далее склонировать себе данный репозиторий:
```shell
git clone https://gitlab.com/vsyscoder/otus-nfs.git
```

Затем перейти в директорию с проектом и выполнить `vagrant up`
```shell
cd otus-nfs
vagrant up
```

После успешного завершения у вас будут подняты 2 виртуальных машины:
- `nfs-server` - 192.168.10.10
- `nfs-client` - 192.168.10.11

Содержимое [Vagrantfile](./Vagrantfile)

## Настройка сервера

Для подключения к серверу необходимо выполнить
```shell
vagrant ssh nfs-server
```

### Для информации

Модули ядра, относящиеся к NFS
```shell
cat /boot/config-$(uname -r) | grep NFS
```
<details><summary>Пример вывода</summary>
<p>

```log
CONFIG_XENFS=m
CONFIG_XEN_COMPAT_XENFS=y
CONFIG_KERNFS=y
CONFIG_NFS_FS=m
# CONFIG_NFS_V2 is not set
CONFIG_NFS_V3=m
CONFIG_NFS_V3_ACL=y
CONFIG_NFS_V4=m
# CONFIG_NFS_SWAP is not set
CONFIG_NFS_V4_1=y
CONFIG_NFS_V4_2=y
CONFIG_PNFS_FILE_LAYOUT=m
CONFIG_PNFS_BLOCK=m
CONFIG_PNFS_FLEXFILE_LAYOUT=m
CONFIG_NFS_V4_1_IMPLEMENTATION_ID_DOMAIN="kernel.org"
# CONFIG_NFS_V4_1_MIGRATION is not set
CONFIG_NFS_V4_SECURITY_LABEL=y
CONFIG_NFS_FSCACHE=y
# CONFIG_NFS_USE_LEGACY_DNS is not set
CONFIG_NFS_USE_KERNEL_DNS=y
CONFIG_NFS_DEBUG=y
CONFIG_NFS_DISABLE_UDP_SUPPORT=y
CONFIG_NFSD=m
CONFIG_NFSD_V2_ACL=y
CONFIG_NFSD_V3=y
CONFIG_NFSD_V3_ACL=y
CONFIG_NFSD_V4=y
# CONFIG_NFSD_BLOCKLAYOUT is not set
# CONFIG_NFSD_SCSILAYOUT is not set
# CONFIG_NFSD_FLEXFILELAYOUT is not set
# CONFIG_NFSD_V4_2_INTER_SSC is not set
CONFIG_NFSD_V4_SECURITY_LABEL=y
CONFIG_NFS_ACL_SUPPORT=m
CONFIG_NFS_COMMON=y
```
</p>
</details>


Просмотр информации о пакете `nfs-utils`
```shell
yum info nfs-utils
```
<details><summary>Пример вывода</summary>
<p>

```log
Загружены модули: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.datahouse.ru
 * elrepo: mirrors.colocall.net
 * extras: centos-mirror.rbc.ru
 * updates: mirror.docker.ru
Доступные пакеты
Название: nfs-utils
Архитектура: x86_64
Период: 1
Версия: 1.3.0
Выпуск: 0.66.el7
Объем: 412 k
Источник: base/7/x86_64
Аннотация: NFS utilities and supporting clients and daemons for the kernel NFS server
Ссылка: http://sourceforge.net/projects/nfs
Лицензия: MIT and GPLv2 and GPLv2+ and BSD
Описание: The nfs-utils package provides a daemon for the kernel NFS server and
        : related tools, which provides a much higher level of performance than the
        : traditional Linux NFS server used by most users.
        : 
        : This package also contains the showmount program.  Showmount queries the
        : mount daemon on a remote host for information about the NFS (Network File
        : System) server on the remote host.  For example, showmount can display the
        : clients which are mounted on that host.
        : 
        : This package also contains the mount.nfs and umount.nfs program.
```
</p>
</details>


### Установка необходимых пакетов

Установка пакетов необходимых для функционирования сервера
```shell
sudo yum install -y nfs-utils
```
[Пример вывода](./logs/nfs-install.log)

Описание сервисов NFS

- `rpcbind` universal addresses to RPC program number mapper
- `nfs-server` NFS server process
- `rpc-statd` NFS status monitor for NFSv2/3 locking
- `nfs-idmapd` NFSv4 ID-name mapping service

Включение автозапуска необходимых сервисов
```shell
sudo systemctl enable rpcbind
sudo systemctl enable nfs-server
sudo systemctl enable rpc-statd
sudo systemctl enable nfs-idmapd
```
<details><summary>Пример вывода</summary>
<p>

```log
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
```
</p>
</details>

Запуск сервисов
```shell
sudo systemctl start rpcbind
sudo systemctl start nfs-server
sudo systemctl start rpc-statd
sudo systemctl start nfs-idmapd
```

Получение информации о запущенных сервисах и используемых портах
```shell
rpcinfo 
```
<details><summary>Пример вывода</summary>
<p>

```log
   program version netid     address                service    owner
    100000    4    tcp       0.0.0.0.0.111          portmapper superuser
    100000    3    tcp       0.0.0.0.0.111          portmapper superuser
    100000    2    tcp       0.0.0.0.0.111          portmapper superuser
    100000    4    udp       0.0.0.0.0.111          portmapper superuser
    100000    3    udp       0.0.0.0.0.111          portmapper superuser
    100000    2    udp       0.0.0.0.0.111          portmapper superuser
    100000    4    local     /var/run/rpcbind.sock  portmapper superuser
    100000    3    local     /var/run/rpcbind.sock  portmapper superuser
    100024    1    udp       0.0.0.0.201.190        status     29
    100024    1    tcp       0.0.0.0.176.97         status     29
    100005    1    udp       0.0.0.0.78.80          mountd     superuser
    100005    1    tcp       0.0.0.0.78.80          mountd     superuser
    100005    2    udp       0.0.0.0.78.80          mountd     superuser
    100005    2    tcp       0.0.0.0.78.80          mountd     superuser
    100005    3    udp       0.0.0.0.78.80          mountd     superuser
    100005    3    tcp       0.0.0.0.78.80          mountd     superuser
    100003    3    tcp       0.0.0.0.8.1            nfs        superuser
    100003    4    tcp       0.0.0.0.8.1            nfs        superuser
    100227    3    tcp       0.0.0.0.8.1            nfs_acl    superuser
    100003    3    udp       0.0.0.0.8.1            nfs        superuser
    100227    3    udp       0.0.0.0.8.1            nfs_acl    superuser
    100021    1    udp       0.0.0.0.183.211        nlockmgr   superuser
    100021    3    udp       0.0.0.0.183.211        nlockmgr   superuser
    100021    4    udp       0.0.0.0.183.211        nlockmgr   superuser
    100021    1    tcp       0.0.0.0.129.255        nlockmgr   superuser
    100021    3    tcp       0.0.0.0.129.255        nlockmgr   superuser
    100021    4    tcp       0.0.0.0.129.255        nlockmgr   superuser
```
</p>
</details>


### Экспорт файловой системы NFS

Создать директорию для экспорта. Установить необходимые разрешения (`0777`).
```shell
sudo mkdir -p /export/shared
sudo chmod 0777 /export/shared
```

Описание экспортируемой директории в конфигурационном файле `/etc/etcports`
```shell
cat << EOF | sudo tee /etc/exports
/export/shared  192.168.10.0/24(rw,async)
EOF
```
Здесь:
- `/export/shared` - экспортируемая директория
- `192.168.10.0/24` - подсеть с которой разрешено монтирование экспортируемой ФС
- `(rw,async)` - опции экспортируемой ФС, подробнее можно почитать в `man exports`

Применение изменений конфигурации
> If you come back and change your /etc/exports file, the changes you make may not take effect immediately. You should run the command `exportfs -ra` to force `nfsd` to re-read the `/etc/exports` file. If you can't find the exportfs command, then you can `kill nfsd` with the `-HUP` flag (see the man pages for kill for details).
```
sudo exportfs -ra
```
где:
- `-a` Export or unexport all directories
- `-r` Reexport all directories, synchronizing `/var/lib/nfs/etab` with `/etc/exports` and files under `/etc/exports.d`.  This  option  removes  entries  in /var/lib/nfs/etab which have been deleted from `/etc/exports` or files under `/etc/exports.d`, and removes any entries from the kernel export table which are no longer valid.

Проверка экспортированных ФС
```shell
cat /var/lib/nfs/etab
```
```log
/export/shared  192.168.10.0/24(rw,async,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,rw,secure,root_squash,no_all_squash)
```


### Настройка firewalld на сервере

Проверка открытых портов
```shell
sudo ss -tunlp
```
<details><summary>Пример вывода</summary>
<p>

```log
Netid  State      Recv-Q Send-Q Local Address:Port   Peer Address:Port              
udp    UNCONN     0      0          127.0.0.1:323               *:*     users:(("chronyd",pid=803,fd=5))
udp    UNCONN     0      0                  *:51646             *:*     users:(("rpc.statd",pid=16653,fd=8))
udp    UNCONN     0      0                  *:20048             *:*     users:(("rpc.mountd",pid=16672,fd=7))
udp    UNCONN     0      0                  *:685               *:*     users:(("rpcbind",pid=16624,fd=7))
udp    UNCONN     0      0          127.0.0.1:717               *:*     users:(("rpc.statd",pid=16653,fd=5))
udp    UNCONN     0      0                  *:47059             *:*                  
udp    UNCONN     0      0                  *:2049              *:*                  
udp    UNCONN     0      0                  *:68                *:*     users:(("dhclient",pid=3425,fd=6))
udp    UNCONN     0      0                  *:111               *:*     users:(("rpcbind",pid=16624,fd=6))
tcp    LISTEN     0      64                 *:2049              *:*                  
tcp    LISTEN     0      128                *:45153             *:*     users:(("rpc.statd",pid=16653,fd=9))
tcp    LISTEN     0      128                *:111               *:*     users:(("rpcbind",pid=16624,fd=8))
tcp    LISTEN     0      128                *:20048             *:*     users:(("rpc.mountd",pid=16672,fd=8))
tcp    LISTEN     0      128                *:22                *:*     users:(("sshd",pid=1151,fd=3))
tcp    LISTEN     0      100        127.0.0.1:25                *:*     users:(("master",pid=1412,fd=13))
tcp    LISTEN     0      64                 *:33279             *:*
```
</p>
</details>


Проверка портов, связанных с функционированием служб относящихся к NFS
```shell
rpcinfo -p
```
<details><summary>Пример вывода</summary>
<p>

```log
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  51646  status
    100024    1   tcp  45153  status
    100005    1   udp  20048  mountd
    100005    1   tcp  20048  mountd
    100005    2   udp  20048  mountd
    100005    2   tcp  20048  mountd
    100005    3   udp  20048  mountd
    100005    3   tcp  20048  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100227    3   udp   2049  nfs_acl
    100021    1   udp  47059  nlockmgr
    100021    3   udp  47059  nlockmgr
    100021    4   udp  47059  nlockmgr
    100021    1   tcp  33279  nlockmgr
    100021    3   tcp  33279  nlockmgr
    100021    4   tcp  33279  nlockmgr
```
</p>
</details>

Получение предопределенных шаблонов сервисов
```shell
firewall-cmd --get-services
```
<details><summary>Пример вывода</summary>
<p>

```log
RH-Satellite-6 amanda-client amanda-k5-client amqp amqps apcupsd audit bacula bacula-client bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc ceph ceph-mon cfengine condor-collector ctdb dhcp dhcpv6 dhcpv6-client distcc dns docker-registry docker-swarm dropbox-lansync elasticsearch etcd-client etcd-server finger freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master git gre high-availability http https imap imaps ipp ipp-client ipsec irc ircs iscsi-target isns jenkins kadmin kerberos kibana klogin kpasswd kprop kshell ldap ldaps libvirt libvirt-tls lightning-network llmnr managesieve matrix mdns minidlna mongodb mosh mountd mqtt mqtt-tls ms-wbt mssql murmur mysql nfs nfs3 nmea-0183 nrpe ntp nut openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole plex pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster quassel radius redis rpc-bind rsh rsyncd rtsp salt-master samba samba-client samba-dc sane sip sips slp smtp smtp-submission smtps snmp snmptrap spideroak-lansync squid ssh steam-streaming svdrp svn syncthing syncthing-gui synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client upnp-client vdsm vnc-server wbem-http wbem-https wsman wsmans xdmcp xmpp-bosh xmpp-client xmpp-local xmpp-server zabbix-agent zabbix-server
```
</p>
</details>

Получение информации о шаблонах сервисов, относящихся к NFS

Получение информации о сервисе `mountd`
```shell
sudo firewall-cmd --info-service mountd
```
<details><summary>Пример вывода</summary>
<p>

```log
mountd
  ports: 20048/tcp 20048/udp
  protocols: 
  source-ports: 
  modules: 
  destination:
```
</p>
</details>

Получение информации о сервисе `nfs`
```shell
sudo firewall-cmd --info-service nfs
```
<details><summary>Пример вывода</summary>
<p>

```log
nfs
  ports: 2049/tcp
  protocols: 
  source-ports: 
  modules: 
  destination:
```
</p>
</details>

Получение информации о сервисе `nfs3`
```shell
sudo firewall-cmd --info-service nfs3
```
<details><summary>Пример вывода</summary>
<p>

```log
nfs3
  ports: 2049/tcp 2049/udp
  protocols: 
  source-ports: 
  modules: 
  destination:
```
</p>
</details>

Получение информации о сервисе `rpc-bind`
```shell
sudo firewall-cmd --info-service rpc-bind
```
<details><summary>Пример вывода</summary>
<p>

```log
rpc-bind
  ports: 111/tcp 111/udp
  protocols: 
  source-ports: 
  modules: 
  destination:
```
</p>
</details>


Включение и запуск firewalld
```shell
echo "Enable firewall"
sudo systemctl enable firewalld
sudo systemctl start firewalld
systemctl status firewalld
```

Открытие необходимых портов посредством включения соответствующих сервисов в firewalld
```shell
{
  sudo firewall-cmd --permanent --add-service=nfs3
  sudo firewall-cmd --permanent --add-service=mountd
  sudo firewall-cmd --permanent --add-service=rpc-bind
  sudo firewall-cmd --reload
  sudo firewall-cmd --list-all
}
```
<details><summary>Пример вывода</summary>
<p>

```log
success
success
success
success
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources: 
  services: dhcpv6-client mountd nfs3 rpc-bind ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```
</p>
</details>


## Настройка NFS клиента

Для подключения к клиенту необходимо выполнить
```shell
vagrant ssh nfs-client
```

### Установка необходимых пакетов

Установка `nfs-utils`
```shell
sudo yum install -y nfs-utils
```
[Пример вывода](./logs/nfs-install-cli.log)


### Монтирование файловой системы NFS

Попытка примонтировать без дополнительных опций
```shell
sudo mount 192.168.10.10:/export/shared /mnt
mount | grep nfs
```
```log
192.168.10.10:/export/shared on /mnt type nfs4 (rw,relatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.10.11,local_lock=none,addr=192.168.10.10)
```
приводит к монтированию NFSv4
```shell
sudo umount /mnt
```

Понтирование NFSv3 по UDP
```shell
sudo mount.nfs -vv 192.168.10.10:/export/shared /mnt -o nfsvers=3,proto=udp,soft
```
```log
...
    nfs-client: + echo Mount NFSv3 UDP
    nfs-client: + sudo mount.nfs -vv 192.168.10.10:/export/shared /mnt -o nfsvers=3,proto=udp,soft
    nfs-client: mount.nfs: trying 192.168.10.10 prog 100003 vers 3 prot UDP port 2049
    nfs-client: mount.nfs: trying 192.168.10.10 prog 100005 vers 3 prot UDP port 20048
    nfs-client: mount.nfs: timeout set for Sun Jun  7 17:26:41 2020
    nfs-client: mount.nfs: trying text-based options 'nfsvers=3,proto=udp,soft,addr=192.168.10.10'
    nfs-client: mount.nfs: prog 100003, trying vers=3, prot=17
    nfs-client: mount.nfs: prog 100005, trying vers=3, prot=17
```

Далее можно запустить небольшую проверку, залив файл размером 1Gb на примонтированную ФС
```shell
{
    dd if=/dev/zero of=/mnt/test1G.zero bs=1M count=1024
    sync
    ls -l /mnt
    rm /mnt/test1G.zero
}
```
```log
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 22.6744 s, 47.4 MB/s
total 1095500
-rw-rw-r--. 1 vagrant vagrant 1073741824 Jun  7 21:34 test1G.zero
```

Включение и запуск firewalld (на клиенте дополнительной настройки не требуется)
```shell
{
  sudo systemctl enable firewalld
  sudo systemctl start firewalld
}
```
```log
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
```

Проверка со включенным фаерволлом

Монтирование:
```shell
sudo umount /mnt
sudo mount.nfs -vv 192.168.10.10:/export/shared /mnt -o nfsvers=3,proto=udp,soft
```

Создание файла размером 1Gb
```shell
{
    test -f /mnt/test1G.zero && rm -f /mnt/test1G.zero
    dd if=/dev/zero of=/mnt/test1G.zero bs=1M count=1024
    sync
    ls -l /mnt
}
```
```log
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 18.9107 s, 56.8 MB/s
total 1049052
-rw-rw-r--. 1 vagrant vagrant 1073741824 Jun  7 22:01 test1G.zero
```

Чтение файла
```shell
dd if=/mnt/test1G.zero of=/dev/null
```
```log
2097152+0 records in
2097152+0 records out
1073741824 bytes (1.1 GB) copied, 2.38379 s, 450 MB/s
```

## Результат

В результате выполнения вышеуказанных действий, у вас подняты сервер NFS и клиентская машина, настроен фаерволл для пропуска nfs-трафика, экспортирована на сервере и смонтирована на клиенте файловая система.
