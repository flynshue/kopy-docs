# kopy

kopy is a kubernetes operator that can sync ConfigMap or secret objects to other namespaces within the cluster.

## Getting Started
The kopy operator runs a deployment on the kubernetes cluster.  It watches for ConfigMap or secrets objects that contain annotations with the sync key `kopy.kot-labs.com/sync`.

```yaml title="ConfigMap/secret object annotations"
metadata:
    annotations:
      kopy.kot-labs.com/sync: app=test-config-00
```

It will take the annotation values and look for any namespaces that are labeled with it and sync the objects into that namespace.

```yaml title="namespace with sync label"
apiVersion: v1
kind: Namespace
metadata:
  name: test-target-config-ns-00
  labels:
      app: test-config-00
```

## Installing with Helm
Add the helm chart repository
```bash
helm repo add kopy https://kopy.kot-labs.com

helm repo update kopy
```

Install helm chart with defaults into the `kopy` namespace
```bash
helm install kopy kopy/kopy \
--create-namespace \
--namespace=kopy
```

Install helm chart into `kopy` namespace with overriding values
```bash
helm upgrade kopy kopy/kopy \
--create-namespace \
--namespace=kopy \
--set controllerManager.container.resources.limits.memory=256Mi
```