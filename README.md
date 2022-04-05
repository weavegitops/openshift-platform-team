# Flux multi-tenancy demo for OpenShift

Here's a Flux multi-tenancy demo for OpenShift. This work has been derived from the original Flux multi-tenancy example, found [here](https://github.com/fluxcd/flux2-multi-tenancy).  A nice thing about the demo in this repo is that we will use only **Web UI of OpenShift** to install Flux and bootstrap the demo. Yes - as a Cluster-Admin user, you can up and running a GitOps system by just clicking. You click to install Flux via OperatorHub, then you click to import one of the following snippets into your cluster, and your multi-tenant GitOps system will be ready to use in minutes.

Persona: **Alice** is an OpenShift Cluster-Admin. She'd like to set up multi-tenancy environments using Flux in an OpenShift-ish way via its Web UI by

  1. Installing **Flux** via OperatorHub.
  2. Bootstraping the multi-tenant setup with copy & paste the below snippets [here](https://github.com/openshift-fluxv2-poc/platform-team#production-cluster-source--kustomization) into OpenShift YAML import.
     ![image](https://user-images.githubusercontent.com/10666/137634319-1aac84e8-a139-4e3b-857b-76eeabcb4770.png)

And achieve a similar result as setting up with the CLI.

As you might see that this multi-tenancy setup uses *Gatekeeper* for policy enforcement, we also demonstrate the seamless integration between Flux's GitOps and OpenShift's Operator Framework by managing the [Gatekeeper Operator's subscription and operator group](https://github.com/openshift-fluxv2-poc/platform-team/blob/main/infra/policy-engine-operator/gatekeeper-operator.yaml) with Flux.  You can see labels in the subscription object of Gatekeeper indicating that Flux manages it, for example.
![image](https://user-images.githubusercontent.com/10666/137672236-d7b3acc1-2d6a-4249-9495-bdd7a684f279.png)


Persona: **Chanwit** is a member of the Dev team. Please change user name, or add other users as team members [here](https://github.com/openshift-fluxv2-poc/platform-team/blob/main/tenants/base/dev-team/rbac.yaml#L31) before proceeding with your fork.

After Alice set up the platform, Chanwit would find the `apps` namespace and workloads running inside it after logging in. These workloads are deployed from his development repository, which is located here: https://github.com/openshift-fluxv2-poc/dev-team. Chanwit can now use this repo as the GitOps repository for his team to configure further and deploy apps.

![image](https://user-images.githubusercontent.com/10666/137634584-270c72f8-b62a-4d58-a3e7-b9af4621a163.png)

## Production Cluster: Source & Kustomization

You can copy the below YAML snippet and import it directly into OpenShift to kick off the setup without using CLI.
If you fork, please change the repo URL at this [line](https://github.com/openshift-fluxv2-poc/platform-team/blob/main/README.md?plain=1#L39) and also this [line](https://github.com/openshift-fluxv2-poc/platform-team/blob/main/README.md?plain=1#L72) to match the forked one.

```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  namespace: flux-system
  name: openshift-platform-team
spec:
  timeout: 20s
  gitImplementation: libgit2
  interval: 1m
  # If you fork, please change this repo URL to match the forked one.
  url: 'https://github.com/weavegitops/openshift-platform-team' 
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  namespace: flux-system
  name: multi-tenant-production
spec:
  timeout: 2m
  path: ./clusters/production
  interval: 5m
  prune: true
  force: false
  sourceRef:
    name: openshift-platform-team
    kind: GitRepository
```

## Staging Cluster: Source & Kustomization
```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  namespace: flux-system
  name: openshift-platform-team
spec:
  timeout: 20s
  gitImplementation: libgit2
  interval: 1m
  # If you fork, please change this repo URL to match the forked one.
  url: 'https://github.com/weavegitops/openshift-platform-team' 
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  namespace: flux-system
  name: multi-tenant-staging
spec:
  timeout: 2m
  path: ./clusters/staging
  interval: 5m
  prune: true
  force: false
  sourceRef:
    name: openshift-platform-team
    kind: GitRepository
```
