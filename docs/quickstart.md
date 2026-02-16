# Quickstart: Managed Services für Azure

Dieses Repository dient als zentraler Hub zur Verwaltung und Bereitstellung verschiedener Managed Services über Git-Submodule.

Diese Anleitung beschreibt die grundlegenden Schritte zur Vorbereitung des Repositories. Detaillierte Anleitungen zur Konfiguration und zum Deployment finden Sie in der `README.md` des jeweiligen Service-Submodules.

---

## Schritt 1: Repository-Vorbereitung (Einmalig)

Bevor Sie einen spezifischen Service bereitstellen können, müssen die grundlegenden Voraussetzungen erfüllt sein.

1.  **Repository klonen:** Klonen Sie dieses Haupt-Repository auf Ihren lokalen Rechner.
    ```bash
    git clone https://github.com/aitimatic-GmbH/DevOpsMaintenance.git
    cd DevOpsMaintenance
    ```

2.  **Submodule initialisieren:** Initialisieren und laden Sie alle verfügbaren Service-Submodule herunter.
    ```bash
    git submodule update --init --recursive
    ```

3.  **GitHub Secrets konfigurieren:** Für die Ausführung der Workflows in GitHub Actions werden folgende Secrets benötigt:

    **Minimum (für Analysis):**
    - `AZURE_CLIENT_ID` - Azure Service Principal App ID (Read-Only)
    - `AZURE_TENANT_ID` - Azure Tenant ID
    - `AZURE_SUBSCRIPTION_ID` - Azure Subscription ID
    - `PAT_SUBMODULE_CHECKOUT` - Personal Access Token für Submodule-Zugriff (nur bei privaten Submodules)

    **Optional (für Remediation / Monitoring Deployment):**
    - `AZURE_WRITE_CLIENT_ID` - Azure Service Principal App ID (Write)
    - `AZURE_WRITE_TENANT_ID` - Azure Tenant ID (Write)

    **Für Managed Monitoring:**
    - `MON_CONFIG_JSON` - JSON-Override für sensible Monitoring-Konfigurationswerte

    Die Erstellung und Konfiguration der Secrets wird in der jeweiligen Service-Anleitung beschrieben.

Nachdem diese Schritte erfolgreich waren, ist Ihr Repository einsatzbereit.

---

## Schritt 2: Einen spezifischen Service bereitstellen

Wählen Sie den Service aus, den Sie bereitstellen möchten, und folgen Sie der detaillierten Anleitung im jeweiligen Submodule-Verzeichnis.

### Verfügbare Services:

*   #### **Azure FinOps Manager**
    *   **Zweck:** Analysiert Cloud-Kosten und identifiziert Einsparpotenziale in Azure.
    *   **Anleitung:** [**`services/azure-finops-manager/README.md`**](../services/azure-finops-manager/README.md)
    *   **Setup-Guide:** [**`setup-guide-finops.md`**](setup-guide-finops.md)

*   #### **Azure Managed Monitoring Essentials**
    *   **Zweck:** Automatisierte Monitoring-Infrastruktur mit Azure Policy, Log Analytics und Alerting.
    *   **Anleitung:** [**`services/azure-managed-monitoring/README.md`**](../services/azure-managed-monitoring/README.md)
    *   **Setup-Guide:** [**`setup-guide-monitoring.md`**](setup-guide-monitoring.md)

---

Ausführlicher Setup Guide: [setup-guide.md](setup-guide.md) | Alle Dokumente: [index.md](index.md)

