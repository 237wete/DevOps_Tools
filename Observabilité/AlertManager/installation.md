# AlertManager installation

## l'ip de notre serveur est 192.168.1.35
Ajout du user alertmanager
``` useradd --no-create-home --shell /bin/false alertmanager ```

Creation des repertoire
* etc alertmanager
``` mkdir /etc/alertmanager ```

* Telechargement de alertmanager
``` cd /tmp/ ```

``` wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-amd64.tar.gz ```

* extraire les données
``` tar -xvf alertmanager-0.28.1.linux-amd64.tar.gz ```
* dans le repertoire deplacer les fichiers
``` cd alertmanager-0.28.1.linux-amd64/ ```

``` mv alertmanager /usr/local/bin/ ```

``` mv amtool /usr/local/bin/ ```

``` chown alertmanager:alertmanager /usr/local/bin/alertmanager ```

``` chown alertmanager:alertmanager /usr/local/bin/amtool ```

``` mv alertmanager.yml /etc/alertmanager ```

``` chown -R alertmanager:alertmanager /etc/alertmanager ```

``` vi /etc/systemd/system/alertmanager.service ```

ajouter les donnees ci-dessous dans le fichier 

``` 
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager \
    --config.file /etc/alertmanager/alertmanager.yml

[Install]
WantedBy=multi-user.target

``` 
Demarrer le service alertmanager et verifier son status 

```
systemctl daemon-reload
systemctl start alertmanager.service 
systemctl status alertmanager.service 

```
ne pas oublier d'activer alertmanager dans le fichier de configuration prometheus.yml 

ouvrir prometheus via le lien  http://192.168.1.35:9093

