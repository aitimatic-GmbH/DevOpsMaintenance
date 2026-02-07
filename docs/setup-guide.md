# Setup Guide: Erste Schritte

Dieses Dokument beschreibt, wie Sie das `DevOpsMaintenance` Repository einrichten und einen oder mehrere Services bereitstellen.

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

## 2. Voraussetzungen

### Lokal
- **Git** installiert
- **Azure CLI** installiert ([Installation](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli))
- **GitHub Account** mit Fork-Berechtigung

### Azure
- **Azure Subscription**
- **Azure Service Principal** (Berechtigungen je nach Service, siehe unten)

### GitHub
- **GitHub Actions** aktiviert in Ihrem Fork
- **Secrets** konfiguriert (service-spezifisch)

---

## 3. Gemeinsame GitHub Secrets

Diese Secrets werden von allen Services benötigt:

| Secret | Beschreibung |
|--------|-------------|
| `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID |
| `PAT_SUBMODULE_CHECKOUT` | Personal Access Token für Submodule-Zugriff (nur bei privaten Submodules) |

Weitere Secrets sind service-spezifisch und werden in den jeweiligen Anleitungen beschrieben.

**Einrichten:** Fork → **Settings → Secrets and variables → Actions → New repository secret**

---

## 4. Service auswählen und konfigurieren

Wählen Sie den Service, den Sie bereitstellen möchten. Jeder Service hat eigene Voraussetzungen, Secrets und Konfiguration.

> Sie können mehrere Services parallel betreiben. Die Secrets werden geteilt wo möglich.

### Verfügbare Services

| Service | Zweck | Anleitung |
|---------|-------|-----------|
| **Azure FinOps Manager** | Cloud-Kosten analysieren und optimieren | [setup-guide-finops.md](setup-guide-finops.md) |
| **Azure Managed Monitoring** | Automatisiertes Monitoring mit Azure Policy | [setup-guide-monitoring.md](setup-guide-monitoring.md) |

---

## 5. Submodule Validation

Unabhängig vom gewählten Service läuft der Workflow `validate-submodules.yml` automatisch bei Push und PR und prüft:
- Sind alle Submodules initialisiert?
- Sind Submodules auf dem richtigen Commit?
- Gibt es uncommitted Changes?

**Manuell starten:** Actions → **Validate Submodules** → Run workflow

---

## Troubleshooting

### Problem: "Repository not found" beim Klonen
**Lösung:** Submodules sind nicht public oder Sie haben keinen Zugriff.
```bash
# Prüfen Sie den Status
git submodule status
# Oder initialisieren Sie erneut
git submodule update --init --recursive
```

### Problem: Workflow schlägt fehl mit "Azure Login failed"
**Lösung:** Service Principal Credentials prüfen
```bash
az login --service-principal \
  -u <CLIENT_ID> \
  -p <SECRET> \
  --tenant <TENANT_ID>

az account show
```

### Problem: "Config file not found"
**Lösung:** Submodule wurde nicht geclont
```bash
git submodule update --init --recursive
```

---

## Support & Contributions

Dieses Repository akzeptiert keine externen Beiträge.

- ✅ Sie können das Repo forken und anpassen
- ✅ Issues für Fragen sind OK (keine Garantie auf Antwort)
- ❌ Pull Requests werden nicht akzeptiert

Siehe [CONTRIBUTING.md](../CONTRIBUTING.md) für Details.

---

**Lizenz:** MIT - siehe [LICENSE](../LICENSE)
**Security:** Siehe [SECURITY.md](../SECURITY.md)
