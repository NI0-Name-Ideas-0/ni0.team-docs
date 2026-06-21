# Backend

Diese Anleitung beschreibt, wie das ChronoScope-Backend in einer Produktivumgebung eingerichtet und gestartet wird. Das Backend ist eine Spring-Boot-Anwendung und stellt die REST-API für Aufgaben, Arbeitszeiten, Planung, Organisationen und Benutzeridentität bereit.

!!! info "Kurzüberblick"
    Das Backend wird im Produktivbetrieb als Docker-Container gestartet. Zusätzlich werden eine SQL-Datenbank, ein SMTP-Server und eine Keycloak-Installation benötigt.

## Voraussetzungen

Für den Betrieb werden folgende Komponenten benötigt:

| Komponente | Empfehlung | Zweck |
|------------|------------|-------|
| Docker | Aktuelle stabile Version | Startet das Backend als Container |
| Docker Compose | Aktuelle stabile Version | Verwaltet Backend und Datenbank gemeinsam |
| Datenbank | MySQL, MariaDB oder kompatibel | Speichert Aufgaben, Arbeitszeiten und Planungsdaten |
| Keycloak | Bereits erreichbar | Prüft Tokens und verwaltet Organisationen |
| SMTP-Server | Lokal oder extern erreichbar | Versendet Einladungen und Account-Link-E-Mails |
| Reverse Proxy | Nginx, Apache oder vergleichbar | Veröffentlicht die API kontrolliert nach außen |
| HTTPS-Zertifikat | z. B. Let's Encrypt | Schützt Login- und API-Kommunikation |

!!! warning "HTTPS verwenden"
    Das Backend sollte in der Produktivumgebung nur über HTTPS erreichbar sein. Zugriffstokens und personenbezogene Daten dürfen nicht über unverschlüsselte Verbindungen übertragen werden.

## Datenbank vorbereiten

ChronoScope benötigt eine SQL-Datenbank. Für eine einfache produktionsnahe Installation kann MySQL direkt über Docker Compose mitgestartet werden. In einer bestehenden Kundenumgebung kann stattdessen auch eine vorhandene MySQL- oder MariaDB-Instanz verwendet werden.

Für eine externe Datenbank werden folgende Informationen benötigt:

| Wert | Beispiel |
|------|----------|
| Datenbankname | `chronoscope` |
| JDBC-URL | `jdbc:mysql://mysql:3306/chronoscope` |
| Benutzername | `chronoscope` |
| Passwort | Starkes individuelles Passwort |

!!! tip "Datenbank nicht öffentlich freigeben"
    Die Datenbank muss nur vom Backend erreichbar sein. Sie sollte nicht direkt aus dem Internet erreichbar sein.

## SMTP-Server vorbereiten

Das Backend verwendet SMTP, um E-Mails zu versenden. Dazu gehören zum Beispiel Einladungen oder Nachrichten zum Verknüpfen von Accounts.

Der SMTP-Server kann intern oder extern betrieben werden. Wichtig ist, dass das Backend den Host und Port erreichen kann.

| Einstellung | Beispiel |
|-------------|----------|
| SMTP-Host | `smtp.example.org` |
| SMTP-Port | `25` oder `587` |

Wenn der SMTP-Server auf demselben Host wie Docker läuft, kann je nach Umgebung `host.docker.internal` verwendet werden.

## Keycloak vorbereiten

ChronoScope nutzt Keycloak für Authentifizierung, Benutzeridentität und Organisationsverwaltung. Für das Backend wird ein vertraulicher Keycloak-Client benötigt, über den das Backend administrative Organisationsfunktionen ausführen kann.

In Keycloak sollten vorbereitet werden:

| Einstellung | Beschreibung |
|-------------|--------------|
| Realm | Realm für ChronoScope, zum Beispiel `ni0` |
| Backend-Client | Vertraulicher Client für Backend-Administrationszugriffe |
| Client-ID | Wird als `KEYCLOAK_CLIENT_ID` im Backend gesetzt |
| Client-Secret | Wird als `KEYCLOAK_CLIENT_SECRET` im Backend gesetzt |
| Berechtigungen | Der Client benötigt Rechte zur Verwaltung der relevanten Organisationen und Benutzer |

