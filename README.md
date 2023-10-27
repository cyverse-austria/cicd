# cicd

# [k8s-resources-deploy](k8s-resources-deploy/) 
k8s-resources-deploy [Pipeline](k8s-resources-deploy/pipeline.yaml) has two tasks:

**[docker-data-task](k8s-resources-deploy/docker-data-task.yaml)** This task is cloning repository [docker-tugraz-data]() to a workspace, and run the `load_secrets` bash from it.

**[k8s-resources-task](k8s-resources-deploy/k8s-resources-task.yaml)** It will clone  the [k8s-resources]() repository to a workspace, generate/load configmaps, secrets & deploy using existing bash script all the services.


## Preq
### Create SA and cluster rolebinding 

```bash
kubectl -n $NAMESPACE apply -f k8s-resources-deploy/tekton-admin-account.yml
```

### Apply Tekton default configs

```bash
kubectl -n $NAMESPACE apply -f k8s-resources-deploy/config-defaults.yaml
```

### Cloning private repository ssh auth
Create a secret with ssh keys to be able to clone the private repositories.
**NOTE:** after created ssh key, add the public key to your private repository and the private key to the secret.

```bash
# generate ssh key
ssh-keygen -t ed25519 -C "admin@example.com"

# encode base64
## MAKE SURE you copy one line code
cat private_key.pem|base64

# encode known_host
## MAKE SURE you copy one line code
ssh-keyscan gitlab.com |base64

kubectl -n $NAMESPACE apply -f ssh-secret.yaml
```

## docker-data-task

### Parameters

* **BASE_IMAGE**: Base image that has Helm, Kubectl, gomplate & skaffold installed. (_default:_ `mbwali/k8s-resources:latest`)
* **ENV**: usually means namespace also the Environment of the cyverse. (_default:_ `qa`)
* **REPO_URL_DOCKER_DATA**: Repository URL that will be cloned. (_required_)
* **userHome**: Home directory for the docker image used. (_default:_ `/home/k8s`)



#### Apply task

```bash
kubectl apply k8s-resources-deploy/docker-data-task.yaml -n $NAMESPACE
```

## k8s-resources-task

### Parameters

* **BASE_IMAGE**: Base image that has Helm, Kubectl, gomplate & skaffold installed. (_default:_ `mbwali/k8s-resources:latest`)
* **ENV**: usually means namespace also the Environment of the cyverse. (_default:_ `qa`)
* **REPO_URL_K8S_RESOURCES**: Repository URL that will be cloned. (_required_)
* **PROJECTS**: List of projects that should be deployed. (_required_)
* **userHome**: Home directory for the docker image used. (_default:_ `/home/k8s`)

#### Apply task

```bash
kubectl apply k8s-resources-deploy/k8s-resources-task.yaml -n $NAMESPACE
```


## Pipeline

### Parameters

* **BASE_IMAGE**: Base image that has Helm, Kubectl, gomplate & skaffold installed. (_default:_ `mbwali/k8s-resources:latest`)
* **ENV**: usually means namespace also the Environment of the cyverse. (_default:_ `qa`)
* **REPO_URL_K8S_RESOURCES**: Repository URL that will be cloned for k8s-resources. (_required_)
* **REPO_URL_DOCKER_DATA**: Repository URL that will be cloned for docker-data. (_required_)
* **PROJECTS**: List of projects that should be deployed. (_required_)

### Apply Pipeline

```bash
kubectl apply k8s-resources-deploy/pipeline.yaml -n $NAMESPACE
```

## PipelineRun

Here is an example how to run the pipeline:
```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: k8s-resources-pipeline-run-UNIQUE-ID
spec:
  serviceAccountName: tekton-admin-account
  pipelineRef:
    name: k8s-resources-pipeline
  workspaces:
  - name: ssh-creds
    secret:
      secretName: ssh-key
  params:
  - name: REPO_URL_K8S_RESOURCES
    value: <YOUR-K8S-resource-repository>
  - name: REPO_URL_DOCKER_DATA
    value: <YOUR-docker-data-repository>
  - name: ENV
    value: qa/prod
  - name: BASE_IMAGE
    value: <YOUR IMAGE>
  - name: PROJECTS
    value:
      - "a"
      - "b"
      - "c"
```

# TODO
* tekton cleanup: https://gist.github.com/ctron/4764c0c4c4ea0b22353f2a23941928ad
