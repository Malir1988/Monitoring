# Chainlink Prometheus/Grafana TLS monitoring
## Indroduction
This documentation is a guide for full Chainlink node monitoring and alerting including the following deployments:
- Prometheus server with TLS & basic-auth
- Prometheus node exporter with TLS & basic-auth
- Grafana with TLS & basic-auth
- Loki & Promtail 
- Full monitoring Chainlink dashboard 
- Alerts and Telegram notification setup
## Comment
- For the example deployment we used the created "Kovan" docker network. Every container needs to be in the same network like the Chainlink node to ensure the communication between them.
- For the creation of the files we used `nano`, you can also do it with `vim`
- You need to copy the files from this repository to your system. Just copy the source code after you've created the file by following the guide.
## Create directories
Create the directories for all necessary files
```bash
mkdir ~/.monitoring
mkdir ~/.monitoring/.tls
mkdir ~/.monitoring/.tls/.prometheus
mkdir ~/.monitoring/.tls/.grafana
mkdir ~/.monitoring/.tls/.node-exporter
```
## TLS certificates
The TLS certificates are created via openssl and stored in the created directories
### Prometheus
```bash
cd ~/.monitoring/.tls/.prometheus && openssl req -x509 -out   ~/.monitoring/.tls/.prometheus/prometheus.crt  -keyout  ~/.monitoring/.tls/.prometheus/prometheus.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
### Node exporter
```bash
cd ~/.monitoring/.tls/.node-exporter && openssl req -x509 -out   ~/.monitoring/.tls/.node-exporter/node-exporter.crt  -keyout  ~/.monitoring/.tls/.node-exporter/node-exporter.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
### Grafana
```bash
cd ~/.monitoring/.tls/.grafana && openssl req -x509 -out   ~/.monitoring/.tls/.grafana/grafana.crt  -keyout  ~/.monitoring/.tls/.grafana/grafana.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
## Authentication
A .htpasswd file is used for the protection of the Prometheus credentials using HTTP authentication and is implemented into a .htaccess file. 
 ### Install HTPASSWD
  ```bash
  yum install httpd-tools
  ```
  or
  ```bash
  sudo apt-get install httpd-tools
  ```
 ### Prometheus auth
  ```bash
  htpasswd -nBC 10 "" | tr -d ':\n'
  ```
  you need to save this value for the prometheusweb.yml
  
### Node exporter auth
```bash
htpasswd -nBC 10 "" | tr -d ':\n'
```
  you need to save this value for the exporterweb.yml
## Node exporter
### Create web.yml
```bash
cd ~/.monitoring && nano exportweb.yml
```
copy the code of the exportweb.yml and just change the username of the basic-auth and the HTPASWD token
### Run node-exporter
```bash
cd ~/.monitoring && docker run -d -p 9100:9100 --name node-exporter --restart unless-stopped --network kovan --user root -v "/:/hostfs" -v /home/<USER>/.monitoring/exporterweb.yml:/hostfs/web.yml -v /home/<USER>/.monitoring/.tls/node-exporter.key:/tls/node-exporter.key -v /home/<USER>/.monitoring/.tls/node-exporter.crt:/tls/node-exporter.crt prom/node-exporter --path.rootfs=/hostfs --web.config=/hostfs/web.yml
```
 You need to change the <USER> to your user name in order to gain access. This will point the initialisation to the created and required files and directories.
 ## Prometheus server
 
 ### Create web.yml
 ```bash
 cd ~/.monitoring && nano prometheusweb.yml
 ```
 ### Create prometheus.yml
 ```bash
 cd ~/.monitoring && nano prometheus.yml
 ``` 
 ### Run prometheus-server
 ```bash
 cd ~/.monitoring && sudo docker run --name prometheus --network kovan --restart=unless-stopped --user root -d -p 9090:9090 -v /home/<USER>/.monitoring/prometheus.yml:/etc/prometheus/prometheus.yml -v /home/<USER>/.monitoring/.tls/prometheus.key:/tls/prometheus.key -v /home/<USER>/.monitoring/.tls/prometheus.crt:/tls/prometheus.crt -v /home/<USER>/.monitoring/prometheusweb.yml:/etc/prometheus/web.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.config.file=/etc/prometheus/web.yml
  ```
 You need to change the <USER> to your user name in order to gain access. This will point the initialisation to the created and required files and directories.
  
 To check if Prometheus scrapes all metrics, you need to check your targets in the Prometheus GUI: `https://localhost:9090/targets`
 
