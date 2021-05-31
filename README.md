# kubernetes-cheat-sheet
Just a bunch of commands that makes my work with k8s easier. No common commands will be shown here. Only commands that were semi-complex and used to solve an issue or find some rare data.

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
