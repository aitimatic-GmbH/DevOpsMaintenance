# Setup Guide: Azure Managed Monitoring Essentials

Diese Anleitung beschreibt die Einrichtung und Nutzung des Monitoring-Service über das DevOpsMaintenance Repository.

> **Voraussetzung:** Folgen Sie zuerst dem [Setup Guide](setup-guide.md) (Schritt 1-3), bevor Sie hier fortfahren.

---

## 1. Azure Service Principal erstellen

Der Monitoring-Service benötigt einen Service Principal mit erweiterten Berechtigungen.

### Service Principal erstellen (Contributor)
```bash
az ad sp create-for-rbac \
  --name "sp-devops-monitoring-contributor" \
  --role "Contributor" \
  --scopes /subscriptions/IHRE-SUBSCRIPTION-ID
```

**Notieren Sie:**
- `appId` → `AZURE_WRITE_CLIENT_ID`
- `tenant` → `AZURE_WRITE_TENANT_ID`

### Erweiterte Berechtigungen

Der Service Principal benötigt zwei zusätzliche Rollen für Policy- und Rollen-Management:

| Rolle | Zweck |
|-------|-------|
| **Resource Policy Contributor** | Policy-Zuweisungen erstellen |
| **User Access Administrator** (eingeschränkt) | Rollen für Policy Managed Identities zuweisen |

Die Rolle "User Access Administrator" wird durch eine ABAC-Condition auf eine Whitelist von Built-In Rollen eingeschränkt.

**Vollständige Anleitung:** [role-assignment-guide.md](../services/azure-managed-monitoring/docs/role-assignment-guide.md)

---

## 2. Ressourcengruppe erstellen

Die Monitoring-Infrastruktur wird in einer dedizierten Ressourcengruppe deployed.

```bash
az group create \
  --name rg-monitoring-prod \
  --location westeurope
```

---

## 3. GitHub Secrets konfigurieren

**Settings → Secrets and variables → Actions → New repository secret**

| Secret Name | Beschreibung | Wert |
|-------------|--------------|------|
| `AZURE_WRITE_CLIENT_ID` | App ID des Service Principals | `<appId>` |
| `AZURE_WRITE_TENANT_ID` | Tenant ID | `<tenant>` |
| `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID | `<subscription-id>` |
| `MON_CONFIG_JSON` | JSON-Override für sensible Konfigurationswerte | Siehe unten |

### MON_CONFIG_JSON erstellen

Das Secret enthält einen JSON-String, der die sensiblen Werte aus dem Template (`config/example.config.json`) überschreibt.

**Minimales Override-Beispiel:**
```json
{
  "deploymentSettings": {
    "resourceGroupName": "rg-monitoring-prod"
  },
  "bicepParameters": {
    "alertReceiverEmail": { "value": "ops@firma.de" },
    "workspaceName": { "value": "log-monitoring-prod" },
    "actionGroupName": { "value": "ag-monitoring-prod" }
  }
}
```

**Secret setzen:**
```bash
MON_CONFIG=$(jq -c . < override.json)
gh secret set MON_CONFIG_JSON --body "$MON_CONFIG"
```

Detaillierte Konfiguration: [configuration-guide.md](../services/azure-managed-monitoring/docs/configuration-guide.md)

---

## 4. Workflows ausführen

### 4.1 Validation (automatisch)

Der Workflow `Run Monitoring Verify` läuft automatisch bei:
- Push auf `feat/**` Branches (Änderungen in `services/azure-managed-monitoring/`)
- Pull Requests nach `main`

**Was passiert:**
- Bicep-Syntax Check (`az bicep build`)
- Template-Validierung gegen Azure (`az deployment group validate`)
- Kein echtes Deployment

**Manuell starten:** Actions → **Run Monitoring Verify** → Run workflow

### 4.2 Deployment (manuell)

Der Workflow `Run Monitoring Deploy` wird manuell gestartet und führt drei Phasen aus:

```
Phase 1: Foundation    → Log Analytics Workspace, Policy Initiatives, DCRs
Phase 2: Assignment    → Policy-Zuweisungen, RBAC-Rollen (dynamisch extrahiert)
Phase 3: Monitoring    → Workbooks, Alert Rules
```

**Starten:** Actions → **Run Monitoring Deploy** → Run workflow

**Was passiert:**
1. **Foundation:** Erstellt die Basis-Infrastruktur (Workspace, Policies)
2. **Assignment:** Weist Policies zu und extrahiert RBAC-Rollen automatisch aus den Policy Definitions
3. **Monitoring:** Stellt Workbooks und Alert Rules bereit

---

## 5. Ergebnisse prüfen

Nach erfolgreichem Deployment:

1. **Azure Portal** → Ressourcengruppe öffnen
2. **Log Analytics Workspace** → Daten fließen nach wenigen Minuten
3. **Workbook** → Interaktive Analyse der VM-Daten
4. **Alerts** → Action Group erhält Benachrichtigungen bei Problemen

---

## Weiterführende Dokumentation

- [Architektur](../services/azure-managed-monitoring/docs/architectur.md)
- [Konfiguration](../services/azure-managed-monitoring/docs/configuration-guide.md)
- [Berechtigungen](../services/azure-managed-monitoring/docs/role-assignment-guide.md)
- [Quickstart (Standalone)](../services/azure-managed-monitoring/docs/QUICKSTART.md)

---

← Zurück zum [Setup Guide](setup-guide.md) | [Dokumentations-Index](index.md)
