---
title: "Papierloses_buero"
date: 2020-03-10T08:00:00+01:00
draft: true
tags: ["HowTo", "Linux"]
---

# Papierloses Büro

Um möglichst wenig Papierkram behalten zu müssen habe ich mir ein **O**ptical
**C**haracter **R**ecognition (OCR) System aufgesetzt. Ein OCR erlaubt es, dass
eingescante Documente durchsuchbar werden. Im wesentlichen, werden die
Buchstaben erkannt und eingelesen. Im neudeutsch würde man das wohl ins
Kerngebiet einer künstlichen Intelligenz (KI) einordnen, aber das hier zu Grunde
liegende Programm *Tesseract* ist fast schon Uralt.

Im wesentlichen braucht es zwei Dinge. Einen Netzwerkfähigen Scanner, am besten
mit Papiereinzug und einen Server. Der Scanner wird angewiesen alles auf dem
Server zu speichern. Ich verwende
[Ocrmypdf](https://github.com/jbarlow83/OCRmyPDF), da es die eingescannten pdfs
quasi unverändert lässt, diese aber durchsuchbar macht.


# 1. Schritt: Scanner einrichten.
Hier kann ich nicht wirklich helfen, aber ich habe es so gemacht, dass der
Scanner auf eine SMB Festplatte im Netzwerk speichert. Mein Scanner erlaubt das.
Hier muss man gegebenenfalls bei der Hardware Auswahl aufpassen. Auch ein
Papiereinzug ist empfehlenswert, da jede Seite einzeln Scannen nicht viel Spaß
macht.

# 2. Schritt: Ocrmypdf Installieren.

Da ich sowieso viele Docker container verwende und es hier bereits einen gut
konfigurierten container. Ein weiterer Vorteil des Containers ist, dass er
bereits den *watcher.py* enthält. Der *watcher.py* ist ein script, dass den
Inhalt eines Ordners laufend beobachtet und sobald sich etwas ändert, ein neuer
scan, wird ocrmypdf aktiviert und erstellt durchsuchbare versionen der pdfs.

https://github.com/jbarlow83/OCRmyPDF
https://ocrmypdf.readthedocs.io/en/latest/introduction.html
https://github.com/tesseract-ocr/tesseract

Ich verwende *docker* und *docker-compose*:

# 3 Schritt: Rechte reparieren

Das Problem bei docker ist, dass ocrmypdf jetzt als root läuft und darum auch
sämtliche scans mit root als owner abgespeichert werden. Da ich aber auch datei
verändern und verschieben können will als normaler benutzer, muss ich das
ändern.
