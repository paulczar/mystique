# Helm Chart - Mystique

This chart is a kind of meta chart designed to allow you to fairly easily deploy arbitrary kubernetes manifests alongside other charts without having to run arbitrary `kubectl create` commands.

> The name Mystique is a nerdy reference to the shapeshifting mutant of the same name from the X-MEN universe.

## TL;DR;

```console
$ helm install .
```

## Introduction

This installs a set of arbitrary helm charts.

## Prerequisites

  - Kubernetes 1.6+

## Installing the Chart

To install the chart with the release name `my-release`:

```console
$ helm install --name my-release .
```

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

This chart takes a single list value of `manifests` each record of which contains a set of keys that helps describe the manifest.

The included as shown below shows that you can set `name`, `apiVersion`, `kind`, `annotations`, `labels`, and `spec`.  It also shows how `spec` (and some of the other fields) can include references to variables that can be set in a values file or with `--set`.

```yaml
manifests:
  - name: mystique
    apiVersion: v1
    kind: Service
    annotations:
      marvel.mutant.name: mystique
    labels:
      xwoman: mystique
    spec:
      externalName: "{{ .Values.demoUrl }}"

demoUrl: www.marvel.com
```

This becomes incredibly useful if you have a helm chart that deploys almost everything you need for an application except for maybe a resource or two. For example the above might be useful if you need to create an ExternalName resource for a database.

Another example of using this chart would be if you want to use the `external-dns` controller to configure DNS for your Kubernetes cluster. Say your cluster is on `192.168.33.33` and you want to call it `k8s.mycluster.com` you'd provide the following values to create an ExternalName service that `external-dns` will register for you.

```yaml
manifests:
  - name: k8s-api
    apiVersion: v1
    kind: Service
    annotations:
      external-dns.alpha.kubernetes.io/hostname: k8s.mycluster.com
    spec:
      externalName: 192.168.33.33
      type: ExternalName

```

Or you might need to create a `ClusterIssuer` for the `cert-manager` controller:

```yaml
manifests:
  - name: my-issuer
    apiVersion: certmanager.k8s.io/v1alpha1
    kind: ClusterIssuer
    spec:
      acme:
        email: "{{ .Values.email }}"
        http01: {}
        privateKeySecretRef:
          name: my-issuer
        server: https://acme-v02.api.letsencrypt.org/directory

email: my@email.address.com

```
