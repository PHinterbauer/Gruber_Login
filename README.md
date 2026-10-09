# Login-Projekt

Dieses Repository enthält zwei eigenständige Implementierungen desselben
Login-Systems:

- **Python/Flask** in [`python_app.py`](python_app.py)
- **PHP** im Ordner [`php/`](php/)

Beide Varianten unterstützen:

1. E-Mail-Adresse und Passwort
2. E-Mail-Adresse, Passwort und sechsstelligen Code per E-Mail
3. E-Mail-Adresse, Passwort und sechsstelligen TOTP-Code aus einer
   Authenticator-App

Zusätzlich gibt es Registrierung, Sitzungen mit Zeitlimit, Passwortänderung
über einen zeitlich begrenzten Link und eine Sicherheitsstufe, die zwischen
den drei Login-Verfahren umgeschaltet werden kann.

Die Anwendungen verwenden **getrennte SQLite-Datenbanken**:

- Python: `login.sqlite3` im Projektverzeichnis
- PHP: `php/login.sqlite3`

Ein in Python angelegtes Konto ist deshalb nicht automatisch in der
PHP-Anwendung vorhanden und umgekehrt.

## Konfiguration

Erstelle aus [`config.example.ini`](config.example.ini) eine lokale Datei
`config.ini` und trage vor dem Start die eigenen Werte ein. `config.ini` wird
über `.gitignore` nicht in Git gespeichert.

```ini
[app]
auth_level = 1
port = 5000
flask_secret = ein-langes-zufaelliges-geheimnis
wipe_database_on_start = false

[mail]
host = smtp.example.com
port = 587
security = starttls
user = deine-adresse@example.com
password = dein-app-passwort
from = deine-adresse@example.com

[php]
app_url = http://127.0.0.1/login/php
```

### Sicherheitsstufe

`auth_level` kann `1`, `2` oder `3` sein:

- `1`: E-Mail-Adresse und Passwort
- `2`: zusätzlich ein Code, der per E-Mail versendet wird
- `3`: zusätzlich ein TOTP-Code aus einer Authenticator-App

In der Python-Anwendung startet die Sicherheitsstufe nach jedem Neustart
immer mit Stufe 1. Nach der Anmeldung kann sie im Dashboard geändert werden;
die Einstellung gilt bis zum nächsten Neustart. In der PHP-Anwendung wird die
Stufe aus `config.ini` gelesen und über das Dashboard in dieser Datei
gespeichert.

Setze `wipe_database_on_start = true` **nur für Tests**. Dann löscht die
Python-Anwendung ihre Datenbank bei jedem Start vollständig. Die PHP-Datenbank
wird davon nicht gelöscht. Für normale Nutzung muss der Wert `false` sein.

Für die Python-Anwendung benötigen Stufe 2 und der Passwort-Reset gültige
SMTP-Daten. Bei Gmail oder Microsoft 365 wird meistens ein App-Passwort
benötigt.

Die PHP-Anwendung verwendet dagegen die native PHP-Funktion `mail()`. Ihre
SMTP-Verbindung muss deshalb in XAMPP/Sendmail eingerichtet werden; die
SMTP-Werte aus `[mail]` werden von PHP nicht für den Versand verwendet.

## Gespeicherte Daten und Zeitlimits

In jeder SQLite-Datenbank werden E-Mail-Adresse, Passwort-Hash, Zeitpunkt der
Passwortänderung, letzter erfolgreicher Login, TOTP-Schlüssel sowie gehashte
Passwort-Reset-Token gespeichert.

Das Passwort wird im Browser zunächst mit SHA-256 in einen Client-Schlüssel
umgewandelt. Der Server erhält nur diesen Schlüssel und speichert davon einen
langsamen Passwort-Hash. Das ursprüngliche Klartextpasswort wird nicht an den
Server übertragen und kann aus dem Hash nicht wiederhergestellt werden.

Aktuelle Zeitlimits:

- Sitzungen: 15 Minuten
- E-Mail-Codes: 5 Minuten
- TOTP-Codes: 30 Sekunden
- Passwort-Reset-Links: 15 Minuten

## Python lokal starten

Voraussetzung: Python 3.11 oder neuer.

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python python_app.py
```

Öffne danach <http://127.0.0.1:5000>.

Wenn PowerShell die Aktivierung blockiert, kann die Anwendung direkt mit dem
Interpreter aus der virtuellen Umgebung gestartet werden:

```powershell
.venv\Scripts\python.exe python_app.py
```

Die Python-Abhängigkeiten sind in [`requirements.txt`](requirements.txt)
aufgeführt. Der Port kann in `config.ini` unter `[app]` geändert werden.

## PythonAnywhere

Lade `python_app.py`, `requirements.txt`, `config.ini` und optional die
vorhandene `login.sqlite3` hoch. Installiere die Abhängigkeiten in der
virtuellen Umgebung:

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

Lege eine Web-App mit manueller Python-Konfiguration an. Verwende in der
WSGI-Datei den tatsächlichen Pfad zu deinem Projekt:

```python
import sys

sys.path.insert(0, "/home/DEIN_NAME/Gruber_Login")
from python_app import app as application
```

Trage bei **Virtualenv** `/home/DEIN_NAME/Gruber_Login/.venv` ein, klicke auf
**Reload** und öffne anschließend die PythonAnywhere-Adresse. SMTP-Zugangsdaten
bleiben ausschließlich in `config.ini` beziehungsweise in der geschützten
Serverkopie.

## PHP mit XAMPP

1. Starte Apache im XAMPP Control Panel.
2. Kopiere den gesamten Projektordner nach
   `C:\xampp\htdocs\login`, sodass `config.ini` neben dem Ordner `php` liegt.
3. Aktiviere in `php.ini` mindestens `pdo_sqlite` und `sqlite3`.
4. Konfiguriere XAMPP Sendmail für das SMTP-Konto, da die PHP-Anwendung
   `mail()` verwendet.
5. Öffne <http://127.0.0.1/login/php/>.

Wenn Apache auf Port 8080 läuft, ändere die PHP-URL in `config.ini`:

```ini
[php]
app_url = http://127.0.0.1:8080/login/php
```

`app_url` wird für Passwort-Reset-Links verwendet und muss daher auf den
öffentlichen PHP-Pfad zeigen. Die PHP-Datenbank `php/login.sqlite3` wird beim
ersten Aufruf automatisch angelegt.

## Empfohlener Testablauf

Teste die Python- und PHP-Anwendung getrennt, da sie eigene Datenbanken
verwenden:

1. Setze die Anwendung auf Sicherheitsstufe 1 und registriere ein Konto.
2. Teste die direkte Anmeldung und das Abmelden.
3. Setze die Sicherheitsstufe auf 2 und teste den E-Mail-Code.
4. Setze sie auf 3, registriere ein neues Konto und richte den angezeigten
   TOTP-Schlüssel in einer Authenticator-App ein.
5. Teste den Login mit dem sechsstelligen TOTP-Code.
6. Fordere einen Passwort-Reset an und prüfe, dass der Link nach 15 Minuten
   nicht mehr funktioniert.
7. Prüfe die Sitzungsablaufzeit von 15 Minuten.

Für den produktiven Betrieb zusätzlich HTTPS, sichere Cookie-Einstellungen,
Rate-Limiting und eine geschützte OTP-Einrichtung verwenden. SMTP-Passwörter
und Flask-Schlüssel niemals veröffentlichen oder in Git einchecken.
