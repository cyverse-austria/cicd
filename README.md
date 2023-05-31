# cicd

# [k8s-resources-deploy](k8s-resources-deploy/) 
k8s-resources-deploy [Pipeline](k8s-resources-deploy/pipeline.yaml) has two tasks:

**[docker-data-task](k8s-resources-deploy/docker-data-task.yaml)** This task is cloning repository [docker-tugraz-data]() to a workspace, and run the `load_secrets` bash from it.

**[k8s-resources-task](k8s-resources-deploy/k8s-resources-task.yaml)** It will clone  the [k8s-resources]() repository to a workspace, generate/load configmaps, secrets & deploy using existing bash script all the services.


## Preq
### Create SA and cluster rolebinding 

```bash
kubectl -n $NAMESPACE apply -f tekton-admin-account.yml
```

### git-clone private repository ssh auth
Create a secret with ssh keys to be able to clone the private repositories.
**NOTE:** after created ssh key, add the public key to your private repository and the private key to the secret.

```bash
# generate ssh key
ssh-keygen -t ed25519 -C "admin@example.com"

# encode base64
cat private_key.pem|base64

# encode known_host
ssh-keyscan gitlab.com |base64

kubectl -n $NAMESPACE apply -f ssh-secret.yaml
```

### install [git-clone](https://hub.tekton.dev/tekton/task/git-clone) task

```bash
kubectl -n $NAMESPACE apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml
```

## docker-data-task

### Parameters

* **BASE_IMAGE**: Base image that has Helm, Kubectl, gomplate & skaffold installed. (_default:_ `mbwali/k8s-resources:latest`)
* **ENV**: usually means namespace also the Environment of the cyverse. (_default:_ `qa`)



#### Apply task

```bash
kubectl apply k8s-resources-deploy/docker-data-task.yaml -n $NAMESPACE
```

## k8s-resources-task

### Parameters

* **BASE_IMAGE**: Base image that has Helm, Kubectl, gomplate & skaffold installed. (_default:_ `mbwali/k8s-resources:latest`)
* **COMMAND**: A command to run! (_required_)
* **ENV**: usually means namespace also the Environment of the cyverse. (_default:_ `qa`)


#### Apply task

```bash
kubectl apply k8s-resources-deploy/k8s-resources-task.yaml -n $NAMESPACE
```


## Pipeline

### Parameters

* **repo-url**: Repository URL to clone from. (_required_)
* **COMMAND**: A command to run (_required_)
* **BASE_IMAGE**: Base image that has Helm, Kubectl, gomplate & skaffold installed. (_default:_ `mbwali/k8s-resources:latest`)
* **ENV**: usually means namespace also the Environment of the cyverse. (_default:_ `qa`)

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
  - name: docker-data-workspace
    volumeClaimTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi

  - name: k8s-resources-workspace
    volumeClaimTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi

  params:
  - name: k8s-resources-repo-url
    value: <YOUR-K8S-resource-repository>
  - name: docker-data-repo-url
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
* use of empty dir
* run as non root user