# Setup instructions

1. Go to https://github.com/akuity/gitops-masterclass-applications-template and click the green "Use this template" button to create your own repo.
2. In your repo, click the green "Code" button and under the "Codespaces" tab click the green "Create codespace on main" button.
3. The generated Codespace (which is essentially VS Code in your browser) will take a while to spin up as it creates new containers. Let it complete. Don't panic. Maybe click to watch the logs.
4. When it comes up, the Codespace will already have Argo CD running. Go the to "Ports" tab (a sibling to the Terminal, Output, Problems, etc. tabs) and Argo CD should be availabe on the top row.
5. Click on the globe icon (you may have to hover over various parts of the row) to open the port in a new browser tab so you can view the Argo CD GUI.
6. From there, use `kubectl apply -f xyz.yaml` to make various changes that are reflected in the Argo CD GUI, but without actually using the GUI or the Argo CD CLI.

## Argo CD Access

The URL will be something like `fictional-goldfish-9wgpr5qvwwcp74j-30179.app.github.dev`

Defaults set for this instance:

U: `admin`
P: `password`

## syncOptions

- Don't store namespaces
  - use `CreateNamespace=true` for the `syncOptions`

- `ApplyOutOfSyncOnly=true`
  - This saves resources

- `RespectIgnoreDifferences`

https://argo-cd.readthedocs.io/en/stable/user-guide/application-specification/

```
metadata:
  annotations:
    argocd.argoproj.io/sync-options: confirm=true

```

Safe to do, avoids errors when a new CRD is added because of the chicken-egg problem.
```
metadata:
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResources=true

```

## App-of-Apps

https://www.youtube.com/watch?v=2pvGL0zqf9o

https://github.com/akuity/gitops-masterclass-applications-template


## Relevant commands

### Kustomize AppSet

```
kubectl apply -f kustomize/appset-gobg.yaml
```

```
kubectl delete -f kustomize/appset-gobg.yaml
```

Better than applying each:

```
kubectl apply -f kustomize/gobg-dev.yaml
```


### Helm AppSet

```
kubectl apply -f helm/appset.yaml
```

### Misc

```
k get pods -A
helm ls -A
argocd account get-user-info
tree kustomize/env
```

### Impersonation

Experimental feature. Note the following flag in the `post-create.sh` install of Argo CD:

```
--set configs.cm."application\.sync\.impersonation\.enabled"="true"
```

The `impersonation/project-endor.yaml` defines a `destinationServiceAccounts` entry for the `kuard` namespace.

The following was the set of actions required to make this work:

```sh
kubectl apply -f impersonation/project-endor.yaml   # create Project with impersonation rule
kubectl apply -f impersonation/app-kuard.yaml       # attempt to create a Application in kuard
                                                    # results in sync error due to missing service account
kubectl create sa kuard-deployer --namespace kuard  # sync error due to lack of permissions
kubectl apply -f impersonation/role.yaml            # create role
kubectl apply -f impersonation/rolebinding.yaml     # grant role to sa
                                                    # still failing due to missing permissions
kubectl apply -f impersonation/updated-role.yaml    # finally fixes it
```
