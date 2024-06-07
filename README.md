# Prometheus-Gmail-Website
Description: Monitoring system checking if website or Ngeinx service is down. If it is down, then notify via gmail. 

Architecture:

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/242ce554-decc-4ca7-8174-2d024537d05f)


Technologies: Prometheus, Prometheus node exporter, Prometheus BlackBox Exporter, AWS, gmail, webiste, nginx
- **Prometheus BlackBox Exporter** - Prometheus BlackBox Exporter is a tool used in the Prometheus ecosystem that allows external monitoring and testing of various network services. It acts as an exporter that performs various checks and tests (probes) on a given target, and then transmits the results of these tests to Prometheus in the form of metrics. Here are some key features and uses of BlackBox Exporter:
- **Node Exporter** - Prometheus Node Exporter is a tool used in the Prometheus ecosystem to monitor system metrics from machines (nodes) in a server environment. Node Exporter collects and exports data about the state of the operating system, such as statistics on CPU, memory, disks, network and other system resources.

On **AWS** create 2 instances of EC2 - medium ubuntu. Create SecurityGroup with open ports 25, 3000-10000, 80, 443, 587, 22, 465, 27017

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/3c3e6ba5-ad07-4cb1-9a9d-d8d62b2da4da)

Go to https://prometheus.io/download/ and copy link for prometheus for Linux

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/3cd8dd1f-9959-47fb-bcc5-a330d5e2b0cc)

**Prometheus/BlackBoxExporter/Alerting** on this machine
```
sudo apt update
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xvf prometheus-2.52.0.linux-amd64.tar.gz
rm prometheus-2.52.0.linux-amd64.tar.gz
mv prometheus-2.52.0.linux-amd64/ prometheus
```
Go to https://prometheus.io/download/  again and copy link for blackbox_exporter for Linux

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/b44282f0-9973-456c-98dd-ef3f8114dc0b)

```
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
rm blackbox_exporter-0.25.0.linux-amd64.tar.gz
mv blackbox_exporter-0.25.0.linux-amd64/ blackbox_exporter
```
Go to https://prometheus.io/download/  again and copy link for AlertManager for Linux

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/565f261a-44de-46fe-9526-5363432f313f)


```
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz
rm alertmanager-0.27.0.linux-amd64.tar.gz
mv alertmanager-0.27.0.linux-amd64/ alertmanager
```

After ls -l you should see this

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/78f47b40-2254-46d0-9e20-c4faff24beb1)


go to https://github.com/jaiswaladi246/Boardgame and copy link to clone this repo. Go to the **VM - Ngnix service** instnace
Also go to https://prometheus.io/download/ and copy node_exporter 

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/690e3c4f-8954-45f0-8772-0d0bde81d048)

```
sudo apt update
git clone https://github.com/jaiswaladi246/Boardgame.git
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz
rm node_exporter-1.8.1.linux-amd64.tar.gz
mv node_exporter-1.8.1.linux-amd64/ node_exporter
```

Start Node exporter service in background

```
cd node_exporter/
./node_exporter &
```

press enter and now it is running in background. It is running on port 9100 

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/95c58ecb-9137-4dee-82f9-2d0523171682)

Type IP of you instance in browser with port :9100 You can click "Metrics" to check if it is running ok, you should see just some metrics

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/f34da9a6-d902-4fca-bbf4-fe4532586b55)

Start aplication (the repo). to run this application we need java and Maven to buid it.

```
cd /home/ubuntu/Boardgame
java
```

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/1b26297f-a815-43bd-b2c3-c29f096a1914)

```
sudo apt install openjdk-17-jre-headless
sudo apt install maven -y
mvn package
cd target
java -jar database_service_project-0.0.2.jar
```
Go to YouIP:8080, you  should see 

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/4e3f03c1-32ef-41ad-a4e9-dd94e0855b35)

Back to Prometheus instance

```
cd cd p
cd prometheus/
./prometheus &
```

Go to YourIP:9090 Then you can go Status->Rules, ane there should not be any ruls cuz they are not any alerting ruls yet

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/e3fe90e3-eefb-4fae-891f-0b0bf2d33531)


Setting up Alert Manager

```
cd alertmanager/
cat alertmanager.yml
```
In alertmanager.yml you can set up email to which u receive notification
![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/d0e27440-e56e-46c6-b289-88f09178ae69)


Setingup Alerting ruls

```
cd /home/ubuntu/prometheus
nano alert_rules.yaml
```

Paste in in alert_rules.yaml, after you copied it you can check if the syntax for yaml is correct here https://www.yamllint.com/

