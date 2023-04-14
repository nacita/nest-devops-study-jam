
#############################
## 0. Persiapan LAB        ##
#############################

# pada LAB ini kita akan menggunakan 2 node, yaitu node1 & node2
# node1 digunakan sebagai target monitoring, 
# node2 digunakan sebagai monitoring server

# pada tiap screenshot, ganti X dengan nama masing-masing


###############################
## 1. Install Node Exporter  ##
###############################

## Jalankan di node1

##1. Download Paket Node Exporter
cd /opt
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz

##2. Start Node Exporter
cd node_exporter-1.5.0.linux-amd64
./node_exporter --help
./node_exporter --version
./node_exporter

##3. Gunakan browser dan akses metrics:
#- http://ip-node1:9100/metrics

#A> Full screenshot kedua metrics tersebut. Beri nama X-nest-mon-A.png

##4. Running Node Exporter as a Service
vi /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter

[Service]
User=root
ExecStart=/opt/node_exporter-1.5.0.linux-amd64/node_exporter

[Install]
WantedBy=default.target


##5. Start Node Exporter Service
systemctl daemon-reload
systemctl enable node_exporter.service
systemctl start node_exporter.service
systemctl status node_exporter.service
journalctl -u node_exporter


##################################
## 2. Install Prometheus Server ##
##################################

### Eksekusi di node node2

##1. Download Paket Prometheus Server
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
tar xvfz prometheus-2.43.0.linux-amd64.tar.gz

##2. Konfigurasi Prometheus Server
cd prometheus-2.43.0.linux-amd64
vi config.yml

global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus-[username]'
    static_configs:
    - targets: ['ip-node2:9090']
  - job_name: 'node-[username]'
    static_configs:
    - targets: ['ip-node1:9100','ip-node-lain:9100']

##3. Start Prometheus Server
./promtool check config config.yml
./prometheus --help
./prometheus --version
./prometheus --config.file=/opt/prometheus-2.43.0.linux-amd64/config.yml

##4. Gunakan browser dan akses url berikut:
#- Metrics: http://ip-node2:9090/metrics
#- Graph: http://ip-node2:9090/
#- Target: http://ip-node2:9090/targets

##5. Running Prometheus Server as a Service
vi /etc/systemd/system/prometheus_server.service

[Unit]
Description=Prometheus Server

[Service]
User=root
ExecStart=/opt/prometheus-2.43.0.linux-amd64/prometheus --config.file=/opt/prometheus-2.43.0.linux-amd64/config.yml --web.external-url=http://ip-node2:9090/

[Install]
WantedBy=default.target

##6. Start Prometheus Server Service
systemctl daemon-reload
systemctl enable prometheus_server.service
systemctl start prometheus_server.service
systemctl status prometheus_server.service
journalctl -u prometheus_server


#########################
##### Latihan Query #####
#########################

##1. Mendapatkan status (up/down) dari target.
#- Pada form expression ketikkan: up
#- Klik button Execute
#- Perhatikan pada tab console kolom element dan value. Value 1 berarti target dalam kondisi up.
#- Eksekusi satu per satu metrics yang ada dalam kolom element.

##2. Mendapatkan uptime node1.
#- Pada form expression ketikkan: (time() - process_start_time_seconds{instance="ip-node1:9100",job="node-[username]"})
#- Klik button Execute
#- Perhatikan pada tab console kolom element dan value. Value berisi uptime target dalam detik.
#- Dapatkan uptime node1.

##3. Mendapatkan persentase rata-rata penggunaan CPU di node1 dalam 5 menit terakhir.
#- Pada form expression ketikkan: 100 - avg (irate(node_cpu_seconds_total{instance="ip-node1:9100",job="node-[username]",mode="idle"}[5m])) by (instance) * 100
#- Klik button Execute
#- Klik tab Graph. Centang stacked.
#B> Full screenshot dan beri nama X-nest-mon-B.png

##4. Mendapatkan persentase penggunaan memory di node0 dan node1.
#- Pada form expression ketikkan: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_memory_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100
#- Klik button Execute
#- Klik tab Graph. Centang stacked.
#C> Full screenshot dan beri nama X-nest-mon-C.png

##5. Masih banyak query metrics yang bisa dieksekusi. Silahkan di-explore secara mandiri.


###########################
## 3. Install Grafana   ###
###########################

##### Eksekusi di node2 #####

##1. Download Paket Grafana
cd /opt
wget https://dl.grafana.com/oss/release/grafana-9.2.15.linux-amd64.tar.gz
tar -zxvf grafana-9.2.15.linux-amd64.tar.gz

