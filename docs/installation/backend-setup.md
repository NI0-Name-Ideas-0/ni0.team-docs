# Backend

## SMTP-Server
Damit die Anwendung E-Mails verschicken kann wird ein SMTP-Server benötigt.
Hierfür kann ein beliebiger verwendet werden, wir empfehlen 
[https://www.opensmtpd.org/](OpenSMTPD).

Der SMTP-Server nutzt einen Port, für die Kommunikation. Diesen benötigen
wir später noch für die Konfiguration des Backends.
Das Socket des SMTP-Servers muss lediglich auf das lokale Interface gebunden werden,
da der Server keine E-Mails empfangen muss.

## Keycloak
Für die Authentifizierung und Authorisierung wird Keycloak genutzt.
Keycloak kann [hier](https://www.keycloak.org/downloads) heruntergeladen werden.

Für die Konfiguration von Keycloak verweisen wir zunächst auf die 
[Dokumentation von Keycloak](https://www.keycloak.org/documentation).
Hierbei sollten Sie für ChronoScope 
[ein Realm anlegen](https://www.keycloak.org/docs/latest/server_admin/index.html#_configuring-realms).

Damit das Backend mit Keycloak für das Organisationsmangagement kommunizieren kann,
ist außerdem das Anlegen eines Clients in Keycloak wichtig.
Am einfachsten ist hier das Vergeben der Admin Permissions.
Keycloak stellt dem Administrator dann eine Client-ID und ein Client-Secret zur Verfügung.
Dieses wird später ebenfalls für die Konfiguration des Spring-Backends benötigt.

## Datenbank
Um die Daten aus dem Backend zu persistieren, wird eine SQL-Datenbank benötigt.
Hier empfehlen wir [MariaDB](https://mariadb.org/download/) oder PostgreSQL.
Auch hier ist es ausreichend, wenn die Datenbank an das lokale Interface gebunden wird.

## Spring-Backend
Nun zum eigentlichen Kern der Anwendung. Sind alle abhängigkeiten installiert, kann es weiter gehen.
Das Backend kann am einfachsten als Docker Container gestartet werden.
Die neue Version erhällt man direkt 
[aus der GitHub Registry](https://github.com/NI0-Name-Ideas-0/ChronoScope-backend/pkgs/container/chronoscope-backend-backend).

Mit folgender Docker Compose Datei lässt sich das Backend später starten.
Vorher müssen aber noch die in der Compose Datei verwendeten Environment Variablen Konfiguriert werden.

| DB_USERNAME            | Der Username des Users in der Datenbank                                                                                            |
|------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| DB_PASSWORD            | Das Password des Users in der Datenbank                                                                                            |
| MYSQL_ROOT_PASSWORD    | Das Root Passwort für den MySQL Server. Dieses kann auch selbst konfiguriert werden und ist hier nur einfachkeitshalber angegeben. |
| SMTP_HOST              | Der Host unter dem der SMTP Server erreichbar ist. Läuft der SMTP-Server auf dem Host kann host.docker.internal verwendet werden   |
| SMTP_PORT              | Der Port des SMTP-Servers, normalerweise 25                                                                                        |
| KEYCLOAK_URL           | Die URL unter welcher Keycloak erreichbar sein wird                                                                                |
| KEYCLOAK_REALM         | Der Name des Realms für ChronoScope                                                                                                |
| KEYCLOAK_CLIENT_ID     | Die Client-ID des zuvor angelegten Clients in Keycloak                                                                             |
| KEYCLOAK_CLIENT_SECRET | Das Client-Secret des zuvor angelegten Clients in Keycloak                                                                         |

Diese können in eine .env Datei im Format
ENV=Value gespeichert werden. Docker ließt diese Datei automatisch aus.

```yml
version: '3.9'

services:
  chronoscope-backend:
    image: ghcr.io/ni0-name-ideas-0/chronoscope-backend-backend:latest
    ports:
      - "127.0.0.1:9120:8080"
    restart: always
    environment:
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
    extra_hosts:
      - "host.docker.internal:10.0.2.2"

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
