# Quickstart: Erste Schritte

Dieses Dokument beschreibt, wie Sie das `DevOpsMaintenance` Repository nutzen können, um FinOps-Analysen für Ihre Azure-Umgebung durchzuführen.

## Voraussetzungen

### Lokal
- **Git** installiert
- **Azure CLI** installiert ([Installation](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli))
- **GitHub Account** mit Fork-Berechtigung

### Azure
- **Azure Subscription** mit Leserechten
- **Azure Service Principal** mit folgenden Berechtigungen:
  - **Für Analysis (Read-Only):**
    - `Reader` Rolle auf Subscription-Ebene
    - Zugriff auf Azure Advisor Recommendations
  - **Für Remediation (Write):**
    - `Contributor` Rolle auf Subscription-Ebene
    - Berechtigung zum Löschen von Ressourcen

### GitHub
- **GitHub Actions** aktiviert in Ihrem Fork
- **Secrets** konfiguriert (siehe unten)

---

## 1. Repository forken und klonen

Da dieses Projekt [Git Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) verwendet, ist ein einfacher `git clone` nicht ausreichend.

### Fork erstellen
1. Gehen Sie zu: https://github.com/aitimatic-GmbH/DevOpsMaintenance
2. Klicken Sie auf **Fork**
3. Wählen Sie Ihren Account

### Mit Submodules klonen
```bash
git clone --recurse-submodules https://github.com/IHR-USERNAME/DevOpsMaintenance.git
cd DevOpsMaintenance
```

**Falls Sie vergessen haben `--recurse-submodules` zu verwenden:**
```bash
git submodule update --init --recursive
```

---

## 2. Azure Service Principal erstellen

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
- Subscription ID → `AZURE_SUBSCRIPTION_ID`

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

## 3. GitHub Secrets konfigurieren

Gehen Sie zu Ihrem Fork: **Settings → Secrets and variables → Actions → New repository secret**

### Required Secrets für Analysis

| Secret Name | Beschreibung | Wert |
|-------------|--------------|------|
| `AZURE_CLIENT_ID` | App ID des Read-Only Service Principal | `<appId>` |
| `AZURE_TENANT_ID` | Tenant ID | `<tenant>` |
| `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID | `<subscription-id>` |

### Optional: Secrets für Remediation

| Secret Name | Beschreibung | Wert |
|-------------|--------------|------|
| `AZURE_WRITE_CLIENT_ID` | App ID des Write Service Principal | `<appId>` |
| `AZURE_WRITE_TENANT_ID` | Tenant ID (kann gleich sein) | `<tenant>` |

**Hinweis:** `AZURE_SUBSCRIPTION_ID` wird für beide verwendet.

---

## 4. Config-Datei anpassen

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

## 5. Workflows ausführen

### 5.1 FinOps Analysis (Read-Only)

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

### 5.2 FinOps Remediation (Write)

**⚠️ WARNUNG:** Remediation löscht/dealloziert Ressourcen!

#### DRY-RUN (empfohlen zuerst!)

1. Führen Sie zuerst eine **Analysis** durch (siehe 5.1)
2. Notieren Sie die **Run ID** (z.B. aus URL: `actions/runs/1234567890`)
3. Gehen Sie zu **Actions** → **Run FinOps Remediation**
4. Klicken Sie auf **Run workflow**
5. Füllen Sie aus:
   - **analysis_run_id**: `1234567890`
   - **resource_type_to_remediate**: `DRY-RUN`
   - **confirmation_phrase**: *leer lassen*
6. Klicken Sie auf **Run workflow**

**Was passiert:**
- Lädt Analysis-Ergebnisse herunter
- Simuliert alle Aktionen (keine echten Änderungen!)
- Zeigt im Log, was gelöscht/dealloziert würde
- Erstellt Remediation-Log

#### Echte Remediation (⚠️ Vorsicht!)

**Nur ausführen, wenn Sie sicher sind!**

1. Führen Sie zuerst **DRY-RUN** durch
2. Prüfen Sie das Log genau
3. Gehen Sie zu **Actions** → **Run FinOps Remediation**
4. Klicken Sie auf **Run workflow**
5. Füllen Sie aus:
   - **analysis_run_id**: `1234567890`
   - **resource_type_to_remediate**: Wählen Sie:
     - `unattached-disks` - Löscht ungenutzte Disks
     - `unassociated-public-ips` - Löscht ungenutzte Public IPs
     - `old-snapshots` - Löscht alte Snapshots
     - `underutilized-vms` - Dealloziert unterlastete VMs
     - `all` - Führt alle Aktionen durch
   - **confirmation_phrase**: `DELETE` (genau so eingeben!)
6. Klicken Sie auf **Run workflow**

**Ressourcentypen:**

| Typ | Aktion | Rückgängig? |
|-----|--------|-------------|
| `unattached-disks` | Löscht Disk | ❌ Nein |
| `unassociated-public-ips` | Löscht Public IP | ❌ Nein |
| `old-snapshots` | Löscht Snapshot | ❌ Nein |
| `underutilized-vms` | Dealloziert VM | ✅ Ja (VM bleibt, wird nur gestoppt) |

---

## 6. Submodule Validation

Der Workflow `validate-submodules.yml` läuft automatisch und prüft:
- Sind alle Submodules initialisiert?
- Sind Submodules auf dem richtigen Commit?
- Gibt es uncommitted Changes?

**Manuell starten:**
1. Gehen Sie zu **Actions** → **Validate Submodules**
2. Klicken Sie auf **Run workflow**

---

## Troubleshooting

### Problem: "Repository not found" beim Klonen
**Lösung:** Submodules sind nicht public oder Sie haben keinen Zugriff.
- Stellen Sie sicher, dass alle Submodules public sind
- Oder entfernen Sie private Submodules

### Problem: Workflow schlägt fehl mit "Azure Login failed"
**Lösung:** Service Principal Credentials prüfen
```bash
# Test lokal
az login --service-principal \
  -u <AZURE_CLIENT_ID> \
  -p <SECRET> \
  --tenant <AZURE_TENANT_ID>

az account show
```

### Problem: "Config file not found"
**Lösung:** Submodule wurde nicht geclont
```bash
git submodule update --init --recursive
```

### Problem: Analysis findet keine Ressourcen
**Lösung:** Service Principal hat keine Leserechte
```bash
# Prüfe Rolle
az role assignment list \
  --assignee <AZURE_CLIENT_ID> \
  --scope /subscriptions/<SUBSCRIPTION_ID>
```

---

## Nächste Schritte

1. **Erste Analysis durchführen** (siehe 5.1)
2. **DRY-RUN testen** (siehe 5.2)
3. **Config anpassen** für Ihre Umgebung
4. **Workflow auf Schedule setzen** (z.B. wöchentlich)

---

## Support & Contributions

**⚠️ Hinweis:** Dieses Repository akzeptiert keine externen Beiträge.

- ✅ Sie können das Repo forken und anpassen
- ✅ Issues für Fragen sind OK (keine Garantie auf Antwort)
- ❌ Pull Requests werden nicht akzeptiert

Siehe [CONTRIBUTING.md](../CONTRIBUTING.md) für Details.

---

**Lizenz:** MIT - siehe [LICENSE](../LICENSE)
**Security:** Siehe [SECURITY.md](../SECURITY.md)
