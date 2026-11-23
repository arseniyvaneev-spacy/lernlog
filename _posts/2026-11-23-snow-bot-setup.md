---
layout: post
title: "Der Bot läuft – nur nicht so, wie ich dachte"
date: 2026-11-23 20:00:00 +0100
---

*Lernlog: Ich baue mir gerade eigene Agenten für Claude Code und schreibe auf, woran ich hängen bleibe. Ich bin kein Experte, sondern lerne das selbst.*

## Worum es geht

Mein Telegram-Bot sollte endlich zum ersten Mal echt laufen: auf meinem eigenen PC, mit meinem Bot-Token, gegen das echte Telegram. Programmiert war alles, die Tests waren grün. Bis der Bot mir das erste Mal geantwortet hat, bin ich trotzdem an vier Stellen hängen geblieben – und an keiner davon war der Code schuld.

## Stolperstein 1: Die Anleitung gehörte zum falschen Projekt

Beim Start kam immer wieder dieselbe Zeile:

```
Fehler: TELEGRAM_ALLOWED_USER_ID fehlt – in .env eintragen
```

Ich hatte aber nach einer Anleitung gearbeitet, in der die Variable `ALLOWED_USER_IDS` hieß. Der Grund: Ich habe zwei Bot-Projekte für denselben Zweck, in zwei verschiedenen Sitzungen mit Claude entstanden – und die Anleitung stammte aus dem anderen. Gleiche Idee, anderer Variablenname.

Aufgefallen ist das erst, als Claude nach der genauen Fehlermeldung im Code gesucht hat und sie nicht im Projekt fand, das es selbst gebaut hatte, sondern im zweiten.

## Stolperstein 2: Die Zeile stand im Terminal, nicht in der Datei

Die `.env` war da, im richtigen Ordner. Aber `Select-String` zeigte:

```
TELEGRAM_ALLOWED_USER_ID=
```

Leer. Ich hatte `TELEGRAM_ALLOWED_USER_ID=…` in PowerShell getippt statt in die Datei – PowerShell hielt die Zeile für einen Befehl und meldete, den gebe es nicht. Der eigentliche Hinweis stand schon vorher in der Dateiliste: `.env` und `.env.example` waren **genau gleich groß**, 935 Bytes. Die Kopie der Vorlage war also nie ausgefüllt worden – auch Token und API-Schlüssel nicht.

Kleiner Nebenbefund: Lange Befehle, die ich vom Handy in PowerShell kopiert hatte, kamen mit Zeilenumbrüchen an und zerfielen in Bruchstücke. Kurze Einzelzeilen funktionieren zuverlässiger.

## Stolperstein 3: Der Bot schwieg – absichtlich

Nach dem Ausfüllen startete der Bot, aber er antwortete auf nichts. Keine Fehlermeldung, nichts im Log.

Das ist Absicht: Der Bot beantwortet nur meine eigene Telegram-ID und ignoriert alle anderen **stumm**. Gut gegen Fremde – aber wenn die ID nicht passt oder ich einem anderen Bot schreibe, sieht das genauso aus wie „Bot ist kaputt“.

Geholfen hat ein Abgleich in zwei Richtungen: Zu welchem Bot gehört der Token wirklich (Telegram gibt den Namen über `getMe` heraus, ohne dass man den Token anzeigen muss)? Und stimmt die ID Ziffer für Ziffer mit der von @userinfobot überein?

Bei mir war es die ID: Die Zahl in der `.env` passte nicht zu meinem Telegram-Konto. Für den Bot war ich damit ein Fremder – und Fremden antwortet er nicht. Mit der richtigen ID und einem Neustart kam sofort die erste Antwort.

## Stolperstein 4: „Läuft“ hieß nicht „läuft mit Claude“

Dann antwortete der Bot. Ich dachte: Claude ist dran. War es nicht.

Der Bot hat einen Gratis-Modus über kostenlose Modelle anderer Anbieter, und auf den schaltet er automatisch um, solange die Claude-Agenten nicht eingerichtet sind. Das war gewollt als Ausfallsicherung – nur merkt man es im normalen Chat kaum. Erst `/start` zeigte: **„Aktiv: Snow Free.“** Und `/snow` sagte: „Snow AI ist noch nicht eingerichtet.“

Der Unterschied ist nicht kosmetisch: Der Gratis-Modus hat **keine Websuche**. Genau das, wofür Snow AI gedacht ist – Quellen lesen und belegen –, fällt damit weg. Antworten mit Quellenangaben können dann aus dem Gedächtnis eines schwächeren Modells stammen.

Ich bin trotzdem dabei geblieben, bewusst: Der Bot bleibt vorerst gratis für Alltag und Ideen, für Recherche mit Belegen nehme ich das Claude-Projekt in der App.

## Was ich daraus mitnehme

- **Eine Fehlermeldung zuerst wörtlich zurückverfolgen.** Wer meldet sie eigentlich? Bei mir war es nicht das Programm, das ich zu starten glaubte.
- **Dateigröße ist ein guter Lügendetektor.** Ist die ausgefüllte Kopie genauso groß wie die Vorlage, wurde nichts ausgefüllt.
- **Stummes Ignorieren ist sicher, aber schwer zu debuggen.** Man braucht einen Weg, von außen zu prüfen, ob die Erkennung stimmt – ohne Geheimnisse preiszugeben.
- **Ausfallsicherungen brauchen eine sichtbare Anzeige.** Ein Fallback, den man nicht bemerkt, ist dieselbe Falle wie in den letzten beiden Beiträgen: Etwas läuft still anders, als man denkt.
- Und eine neue Lehre: **Zwei Projekte für denselben Zweck kosten mehr, als man denkt** – nicht im Code, sondern beim Verwechseln.

---

*Hattest du schon einen Fehler, der aus einem ganz anderen Programm kam, als du dachtest? Und merkst du bei deinen Tools, wenn sie still auf einen Ersatz umschalten?*

*Geschrieben mit Claude, auf Basis meiner eigenen Fehler.*
