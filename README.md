# PrivX External Secrets Operator (ESO) provider

This repository contains the code for PrivX provider for the Kubernetes ESO (External Secrets Operator).

## Deployment

This is to be included in the ESO repository. Before that, you must fork / vendor ESO and add your provider there.

You cannot load a Go module dynamically into a running External Secrets Operator (ESO). Your provider must be **compiled into the operator binary**. The standard approach is to fork (or vendor) ESO and add your provider to it.

The changes are as a patch set in the `/eso-patches` directory.

First download this repository and the external-secrets repository, then apply the patches.

```bash
    git clone git@github.com:SSHcom/k8s-eso-privx.git
    git clone git@github.com:external-secrets/external-secrets.git
    cd external-secrets
    git am ../k8s-eso-privx/eso-patches/*.patch
    # If conflicts, fix them and continue: git am --continue
    go mod tidy
```
If the patch does not succeed, you can checkout the external-secrets git commit last tested befor applying the patch
```bash
git checkout d9fd335a20378c47833b9350840b25963854a2be
```

Now you can compile and install the ESO according to the ESO developer guide https://external-secrets.io/latest/contributing/devguide/

Installing the patches also installs documentation for the PrivX provider, in folder docs/provider/privx.md

