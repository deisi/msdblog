---
title: "Papierloses Büro"
date: 2020-04-13T08:00:00+01:00
draft: false
tags: ["HowTo", "Linux"]
---

Papier ist soooo 20. Jahrhundert! Darum will ich so wenig wie möglich auf Papier
haben. Ich scanne lieber alles hirnlos ein und speichere es auf meinen
Festplatten. Damit ich das ganze später auch halbwegs sinnvoll benutzen kann,
müssen die Dokumente aber durchsuchbar sein. Ansonsten hätte man, gegenüber dem
einheften, nichts gewonnen. Mittel **O**ptical **C**haracter **R**ecognition
(OCR) ist das möglich. Hier werden gedruckte Buchstaben erkannt und aus dem Scan
wird ein durchsuchbarer Scan. Der OpenSource-Branchen-Primus ist wohl
[Tesseract](https://github.com/tesseract-ocr/tesseract). Tesseract ist aber eher
ein backend und nicht wirklich einfach zu benutzen. Aus diesem Grund verwende
ich [Ocrmypdf](https://github.com/jbarlow83/OCRmyPDF). Als Pythonanwendung hat
Ocrmypdf natürlich gleich einen Bonus in meinem Herzen und auch einen Commit ;).

Am Ende des Tutorials haben wir folgendes Setup:

![img](/ocrmypdf/ocrmypdf.svg)

# 1. Schritt: Scanner einrichten.
Hier kann ich nicht wirklich helfen, aber ich habe es so gemacht, dass der
Scanner auf eine SMB Festplatte im Netzwerk speichert. Mein Scanner erlaubt das.
Hier muss man gegebenenfalls bei der Hardwareauswahl aufpassen. Ein Papiereinzug
sollte vorhanden sein, da das scannen so schon nervig genug ist.

# 2. Schritt: Ocrmypdf installieren.
Ich benutze den offiziellen
[Dockercontainer](https://hub.docker.com/r/jbarlow83/ocrmypdf/). Ein Vorteil des
Dockercontainers ist, dass er bereits mit einem `watcher` Script ausgestattet
ist. Dieses Script ermöglicht es einem den Inhalt eines Ordners zu überwachen
und sobald ein neuer Scan im Ordner erscheint, wird automatisch *ocrmypdf*
aktiviert und eine durchsuchbare Version erstellt. Wir brauchen also erst mal
*docker* und *docker-compose*:

``` bash
sudo apt install docker docker-compose
```
Jetzt fehlt uns noch ocrmypdf. Das können wir mit *docker-compose*
installieren und zwar:

``` bash
mkdir ocrmypdf
cd ocrmypdf
touch docker-compose.yml
```
Die *docker-compose.yml* wird mit folgendem Inhalt gefüllt:
``` yaml
version: "3.3"
services:
  ocrmypdf:
    restart: always
    container_name: ocrmypdf
    user: 1000:1000
    image: jbarlow83/ocrmypdf
    volumes:
      - "PFAD_ZUM_INPUT_FOLODER:/input"
      - "PFAD_ZUM_OUTPUT_FOLDER:/output"
    environment:
      - OCR_OUTPUT_DIRECTORY_YEAR_MONTH=1
      - OCR_ON_SUCCESS_DELETE=1 
      - OCR_DESKEW=1
      - PYTHONUNBUFFERED=1
    entrypoint: python3
    command: watcher.py
```

**Wichtig** die Datei muss *docker-compose.yml* heißen. Etwas mehr Dokumentation
findet man bei
[ocrmypdf](https://ocrmypdf.readthedocs.io/en/latest/batch.html#hot-watched-folders)
selbst. Natürlich muss `PFAD_ZUM_INPUT_FOLODER` und `PFAD_ZUM_OUTPUT_FOLDER` den
eigenen Bedürfnissen angepasst werden. `OCR_OUTPUT_DIRECTORY_YEAR_MONTH=1` sorgt
dafür, dass jahres- und monatsweise Ordner erstellt werden.
`OCR_ON_SUCCESS_DELETE=1` löscht den original Scan nach erfolgreichem OCR,
`OCR_DESKEW=1` korrigiert leichte Schieflagen von Dokumenten,
`PYTHONUNBUFFERED=1` sorgt dafür, dass der Dockerlog auch alle Informationen
enthält und `user: 1000:1000` bewirkt, dass alle Datein vom Standarduser
erstellt werden und nicht wie sonst root. `restart=always` bedeutet, dass der
Container immer ausgeführt wird. Auch automatisch nach einem Neustart.

Das Ganze wird aktiviert mit: 

``` bash
sudo docker-compose up -d
```

das `-d` steht hierbei für *detached*. D.h. alles läuft im Hintergrund ab. Will
man aber erst mal sehen ob es überhaupt funktioniert, kann man das `-d`
weglassen und bekommt so den Log im Terminal angezeigt. Mit `Ctl + c` kann das
ganze dann beendet werden.

Viel Freude mit dem Setup! 

Deisi
