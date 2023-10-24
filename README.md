# NaaVRE-dev-environment
Integrated development environment for NaaVRE


## Getting started

### Git setup

To integrate the different components of NaaVRE, we use Git submodules:

```shell
git clone --recurse-submodules git@github.com:QCDIS/NaaVRE-dev-environment.git
```

Checkout the [Git Submodules documentation](https://git-scm.com/book/en/v2/Git-Tools-Submodules).

### Conda environment

Install Anaconda from these instructions: https://linuxize.com/post/how-to-install-anaconda-on-ubuntu-20-04/

Setup a new conda environment and install dependencies:

```shell
conda create --name naavre-dev
conda activate naavre-dev
conda install -c conda-forge pre_commit minikube tilt
```

### Pre-commit hooks

To install and enable pre-commit hooks, run:

```shell
conda activate naavre-dev
pip install pre-commit
pre-commit install
pip install ggshield
ggshield auth login
```

### Minikube cluster

The NaaVRE components are deployed by tilt to a local Kubernetes using minikube. We use ingress-dns to access those resources. To configure it, follow step 3 section of the [minikube ingress-dns setup guide](https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/). Chose your operating system. For most linux distributions, follow the section “Linux OS with Network Manager”. 


## Run the dev environment

```shell
minikube start
minikube addons enable ingress
minikube addons enable ingress-dns
tilt up
```

Optional: 

```shell
minikube dashboard --url
```

To reset the environment, exit Tilt and run:

```shell
minikube delete
```


## Access the services

The dev services are only accessible locally, using the domain name naavre-dev.minikube.test (provided minikube ingress-dns was setup). That allows us to use insecure credentials to login to the services.

### Keycloak

https://naavre-dev.minikube.test/auth/

| Account                  | Username | Password |
|--------------------------|----------|----------|
| Superuser (master realm) | `admin`  | `admin`  |
| User (vre realm)         | `user`   | `user`   |

### Argo

https://naavre-dev.minikube.test/argowf/

Login through keycloak.

| Account                   | Token        |
|---------------------------|--------------|
| `vre-api` service account | `token_argo` |

### VREPaaS

UI: https://naavre-dev.minikube.test/vreapp

Login through keycloak.

Admin interface: https://naavre-dev.minikube.test/vre-api-test/admin/

| Account       | Username | Password | Token          |
|---------------|----------|----------|----------------|
| Administrator | `admin`  | `admin`  |                |
| API user      | `user`   | `user`   | `token_vreapi` |

### NaaVRE-dev

https://naavre-dev.minikube.test/n-a-a-vre-dev

No authentication.

This version of NaaVRE runs Jupyter Lab alone (i.e. without Jupyter Hub), and updates automatically when the NaaVRE code is changed. It is suited for testing NaaVRE features, but not for testing integration (in that case, see NaaVRE section below).

### NaaVRE

https://naavre-dev.minikube.test/n-a-a-vre/

Login through keycloak.

This version of NaaVRE is controlled by Jupyter Hub, and is closer to the actual deployed version. However, it will not update automatically.

To show changes to the NaaVRE component in Tilt:
- Wait for the NaaVRE-dev/n-a-a-vre-dev resource to be updated (this can take a while)
- Restart the user server (from Jupyter Lab: File > “Hub Control Panel”, then “Stop my Server” and “Start my Server”)

This is necessary because the Jupyter Lab pod is started dynamically by Jupyter Hub, which prevents Tilt from detecting when it should reload it.
It is usually not necessary to reload the NaaVRE/hub and proxy resources, even if Tilt says it has changes.


## Development cycle

The different components of NaaVRE have their own Git repositories, which are included as submodules of the NaaVRE-dev-environment repository. In the context of the dev repo, these submodules are references to a commit in the component repo.
When in root directory of this repo, `git` commands apply to the NaaVRE-dev-environment repo.
When in the submodule directory, `git` commands apply to the submodule repo.

For any development task, follow this cycle:

1. On GitHub, create an issue in the appropriate component repository
2. Create a branch linked to the issue (e.g. `nnn-my-branch`)
3. Checkout this branch in the submodule:

   ```bash
   cd submodules/COMPONENT
   git fetch origin
   git checkout nnn-my-branch
   ```

4. Edit code in the submodule while checking the changes with Tilt

   *Note:* During development, running `git status` in the NaaVRE-dev-environment root directory will show unstaged changes to the submodule, such as `modified: submodule/COMPONENT (untracked content)` or `(new commits)`.

5. Commit and push changes from the submodule directory
6. On GitHub, create a pull request in the submodule repo
7. Once it is merged:
   - In the submodule directory, switch back to the main branch and pull the latest changes
   - In the NaaVRE-dev-environment directory, stage and commit the changes to the submodule. An appropriate commit message would be “update COMPONENT ref merging COMPONENT/nnn-my-branch”


## Troubleshooting

### Context deadline exceeded when pulling NaaVRE image

If you get an error similar to `Failed to pull image "qcdis/n-a-a-vre-laserfarm:v2.0-beta": rpc error: code = Unknown desc = context deadline exceeded` in the `continuous-image-puller` logs:

- Reset the cluster (`minikube delete` and re-run the startup commands)
- Run `minikube image load qcdis/n-a-a-vre-laserfarm:v2.0-beta` in your terminal
- Run tilt
