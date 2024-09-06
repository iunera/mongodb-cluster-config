# mongodb-cluster-config
This repo contains the configuration files for the MongoDB cluster setup based on kubernetes, [mongodb-kubernetes-operator](https://github.com/mongodb/mongodb-kubernetes-operator) and [fluxcd](https://fluxcd.io/flux/components/kustomize/kustomizations/) as gitops tool. 

Our approach is to deploy dedicated mongodb cluster into certain namespaces for specific projects rather than having one 'central' deployed Database cluster. This limits the amount of ops need to be done in the cluster and allows us to focus on kubernetes based gitops.

# Installation
## Operator
The Repo is included in fluxcd with following setup. It installs

* [mongodb-kubernetes-operator](./kubernetes/mongodb-kubernetes-operator) via helm
* the CRDs via manifests

```
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: mongodb-cluster-config
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  timeout: 60s
  url: ssh://git@github.com/iunera/mongodb-cluster-config
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: mongodb-cluster-config
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./kubernetes/
  prune: true
  sourceRef:
    kind: GitRepository
    name: mongodb-cluster-config
```

## Database in Namespace

### RBACs
Furthermore each database cluster will be deployed have to has access to some [permissions](kubernetes/fahrbar-common/rbac/mongodb-database-rbac.yaml) on the Kubernetes API. Therefor foreach namespace a serviceaccount and a role have to be created and binded. This is done by [kubernetes/fahrbar-common/rbac](./kubernetes/fahrbar-common/rbac/mongodb-database-rbac.yaml)

### Provide secrets
Our approach is to have a single point of truth/administration for passwords. Therefore we store all passwords via [SOPS](https://github.com/getsops/sops) on the private git repositories. We recommend to do so in general. Fluxcd is capable of decrypting SOPS encrypted Kubernetes Secrets on the fly. 
The the database mentioned below the following secrets would fit the semantics 

```
apiVersion: v1
kind: Secret
metadata:
  name: clusteradmin-password
  namespace: fahrbar-common
type: Opaque
stringData:
  password: securestring
---
apiVersion: v1
kind: Secret
metadata:
  name: occupancyapi-dev-v1-admin-password
  namespace: fahrbar-common
type: Opaque
stringData:
  password: securestring
---
apiVersion: v1
kind: Secret
metadata:
  name: occupancyapi-staging-v1-admin-password
  namespace: fahrbar-common
type: Opaque
stringData:
  password: securestring
---
apiVersion: v1
kind: Secret
metadata:
  name: occupancyapi-prod-v1-admin-password
  namespace: fahrbar-common
type: Opaque
stringData:
  password: securestring
```

### Deploy the database cluster

Finally we are ready to deploy the 3 node mongodb cluster with `SCRAM Auth`, `MongoDB RBAC` and persistency. The [kubernetes/fahrbar-common/database/mongodb-database.yaml](./kubernetes/fahrbar-common/database/mongodb-database.yaml) config shows the details.

For more info check the https://github.com/mongodb/mongodb-kubernetes-operator/tree/master/config/samples or [RTFM](https://github.com/mongodb/mongodb-kubernetes-operator/tree/master/docs).



# License
[Open Compensation Token License, Version 0.20](https://github.com/open-compensation-token-license/license/blob/main/LICENSE.md)
