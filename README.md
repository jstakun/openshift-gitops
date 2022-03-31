# openshift-gitops
Initial setup

1. Install OpenShift Gitops 1.3.1+ operator

2. Edit argocd-rbac-cm config map in openshift-gitops project and set policy.default: ''

3. Create gitops-admins group in OCP

4. Setup app managment ArgoCD application 
```
oc create -f config/app-management-appproject.yaml
oc create -f config/app-management-application.yaml
```
---
Sample application onboarding

Sample application manifests (namespace, role binding, ...) are stored in the [manifests](https://github.com/jstakun/openshift-gitops/tree/main/manifests) directory

1. Synchronize apps-management application in ArgoCD UI/CLI

2. Sync sample-app-stage, sample-app-test, sample-app-prod and sample-app-cicd ArgoCD applications

3. There are 2 role bindings created for each project with view and admin project roles which are bound to sample-app-viewers and sample-app-admins groups. If you want to use them you must create these groups

---
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
5. Optionally modify maven mirror settings in maven-settings configmap to point to your maven repo

---
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
stringData:
  password: ROBOT_ACCOUNT_PASSWORD
  username: ROBOT_ACCOUNT_NAME
type: kubernetes.io/basic-auth
```

3. Add secret to pipeline service account in namespace where pipeline will be executed

```
oc patch serviceaccount pipeline -p '{"secrets": [{"name": "quay-creds"}]}'
```

---
If you want to use github as source code repository here are the steps to take:

1. Create token in github with repo scope

2. Create github-creds opaque secret

```
kind: Secret
apiVersion: v1
metadata:
  name: github-creds
stringData:
  .git-credentials: https://USERNAME:TOKEN@github.com
  .gitconfig: |
    [credential "https://github.com"]
      helper = store
type: Opaque
```

3. When staring new pipeline select github-creds secret in git-basic-auth workspace 

---
If you want to test security pipelines following steps are required 

1. Install Advanced Cluster Security for Kubernetes and configure it

2. Generate API token with Continuous Integration role

3. Create secret which will be referenced in the pipeline:

```
kind: Secret
apiVersion: v1
metadata:
  name: roxsecrets
stringData:
  rox_api_token: API_TOKEN
  rox_central_endpoint: central.stackrox.svc.cluster.local:443
type: Opaque
```

---
If you want to deploy sample nexus repo you can use following deployment

```
oc create -f https://raw.githubusercontent.com/jstakun/openshift-cicd-demo/main/infra/nexus.yaml
```
Nexus url: http://nexus.sample-app-cicd.svc:8081/repository/maven-public/

---
If you want to play with image signing you must install locally cosign and run following command
```
cosign generate-key-pair k8s://sample-app-cicd/signing-secrets
```
This command must be executed in context of your project where pipeline will be running so that signing secret can be stored there and accessed by pipeline task when it will be executed

---
Run secure pipeline
```
oc create -f config/pipelinerun-sample-app-pipeline.yaml
```
