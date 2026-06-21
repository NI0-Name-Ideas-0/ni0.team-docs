# Organisations-Admins in Keycloak konfigurieren

Die Verwaltung von Organisations Administratoren ist standartmäßig nicht in Keycloak integriert.
Um diese umzusetzen, verwenden wir die Keycloak Gruppen.

Dabei muss eine Gruppe "org-admins" angelegt werden.
Für jede Organisation müssen entsprechende Untergruppen angelegt werden,
wie im folgenden Bild gezeigt.

Administratoren werden über die Angehörigkeit zu diesen Gruppen identifiziert.

Wie Gruppen erstellt werden können ist
[hier](https://wjw465150.gitbooks.io/keycloak-documentation/content/server_admin/topics/groups.html) dokumentiert.

Das Backend fragt falls nötig automatisch diese Gruppen aus Keycloak ab um die Rechte zu prüfen.
