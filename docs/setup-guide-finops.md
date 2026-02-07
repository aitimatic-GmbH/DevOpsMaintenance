# Setup Guide: Azure FinOps Manager

Diese Anleitung beschreibt die Einrichtung und Nutzung des FinOps-Service über das DevOpsMaintenance Repository.

> **Voraussetzung:** Folgen Sie zuerst dem [Setup Guide](setup-guide.md) (Schritt 1-3), bevor Sie hier fortfahren.

---

## 1. Azure Service Principal erstellen

Erstellen Sie zwei Service Principals: einen für **Read-Only** (Analysis) und einen für **Write** (Remediation).

### Analysis Service Principal (Read-Only)
```bash
az ad sp create-for-rbac \
  --name "sp-devops-finops-reader" \
  --role "Reader" \
  --scopes /subscriptions/IHRE-SUBSCRIPTION-ID
```

**Notieren Sie:**
- `appId` → `AZURE_CLIENT_ID`
- `tenant` → `AZURE_TENANT_ID`

### Remediation Service Principal (Write)
```bash
az ad sp create-for-rbac \
  --name "sp-devops-finops-contributor" \
  --role "Contributor" \
  --scopes /subscriptions/IHRE-SUBSCRIPTION-ID
```

**Notieren Sie:**
- `appId` → `AZURE_WRITE_CLIENT_ID`
- `tenant` → `AZURE_WRITE_TENANT_ID`

---

## 2. GitHub Secrets konfigurieren

**Settings → Secrets and variables → Actions → New repository secret**

### Secrets für Analysis

| Secret Name | Beschreibung | Wert |
|-------------|--------------|------|
| `AZURE_CLIENT_ID` | App ID des Read-Only Service Principal | `<appId>` |
| `AZURE_TENANT_ID` | Tenant ID | `<tenant>` |
| `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID | `<subscription-id>` |

### Secrets für Remediation (optional)

| Secret Name | Beschreibung | Wert |
|-------------|--------------|------|
| `AZURE_WRITE_CLIENT_ID` | App ID des Write Service Principal | `<appId>` |
| `AZURE_WRITE_TENANT_ID` | Tenant ID (kann gleich sein) | `<tenant>` |

**Hinweis:** `AZURE_SUBSCRIPTION_ID` wird für beide verwendet.

---

## 3. Config-Datei anpassen

Die FinOps-Konfiguration liegt im Submodule: `services/azure-finops-manager/finops.config.json`

### Config-Optionen

```json
{
  "schema_version": "1.0",
  "analysis_modules": {
    "old_snapshots": {
      "enabled": true,
      "retention_days": 90
    },
    "unattached_disks": {
      "enabled": true
    },
    "unassociated_public_ips": {
      "enabled": true
    },
    "azure_advisor": {
      "enabled": true,
      "categories": ["Cost", "HighAvailability"]
    },
    "underutilized_vms": {
      "enabled": true,
      "evaluation_period_days": 7,
      "max_cpu_percentage_threshold": 5
    }
  },
  "global_settings": {
    "excluded_resource_groups": [
      "rg-production-critical",
      "rg-do-not-touch"
    ]
  }
}
```

### Anpassen

1. **Module aktivieren/deaktivieren**: Setzen Sie `enabled` auf `true`/`false`
2. **Resource Groups ausschließen**: Fügen Sie RG-Namen zu `excluded_resource_groups` hinzu
3. **Thresholds anpassen**: z.B. `max_cpu_percentage_threshold` für VM-Analyse

**Wichtig:** Die Config-Datei wird direkt aus dem Submodule verwendet. Änderungen müssen dort committet werden.

---

## 4. Workflows ausführen

### 4.1 FinOps Analysis (Read-Only)

Der Analysis-Workflow läuft automatisch bei:
- Push auf Feature-Branches
- Pull Requests nach `main`

**Manuell starten:**
1. Gehen Sie zu **Actions** → **Run FinOps Analysis**
2. Klicken Sie auf **Run workflow**
3. Wählen Sie Branch (z.B. `main`)
4. Klicken Sie auf **Run workflow**

**Was passiert:**
- Checkt Repository + Submodules aus
- Authentifiziert mit Azure (Read-Only)
- Führt alle aktivierten Analyse-Module aus:
  - Unattached Disks
  - Unassociated Public IPs
  - Old Snapshots
  - Azure Advisor Recommendations
  - Underutilized VMs
- Generiert Markdown-Report
- Lädt Ergebnisse als Artifacts hoch

**Ergebnisse anschauen:**
1. Gehen Sie zur Workflow-Run-Seite
2. Scrollen Sie zu **Artifacts**
3. Laden Sie herunter:
   - `finops-analysis-results` (TSV-Dateien)
   - `finops-cost-report` (Markdown-Report)

### 4.2 FinOps Remediation (Write)

**⚠️ WARNUNG:** Remediation löscht/dealloziert Ressourcen!

#### DRY-RUN (empfohlen zuerst!)

1. Führen Sie zuerst eine **Analysis** durch (siehe 4.1)
2. Notieren Sie die **Run ID** (z.B. aus URL: `actions/runs/1234567890`)
3. Gehen Sie zu **Actions** → **Run FinOps Remediation**
4. Klicken Sie auf **Run workflow**
5. Füllen Sie aus:
   - **analysis_run_id**: `1234567890`
   - **resource_type_to_remediate**: `DRY-RUN`
   - **confirmation_phrase**: *leer lassen*
6. Klicken Sie auf **Run workflow**

#### Echte Remediation (⚠️ Vorsicht!)

**Nur ausführen, wenn Sie sicher sind!**

1. Führen Sie zuerst **DRY-RUN** durch
2. Prüfen Sie das Log genau
3. Gehen Sie zu **Actions** → **Run FinOps Remediation**
4. Füllen Sie aus:
   - **analysis_run_id**: `1234567890`
   - **resource_type_to_remediate**: Wählen Sie den Typ
   - **confirmation_phrase**: `DELETE` (genau so eingeben!)

**Ressourcentypen:**

| Typ | Aktion | Rückgängig? |
|-----|--------|-------------|
| `unattached-disks` | Löscht Disk | ❌ Nein |
| `unassociated-public-ips` | Löscht Public IP | ❌ Nein |
| `old-snapshots` | Löscht Snapshot | ❌ Nein |
| `underutilized-vms` | Dealloziert VM | ✅ Ja (VM bleibt, wird nur gestoppt) |

---

## Nächste Schritte

1. **Erste Analysis durchführen** (siehe 4.1)
2. **DRY-RUN testen** (siehe 4.2)
3. **Config anpassen** für Ihre Umgebung
4. **Workflow auf Schedule setzen** (z.B. wöchentlich)

---

← Zurück zum [Setup Guide](setup-guide.md) | [Dokumentations-Index](index.md)
