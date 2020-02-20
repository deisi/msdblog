---
title: "DSGVO mit Disqus und GoogleAnalytics"
date: 2020-02-19T08:00:00+01:00
draft: true
tags: ["HowTo"]
---

Ich habe schon gerne etwas Statistik über den Blog. Man kann jetzt drüber
streiten ob man unbedingt GoogleAnalytics (GA) braucht, aber wenigstens die
Zugriffszahlen interessieren mich dann schon. Auch finde ich, dass eine
Kommentarfunktion dazu gehört. Da ich den Blog nicht selbst hoste, sondern auf
github. Kann ich aber nicht einfach das über den Apache log o.ä. machen. Darum
werde ich erst mal sowohl GA, als auch Disqus einbinden bis ich was besseres
finde. Allerdings mit den DSGVO privacy Einstellungen von hugo. Das Bedeutet,
ich benutze folgende `config.toml` für den Blog.

```toml
[privacy]
  [privacy.disqus]
    disable = false
  [privacy.googleAnalytics]
    anonymizeIP = true
    disable = false
    respectDoNotTrack = true
    useSessionStorage = tru
```

## Quellen
https://gohugo.io/about/hugo-and-gdpr/
