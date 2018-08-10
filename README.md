# Building containers with Kubernetes and KNative

This tutorial uses GKE //TODO Who supports KNative?  Bare metal is out of scope, suggest kube adm
Knative is a framework to build kubernetes native applications. FaaS is just one of the capabilities.  It is new, bleeding edge

Created a simple hello-world we app
Multi Stage dockerfile

1.10.5-gke.3

gcloud container clusters get-credentials knative-builder --zone us-east1-b --project knative-playground


https://github.com/GoogleCloudPlatform/knative-build-tutorials

make sure cluster is up
kubectl get nodes


Permissions:  cluster admin to current user

kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=$(gcloud config get-value core/account)


Install knative build

kubectl apply -f https://storage.googleapis.com/build-crd/latest/release.yaml


Is it working:

kubectl get namespace knative-build
kubectl -n knative-build get pods -w


```
 nkennedy@xps  ~/go/src/github.com/colek42/knative-playground   master ?  kubectl -n knative-build get pods -w                                    ✔  1798  23:33:32 
NAME                               READY     STATUS    RESTARTS   AGE
build-controller-b76f56b4c-6km8j   1/1       Running   0          1m
build-webhook-6867b944b4-vbfxq     1/1       Running   0          1m
```

we have new crds 


```
 nkennedy@xps  ~/go/src/github.com/colek42/knative-playground   master ?  kubectl get crd                                                       1 ↵  1799  23:34:38 
NAME                                    CREATED AT
backendconfigs.cloud.google.com         2018-08-10T03:24:50Z
builds.build.knative.dev                2018-08-10T03:32:37Z
buildtemplates.build.knative.dev        2018-08-10T03:32:37Z
scalingpolicies.scalingpolicy.kope.io   2018-08-10T03:25:10Z
```


made a simple go hello world server github.com/boxboat/hellow-world-webapp

```
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: docker-build
spec:
  serviceAccountName: knative-build
  source:
    git:
      url: https://github.com/boxboat/hello-world-webapp.git
      revision: master
  steps:
  - name: build-and-push
    image: gcr.io/kaniko-project/executor:v0.1.0
    args:
    - --dockerfile=/workspace/Dockerfile
    - --destination=docker.io/boxboat/hello-world-webapp
```

#Authentication



we are using the kaniko project to build our images to avoid having to run docker, which has security implications
add info abount kaniko, links


```
apiVersion: v1
kind: Secret
metadata:
  name: basic-user-pass
  annotations:
    build.knative.dev/docker-0: https://gcr.io  # Described below
type: kubernetes.io/basic-auth
stringData:
  username: <username>
  password: <password>
```

stringData is converted to base64 encoded data when applied to the kubernetes cluster


service account needs to use this secret

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot
secrets:
- name: basic-user-pass
```


Ensure the Build uses the correct service account

When this build executes, before steps execute, a ~/.docker/config.json will be generated containing the credentials configured in the Secret, and these credentials are then used to authenticate with the Docker registry.


look at logs - auth failing

  annotations:
    build.knative.dev/docker-0: https://gcr.io ==>  build.knative.dev/docker-0: https://index.docker.io/v1/



