# openshift-gitops
Initial setup

1. Install Gitops operator 
```
oc apply -f bootstrap/gitops/gitops-operator.yaml
```
2. Approve Gitops operator creation
```
PLAN=$(oc get installplan -A | grep openshift-gitops-operator | awk '{print $2}')
oc patch installplan $PLAN --namespace openshift-operators --type merge --patch '{"spec":{"approved":true}}'
```
3. Create gitops-admins group and cluster config ArgoCD applications
```
oc apply -k bootstrap/default/
```
4. Edit argocd-rbac-cm config map in openshift-gitops project and set policy.default: ''
```
oc edit cm argocd-rbac-cm -n openshift-gitops
```
---
Sample application onboarding

Sample application manifests (namespace, role binding, ...) are stored in the [apps/bootstrap](https://github.com/jstakun/openshift-gitops/tree/main/apps/bootstrap) directory

1. Synchronize app-infra-commons application in ArgoCD UI/CLI

This will enable user workload monitoring and will grant openshift-gitops service account permission to create service monitors in application projects

2. Synchronize apps-management application in ArgoCD UI/CLI

3. Synchronize app-infra-nexus and app-infra-pipelines applications in ArgoCD UI/CLI 

4. Login to ArgoCD as sample-app-admins group member and sync sample-app-stage, sample-app-test, sample-app-prod and sample-app-cicd ArgoCD applications

---

If you want to play with Tekton CI pipeline which has been installed in sample-app-cicd project here are additional steps to take:

1. Setup image registry so that pipeline can push images to it and deployments can pull from it. Check below for sample quay.io setup.

2. Setup git repository where you should clone this repo. Check below for sample github setup.

3. (Optional) If build-image task fails you can increase resources for build-and-push step in buildah cluster task
```
spec:
  steps:
    name: build-and-push
      resources: 
        requests:
          memory: 4Gi
          cpu: 2
        limits:
          memory: 8Gi
          cpu: 4 
```
4. (Optional) Modify maven mirror settings in maven-settings configmap to point to your local maven repo

---
If you want to use quay.io image registry to push images from the pipeline here are the steps to take:

1. Create robot account in quay.io

2. Create basic auth secret in openshift with robot account credentials

```
kind: Secret
apiVersion: v1
metadata:
  name: quay-creds
  namespace: sample-app-cicd
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
  namespace: sample-app-cicd
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

1. Synchronize app-infra-acs application in ArgoCD UI/CLI 

This will install [Advanced Cluster Security for Kubernetes](https://docs.openshift.com/acs/3.69/installing/install-ocp-operator.html) 

2. Generate API token with Continuous Integration role

3. Create secret which will be referenced in the pipeline:

```
kind: Secret
apiVersion: v1
metadata:
  name: roxsecrets
  namespace: sample-app-cicd
stringData:
  rox_api_token: API_TOKEN
  rox_central_endpoint: central.stackrox.svc.cluster.local:443
type: Opaque
```
4. For image signing install [Cosign CLI](https://github.com/sigstore/cosign/releases) and run following command
```
cosign generate-key-pair k8s://sample-app-cicd/signing-secrets
```
Execute cosign command in the context of sample-app-cicd project so that signing secret is stored there and accessed by the pipeline task when it will be executed. You can later reuse this secret for signing other images.

5. In order to execute dependency analysis install [CRDA CLI](https://github.com/fabric8-analytics/cli-tools/releases).

6. Run following command to assign user key in ~/.crda/config.yaml.
```
crda auth
```

7. Create secret containing user key generated in previous step
```
apiVersion: v1
kind: Secret
metadata:
  name: crda
  namespace: sample-app-cicd
type: Opaque
stringData:
  crda-key: {{CRDA_USER_KEY}}
```
---
If you want to integrate with Sonar create following secret. Make sure to replace all SONAR_ placeholders and sonar.host.url properties with your settings.
```
kind: Secret
apiVersion: v1
metadata:
  name: sonar-properties
  namespace: sample-app-cicd
stringData:
  sonar-project.properties: |
    sonar.organization=SONAR_ORG
    sonar.login=SONAR_TOKEN
    sonar.qualitygate.wait=false
    sonar.host.url=https://sonarcloud.io
    sonar.projectKey=SONAR_PROJECT_KEY
    sonar.java.binaries=target
```
