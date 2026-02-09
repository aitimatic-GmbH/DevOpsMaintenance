# DevOps Maintenance Service Katalog

Willkommen beim `DevOpsMaintenance` Service Katalog! Dieses Repository bündelt eine Sammlung von wiederverwendbaren, produktionsreifen Infrastructure-as-Code (IaC) und Automatisierungs-Modulen zur Wartung und Optimierung von Cloud-Umgebungen.

Jeder Service ist eigenständig und über Git-Submodule integriert.

## Konzept: Wiederverwendbare Best Practices

Dieser Katalog bietet kuratierte, sofort einsetzbare Lösungen für Standardaufgaben wie Kostenkontrolle oder Monitoring:

* **Konfigurierbar:** Anpassung über zentrale Konfigurationsdateien (z.B. `*.config.json`), ohne Codeänderung.
* **Automatisiert:** Robuste Pipelines (z.B. GitHub Actions) liefern wiederholbare Ergebnisse.
* **Sicher:** Passwortlose OIDC-Verbindungen und Prinzip der geringsten Rechte (Least Privilege).

## Erste Schritte

1. Repository klonen: `git clone --recurse-submodules <repo-url>`
2. Submodule initialisieren und aktualisieren: `git submodule update --init --recursive`
3. [Quickstart-Anleitung](./docs/quickstart.md) folgen, um die Services zu nutzen.

---

## Verfügbare Services

### 1. Azure FinOps Manager

Ein leistungsstarker Service zur proaktiven Senkung von Cloud-Kosten in Microsoft Azure.

| Service | `azure-finops-manager` |
| :--- | :--- |
| **Ziel** | Identifiziert und meldet automatisch Kostenfallen wie ungenutzte Disks, alte Snapshots oder überdimensionierte Ressourcen. |
| **Ansatz** | Ein zentraler, konfigurierbarer GitHub Actions Workflow führt Analyse- und Reporting-Jobs aus. Die Ergebnisse werden in einem management-tauglichen Markdown-Report aufbereitet. |
| **Technologie** | GitHub Actions, OIDC für sichere Azure-Verbindung, JSON-Konfiguration. |
| **Details** | siehe die [**README des `azure-finops-manager`**](./services/azure-finops-manager/README.md). |

*weitere Services werden hier in Zukunft hinzugefügt.*

---

## ⚠️ Hinweis
- Dieses Repository ist eine Referenzimplementierung für DevOps- und FinOps-Praktiken.
- Keine externen Beiträge, kein Support, keine garantierten Updates.
- Forken und anpassen erlaubt – Nutzung auf eigenes Risiko.

---

## Lizenz

MIT License – siehe [LICENSE](LICENSE)  
Copyright (c) 2026 aitimatic GmbH

**Haftungsausschluss:** Bereitstellung „as-is“ ohne Gewährleistung. Autoren und Inhaber haften nicht für Schäden oder Ansprüche.

Siehe [SECURITY.md](SECURITY.md) für Security-Richtlinien.

---

## Beiträge

Dieses Repository akzeptiert keine externen Beiträge. Siehe [CONTRIBUTING.md](CONTRIBUTING.md) für Details.


---
