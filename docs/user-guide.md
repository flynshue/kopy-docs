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

Here's what it looks like in the kopy operator logs
```bash
$ kubectl -n kopy logs -f kopy-controller-manager-c5c5c88cb-zqtwg
2025-01-26T18:24:14Z	INFO	source object	{"controller": "configmap", "controllerGroup": "", "controllerKind": "ConfigMap", "ConfigMap": {"name":"example-configmap","namespace":"demo"}, "namespace": "demo", "name": "example-configmap", "reconcileID": "a476e927-8612-4882-8962-e61652314fcc"}
2025-01-26T18:25:22Z	INFO	need to add reconile	{"source.configMap": "example-configmap", "source.Namespace": "demo", "target.Namespace": "example-target"}
2025-01-26T18:25:22Z	INFO	source object	{"controller": "configmap", "controllerGroup": "", "controllerKind": "ConfigMap", "ConfigMap": {"name":"example-configmap","namespace":"demo"}, "namespace": "demo", "name": "example-configmap", "reconcileID": "ee618d24-60e3-44f2-a8b1-c4a2836aa2c5"}
2025-01-26T18:25:22Z	INFO	successfully synced	{"controller": "configmap", "controllerGroup": "", "controllerKind": "ConfigMap", "ConfigMap": {"name":"example-configmap","namespace":"demo"}, "namespace": "demo", "name": "example-configmap", "reconcileID": "ee618d24-60e3-44f2-a8b1-c4a2836aa2c5", "target.Namespace": "example-target"}
```