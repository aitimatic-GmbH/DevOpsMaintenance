# Setup Guide: KeyVault Managed Identity

Diese Anleitung beschreibt die Einrichtung und Nutzung des KeyVault-Service über das DevOpsMaintenance Repository.

> **Voraussetzung:** Folgen Sie zuerst dem [Setup Guide](setup-guide.md) (Schritt 1-3), bevor Sie hier fortfahren.

---

## 1. GitHub Secrets

Der KeyVault-Service verwendet dieselben Secrets wie der Monitoring-Service. Diese sind bereits im Repository konfiguriert.

| Secret Name | Beschreibung |
|-------------|--------------|
| `AZURE_WRITE_CLIENT_ID` | Service Principal mit Contributor-Berechtigung (Deployment) |
| `AZURE_WRITE_TENANT_ID` | Tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID |
| `PAT_SUBMODULE_CHECKOUT` | Personal Access Token für Submodule-Zugriff |

> Keine neuen Secrets erforderlich.

---

## 2. Umgebungen konfigurieren

Der Service unterstützt drei Umgebungen: `dev`, `staging`, `prod`. Die Parameter liegen im Submodule unter `infra/environments/`:

- `infra/environments/dev.bicepparam`
- `infra/environments/staging.bicepparam`
- `infra/environments/prod.bicepparam`

### Optionale Komponenten aktivieren

In den `.bicepparam`-Dateien können einzelne Komponenten aktiviert werden:

| Parameter | Standard | Beschreibung |
|-----------|----------|--------------|
| `deployWebApp` | `false` | App Service deployen |
| `deployNetworking` | `false` | VNet und Private Endpoints |
| `deployFunctions` | `false` | Azure Functions deployen |

---

## 3. Workflows ausführen

### 4.1 Validation (automatisch)

Der Workflow `Run KeyVault Validate` läuft automatisch bei:
- Push auf `feat/**` Branches (Änderungen in `services/service-keyvault-managed-identity/`)
- Pull Requests nach `main`

**Was passiert:**
- Bicep Lint & Build (ohne Azure-Credentials)
- Template-Validierung gegen Azure für alle drei Umgebungen (Matrix: dev / staging / prod)

**Manuell starten:** Actions → **Run KeyVault Validate** → Run workflow

### 4.2 Deployment (manuell)

Der Workflow `Run KeyVault Deploy` wird manuell gestartet:

1. Actions → **Run KeyVault Deploy** → Run workflow
2. Umgebung auswählen (`dev`, `staging`, `prod`)
3. Optional: What-If Vorschau überspringen

**Was passiert:**
1. **What-If:** Zeigt alle geplanten Änderungen ohne echtes Deployment
2. **Deploy:** Wartet auf Environment-Approval (konfigurierbar), deployt dann nach Azure

---

## 4. Ergebnisse prüfen

Nach erfolgreichem Deployment:

1. **Azure Portal** → Ressourcengruppe öffnen
2. **Key Vault** → Secrets und Access Policies prüfen
3. **Managed Identities** → Zuweisungen verifizieren

---

## Weiterführende Dokumentation

- [Architektur](../services/service-keyvault-managed-identity/docs/architecture.md)
- [Konzepte](../services/service-keyvault-managed-identity/docs/concepts.md)
- [Deployment Guide](../services/service-keyvault-managed-identity/docs/deployment-guide.md)
- [Backlog](../services/service-keyvault-managed-identity/docs/BACKLOG.md)

---

← Zurück zum [Setup Guide](setup-guide.md) | [Dokumentations-Index](index.md)
