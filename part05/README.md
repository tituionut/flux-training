# A

## exercise 1
In this exercise we will try to create helm releases using Flux `HelmRepository` and `HelmRelease` CRDs.
We will not use Flux Git source reconciliation yet.

1. Define a `HelmRepository` for `bitnami`
   * For now, you do not need to push the created `HelmRepository` to Git
   * Let's use oldschool `https` and not modern `oci://`
   * https://charts.bitnami.com/bitnami
2. Define a `HelmRelease` for `nginx`
   * For now, you do not need to push the created `HelmRepository` to Git
   * https://artifacthub.io/packages/helm/jfrog/nginx
   * If you have `helm` installed
     * `helm search repo bitnami/nginx`
   * Use version `15.2.0` of the chart
     * Reason we are using such an old version, because there are changes in latest bitnami versions where it redirects to OCI registry instead
   * Use the following values: 
      ```
         replicaCount: 3
         service:
           type: ClusterIP
     ```
3. Apply both files using `kubectl apply -f` command
4. Get `helmrepositories` and `helmreleases` objects
5. If you have `helm` installed
   * Look at the status of the Helm release
6. Make sure the application is created
7. Change version of chart to `15.2.1` and apply again
8. Get `helmrepositories` and `helmreleases` objects
9. If you have `helm` installed
   * Look at the status of the Helm release
10. Continue to exercise 2


## exercise 2
1. Define a new `HelmRelease` while keeping the existing `HelmRepository` and `HelmRelease` 
2. Define a `HelmRelease` from `bitnami` for `apache` (Apache Web Server) for version `9.1.12`
   * Again the reason we are using the old version, is that in the newer versions, the chart is moved to OCI registry
   * Use the following values:
    ```
       replicaCount: 3
       service:
         type: ClusterIP
   ```
3. Apply and make sure the application is created


## exercise 3
Delete manually created `HelmRepository` and `HelmRelease` objects


# B

## exercise 1

1. Remove everything from `workshop-flux-bootstrap` except `gotk-*` files
2. Remove everything from `workshop-flux-myteam`
3. In your `workshop-flux-myteam` create
   * (See `demo01` for the inspiration)
   * `HelmRepository` and `HelmRelease` for `nginx` (as in the first exercise)
     * Remember, that namespaces have to be defined
4. Define `GitRepository` and `Kustomization` in `workshop-flux-bootstrap` that would point to `workshop-flux-myteam`
5. Inspect `gitrepositories`, `kustomizations`, `helmrepositories`, `helmreleases` objects
6. If you have `helm` installed
   * Look at the status of the Helm release
7. Make sure the application is created


## exercise 2
Even though `HelmRelease` can reference `HelmRepository` from another namespace, let's pretend we do not want to share any resources between namespaces.
Let's use Kustomize to create `HelmRepository` for bitnami for `dev` and `prod` namespaces.
And then we `HelmRelease` for `nginx` in `dev` and `HelmRelease` for `nginx` and `apache` in `prod`.
One could end up in having something similar to the following structure:

```
workshop-flux-myteam/
├── myteam/
│   ├── dev/
│   │   ├── helmrelease-nginx.yaml
│   │   └── kustomization
│   │       └── kustomization.yaml       # points to ../../../components/overlays/dev
│   ├── prod/
│   │   ├── helmrelease-apache.yaml
│   │   ├── helmrelease-nginx.yaml
│   │   └── kustomization
│   │       └── kustomization.yaml       # points to ../../../components/overlays/prod
└── components/
    ├── base/
    │   ├── namespace.yaml
    │   ├── helmrepository.yaml
    │   └── kustomization.yaml
    └── overlays/
        ├── dev/
        │   └── kustomization.yaml
        └── prod/
            └── kustomization.yaml
```

1. Remove everything from `workshop-flux-bootstrap` except `gotk-*` files
2. Make the structure above in the `workshop-flux-myteam`
3. Define `GitRepository` in `workshop-flux-bootstrap` that points to `workshop-flux-myteam`
4. Define two `Kustomization` objects in `workshop-flux-bootstrap`
   * One that points to `./myteam/dev` in `workshop-flux-myteam
   * Another that points to `./myteam/prod` in `workshop-flux-myteam
5. Inspect `gitrepositories`, `kustomizations`, `helmrepositories`, `helmreleases` objects
6. Make sure the applications are created


# C

# exercise 1
Let's see drift detection in Helm charts.

1. Remove everything from `workshop-flux-bootstrap` except `gotk-*` files
2. Remove everything from `workshop-flux-myteam`
3. Create a `HelmRepository` and `HelmRelease` from `demo02` in `workshop-flux-myteam` root
4. Create application using `app.yaml` from earlier exercises in `workshop-flux-myteam` root
5. Define `GitRepository` and `Kustomization` in `workshop-flux-bootstrap` that would point to `workshop-flux-myteam`
6. Make sure the applications are created
7. Now change the amount of replicas on `podinfo` and `my-deployment` Deployments
8. Wait for reconciliation
   * Either by waiting a little bit (based on the `spec.interval`)
   * Or annotating the `flux-system` Kustomization
   * `kubectl annotate --overwrite kustomization/myteam -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`
9. Do both deployments get updated?
10. Now add `.spec.driftDetection.mode` to `HelmRelease`
11. Wait for Git reconciliation
    * Not `Kustomization` reconciliation, but `GitRepository` reconciliation
12. State of the `Deployment` will be updated, but this is not because of `.spec.driftDetection.mode`, but because of we have changed the object
13. Change the amount of replicas on `podinfo` `Deployment`
14. Wait for reconciliation of a HelmRelease (see `spec.interval`)
    * Or force it
      * `kubectl annotate --overwrite helmrelease/<HELM_RELEASE> reconcile.fluxcd.io/requestedAt="$(date +%s)"`
15. Is it changed back?


# D

## exercise 1
Let's pretend we have a single application Git repository that contains both source code, helm chart and Flux objects to reconcile.

```
workshop-flux-myteam/
├── src/
│   └── ..
├── helm-chart
│   └── ..
└── flux
    ├── gitrepository.yaml
    ├── helmchart.yaml
    └── helmrelease.yaml
```

1. Remove everything from `workshop-flux-bootstrap` except `gotk-*` files
2. Remove everything from `workshop-flux-myteam`
3. Make the above structure in `workshop-flux-myteam`
   * `src` is empty, this just for illustration
   * `helm-chart` use the one from `demo01`
   * `flux` directory should contain
     * `HelmRelease`, `HelmChart` and `GitRepository`, see `demo02` for inspiration
4. Define `GitRepository` in `workshop-flux-bootstrap` that points to `workshop-flux-myteam`
5. Define `HelmRelease` in `workshop-flux-bootstrap` that points to `./flux` in the `workshop-flux-myteam` `GitRepository`
