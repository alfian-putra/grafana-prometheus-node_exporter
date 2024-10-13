# Linux Server Monitoring Menggunakan Grafana & Prometheus

Pada project ini kita akan melakukan monitoring Linux Server (Fedora) menggunakan Stack sebagai berikut :

- Node Exporter

Node Exporter adalah exporter metrics adalah tools yang digunakan untuk mengumpulkan data dari server kemudian yang pada akhirnya data tersebut dapat diakses menggunakan API end point oleh metrics aggregator yang akan mengolah data tersebut.

- Prometheus

Dari web resmi Prometheus : Prometheus® is an open source monitoring system developed by engineers at SoundCloud in 2012. Prometheus was the second project accepted into the Cloud Native Computing Foundation after Kubernetes, and also the second to graduate.

The Prometheus monitoring system includes a rich, multidimensional data model, a concise and powerful query language called PromQL, an efficient embedded time series database, and over 150 integrations with third-party systems.

Grafana Labs is proud to support the development of the Prometheus project by employing Prometheus maintainers, building first-class integration with Prometheus into Grafana, and ensuring Grafana customers receive Prometheus features they need.

- Grafana

Berfungsi sebagai frontend yang bertugas visualisasi, query dan alerting data yang disediakan Prometheus. Grafana menyediakan fleksibilitas untuk customize dashboard serta alerting ke berbagai sumber sepertio telegram, slack, dan email.

## Environment

Pada test kali ini environnment yang akan kita gunakan adalah sebagai berikut :

- Linux Fedora 40
- Node Exporter 1.8.2
- Prometheus 2.25.0
- Grafana 10.2..6

## Instalasi

Kita akan menempatkan semua tools yang akan kita install di /opt :
```
cd /opt
```
- Node Exporter

1. Download Node Exporter.
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz 
```
2. extract kemudian jalankan
```
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
./node_exporter-1.8.2.linux-amd64/node_exporter
```
![node exporter](./6.0.%20Node%20Exporter.png)

3. Untuk mempermudah management node exporter ke depannya kita akan menggunakan systemd :
```
vi /etc/systemd/system/node_exporter.service
```
Tuliskan syntax sebagai beriukut :
```
[Unit]
Description=Node Exporter

[Service]
Type=simple
ExecStart=/opt/node_exporter-1.8.2.linux-amd64/node_exporter                                                           
```
Jalankan perintah sebagai berikut :
```
systemctl daemon-reload # untuk reload / update service yang baru saja ditambahkan
systemctl start node_exporter
systemctl status node_exporter
```

![](./6.0.%20Node%20Exporter%20_%20systemctl.png)

- Prometheus

1. Download tarbal dari web resmi Prometheus ;

```
wget https://github.com/prometheus/prometheus/releases/download/v2.25.0/prometheus-2.25.0.linux-amd64.tar.gz
```

2. extracttarbal yang telah di-download :

```
tar -xvf prometheus-2.25.0.linux-amd64.tar.gz
```

3. Configure Prometheus :

```
vi ./
```

Pastikan bagian di bawah ini sudah sesuai :

scrape_configs:
```
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['<NODE-EXPORTER-SERVER_IP>:9100']
```

4. Jalankan Prometheus :

./prometheus-2.25.0.linux-amd64/prometheus --config.file=./prometheus-2.25.0.linux-amd64/prometheus.yml

![](./6.0.%20Prometheus%20.png)

5. Selanjutnya kita akan menambahkan prometheus.service ke systemd
```
vi /etc/systemd/system/prometheus.service
```
kemudian tambahkan syntax berikut
```
[Unit]
Description=Prometheus

[Service]
Type=simple
ExecStart=/opt/prometheus-2.25.0.linux-amd64/prometheus --config.file=/opt/prometheus-2.25.0.linux-amd64/prometheus.yml
```

![](./6.0.%20Prometheus%20_%20systemctl.png)

- Grafana

1. install grafana-server :
```
dnf update
dnf install -y grafana-server
```
2. start grafana
```
systemctl start grafana-server
```

__NOTE :__ config file grafana ada di /etc/grafana.ini

4. Login ke <GRAFANA_SERVER>:3000 menggunakan admin:admin, kemudian akan diminta untuk mengganti password.

5. Configure Datasource di Home > Connections > Data sources

![](./6.0.%20Grafana%20_%203.%20config%20datasource.png)

save

![](6.0.%20Grafana%20_%204.%20config%20save.png)

6. kemudian kita akan menggunakan dashboard yang yang dapat kita temukan [disini](https://grafana.com/grafana/dashboards/1860-node-exporter-full/).

7. Import Grafana dashboard. di Home > Dashboards > + Create Dashboard > import. kemudian masukan json yang telah kita download.

![](./6.0.%20Grafana%20_%205.%20import%20dashboard.png)

Berikut adalah hasilnya :

![](./6.0.%20Grafana%20_%206.%20FINAL.png)

## Error yang Ditemui

1. firewall error

![](./6.1.%20ERR%20firewall.png)

Kita dapat mendiagnosa error ini jika saat akses ke url dari luar (bukan local) terjadi error seperti gambar diatas. Yang dapat kita lakukan adalah dengan stop firewalld :
```
systemctl stop firewalld
```
atau allow port tersebut di firewalld :
```
firewall-cmd --permanent --add-port=<PORT_GRAFANA>/tcp
```
2. Port 9090 digunakan service lain

Di Fedora terdapat service bawaan cockpit yang menggunakan port yang sama seperti Prometheus. Untuk mengatasi masalah ini kita dapat mendisable service cockpit :
```
systemctl disable cockpit
```
atau mengganti port prometheus ke port lain.

3. selinux

![](./6.1.%20ERR%20selinux.png)

```
dial tcp [::1]:9090: connect: permission denied 
```

jika ditemui error seperti diatas saat configurasi datasource Grafana, maka yang dapat kita lakukan adalah disable selinux dengan cara :
```
vi /etc/selinux/config
```
kemudian ubah dari :
```
SELINUX=enforcing
```
menjadi
```
SELINUX=disabled
```
save, kemudian reboot.
