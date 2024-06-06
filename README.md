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




