# A 

In this exercise we will continue working on the setup we have created before.
It is assumed that you have Flux installed and synchronized with `workshop-flux-bootstrap` Git directory.
If you have installed Flux in a different way, it is perfectly fine.
The most important is that you have a Git directory that Flux is synchronized with.


## exercise 1
Triggering synchronization

1. Look at the source controller logs
2. Update the application in `workshop-flux-bootstrap` Git repository
   * E.g. by changing the amount of replicas
3. Annotate your "root" Git repository to force synchronization
   * `kubectl annotate --overwrite gitrepository/flux-system -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`
4. Look at the source controller logs


## exercise 2
Flux will use `Kustomization` object (not Kustomize `Kustomization`, but Flux `Kustomization`) to make sure the state of the cluster is the same as the state in the Git repository.

1. If you have not done so already, push `app.yaml` to `workshop-flux-bootstrap`
2. Make sure the application is deployed
3. Now delete the application via `kubectl delete -f app.yaml`
4. Check the application gets recreated
   * Either by waiting a little bit (based on the `spec.interval`)
   * Or annotating the `flux-system` Kustomization
     * `kubectl annotate --overwrite kustomization/flux-system -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`
5. Inspect at the kustomize controller logs


## exercise 3
Plain manifests.

1. If you have not done so already, push `app.yaml` to `workshop-flux-bootstrap`
2. Make sure the application is deployed
3. Push `app2.yaml` to `workshop-flux-bootstrap`
   * Just copy `app.yaml` and adjust the names
4. Make sure the second application is deployed
   * Wait for synchronization with git or force it
5. Change a property in the second application, e.g. amount of replicas
   * Wait for synchronization with git or force it
6. Verify the changes are updated
7. Delete both applications from `workshop-flux-bootstrap`
   * Wait for synchronization with git or force it
8. Make sure the applications are deleted from the cluster


# B

## exercise 1

1. If you have not done so already, push `app.yaml` to `workshop-flux-bootstrap`
2. In a separate console `get` kustomizations with `--watch` option
3. Change `app.yaml` and push to repository (e.g. change amount of replicas)
4. Wait for the changes to be applied
   * Or annotate the `GitRepository`
     * `kubectl annotate --overwrite gitrepository/flux-system -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`
5. Can you see that the git revision got updated in the kustomization?
6. What does kustomization-controller have in the logs?
7. Change the application (e.g. amount of replicas on the `Deployment` using `kubectl edit`)
8. Check the application gets restored to the "state" in git
   * Either by waiting a little bit (based on the `spec.interval`)
   * Or annotating the `flux-system` Kustomization
     * `kubectl annotate --overwrite kustomization/flux-system -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`

## exercise 3

1. Make an error in `app.yaml`
   * E.g. make the file invalid yaml
2. Push the changes
3. Wait for reconciliation (or force it)
4. Can we see the error in `get` and `describe` commands in Kustomization?
5. What does kustomization-controller have in the logs?
6. Change `app.yaml` back to valid yaml
7. Push the changes
8. Wait for reconciliation (or force it)
9. Do the errors go away?
10. Make a "subtle" error in `app.yaml`
    * E.g. change the image tag to a non-existing one
11. Push the changes
12. Wait for reconciliation (or force it)
13. Is there an error in `get` and `describe` commands in Kustomization? Why?
