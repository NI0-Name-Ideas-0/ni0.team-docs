# Frontend

Diese Anleitung beschreibt, wie das ChronoScope-Frontend in einer Produktivumgebung gebaut und bereitgestellt wird. Das Frontend ist eine Angular-Anwendung und wird nach dem Build als statische Website ausgeliefert. Es benötigt deshalb keinen dauerhaft laufenden Node.js-Prozess auf dem Server.

!!! info "Kurzüberblick"
	Das Frontend wird einmal mit `pnpm build` gebaut. Anschließend werden die erzeugten Dateien über einen Webserver wie Nginx, Apache oder einen bestehenden Reverse Proxy ausgeliefert.

## Voraussetzungen

Für die Bereitstellung werden folgende Komponenten benötigt:

| Komponente | Empfehlung | Zweck |
|------------|------------|-------|
| Node.js | Version 22 LTS | Baut die Angular-Anwendung |
| pnpm | Version 10.x | Installiert die Abhängigkeiten reproduzierbar |
| Webserver | Nginx, Apache oder vergleichbar | Liefert die gebauten Dateien aus |
| HTTPS-Zertifikat | z. B. Let's Encrypt | Schützt Anmeldung und API-Kommunikation |
| ChronoScope-Backend | Bereits erreichbar | Wird vom Frontend über die API angesprochen |
| Keycloak | Bereits erreichbar | Übernimmt Anmeldung und Benutzeridentität |

!!! warning "HTTPS verwenden"
	In der Produktivumgebung sollte das Frontend ausschließlich über HTTPS erreichbar sein. Anmeldedaten und Zugriffstokens dürfen nicht über unverschlüsselte Verbindungen übertragen werden.

## Repository vorbereiten

Klonen Sie das Frontend-Repository auf eine Maschine, auf der der Produktionsbuild erzeugt werden soll:

```powershell
git clone <repository-url>
Set-Location ChronoScope-frontend
```

Aktivieren Sie anschließend die im Projekt festgelegte pnpm-Version über Corepack:

```powershell
corepack enable
corepack prepare pnpm@10.33.0 --activate
pnpm --version
```

Installieren Sie danach die Abhängigkeiten:

```powershell
pnpm install
```

!!! tip "Build-Server statt Produktivserver"
	Node.js und pnpm werden nur zum Erzeugen der statischen Dateien benötigt. Auf dem eigentlichen Webserver müssen diese Werkzeuge nicht installiert sein, wenn die fertigen Build-Dateien dort nur ausgeliefert werden.

## Backend-Adresse konfigurieren

Das Produktiv-Frontend verwendet die Datei `src/environments/environment.prod.ts`. Tragen Sie dort die öffentlich erreichbare Backend-URL ein:

```ts
export const environment = {
  production: true,
  apiUrl: 'https://chronoscope.example.org/api',
};
```

Die URL muss aus Sicht des Browsers erreichbar sein. Wenn das Backend über denselben Host per Reverse Proxy unter `/api` veröffentlicht wird, ist eine URL wie `https://chronoscope.example.org/api` üblich.

| Situation | Beispielwert für `apiUrl` |
|-----------|---------------------------|
| Backend unter derselben Domain mit API-Pfad | `https://chronoscope.example.org/api` |
| Backend unter eigener Subdomain | `https://api.chronoscope.example.org` |
| Testsystem beim Kunden | `https://test-chronoscope.example.org/api` |

!!! warning "CORS beachten"
	Wenn Frontend und Backend über unterschiedliche Domains erreichbar sind, muss das Backend Anfragen von der Frontend-Domain erlauben. Andernfalls blockiert der Browser die API-Aufrufe.

## Anmeldung in Keycloak konfigurieren

Das Frontend nutzt OpenID Connect über Keycloak. Die Weiterleitungsadresse nach der Anmeldung wird aus der geöffneten Frontend-Adresse gebildet. Dadurch muss die öffentliche Frontend-URL im Keycloak-Client erlaubt sein.

Im Keycloak-Client `chronoscope` sollten mindestens diese Werte gesetzt werden:

| Einstellung | Beispiel |
|-------------|----------|
| Valid redirect URIs | `https://chronoscope.example.org/*` |
| Web origins | `https://chronoscope.example.org` |
| Client type | Public client |
| Standard flow | Aktiviert |

Wenn für den Kunden ein eigener Realm, eine andere Keycloak-URL oder eine andere Client-ID genutzt wird, müssen diese Werte zusätzlich im Frontend-Code angepasst und anschließend neu gebaut werden.

!!! note "Aktuelle Voreinstellung"
	Die Anwendung ist derzeit auf den Keycloak-Realm `https://auth.ni0.team/realms/ni0` und die Client-ID `chronoscope` ausgelegt. Für eine kundeneigene Installation sollte geprüft werden, ob diese Werte übernommen oder ersetzt werden sollen.

