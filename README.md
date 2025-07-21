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

### Argo CD Image Updater

The feature `Argo CD Image Updater` is used to monitor an image in a repository such as docker registery or ghcr.

Every 3 minutes, it will check if a new image has been uploaded.

To make it works, we must use a file named `kustomization.yaml` where we define the image used. Argo CD Image Updater relies on this file to know how and where to modify the image reference. The value declared in this file automatically overrides those specified in the `deployment.yaml` file.

Next, we configured Argo CD Image Updater by adding three annotations under the `Deployment` metadata in the `deployment.yaml` file.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ....
  annotations:
    argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/git-creds
    argocd-image-updater.argoproj.io/image-list: landing-page=ghcr.io/ferum-pdg/landing-page/landing-page
    argocd-image-updater.argoproj.io/landing-page.update-strategy: digest
```

**Write-back method :** Argo CD Image Updater uses a write-back method to propagate updated image version to Argo CD. In our setup, the selected method is `git`. When a new version of a container image is detected, the Image Updater does not directly modify the `deployment.yaml` or `kustomization.yaml`. Instead, it writes the update into a file named `.argocd-source-<appName>.yaml`. This file contains an `images:` section that holds the new image name, then the tool run `kustomize edit set image` command, which will dynamically overrides the image used in the `kustomization.yaml`. The change is then automatically committed and pushed to the Git repository. Argo CD detects this Git commit and automatically applies the change to the cluster by syncing the updated manifests. 

**Image-list :** Defines the image name to monitor.

**Update-strategy :** Defines how Argo CD Image Updater will find new verison of an image that is to be updated. By using the digest strategy, it will inspect a single tag in the registry for changes and when the currently used digest differs from what is found in the registry, it will update the `images:` section into the file `.argocd-source-<appName>.yaml`. Then, the image used will appear as `...:latest@sha<somelonghaststring>`.