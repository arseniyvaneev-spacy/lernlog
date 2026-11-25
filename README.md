# Lernlog

Quelle der Seite **https://arseniyvaneev-spacy.github.io/lernlog/** – ein deutschsprachiges Lernprotokoll zum Bauen eigener Agenten mit Claude Code.

## Aufbau

| Pfad | Inhalt |
|---|---|
| `_posts/JJJJ-MM-TT-titel.md` | ein Beitrag pro Datei, mit Front Matter (`layout`, `title`, `date`) |
| `index.md` | Startseite mit Beitragsliste |
| `ueber.md` | Über diese Seite, inkl. Hinweis auf die Zusammenarbeit mit Claude |
| `_config.yml` | Titel, Autor, Theme (`minima`) |

Gebaut wird automatisch von GitHub Pages (Jekyll), kein eigener Workflow nötig.

## Neuer Beitrag

1. Datei `_posts/JJJJ-MM-TT-kurztitel.md` anlegen, oben:
   ```yaml
   ---
   layout: post
   title: "Titel des Beitrags"
   date: JJJJ-MM-TT 12:00:00 +0100
   ---
   ```
2. Text in Markdown darunter, am Ende die Zeile *Geschrieben mit Claude, auf Basis meiner eigenen Fehler.*
3. Links auf ältere Beiträge mit `{{ site.baseurl }}{% post_url JJJJ-MM-TT-kurztitel %}` – GitHub Pages baut mit Jekyll 3, dort fügt `post_url` den Pfad `/lernlog` nicht selbst hinzu.
4. Vor dem Veröffentlichen prüfen: keine Tokens, keine Telegram- oder Nutzer-IDs, keine Links auf private Repositories.

Die Entwürfe entstehen im privaten Repo `ideen` und werden erst hier veröffentlicht.

## Lokal ansehen (optional)

```bash
gem install jekyll minima jekyll-feed jekyll-seo-tag
jekyll serve
```