```
groups:
- name: alert_rules                   # Name of the alert rules group
  rules:
    - alert: InstanceDown
      expr: up == 0                   # Expression to detect instance down
      for: 1m
      labels:
        severity: "critical"
      annotations:
        summary: "Endpoint {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

    - alert: WebsiteDown
      expr: probe_success == 0        # Expression to detect website down
      for: 1m
      labels:
        severity: critical
      annotations:
        description: The website at {{ $labels.instance }} is down.
        summary: Website down

    - alert: HostOutOfMemory
      expr: node_memory_MemAvailable / node_memory_MemTotal * 100 < 25  # Expression to detect low memory
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host out of memory (instance {{ $labels.instance }})"
        description: "Node memory is filling up (< 25% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostOutOfDiskSpace
      expr: (node_filesystem_avail{mountpoint="/"} * 100) / node_filesystem_size{mountpoint="/"} < 50  # Expression to detect low disk space
      for: 1s
      labels:
        severity: warning
      annotations:
        summary: "Host out of disk space (instance {{ $labels.instance }})"
        description: "Disk is almost full (< 50% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostHighCpuLoad
      expr: (sum by (instance) (irate(node_cpu{job="node_exporter_metrics",mode="idle"}[5m]))) > 80  # Expression to detect high CPU load
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host high CPU load (instance {{ $labels.instance }})"
        description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: ServiceUnavailable
      expr: up{job="node_exporter"} == 0  # Expression to detect service unavailability
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Service Unavailable (instance {{ $labels.instance }})"
        description: "The service {{ $labels.job }} is not available\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HighMemoryUsage
      expr: (node_memory_Active / node_memory_MemTotal) * 100 > 90  # Expression to detect high memory usage
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "High Memory Usage (instance {{ $labels.instance }})"
        description: "Memory usage is > 90%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: FileSystemFull
      expr: (node_filesystem_avail / node_filesystem_size) * 100 < 10  # Expression to detect file system almost full
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "File System Almost Full (instance {{ $labels.instance }})"
        description: "File system has < 10% free space\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

Edit prometheus.yaml

```
nano prometheus.yml
```
Modify like this, because we already created fime called alert_rules.yaml

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/7e46f8f2-94b7-4be0-8933-db6d358678a6)

Restart Prometheus to see the alert rules from alert_rules.yaml

```
cd /home/ubuntu/prometheus
pgrep prometheus
kill 1490
./prometheus &
```

Go to AlertManager and run it

```
cd alertmanager/
./alertmanager &
```
As you can see Alert Manager is running on port 9093 so go to browser and check it
![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/f7fa0e4f-e38d-4bdf-b4bd-fe850442b7c9)

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/5b2dcdd9-b83a-4ba7-9269-8667208bdf5d)

Go to prometheus "Status"->"Rules" now you should see the rules and thier state

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/5e7344f3-8cc3-43a4-9f35-2f9ceb7502b5)

Configure Black Exporter 

```
vim prometheus.yml
```

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/0b1eacaa-b212-4c0d-bec9-20cf436e322d)

copy that 
```
  - job_name: "node_exporter"         # Job name for node exporter

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["3.110.195.114:9100"]  # Target node exporter endpoint
```
![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/3ccd2aa5-f3a7-409b-9ac4-3a641031acf9)

Go to https://www.yamllint.com/ and paste it and click "Go". this should pops out, copy it and past in at the end of prometheus.yml

```
- job_name: node_exporter
  static_configs:
    - targets:
        - YourIpForNodeExporter:9100
```

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/db5a50e8-0922-4eee-9eb2-712e1e1c72e8)

restart Prometheus 

```
cd /home/ubuntu/prometheus
pgrep prometheus
kill PID
./prometheus &
```

Go to Prometheus Status->Targets, now you should see 

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/f659eefe-0852-4791-a70a-8d3d509d0ff2)

Adding BlackBox_Exporter to targets. Go to https://prometheus.io/download/#blackbox_exporter click in the link which is pointed on screen https://github.com/prometheus/blackbox_exporter find section "Prometheus Configuration" and copy example config

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/da9f2c01-acb7-43d6-b55b-78292a0cf587)

Example config

```
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://example.com:8080 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
  - job_name: 'blackbox_exporter'  # collect blackbox exporter's operational metrics.
    static_configs:
      - targets: ['127.0.0.1:9115']
```

Go to https://www.yamllint.com/ use it as previously
the output

```
scrape_configs:
  - job_name: blackbox
    metrics_path: /probe
    params:
      module:
        - http_2xx
    static_configs:
      - targets:
          - http://prometheus.io
          - https://prometheus.io
          - http://example.com:8080
    relabel_configs:
      - source_labels:
          - __address__
        target_label: __param_target
      - source_labels:
          - __param_target
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115
  - job_name: blackbox_exporter
    static_configs:
      - targets:
          - 127.0.0.1:9115