## Produktionsbuild erstellen

Führen Sie vor dem Build optional die Tests aus:

```powershell
pnpm test
```

Erstellen Sie anschließend den Produktionsbuild:

```powershell
pnpm build
```

Angular erzeugt die auslieferbaren Dateien im Ordner `dist`. Je nach Angular-Version liegt die eigentliche Website im Projektordner selbst oder in einem darunterliegenden `browser`-Ordner.

Prüfen Sie nach dem Build, welcher Ordner eine `index.html` enthält:

```powershell
Get-ChildItem -Path .\dist -Filter index.html -Recurse
```

Diesen Ordner veröffentlichen Sie über den Webserver.

## Webserver einrichten

Das Frontend muss als Single-Page-Application ausgeliefert werden. Das bedeutet: Direkte Aufrufe wie `/calendar` oder `/settings` dürfen nicht zu einem 404-Fehler führen, sondern müssen auf die `index.html` zurückfallen.

### Beispiel mit Nginx

In diesem Beispiel liegen die gebauten Dateien unter `/var/www/chronoscope` und das Backend wird unter `/api` weitergeleitet:

```nginx
server {
	listen 80;
	server_name chronoscope.example.org;

	root /var/www/chronoscope;
	index index.html;

	location /api/ {
		proxy_pass http://127.0.0.1:9120/;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
	}

	location / {
		try_files $uri $uri/ /index.html;
	}
}
```

Für den Produktivbetrieb sollte zusätzlich HTTPS aktiviert werden, zum Beispiel über ein Let's-Encrypt-Zertifikat.

### Beispiel mit Apache

Bei Apache kann das Frontend mit `mod_rewrite` ebenfalls auf die `index.html` zurückfallen:

```apache
<VirtualHost *:80>
	ServerName chronoscope.example.org
	DocumentRoot /var/www/chronoscope

	<Directory /var/www/chronoscope>
		Require all granted
		RewriteEngine On
		RewriteBase /
		RewriteRule ^index\.html$ - [L]
		RewriteCond %{REQUEST_FILENAME} !-f
		RewriteCond %{REQUEST_FILENAME} !-d
		RewriteRule . /index.html [L]
	</Directory>

	ProxyPass /api/ http://127.0.0.1:9120/
	ProxyPassReverse /api/ http://127.0.0.1:9120/
</VirtualHost>
```

## Dateien veröffentlichen

Kopieren Sie den Ordner, der die `index.html` enthält, in das Webserver-Verzeichnis. Unter Windows kann das zum Beispiel so vorbereitet werden:

```powershell
Copy-Item -Path .\dist\ChronoScope-frontend\browser\* -Destination <zielordner> -Recurse -Force
```

Falls die `index.html` direkt unter `dist\ChronoScope-frontend` liegt, passen Sie den Quellpfad entsprechend an.

Nach dem Kopieren muss der Webserver die neuen Dateien ausliefern. Je nach Umgebung reicht ein Reload der Konfiguration aus.

## Abnahme prüfen

Nach der Bereitstellung sollten folgende Punkte geprüft werden:

| Prüfung | Erwartetes Ergebnis |
|---------|---------------------|
| Frontend-URL öffnen | Die ChronoScope-Oberfläche wird geladen |
| Seite neu laden | Die Anwendung bleibt erreichbar und zeigt keinen 404-Fehler |
| Anmeldung starten | Der Browser leitet zu Keycloak weiter |
| Anmeldung abschließen | Der Browser kehrt zur Frontend-URL zurück |
| API-Aufruf auslösen | Aufgaben, Organisationen oder Benutzerdaten werden geladen |
| HTTPS prüfen | Browser zeigt eine sichere Verbindung an |

!!! success "Bereit für die Übergabe"
	Wenn Anmeldung, API-Kommunikation und direkter Seitenreload funktionieren, ist das Frontend aus Sicht der Produktivbereitstellung einsatzbereit.

## Häufige Probleme

| Problem | Mögliche Ursache | Lösung |
|---------|------------------|--------|
| Die Anwendung lädt, aber Daten fehlen | Backend-URL ist falsch oder nicht erreichbar | `apiUrl` prüfen und Frontend neu bauen |
| Browser meldet CORS-Fehler | Backend erlaubt die Frontend-Domain nicht | CORS-Konfiguration im Backend ergänzen |
| Login endet mit Redirect-Fehler | Frontend-URL ist in Keycloak nicht erlaubt | Valid redirect URIs und Web origins prüfen |
| Direkter Aufruf einer Unterseite zeigt 404 | Webserver leitet SPA-Routen nicht auf `index.html` um | `try_files` oder Rewrite-Regel ergänzen |
| Änderungen an `environment.prod.ts` erscheinen nicht | Alte Build-Dateien werden ausgeliefert | Neu bauen, Dateien erneut kopieren und Cache prüfen |
