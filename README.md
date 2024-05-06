# NaaVRE-dev-environment

Integrated development environment for NaaVRE.

OS support:

- ✅ Linux (tested on Ubuntu 22.04 and Arch Linux)
- ❌ Windows (user report: some conda packages cannot be installed)
- ❌ WSL2 (user report: Minikube ingress DNS does not work)
- ❔ macOS

## Getting started

*Run these steps once, when setting up the environment.*

### Git setup

To integrate the different components of NaaVRE, we use Git submodules:

```shell
git clone --recurse-submodules git@github.com:QCDIS/NaaVRE-dev-environment.git
```

If you get an error:

```
Cloning into 'NaaVRE-dev-environment'...
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

then you need to add your ssh key to your GitHub account. Follow the instructions [here](https://docs.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account).

Check out the [Git Submodules documentation](https://git-scm.com/book/en/v2/Git-Tools-Submodules).

### Conda environment

Install Conda from these instructions: https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html

Setup a new conda environment and install dependencies:

```shell
cd NaaVRE-dev-environment
conda env create -n naavre-dev --file environment.yml
conda activate naavre-dev
```

### Pre-commit hooks

To install and enable pre-commit hooks, run:

```shell
conda activate naavre-dev
pre-commit install
ggshield auth login
```

### Minikube cluster

The NaaVRE components are deployed by tilt to a local Kubernetes using
minikube. We use ingress-dns to access those resources. To configure it, start minikube (`minikube start`), and follow step 3 section of the [minikube ingress-dns setup guide](https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/). Choose your operating system.

For Linux, pick the configuration matching your distribution's DNS setup. To find the DNS setup, run `head /etc/resolv.conf`:

- If it mentions resolvconf: follow the [Linux OS with resolvconf](https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/#linux-os-with-resolvconf) instructions.
- If it contains `# Generated by NetworkManager`: follow the [Linux OS with Network Manager](https://minikube.sigs.k8s.io/docs/handbook/addons/ingress-dns/#linux-os-with-network-manager) instructions.
- (**For Ubuntu>=22.04**) If it contains `# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).`: run the following commands (systemd-resolved is not covered by the minikube documentation):
```shell
sudo mkdir /etc/systemd/resolved.conf.d
sudo tee /etc/systemd/resolved.conf.d/minikube.conf << EOF
[Resolve]
DNS=$(minikube ip)
Domains=~test
EOF
sudo systemctl restart systemd-resolved
```

### Helm dependencies

During the initial setup, and after updating `submodules/VREPaaS-helm-charts`, run:

```shell
helm dependency build services/vrepaas/submodules/VREPaaS-helm-charts
```

### GitHub repository for building cells