!!! note "Frontend separat konfigurieren"
    Für die Anmeldung im Browser benötigt auch das Frontend einen Keycloak-Client mit erlaubten Redirect-URLs. Diese Konfiguration ist in der Frontend-Installationsanleitung beschrieben.

## Umgebungsvariablen konfigurieren

Legen Sie im Verzeichnis der Docker-Compose-Datei eine `.env`-Datei an. Docker Compose liest diese Datei automatisch aus.

```properties
DB_USERNAME=chronoscope
DB_PASSWORD=<sicheres-datenbank-passwort>
MYSQL_ROOT_PASSWORD=<sicheres-root-passwort>

SMTP_HOST=smtp.example.org
SMTP_PORT=25

KEYCLOAK_URL=https://auth.example.org
KEYCLOAK_REALM=ni0
KEYCLOAK_CLIENT_ID=chronoscope-admin
KEYCLOAK_CLIENT_SECRET=<keycloak-client-secret>
```

| Variable | Bedeutung |
|----------|-----------|
| `DB_USERNAME` | Benutzername für die Datenbank |
| `DB_PASSWORD` | Passwort des Datenbankbenutzers |
| `MYSQL_ROOT_PASSWORD` | Root-Passwort für den MySQL-Container |
| `SMTP_HOST` | Hostname oder IP-Adresse des SMTP-Servers |
| `SMTP_PORT` | Port des SMTP-Servers, häufig `25` oder `587` |
| `KEYCLOAK_URL` | Basis-URL der Keycloak-Installation |
| `KEYCLOAK_REALM` | Name des Keycloak-Realms |
| `KEYCLOAK_CLIENT_ID` | Client-ID des Backend-Clients |
| `KEYCLOAK_CLIENT_SECRET` | Client-Secret des Backend-Clients |

!!! warning "Secrets schützen"
    Die `.env`-Datei enthält Passwörter und Secrets. Sie sollte nicht in ein Git-Repository eingecheckt und nur für Administratorinnen und Administratoren lesbar sein.

## Backend mit Docker Compose starten

Die aktuelle Backend-Version kann als Container aus der GitHub Container Registry bezogen werden:

