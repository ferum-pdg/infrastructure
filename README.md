# Infrastructure

In this repository, you will find the different configurations for our Kubernetes cluster on Infomaniak.


For the moment, the landing page is accessible through HTTP :
```
http://83.228.200.235
```

## Pipeline CI/CD

To have git as source for the cluster configuration, we use the `Arog CD` tool.

At each commit on the branch `main`, the tool will sync the modification whit what is currently running in the cluster.

To access to the Argo CD UI, you can run this command :

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

The UI can then be accessed using https://localhost:8080