To containerize cells from this dev environment, you need to set up a personal GitHub repository. It will be used to commit the cells code and build and publish the container images:
1. Create your repository from the [QCDIS/NaaVRE-cells](https://github.com/QCDIS/NaaVRE-cells) template, and follow instructions from its README file to generate an access token.
2. Set the values of `CELL_GITHUB` and `CELL_GITHUB_TOKEN` in [./services/naavre/helm/values-integration.yaml](services/naavre/helm/values-integration.yaml) and [./services/naavre-dev/helm/values-dev.yaml](services/naavre/helm/values-dev.yaml).


## Run the dev environment

*Run these steps every time you want to start the dev environment.*

### Start minikube

```shell
minikube start  --addons=ingress,ingress-dns
# Optional:
minikube dashboard --url
```

### Start the services needed by NaaVRE

```shell
tilt up
```

This will open the Tilt dashboard in your browser, and deploy the services needed by NaaVRE.

After starting Tilt, we need to configure the connection between Argo and the VREPaaS. To that end, open a terminal and run `token=$(kubectl get secret vre-api.service-account-token -o jsonpath='{.data.token}' | base64 -d); echo "Bearer $token"`. Next, edit [services/vrepaas/helm/values.yaml](services/vrepaas/helm/values.yaml) add the output of the previous command (`Bearer ey.....`) to `global.argo.token`.

After updating the helm values, open the Tilt web interface, wait for the `Tiltfile` resource to update, and trigger a manual update on `vrepaas-vreapi`.

### Start NaaVRE

There three options for starting NaaVRE

#### Option 1: Run NaaVRE locally

Run the NaaVRE dev server locally (e.g. from a separate clone of the repository).

To that end, follow the instructions from https://github.com/QCDIS/NaaVRE/blob/main/README.md#development, creating the file `export_VARS` containing:

```
export API_ENDPOINT="https://naavre-dev.minikube.test/vre-api-test"
export ARGO_WF_SPEC_SERVICEACCOUNT="executor"
export CELL_GITHUB=""
export CELL_GITHUB_TOKEN=""
export JUPYTERHUB_SINGLEUSER_APP="jupyter_server.serverapp.ServerApp"
export JUPYTERHUB_USER="user"
export MODULE_MAPPING_URL="https://raw.githubusercontent.com/QCDIS/NaaVRE-conf/main/module_mapping.json"
export NAAVRE_API_TOKEN="token_vreapi"
export PROJ_LIB="/venv/share/proj"
export SEARCH_API_ENDPOINT=""
export SEARCH_API_TOKEN=""
export VLAB_SLUG="n-a-a-vre"
```

(Fill in your values for `CELL_GITHUB` and `CELL_GITHUB_TOKEN`.)

This option is recommended when developing NaaVRE Jupyter lab extensions, because it provides the fastest reloading on code changes. However, is options 2 and 3 are easier to setup.

#### Option 2: Run NaaVRE with Tilt (Jupyter Lab only)

Run a dev image of NaaVRE with Tilt, built from [./services/naavre/submodules/NaaVRE](./services/naavre/submodules/NaaVRE). This option deploys NaaVRE as a standalone Jupyter Lab service.

To that end, run the following command (make sure `tilt up` is running):

```shell
tilt enable n-a-a-vre-dev
```

This option is recommended when jointly developing NaaVRE and the VREPaaS, if you don’t need to test integration between NaaVRE Jupyter hub or Keycloak.

#### Option 3: Run NaaVRE with Tilt (Jupyter Hub integration)

Similar to option 2, but NaaVRE is deployed through Jupyter Hub.

To that end, run the following command (make sure `tilt up` is running):

```shell
tilt enable n-a-a-vre-dev hub proxy user-placeholder user-scheduler
```

This option is recommended to test integration of NaaVRE with Jupyter Hub or Keycloak.

### Start extra services (optional)

To test the integration of extra services, run:

```shell
tilt enable [n-a-a-vre-dev hub proxy user-placeholder user-scheduler] minio
```

### Resetting the dev environment

To reset the services, exit tilt and run `tilt down`. To fully reset the minikube cluster, run `minikube delete`.


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

| Account                   | Token   |
|---------------------------|---------|
| `vre-api` service account | Dynamic |

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

### NaaVRE-integration

https://naavre-dev.minikube.test/n-a-a-vre-integration/

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
   cd services/component/submodules/COMPONENT
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


## Extra services

### Minio

Admin interface: http://127.0.0.1:9001/

| Account       | Username | Password   | Token          |
|---------------|----------|------------|----------------|
| Administrator | `admin`  | `password` |                |

### Velero


Before installing Velero, you need to create an access key and bucket.
To create the access kay and bucket access the Minio UI (http://127.0.0.1:9001/).

#### Access key
Create an access key with the following id and secret:
```
aws_access_key_id = minio
aws_secret_access_key = minio123
```

After creating the key you need to specify its Access Key Policy to allow Velero to access the bucket `naavre-dev.minikube.test`:
```yaml
{
 "Version": "2012-10-17",
 "Statement": [
  {
   "Effect": "Allow",
   "Action": [
    "s3:GetBucketLocation",
    "s3:ListBucket",
    "s3:ListBucketMultipartUploads"
   ],
   "Resource": [
    "arn:aws:s3:::naavre-dev.minikube.test"
   ]
  },
  {
   "Effect": "Allow",
   "Action": [
    "s3:AbortMultipartUpload",
    "s3:DeleteObject",
    "s3:GetObject",
    "s3:ListMultipartUploadParts",
    "s3:PutObject"
   ],
   "Resource": [
    "arn:aws:s3:::naavre-dev.minikube.test/*"
   ]
  }
 ]
}
```

#### Bucket

Create a bucket named `naavre-dev.minikube.test` and add the following access policy:
```yaml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "minio"
                ]
            },
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads"
            ],
            "Resource": [
                "arn:aws:s3:::naavre-dev.minikube.test"
            ]
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "minio"
                ]
            },
            "Action": [
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:ListMultipartUploadParts",
                "s3:PutObject",
                "s3:AbortMultipartUpload"
            ],
            "Resource": [
                "arn:aws:s3:::naavre-dev.minikube.test/*"
            ]
        }
    ]
}
```

#### Install Velero

Follow the instructions [here](https://velero.io/docs/v1.10/basic-install/) to install Velero.

```shell
velero install --provider aws --use-node-agent --plugins velero/velero-plugin-for-aws:v1.2.1 \
--bucket naavre-dev.minikube.test --secret-file ./services/velero/credentials-velero --backup-location-config \
region=minio,s3ForcePathStyle="true",s3Url=http://host.minikube.internal:9000
```

#### Backup and restore

To backup a namespace:
```shell
velero backup create default-ns-backup --default-volumes-to-fs-backup --include-namespaces default --wait
```

You must apply an annotation to every pod which contains volumes for Velero to use FSB for the backup. For keycloak, we
have annotated postgresql in helm_config/keycloak/values.yaml:
```yaml
postgresql:
  enabled: true
  auth:
    postgresPassword: fake_postgres_password
    password: fake_password
  annotations:
    backup.velero.io/backup-volumes: pvc-volume,emptydir-volume
```
of
```yaml
singleuser:
  extraAnnotations:
    backup.velero.io/backup-volumes: pvc-volume,emptydir-volume
  cmd: ['/usr/local/bin/start-jupyter-venv.sh']
```

To restore a namespace:
```shell
velero restore create --from-backup default-ns-backup
```

#### Simulate a disaster

Find the container running Minikube:
```shell
docker ps | grep k8s-minikube
```
Access the container running Minikube
```shell
docker exec -it <container-id> /bin/bash
```
Delete the Keycloak postgresql data directory:

```shell
    rm -r  /tmp/hostpath-provisioner/default/data-keycloak-postgresql-0/
```

Go to https://naavre-dev.minikube.test/auth/. If the DB is missing, you won't be able to log in or you will get an error
message:
```
Unexpected Application Error!
Network response was not OK.

NetworkError@https://naavre-dev.minikube.test/auth/resources/9qowb/admin/keycloak.v2/assets/index-d73da1a7.js:67:43535
fetchWithError@https://naavre-dev.minikube.test/auth/resources/9qowb/admin/keycloak.v2/assets/index-d73da1a7.js:67:43710
```


Restore the namespace:
```shell
velero restore create --from-backup default-ns-backup --wait
```

Try again to log in to Keycloak.
