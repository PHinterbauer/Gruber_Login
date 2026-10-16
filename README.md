# Login-Projekt

Eine einfache Login-Anwendung mit **Flask**, **PHP** und **SQLite**.

Das Projekt enthält zwei getrennte Versionen:

- **Python:** [`python_app.py`](python_app.py)
- **PHP:** [`php/`](php/)

Beide Versionen bieten Registrierung, Login, Passwort-Reset und drei
Sicherheitsstufen. Die Python- und PHP-Version sind eigenständige
Anwendungen und verwenden jeweils ihre eigene Datenbank.

## Funktionen

| Stufe | Anmeldung |
| --- | --- |
| 1 | E-Mail-Adresse und Passwort |
| 2 | Zusätzlich ein Code per E-Mail |
| 3 | Zusätzlich ein TOTP-Code aus einer Authenticator-App |

Weitere Funktionen:

- Sitzungen laufen nach 15 Minuten ab.
- E-Mail-Codes sind 5 Minuten gültig.
- TOTP-Codes sind 30 Sekunden gültig.
- Passwort-Reset-Links sind 15 Minuten gültig.
- Die Login-Seite besitzt mehrere Designs, unter anderem **Bier** und **Käse**.

## Voraussetzungen

Für die Python-Version werden benötigt:

- Windows mit Python 3.11 oder neuer
- ein Terminal wie Eingabeaufforderung oder PowerShell
- eine Internetverbindung für die einmalige Installation der Pakete

Für die PHP-Version werden benötigt:

- XAMPP mit Apache
- PHP mit den Erweiterungen `pdo_sqlite` und `sqlite3`

## Python-Version starten

### 1. Projektordner öffnen

Öffne die Eingabeaufforderung und wechsle in den Ordner, in dem
`python_app.py` liegt. Beispiel:

```cmd
cd "C:\Pfad\zum\Gruber_Login"
```

### 2. Virtuelle Umgebung erstellen

Eine virtuelle Umgebung hält die Python-Pakete dieses Projekts getrennt von
anderen Python-Projekten. Sie muss normalerweise nur einmal erstellt werden:

```cmd
py -m venv .venv
```

### 3. Umgebung aktivieren

In der Windows-Eingabeaufforderung:

```cmd
.venv\Scripts\activate
```

In PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Nach erfolgreicher Aktivierung steht meist `(.venv)` am Anfang der
Terminalzeile. Dadurch weißt du, dass die folgenden Pakete in der richtigen
Umgebung installiert werden.

### 4. Abhängigkeiten installieren

```cmd
python -m pip install -r requirements.txt
```

`requirements.txt` installiert Flask für den Webserver sowie QR-Code- und
Bild-Unterstützung für die OTP-Anmeldung.

### 5. Anwendung starten

```cmd
python python_app.py
```

Wenn der Start erfolgreich war, läuft der lokale Webserver auf Port 5000.
Öffne danach im Browser:

<http://127.0.0.1:5000>

Zum Beenden des Servers drückst du im Terminal `Strg+C`. Die virtuelle
Umgebung kann anschließend mit folgendem Befehl verlassen werden:

```cmd
deactivate
```

Falls PowerShell das Aktivieren aus Sicherheitsgründen blockiert, kann die
Anwendung auch direkt mit dem Python-Interpreter der virtuellen Umgebung
gestartet werden:

```powershell
.venv\Scripts\python.exe python_app.py
```

## Konfiguration

Die Datei `config.ini` enthält lokale Einstellungen wie den Flask-Schlüssel
und die SMTP-Zugangsdaten. Sie wird nicht in Git gespeichert. Erstelle sie
einmalig aus der Vorlage:

```cmd
copy config.example.ini config.ini
```

Passe anschließend mindestens diese Werte an:

```ini
[app]
flask_secret = ein-langes-zufaelliges-geheimnis
port = 5000

[mail]
host = smtp.example.com
port = 587
security = starttls
user = deine-adresse@example.com
password = dein-app-passwort
from = deine-adresse@example.com
```

Für Sicherheitsstufe 2 und Passwort-Reset werden gültige SMTP-Daten benötigt.
Bei vielen Mail-Anbietern ist dafür ein App-Passwort erforderlich.

**Wichtig:** `config.ini`, SMTP-Passwörter und geheime Schlüssel dürfen nicht
veröffentlicht oder in Git eingecheckt werden.

Die Anwendung startet standardmäßig unter `http://127.0.0.1:5000`. Der Port
kann im Abschnitt `[app]` geändert werden. Für lokale Tests kann
`wipe_database_on_start = true` verwendet werden; dadurch wird die Python-
Datenbank bei jedem Start gelöscht. Diese Einstellung darf im normalen
Betrieb nicht aktiviert sein.

## PHP-Version mit XAMPP

1. Kopiere das Projekt nach `C:\xampp\htdocs\login`.
2. Starte Apache im XAMPP Control Panel.
3. Aktiviere in `php.ini` die Erweiterungen `pdo_sqlite` und `sqlite3`.
4. Starte Apache nach einer Änderung an `php.ini` neu.
5. Konfiguriere den Mail-Versand über XAMPP/Sendmail.
6. Öffne <http://127.0.0.1/login/php/>.

Die Python- und PHP-Version verwenden getrennte SQLite-Datenbanken. Ein Konto
aus der Python-Version ist daher nicht automatisch in der PHP-Version vorhanden.

## Empfohlener Testablauf

1. Starte die Python-Version mit Sicherheitsstufe 1.
2. Lege ein Konto an und teste Login und Logout.
3. Stelle im Dashboard Sicherheitsstufe 2 ein und prüfe den E-Mail-Code.
4. Teste die Passwort-Zurücksetzung mit einem gültigen SMTP-Konto.
5. Verwende für Sicherheitsstufe 3 eine Authenticator-App und teste den
   angezeigten TOTP-Schlüssel.
6. Prüfe, dass eine Sitzung nach Ablauf des Zeitlimits nicht mehr gültig ist.

## Dateien

| Datei/Ordner | Zweck |
| --- | --- |
| `python_app.py` | Flask-Anwendung |
| `php/` | PHP-Anwendung |
| `static/style.css` | Designs der Python-Anwendung |
| `requirements.txt` | Python-Abhängigkeiten |
| `config.example.ini` | Vorlage für die Konfiguration |
| `login.sqlite3` | Python-Datenbank |

## Sicherheit

Die Anwendung speichert keine Klartextpasswörter. Passwörter werden vor der
Speicherung gehasht. E-Mail-Codes, TOTP-Codes und Reset-Links sind zeitlich
begrenzt. Für den produktiven Einsatz sollten zusätzlich HTTPS, Rate-Limiting,
sichere Cookie-Einstellungen und eine geschützte Serverkonfiguration verwendet
werden.

## Lizenz und Projektstatus

Dieses Projekt ist eine Schul-/Lernanwendung. Vor einem produktiven Einsatz
sollten Konfiguration, Sicherheit und Fehlerbehandlung nochmals geprüft werden.
