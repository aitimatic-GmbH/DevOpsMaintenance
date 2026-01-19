# DevOps Maintenance Service Katalog

Willkommen beim `DevOpsMaintenance` Service Katalog! Dieses Repository bündelt eine Sammlung von wiederverwendbaren, produktionsreifen Infrastructure-as-Code (IaC) und Automatisierungs-Modulen zur Wartung und Optimierung von Cloud-Umgebungen.

Jeder Service in diesem Katalog ist als eigenständiges, in sich geschlossenes Produkt konzipiert und über Git-Submodule integriert.

## Das Konzept: Wiederverwendbare Best Practices

Anstatt das Rad für Standard-Aufgaben wie Kostenkontrolle oder Monitoring jedes Mal neu zu erfinden, bietet dieser Katalog kuratierte, sofort einsetzbare Lösungen. Die Services sind:

*   **Konfigurierbar:** Jeder Service lässt sich über zentrale Konfigurationsdateien (z.B. `*.config.json`) an spezifische Bedürfnisse anpassen, ohne den Code ändern zu müssen.
*   **Automatisiert:** Die Kernlogik wird durch robuste Pipelines (z.B. GitHub Actions) abgebildet, die wiederholbare und zuverlässige Ergebnisse liefern.
*   **Sicher:** Wir setzen auf moderne Sicherheitsprinzipien wie passwortlose OIDC-Verbindungen und das Prinzip der geringsten Rechte (Least Privilege).

## Erste Schritte

Um das Repository und alle enthaltenen Services für die eigene Nutzung vorzubereiten, folgen Sie bitte unserer [**Quickstart-Anleitung**](./docs/quickstart.md).

---

## Verfügbare Services

### 1. Azure FinOps Manager

Ein leistungsstarker Service zur proaktiven Senkung von Cloud-Kosten in Microsoft Azure.

| Service | `azure-finops-manager` |
| :--- | :--- |
| **Ziel** | Identifiziert und meldet automatisch Kostenfallen wie ungenutzte Disks, alte Snapshots oder überdimensionierte Ressourcen. |
| **Ansatz** | Ein zentraler, konfigurierbarer GitHub Actions Workflow führt Analyse- und Reporting-Jobs aus. Die Ergebnisse werden in einem management-tauglichen Markdown-Report aufbereitet. |
| **Technologie** | GitHub Actions, OIDC für sichere Azure-Verbindung, JSON-Konfiguration. |
| **Details** | Für eine vollständige Beschreibung und die Setup-Anleitung, siehe die [**README des `azure-finops-manager`**](./services/azure-finops-manager/README.md). |

---
*... weitere Services werden hier in Zukunft hinzugefügt ...*

---

## Beitrag leisten

Informationen zur Mitarbeit an diesem Projekt werden zu einem späteren Zeitpunkt bereitgestellt.
