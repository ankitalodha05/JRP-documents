Here is a well-structured and professional document for your EC2 monitoring setup using **Prometheus**, **Node Exporter**, and **Grafana** on AWS.

---

# 📊 EC2 Server Monitoring with Prometheus and Grafana

## 🧾 Assumptions

* You are monitoring **two EC2 Amazon Linux servers** (`server1` and `server2`) using a **third Ubuntu EC2 server** (`monitoring-server`).
* Amazon Linux repositories may have limitations; hence the monitoring tools are installed on Ubuntu.
* All servers have public IPs and SSH access is available.

---

## 🖥️ Server Details

| Role                | OS           | Public IP (example) | Description               |
| ------------------- | ------------ | ------------------- | ------------------------- |
| `server1`           | Amazon Linux | 192.168.1.10        | Application Node 1        |
| `server2`           | Amazon Linux | 192.168.1.20        | Application Node 2        |
| `monitoring-server` | Ubuntu       | 192.168.1.30        | Prometheus & Grafana Host |

---

## 🧩 Step 1: Install Prometheus on `monitoring-server`

### 🔹 Download & Install

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.48.0/prometheus-2.48.0.linux-amd64.tar.gz
tar xvf prometheus-2.48.0.linux-amd64.tar.gz
sudo mv prometheus-2.48.0.linux-amd64 /opt/prometheus
```

### 🔹 Configure Prometheus

```bash
sudo nano /opt/prometheus/prometheus.yml
```

Paste the following config (replace with your real public IPs):

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'ec2_servers'
    static_configs:
      - targets: ['192.168.1.10:9100']
        labels:
          instance: 'server1'
      - targets: ['192.168.1.20:9100']
        labels:
          instance: 'server2'
```

### 🔹 Start Prometheus

```bash
sudo /opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml &
```

### 🔹 Verify Prometheus

Open in browser:
[http://192.168.1.30:9090](http://192.168.1.30:9090)

---

## 📦 Step 2: Install Node Exporter on `server1` and `server2`

### 🔹 Download & Install

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
```

### 🔹 Start Node Exporter

```bash
sudo /usr/local/bin/node_exporter &
```

### 🔹 Verify

Visit:

* [http://192.168.1.10:9100/metrics](http://192.168.1.10:9100/metrics)
* [http://192.168.1.20:9100/metrics](http://192.168.1.20:9100/metrics)

You should see metric outputs.

---

## 📈 Step 3: Install Grafana on `monitoring-server`

### 🔹 Install Dependencies and Grafana

```bash
sudo apt-get update 
sudo apt-get install -y software-properties-common 

sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main" 
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add - 

sudo apt-get update 
sudo apt-get install grafana 

sudo systemctl start grafana-server 
sudo systemctl enable grafana-server
```

### 🔹 Access Grafana

Visit:
[http://192.168.1.30:3000](http://192.168.1.30:3000)
Login with default credentials:
**Username:** `admin`
**Password:** `admin`

---

## ⚙️ Step 4: Configure Grafana

### 🔸 Add Prometheus as Data Source

1. Click the **Gear Icon → Data Sources**.
2. Click **Add data source → Prometheus**.
3. Set URL: `http://192.168.1.30:9090`
4. Click **Save & Test**. It should show "Data source is working."

### 🔸 Create a Dashboard

1. Click **+ → Dashboard → Add new panel**.
2. In Query Editor, choose Prometheus as data source.
3. Use a query like:

   ```
   node_cpu_seconds_total{mode="idle"}
   ```
4. Set a title (e.g., "CPU Usage - Server 1").
5. Click **Apply** and then **Save Dashboard**.
6. Repeat to add more metrics.

---

## 🎯 Final Notes

* Prometheus scrapes metrics from **Node Exporter** every 15 seconds.
* Grafana visualizes the collected metrics.
* This setup enables **real-time performance monitoring** of your EC2 servers.

---

Would you like this exported as a **PDF**, **Word**, or **Markdown** document for sharing or training use?
