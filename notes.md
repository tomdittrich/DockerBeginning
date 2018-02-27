## Quellen & hilfreiche Links
<https://www.tutorialspoint.com/docker/index.htm> <br>
<https://docs.docker.com/> <br>
<http://container-solutions.com/understanding-volumes-docker/> <br>
[CLI Befehlsreferenz](https://docs.docker.com/engine/reference/commandline/cli/)

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

## Container
+ Container nutzt idR. den Kernel des Host-OS
+ Container sind Instanzen von Images
    + mit `run` wird jedesmal ein neuer Container erzeugt (!)
    + `docker run –it <image> /bin/bash`
    + `-it` für Interactive Mode (erzeugt Bash shell in Container)
+ `docker ps` zeigt die aktuell laufenden Container
    + `-a` zeigt alle an
+ brauchbare run Parameter:
    + siehe: <https://docs.docker.com/engine/reference/run/>
    + `-p 80:80` für die Portzuweisung vom Image zum Host (HOST:CONTAINER)
    + `-P` ordnet zufälligen Host-Ports alle EXPOSE Ports aus dem Dockerfile zu
    + `-d` für Detach (um nicht in den Container zu "gehen")
    + `--name=YOURNAME` benennt den Container
    + `--rm` löscht den Container nach dem Durchlauf gleich wieder
+ für die nachfolgenden Befehle kann die ContainerID oder der Name genutzt werden
    + den Namen muss man bei der Erstellung mit `--name` angeben
+ `docker start ContainerID` startet bereits existierenden Container (!= run)
    + Mit `run` wird jedesmal ein neuer Container erstellt. Irgendwann wird es voll in der Liste. Daher lieber einen alten Container starten.
    + mit Parameter `--attach` geht man in gleich in den Container rein
+ `docker stop ContainerID` stopt
+ `docker pause ContainerID` pausiert, `unpause` lässt weiterlaufen
+ `docker kill ContainerID` 
+ `docker rm ContainerID` löscht
+ `docker top ContainerID` zeigt top-level Prozesse eines Containers
+ `docker stats ContainerID` zeigt CPU und RAM Initialisierung
+ `docker attach ContainerID` verbindet sich mit Container
    + `CTRL+P+Q` um Container zu verlassne, ohne ihn zu schließen
+ `docker exec -it <container_id_or_name> echo "Hello from container!"` führt Kommando innerhalb eines Containers aus
    + dieser muss aber laufen
+ `docker cp` kopiert Dateien in oder aus Container
    + in Container hinein: `docker cp foo.txt mycontainer:/foo.txt` oder `docker cp src/. mycontainer:/target`
    + aus Container heraus: `docker cp mycontainer:/foo.txt foo.txt`
+ `service docker stop` stoppt Docker Daemon
+ `docker port <container>` zeigt die zugewiesen Ports an

## Erstellen eines Docker Images
### kurzer Überblick wie es geht
+ einfache Text Datei erstellen:

Beispiel #1
+ baut aus bestehenden Image, hier "alpine" Version 3.5
```docker
# our base image
FROM alpine:3.5

# Install python and pip
RUN apk add --update py2-pip

# install Python modules needed by the Python app
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt

# copy files required for the app to run
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/

# tell the port number the container should expose
EXPOSE 5000

# run the application
CMD ["python", "/usr/src/app/app.py"]
```

Beispiel #2
+ für minimale oder gar neue Images, wird `FROM scratch` genutzt 
+ [SCRATCH Docker Hub](https://hub.docker.com/_/scratch/)
```docker
FROM scratch
COPY hello /
CMD ["/hello"]
```

### Befehle
+ [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#run)
+ `FROM` nimmt bestehenden Container als Basis
+ `RUN` führt Befehle aus
    + jedes run erzeugt neuen Layer > mehr Größe
    + mit `&&` kann man Befehle koppeln > ist aber unübersichtlicher
+ `COPY source.txt /destination/` kopiert Dateien von aussen (Configs usw)
+ `EXPOSE 443` Port Nummern, welche vergeben werden müssen
    + mit `run -P` werden diese Ports beim Container-Bau automatisch eingerichtet
+ `CMD echo "Hello world"` wird als default Befehl aufgerufen, wenn beim `run` eines Containers kein Befehl Parameter mitgegeben wird
    + bei mehreren CMDs im Dockerfile wird nur das letzte als Default genommen
    + bei exec Schreibweise wird der Parameter (ohne dem Entrypoint hinzugefügt):
```docker
ENTRYPOINT ["/bin/echo", "Hello"]  
CMD ["world"] 
```
+ 'ENTRYPOINT` führt Befehl beim Start/Run von Container auf
    + falls Container selbst etwas starten soll (Apache Server)
    + wird durch Befehl Parametet bei `run` NICHT abgesägt

#### shell vs exec
+ shell: `CMD echo "Hello World"`
+ exec: `CMD ["/bin/echo", "Hello world"]`
    + `["executable", "param1", "param2", ...]`
+ siehe <http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/>

### Image bauen
+ `docker build -t ImageName:TagName dir`
    + `-t` optional für ein Tag
    + ImageName: der Name
    + TagName: Tagname
    + Dir: wo das Dockerfile liegt

### Best Practice
+ kleine Images
+ Multi-Stage-Builds nutzen um Overhead zu vermeiden
+ mehrere Layer durch mehrere `RUN` vermeiden, lieber durch `&&` verbinden
+ tags sinnvoll nutzen, nicht nur immer `latest`
+ Daten nicht in Containers "writable layer" schreiben, sondern [volumes](https://docs.docker.com/engine/admin/volumes/volumes/) nutzen
+ `sudo` vermeiden


## Multi-Stage-Build
<https://docs.docker.com/engine/userguide/eng-image/multistage-build/>

## Upload zum HUB
+ Online ein Repo erstellen
    + ein private möglich in kostenloser Version
    + für push muss man sich mit `docker login` anmelden
    + eigener Server/HUB möglich: <https://www.tutorialspoint.com/docker/docker_private_registries.htm>
+ taggen für Repo Upload
    + `docker tag <ImageID> demousr/demorep:1.0`
    + erstellt eine "kopie" vom Image. Der Name muss mit dem Repo übereinstimmen
+ `docker push demousr/demorep:1.0` pusht es
+ `docker pull demousr/demorep:1.0` holt es wieder runter

## Volumes
+ in früheren Versionen "data containers", jetzt werden Volumes genutzt
+ Volumes zum speichern von persistenten Daten
+ teilen von Daten zwischen Containern und zw. Host & Container
+ existiert als normales Verzeichnis auf Host System
+ `docker volume create --name my-volume`
+ `docker volume inspect my-volume` für Details über Volume inkl. Speicherort
    + idR. unter `var/lib/docker/volumes`...
+ um mit einem Container zu verbinden:
    + `docker run -v my-volume:/data hello-world`
    + Volume existiert als `/data` Ordner im Container
+ wird Container gelöscht, bleiben die Daten im Volume erhalten
+ Volume kann auch beim Container Erstellen erzeugt werden:
    + `docker run hello-word -v /data`
+ ein neues Volume kann auch mit einem Pfad auf dem Host verbunden werden:
    + `docker run -v /home/tom/data:/data hello-world`
    + geht nur mit `run` Befehl, nicht in Dockerfile
    + wird nicht unter `docker volume ls` angezeigt
+ zum löschen `docker volume rm my-volume`
    + für Übersicht der vorhandenen Volumes `volume ls`
    + zum löschen aller nich genutzer Volumes: `docker volume rm $(docker volume ls -q)`

### Volumes im Dockerfile
+ um Zugriffsrechte oder Daten/Konfigurations Dateien bei der Erstellung zu berücksichtigen
+ alles nach `VOLUME` wird auf das Volume nicht mehr angewendet
```Docker
FROM debian:wheezy
RUN useradd foo
VOLUME /data
RUN touch /data/x
RUN chown -R foo:foo /data
```
+ Zeile 4&5 funktionieren so NICHT
+ folgendes Beispiel hingegen schon:
```Docker
FROM debian:wheezy
RUN useradd foo
RUN mkdir /data &amp;&amp; touch /data/x
RUN chown -R foo:foo /data
VOLUME /data
```

## Ports
+ Container Ports müssen zu Host Ports gemappt werden
+ nach dem Format `HOSTPORT:CONTAINERPORT`
+ Beispiel: `docker run -p 8080:8080 -p 50000:50000 jenkins`
+ `docker run -P jenkins` ordnet zufälligen Host-Ports alle EXPOSE Ports aus dem Dockerfile zu
+ `docker port <container>` zeigt die zugewiesen Ports an

## Container verbinden
+ um nicht alles über Ports zu realisieren
+ `--link` ist eine Möglichkeit, ist aber "deprecated"
+ .... TBC ...
 
## Netzwerk
+ Docker hat einen eigenen Netzwerkadapter auf dem Host: `docker0`
+ `docker network ls` listet alle Netzwerke im Docker Host
+ `docker network inspect <NETWORKNAME>` für mehr Details eines Netzwerks (z.B. `bridge`)
+ beim Start eines Containers, wird dieser mit `bridge` Adapter verbunden

### Eigenes Netzwerk erstellen:
+ `docker network create –-driver drivername name`
    + Beispiel: `docker network create –-driver bridge new_nw`
+ Container beim erstellen mit Netzwerk verbinden
    + `docker run –it –-network=new_nw ubuntu:latest /bin/bash`
+ alle Container innerhalb des Netzwerks können miteinander kommunizieren
+ ... TBC ....

## Docker Compose
+ [Installations Anleitung](https://docs.docker.com/compose/install/)
+ [Kommandos](https://docs.docker.com/compose/reference/)
+ lädt mehrere Container als einzelnen Service
+ Composer Files in YAML > `docker-compose.yml`
+ starten mit `docker-compose up`
    + sucht im aktuellen Ordner nach einer compose YAML und führt sie aus

## lose Sammlung
Restricting networking. By restricting Docker networking so only linked containers can communicate you stop attackers with access to a container from being able to probe and compromise other containers. This can be achieved by using the --icc=false  and --iptables  flags when starting the Docker daemon

[Security Cheat sheet](https://container-solutions.com/content/uploads/2015/06/15.06.15_DockerCheatSheet_A2.pdf)<br>
[Running Secured Docker Registry 2.0](http://container-solutions.com/running-secured-docker-registry-2-0/)