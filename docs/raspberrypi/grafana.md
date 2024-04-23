# InfluxDB

# Grafana
    # mkdir -p /etc/apt/keyrings/
    # wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | tee /etc/apt/keyrings/grafana.gpg > /dev/null
    # echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | tee /etc/apt/sources.list.d/grafana.list

    # apt update
    # apt install -y grafana

    # systemctl enable grafana-server
    # systemctl start grafana-server