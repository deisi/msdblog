---
title: "Verschlüsselung mittels PLZ bei einer Bank!?!"
date: 2020-02-28T08:30:00+01:00
tags: ["Security"]
description: "Banken verschlüsseln mit PLZ..."

---

Ich habe gerade ein Konto bei einer großen deutschen online Bank eröffnet und
ich traue meinen Augen kaum. Ich zitiere mal aus der Mail:

```
...
im Anhang bekommen Sie wichtige Unterlagen für Ihr Konto - als PDF-Date
und verschlüsselt. Bitte geben Sie beim Öffnen des PDFs einfach Ihre 
Postleitzahl ein.
...
```

WTF????? Euer Ernst???? Ein Witz oder? Habe ich gedacht, aber nein, echt Wahr.
Verschlüsselung quasi mit Passwort in der selben Mail. Welches Genie war den
hier am Werk?

Hier wurde auf ganzer Linie nicht verstanden was die Rolle von Verschlüsselung
beim Thema Sicherheit ist. Der Bank muss bewusst sein, das e-mails nicht Sicher
sind. Darum auch die Verschlüsselung, aber warum dann mit einer 5 stelligen Zahl
bei einem angehängten PDF? Das sind, stellen wir uns mal doof, gerade mal
100.000 Kombinationsmöglichkeiten. Als echt arme Hacker leisten wir uns nur
Hardware mit 1. Versuch/Sekunde, dann brauchen wir immer noch weniger als 30
Stunden um garantiert die richtige Nummer zu finden. Ich habe es nicht
ausprobiert, aber ich denke ein erfolgreicher Angriff auf Minutenskala ist ohne
Probleme möglich. Man muss ja nur PLZs ausprobieren, das seht ja unverschlüsselt
in der Mail und 00000 gibt es z.B. nicht. Auch ist 1 Versuch/Sekunde nicht
wirklich Zeitgemäß.

Ein Leser dieses Blogs hat es sogar noch leichter, denn ich bin von Gesetzes
wegen verpflichtet meine Adresse im Impressum an zu geben. Danke also für nix.

Man kann jetzt argumentieren, dass die Informationen in dem PDF nicht so wichtig
sind. Stimmt auch, aber nichts desto trotz handelt es sich um personenbezogene
Daten und die werden hier wirklich nicht angemessen geschützt. In Zeiten des
Identitätsdiebstahls fühlt sich das alles nicht sehr gut an.

Das es keine praktikable alternative Lösung gibt, kann auch nicht stimmen. Es
gibt mittels Webinterface ja einen sicheren bidirektionalen Kanal zwischen der
Bank und mir. Warum der nicht für die Übermittlung genutzt wird ist mir völlig
schleierhaft. Nach der Aktion habe ich schon fast keinen Bock mehr auf die Bank.
Wenn die mein Geld so so gut schützen wie meine Daten, dann bin ich bald Pleite.
