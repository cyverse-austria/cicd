
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

## Task

## Parameters

* **BASE_IMAGE**: Base image that has Helm, Kubectl, gomplate & skaffold installed. (_default:_ `mbwali/k8s-resources:latest`)
* **COMMAND**: A command to run! (_required_)
* **ENV**: usually means namespace also the Environment of the cyverse. (_default:_ `qa`)


### Apply task

```bash
kubectl apply k8s-resources-deploy/task.yaml -n $NAMESPACE
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
  - name: shared-data
    volumeClaimTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
  params:
  - name: repo-url
    value: <YOUR-K8S-resource-repository>
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
