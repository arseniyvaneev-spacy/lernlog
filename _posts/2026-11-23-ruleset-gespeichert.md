---
layout: post
title: "Gespeichert heißt nicht aktiv: Drei Rulesets, drei Fehlversuche"
date: 2026-11-23 10:00:00 +0100
---

*Lernlog: Ich baue mir gerade eigene Agenten für Claude Code und schreibe auf, woran ich hängen bleibe. Ich bin kein Experte, sondern lerne das selbst.*

## Worum es geht

Im [letzten Beitrag]({{ site.baseurl }}{% post_url 2026-11-20-push-schutz %}) war der Hook, der meinen Agenten vom Push auf `main` abhalten soll, nur eine Leitplanke. Die harte Grenze sollte ein **Ruleset** auf GitHub sein: eine Regel direkt am Repo, die Pushes auf `main` und Force-Pushes ablehnt, egal wer oder was pusht.

Also Ruleset angelegt, gespeichert, fertig. Dachte ich. Insgesamt habe ich drei Rulesets eingerichtet, und keins davon hat beim ersten Mal getan, was ich wollte.

## Fehlversuch 1: Das Ziel traf keinen Branch

Ich hatte das Ruleset angelegt, auf „Active“ gestellt, und auf der Bypass-Liste stand niemand. Dann habe ich testweise direkt auf `main` gepusht.

Der Push ging durch.

Die Ursache stand klein unter den Einstellungen: **Target pattern „main“ – 0 branches match.** Die Regel war aktiv, sie galt nur für keinen einzigen Branch. GitHub hat das angezeigt, aber nicht als Fehler, sondern als neutrale Info.

## Fehlversuch 2: Das richtige Ruleset im falschen Repo

Nach der Korrektur ging der Test-Push wieder durch. Diesmal war das Ruleset völlig in Ordnung, nur lag es im falschen Repository: in meiner Agenten-Sammlung statt in dem Repo, das ich schützen wollte. Rulesets gelten immer nur für das Repo, in dem sie stehen. Aufgefallen ist mir das erst auf einem Screenshot, auf dem oben links der falsche Name stand.

Erst danach hat GitHub den Test-Push mit `GH013: Repository rule violations` abgelehnt. Genau diese Meldung wollte ich sehen.

## Fehlversuch 3: Pflicht-Checks, die keine Pflicht waren

Ein paar Tage später kam der nächste Schritt. Meine Repos haben inzwischen automatische Prüfungen (Tests, Linter, einen Sicherheits-Check), und ich hatte zweimal zu früh gemergt: einmal, während eine Prüfung noch lief, und einmal, obwohl eine schon rot war. Danach war `main` rot.

Die Lösung heißt **Required Status Checks**: Das Ruleset lässt einen Merge erst zu, wenn bestimmte Prüfungen grün sind. In drei Repos eingetragen, gespeichert.

Diesmal habe ich nicht auf „gespeichert“ vertraut, sondern es getestet: in jedem Repo ein Probe-PR mit einem **absichtlich roten** Test. Danach habe ich bei GitHub nachgesehen, ob der Merge gesperrt ist. GitHub kennt dafür zwei Zustände:

- `blocked`: Eine Pflicht-Prüfung ist rot oder läuft noch, der Merge ist gesperrt.
- `unstable`: Prüfungen sind rot, aber keine davon ist Pflicht. Der Merge-Knopf ist frei.

Ergebnis: **In zwei von drei Repos stand `unstable`.** Der rote Test hätte sich einfach mergen lassen.

Die Ursache war peinlich vertraut: **wieder das falsche Repo.** Die Pflicht-Checks standen zwar in einem Ruleset, aber nicht in dem Repo, dessen PR ich gerade getestet hatte. Derselbe Fehler wie bei Fehlversuch 2, nur Tage später und mit mehr Erfahrung.

Nach der Korrektur stand überall `blocked`. Und in einem Repo habe ich zusätzlich die Gegenprobe gemacht: roten Test entfernt, Prüfungen grün, Status `clean`, also Merge wieder frei.

## Eine Falle, in die ich fast getappt wäre

Nicht jede Prüfung darf Pflicht werden. In einem Repo laufen manche Prüfungen nur, wenn bestimmte Dateien geändert werden. Trägt man so eine als Pflicht ein, wartet jeder andere PR für immer auf eine Prüfung, die nie startet, und lässt sich nie mehr mergen. Pflicht dürfen nur Prüfungen werden, die bei **jedem** PR laufen.

## Was ich daraus mitnehme

- **„Gespeichert“ ist keine Bestätigung.** Bei drei von drei Rulesets hat es beim ersten Mal nicht gegriffen, und GitHub hat nichts davon als Fehler gemeldet.
- **Schutz testet man mit einem Fall, der verboten sein soll.** Ein grüner PR beweist gar nichts: Der lässt sich mit und ohne Ruleset mergen. Erst der absichtlich rote zeigt, ob die Sperre existiert.
- **Danach auch die Gegenprobe.** Eine Regel, die alles sperrt, ist genauso kaputt wie eine, die nichts sperrt. Sie fällt nur schneller auf.
- Das ist dieselbe Lektion wie bei meinem Hook, eine Ebene höher: **Ein Schutz, der still nicht greift, ist schlimmer als keiner, weil man sich auf ihn verlässt.**

---

*Hast du deine Branch-Regeln schon einmal mit einem absichtlich verbotenen Push oder PR getestet? Oder nur gespeichert?*

*Geschrieben mit Claude, auf Basis meiner eigenen Fehler.*
