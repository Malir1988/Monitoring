
# Prom-Grafana-Monitoring
## Indroduction
in this github we provide a step by step proccess of full chainlink monitoring & alerting including:
- Prometheus-server with TLS & Basic-Auth
- Prometheus-node-exporter with TLS & Basic-Auth
- Grafana with TLS & Basic-Auth
- Loki & Promtail
- Full monitoring chainlink Dashboard
- setting Alerts to Telegram 
## Pre-information
For the example setup we use the created docker network: "kovan"
## Create Directorys
first of all you need to create the Directorys for all neccasery files.
```bash
mkdir ~/.monitoring
mkdir ~/.monitoring/.tls
mkdir ~/.monitoring/.tls/.prometheus
mkdir ~/.monitoring/.tls/.grafana
mkdir ~/.monitoring/.tls/.node-exporter
```
## TLS-certificates
The TLS certificates are created via openssl and saved on the created directorys
1) Prometheus
```bash
cd ~/.monitoring/.tls/.prometheus && openssl req -x509 -out   ~/.monitoring/.tls/.prometheus/prometheus.crt  -keyout  ~/.monitoring/.tls/.prometheus/prometheus.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
2) Node-exporter
```bash
cd ~/.monitoring/.tls/.node-exporter && openssl req -x509 -out   ~/.monitoring/.tls/.node-exporter/node-exporter.crt  -keyout  ~/.monitoring/.tls/.node-exporter/node-exporter.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
3) Grafana
```bash
cd ~/.monitoring/.tls/.grafana && openssl req -x509 -out   ~/.monitoring/.tls/.grafana/grafana.crt  -keyout  ~/.monitoring/.tls/.grafana/grafana.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
## Authentication
A .htpasswd file is used for protecting the password of Prometheus credentials using HTTP authentication and implemented using rules within a .htaccess file.
  1) install HTPASSWD
  ```bash
  yum install httpd-tools
  ```
  or
  ```bash
  sudo apt-get install httpd-tools
  ```
  2) Prometheus auth
  ```bash
  htpasswd -nBC 10 "" | tr -d ':\n'
  ```
  you need to save this value for the web.yml of prometheus
  
  3) Node-exporter auth
  ```bash
  htpasswd -nBC 10 "" | tr -d ':\n'
  ```
  you need to save this value for the web.yml of node-exporter
## Node Exporter
1) create web.yml
    ```bash
    cd ~/.monitoring && nano exportweb.yml
    ```
    copy there inside the exportweb.yml of the git and just change the username of the basic-auth and the HTPASWD-token
 2) run node-exporter
    ```bash
    cd ~/.monitoring && docker run -d -p 9100:9100 --restart unless-stopped --network kovan -v "/:/hostfs" -v /home/<USER>/.monitoring/exporterweb.yml:/hostfs/web.yml -v /home/<USER>/.monitoring/.tls/node-exporter.key:/tls/node-exporter.key -v /home/<USER>/.monitoring/.tls/node-exporter.crt:/tls/node-exporter.crt prom/node-exporter --path.rootfs=/hostfs --web.config=/hostfs/web.yml
    ```
    You need to change the <USER> to your Username you gain access. This will point the initialisation to the created and neccassery files and directorys.
 ## Prometheus-server
  
 1) create web.yml
 ```bash
 cd ~/.monitoring && nano prometheusweb.yml
 ```
 2) create prometheus.yml
    ```bash
    cd ~/.monitoring && nano prometheus.yml
    ``` 
 3) run prometheus-server
  ```bash
  cd ~/.monitoring && sudo docker run --name prometheus --network kovan --restart=unless-stopped -d -p 9090:9090 -v /home/<USER>/.monitoring/prometheus.yml:/etc/prometheus/prometheus.yml -v /home/<USER>/.monitoring/.tls/prometheus.key:/tls/prometheus.key -v /home/<USER>/.monitoring/.tls/prometheus.crt:/tls/prometheus.crt -v /home/<USER>/.monitoring/prometheusweb.yml:/etc/prometheus/web.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.config.file=/etc/prometheus/web.yml
   ```
 You need to change the <USER> to your Username you gain access. This will point the initialisation to the created and neccassery files and directorys.
 ## Loki
  
 1) create loki.yml
    ```bash
    cd ~/.monitoring && nano loki.yml
    ```
 2) run Loki
     ```bash
    cd ~/.monitoring && sudo docker run -d -p 3100:3100 --name loki --network kovan --restart unless-stopped -v /home/<USER>/.monitoring/loki.yml:/mnt/config/loki.yml grafana/loki:2.2.0 -config.file=/mnt/config/loki.yml
    ```
 ## Promtail
 1) create promtail.yml
    ```bash
    cd ~/.monitoring && nano promtail.yml
    ```
 2) run promtail
    ```bash
    cd ~/.monitoring && sudo docker run -d --name promtail --network kovan --restart unless-stopped -v /home/<USER>/.monitoring/promtail.yml:/mnt/config/promtail.yml -v /var/log:/var/log grafana/promtail:2.2.0 -config.file=/mnt/config/promtail.yml
    ```
 ## Grafana
 
 1) create default.ini
    ```bash
    cd ~/.monitoring && nano default.ini
    ```
 2) run Grafana
    ```bash
    cd ~/.monitoring && docker run -d -p 3000:3000 --name grafana --network kovan --restart unless-stopped -v /home/<USER>/.monitoring/.tls/.grafana/grafana.key:/tls/grafana.key -v /home/<USER>/.monitoring/.tls/.grafana/grafana.crt:/tls/grafana.crt -v /home/<USER>/.monitoring/grafana.ini:/etc/grafana/grafana.ini -e GF_PATHS_CONFIG=/etc/grafana/grafana.ini grafana/grafana:latest
    ```
 ## Datasource Integration
  1) visit on your explorer https://localhost:3000
  2) you will be prompt to type your set username and password
  3) ADD a new Datasource
  4) Prometheus: You need to add:  - target: https://<PROMETHEUS_CONTAINER_ID>:9090
                                   - enable: Basic Auth, Credentials, CA_Cert, Skip_TLS_VERIFY
                                   - asdasd
  5) LOKI:                         - target: http://<LOKI_CONTAINER_ID>:3100 
  6) LOKI as Prometheus datasource: - target: http://<LOKI_CONTAINER_ID>:3100 
 ## Chainlink Dashboard
 
 ## Alerting
  
