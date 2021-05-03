---
title: "Testkernel kompilieren"
date: 2021-05-03T08:00:00+01:00
tags: ["HowTo", "Kernel"]
description: "Kernel selbst kompilieren"
---

Ich habe vor einiger Zeit angefangen gelegentlich meinen eigenen Linuxkernel zu
kompilieren. So kann ich der Kernelentwicklung folgen und nach Regressionen
ausschau halten. Den Kernel selbst zu kompilieren und zu testen ist quasi die
einfachste Art und Weise einen bescheidenen Beitrag zur Linuxkernelentwicklung
zu leisten. Vorausgesetzt man berichtet wenn man plötzlich Probleme haben
sollte. 

Ich habe extra ein Subordner für den Kernel erstellt. Warum wird am Ende klar:
``` bash
mkdir kernel
cd kernel
```

Danach kann man Linus Kernelbranch herunterladen mit:
``` bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
```

Unter Ubuntu braucht es die folgenden Abhängigkeiten. 
``` bash
sudo apt-get install git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex
```

Um einen Kernel zu bauen braucht man eine config Datei. Die aktuell verwendete
ist grundsätzlich keine schlechte Idee.
``` bash
cp /boot/config-$(uname -r) .config
```

Jede Kernelversion bringt Änderungen zur Konfiguration mit sich. Mit
`olddefconfig` kann man die neuen Sachen auf default setzen. Das sollte sicher
sein.

``` bash
make olddefconfig
```

Für den kernel geht das gute alte (Linus will es so) `make` und `make install`.
Ich persönlich versuche aber möglichst wenig an meinem Paketmanager vorbei zu
installieren. Auf lange Sicht macht das nur Stress. Zum Glück ist es beim Kernel
super einfach ein `.deb` Paket zu bauen:

```bash
make -j 10 bindeb-pkg
```

`-j 10` bedeutet, dass ich mit 10 threads Kompiliere, was für meinen 8 Kern
Prozessor okay ist. Je nachdem was du hast, nimm mehr oder weniger `Anzahl der
Kerne + 2` geht meistens gut.

`bindeb-pkg` sorgt dafür, dass wir installierbare `.deb` Pakete bekommen. Es gibt
auch die Option `deb-pkg` aber bei allen von mir getesteten Systemen ist
`deb-pkg` kaputt. `bindeb-pkg` hat hingegen immer funktioniert. Das
Ergebnis von obigem `make` sind dann 4 `.deb` Dateien. Allerdings landen diese
Dateien eine Ordner ebene höher. Darum auch der Initiale `kernel` Ordner im
ersten Befehl. `ls ..` zeigt jetzt also u.a.:

```bash
linux-headers-X_X-Y_amd64.deb
linux-image-X_X-Y_amd64.deb
linux-image-X-dbg_X-Y_amd64.deb
linux-libc-dev-X-Y_amd64.deb
```

X und Y sind hierbei spezifische versionsnummern. X ergibt sich aus der
Kernelversionsnummer und Y ist eine Buildnummer.

Das Ergebnis kann jetzt installiert und verwendet werden:
```bash
sudo dpkg -i linux-headers-X_X-Y_amd64.deb linux-image-X_X-Y_amd64.deb
```

Die `libc-dev` und `dbg` Pakete sind dabei optional und brauchen nur installiert
werden, wenn es Probleme gibt. Sie dienen zum debuggen, was, wenn alles geht, ja
nicht gebraucht wird.

Danach kann das System bereits neu installiert werden. Dadurch, dass wir die
`.deb` Pakete installieren, werden auch alle dkms module aktualisiert und grub
wird über die neue Version informiert. Das simple `make install` kümmert sich
zwar auch um grub, aber `dkms` module sind danach oft kaputt.

Will man den neuen Kernel wieder los werden, kann man ihn mit apt wieder deinstallieren.
```bash
sudo apt remove linux-headers-X  linux-image-X
```

Hier natürlich aufpassen, dass man die richtige Version deinstalliert, da man ja
jetzt mindestens zwei Kernel im System hat. Ich würde keinem empfehlen den
Default Kernel zu deinstallieren.

Viel Spaß mit dem Setup
Deisi

