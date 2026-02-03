# Quickstart: Erste Schritte

Dieses Dokument beschreibt, wie Sie das `DevOpsMaintenance` Repository und alle zugehörigen Service-Module korrekt auf Ihren lokalen Rechner klonen.

## Voraussetzungen

- Git muss auf Ihrem System installiert sein.

## Klonen des Repositories inklusive aller Services

Da dieses Projekt [Git Submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules ) verwendet, um die einzelnen Services zu verwalten, ist ein einfacher `git clone` nicht ausreichend.

Verwenden Sie den folgenden Befehl, um das Haupt-Repository zu klonen und gleichzeitig alle verknüpften Service-Module (wie `azure-finops-manager`) herunterzuladen:

```bash
git clone --recurse-submodules https://github.com/ihre-organisation/DevOpsMaintenance.git
