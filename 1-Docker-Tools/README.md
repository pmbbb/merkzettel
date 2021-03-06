# Compose
## Linux
### Anforderungen:
- [Docker 1.9+](https://docs.docker.com/installation)
- [Docker-Compose 1.5.1+](https://docs.docker.com/compose/install)
- Kernel 3.16+

### Installation Docker-Machine (Linux)

```Bash
# Zuerst zu dem user root wechseln
su
curl -L https://github.com/docker/compose/releases/download/1.5.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose && \
  chmod +x /usr/local/bin/docker-compose
exit
# Test ob docker-compose korrekt installiert wurde
docker-compose version
```

### Beispiel
Um das Beispiel lokal zu starten, muss sich der Anwender in dem Ordner mit diesem Beispiel befinden.

Im ersten Schritt erstellen wir das benötigte Netzwerk und starten danach die Container.

```Bash
sudo docker network create todoapp_network
sudo docker-compose up -d
```

Zeige alle Container an

```Bash
sudo docker-compose ps
sudo docker network inspect todoapp_network
```

Ausgabe des Ports der ToDo Anwendung

```Bash
sudo docker-compose port todoApp 3000
0.0.0.0:32768
```

Wir können diesen Port einfach verwenden und in einem beliebigen Browser localhost:32768 eingeben.

Jetzt können wir zwei neue ToDo Container erstellen

```Bash
sudo docker-compose scale todoApp=3
```

Nun können wir uns die Ports der drei Instanzen ausgeben lassen. Diese können entsprechend in dem Browser aufgerufen werden. Werden nun zwei Browserfenster gleichzeitig geöffnet und dabei unterschiedliche Ports verwendet, können wir sehen, dass die App Änderungen von einem Fenster in das andere übernimmt.

```Bash
sudo docker-compose port --index=1 todoApp 3000
sudo docker-compose port --index=2 todoApp 3000
sudo docker-compose port --index=3 todoApp 3000
```

## OS X und Windows
### Anforderungen:
- [Docker Toolbox](https://www.docker.com/docker-toolbox)

### Beispiel
Um das Beispiel lokal zu starten, muss sich der Anwender in dem Ordner mit diesem Beispiel befinden.

Zu erst müssen wir schauen ob unsere virtuelle Maschine schon am Laufen ist. Wenn diese noch nicht gestartet wurde starten wir diese:

```Bash
docker-machine status default
# Bei der Ausgabe: "Running" läuft die Machine schon
docker-machine start default
```

Im nächsten Schritt erstellen wir das benötigte Netzwerk und starten danach die Container.

```Bash
docker network create todoapp_network
docker-compose up -d
```

Zeige alle Container an

```Bash
docker-compose ps
docker network inspect todoapp_network
```

Ausgabe des Ports der ToDo Anwendung

```Bash
docker-compose port todoApp 3000
0.0.0.0:32768
```

Nun benötigen wir noch die IP-Addresse der virtuellen Maschine:

```Bash
docker-machine ip default
192.168.99.100
```

Wir können diesen Port und die IP einfach verwenden und in einem beliebigen Browser 192.168.99.100:32768 eingeben.

Jetzt können wir zwei neue ToDo Container erstellen

```Bash
docker-compose scale todoApp=3
```

Nun können wir uns die Ports der drei Instanzen ausgeben lassen. Diese können entsprechend in dem Browser aufgerufen werden. Werden nun zwei Browserfenster gleichzeitig geöffnet und dabei unterschiedliche Ports verwendet, können wir sehen, dass die App Änderungen von einem Fenster in das andere übernimmt.

```Bash
docker-compose port --index=1 todoApp 3000
docker-compose port --index=2 todoApp 3000
docker-compose port --index=3 todoApp 3000
```

## Compose v2
- [Docker 1.10+](https://docs.docker.com/installation)
- [Docker-Compose 1.6+](https://docs.docker.com/compose/install)
- Kernel 3.16+

### Beispiel
Um das Beispiel lokal zu starten, muss sich der Anwender in dem Ordner mit diesem Beispiel befinden.

Im ersten Schritt starten wir alle Container.

```Bash
sudo docker-compose -f docker-compose_v2.yml up -d
```

Zeige alle Container an

```Bash
sudo docker-compose -f docker-compose_v2.yml ps
sudo docker network inspect todoapp_network
```

Ausgabe des Ports der ToDo Anwendung

```Bash
sudo docker-compose -f docker-compose_v2.yml port todoApp 3000
0.0.0.0:32768
```

Wir können diesen Port einfach verwenden und in einem beliebigen Browser localhost:32768 eingeben.

Jetzt können wir einen neuen Redis Slave Container erstellen

```Bash
sudo docker-compose -f docker-compose_v2.yml scale redis-slave=2
```

Nun können wir in einem neuen Terminalfenster, mithilfe von watch, alle 500ms von dem frontend Container eine DNS Anfrage auf den Redis Slave ausführen.

```Bash
watch -n 0.5 sudo docker exec $(sudo docker ps -f name=1_docker_tools_todoApp_1 -q) getent hosts  redis-slave
```

Im nächsten Schritt können wir den ersten Redis Slave beenden. Wenn wir nun wieder in das Terminalfenster der DNS Abfrage schauen sehen wir die IP-Addresse des Redis Slave 2.

```Bash
sudo docker kill $(sudo docker ps -f name=1_docker_tools_redis-slave_1 -q)
```

# Swarm
## Anforderungen
- [Docker 1.9+](https://docs.docker.com/installation)
- [Docker-Machine](https://docs.docker.com/machine/install-machine)
- [Virtualbox](https://www.virtualbox.org)

oder
- [Docker Toolbox](https://www.docker.com/docker-toolbox)

## Installation Docker-Machine (Linux)

```Bash
# Zuerst zu dem user root wechseln
su
curl -L https://github.com/docker/machine/releases/download/v0.5.6/docker-machine_linux-amd64 >/usr/local/bin/docker-machine && \
  chmod +x /usr/local/bin/docker-machine
exit
# Test ob docker-machine korrekt installiert wurde
docker-machine version
```

## Vorkonfiguration

```Bash
docker-machine create \
    -d virtualbox \
    cluster-store

docker $(docker-machine config cluster-store) run -d \
    -p "8500:8500" \
    -h "consul" \
    progrium/consul -server -bootstrap -ui-dir /ui
```

## Erstellung des Swarm-Clusters mit Docker-Machine
Erstellung des Swarm Masters

```Bash
docker-machine create \
    --driver virtualbox \
    --swarm \
    --swarm-master \
    --swarm-discovery="consul://$(docker-machine ip cluster-store):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip cluster-store):8500" \
    --engine-opt="cluster-advertise=eth1:0" \
    swarm-master
```

Erstellung von zwei Swarm-Nodes

```Bash
docker-machine create \
    --driver virtualbox \
    --swarm \
    --swarm-discovery="consul://$(docker-machine ip cluster-store):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip cluster-store):8500" \
    --engine-opt="cluster-advertise=eth1:0" \
    swarm-node-00

docker-machine create \
    --driver virtualbox \
    --swarm \
    --swarm-discovery="consul://$(docker-machine ip cluster-store):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip cluster-store):8500" \
    --engine-opt="cluster-advertise=eth1:0" \
    swarm-node-01
```

Mit diesem Befehl überprüfen wir die Erstellung des Clusters

```Bash
eval $(docker-machine env --swarm swarm-master)
docker info
```

Nun erstellen wir abschließend noch ein Overlay Netzwerk, so dass die Hosts auch miteinander kommunizieren können.

```Bash
docker network create todoapp_network
docker $(docker-machine config swarm-master) network ls
```

Dank des gemeinsamen Key-Value Store Consul ist das Netzwerk auf allen Nodes des Swarm Clusters verfügbar

```Bash
docker $(docker-machine config swarm-node-00) network ls
docker $(docker-machine config swarm-node-01) network ls
```

## Beispiel
Die Erstellung des Redis-Master Containers

```Bash
docker run -d -p 6379:6379 --name=redis-master --net=todoapp_network redis
```

Nun können wir den Redis-Slave erstellen

```Bash
docker run -d -p 6379:6379 --name=redis-slave --net=todoapp_network johscheuer/redis-slave:v1
```

Bei dem Versuch einen dritten Redis-Slave zu starten erhalten wir folgenden Fehler: "Error response from daemon: unable to find a node with port 6379 available". Welcher uns mitteilt, dass es keine Node in dem Swarm-Cluster gibt, die die Anforderung (Port 6379 frei) erfüllt verfügbar ist.

Abschließend kann die ToDo-App gestartet werden.

```Bash
docker run -d -p 3000:3000 --net=todoapp_network johscheuer/todo-app-web:v1
```

Es ist zu beachten, dass es in dem Swarm Cluster nur ein Container mit einem eindeutigen Namen geben darf. Würden man versuchen einen zweiten Redis Slave zu starten, würde dies zu einem Fehler führen.

```Bash
docker run -d -p 6379:6379 --name=redis-slave --net=todoapp_network johscheuer/redis-slave:v1
```

## Compse + Swarm + Networking
Wir können nun das Compose Beispiel auch auf dem Swarm Cluster starten. Hierfür entfernen wir die zuvor gestarteten Container.

```Bash
docker rm -f $(docker ps -q)
```

Nun können wir mit dem gewohnten Docker Compose Commando unsere Anwendung starten

```Bash
docker-compose up -d
docker-compose ps
```
