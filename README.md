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

## Get all pods with their probe's timeoutSeconds

```bash
kubectl get pods -A \
-o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,LIVENESS_PROBE_TIMEOUT:spec.containers[*].livenessProbe.timeoutSeconds,READINESS_PROBE_TIMEOUT:spec.containers[*].readinessProbe.timeoutSeconds' \
| awk '{if ($3 != "<none>" || $4 != "<none>") print $0 }'
```

## Delete CRDs with finalizers

```bash
kubectl get crd | grep "cattle.io" | cut -d ' ' -f1 | while read file
do
  kubectl patch crd $file -p '{"metadata":{"finalizers":null}}'
  kubectl delete crd $file
done
```

## Delete finalizers on specific namespace

```bash
export NAMESPACE='cattle-system'
kubectl patch namespace $NAMESPACE -p '{"metadata":{"finalizers":null}}'
export NAMESPACE='cert-manager'
kubectl patch namespace $NAMESPACE -p '{"metadata":{"finalizers":null}}'
```

## Delete finalizers on all namespaces

```bash
NAMESPACES=$(kubectl get namespaces --no-headers -o custom-columns=NAME:.metadata.name | awk '{print $1}' | tail -n +2)
NAMESPACES=( $( echo $NAMESPACES ) )

for ns in "${NAMESPACES[@]}"; do
    kubectl patch namespace $ns -p '{"metadata":{"finalizers":null}}'
done
```

## Report all requests and limits for all pods

```bash
k get pods -A -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,REQUESTS_CPU:.spec.containers[*].resources.requests.cpu,REQUESTS_MEMORY:.spec.containers[*].resources.requests.memory,LIMIT_CPU:.spec.containers[*].resources.limits.cpu,LiMIT_MEMORY:.spec.containers[*].resources.limits.memory'
```

## Redis Connect

```bash
REDIS_HOST=somehost
REDIS_PORT=someport
REDIS_NAMESPACE=somens
REDIS_SECRET=somesecret

redis-cli -h $REDIS_HOST \
-p $REDIS_PORT --tls --sni \
$REDIS_HOST -a \
$(kubectl -n $REDIS_NAMESPACE get secret $REDIS_SECRET -o jsonpath='{.data.redis-password}' | base64 -d)
```

## Ingresses with specific annotations

```bash
kubectl get ingress -A -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,EXTERNAL-DNS-TARGET-ANNOTATION:".metadata.annotations.external-dns\.alpha\.kubernetes\.io/target" | grep -v "<none>"
```

## Mass removal of annotations

```bash
k get middlewares -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name -A | sed 's/ \+/ /g' | while read file; do
namespace=$(echo $file | cut -d ' ' -f 1)
name=$(echo $file | cut -d ' ' -f 2)
kubectl annotate middleware $name -n $namespace 'kubectl.kubernetes.io/last-applied-configuration'-
done


k get ingressroute -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name -A | sed 's/ \+/ /g' | while read file; do
namespace=$(echo $file | cut -d ' ' -f 1)
name=$(echo $file | cut -d ' ' -f 2)
kubectl annotate ingressroute $name -n $namespace 'kubectl.kubernetes.io/last-applied-configuration'-
done
```

## Stale GroupVersion discovery: metrics.k8s.io/v1beta1

```bash
kubectl delete APIServices v1beta1.metrics.k8s.io
```
