# README

## Install OpenShift Pipelines Operator

Install version 1.2.3 of Red Hat OpenShift Pipelines Operator

## Configure pipeline service account used by tekton

Create docker registry secret
```
kubectl create secret docker-registry registry-credentials --docker-server=https://index.docker.io/v2/  --docker-username=gkovan --docker-email=gkovan@hotmail.com --docker-password=<my-docker-hub-pw> -n dev
```

```
kubectl patch serviceaccount pipeline -p "{\"secrets\": [{\"name\": \"registry-credentials\"}]}" -n dev
```

```
kubectl patch serviceaccount pipeline -p "{\"imagePullSecrets\": [{\"name\": \"registry-credentials\"}]}" -n dev
```

## Setup Tekton chains

Install tekton chains

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/chains/latest/release.yaml
```

Generate key pair and create signing secret
```
cosign generate-key-pair -k8s tekton-chains/signing-secrets
```

Create docker registry secret
```
kubectl create secret docker-registry registry-credentials --docker-server=https://index.docker.io/v2/  --docker-username=gkovan --docker-email=gkovan@hotmail.com --docker-password=<my-docker-hub-pw> -n tekton-chains
```

Patch tekton-chains-controller service account to be able to push signature
```
kubectl patch serviceaccount tekton-chains-controller -p "{\"imagePullSecrets\": [{\"name\": \"registry-credentials\"}]}" -n tekton-chains
```

Patch pipeline service account to be able to push signature (I don't think this is necessary)
```
kubectl patch serviceaccount pipeline -p "{\"imagePullSecrets\": [{\"name\": \"registry-credentials\"}]}" -n tekton-chains
```

# Pipeline for quarkus-base-image

Create the tasks
```
oc apply -f ./quarkus-base-image/task/
```

Create the pipeline
```
oc apply -f ./quarkus-base-image/pipeline/
```

An event listener and tekton trigger have not been setup yet for the quarkus base image so you need to manually start the pipeline using OpenShift console (note: default values exist for params)"

# Pipeline for quarkus-hello-service microservice

Create the tasks
```
oc apply -f ./quarkus-hello-service/task/
```

Create the pipeline
```
oc apply -f ./quarkus-hello-service/pipeline/
```

Create the trigger
```
oc apply -f ./quarkus-hello-service/trigger/
```

Expose the service to create a route
```
oc expose service el-quarkus-hello-service-app
``` 

You will need to do a commit against the git repo to kick off the pipeline.
In the quarkus-hello-service, folder, enter command to do an empty commit:
```
git commit -m "empty-commit" --allow-empty && git push origin main
```
