# openshift-gitops
Initial setup

1. Install gitops 1.3.1+ operator

2. Create gitops-admins group in OCP

3. oc create -f config/app-management-appproject.yaml

4. oc create -f config/app-management-application.yaml

Sample application onboarding

Sample application manifests (namespace, role binding, ...) are stored in https://github.com/jstakun/openshift-gitops.git 

1. Create sample-apps-admin group in OCP

2. Sync apps-management ArgoCD application with OCP cluster

3. oc create -f config/sample-appproject.yaml

4. oc create -f config/sample-applicationset.yaml

