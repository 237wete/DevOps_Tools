## Configuration de la rotation des logs journalier du service HAPROXY


### 1- Ajouter les lignes suivante dans le ficher /etc/haproxy/haproxy.cfg

```
global
    log 127.0.0.1 local0 info
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
```

### 2-Ajouter ces lignes dans le fichiers /etc/rsyslog.conf

```
# Collect log with UDP
$ModLoad imudp
$UDPServerAddress 127.0.0.1
$UDPServerRun 514

# Creating separate log files based on the severity
local0.* /var/log/haproxy-traffic.log
local0.notice /var/log/haproxy-admin.log
```

### 3- Crée le fichiers /etc/logrotate.d/haproxy avec le contenu ci-dessous

```
/var/log/haproxy.log {
    daily
    rotate 10
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
    postrotate
        /bin/kill -HUP `cat /var/run/haproxy.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
```

### 4-Redemarrage des services

```
systemctl restart haproxy.service
systemctl restart rsyslog
```

### 5- verification des config
```
sudo logrotate -d /etc/logrotate.conf
sudo logrotate -f /etc/logrotate.conf
ls -ltrh /var/log/haproxy*
```




tester le fontionnement avec les commandes

sudo logrotate -f /etc/logrotate.conf

ls -ltrh /var/log/haproxy*
