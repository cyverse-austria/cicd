# cicd

# k8s-resources-deploy
[k8s-resources-deploy](k8s-resources-deploy/README.md) pipeline to deploy [k8s-resources]() which is kubernetes resources for cyverse.
It will clone a repository to a workspace, generate/load configmaps, secrets & deploy using existing bash script all the services.

# Preq
## Create SA and cluster rolebinding 

```bash
kubectl -n $NAMESPACE apply -f tekton-admin-account.yml
```

## git-clone private repository ssh auth
Create a secret with ssh keys to be able to clone the private repositories.
**NOTE:** after created ssh key, add the public key to your private repository and the private key to the secret.

```bash
# generate ssh key
ssh-keygen -t ed25519 -C "mojib.wali@tugraz.at"

# encode base64
cat private_key.pem|base64

# encode known_host
ssh-keyscan gitlab.com |base64

kubectl -n $NAMESPACE apply -f ssh-secret.yaml
```

## install [git-clone](https://hub.tekton.dev/tekton/task/git-clone) task

```bash
kubectl -n $NAMESPACE apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml
```

# Task

## Parameters

* **HELM_IMAGE**: Helm image that has Helm, Kubectl and gomplate installed (_default:_ `mbwali/k8s-resources:latest`)
* **COMMAND**: A command to run! (_required_)



## Apply task

```bash
kubectl apply k8s-resources-deploy/task.yaml -n $NAMESPACE
```

# Pipeline

## Parameters

* **repo-url**: Repository URL to clone from. (_required_)
* **COMMAND**: A command to run (_required_)


## Tasks

* **clone-prvt-repo**: Clone a private Repository to a workspace.
* **command**: Run a giving command.


## Apply Pipeline

```bash
kubectl apply k8s-resources-deploy/pipeline.yaml -n $NAMESPACE
```


# TODO

* add helm to the image:
* tekton cleanup: https://gist.github.com/ctron/4764c0c4c4ea0b22353f2a23941928ad
