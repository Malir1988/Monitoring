# Prom-Grafana-Monitoring
## Create Directory / TLS-certificates / Authentication
```bash
mkdir .monitor
mkdir ~/.monitor/.tls
mkdir ~/.monitor/.tls/.prometheus
mkdir ~/.monitor/.tls/.grafana
mkdir ~/.monitor/.tls/.node-exporter
```
1) Create the certificates
Prometheus
```bash
cd ~/.monitor/.tls/.prometheus && openssl req -x509 -out   ~/.monitor/.tls/.prometheus/prometheus.crt  -keyout  ~/.monitor/.tls/.prometheus/prometheus.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
Node-exporter
```bash
cd ~/.monitor/.tls/.node-exporter && openssl req -x509 -out   ~/.monitor/.tls/.node-exporter/node-exporter.crt  -keyout  ~/.monitor/.tls/.node-exporter/node-exporter.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
Grafana
```bash
cd ~/.monitor/.tls/.grafana && openssl req -x509 -out   ~/.monitor/.tls/.grafana/grafana.crt  -keyout  ~/.monitor/.tls/.grafana/grafana.key -newkey rsa:2048 -nodes -sha256 -days 365 -subj '/CN=localhost' -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
2) Authentication
  install HTPASSWD
  ```bash
  yum install httpd-tools
  ```
  or
  ```bash
  sudo apt-get install httpd-tools
  ```
  Prometheus auth
  ```bash
  htpasswd -nBC 10 "" | tr -d ':\n'
  ```
  you need to save this value for the web.yml of prometheus
  
  Node-exporter auth
  ```bash
  htpasswd -nBC 10 "" | tr -d ':\n'
  ```
  you need to save this value for the web.yml of node-exporter
## Node Exporter
1) create web.yml
    ```bash
    cd ~/.monitoring && nano exportweb.yml
    ```
    copy there inside the exportweb.yml of the git and just change the username of the basic-auth
 2) create node-exorter.yml