##2. Start Grafana
cd grafana-9.2.15
screen -R grafana
./bin/grafana-server web

##3. Gunakan browser dan akses http://ip-node2:3000/login:
#D> Full screenshot form login Grafana dan beri nama X-nest-mon-D.png

# keluar dari screen dengan Ctrl+c, lalu ketik 
exit

##4. Running Grafana as a service
vi /etc/systemd/system/grafana.service

[Unit]
Description=Grafana

[Service]
User=root
ExecStart=/opt/grafana-9.2.15/bin/grafana-server -homepath /opt/grafana-9.2.15/ web

[Install]
WantedBy=default.target

##5. Start Grafana Service
systemctl daemon-reload
systemctl enable grafana.service
systemctl start grafana.service
systemctl status grafana.service
journalctl -u grafana

##6. Menambahkan Data Source
#- Gunakan browser dan akses Dashboard Grafana: http://ip-node2:3000/login
#- Configuration > Data Sources > Add data source
#- Name: Prometheus-[username]
#- Type: Prometheus
#- Version: 2.40.x
#- URL: http://ip-node2:9090
#- Save & Test
#- Pastikan mendapat pesan: Data source is working


############################
##### Create Dashboard #####
############################

##### Akses ke dashboard Grafana dan buat dashboard untuk node1 #####

##1. Buat Panel: Status
#- Create > Dashboard > Add > SingleStat
#- Klik Panel Title > Edit
#- Tab General > Title: Status
#- Tab Metrics > Data Sources: Prometheus-[username], Metrics: up{instance="ip-node1:9100",job="node-[username]"}
#- Tab Options > Stat: Current
#- Tab Value Mappings > Set value mappings: 1 -> Up, 0 -> Down
#- Save Dashboard > New name: node1

##2. Buat Panel: Uptime
#- Dashboard node1 > Add Panel > SingleStat
#- Klik Panel Title > Edit
#- Tab General > Title: Uptime
#- Tab Metrics > Data Sources: Prometheus-[username], Metrics: (time() - process_start_time_seconds{instance="ip-node1:9100",job="node-[username]"})
#- Tab Options > Stat: Current, Unit: Seconds
#- Save Dashboard

##3. Buat Panel: CPU Core
#- Dashboard node1 > Add Panel > SingleStat
#- Klik Panel Title > Edit
#- Tab General > Title: CPU Core
#- Tab Metrics > Data Sources: Prometheus-[username], Metrics: count(count(node_cpu_seconds_total{instance="ip-node1:9100",job="node-[username]"}) without (mode)) by (instance)
#- Tab Options > Stat: Current, Unit: none
#- Save Dashboard

##4. Buat Panel: Memory
#- Dashboard node1 > Add Panel > SingleStat
#- Klik Panel Title > Edit
#- Tab General > Title: Memory
#- Tab Metrics > Data Sources: Prometheus-[username], Metrics: node_memory_MemTotal_bytes{instance="ip-node1:9100",job="node-[username]"}
#- Tab Options > Stat: Current, Unit: none
#- Save Dashboard

##5. Buat Panel: Memory Used
#- Dashboard node1 > Add Panel > Graph
#- Klik Panel Title > Edit
#- Tab General > Title: Memory Used
#- Tab Metrics > Data Sources: Prometheus-[username], Metrics: (node_memory_MemTotal_bytes{instance="ip-node1:9100",job="node-[username]"} - (node_memory_MemFree_bytes{instance="ip-node1:9100",job="node-[username]"} + node_memory_Cached_bytes{instance="ip-node1:9100",job="node-[username]"} + node_memory_Buffers_bytes{instance="ip-node1:9100",job="node-[username]"})) / node_memory_MemTotal_bytes{instance="ip-node1:9100",job="node-[username]"} * 100
#- Tab Axes > Unit: percent (0-100)
#- Tab Legend > Show (✔), As Table (✔), Current (✔)
#- Save Dashboard

##6. Buat Panel: CPU Used
#- Dashboard node1 > Add Panel > Graph
#- Klik Panel Title > Edit
#- Tab General > Title: CPU Used
#- Tab Metrics > Data Sources: Prometheus-[username], Metrics: 100 - (avg by (instance) (irate(node_cpu_seconds_total{instance="ip-node1:9100",job="node-[username]",mode="idle"}[5m])) * 100)
#- Tab Axes > Unit: percent (0-100)
#- Tab Legend > Show (✔), As Table (✔), Current (✔)
#- Save Dashboard

#E> Full screenshot dashboard node1 dan beri nama X-nest-mon-E.png