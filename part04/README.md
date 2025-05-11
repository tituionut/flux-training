# A

## exercise 1
Basic Kustomize usage

1. Remove everything from `workshop-flux-bootstrap` except `gotk-*` files
2. Make sure any applications and namespace removed
3. In your `workshop-flux-myteam` create
   * (See `example01` for the inspiration)
   * `kustomization.yaml` that points to an `app.yaml`
4. Create a `GitRepository` and `Kustomization` in `workshop-flux-bootstrap` that would point to `workshop-flux-myteam`
5. Wait for reconciliation or force it
   * Note, `workshop-flux-bootstrap`
6. Verify the application is created
7. Now create more applications in the root of `workshop-flux-myteam`
   * You can simply copy `app.yaml` and change the name of resources and provide namespace
8. Wait for reconciliation or force it
   * Note, `workshop-flux-myteam`
9. Are you applications created?
10. Now move the `kustomization.yaml` and `app.yaml` to a subfolder `myapp`
    * You could as well keep "consistency" and call folder `kustomization`
11. Wait for reconciliation or force it
    * Note, `workshop-flux-myteam`
12. Verify that all applications are created now


# B

## exercise 1
Basic use of Kustomize overlays.

Let's try the following `workshop-flux-myteam` structure.

```
workshop-flux-myteam/
├── base/
│   ├── deployment.yaml
│   ├── kustomization.yaml 
│   ├── namespace.yaml 
│   └── service.yaml
└── overlays/
    ├── dev/
    │   ├── index.html
    │   └── kustomization.yaml
    └── prod/
        ├── index.html
        └── kustomization.yaml
```

1. Create the above structure in `workshop-flux-myteam`
   * Make sure you can build overlays with `kubectl kustomize <PATH>`
2. Remove everything from `workshop-flux-bootstrap` except `gotk-*` files
3. Define one `GitRepository` in `workshop-flux-bootstrap` that points to `workshop-flux-myteam`
4. Define two `Kustomization` objects
   * One that points to `./overlays/dev`
   * Another that points to `./overlays/prod`
5. Make sure the applications are created
6. Port forward to application and verify the correct `index.html` is returned


# C
Using Kustomize to "prepare" environment.

Let's aim for the following structure for our `workshop-flux-myteam`.

```
workshop-flux-myteam/
├── myteam/
│   ├── dev/
│   │   ├── app1.yaml
│   │   ├── app2.yaml
│   │   ├── app3.yaml
│   │   ├── ..                       we could as well have kustomization.yaml here as well!
│   │   └── kustomization
│   │       └── kustomization.yaml   # points to ../../../components/overlays/dev (or base, but our ../../../components/overlays/dev could add some extras)
│   ├── prod/
│   │   ├── app1.yaml
│   │   ├── ..                       we could as well have kustomization.yaml here as well!
│   │   └── kustomization
│   │       └── kustomization.yaml   # points to ../../../components/overlays/prod
└── components/
    ├── base/
    │   ├── configmap1.yaml
    │   ├── configmap2.yaml
    │   ├── configmap3.yaml
    │   ├── namespace.yaml
    │   └── kustomization.yaml
    └── overlays/
        ├── dev/
        │   └── kustomization.yaml
        └── prod/
            └── kustomization.yaml
```

Use application from `example01`.

In our `workshop-flux-bootstrap` we will have:
* `GitRepository` pointing to `workshop-flux-myteam`
* `Kustomization` pointing to `myteam/dev`
* `Kustomization` pointing to `myteam/prod`

As a bonus you could:
* Delete `myteam/xxx/kustomization`
* Add `Deployment` and `Service` from `example02`
* Add `kustomization.yaml` to `myteam/xxx` instead referencing all the apps
* As in `example02` generate `index.html` `ConfigMap` in `components/overlays/xxx`
* Use the generated `Configmap` in `myteam/xxx/deployment.yaml`

So one could end up in something like this instead:

```
workshop-flux-myteam/
├── myteam/
│   ├── dev/
│   │   ├── app1.yaml
│   │   ├── app2.yaml
│   │   ├── app3.yaml
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml       # points to ../../components/overlays/dev
│   ├── prod/
│   │   └── ..
└── components/
    ├── base/
    │   ├── configmap1.yaml
    │   ├── configmap2.yaml
    │   ├── configmap3.yaml
    │   ├── namespace.yaml
    │   └── kustomization.yaml
    └── overlays/
        ├── dev/
        │   ├── index.html
        │   └── kustomization.yaml
        └── prod/
            └── ..
```
