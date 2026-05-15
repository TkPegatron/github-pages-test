---
layout: post
title:  "Kubernetes Primer"
date:   2026-05-15 17:49:35 -0400
categories: kubernetes guide
---

# Kubernetes administrative operations primer

Kubernetes provides an API abstraction layer over the management of container runtimes and resources. A primary selling point for me, is that this allows us to configure the runtime of clustered hosts using structured text to provide values to the api. This structured text is written in a format called Yet-Another Markup Language or YAML.

In our environment the kubernetes cluser is running the Rancher K3s distribution of the kube-api server and related daemons. Installed within the cluster is a series of operators which interact with the kubernetes API to provide additional functionality that can be used to further tune, secure, and manage the lifecycle of applications running on the platform.

Cluster Operators:

- `FluxCD` (read "Flux Continuous Deployment"): Configured to read manifests from a Git repository and apply or reconcile them against the kubernetes API based on defined manifests which inform the flux operator of *how* resources should be deployed.
- `Longhorn`: a CSI driver that provides block storage abstraction for persisten volumes and claims. My CSI driver of choice, it is lightweight and easy to manage in liue of Tintri's CSI driver as unfortunately they have not yet added support for kubernetes genericly, only VMWare's kubernetes distribution.
- `Cilium`: a Container Network Interface (CNI) that provides custom resource abstractions over networking features, provides a Border Gateway Protocol (BGP) control plane and overlay network within the cluster.

[Cluster Configuration Repository](https://github.com/hch-napoleon/kubernetes-flux).

## Administrative Control

As should be mentioned in the tools section of this wiki, there are two main tools that I utilize for accessing and controling a kubernetes cluster.

- `kubectl`: The official kubernetes API client program, a CLI application.
- `OpenLens`: A community made graphical interface for the kubernetes API

Generally, I use kubectl from within a "devcontainer" on Visual Studio Code.

### Authentication

To access or control a cluster, these tools will need to authenticate using an authentication mechanism. Both read a `kubeconfig` file. Two authentication methods are permitted for our clusters; certificate, and oidc token.

Prioritize using the OIDC authentication flow which I will demonstrate here:

`kubeconfig`

```yaml
apiVersion: v1
kind: Config

current-context: internal
preferences: {}

clusters:
  - name: dmz
    cluster:
      certificate-authority-data: REDACTED
      server: https://REDACTED:6443
  - name: internal
    cluster:
      certificate-authority-data: REDACTED
      server: https://REDACTED:6443

contexts:
  - name: dmz
    context:
      cluster: dmz
      user: azure-broker
  - name: internal
    context:
      cluster: internal
      user: azure-broker

users:
  - name: azure-broker
    user:
      exec:
        apiVersion: client.authentication.k8s.io/v1beta1
        command: kubectl
        args:
          - oidc-login
          - get-token
          - --skip-open-browser
          - --oidc-issuer-url=https://login.microsoftonline.com/{{ entraid-tenant-uid }}/v2.0
          - --oidc-client-id={{ application-uid }}
```

When authenticating using that configuration the kubectl command will prompt you to

```none
Please visit the following URL in your browser: http://localhost:8000/
```

In visual studio code, this authentication flow forwards the TCP port used by the flow to the local machine over SSH\*, the chord `ctrl+lclick` on the link will open the browser and run you through the OIDC flow for our EndraID tenant.

\* As a convenience Visual Studio Code has a mechanism that automatically detects ports opening within development containers and forwards them to the localhost. In this context the unraveled meaning is that if a development container has a process running in the foreground which opens a listening port, that port is forwarded or redirected to the local machine, in this example the `kubectl` command plugin `oidc-login` opens a port on the devcontainers network namespace which vscode captures and maps the localhost's port 8000 to the ssh connection and patches it into the container over secure shell.

In the event that OIDC is not working as an authentication flow, it is acceptable to fall back to using the root account's credentials which can be downloaded from one of the nodes using ssh. The kubeconfig for the root user is located at `/etc/rancher/k3s/k3s.yaml`

### Commands frequently used

TODO: Explain common administrative tasks and functions

## Flux Continuous Deployment

TODO: Explain what flux is doing and how the repository is structured.