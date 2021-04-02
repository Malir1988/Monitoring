# Prom-Grafana-Monitoring
## Indroduction
This git provides a step by step proccess of full chainlink monitoring & alerting including:
- Prometheus-server with TLS & Basic-Auth
- Prometheus-node-exporter with TLS & Basic-Auth
- Grafana with TLS & Basic-Auth
- Loki & Promtail
- Full monitoring chainlink Dashboard
- setting Alerts and send notifications to Telegram 
## Pre-information
- For the example setup we use the created docker network: "kovan". Every container needs to be in the same network as the chainlink node to ensure the communication between all of them.
- For creating the files we used `nano` but you can also easily create the files with `vim`
- You need to copy the files of this github inside of your system. Just copy the sourcecode inside after you created the file by following the guide.
## Create Directorys
first of all you need to create the Directorys for all necessary files.
```bash
mkdir ~/.monitoring
mkdir ~/.monitoring/.tls
mkdir ~/.monitoring/.tls/.prometheus
mkdir ~/.monitoring/.tls/.grafana
mkdir ~/.monitoring/.tls/.node-exporter
```
## TLS-certificates
The TLS certificates are created via openssl and saved on the created directorys
### Prometheus
```bash
cd ~/.monitoring/.tls/.prometheus && openssl req -x509 -out   ~/.monitoring/.tls/.prometheus/prometheus.crt  -keyout  ~/.monitoring/.tls/.prometheus/prometheus.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
### Node-exporter
```bash
cd ~/.monitoring/.tls/.node-exporter && openssl req -x509 -out   ~/.monitoring/.tls/.node-exporter/node-exporter.crt  -keyout  ~/.monitoring/.tls/.node-exporter/node-exporter.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
### Grafana
```bash
cd ~/.monitoring/.tls/.grafana && openssl req -x509 -out   ~/.monitoring/.tls/.grafana/grafana.crt  -keyout  ~/.monitoring/.tls/.grafana/grafana.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
## Authentication
A .htpasswd file is used for protecting the password of Prometheus credentials using HTTP authentication and implemented using rules within a .htaccess file.
 ### install HTPASSWD
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
  
### Node-exporter auth
  ```bash
  htpasswd -nBC 10 "" | tr -d ':\n'
  ```
  you need to save this value for the exporterweb.yml
## Node Exporter
### create web.yml
```bash
cd ~/.monitoring && nano exportweb.yml
```
copy there inside the exportweb.yml of the git and just change the username of the basic-auth and the HTPASWD-token
### run node-exporter
```bash
cd ~/.monitoring && docker run -d -p 9100:9100 --name node-exporter --restart unless-stopped --network kovan -v "/:/hostfs" -v /home/<USER>/.monitoring/exporterweb.yml:/hostfs/web.yml -v /home/<USER>/.monitoring/.tls/node-exporter.key:/tls/node-exporter.key -v /home/<USER>/.monitoring/.tls/node-exporter.crt:/tls/node-exporter.crt prom/node-exporter --path.rootfs=/hostfs --web.config=/hostfs/web.yml
```
 You need to change the <USER> to your Username you gain access. This will point the initialisation to the created and necessary files and directorys.
 ## Prometheus-server
 
 ### create web.yml
 ```bash
 cd ~/.monitoring && nano prometheusweb.yml
 ```
 ### create prometheus.yml
    ```bash
    cd ~/.monitoring && nano prometheus.yml
    ``` 
 ### run prometheus-server
  ```bash
  cd ~/.monitoring && sudo docker run --name prometheus --network kovan --restart=unless-stopped -d -p 9090:9090 -v /home/<USER>/.monitoring/prometheus.yml:/etc/prometheus/prometheus.yml -v /home/<USER>/.monitoring/.tls/prometheus.key:/tls/prometheus.key -v /home/<USER>/.monitoring/.tls/prometheus.crt:/tls/prometheus.crt -v /home/<USER>/.monitoring/prometheusweb.yml:/etc/prometheus/web.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.config.file=/etc/prometheus/web.yml
   ```
 You need to change the <USER> to your Username you gain access. This will point the initialisation to the created and neccassery files and directorys.
  
 To check if Prometheus scrapes all metrics your need to check your targets on the prometheus GUI: https://localhost:9090/targets
 
![s6_Prometheus targets](https://user-images.githubusercontent.com/77073086/113423325-935fb080-93ce-11eb-9e2d-ba3401b2ea41.JPG)

 ## Loki
  
### create loki.yml
    ```bash
    cd ~/.monitoring && nano loki.yml
    ```
### run Loki
     ```bash
    cd ~/.monitoring && sudo docker run -d -p 3100:3100 --name loki --network kovan --restart unless-stopped -v /home/<USER>/.monitoring/loki.yml:/mnt/config/loki.yml grafana/loki:2.2.0 -config.file=/mnt/config/loki.yml
    ```
 ## Promtail
 
### create promtail.yml
    ```bash
    cd ~/.monitoring && nano promtail.yml
    ```
### run promtail
    ```bash
    cd ~/.monitoring && sudo docker run -d --name promtail --network kovan --restart unless-stopped -v /home/<USER>/.monitoring/promtail.yml:/mnt/config/promtail.yml -v /var/log:/var/log grafana/promtail:2.2.0 -config.file=/mnt/config/promtail.yml
    ```
 ## Grafana
 
### create default.ini
    ```bash
    cd ~/.monitoring && nano grafana.ini
    ```
### run Grafana
    ```bash
    cd ~/.monitoring && docker run -d -p 3000:3000 --name grafana --network kovan --restart unless-stopped -v /home/<USER>/.monitoring/.tls/.grafana/grafana.key:/tls/grafana.key -v /home/<USER>/.monitoring/.tls/.grafana/grafana.crt:/tls/grafana.crt -v /home/<USER>/.monitoring/grafana.ini:/etc/grafana/grafana.ini -e GF_PATHS_CONFIG=/etc/grafana/grafana.ini grafana/grafana:latest
    ```
 ## Datasource Integration
- open your Grafana GUI on your explorer https://localhost:3000
- you will be prompt to type your set username and password
- ADD a new Datasource
### Prometheus
- target: https://<PROMETHEUS_CONTAINER_ID>:9090
- enable: `Basic_Auth`, `Credentials` , `CA_Cert`, `Skip_TLS_VERIFY`
- `SAVE & TEST`

![s5_Prometheus datasource](https://user-images.githubusercontent.com/77073086/113423338-99559180-93ce-11eb-8426-ca2afcb764d8.JPG)

### LOKI
- target: http://<LOKI_CONTAINER_ID>:3100 
- `SAVE & TEST`

## Grafana Dashboards
 
 ![s4_Import dashboard](https://user-images.githubusercontent.com/77073086/113423336-99559180-93ce-11eb-97d6-e594f672e379.JPG)
 
### Chainlink Dashboard:           
- click on "Create" -> "Import" 
- "Import via panel JSON"
- paste the json of the dashboard.file in this git

 ![s1_Chainlink dashboard](https://user-images.githubusercontent.com/77073086/113423330-95297400-93ce-11eb-84f0-123b582e5296.png)
 
### Host Dashboard:                
- click on "Create" -> "Import"
- "Import via grafana.com"
- type in: 11952
- https://grafana.com/grafana/dashboards/11952

  ![s2_Host dashboard](https://user-images.githubusercontent.com/77073086/113423331-96f33780-93ce-11eb-8439-3c134eeabecd.png)
  
 ## Alerting
 
 ### Create a notification channel:
  - "Alerting" -> "Notification channels" -> "Add channel"
  - Name: Telegram
  - Type: Telegram
  - BOT API TOKEN: You need to create A BOT API Token https://medium.com/shibinco/create-a-telegram-bot-using-botfather-and-get-the-api-token-900ba00e0f39
  - Chat ID: get the Chat ID of your Telegram channel

### Alerting
You can create now alerts inside of your Dashboard. You can only set alerts on "graphs" as displayed metrics. 
     
For a list of important alerts for a running chainlink node please take a look on our security paper: https://linkriver.io/wp-content/uploads/2021/03/Chainlink_Node_Operations_Research_Paper.pdf

![s3_Alert-rules](https://user-images.githubusercontent.com/77073086/113423335-98bcfb00-93ce-11eb-8c1b-f6c7ff42821e.JPG)
