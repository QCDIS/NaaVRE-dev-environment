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

[//]: # (Todo: add GitGuardian setup)

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



## Troubleshooting

### Context deadline exceeded when pulling NaaVRE image

If you get an error similar to `Failed to pull image "qcdis/n-a-a-vre-laserfarm:v2.0-beta": rpc error: code = Unknown desc = context deadline exceeded` in the `continuous-image-puller` logs:

- Reset the cluster (`minikube delete` and re-run the startup commands)
- Run `minikube image load qcdis/n-a-a-vre-laserfarm:v2.0-beta` in your terminal
- Run tilt
