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
