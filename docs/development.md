# Entwicklung

## Lokaler Build

Beide Images können lokal mit Docker (Buildx) gebaut werden.

**Default-Image (Terraform):**

```bash
docker build --platform linux/amd64 -t devcon:amd64 .
# oder für arm64 (z. B. Apple Silicon):
docker build --platform linux/arm64 -t devcon:arm64 .
```

**OpenTofu-Variante:**

```bash
docker build -f Dockerfile.opentofu --platform linux/amd64 -t devcon:opentofu-amd64 .
```

Ohne `--platform` baut Docker für die Architektur des Hosts. Für Multi-Arch-Builds (z. B. beide Plattformen in einem Schritt) kann Buildx mit `--platform linux/amd64,linux/arm64` genutzt werden; die Images werden dann typischerweise direkt in eine Registry gepusht.

## Versionen anpassen

Alle Tool-Versionen stehen als `ENV` am Anfang der Dockerfiles:

- [Dockerfile](../Dockerfile) – u. a. `GOLANG_VERSION`, `TERRAFORM_VERSION`, `VAULT_VERSION`, `KUBECTL_VER`, `HELM_VERSION`, `COSIGN_VERSION`, `INFRACOST_VERSION`
- [Dockerfile.opentofu](../Dockerfile.opentofu) – gleiche ENVs, zusätzlich `OPENTOFU_VERSION` statt Terraform

Nach Änderungen einen lokalen Build ausführen und optional die CI (Push/PR) laufen lassen, um beide Varianten und beide Architekturen zu prüfen.

## Renovate

Das Repo nutzt [Renovate](https://www.mend.io/renovate/) ([renovate.json](../renovate.json)) mit `config:recommended`. Renovate erstellt PRs für Dependency-Updates (z. B. GitHub-Actions-Pins). Die Dockerfile-ENVs werden von der Standard-Konfiguration nicht automatisch erfasst; Versions-Updates dort sind manuell oder über angepasste Renovate-Regeln möglich.

## Tests

Es gibt keine automatisierten Tests im Repo. Empfohlene manuelle Checks:

- Lokaler Build beider Dockerfiles für die genutzte Plattform
- Nach Push/PR: Prüfen der GitHub Actions (Build, Trivy, ggf. Signing) und des Security-Tabs (Trivy-SARIF)
