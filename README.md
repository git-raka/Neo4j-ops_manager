# Neo4j-ops_manager
## NOM SERVER INSTALLATION
Download the ops manager
```
wget https://dist.neo4j.org/ops-manager/1.7.2/neo4j-ops-manager-server-1.7.2-unix.tar.gz
tar -xvf neo4j-ops-manager-server-1.7.2-unix.tar.gz
```
go to ops manager directory and create the pfx cert,but its not recommended for production
change the -i with your cidr ip neo4j
```
java -jar ./lib/server.jar ssc -n localhost \
	-o ./certificates \
	-p changeit \
	-d nom.example.com \
	-i 192.168.10.1,172.16.10.1
```
create system service
```
sudo nano /etc/systemd/system/ops.service
```
append this
```
[unit]
Description=Neo4j Ops Manager Server
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/neo4j/ops/bin/server \
    --spring.neo4j.uri=neo4j://gds.pegadaian.co.id:7687 \
    --spring.neo4j.authentication.username=ops.manager \
    --spring.neo4j.authentication.password=Gadai123! \
    --server.port=8888 \
    --server.ssl.key-store-type=PKCS12 \
    --server.ssl.key-store=file:/home/neo4j/ops/certificates/pegadaian.pfx \
    --server.ssl.key-store-password=Gadai123! \
    --grpc.server.port=9095 \
    --grpc.server.security.key-store-type=PKCS12 \
    --grpc.server.security.key-store=file:/home/neo4j/ops/certificates/pegadaian.pfx \
    --grpc.server.security.key-store-password=Gadai123! \
Restart=on-failure
TimeoutSec=120
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="LOGGING_FILE_NAME=/home/neo4j/ops/logs/output.log"
[Install]
WantedBy=multi-user.target
```
After that you can start the ops.service
```
sudo systemctl daemon-reload
sudo systemctl start ops
```

## AGENT INSTALLATION
go to each vm with already installed neo4j for deploying agent
```
sudo nano /etc/hosts
```
And add all instance private ip and domain
```
10.10.0.2 neo4jn1.pegadaian.co.id
10.10.0.3 neo4jn2.pegadaian.co.id
10.10.0.4 neo4jn3.pegadaian.co.id
10.10.0.7 gds.pegadaian.co.id
```
Edit neo4j conf and restart after that
```
metrics.prometheus.endpoint=127.0.0.1:2004
metrics.prometheus.enabled=true
metrics.enabled=true
metrics.filter=*
metrics.jmx.enabled=true
metrics.namespaces.enabled=true
```
Download latest neo4j ops manager Agent 
```
wget "https://dist.neo4j.org/ops-manager/1.7.2/neo4j-ops-manager-agent-1.7.2-linux-amd64.tar.gz?_ga=2.198499112.605132663.1692460519-1427712525.1690905495" -O ops.tar.gz
tar -xvf ops.tar.gz
```
Move or copy agent in bin directory of ops and install ( create systemd service )
```
sudo mv agent /usr/local/bin/.
sudo /usr/local/bin/agent  service install
```
After running service install, they will automaticly create .service in /etc/systemd/system
```
sudo nano /etc/systemd/system/neo4j-ops-manager-agent.service
```
Before we go,make sure you have created an agent to get the client_id and client_secret 
see [agent installation](https://neo4j.com/docs/ops-manager/current/addition/agent-installation/).
if you already create agent next step is,Create Like this in `neo4j-ops-manager-agent.service`
```
[Unit]
Description=Agent for Neo4j Operations Manager
ConditionFileIsExecutable=/usr/local/bin/agent


[Service]
StartLimitInterval=5
StartLimitBurst=10
ExecStart=/usr/local/bin/agent "service"
Environment="CONFIG_SERVER_ADDRESS=neo4jn3.pegadaian.co.id:9090"
Environment="CONFIG_TOKEN_URL=https://neo4jn3.pegadaian.co.id:8080/api/login/agent"
Environment="CONFIG_TOKEN_CLIENT_ID=01a31780-43b8-4c8a-92f9-a1d3b9eb923c"
Environment="CONFIG_TOKEN_CLIENT_SECRET=gSFQZ3P3PqXQbTh5Ir7b22GesVKk9idhTvuFtLM3LimUoeMRtv9Rh30ru9hmMLid"
Environment="CONFIG_TLS_TRUSTED_CERTS=/home/neo4j/cert/neo4j.cert"
Environment="CONFIG_INSTANCE_1_NAME=Neo4j_Cluster_2"
Environment="CONFIG_INSTANCE_1_BOLT_URI=bolt://localhost:7687"
Environment="CONFIG_INSTANCE_1_BOLT_USERNAME=ops.manager"
Environment="CONFIG_INSTANCE_1_BOLT_PASSWORD=Gadai123!"
Environment="LOGGING_FILE_NAME=/home/neo4j/agent.log"
Restart=always
RestartSec=120
EnvironmentFile=-/etc/sysconfig/neo4j-ops-manager-agent

[Install]
WantedBy=multi-user.target
```
after that start
```
sudo systemctl daemon-reload
sudo systemctl start neo4j-ops-manager-agent.service
```

