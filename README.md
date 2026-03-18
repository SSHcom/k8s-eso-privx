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
Now you can compile and install the ESO according to the ESO developer guide https://external-secrets.io/latest/contributing/devguide/

Installing the patches also installs documentation for the PrivX provider, in folder docs/provider/privx.md

## Go versions

If you go version is lower than required by latest ESO version, you can use an older version. Check available tags in the external-secrets repository and see what Go version they use

```bash
$ git tag
...
v1.3.2
v2.0.0
v2.0.1
v2.1.0
$ git show v2.1.0:go.mod | grep '^go '
go 1.25.7
$ git show v2.0.0:go.mod | grep '^go '
go 1.25.6
```
Then you can select a suitable version to apply the patches to
```bash
git switch --detach v2.1.0
```
Rocky Linux 9 and RHEL 9 for example are limited to Go 1.25.7