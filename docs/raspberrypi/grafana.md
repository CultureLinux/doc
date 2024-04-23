# InfluxDB
## install
    # curl https://repos.influxdata.com/influxdata-archive.key | gpg --dearmor | tee /usr/share/keyrings/influxdb-archive-keyring.gpg >/dev/null
    # echo "deb [signed-by=/usr/share/keyrings/influxdb-archive-keyring.gpg] https://repos.influxdata.com/debian stable main" | tee /etc/apt/sources.list.d/influxdb.list
    
    # apt update
    # apt install influxdb2

    # systemctl unmask influxdb
    # systemctl enable influxdb
## access
    http://<IPADDRESS>:8086/

# Grafana
## install
    # mkdir -p /etc/apt/keyrings/
    # wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | tee /etc/apt/keyrings/grafana.gpg > /dev/null
    # echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | tee /etc/apt/sources.list.d/grafana.list

    # apt update
    # apt install -y grafana

    # systemctl enable grafana-server
    # systemctl start grafana-server
## access
    http://<IPADDRESS>:3000/    