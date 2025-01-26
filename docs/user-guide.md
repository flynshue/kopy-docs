# User Guide
The kopy operator runs a deployment on the kubernetes cluster.  It watches for ConfigMap or secrets objects that contain annotations with the sync key `flynshue.io/sync`.

## Sync ConfigMaps
Create a ConfigMap and add annotation with sync key `flynshue.io/sync`

```yaml title="example-configmap.yaml"
apiVersion: v1
data:
  URL: https://example.kopy.io
kind: ConfigMap
metadata:
  name: example-configmap
  annotations:
    flynshue.io/sync: "app=foobar"
```

Create a new namespace with labels that match value of the `flynshue.io/sync` key annotations
```yaml title="example-target-ns.yaml"
apiVersion: v1
kind: Namespace
metadata:
  name: example-target
  labels:
    app: foobar
```

Once the namespace has the labels, kopy will sync the ConfigMap over
```bash
$ kubectl -n example-target get cm
NAME                DATA   AGE
example-ConfigMap   1      68s
kube-root-ca.crt    1      96s
```

Here's what the copy ConfigMap will look like
```bash
$ kubectl -n example-target get cm example-configmap -o yaml
apiVersion: v1
data:
  URL: https://example.kopy.io
kind: ConfigMap
metadata:
  creationTimestamp: "2025-01-26T18:25:22Z"
  finalizers:
  - flynshue.io/finalizer
  labels:
    flynshue.io/origin.namespace: demo
  name: example-configmap
  namespace: example-target
  resourceVersion: "1088"
  uid: c8d16c64-1c20-47ae-9dc6-c3cdb2d3e701
```
