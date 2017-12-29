## Quellen
<https://www.tutorialspoint.com/docker/index.htm>
<https://docs.docker.com/>

## Installation
+ Installations Anleitung: <https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/>
+ immer als root/sudo, man kann auch die Rechte für non-root Nutzer einrichten
+ `docker run hello-world` zum testen ob die Umgebung läuft (lädt kleines HelloWorld runter)
+ `docker login` zum einloggen (notwendig für push/pull von eigenen Repos)

The `run` command is used to mention that we want to create an instance of an image, which is then called a container. Finally, "hello-world" represents the image from which the container is made.

## Image
+ `docker images` zeigt alle Images an
+ `docker rmi ImageID` entfernt ein Image, wobei ImageID die ID unter `docker images` ist
+ `docker inspect hello-world` für Details eines Images
+ `docker history ImageID` zeigt alle Befehle, welche gegen das Image liefen
+ Container sind Instanzen von Images
    + mit `run` wird jedesmal ein neuer Container erzeugt (!)
    + `docker run –it <image> /bin/bash`
    + `-it` für Interactive Mode (erzeugt Bash shell in Container)
+ brauchbare Parameter:
    + siehe: <https://docs.docker.com/engine/reference/run/>
    + `-p 80:80` für die Portzuweisung vom Image zum Host (HOST:CONTAINER)
    + `-d` für Detach (um nicht in den Container zu "gehen")
    + `--name=YOURNAME` benennt den Container
    
## Container Kram
+ `docker ps` zeigt die aktuell laufenden Container
    + `-a` zeigt alle an
+ `docker start ContainerID` startet Container. 
    + Mit `run` wird jedesmal ein neuer Container erstellt. Irgendwann wird es voll in der Liste. Daher lieber einen alten Container starten.
    + mit Parameter `--attach` geht man in gleich in den Container rein
+ `docker stop ContainerID` stopt
+ `docker pause ContainerID` pausiert, `unpause` lässt weiterlaufen
+ `docker kill ContainerID` 
+ `docker rm ContainerID` löscht
+ `docker top ContainerID` zeigt + baut aus bestehenden Image, hier "ubuntutop-level Prozesse eines Containers
+ `docker stats ContainerID` zeigt CPU und RAM Initialisierung
+ `docker attach ContainerID` verbindet sich mit Container
    + `CTRL+P+Q` um Container zu verlassne, ohne ihn zu schließen
+ `docker exec -it <container_id_or_name> echo "Hello from container!"` führt Kommando innerhalb eines Containers aus
    + dieser muss aber laufen
+ `service docker stop` stoppt Docker Daemon

## Erstellen eines Docker Images
+ einfache Text Datei erstellen:

Beispiel #1
+ baut aus bestehenden Image, hier "ubuntu"
```docker
#This is a sample Image
FROM ubuntu 
MAINTAINER demousr@gmail.com 

RUN apt-get update 
RUN apt-get install –y nginx 
CMD [“echo”,”Image created”]
```

Beispiel #2
+ für minimale oder gar neue Images, wird `FROM scratch` genutzt 
+ [SCRATCH Docker Hub](https://hub.docker.com/_/scratch/)
```docker
FROM scratch
COPY hello /
CMD ["/hello"]
```
+ `docker build  -t ImageName:TagName dir`
    + `-t` für ein Tag
    + ImageName: der Name
    + TagName: Tagname
    + Dir: wo das Dockerfile (siehe oben) ist

## Upload zum HUB
+ Online ein Repo erstellen
    + ein private möglich in kostenloser Version
    + für push muss man sich mit `docker login` anmelden
    + eigener Server/HUB möglich: <https://www.tutorialspoint.com/docker/docker_private_registries.htm>
+ taggen für Repo Upload
    + `docker tag <ImageID> demousr/demorep:1.0`
    + erstellt eine "kopie" vom Image. Der Name muss mit dem Repo übereinstimmen
+ 'docker push demousr/demorep:1.0' pusht es
+ `docker pull demousr/demorep:1.0` holt es wieder runter

## Ports
+ Container Ports müssen zu Host Ports gemappt werden
+ nach dem Format `HOSTPORT:CONTAINERPORT`
+ Beispiel: `docker run -p 8080:8080 -p 50000:50000 jenkins`

## Container verbinden
+ um nicht alles über Ports zu realisieren
+ `--link` ist eine Möglichkeit, ist aber "deprecated"
+ .... TBC ...
 
## Netzwerk
+ Docker hat einen eigenen Netzwerkadapter auf dem Host: `docker0`
+ `docker network ls` listet alle Netzwerke im Docker Host
+ `docker network inspect <NETWORKNAME>` für mehr Details eines Netzwerks (z.B. `bridge`)
+ beim Start eines Containers, wird dieser mit `bridge` Adapter verbunden

Eigenes Netzwerk erstellen:
+ `docker network create –-driver drivername name`
+ Beispiel: `docker network create –-driver bridge new_nw`
+ Container beim erstellen mit Netzwerk verbinden
    + `docker run –it –-network=new_nw ubuntu:latest /bin/bash`
+ ... TBC ....

## Docker Compose
+ [Installations Anleitung](https://docs.docker.com/compose/install/)
+ [Kommandos](https://docs.docker.com/compose/reference/)
+ lädt mehrere Container als einzelnen Service
+ Composer Files in YAML > `docker-compose.yml`
+ starten mit `docker-compose up`
    + sucht im aktuellen Ordner nach einer compose YAML und führt sie aus