# Schreibguide

Diese Seite hilft dem Team, später konsistente, nutzerfreundliche Dokumentation zu schreiben.

## Stil

-- Schreibe für Anwenderinnen und Anwender, nicht für Entwickler.
-- Erkläre zuerst den Nutzen, danach Details.
- Verwende kurze Abschnitte und konkrete Beispiele.
-- Nutze Screenshots für sichtbare Oberflächen und Tabellen für Vergleiche.
- Vermeide interne Begriffe wie DTO, Repository oder Spring-Service.

## Bilder verwenden

Lege Bilder unter `docs/assets/images` ab. MkDocs kopiert diesen Ordner beim Build automatisch in die Website.

Ein Bild bindest du relativ zur aktuellen Markdown-Datei ein:

```markdown
![Beschreibung des Screenshots](assets/images/calendar-overview.png)
```

Aus Unterordnern heraus gehst du eine Ebene zurück:

```markdown
![Aufgabenliste](../assets/images/task-list.png)
```

## Empfohlene Dateinamen

- `calendar-overview.png`
- `task-list.png`
- `create-task-dialog.png`
- `planning-conflict.png`

## Gute Bildregeln

-- Nutze PNG für Screenshots.
-- Nutze SVG für Logos oder einfache Grafiken.
- Schneide Screenshots so zu, dass der relevante Bereich klar sichtbar ist.
- Vermeide echte personenbezogene Daten in Beispielen.
-- Gib jedem Bild einen aussagekräftigen Alternativtext.
