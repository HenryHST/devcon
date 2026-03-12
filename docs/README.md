# devcon – Dokumentation

**devcon** ist ein vollständiges Development-Environment als Container-Image. Statt Tools direkt auf dem Host zu installieren, läuft alles in einem Docker-Container (z. B. mit Docker for Mac).

## Übersicht

- **Base:** Ubuntu Noble (24.04)
- **Zwei Image-Varianten:**
  - **Default:** Enthält Terraform sowie die übrigen Tools (HashiCorp, Kubernetes, Cloud-CLIs, Zsh).
  - **OpenTofu:** Gleicher Stack, aber OpenTofu anstelle von Terraform; zusätzlich ein `terraform`-Symlink auf `tofu`.
- **Multi-Arch:** Beide Images werden für `linux/amd64` und `linux/arm64` gebaut und nach GHCR gepusht.

Ausführliche Beschreibung der Architektur und der enthaltenen Komponenten:

- [Architektur](architecture.md) – Aufbau des Images, Komponenten, Varianten, Multi-Arch

## Pipelines

Zwei GitHub-Actions-Workflows steuern Build, Scan, Signing und Releases:

- [Pipelines](pipelines.md) – Docker-Workflow (Build, Trivy, Cosign) und Release-Workflow (GitHub Release)

## Entwicklung

Lokaler Build, Anpassung von Versionen und Hinweise zur Wartung:

- [Entwicklung](development.md) – Lokaler Build, Versionen, Renovate
