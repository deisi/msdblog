---
title: "LUKS mit Yubikey"
date: 2020-03-07T08:00:00+01:00
draft: true
tags: ["Security", "HowTo", "Linux"]
description: "Festplatten verschlüsselung mit Yubikeys unter Ubuntu"
---

Ich habe jetzt vor einiger Zeit zwei Ubuntu und einen Archlinux PC auf eine
2-Faktor-Vollverschlüsselung umgestellt. Mit dem Ergebnis bin ich bisher sehr
zufrieden. Darum dokumentiere ich hier die nötigen Schritte. Getestet wurde
diese Anleitung für Ubuntu 19.10.
# Begriffe

Die folgenden Begriffe sollten so ungefähr bekannt sein.
- **[LUKS](https://wiki.archlinux.org/index.php/Dm-crypt)**: Ein System zur
  Verschlüsselung von Festplatten unter Linux.
- **[LUKS-Header](https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption#Backup_and_restore)**:
  Die wichtigste Datei einer verschlüsselten Festplatte. Hier werden die
  gültigen Passwörter zur Entschlüsselung in Keyslots abgelegt. Bis zu 8 können
  gleichzeitig aktiviert sein. **Achtung** Wird der LUKS-Header beschädigt und
  besitzt man kein Backup, sind die Daten unwiederbringlich verloren.
- [initramfs](https://wiki.archlinux.org/index.php/Mkinitcpio): So etwas wie ein
  kleines Filesystem, dass während des boot Vorgangs gestartet wird. Für die
  Entschlüsselung während des boot Vorgangs müssen wir hier etwas anpassen.

# Voraussetzungen

Als zweiten Faktor verwende ich [yubikeys](https://www.yubico.com/) nicht ganz
billig, aber sie tun was sie sollen. Da ich den *challenge-response* Modus für
die Verschlüsselung verwende, muss es einer der teureren *Yubikeys* (schwarz)
sein. Die günstigeren *Security Keys* (blau) unterstützen den
challenge-response-modus nicht und können nicht verwendet werden. [Allerdings
könnte es in Zukunft möglich sein LUKS mit FIDO2 zu
verwenden.](https://github.com/shimunn/fido2luks) Dann wären auch die blauen
Keys okay, aber bisher ist das noch sehr frühes Entwicklungsstadium und Fedora
only glaube ich. Es empfiehlt sich sogar 2 bis 3 dieser Keys zu kaufen, da man
so auch einen verlieren kann ohne gleich komplett angeschmiert zu sein.


# Das Konzept

![concept](/yubikey/challenge_response.svg)


Die Idee ist, dass die Festplatte mit einem Passwort verschlüsselt wird, das in
Wahrheit aus zwei Teilen besteht. Der *challenge* und der *response*. Die
*challenge* ist der erste Faktor. Sie ist quasi der Teil des Passworts, den ich
kenne, oder anders gesagt, das Passwort im klassischen Sinne. Der zweite Teil
ist die *response*. Die *response* wird vom Key als Antwort auf die *challenge*
erstellt. Das besondere an Yubikeys und anderen Cryptokeys ist, dass es nicht
möglich ist, das der *response* zu Grunde liegende Geheimnis aus dem Key zu
extrahieren. Selbst wenn man den Key besitzt, kommt man nicht mehr an das
Geheimnis heran. Darum handelt es sich hier fast tatsächlich um einen zweiten
Faktor, denn man muss den Key tatsächlich besitzen.

Ich sage allerdings fast, da bei der Einrichtung des Yubikeys, das Geheimnis
doch angezeigt wird. Dies ist meines Wissens nach aber der einzige Moment in dem
das Möglich ist. Dadurch kann man das Geheimnis natürlich auch abspeichern und
aus dem Besitz, wird doch wieder Wissen. Um mehr als einen Yubikey mit der
selben *response* zu erstellen, ist das leider auch zwingend notwendig, da nur
so ein zweiter Key mit der selben *response* erstellt werden kann. Es ist also
jedem selbst überlassen hier eine Entscheidung zu treffen.

Die *response* wird als Key in einem Keyslot des LUKS-Headers abgelegt. Somit
braucht es zwei Faktoren um die Festplatte zu entschlüsseln. Siehe [5] für mehr
Details.

# Schritt 1: Das Ubuntu System verschlüsseln.

Das ist zum Glück inzwischen verboten einfach. Hier kann man den grafischen
Ubuntu-Installer verwenden um die Vollverschlüsselung zu aktivieren. **ACHTUNG**
alle Daten werden hierdurch gelöscht. Bitte also vorher **BACKUP** machen.
Während der Installation vergibt man erst mal ein einfaches Passwort. Das ändern
wir dann später.

# Schritt 2: Den Yubikey vorbereiten.

Der Yubikey kommt standardmäßig mit deaktiviertem challenge-response-modus. Um
diesen zu aktivieren braucht man den [Yubikey
Manager](https://www.yubico.com/products/services-software/download/yubikey-manager/).
Für Ubuntu gibt es eine
[PPA](https://support.yubico.com/support/solutions/articles/15000010964-enabling-the-yubico-ppa-on-ubuntu):

```bash
sudo add-apt-repository ppa:yubico/stable && sudo apt-get update
sudo apt-get install yubikey-manager-qt
```
Unter `Applications -> OTP` z.b. den zweiten Slot des Yubikey, in den challenge-response-modus versetzen. Am Ende sollte es in Yubikey Manager etwa so aus sehen

![img](/yubikey/yubikey_manager.png)

# Schritt 3: Challenge-Response des Yubikey im LUKS-Header ablegen.

Hier zu yubikey-luks installieren:
``` bash
sudo apt install yubikey-luks
```
Und den Schlüssel registrieren:
``` bash
sudo yubikey-luks-enroll
```
Als nächstes muss das initramfs Modul für den yubikey aktiviert werden. Dazu
muss man `keyscript=/usr/share/yubikey-luks/ykluks-keyscript` ans Ende der
`/etc/crypttab` Datei anhängen. Also:

``` bash
sudo nano /etc/crypttab

cryptroot /dev/sda none  luks,keyscript=/usr/share/yubikey-luks/ykluks-keyscript
```

Hier muss `/dev/sda` durch die eigene Festplatte ersetzt werden falls nötig.
`lsblk` hilft hier weiter. Nach einer Änderung and der `/etc/crypttab` muss das
initramfs neu gebaut werden, darum:

``` bash
sudo update-initramfs -u
```

Damit der Suspend/Ruhe Modus geht:

``` bash
sudo systemctl enable yubikey-luks-suspend.service
```

Jetzt kann neu gestartet werden. Wenn alles gut gegangen ist, wird man beim
Booten aufgefordert den Yubikey einzustecken und das Passwort ein zu geben. Wenn
nicht, nicht schlimm, man hat ja noch das Passwort von der Installation.
Vielleicht hilft ein Blick auf https://github.com/cornelinux/yubikey-luks


# Bonus Schritt 4: Das alte einfache Passwort löschen.

Während der Installation benutze ich ein einfaches Passwort um die Festplatte zu
verschlüsseln, da man es doch ein paar mal eingeben muss. Dieses Passwort sollte
man entweder deaktivieren, oder durch ein möglichst langes anderes Passwort
ersetzen. Ich empfehle letzteres. Man schwächt zwar seine 2-Faktor
Verschlüsselung etwas, aber wenn dieses Passwort sehr lang ist und sicher im
Passwortmanager verwahrt wird, ist es nicht weiter Tragisch. Darum fügt man erst
ein drittes neues Passwort hinzu:

``` bash
sudo cryptsetup luksAddKey /dev/<device> 

Enter any passphrase:
Enter new passphrase for key slot:
Verify passphrase:
```

Wobei `<device>` hier ein Platzhalter für die eigene Festplatte ist. Also z.b.
`sda`. Das schwache Passwort aus der Installation kann man jetzt aus dem
LUKS-Header löschen mit:
```bash
sudo cryptsetup luksRemoveKey /dev/<device>

Enter LUKS passphrase to be deleted:
```
und gibt das **zu löschende** Passwort an. Für mehr Optionen lohnt [hier](https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption#Key_management) ein Blick.


# Bonus Schritt 5: Ein Backup des LUKS-Header erstellen.
Der LUKS-Header ist ein *single point of failure*. Darum sollte man unbedingt
ein Backup vom LUKS-Header erstellen. Allerdings erst nachdem die finale Version
der Keys dort abgelegt worden ist. Ansonsten kann jemand, der über den alten
LUKS-Header verfügt, diesen benutzen um die Festplatte mit dem schwachen
Passwort aus der Installation zu entschlüsseln.

```bash
sudo cryptsetup luksHeaderBackup /dev/<device> --header-backup-file /mnt/<backup>/<file>.img
```

reicht dafür. Siehe
[hier](https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption#Backup_and_restore)
für nähere Informationen. Wo und wie man das Backup am besten speichert ist ein
Problem für sich. Ich habe es so gemacht, dass ich 3 verschlüsselte Rechner habe
und auf jedem ist ein Backup sämtlicher LUKS-Header aller drei Systeme.

# Bonus Schritt 6: Automatisches Login aktivieren.
Nervig ist, dass man jetzt beim booten nach zwei Passwörtern gefragt wird. Das
für die Verschlüsselung der Festplatten und dass für den User Login. Da ich eh
nur einen User habe, und meine Festplatte ja verschlüsselt ist, habe ich den
User Login deaktiviert. Das geht bei Ubuntu über die GUI unter *Settings >
Details > Users* und dann die Checkbox *Automatic Login*. Im Deutschen heißen
sie Sachen Wahrscheinlich *Systemeinstellungen > Details > Benutzer >
Automatischer Login* oder so ähnlich.

Allerdings reicht das noch nicht ganz, denn jetzt wird man trotzdem noch nach
dem Passwort für die Keychain gefragt. Standard mäßig ist das gleich dem User
Passwort. IMO kann man auch auf dieses Passwort verzichten. Dazu muss ein leeres
Passwort eingegeben werden:

![seahorse](/yubikey/seahorse.png) Starte oder installiere falls nötig das
Programm `seahorse`. Bei Ubuntu taucht es auch als *Passwords and Keys* auf. Am
linken Rand mit rechter Maustaste auf Login das Passwort ändern. Dazu erst das
alte User-Passwort eingeben und dann für das Neue leer lassen.

Viel Freude mit dem Setup! 

Deisi


# Quellen und Referenzen
- https://github.com/cornelinux/yubikey-luks
- https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption
