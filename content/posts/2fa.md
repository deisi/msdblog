--
title: "Zwei Faktor Authentifizierung (2FA) mit SSH"
date: 2019-01-08T18:47:05+01:00
draft: false
tags: ["Security", "HowTo"]
description: "Zwei Faktor Authentifizierung mit Google Authenticator"
images: ["/2fa/00.png"]
---

Es zeigt sich, das zwei Faktor Authentifizierung (2FA) mit SSH sowohl gut, als
auch schlecht funktioniert. Die Installation und die initiale Inbetriebnahme ist
verboten einfach, allerdings steckt der Teufel im Detail.

## 1. Google PAM Modul Installieren

PAM steht für **P**luggable **A**uthentication **M**odule und bezeichnet den
Namen unter dem google seinen Service für 2FA führt. Ein anderer Begriff denn
man öfter mal ließt ist Time-based One-Time Password (TOTP). Dies bezeichnet das
sich zeitlich ändernde Passwort, das mittels Google Authenticator zur Verfügung
gestellt wird.
 
 ```bash
$ sudo apt-get update
$ sudo apt-get install libpam-google-authenticator
```

Unter Arch Linux muss man noch `qrencode` installieren, wenn man den QR code
direkt im Browser angezeigt bekommen will
 
 ```bash
 $ sudo pacman -S qrencode libpam-google-authenticator
 ```

Als nächstes hangelt man sich durch die interaktive Einrichtung.
 
 ```bash
$ google-authenticator
 ```
 
Die ist sehr selbsterklärend, also verzichte ich auf eine Beschreibung. Ich
empfehle euch allerdings die ganzen **emergency codes** die ihr am Ende bekommt
irgendwo ab zu speichern. Im Falle eines Falles sind sie dann da.

![img](/2fa/00.png)

Diesen QR Code könnt ihr dann mit dem `Google Authenticator` fotografieren. Und
zack läuft das Ding schon mal auf eurem Handy.

## 2. Aktivieren des Moduls

Als nächstes muss noch das Modul für die SSH Verbindung aktiviert werden.
Hierfür müssen zwei Dateien editiert werden. In der `/etc/pam.d/sshd`

```config
auth required pam_google_authenticator.so
```

ans Ende anfügen und als zweites, in der `/etc/ssh/sshd_config`

```config
ChallengeResponseAuthentication yes
AuthenticationMethods keyboard-interactive
```

eintragen.

## 3. SSH Daemon neu starten

```
sudo systemctl restart sshd.service
```

Es ist eine gute Idee die aktuell aktive Shell nicht zu schließen, sondern statt
dessen die Verbindung mit einer neuen Shell zu testen. Hier sollte jetzt erst
nach dem Passwort und dann nach einem `Verification code` gefragt werden. Beim
`Verification code` handelt es sich um den gerade aktiven Google Authenticator
Key. Beim eingeben des Keys müsst ihr euch übrigens nicht tot hetzen, da immer
mindestens 3, vielleicht sogar 17 Keys gleichzeitig gültig sind.

## 4. SSH Key statt Passwort

Wenn ihr vorher anstelle von Passwörtern ssh Keys verwendet habt, so wie ich,
dann dürfte euch aufgefallen sein, dass ihr auf einmal trotzdem Passwörter
eingeben müsst. Der SSH Key wird einfach ignoriert. Dabei ist es auch egal, dass
ihr bei `PasswordAuthentication no` eingegeben habt. Der SSH Key wird einfach
ignoriert.

Bisher habe ich es nicht hinbekommen per default Keys und als Fallback ein
Passwort zu haben. Stattdessen kann ich euch nur sagen wie man komplett auf das
Passwort verzichtet.

Öffnet dazu wieder `/etc/pam.d/sshd` und kommentiert die Zeile:
```bash
# Standard Un*x authentication.
#@include common-auth
```

wie gezeigt aus und startet den SSH Daemon neu. 

Im Internet kursieren verschiedene Lösungen die die `AuthenticationMethods`
ändern, aber alles was ich bisher ausprobiert habe, hatte nicht den gewünschten
Effekt. Wer eine Lösung kennt, immer her damit.

## Quellen

[1](https://hostadvice.com/how-to/how-to-enable-two-factor-authentication-on-ubuntu-18-04-vps/)
[2]( https://www.linode.com/docs/security/authentication/use-one-time-passwords-for-two-factor-authentication-with-ssh-on-ubuntu-16-04-and-debian-8/ )
[3](https://blog.webnersolutions.com/ubuntu-google-authenticator-not-working-with-ssh-keys)
