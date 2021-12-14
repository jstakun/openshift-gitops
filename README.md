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

If you want to play with Tekton CI pipeline which has been installed in sample-app-cicd project here are additional steps to take:

1. Install OpenShift Pipelines operator

2. Setup image registry so that pipeline can push images to it and deployments can pull from it

3. Setup git repository where you should clone this repo

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

