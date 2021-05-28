# kubernetes-cheat-sheet
Just a bunch of commands that makes my work with k8s easier. No common commands will be shown here. Only commands that were semi-complex and used to solve an issue or find some rare data.

## Deployments with very high revision history limit

This one will print the namespace, the deployment name and the revisionHistoryLimit for deployments where revisionHistoryLimit is greater than 10.

```bash
kubectl get deployments -A -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,REV_HISTORY:.spec.revisionHistoryLimit | awk '{ if ($3 > 10) print }'
```
