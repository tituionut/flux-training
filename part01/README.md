# A 

You need:
* [`kubectl`](https://kubernetes.io/docs/tasks/tools/#kubectl)
  * You can simply download the binary `kubectl` from the [Kubernetes release page](https://kubernetes.io/releases/download/#binaries) and put it into your `PATH`
* Git account, where you could create public repositories (GitHub, GitLab, etc)
* Optionally `flux` cli tool
  * See installation process [here](https://fluxcd.io/flux/cmd/)
  * If you have troubles running it, it is not a big deal

To configure access to the cluster:
1. Download you cluster `cluster-<CLUSTER>-kubeconfig.yaml` file
2. Create `.kube` directory in your home directory
   * Windows `%USERPROFILE%/.kube`
   * Everywhere else `~/.kube`
   * If you have `.kube` directory present, back up existing `.kube/config` file
3. Copy the `cluster-<CLUSTER>-kubeconfig.yaml` file to the `.kube/config` file (yes, file without extension) 
   * Windows `%USERPROFILE%/.kube/config` (yes, file without extension)
   * Everywhere else `~/.kube/config` (yes, file without extension)
4. Make sure you get nodes out of _your_ cluster by writing
   * `kubectl get nodes`


# B

Here we will experiment with different ways to install Flux.
Even though we could outsource this to `flux bootstrap`, it is a good idea to attempt manual installation first.
In this way you will understand how it works under the hood.


## exercise 1
* Create a new Git repository (public) on your Git account
  * Name it something like `workshop-flux-bootstrap`
  * Note which branch you create
  * We can also work with private repositories, but it requires your token or ssh key to be saved in the cluster


## exercise 2
Install via yaml files.

There are several options to obtain the yaml files to install Flux (CRds, controllers, etc):
* One could simply point to the releases from GitHub
  * see https://github.com/fluxcd/flux2/releases
  * Flux automatically publishes `install.yaml` manifest
    * `kubectl apply -f https://github.com/fluxcd/flux2/releases/latest/download/install.yaml`
    * `kubectl apply -f https://github.com/fluxcd/flux2/releases/download/<VERSION>/install.yaml`
    * `kubectl apply -f https://github.com/fluxcd/flux2/releases/download/v2.5.1/install.yaml`
* Generate the yaml files from `flux` cli tool
  * `flux install --export --namespace flux-system > gotk-components.yaml`
  * `flux create source git flux-system --url=<GIT_HTTP_URL> --branch=<GIT_BRANCH_NAME> --export > gotk-repository.yaml`
  * `flux create kustomization flux-system --source=GitRepository/flux-system --path="." --prune=true --interval=1m --export > gotk-kustomization.yaml`

There are already pre-generated Flux installation yaml files in `install` directory in this folder.
"gotk" abbreviation stands for [GitOps Toolkit](https://fluxcd.io/flux/components/).

1. Adjust and push files to your `workshop-flux-boostrap` repository
   1. (This is though not strictly necessary. But in this way Flux CRDs, Flux controllers, bootstrap Git repository and bootstrap Kustomization are also managed by Flux. So in this way Flux can manage itself.)
   2. `install/gotk-components.yaml`
      * This file contains all the components required to run Flux
      * Push the `gotk-components.yaml` file to the root of your Git repository
        * This is not strictly necessary
        * However, this gives us an opportunity to manage Flux through itself
   3. `install/gotk-repository.yaml`
      * This file describes a git repository that Flux should watch
      * Adjust `<HTTP_GIT_URL>` and `<GIT_BRANCH_NAME>` in the `gotk-repository.yaml` file
        * fx `https://gitlab.com/<username>/<project_name>` and `master`
      * Push the `gotk-repository.yaml` file to the root of your Git repository
   4. `install/gotk-kustomization.yaml`
      * This file describes a _Kustomization_ that Flux should watch
        * This is not Kustomize, but a Flux Kustomization
        * We will get back to this later
      * Push the `gotk-kustomization.yaml` file to the root of your Git repository
2. Install Flux
   1. Apply the `workshop-flux-boostrap/gotk-components.yaml` file
   2. Analyze `kubectl api-resources | grep flux`
   3. List CRDs `kubectl get crd`
   4. List running pods from the `flux-system` namespace -- you should see Flux controllers
3. Setup Flux Git repository and Flux Kustomization
   1. Apply the `workshop-flux-boostrap/gotk-repository.yaml` file
   2. Apply the `workshop-flux-boostrap/gotk-kustomization.yaml` file
4. Make sure you do not have any errors in `kustomizations` and `gitrepositories` objects in `flux-system` namespace
5. Application
   1. Now push a `app.yaml` to the root your Git repository
      * Do not apply it, push it at the root of your Git repository -- Flux will do the job
   2. Now you can either wait one minute for Flux to fetch it from Git based on the interval in `GitRepository` `spec.interval` or you can manually force it.
      * `kubectl annotate --overwrite gitrepo/flux-system -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`
   3. Make sure you do not have any errors in `kustomizations` and `gitrepositories` objects in `default` namespace
   4. Describe `kustomizations` and `gitrepositories`
   5. Verify the application is created in the cluster


If you wish to uninstall Flux.
* Remove CRDs finalizers
  * `kubectl edit crd gitrepositories.source.toolkit.fluxcd.io`
    * Delete finalizers
  * `kubectl edit crd kustomizations.kustomize.toolkit.fluxcd.io`
    * Delete finalizers
* `kubectl delete` on files from your `workshop-flux-boostrap` Git repository
* Delete Git repo content and push it


## exercise 3 (optional)
Install via Helm chart.
In this case you can say that the Flux is managed by Helm now. 

1. Adjust and push files to your `workshop-flux-boostrap` repository
   1. (This is though not strictly necessary. But in this way Flux itself, bootstrap Git repository and bootstrap Kustomization are also managed by Flux. So in this way Flux manages itself now.)
   2. `install/gotk-repository.yaml`
      * This file describes a git repository that Flux should watch
      * Adjust `<HTTP_GIT_URL>` and `<GIT_BRANCH_NAME>` in the `gotk-repository.yaml` file
        * fx `https://gitlab.com/<username>/<project_name>` and `master`
      * Push the `gotk-repository.yaml` file to the root of your Git repository
   3. `install/gotk-kustomization.yaml`
      * This file describes a _Kustomization_ that Flux should watch
        * This is not Kustomize, but a Flux Kustomization
        * We will get back to this later
      * Push the `gotk-kustomization.yaml` file to the root of your Git repository
2. Install Helm chart
   1. See https://artifacthub.io/packages/helm/fluxcd-community/flux2
   2. Run
      * `helm repo add fluxcd-community https://fluxcd-community.github.io/helm-charts`
      * `helm install flux-system -n flux-system fluxcd-community/flux2 --create-namespace --version 2.15.0`
   3. Analyze `kubectl api-resources | grep flux`
   4. List CRDs `kubectl get crd`
3. Setup Flux Git repository and Flux Kustomization
   1. Apply the `workshop-flux-boostrap/gotk-repository.yaml` file
   2. Apply the `workshop-flux-boostrap/gotk-kustomization.yaml` file
   3. Make sure you do not have any errors in `kustomizations` and `gitrepositories` objects in `default` namespace
4. Application
   1. Now push a `app.yaml` to your Git repository
      * Do not apply it, push it at the root of your Git repository -- Flux will do the job
   2. Now you can either wait one minute for Flux to fetch it from Git based on the interval in `GitRepository` `spec.interval` or you can manually force it.
      * `kubectl annotate --overwrite gitrepo/flux-system -n flux-system reconcile.fluxcd.io/requestedAt="$(date +%s)"`
   3. Make sure you do not have any errors in `kustomizations` and `gitrepositories` objects in `default` namespace
   4. Describe `kustomizations` and `gitrepositories`
   5. Verify the application is created in the cluster

If you wish to uninstall Flux
* `helm uninstall -n flux-system flux-system`
  * Or other name and namespace that you have used
* Remove CRDs
  * `kubectl edit crd gitrepositories.source.toolkit.fluxcd.io`
    * Delete finalizers
  * `kubectl edit crd kustomizations.kustomize.toolkit.fluxcd.io`
    * Delete finalizers
* `kubectl delete crd gitrepositories.source.toolkit.fluxcd.io kustomizations.kustomize.toolkit.fluxcd.io`
* Delete Git repo content and push it

## exercise 4 (optional)

Try to set up `flux` using `flux boostrap` command.
Note there are variations for `git` and `github` (or `gitlab`) repositories.

* See https://fluxcd.io/flux/cmd/flux_bootstrap/
  * fx `flux bootstrap gitlab --owner=<USERNAME> --repository=<REPOSITORY_NAME> --branch=master --token-auth`

`flux bootstrap` automatically generates and adds required manifests to the Git repository.
So you do not need to do it manually, as in previous steps.

* After installation
  * Check the Git repository
  * Analyze `kubectl api-resources | grep flux`
  * Get `kustomizations` and `gitrepositories` from all namespaces
  * Describe `kustomizations` and `gitrepositories` from all namespaces

* If you wish to uninstall Flux
  * `flux uninstall`
    * Use `--namespace` option if you have installed it to a non-default (`flux-system`) namespace
  * Delete Git repo content and push it