![s6_Prometheus targets](https://user-images.githubusercontent.com/77073086/113423325-935fb080-93ce-11eb-9e2d-ba3401b2ea41.JPG)

 ## Loki
  
### Create loki.yml
```bash
cd ~/.monitoring && nano loki.yml
```
### Run Loki
```bash
cd ~/.monitoring && sudo docker run -d -p 3100:3100 --name loki --network kovan --restart unless-stopped -v /home/<USER>/.monitoring/loki.yml:/mnt/config/loki.yml grafana/loki:2.2.0 -config.file=/mnt/config/loki.yml
```
 ## Promtail
 
### Create promtail.yml
```bash
cd ~/.monitoring && nano promtail.yml
```
### Run promtail
```bash
cd ~/.monitoring && sudo docker run -d --name promtail --network kovan --restart unless-stopped --user root -v /home/<USER>/.monitoring/promtail.yml:/mnt/config/promtail.yml -v /var/log:/var/log -v /var/lib/docker:/var/lib/docker grafana/promtail:2.2.0 -config.file=/mnt/config/promtail.yml
```
 ## Grafana
 
### Create default.ini
```bash
cd ~/.monitoring && nano grafana.ini
```
### Run Grafana
```bash
cd ~/.monitoring && docker run -d -p 3000:3000 --name grafana --network kovan --restart unless-stopped --user root -v /home/<USER>/.monitoring/.tls/.grafana/grafana.key:/tls/grafana.key -v /home/<USER>/.monitoring/.tls/.grafana/grafana.crt:/tls/grafana.crt -v /home/<USER>/.monitoring/grafana.ini:/etc/grafana/grafana.ini -e GF_PATHS_CONFIG=/etc/grafana/grafana.ini grafana/grafana:latest
```
 ## Data source Integration
- Open your Grafana GUI in your explorer `https://localhost:3000`
- Fill in your `username` and `password`
- Add a new data source
### Prometheus
- Target: `https://<PROMETHEUS_CONTAINER_ID>:9090`
- Enable: `Basic_Auth`, `Credentials` , `CA_Cert`, `Skip_TLS_VERIFY`
- `SAVE & TEST`

![s5_Prometheus datasource](https://user-images.githubusercontent.com/77073086/113423338-99559180-93ce-11eb-8426-ca2afcb764d8.JPG)

### LOKI
- Target: `http://<LOKI_CONTAINER_ID>:3100` 
- `SAVE & TEST`

## Grafana dashboards
 
 ![s4_Import dashboard](https://user-images.githubusercontent.com/77073086/113423336-99559180-93ce-11eb-97d6-e594f672e379.JPG)
 
### Chainlink dashboard:           
- Click on `Create` -> `Import` 
- `Import via panel JSON`
- Paste the JSON code from this repo's dashboard.file 

![s1_chainlink_dashboard](https://user-images.githubusercontent.com/77073086/113481513-46491080-949a-11eb-8b58-909678625ea0.JPG)
 
### Host dashboard:                
- Click on `Create` -> `Import`
- `Import via grafana.com`
- Type in: `11952`
- https://grafana.com/grafana/dashboards/11952

  ![s2_Host dashboard](https://user-images.githubusercontent.com/77073086/113423331-96f33780-93ce-11eb-8439-3c134eeabecd.png)
  
 ## Alerting
 
 ### Create a notification channel:
  - "Alerting" -> "Notification channels" -> "Add channel"
  - Name: Telegram
  - Type: Telegram
  - BOT API TOKEN: You need to create A BOT API Token https://medium.com/shibinco/create-a-telegram-bot-using-botfather-and-get-the-api-token-900ba00e0f39
  - Chat ID: get the chat ID of your Telegram channel

### Alerting
You can now set alerts on your Dashboard. You can only set alerts on "graph-visualisations" as displayed metrics. 
     
For a list of important alerts for a running Chainlink node you can have a look at our security research paper: 
https://linkriver.io/wp-content/uploads/2021/03/Chainlink_Node_Operations_Research_Paper.pdf

![s3_Alert-rules](https://user-images.githubusercontent.com/77073086/113423335-98bcfb00-93ce-11eb-8c1b-f6c7ff42821e.JPG)
