---
title: "Lizens des Blogs auf CC BY Umstellen"
date: 2020-03-21T08:00:00+01:00
draft: true
tags: ["HowTo"]
---

Lizenzen sind für mich immer verwirrend. Ich verwende für den Blog
[Hugo](https://gohugo.io/), mit dem [Kiss](https://github.com/ribice/kiss)
Theme. Das *Kiss* Theme steht dabei unter einer
[MIT](https://github.com/ribice/kiss/blob/master/LICENSE.md) Lizens. Ob das
jetzt auch für den Blog gelten muss weiß ich allerdings nicht. Ich denke mal,
die Lizens erlaubt es mir meine eigenen Inhalte under die [CC
BY](https://creativecommons.org/licenses/) Lizens zu stellen. Ich wähle die *CC
BY*, da dies die liberalste aller *CC* Lizenzen ist und hoffentlich
aufwandsloses wieder verwerten ermöglicht.

Dokumentation, wie man auf github etwas unter einen CC lizens stellt habe ich
[hier](https://github.com/santisoler/cc-licenses) gefunden.

Man erstellt eine *Readme.md* und hängt folgendes an.

```
Shield: [![CC BY 4.0][cc-by-shield]][cc-by]

This work is licensed under a [Creative Commons Attribution 4.0 International
License][cc-by].

[![CC BY 4.0][cc-by-image]][cc-by]

[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-image]: https://i.creativecommons.org/l/by/4.0/88x31.png
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg
```

Außerdem erstellt man eine Datei mit dem Namen *Licence* und trägt hier den Inhalt von https://github.com/santisoler/cc-licenses/blob/master/LICENSE-CC-BY ein.

Das reicht aber leider immer noch nicht, den jetzt haben wir zwar das
Respository des Blogs unter die *CC BY* Lizens gestellt, aber üblich ist, dass
man auch unter jedem Blogeintrag selbst die Lizens definiert. Darum muss man
noch die `config.toml` Datei bearbeiten und hier bei `copyright`

``` toml
copyright = '<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.'
```
 Eintragen. Will man eine andere *CC* Lizens verwenden, kann man sich den Text bei https://creativecommons.org/choose/ erstellen lassen.
