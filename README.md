## DEVCON: a Complete Dev Environment in a Container

As a rule of thumb, I never install any packages directly onto my Mac unless I absolutely have to. So I created this sample development container that I use with Docker for Mac to be my sole dev environment. There is a breakdown in the Dockerfile of which tools are installed.

### Dokumentation

Ausführliche Beschreibung von Architektur, Pipelines und Entwicklung: **[docs/](docs/README.md)**  
– [Architektur](docs/architecture.md) (Aufbau, Komponenten, Varianten, Multi-Arch)  
– [Pipelines](docs/pipelines.md) (Docker-Workflow, Release-Workflow)  
– [Entwicklung](docs/development.md) (lokaler Build, Versionen, Renovate)

### Build pipeline (CI/CD)

The GitHub Actions workflow in [.github/workflows/docker-publish.yml](.github/workflows/docker-publish.yml) runs on push to `main`, on pull requests targeting `main`, and on version tags `v*.*.*`. It does the following:

1. **Build** – Builds two multi-arch images (linux/amd64, linux/arm64):
   - Default image (Terraform) from [Dockerfile](Dockerfile)
   - OpenTofu variant from [Dockerfile.opentofu](Dockerfile.opentofu)
2. **Push** – Pushes images to GHCR (`ghcr.io`) only when not a pull request. Tags include `latest`, `latest-opentofu`, and (on tags) semver.
3. **Trivy** – Runs [Trivy](https://github.com/aquasecurity/trivy) against both images for vulnerability scanning. The job fails on **CRITICAL** or **HIGH** issues (unfixed vulnerabilities are ignored). SARIF results are uploaded to GitHub Code Scanning so findings appear under the repository’s **Security** tab.
4. **Signing** – After push, images are signed with [Cosign](https://docs.sigstore.dev/cosign/overview/) (Sigstore). Signatures use OIDC and the public Rekor transparency log (for public repos).

Ausführliche Pipeline-Beschreibung und Ablaufdiagramm: [docs/pipelines.md](docs/pipelines.md).

To build and sign images yourself, use the same Dockerfiles and optionally verify with:

```bash
cosign verify ghcr.io/OWNER/devcon:latest
```

### Release pipeline

When you push a version tag (e.g. `v1.0.0`), two things happen:

1. **Docker workflow** ([docker-publish.yml](.github/workflows/docker-publish.yml)) builds and pushes images tagged with that version (and signs them).
2. **Release workflow** ([release.yml](.github/workflows/release.yml)) creates a [GitHub Release](https://docs.github.com/en/repositories/releasing-projects-on-github) for the tag with:
   - Auto-generated release notes from commits since the previous tag
   - A short description of the container images and their tags
   - Pull/run and Cosign verify commands for that version

To publish a new release:

```bash
git tag v1.0.0
git push origin v1.0.0
```

### Package Versions

See the [Dockerfile](Dockerfile) for current tool versions. Example (as of last update):

- Go, Terraform, Vault, Consul, Packer, Boundary, Waypoint, hcdiag
- kubectl, Helm, Krew, eksctl
- AWS CLI v2, gcloud, Azure CLI
- Cosign, Snyk, Infracost
- Docker CLI & Compose, PostgreSQL, Redis, Node.js, etc.
### Usage

**Default image (Terraform):**
```
$ docker run -it --rm --hostname devcon -v /var/run/docker.sock:/var/run/docker.sock HenryHST/devcon:latest
```

**OpenTofu variant** (same image with OpenTofu instead of Terraform; `tofu` and `terraform` symlink available):
```
$ docker run -it --rm --hostname devcon -v /var/run/docker.sock:/var/run/docker.sock HenryHST/devcon:latest-opentofu
```


Optionally, you can mount your local Mac dev directory inside the container by adding `-v /path/to/dir:/root`. Typically, I mount a specific dev directory from my Mac that contains my dot files (including  git, ssh config + keys) to make it easier to use across both Mac and within the dev container. This way I can make sure that the dev container is a throw-away, leaving no keys/secrets exposed or written in it. 

Feel free to use and adjust to fit your own dev tooling!

Cheers 
 
