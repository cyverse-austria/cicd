# cicd

# k8s-resources-deploy
[k8s-resources-deploy](k8s-resources-deploy/README.md) pipeline to deploy [k8s-resources]() which is kubernetes resources for cyverse.
It will clone a repository to a workspace, generate/load configmaps, secrets & deploy using existing bash script all the services.

# TODO

* tekton cleanup: https://gist.github.com/ctron/4764c0c4c4ea0b22353f2a23941928ad
* combine both tasks - into one pipeline.