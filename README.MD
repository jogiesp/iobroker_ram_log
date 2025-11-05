# 🐍 ioBroker-Logs im RAM – Das Projekt, das deine SD-Karte liebt 

Willkommen zu unserem genialen **iobroker-Log-Projekt**, das die SD-Karte deines Raspberry Pi schützt:

> SD-Karten sind wie kleine Hunde 🐶: Man muss gut zu ihnen sein, sonst werden sie müde und kaputt 💔.

Wir haben eine **clevere Lösung**, die dafür sorgt, dass **alle Log-Dateien von ioBroker in den RAM geschrieben werden**, also in den blitzschnellen Arbeitsspeicher des Pi ⚡.

**Das bedeutet:**

* Deine SD-Karte wird **nicht mehr ständig vollgeschrieben** 🛡️
* Dein Raspberry Pi bleibt **schnell, glücklich und gesund** 😎
* Ein kleiner Helfer (Cron) sorgt dafür, dass **alte Logs automatisch gelöscht werden** ✨

---

## 🎯 Was dieses Projekt tut

1. Es wird ein **RAM-Verzeichnis** erstellt (`/run/iobroker-logs`), das beim Neustart leer ist 💨.
2. Die bisherigen Log-Dateien werden dorthin verschoben, und ein **symbolischer Link** sorgt dafür, dass ioBroker denkt, alles sei wie gewohnt (`/opt/iobroker/log`) 🔗.
3. Ein **Cron-Job mit `@reboot`** erstellt das RAM-Verzeichnis und startet ioBroker beim Boot.
4. Alte Logs im RAM werden automatisch gelöscht – alles läuft **von alleine**.

---

## ⚠️ Wichtiger Punkt: Cron beim Boot

> Dies ist **der entscheidende Punkt**, damit ioBroker nach jedem Neustart funktioniert.
> Wenn Cron nicht korrekt eingerichtet ist, startet ioBroker **nicht** und Logs existieren nicht.

---

## 🧑‍💻 Anleitung – Schritt für Schritt

### 1️⃣ Vorbereitendes Skript erstellen

```bash
sudo nano /usr/local/bin/iobroker-ramlog.sh
```

Inhalt:

```bash
#!/bin/bash
# RAM-Log für ioBroker vorbereiten

# 1. RAM-Verzeichnis anlegen
mkdir -p /run/iobroker-logs/iobroker
chown -R iobroker:iobroker /run/iobroker-logs

# 2. Symbolischen Link setzen
ln -sfn /run/iobroker-logs/iobroker /opt/iobroker/log

# 3. ioBroker starten
systemctl start iobroker.service
```

Speichern und ausführbar machen:

```bash
sudo chmod +x /usr/local/bin/iobroker-ramlog.sh
```

---

### 2️⃣ Cron-Job für Boot einrichten

* Cron für root bearbeiten:

```bash
sudo crontab -e
```

* Folgende Zeile hinzufügen:

```cron
@reboot /usr/local/bin/iobroker-ramlog.sh
```

💡 **Erklärung:**

* `@reboot` → Skript wird **bei jedem Boot automatisch ausgeführt**
* Erst wird das RAM-Verzeichnis erstellt, dann der Link gesetzt, und iobroker gestartet

> **WICHTIG:** Ohne diesen Schritt startet ioBroker nach einem Neustart **nicht** und das RAM-Log existiert nicht. Dies ist der zentrale Punkt für eine funktionierende Einrichtung!

---

### 3️⃣ Cron-Job: Alte Logs löschen (optional)

Damit der RAM nicht voll wird, kannst du einen weiteren Cron-Job einrichten, z. B. alle 2 Tage:

```bash
sudo crontab -e
```

Füge hinzu:

```cron
0 4 */2 * * rm -rf /run/iobroker-logs/iobroker/*
```

* `0 4 */2 * *` → alle 2 Tage um 04:00 Uhr morgens
* `rm -rf /run/iobroker-logs/iobroker/*` → löscht alle Dateien im Log-Verzeichnis
* ioBroker legt beim nächsten Log-Eintrag automatisch neue Dateien an

---

### 4️⃣ Test

* Neustart durchführen:

```bash
sudo reboot
```

* Prüfen:

```bash
ls -l /run/iobroker-logs/iobroker
tail -f /opt/iobroker/log/iobroker.current.log
systemctl status iobroker.service
```

✅ Ergebnis:

* RAM-Verzeichnis existiert sofort nach Boot
* Symbolischer Link ist korrekt
* ioBroker startet automatisch
* Alte Logs können per Cron gelöscht werden

---

## 💡 Bonus-Tipps

* Backup-Skripte einbauen, falls bestimmte Logs doch auf SD gespeichert werden sollen 💾
* Logs live beobachten:

```bash
tail -f /opt/iobroker/log/iobroker.current.log
```

* Die SD-Karte wird entspannt geschont 🐶🛋️, während der RAM alles erledigt ⚡

---

## ✅ Vorteile

* SD-Karte wird **geschont** – weniger Verschleiß
* ioBroker-Logs werden **schnell im RAM geschrieben**
* Automatisches Löschen alter Logs verhindert RAM-Stau
* Einfache Einrichtung über Cron, **kein komplexes Systemd-Gedöns**
* Für Anfänger leicht verständlich

---

## ⚠️ Nachteile / Risiken

* Logs gehen beim Neustart **verloren**, außer ein Backup wird erstellt
* RAM wird für Logs verwendet → bei sehr großen Logmengen kann RAM knapp werden
* **Cron muss korrekt eingerichtet sein** – sonst startet ioBroker nicht und Logs existieren nicht
* Wenn der Pi abstürzt, gehen alle Logs im RAM verloren

---

## 📝 Haftungsausschluss

Dieses Projekt wird **ohne Garantie** bereitgestellt.
Der Autor übernimmt **keine Haftung** für Schäden an Hardware, Datenverlust oder sonstige Probleme.
Benutzung erfolgt **auf eigene Verantwortung**.

---
