# prometheus installation

## l'ip de notre serveur est 192.168.1.35
Ajout du user prometheus
``` useradd --no-create-home --shell /bin/false prometheus ```

Creation des repertoire
* etc prometheus
``` mkdir /etc/prometheus ```

* etc prometheus
``` mkdir /var/lib/prometheus ```

* donner mes droits
``` chown prometheus:prometheus /var/lib/prometheus/ ```
* Telechargement de prometheus
``` cd /tmp/ ```

``` wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz ```

* extraire les données
``` tar -xvf prometheus-2.53.4.linux-amd64.tar.gz ```
* dans le repertoire deplacer les fichiers
``` cd prometheus-2.53.4.linux-amd64/ ```

``` mv console* /etc/prometheus/ ```

``` mv prometheus.yml /etc/prometheus/ ```

``` chown -R prometheus:prometheus /etc/prometheus/ ```

``` mv prometheus /usr/local/bin/ ```

``` mv promtool /usr/local/bin/ ```

``` chown prometheus:prometheus /usr/local/bin/prom* ```

``` vi /etc/systemd/system/prometheus.service ```

ajouter les donnees ci-dessous dans le fichier 

``` 
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

``` 
Demarrer le service prometheus et verifier son status 

```
systemctl start prometheus.service 
systemctl status prometheus.service 

```

ouvrir prometheus via le lien  http://192.168.1.35:9090


![image](https://github.com/user-attachments/assets/934c57c6-0075-438a-a90b-d3dbfd7af796)

