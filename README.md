# Wordpress Installation mit Docker
## Beschreibung
WordPress ist ein freies Content-Management-System (CMS). Mit Wordpress können leicht Webseiten, Blog und Online-Shops erstellt und verwaltet werden. Die Installation hingegen kann komplexer sein, da beispielweise eine Datenbank benötigt wird, die im Hintergrund läuft und die Daten der Wordpress-Seite speichert. Im folgenden Beispiel verwende ich die Docker, um diesen Prozess zu erleichtern und Wordpress in wenigen Minuten zum Laufen zu bringen.
[Docker](https://www.docker.com/) selbst ist eine Open-Source-Software, in der Anwendungen mit Hilfe von Containervirtualisierung isoliert werden können.
## Vorgehen
1. Installation mit Docker
2. Installation mit Docker Compose

## Installation mit Docker
### Installation Docker
Bevor wir mit der Installation von Wordpress beginnen können, müssen wir sicher stellen, dass Docker auf unserem Rechner installiert ist.
Auf dieser [Seite](https://docs.docker.com/get-docker/) kann Docker heruntergeladen werden. In meinem Beispiel wird Windows als Betriebsystem verwendet. 

Um zu prüfen, ob die Installation erfolgreich war öffnen wir ein Terminal und führen folgenden Befehl aus:

`$ docker version`

Und folgende Ausgabe sollte erscheinen:

```
Client: Docker Engine - Community
  Cloud integration    0.1.18
  Version              19.03.13
  API version          1.40[
  Go version           go1.13.15
  ....
```
### Einrichtung der MySql-Datenbank
Nun können mit der Wordpress Installation beginnen. Dafür brauchen wir als erstes eine Datenbank, in der alle Informationen zu Wordpress gespeichert werden können.
Hier wird MySql als Datenbank verwendet. Wir installieren die MySql-Datenbank mit Hilfe des [offizielen mysql docker Images](https://hub.docker.com/_/mysql).

`$ docker run --name mysql-datenbank -e MYSQL_ROOT_PASSWORD=password -d mysql`

Schauen wir uns den Befehl genau an, um zu verstehen was genau passiert. 
Das erste Segment ist `docker run`, das genau das tut, was es sagt: einen Docker-Container starten. Als nächstes wird mit `--name mysql-datenbank` der Name des Containers festgelegt, den wir erstellen. Dieser kann nach belieben geändert werden.Mit `-e` werden Umgebungsvariablen für die Ausführung des Containers gesetzt und hier setzen wir das `MYSQL_ROOT_PASSWORD`, das für die Ausführung dieses Images zwingend erforderlich ist. Das Flag`-d` sorgt dafür, dass der Container im Hintergrund läuft und nicht das Terminal blockiert. Zuletzt gibt `mysql` an, dass wir die neuste Version des mysql-Images ausführen wollen. Hier können wir entweder nur `mysql` oder `mysql:latest` verwenden, beide Befehle referenzieren die aktuellste Verion.

**Achtung!
In einem realen Szenario sollte jedoch ein sicheres Passwort gewählt werden. Dies simple Passwort wurde hier aus Testzwecken verwendet.**

Jetzt prüfen wir, ob der Container korrekt erstellt wurde und läuft. Dafür nutzen wir folgenden Befehl:
`docker container ls`

Folgende Ausgabe sollte erschienen:
ToDo Sceenshot einfügen

Als nächstes erstellen wir in der Datenbank eine Tabelle für Wordpress:

`$ docker exec -it mysql-datenbank mysql -u root -p`

Dieser Befehl führt `mysql -u root -p` innerhalb des `mysql-datenbank`-Containers aus. Das Flag `-it` führt einen interaktiven Start aus.
Das beduetet wir befinden uns jetzt in der MySql-Eingabeaufforderung und können eine Tabelle erstellen.

Als Passwort verwenden wir das vorher im Befehl `$ docker run --name mysql-datenbank -e MYSQL_ROOT_PASSWORD=password -d mysql` festgelegte Passwort.

`mysql> create database wordpress;`

Mit `exit` können wir aus dem interaktiven MySql-Modus zurück kehren.

### Einrichtung Wordpress
Genau wie beim Einrichten von mysql können wir das [offizielle wordpress-Image](https://hub.docker.com/_/wordpress/) verwenden, Wordpress zum Laufen zu bringen.

`$ docker run --name lokales-wordpress -p 8080:80 -d wordpress`

Der Befehl ist sher ähnlich zu dem von MySql, der Unterschied ist, dass wir hier `-p 8080:80` haben, was Docker anweist, ein Port-Mapping durchzuführen. Der Port 8080 unseres Rechners wird an den Port 80 des Containers weitergeleitet. Jetzt können wir auf unser Wordpress zugreifen, indem wir http://localhost:8080 im Browser aufrufen.
Jedoch können wir die Installation nicht finalisierien, zuerst müssen wir noch ein Netzwerk erstellen, dass Wordpress mit der Datenbank kommunizieren kann.

### Einrichtung des Docker Netzwerks
Um die Einrichtung von Wordpress abzuschließen, müssen wir ein Netzwerk erstellen. Denn die Datenbank befindet sich in einem seperaten Container und Wordpress hat keinen Zugang auf diesen.

Um dieses Problem zu beheben, erstellen wir ein neues Netzwerk und hängen beide Container an dieses an.

`$ docker network create --attachable wordpress-netzwerk`

Mit dem obigen Befehl wird Docker angewiesen, ein neues Netzwerk namens `wordpress-netzwerk` zu erstellen, das manuell an Container angehängt werden kann. Nachdem das Netzwerk erstellt wurde, können wir es mit den Containern verbinden.

```
$ docker network connect wordpress-network mysql-datenbank
$ docker network connect wordpress-network lokales-wordpress
```
### Die Installation abschließen
Abschließend rufen wir http://localhost:8080 im Browser auf und gelangen zum Wordpress-Assistenten. Dort stellen `mysql-datenbank` als Datenbank-Host ein. Nun sind wir in der Lage, den Assistenten fortzusetzen und Wordpress zum Laufen zu bringen.

## Installation mit Docker Compose
Die Verwendung von Docker Compose ist eine weitere Methode Wordpress mit einer MySql-Datenbank über Docker zu installieren.
Docker Compose ist ein Werkzeug zur Definition und Ausführung von Multi-Container-Docker-Anwendungen. Bei Compose wird eine YAML-Datei verwendet, um die Anwendung zu konfigurieren, ähnlich wie wir es bei der "manuellen" Konfiguration über Befehle gemacht haben. Dann können wir mit einem einzigen Befehl beider Container inklusive Konfiguration hochfahren.

Als erstes erstellen wir zur Übersichtlichkeit ein neues Verzeichnis. Dafür öffnen wir unser Terminal und führen folgenden Befehl aus:

`$ mkdir wordpress`

Nun spring wir mit `$ cd wordpress/` in das angelegte Verzeichnis.
Jetzt können wir unsere Konfigurationsdatei erstellen, das sogenannte Docker Compose File. Dieses sollten wir `docker-compose` nennen und der Dateiname muss entweder `.yml` oder `.yaml` sein. Da...

`$vi docker-compose.yaml`

Inhalt der `docker-compose.yaml`
```
version: "3.3"

services:
  db:
    image: mysql
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: mysql-datenbank
      MYSQL_USER: mysqluser
      MYSQL_PASSWORD: password

  wordpress:
    depends_on:
      - db
    image: wordpress
    ports:
      - "8080:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: mysqluser
      WORDPRESS_DB_PASSWORD: password
      WORDPRESS_DB_NAME: mysql-datenbank
volumes:
  db_data: {}
```
Dann können wir unser Terminal öffnen und mit `$ cd` in das von uns erstellte Verzeichnis springen, in dem die YAML-Datei liegt.
Mit dem Befehl

`docker-compose up -d`

beide Container starten. Jetzt können wir wie schon im ersten Beispiel erklärt im Browser http://localhost:8080 aufrufen und die Wordpress installation abschließen.

Um beide Container zu stoppen nutzen wir den Befehl

`$ docker-compose down`

## Anleitung zum Testen
### Test mit Katacoda
Katacoda ist eine Online-Plattform, die Hunderte von Szenarien und Sandbox-Umgebungen bietet, um verschiedene Arten von Technologien kennenzulernen und mit ihnen zu spielen.
Unter andererm bietet Katacoda einen Docker-Playground in dem Docker Befehle ausgeführt werden können.
Jedoch muss hierfür ein Account erstellt werden.

Über folgenden [Link](https://www.katacoda.com/courses/docker/playground) kommen wir zu dem Docker Playground.



