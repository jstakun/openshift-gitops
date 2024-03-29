
```
oc get installplan -A

PLAN=$(oc get installplan -A | grep compliance-operator | awk '{print $2}')
oc patch installplan $PLAN --namespace openshift-compliance --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep openshift-pipelines-operator | awk '{print $2}')
oc patch installplan $PLAN --namespace openshift-operators --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep container-security | awk '{print $2}')
oc patch installplan $PLAN --namespace openshift-operators --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep devspace |  awk '{print $2}')
oc patch installplan $PLAN --namespace openshift-operators --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep rhacs-operator | awk '{print $2}')
oc patch installplan $PLAN --namespace rhacs-operator --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep rhsso-operator | awk '{print $2}')
oc patch installplan $PLAN --namespace rhsso --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep logging | awk '{print $2}')
oc patch installplan $PLAN --namespace openshfit-logging --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep loki | awk '{print $2}')
oc patch installplan $PLAN --namespace openshift-operators-redhat --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep web-terminal | awk '{print $2}')
oc patch installplan $PLAN --namespace openshift-operators --type merge --patch '{"spec":{"approved":true}}'

PLAN=$(oc get installplan -A | grep rhods-operator | awk '{print $2}')
oc patch installplan $PLAN --namespace redhat-rhods-operator --type merge --patch '{"spec":{"approved":true}}'
```
