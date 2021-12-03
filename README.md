# openshift-gitops
Initial setup

1. Install gitops 1.3.1+ operator

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
