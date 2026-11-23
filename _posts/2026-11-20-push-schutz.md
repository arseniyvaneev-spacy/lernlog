---
layout: post
title: "Warum mein Push-Schutz still ausgeschaltet gewesen wäre"
date: 2026-11-20 12:00:00 +0100
---

*Lernlog: Ich baue mir gerade eigene Agenten für Claude Code und schreibe auf, woran ich hängen bleibe. Ich bin kein Experte, sondern lerne das selbst.*

## Was ich gebaut hatte

Ich habe einen Agenten, der für mich auf GitHub arbeitet: Branch anlegen, committen, pushen, Pull Request öffnen. Was er **nie** darf: direkt auf `main` pushen oder per Force-Push Historie umschreiben.

Damit das nicht nur eine Bitte im Prompt bleibt, hängt an dem Agenten ein **Hook** – ein kleines Skript, das Claude Code vor jedem Shell-Befehl aufruft. Erkennt es einen verbotenen Push, beendet es sich mit Exit-Code 2, und Claude Code bricht den Befehl ab. Im Frontmatter des Agenten stand:

```yaml
hooks:
  PreToolUse:
    - matcher: Bash
      hooks:
        - type: command
          command: '"$CLAUDE_PROJECT_DIR"/.claude/hooks/guard-git.sh'
```

Getestet, funktioniert. Dachte ich.

## Wo das Problem lag

Meine Agenten liegen in einem eigenen Repo und werden per Installationsskript **global** nach `~/.claude/` verlinkt, damit ich sie in jedem Projekt nutzen kann. Das Skript hat aber nur Agenten und Skills verlinkt – **keine Hooks**.

In jedem anderen Projekt passiert dann Folgendes: Der Agent ist da. `$CLAUDE_PROJECT_DIR` zeigt auf das aktuelle Projekt, und dort gibt es keine `guard-git.sh`. Der Hook-Befehl schlägt fehl, mit Exit-Code 127 für „Datei nicht gefunden“.

Und jetzt der Haken: **In Claude Code blockiert nur Exit-Code 2.** Jeder andere Fehler gilt als „Hook hatte ein Problem“, und der Befehl läuft trotzdem durch. Der Schutz wäre also nicht kaputt gewesen, sondern **einfach weg** – ohne eine Fehlermeldung, die mir aufgefallen wäre.

Getestet hatte ich nur im Repo, in dem der Hook liegt. Dort war alles grün.

## Die Lösung

Zwei Änderungen:

1. **Das Installationsskript verlinkt jetzt auch die Hooks** nach `~/.claude/hooks/`.
2. **Der Hook-Befehl sucht an zwei Stellen – und sperrt, wenn er nichts findet:**

```yaml
command: 'h="$CLAUDE_PROJECT_DIR/.claude/hooks/push-waechter.sh";
  [ -x "$h" ] || h="$HOME/.claude/hooks/push-waechter.sh";
  if [ -x "$h" ]; then exec "$h"; fi;
  if grep -q push; then echo "push-waechter nicht gefunden – Pushes gesperrt." >&2; exit 2; fi;
  exit 0'
```

Erst im Projekt nachsehen, dann in der globalen Installation. Ist der Hook nirgends da, wird jeder Befehl mit „push“ abgelehnt; alles andere läuft normal weiter.

## Was ich daraus mitnehme

- **Schutzmechanismen müssen im Zweifel sperren, nicht durchlassen.** Ein Wächter, der bei einem Fehler still aufgibt, ist schlimmer als keiner – weil man sich auf ihn verlässt.
- **Den Fehlerfall testen, nicht nur den Normalfall.** Die eigentliche Frage war nicht „Blockiert der Hook einen Push auf `main`?“, sondern „Was passiert, wenn der Hook gar nicht da ist?“.
- **Ein Hook ist eine Leitplanke, keine Mauer.** Er fängt Versehen ab. Die harte Grenze ist ein Branch-Schutz direkt auf GitHub. Den hatte ich übrigens zuerst im falschen Repo eingerichtet – aber das ist der nächste Beitrag.

---

*Hattest du schon einen Schutz, der still nichts getan hat? Und welche Fallstricke beim Bauen von Agenten würdest du gern als Nächstes lesen?*

*Geschrieben mit Claude, auf Basis meiner eigenen Fehler.*
