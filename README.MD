# 🐍 ioBroker-Logs im RAM – Das Projekt, das deine SD-Karte liebt

## 📑 Inhaltsverzeichnis
- [Was dieses Projekt tut](#was-dieses-projekt-tut)
- [Wichtiger Punkt Cron beim Boot](#wichtiger-punkt-cron-beim-boot)
- [Anleitung Schritt für Schritt](#anleitung-schritt-fuer-schritt)
  - [1 Vorbereitendes Skript erstellen](#1-vorbereitendes-skript-erstellen)
  - [2 Cron-Job für Boot einrichten](#2-cron-job-fuer-boot-einrichten)
  - [3 Cron-Job Alte Logs löschen optional](#3-cron-job-alte-logs-loeschen-optional)
  - [4 Test](#4-test)
- [Bonus Tipps](#bonus-tipps)
- [Vorteile](#vorteile)
- [Nachteile Risiken](#nachteile-risiken)
- [RAM-Log Rückbau Mini-Checkliste](#ram-log-rueckbau-mini-checkliste)
- [Haftungsausschluss](#haftungsausschluss)

---

<a id="was-dieses-projekt-tut"></a>
## 🎯 Was dieses Projekt tut

1. Es wird ein **RAM-Verzeichnis** erstellt (`/run/iobroker-logs`), das beim Neustart leer ist 💨.
2. Die bisherigen Log-Dateien werden dorthin verschoben, und ein **symbolischer Link** sorgt dafür, dass ioBroker denkt, alles sei wie gewohnt (`/opt/iobroker/log`) 🔗.
3. Ein **Cron-Job mit `@reboot`** erstellt das RAM-Verzeichnis und startet ioBroker beim Boot.
4. Alte Logs im RAM werden automatisch gelöscht – alles läuft **von alleine**.

---

<a id="wichtiger-punkt-cron-beim-boot"></a>
## ⚠️ Wichtiger Punkt Cron beim Boot

> Dies ist **der entscheidende Punkt**, damit ioBroker nach jedem Neustart funktioniert.
> Wenn Cron nicht korrekt eingerichtet ist, startet ioBroker **nicht** und Logs existieren nicht.

---

<a id="anleitung-schritt-fuer-schritt"></a>
## 🧑‍💻 Anleitung Schritt für Schritt

<a id="1-vorbereitendes-skript-erstellen"></a>
### 1️⃣ Vorbereitendes Skript erstellen

```bash
sudo nano /usr/local/bin/iobroker-ramlog.sh
````

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

<a id="2-cron-job-fuer-boot-einrichten"></a>

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

<a id="3-cron-job-alte-logs-loeschen-optional"></a>

### 3️⃣ Cron-Job Alte Logs löschen optional

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

<a id="4-test"></a>

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

<a id="bonus-tipps"></a>

## 💡 Bonus Tipps

* Backup-Skripte einbauen, falls bestimmte Logs doch auf SD gespeichert werden sollen 💾
* Logs live beobachten:

```bash
tail -f /opt/iobroker/log/iobroker.current.log
```

* Die SD-Karte wird entspannt geschont 🐶🛋️, während der RAM alles erledigt ⚡

---

<a id="vorteile"></a>

## ✅ Vorteile

* SD-Karte wird **geschont** – weniger Verschleiß
* ioBroker-Logs werden **schnell im RAM geschrieben**
* Automatisches Löschen alter Logs verhindert RAM-Stau
* Einfache Einrichtung über Cron, **kein komplexes Systemd-Gedöns**
* Für Anfänger leicht verständlich

---

<a id="nachteile-risiken"></a>

## ⚠️ Nachteile Risiken

* Logs gehen beim Neustart **verloren**, außer ein Backup wird erstellt
* RAM wird für Logs verwendet → bei sehr großen Log-Mengen kann RAM knapp werden
* **Cron muss korrekt eingerichtet sein** – sonst startet ioBroker nicht und Logs existieren nicht
* Wenn der Pi abstürzt, gehen alle Logs im RAM verloren

---

<a id="ram-log-rueckbau-mini-checkliste"></a>

## 🔧 RAM-Log Rückbau Mini-Checkliste

1. **ioBroker stoppen**

```bash
sudo systemctl stop iobroker.service
```

2. **RAM-Logs sichern**

```bash
sudo mkdir -p /opt/iobroker/log_backup
sudo cp -r /run/iobroker-logs/iobroker/* /opt/iobroker/log_backup/
sudo chown -R iobroker:iobroker /opt/iobroker/log_backup
```

3. **Link entfernen & Original-Verzeichnis wiederherstellen**

```bash
sudo rm /opt/iobroker/log
sudo mkdir -p /opt/iobroker/log
sudo chown -R iobroker:iobroker /opt/iobroker/log
```

4. **Cron und Systemd-Service löschen**

```bash
sudo crontab -e    # RAM-Log Zeilen löschen
sudo systemctl disable iobroker-ramlog.service
sudo rm /etc/systemd/system/iobroker-ramlog.service
sudo systemctl daemon-reload
```

5. **ioBroker neu starten & Kontrolle**

```bash
sudo systemctl start iobroker.service
ls -l /opt/iobroker/log
ls -l /opt/iobroker/log_backup
```

* `/opt/iobroker/log` → normales Log-Verzeichnis auf SD
* `/opt/iobroker/log_backup` → gesicherte RAM-Logs

---

<a id="haftungsausschluss"></a>

## 📝 Haftungsausschluss

Dieses Projekt wird **ohne Garantie** bereitgestellt.
Der Autor übernimmt **keine Haftung** für Schäden an Hardware, Datenverlust oder sonstige Probleme.
Benutzung erfolgt **auf eigene Verantwortung**.

```