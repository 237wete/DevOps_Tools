
# Grafana Installation

![image](https://github.com/user-attachments/assets/697de0e4-e7f4-4130-b443-250a5eaad232)

## Import the GPG key

``` 
wget -q -O gpg.key https://rpm.grafana.com/gpg.key
sudo rpm --import gpg.key
``` 

Create ```  /etc/yum.repos.d/grafana.repo ```  with the following content:


```
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

To install Grafana OSS, run the following command

```
sudo dnf install grafana
```
systemctl enable grafana-server.service
systemctl start grafana-server.service

Acceder à l'application grafana via le lien http://192.168.1.35:3000
login: admin
password: admin

![image](https://github.com/user-attachments/assets/18ff7a0c-2edd-4626-aed8-1ffcc147f9f2)