[ChronoScope Backend Package](https://github.com/NI0-Name-Ideas-0/ChronoScope-backend/pkgs/container/chronoscope-backend-backend)

Mit folgender `docker-compose.yml` wird das Backend gemeinsam mit MySQL gestartet:

```yml
version: '3.9'

services:
  chronoscope-backend:
    image: ghcr.io/ni0-name-ideas-0/chronoscope-backend-backend:latest
    restart: always
    ports:
      - "127.0.0.1:9120:9020"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SERVER_PORT=9020
      - DB_URL=jdbc:mysql://mysql:3306/chronoscope
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - SMTP_HOST=${SMTP_HOST}
      - SMTP_PORT=${SMTP_PORT}
      - KEYCLOAK_URL=${KEYCLOAK_URL}
      - KEYCLOAK_REALM=${KEYCLOAK_REALM}
      - KEYCLOAK_CLIENT_ID=${KEYCLOAK_CLIENT_ID}
      - KEYCLOAK_CLIENT_SECRET=${KEYCLOAK_CLIENT_SECRET}
    depends_on:
      - mysql
    networks:
      - chrononet

  mysql:
    image: mysql:8.0
    restart: always
    environment:
      - MYSQL_DATABASE=chronoscope
      - MYSQL_USER=${DB_USERNAME}
      - MYSQL_PASSWORD=${DB_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - chrononet

volumes:
  mysql_data:

networks:
  chrononet:
```

Starten Sie anschließend die Container:

```powershell
docker compose up -d
```

Prüfen Sie danach, ob die Container laufen:

```powershell
docker compose ps
```

Die API ist in diesem Beispiel lokal auf dem Server unter `http://127.0.0.1:9120` erreichbar. Nach außen sollte sie über einen Reverse Proxy und HTTPS veröffentlicht werden.

!!! note "Portbindung"
    Die Compose-Datei bindet den Backend-Port bewusst nur an `127.0.0.1`. Dadurch ist das Backend nicht direkt aus dem Internet erreichbar, sondern nur über den vorgeschalteten Reverse Proxy.

## Externe Datenbank verwenden

Wenn der Kunde bereits eine Datenbank betreibt, kann der `mysql`-Service aus der Compose-Datei entfernt werden. Setzen Sie `DB_URL`, `DB_USERNAME` und `DB_PASSWORD` dann passend zur externen Datenbank.

Beispiel:

```properties
DB_URL=jdbc:mysql://db.example.internal:3306/chronoscope
DB_USERNAME=chronoscope
DB_PASSWORD=<sicheres-datenbank-passwort>
```

Der Datenbankserver muss vom Backend-Container aus erreichbar sein.

## Reverse Proxy einrichten

Für den produktiven Zugriff sollte das Backend hinter einem Reverse Proxy betrieben werden. Das passt besonders gut, wenn das Frontend unter derselben Domain läuft und API-Aufrufe unter `/api` weitergeleitet werden.

### Beispiel mit Nginx

```nginx
server {
    listen 80;
    server_name chronoscope.example.org;

    location /api/ {
        proxy_pass http://127.0.0.1:9120/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Für den Produktivbetrieb sollte zusätzlich HTTPS aktiviert werden. Wenn Frontend und Backend unter derselben Domain laufen, kann der Frontend-Webserver dieselbe `/api`-Weiterleitung enthalten.

## CORS konfigurieren

Das Backend erlaubt im Produktivprofil nur konfigurierte Ursprünge. In der Standardkonfiguration ist `https://chronoscope.ni0.team` eingetragen. Für eine Kundeninstallation muss die öffentlich genutzte Frontend-Adresse erlaubt werden.

Wenn die Anwendung unter `https://chronoscope.example.org` läuft, sollte dieser Ursprung in der Backend-Konfiguration hinterlegt sein:

```yml
cors:
  allowed-origins:
    - https://chronoscope.example.org
```

Wird das Backend als unverändertes Container-Image betrieben und das Frontend greift über dieselbe Domain auf `/api` zu, entsteht im Browser kein Cross-Origin-Aufruf. In diesem Fall ist in der Regel keine zusätzliche CORS-Freigabe nötig.

!!! warning "Keine Wildcards in Produktion"
    Verwenden Sie in der Produktivumgebung keine pauschale Freigabe wie `*`, wenn echte Benutzer- oder Organisationsdaten verarbeitet werden.

## Betrieb prüfen

Nach dem Start sollten folgende Punkte geprüft werden:

| Prüfung | Erwartetes Ergebnis |
|---------|---------------------|
| Containerstatus prüfen | Backend und Datenbank laufen |
| Logs prüfen | Keine wiederholten Datenbank-, SMTP- oder Keycloak-Fehler |
| Backend lokal erreichen | Reverse Proxy kann `127.0.0.1:9120` erreichen |
| Frontend anmelden | Keycloak-Login funktioniert |
| API-Aufruf auslösen | Aufgaben, Organisationen oder Identitätsdaten werden geladen |
| E-Mail-Funktion testen | Einladung oder Account-Link-Mail wird versendet |

Nützliche Befehle:

```powershell
docker compose logs -f chronoscope-backend
docker compose restart chronoscope-backend
docker compose pull
docker compose up -d
```

!!! success "Bereit für die Übergabe"
    Wenn Datenbankverbindung, Keycloak-Anmeldung, API-Aufrufe und E-Mail-Versand funktionieren, ist das Backend aus Sicht der Produktivbereitstellung einsatzbereit.

## Häufige Probleme

| Problem | Mögliche Ursache | Lösung |
|---------|------------------|--------|
| Backend startet nicht | Pflichtvariable fehlt | `.env` prüfen, besonders `DB_PASSWORD` und Keycloak-Secrets |
| Datenbankfehler beim Start | Datenbank nicht erreichbar oder falsche Zugangsdaten | `DB_URL`, Benutzername und Passwort prüfen |
| Login funktioniert, API liefert aber 401 | Keycloak-Realm oder Token-Issuer passt nicht | Realm, Frontend-Keycloak-Konfiguration und Backend-Keycloak-URL abgleichen |
| Organisationsfunktionen schlagen fehl | Backend-Client hat zu wenige Rechte | Berechtigungen des Keycloak-Clients prüfen |
| Browser meldet CORS-Fehler | Frontend-Origin ist nicht erlaubt | `cors.allowed-origins` passend zur Frontend-URL setzen |
| E-Mails werden nicht versendet | SMTP-Host oder Port ist falsch | SMTP-Erreichbarkeit aus dem Backend-Container prüfen |
