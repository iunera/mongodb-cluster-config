# Readme

```
https://github.com/mongodb/mongodb-kubernetes-operator/blob/master/docs/install-upgrade.md#install-the-operator-using-Helm
helm repo add mongodb https://mongodb.github.io/helm-charts   
helm install community-operator-crds mongodb/community-operator-crds --namespace mongodb-kubernetes-operator
helm install community-operator mongodb/community-operator --namespace mongodb-kubernetes-operator --set community-operator-crds.enabled=false
```