```

You need to modify this part (remove scrap config and the --- on the beggining)

```
  - job_name: blackbox
    metrics_path: /probe
    params:
      module:
        - http_2xx
    static_configs:
      - targets:
          - http://prometheus.io
          - https://prometheus.io
          - http://IpInstanceWithService:8080
    relabel_configs:
      - source_labels:
          - __address__
        target_label: __param_target
      - source_labels:
          - __param_target
        target_label: instance
      - target_label: __address__
        replacement: IpWithBlackBoxExporter:9115
```

Paste it into prometheus.yml

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/4d981d83-9295-4388-8288-4f55355a69ea)

restart Prometheus 

```
cd /home/ubuntu/prometheus
pgrep prometheus
kill PID
./prometheus &
```
Now if you check Prometheus status->targets blackbox exporter is present 

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/6ef9ead7-527e-4595-bc4b-f2ef36846285)

Why they are "DOWN"? because BlackBox Exporter is not started yet. Starting BlackBoxExporter

```
cd /home/ubuntu/blackbox_exporter
./blackbox_exporter &
```

run in your browser on port :9100

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/3d99fefb-52c1-4ebe-af39-c803388ed938)

Now it is running and getting information from the Instance on which is the Application. If you check prometheus targets, now they are UP

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/24473ded-a96c-43ca-818b-fb5d8d907d88)

=====Setting up the mail notification=============

```
cd /home/ubuntu/alertmanager
rm alertmanager.yml
```

Copy this routning configuration 

```
route:
  group_by: ['alertname']             # Group by alert name
  group_wait: 30s                     # Wait time before sending the first notification
  group_interval: 5m                  # Interval between notifications
  repeat_interval: 1h                 # Interval to resend notifications
  receiver: 'email-notifications'     # Default receiver

receivers:
- name: 'email-notifications'         # Receiver name
  email_configs:
  - to: jaiswaladi246@gmail.com       # Email recipient
    from: test@gmail.com              # Email sender
    smarthost: smtp.gmail.com:587     # SMTP server
    auth_username: your_email         # SMTP auth username
    auth_identity: your_email         # SMTP auth identity
    auth_password: "bdmq omqh vvkk zoqx"  # SMTP auth password
    send_resolved: true               # Send notifications for resolved alerts
inhibit_rules:
  - source_match:
      severity: 'critical'            # Source alert severity
    target_match:
      severity: 'warning'             # Target alert severity
    equal: ['alertname', 'dev', 'instance']  # Fields to match
```

Paste it into https://www.yamllint.com/ the output

```
route:
  group_by:
    - alertname
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: email-notifications
receivers:
  - name: email-notifications
    email_configs:
      - to: your_email
        from: test@gmail.com
        smarthost: smtp.gmail.com:587
        auth_username: your_email
        auth_identity: your_email
        auth_password: app password
        send_resolved: true
inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal:
      - alertname
      - dev
      - instance

```

You have to generate "auth_password". You have to have 2 step verification on your gmail account. create new file with the content 

```
nano alertmanager.yml
```

restart alert manager

```
pgrep alertmanager
kill PID
./alertmanager &
```

restart alert prometheus

cd /home/ubuntu/prometheus
pgrep prometheus
kill PID
./prometheus &
```

Now yo ucan check Alerts in Prometheus, no laert is trigged  everything is working fine. 

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/d8fd066e-1143-421b-9aad-4934067bf949)

Lets stop the Service for app, go to the correct isntance, this app is working on port 8080, checkid the PID and killing it

```
sudo lsof -i :8080
kill PID
```

Checking the Prometheus alerts, we can see that alert responsible for checking if website is down is now status "PENDING" after 1min it will be "FIRING"
![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/0af5cc38-0e6c-44c0-844e-0537583f8adf)

![image](https://github.com/jeti20/Prometheus-Gmail-Website/assets/61649661/65aca3dd-2e66-422f-9450-975bb00d9e6d)

also you should receive email about this alert



=======PROBLEMS I FACED====
After taking a brake fro mthis project and then returned to it I had to start the node_exporter as well as the application. Aplication works fine, but I cannot display the nodexporter through IP:9100. Node exporteer was runnig fine. Checked if it is running: 
```
sudo netstat -tuln | grep :9100
sudo lsof -i :9100
sudo kill 1112
./node_exporter &
```
I killed the process and started it once again. It turned out that I pasted wrong IP in the browser :)


AlertManager 9093
Prometheus 9090
Aplication 8080
NodeExporter 9100
BlackBoxExporter 9115
