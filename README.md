# openshift-gitops
Initial setup

1. Install OpenShift Gitops 1.3.1+ operator

2. Create gitops-admins group in OCP

3. oc create -f config/app-management-appproject.yaml

4. oc create -f config/app-management-application.yaml

Sample application onboarding

Sample application manifests (namespace, role binding, ...) are stored in https://github.com/jstakun/openshift-gitops.git in manifests directory

1. Create sample-app-admins group in OCP

2. Synchronize apps-management ArgoCD application

3. oc create -f config/sample-appproject.yaml

4. oc create -f config/sample-applicationset.yaml

5. Sync sample-app-stage, sample-app-test, sample-app-prod and sample-app-cicd ArgoCD applications

6. There are 2 role bindings created for each project with view and admin roles which are bound to sample-app-viewer and sample-app-admin groups. If you want to use them you must create these groups

If you want to play with Tekton CI pipeline which has been installed in sample-app-cicd project here are additional steps to take:

1. Install OpenShift Pipelines operator

2. Setup image registry so that pipeline can push images to it and deployments can pull from it. Check below for sample quay.io setup.

3. Setup git repository where you should clone this repo. Check below for sample github setup.

4. Increase resources for build container in buildah cluster task
```
spec:
  steps:
    name: build
      resources: 
        requests:
          memory: 4Gi
          cpu: 2
        limits:
          memory: 8Gi
          cpu: 4 
```
5. Modify maven mirror settings in maven-settings configmap and in build-image pipeline task (BUILD_EXTRA_ARGS parameter) to point to your maven repo

If you want to use quay.io image registry to push images from the pipeline here are the steps to take:

1. Create robot account in quay.io

2. Create basic auth secret in openshift with robot account credentials

```
kind: Secret
apiVersion: v1
metadata:
  name: quay-creds
  annotations:
    tekton.dev/docker-0: 'https://quay.io'
data:
  password: ROBOT_ACCOUNT_PASSWORD
  username: ROBOT_ACCOUNT_NAME
type: kubernetes.io/basic-auth
```

3. Add secret to pipeline service account in namespace where pipeline will be executed

```
oc patch serviceaccount pipeline -p '{"secrets": [{"name": "quay-creds"}]}'
```

If you want to use github as source code repository here are the steps to take:

1. Create token in github with repo scope

2. Create github-creds opaque secret

```
kind: Secret
apiVersion: v1
metadata:
  name: github-creds
data:
  .git-credentials: https://USERNAME:TOKEN@github.com
  .gitconfig: |
    [credential "https://github.com"]
      helper = store
type: Opaque
```

3. When staring new pipeline select github-creds secret in git-basic-auth workspace 

If you want to test security pipelines following steps are required 

1. Install Advanced Cluster Security for Kubernetes and configure it

2. Generate API token with Continuous Integration role

3. Create secret which will be referenced in the pipeline:

```
kind: Secret
apiVersion: v1
metadata:
  name: roxsecrets
data:
  rox_api_token: API_TOKEN
  rox_central_endpoint: central.stackrox.svc.cluster.local:443
type: Opaque
```







