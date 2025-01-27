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

## Sync Secrets
Syncing Secrets is similar to the ConfigMap process.  You just need to add annotation with the sync key to the Secret.

Create a Secret and add annotation with sync key `flynshue.io/sync`

```yaml title="fake-secret.yaml"
apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  creationTimestamp: null
  name: fake-secret
  namespace: fake-secret-ns
  annotations:
    flynshue.io/sync: "app=fakesecret"
```

Create a new namespace with labels that match value of the `flynshue.io/sync` key annotations
```yaml title="another-namespace.yaml"
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: another-namespace
  labels:
    app: fakesecret
```

Once the namespace has the labels, kopy will sync the ConfigMap over
```bash
$ kubectl -n another-namespace get secrets
NAME          TYPE     DATA   AGE
fake-secret   Opaque   1      57s
```

Here's what the secret looks like.
```bash
$ kubectl -n another-namespace get secrets -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    key1: c3VwZXJzZWNyZXQ=
  kind: Secret
  metadata:
    creationTimestamp: "2025-01-27T15:38:20Z"
    finalizers:
    - flynshue.io/finalizer
    labels:
      flynshue.io/origin.namespace: fake-secret-ns
    name: fake-secret
    namespace: another-namespace
    resourceVersion: "1048"
    uid: 7d035a18-7f50-4713-8166-c8d4fe32833d
  type: Opaque
kind: List
metadata:
  resourceVersion: ""
```

## Tips

### Deleting a source ConfigMap/Secret
When you delete a ConfigMap/Secret object that contains an annotation with sync key `flynshue.io/sync`, the kopy operator will remove the finalizer from any copies of the object so that the copies of that object remain in their namespaces.

This allows applications that are using copies of the object to continue to operate without disruption in the event that the source object was deleted by accident.

### Deleting a copy object
When you delete a ConfigMap/Secret object that resides in a namespace that contains a label that matches a source ConfigMap/Secret object, the kopy operator will re-sync the object back into the namespace.  In order to delete a copy of that object, you'll have to remove the label from the namespace and then remove the finalizer from the object copy.

Remove label from namespace
```yaml title="another-namespace.yaml" hl_lines="7"
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: another-namespace
  labels:
    app: fakesecret # remove this
```

Remove finalizer
```yaml hl_lines="7-8"
apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  creationTimestamp: "2025-01-27T15:38:20Z"
  finalizers: # remove this
  - flynshue.io/finalizer # remove this
  labels:
    flynshue.io/origin.namespace: fake-secret-ns
  name: fake-secret
  namespace: another-namespace
  resourceVersion: "1048"
  uid: 7d035a18-7f50-4713-8166-c8d4fe32833d
type: Opaque
```
