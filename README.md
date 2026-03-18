# PrivX External Secrets Operator (ESO) Provider

This repository contains the PrivX provider for the [Kubernetes External Secrets Operator (ESO)](https://external-secrets.io/). It enables ESO to fetch secrets from [SSH.com PrivX](https://www.ssh.com/products/privileged-access-management-privx) and expose them as native Kubernetes `Secret` objects.

> **Note:** Because Go does not support dynamic module loading, this provider must be **compiled into the ESO operator binary**. It cannot be loaded at runtime. The standard approach is to patch the official ESO source tree and build a custom operator image.

---

## Provider capabilities

Once deployed, the PrivX provider supports the following ESO features:

| Feature | Description |
|---|---|
| `SecretStore` / `ClusterSecretStore` | Connect ESO to a PrivX Vault instance using OAuth credentials |
| `ExternalSecret` | Fetch a named secret from PrivX Vault into a Kubernetes `Secret` |
| `dataFrom.find` | Fetch multiple secrets by name using a regular expression |
| `PushSecret` | Write a Kubernetes `Secret` value back into PrivX Vault |

Authentication is handled via the PrivX OAuth2 client credentials flow. Four credential values are required: an OAuth client ID/secret pair and a PrivX API client ID/secret pair. See `docs/provider/privx.md` (installed by the patches) for full configuration examples.

> **Permissions:** The PrivX API user must have a role assigned in PrivX that is listed as a **reader** on each secret to be fetched, or a **writer** for PushSecret. Accessing a secret without the correct role assignment will succeed at the API level but return no data.

---

## Prerequisites

Before building, ensure the following are available on your build host:

| Requirement | Minimum version | Notes |
|---|---|---|
| Go | see [Go version compatibility](#go-version-compatibility) | Rocky Linux 9 / RHEL 9 ship Go 1.25.7 |
| Kubernetes | 1.30 | ESO uses APIs not available in earlier versions |
| Docker (or equivalent) | any recent | Required to build and push the operator image |
| OS | Rocky Linux 9 / RHEL 9 (tested) | Other RHEL-compatible distros should work |

> **Kubernetes compatibility:** ESO relies on APIs introduced in Kubernetes 1.30. If your cluster runs an older version (e.g. 1.28), you must either upgrade the cluster or manually patch the compatibility shims in the ESO source.

---

## Repository layout

```
k8s-eso-privx/
├── eso-patches/        # Git patch series to apply to the ESO source tree
└── ...                 # Provider source code (compiled into ESO, not standalone)
```

After applying the patches, PrivX provider usage documentation is installed at `docs/provider/privx.md` inside the ESO source tree.

---

## Build and installation

### 1. Clone both repositories

```bash
git clone git@github.com:SSHcom/k8s-eso-privx.git
git clone git@github.com:external-secrets/external-secrets.git
```

### 2. Select a compatible ESO version

You need to pick an ESO release whose Go version requirement matches what your system provides. See [Go version compatibility](#go-version-compatibility) for details on finding the right tag.

```bash
cd external-secrets
git switch --detach v2.1.0   # replace with your chosen tag
```

### 3. Apply the patch series

```bash
git am ../k8s-eso-privx/eso-patches/*.patch
```

If any patch fails due to conflicts:

```bash
# Fix conflicts manually, then:
git am --continue
```

### 4. Tidy Go modules

```bash
go mod tidy
```

### 5. Build the operator Docker image

Because this provider is not part of the upstream ESO image, you must build a custom Docker image and tag it to match the image reference defined in your ESO Helm chart or deployment manifests.

Check the image name used in your ESO deployment (e.g. in `deploy/charts/external-secrets/values.yaml`) and build accordingly:

```bash
# Replace the tag with whatever your deployment expects
docker build -t ghcr.io/your-org/external-secrets:latest .
docker push ghcr.io/your-org/external-secrets:latest
```

> **Important:** If the image name in your cluster does not match what you built and pushed, ESO pods will either fail to start or will run the upstream image without the PrivX provider — silently giving you no PrivX support.

### 6. Deploy to Kubernetes

Follow the [ESO developer guide](https://external-secrets.io/latest/contributing/devguide/) for Helm-based or manifest-based deployment, substituting your custom image reference.

---

## Go version compatibility

The Go version required by ESO varies by release. If your system Go is older than what the latest ESO requires, select an older ESO tag whose `go.mod` matches your available version.

List available tags and check their Go requirements:

```bash
cd external-secrets

# List all tags
git tag

# Check the Go version required by a specific tag
git show v2.1.0:go.mod | grep '^go '
# go 1.25.7

git show v2.0.0:go.mod | grep '^go '
# go 1.25.6
```

Once you have identified a suitable tag, check it out before applying the patches:

```bash
git switch --detach v2.1.0
```

### System Go version limits by OS

| OS | Maximum available Go version |
|---|---|
| Rocky Linux 9 | 1.25.7 |
| RHEL 9 | 1.25.7 |

If your system Go version is lower than required by the ESO tag you want to use, either:
- Select an older ESO tag with a lower Go requirement (preferred), or
- Install a newer Go toolchain manually outside the system package manager.

---

## Provider documentation

After applying the patches, full provider documentation is available at:

```
external-secrets/docs/provider/privx.md
```

This covers `SecretStore`, `ClusterSecretStore`, `ExternalSecret`, `dataFrom.find`, `PushSecret`, and authentication configuration with complete YAML examples.

---

## Known issues and workarounds

| Issue | Workaround |
|---|---|
| Kubernetes < 1.30 | Upgrade the cluster. Using 1.28 or earlier without source changes will not work. |
| System Go older than latest ESO requires | Select an older ESO tag whose `go.mod` matches your Go version — see [Go version compatibility](#go-version-compatibility). |
| Patch conflicts | Ensure you have checked out a specific ESO tag before applying patches, not a rolling `main` branch. |
| ESO pods use wrong image | Ensure your deployment manifest or Helm values reference the locally built image, not the upstream one. |
| Secret fetch returns no data | Check that the PrivX API user's role is listed as a reader on the target secret in PrivX. |
