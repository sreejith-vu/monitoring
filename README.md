# monitoring
sudo useradd --no-create-home --shell /bin/false prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo mkdir /etc/prometheus
 sudo mkdir /var/lib/prometheus
 sudo chown prometheus:prometheus /etc/prometheus
 sudo chown prometheus:prometheus /var/lib/prometheus
 curl -LO https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
 sha256sum prometheus-2.0.0.linux-amd64.tar.gz
 tar xvf prometheus-2.0.0.linux-amd64.tar.gz
 
  cp prometheus-2.0.0.linux-amd64/prometheus /usr/local/bin/
 cp prometheus-2.0.0.linux-amd64/promtool /usr/local/bin/
 
  chown prometheus:prometheus /usr/local/bin/prometheus
 chown prometheus:prometheus /usr/local/bin/promtool
 cp -r prometheus-2.0.0.linux-amd64/consoles /etc/prometheus
 cp -r prometheus-2.0.0.linux-amd64/console_libraries /etc/prometheus
 chown -R prometheus:prometheus /etc/prometheus/consoles
 chown -R prometheus:prometheus /etc/prometheus/console_libraries
 
 sudo nano /etc/prometheus/prometheus.yml
 sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
 
 sudo -u prometheus /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries
 sudo nano /etc/systemd/system/prometheus.service
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
 sudo systemctl daemon-reload
 sudo systemctl start prometheus
 sudo systemctl status prometheus
 curl -L localhost:9090

curl -LO https://github.com/prometheus/node_exporter/releases/download/v0.15.1/node_exporter-0.15.1.linux-amd64.tar.gz
 tar xvf node_exporter-0.15.1.linux-amd64.tar.gz
 cp node_exporter-0.15.1.linux-amd64/node_exporter /usr/local/bin
 sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
 sudo nano /etc/systemd/system/node_exporter.service
 ```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
 ```
 systemctl daemon-reload
 systemctl start node_exporter
 systemctl status node_exporter
 sudo nano /etc/prometheus/prometheus.yml
 ```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```
 
 systemctl restart prometheus
 systemctl status prometheus

apt-get install nginx

cp /etc/nginx/sites-available/default /etc/nginx/sites-available/prometheus
ln -s /etc/nginx/sites-available/prometheus /etc/nginx/sites-enabled/
 nginx -t
 systemctl reload nginx
 systemctl status nginx
 
 
 wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
 sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
 sudo apt-get update
 sudo apt-get install grafana
 sudo apt-get install -y apt-transport-https
 sudo service grafana-server start
 curl -L localhost:3000
 
 vi /etc/nginx/sites-available/prometheus

```
# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		        auth_basic "Prometheus server authentication";
		        auth_basic_user_file /etc/nginx/.htpasswd.prometheus;
		        proxy_pass http://localhost:9090;
		        proxy_http_version 1.1;
		        proxy_set_header Upgrade $http_upgrade;
		        proxy_set_header Connection 'upgrade';
		        proxy_set_header Host $host;
		        proxy_cache_bypass $http_upgrade;
	}


  	location /grafana/ {
		        auth_basic "Prometheus server authentication";
		        auth_basic_user_file /etc/nginx/.htpasswd.grafana;
   			proxy_pass http://localhost:3000/;
			proxy_set_header Host $host;
   			proxy_set_header X-Real-IP $remote_addr;
   			proxy_set_header X-Forwarded-Host $host;
   			proxy_set_header X-Forwarded-Server $host;
   			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

```
root_url = http://localhost:3000/grafana/
```
```
allow_sign_up = true
```
