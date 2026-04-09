# renovate-config

Zentrale [Renovate](https://docs.renovatebot.com/)-Konfiguration für alle Cuveo-Projekte.

---

## Was ist Renovate?

Renovate ist ein Bot, der automatisch prüft ob npm-Packages in deinen Projekten veraltet oder unsicher sind — und dann einen **Pull Request** öffnet, der die Änderung direkt mitbringt.

Du schaust einmal in den PR, siehst welche Packages auf welche Version steigen, und mergst. Fertig.

**Was du nie mehr machen musst:**
- Manuell `npm outdated` laufen lassen und schauen was veraltet ist
- Manuell `package.json` anpassen und `npm install` laufen lassen
- Nach Security-Advisories Ausschau halten

---

## Was dieses Repo macht

Alle Cuveo-Projekte teilen sich dieselben Renovate-Regeln. Statt die Konfiguration in jedem Repo einzeln zu pflegen, liegt sie hier einmal zentral. Jedes Projekt referenziert nur:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>cuveodev/renovate-config"]
}
```

Wenn du den Schedule ändern willst oder eine neue Regel brauchst — **eine Datei hier**, alle Projekte übernehmen es automatisch.

---

## Wie es abläuft

### Monatliche Dependency-Updates

Am **1. eines jeden Monats** prüft Renovate alle registrierten Repos auf veraltete Packages. Alle Patch- und Minor-Updates kommen gebündelt als **ein einziger PR** — kein Spam, kein Rauschen.

```
1. Februar → Renovate öffnet PR: "chore: update non-major dependencies"
   └── react 18.2 → 18.3
   └── tailwindcss 4.0.1 → 4.1.0
   └── typescript 5.3 → 5.4
   └── ... (alle auf einmal)

Du: PR ansehen → mergen
```

Major-Updates (z.B. Next.js 16 → 17) kommen als **separater PR** mit dem Label `major-update` — damit sie nie unbemerkt in einem Sammel-PR untergehen.

### Sofortige Security-Alerts

Wenn eine bekannte Sicherheitslücke in einer deiner Dependencies entdeckt wird, öffnet Renovate **sofort** — unabhängig vom monatlichen Schedule — einen separaten PR mit dem Label `security`.

```
Irgendwann im Monat → CVE für Package X bekannt
→ Renovate öffnet innerhalb von Stunden PR: "fix: update [package] (security)"
→ Du mergst → Lücke geschlossen
```

---

## Konfiguration (`default.json`)

| Einstellung | Wert | Bedeutung |
|---|---|---|
| `schedule` | 1. des Monats | Monatlicher Lauf |
| `prConcurrentLimit` | 2 | Max. 2 offene PRs gleichzeitig |
| Patch + Minor | gruppiert | Kommen als ein PR |
| Major | separat | Eigener PR mit Label `major-update` |
| `automerge` | `false` | Du entscheidest immer selbst |
| Security-Alerts | sofort | Bypass des Schedules, Label `security` |

---

## Angebundene Projekte

| Projekt | Repo |
|---|---|
| cuveo.dev | `cuveodev/cuveo.dev` |
| Saformance | `cuveodev/saformance` |
| Taxi Nord Niebüll | `cuveodev/taxi-nord-niebuell` |

---

## Neues Projekt anbinden

1. `renovate.json` ins Root des Repos legen:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>cuveodev/renovate-config"]
}
```

2. Committen & pushen — Renovate erkennt die Datei automatisch.

---

## Konfiguration anpassen

Alle Änderungen an `default.json` in diesem Repo gelten sofort für alle angebundenen Projekte.

**Beispiele:**

```json
// Schedule auf wöchentlich ändern
"schedule": ["every monday"]

// Automerge für DevDep-Patches aktivieren
{
  "matchDepTypes": ["devDependencies"],
  "matchUpdateTypes": ["patch"],
  "automerge": true
}
```

---

## Verhältnis zu den bestehenden CI-Workflows

| Aufgabe | Vorher | Jetzt |
|---|---|---|
| Veraltete Packages erkennen | `dependency-maintenance.yml` wöchentlich | Renovate monatlich als PR |
| Security-Lücken melden | `dependency-maintenance.yml` → GitHub Issue | Renovate sofort als PR |
| Unabhängiger `npm audit` | — | `dependency-maintenance.yml` monatlich (Sicherheitsnetz) |
| Full-Stack Health-Check | `monthly-maintenance.yml` | unverändert |

Der `dependency-maintenance.yml`-Workflow läuft weiterhin monatlich als unabhängige Sicherheitsschicht — er prüft `npm audit` losgelöst von Renovate und eskaliert per GitHub Issue falls etwas gefunden wird.
