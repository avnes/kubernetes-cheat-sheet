# kubernetes-cheat-sheet
Just a bunch of commands that makes my work with k8s easier. No common commands will be shown here. Only commands that were semi-complex and used to solve an issue or find some rare data.

## Find events in a namespace

```bash
NS='kube-system' # If it is kube-system you care about
kubectl -n $NS get events --sort-by='{.lastTimestamp}'
```

## Find number of unused ReplicaSets

```bash
kubectl get replicaset -A | awk ‘{if ($2 + $3 + $4 == 0) print $1}’ | wc -l
```

## Deployments with very high revision history limit

This one will print the namespace, the deployment name and the revisionHistoryLimit for deployments where revisionHistoryLimit is greater than 10.

```bash
kubectl get deployments -A -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,API_VERSION:.apiVersion,REV_HISTORY:.spec.revisionHistoryLimit | awk '{ if ($4 > 10) print }'
```

## Change revisionHistoryLimit for a deployment

```bash
kubectl patch deployment -n <namespace> <deployment> --type=json -p=’[{“op”: “replace”, “path”: “/spec/revisionHistoryLimit”, “value”: 10}]’
```

### Get all pods with their probe's timeoutSeconds

```bash
kubectl get pods -A \
-o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,LIVENESS_PROBE_TIMEOUT:spec.containers[*].livenessProbe.timeoutSeconds,READINESS_PROBE_TIMEOUT:spec.containers[*].readinessProbe.timeoutSeconds' \
| awk '{if ($3 != "<none>" || $4 != "<none>") print $0 }'
```

### Delete CRDs with finalizers

```bash
kubectl get crd | grep "cattle.io" | cut -d ' ' -f1 | while read file
do
  kubectl patch crd $file -p '{"metadata":{"finalizers":null}}'
  kubectl delete crd $file
done
```

### Delete finalizers on namespace
```bash
export NAMESPACE='cattle-system'
kubectl patch namespace $NAMESPACE -p '{"metadata":{"finalizers":null}}'
export NAMESPACE='cert-manager'
kubectl patch namespace $NAMESPACE -p '{"metadata":{"finalizers":null}}'
```

### Report all requests and limits for all pods
```bash
k get pods -A -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,REQUESTS_CPU:.spec.containers[*].resources.requests.cpu,REQUESTS_MEMORY:.spec.containers[*].resources.requests.memory,LIMIT_CPU:.spec.containers[*].resources.limits.cpu,LiMIT_MEMORY:.spec.containers[*].resources.limits.memory'
```

