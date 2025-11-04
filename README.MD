# 🐍 iobroker-Logs im RAM – Das Projekt, das deine SD-Karte liebt 💖

Willkommen zu unserem kleinen, genialen **iobroker-Log-Projekt**, bei dem wir etwas ganz Wichtiges gelernt haben:

> SD-Karten sind wie kleine Hunde 🐶: Man muss gut zu ihnen sein, sonst werden sie müde und kaputt 💔.

Wir haben eine **clevere Lösung gefunden**, die dafür sorgt, dass **alle Log-Dateien von ioBroker in den RAM geschrieben werden**, also in den blitzschnellen Arbeitsspeicher des Raspberry Pi ⚡. Das bedeutet:

* Deine SD-Karte wird **nicht mehr ständig vollgeschrieben** 🛡️
* Dein Raspberry Pi bleibt **schnell, glücklich und gesund** 😎
* Ein kleiner Helfer (Cron) kümmert sich darum, dass **alte Logs regelmäßig verschwinden**, wie von Zauberhand ✨

---

## 🎯 Was dieses Projekt tut

1. Wir legen ein spezielles **RAM-Verzeichnis** an (`/run/iobroker-logs`), das **flüchtig ist** (alles darin geht beim Neustart verloren 💨).
2. Die bisherigen Log-Dateien werden dorthin verschoben, und ein **magischer Link** sorgt dafür, dass ioBroker denkt, alles sei wie gewohnt (`/opt/iobroker/log`) 🔗.
3. Ein Cron-Job löscht automatisch alte Logs, damit der RAM **nicht zu voll wird** 🗑️
4. Alles läuft **von alleine**, du musst dich nur zurücklehnen und staunen 😍

---

## 🧑‍💻 Warum es genial ist

* Deine SD-Karte wird **geschont** – keine ständigen Schreibvorgänge mehr 💖
* Dein Raspberry Pi bleibt **flink wie ein Ninja 🥷**
* Selbst ein 8-jähriger Schüler kann verstehen:

  > Wir legen unsere Logs nicht auf die kleine SD-Karte 🐶, sondern in den blitzschnellen RAM-Speicher ⚡
* Und das Beste: ioBroker merkt **nicht einmal**, dass alles im RAM liegt – Magie pur ✨

---

## ⚡ Schneller Start

1. RAM-Log-Verzeichnis anlegen:

```bash
sudo mkdir /run/iobroker-logs
sudo chown iobroker:iobroker /run/iobroker-logs
```

2. Bestehende Logs verschieben und Link erstellen:

```bash
sudo mv /opt/iobroker/log /run/iobroker-logs/iobroker
sudo ln -s /run/iobroker-logs/iobroker /opt/iobroker/log
```

---

## 🕒 Cron-Job: Alte Logs automatisch löschen

Damit dein RAM nicht überläuft, löschen wir **alle 2 Tage die alten Logs**.

1. Cron-Job bearbeiten:

```bash
sudo crontab -e
```

2. Zeile einfügen:

```bash
0 4 */2 * * rm -rf /run/iobroker-logs/iobroker/*
```

### Erklärung:

* `0 4 */2 * *` → alle 2 Tage um 04:00 Uhr morgens ⏰
* `rm -rf /run/iobroker-logs/iobroker/*` → löscht alle Dateien im Log-Verzeichnis 🗑️
* ioBroker legt beim nächsten Log-Eintrag **automatisch eine neue Datei an** 📝

💡 Tipp für Anfänger: `sudo crontab -e` öffnet den Editor (meist `nano`) – einfach Zeilen einfügen, speichern mit `Ctrl+O`, und mit `Ctrl+X` wieder raus.

---

## 💡 Bonus-Tipps

* Du kannst **Backup-Skripte** einbauen, falls du bestimmte Logs doch speichern willst 💾
* Logs live beobachten:

```bash
tail -f /opt/iobroker/log/iobroker.current.log
```

* So sieht deine SD-Karte jetzt aus: sie **chillt entspannt im Corner 🐶🛋️**, während der RAM alle Log-Arbeiten übernimmt ⚡

---

## ❤️ Danke an unsere SD-Karten

Für all die Jahre, in denen ihr unermüdlich Log-Dateien gespeichert habt. Jetzt gönnen wir euch eine **wohlverdiente Pause** 🏖️

---

## ✅ Vorteile

* SD-Karte wird **geschont**, weniger Verschleiß
* IoBroker-Logs werden **schnell im RAM geschrieben**
* Automatisches Löschen alter Logs verhindert RAM-Stau
* Einfache Einrichtung, **keine Änderungen in ioBroker notwendig**
* Für Anfänger leicht verständlich, funktioniert „out-of-the-box“

## ⚠️ Nachteile

* Logs gehen beim Neustart **verloren**, außer du machst ein Backup
* RAM wird für Logs verwendet → bei sehr großen Logmengen kann RAM knapp werden
* Cron-Job muss korrekt eingerichtet sein, sonst werden alte Logs nicht gelöscht
* Wenn der Pi abstürzt, gehen alle Logs im RAM verloren

---
