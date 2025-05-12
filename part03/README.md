# A

## exercise 1
In this exercise we will deploy application that is not managed by us.

1. Analyze https://github.com/stefanprodan/podinfo/tree/master/kustomize
2. In your `workshop-flux-bootstrap` Git repository define a new `GitRepository`
   * That would point to `https://github.com/stefanprodan/podinfo`
   * Branch `master`
3. Wait for reconciliation or force it
   * `kubectl annotate --overwrite gitrepository/flux-system -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`
4. Make sure `GitRepository` is created
5. In your `workshop-flux-bootstrap` Git repository define a new `Kustomization`
   * That would point to the `./kustomize` folder in the new `GitRepository`
   * Also add `spec.targetNamespace: default`
6. Wait for reconciliation or force it
7. Make sure `Kustomization` is created
8. Verify the application is created
9. Use not a `master` branch, but a specific tag
   * Check documentation at https://fluxcd.io/flux/components/source/gitrepositories/#reference
   * Try using a tag or another branch instead in the `GitRepository`
10. Delete the created `Kustomization` and `GitRepository`


## exercise 2
In this exercise we will create a new repository which you will make use of in this course.

* Create a new Git repository (public) on your Git account
  * Name it something like `workshop-flux-myteam`
  * Create a `master` branch
  * We can also work with private repositories, but it requires your token or ssh key to be saved in the cluster


## exercise 3
In this exercise we will add new synchronization to our new `workshop-flux-myteam` Git repository.

1. First, remove `app.yaml` from our initial `workshop-flux-bootstrap`.
   1. So that we have only `gotk-*` files left.
   2. Make sure the application is removed from the cluster
2. Push `app.yaml` to the `workshop-flux-myteam` instead
3. Now in the `workshop-flux-bootstrap` define a `GitRepository` and `Kustomization` that would point to `workshop-flux-myteam`
   * Use meaningful names, e.g. `myteam-gitrepository.yaml` and `myteam-kustomization.yaml`
   * Use `default` namespace
   * Use a meaningful resource names, e.g. `myteam`
4. Wait for reconciliation or force it
5. Inspect `gitrepositories` and `kustomizations`
6. Verify the application is created
7. Now, delete the newly created `GitRepository` and `Kustomization` from `workshop-flux-bootstrap`
8. Wait for reconciliation or force it
9. Is application, new `GitRepository` and `Kustomization` removed?


## exercise 4
In this exercise we will try to create multiple namespaces and deploy applications in each of them.

First, we need a namespace. We have two options:
1. We can create in `workshop-flux-bootstrap`
   * In this way namespace creation is "centralized"
2. We can create in `workshop-flux-myteam`
   * It is possible to restrict new resource creation (e.g. restrict new namespace creation)
     * Impersonating service accounts https://fluxcd.io/flux/components/kustomize/kustomizations/#service-account-reference
     * [Kyverno](https://kyverno.io/) policies / admission controllers
     * etc

In this exercise we will create a new namespace in `workshop-flux-myteam`.
So the structure of `workshop-flux-myteam` should look like:

```
workshop-flux-myteam/
├── team1/
│   ├── namespace.yaml    # team1 namespace
│   └── app.yaml          # remember namespace
└── team2/
    ├── namespace.yaml    # team2 namespace
    └── app.yaml          # remember namespace
```

1. You should have only `gotk-*` files in `workshop-flux-bootstrap`
2. Create a new structure in `workshop-flux-myteam`

### exercise 4a
1. Create
   1. One `GitRepository` in `workshop-flux-bootstrap` that would point to `workshop-flux-myteam`
   2. One `Kustomization` in `workshop-flux-bootstrap` that would point to the root
2. Wait for reconciliation or force it
3. Inspect `gitrepositories` and `kustomizations`
4. Verify the namespaces and applications are created
5. Now make an error in `team1/app.yaml` in `workshop-flux-myteam`
6. Wait for reconciliation or force it
   * Note, we are reconciling not the "original" `GitRepository`, but the new one
7. Inspect `gitrepositories` and `kustomizations`
8. Without fixing an error make a valid change to `team2/app.yaml` (e.g. amount of replicas)
9. Wait for reconciliation or force it
10. Is the change to `team2/app.yaml` applied?
11. Fix `team1/app.yaml` error and push
12. Wait for reconciliation or force it
13. Is the change to `team2/app.yaml` applied now?
14. Restore `team1/app.yaml`  and `team2/app.yaml` to the original values
15. From `workshop-flux-bootstrap` delete new `Kustomization` using `workshop-flux-myteam` `GitRepository`
    * But keep the `GitRepository` pointing to `workshop-flux-myteam`
16. Wait for reconciliation or force it
    * Note, here we are reconciling the `workshop-flux-bootstrap`

### exercise 4b
This is continuation of exercise 4a.

1. Instead of deleted `Kustomization` that was using `workshop-flux-myteam` `GitRepository`, create wo:
   * One `Kustomization` that points to `workshop-flux-myteam` subfolder `./team1` (`Kustomization.spec.path`)
   * Another `Kustomization` that points `workshop-flux-myteam` subfolder `./team2` (`Kustomization.spec.path`)
2. Wait for reconciliation or force it
3. Inspect `gitrepositories` and `kustomizations`
4. Verify the namespaces and applications are created
5. Now make an error in `team1/app.yaml` in `workshop-flux-myteam`
6. Wait for reconciliation or force it
   * Note, we are reconciling not the "original" `GitRepository`, but the one that points to the `workshop-flux-myteam`
7. Inspect `gitrepositories` and `kustomizations`
8. Without fixing an error make a valid change to `team2/app.yaml` (e.g. amount of replicas)
9. Wait for reconciliation or force it
10. Is the change to `team2/app.yaml` applied?


## exercise 5
One git repository, multiple branches.

1. Delete everything from `workshop-flux-bootstrap` except `gotk-*` files
2. Make sure the applications are removed
3. Create two branches in `workshop-flux-myteam`
   * `team1`
   * `team2`
4. In branch `team1` create `Namespace` `team1` and application `app.yaml` belonging to that namespace
5. In branch `team2` create `Namespace` `team2` and application `app.yaml` belonging to that namespace
6. Push both branches
7. In your `workshop-flux-bootstrap` create
   * Two `GitRepository` objects
      * One for `team1`
      * Another for `team2`
   * Two `Kustomization` objects for corresponding created `GitRepository` objects
8. Wait for reconciliation or force it
9. Inspect `gitrepositories` and `kustomizations`
10. Verify the namespaces and applications are created
11. Remove `GitRepository` and `Kustomization` for `team1`
12. Wait for reconciliation or force it
13. Verify that `team1` applications are removed


# B

## exercise 1 (optional)

We would like to have application in `workshop-flux-myteam-myapp` Git repository.
There are a number of ways to achieve this (e.g. where we would place of `GitRepository`, `Kustomization`).
But for the sake of this exercise, we would like to have the following structure:

```
workshop-flux-boostrap/               # our initial Git repository
├── "gotk-* file"
├── myteam-gitrepository.yaml         # pointing to workshop-flux-myteam
└── myteam-kustomization.yaml

workshop-flux-myteam/
├── myteam-myapp-gitrepository.yaml   # pointing to workshop-flux-myteam-myapp
└── myteam-myapp-kustomization.yaml

workshop-flux-myteam-myapp/
└── app.yaml                          # use default namespace
```
