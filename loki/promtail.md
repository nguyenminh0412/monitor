# promtail is agent log => deploy on target host

sudo wget -qO /opt/promtail/promtail.gz "https://github.com/grafana/loki/releases/download/v2.4.1/promtail-linux-amd64.zip"
sudo gunzip /opt/promtail/promtail.gz
sudo chmod a+x /opt/promtail/promtail
sudo ln -s /opt/promtail/promtail /usr/local/bin/promtail
promtail -version

sudo tee /etc/systemd/system/promtail.service<<EOF
[Unit]
Description=Promtail client for sending logs to Loki
After=network.target

[Service]
ExecStart=/opt/promtail/promtail -config.file=/opt/promtail/promtail.yaml
Restart=always
TimeoutStopSec=3

[Install]
WantedBy=multi-user.target
EOF

sudo tee /opt/promtail/promtail.yaml<<EOF
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://192.160.59.10:3100/loki/api/v1/push

scrape_configs:
- job_name: MongoDB_logs
  static_configs:
  - targets:
      - 192.160.59.10
    labels:
      job: 	nginx
      __path__: /var/log/nginx/*.log
EOF

sudo systemctl enable promtail
sudo service promtail restart 
sudo systemctl status promtail
