```
#oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n apps-infra
#oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n stackrox
#oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n openshift-workspaces
```

```
oc get installplan -A
oc patch installplan install-8lvrn --namespace openshift-compliance --type merge --patch '{"spec":{"approved":true}}'
oc patch installplan install-l2htz --namespace openshift-operators --type merge --patch '{"spec":{"approved":true}}'
oc patch installplan install-9qw8q --namespace openshift-workspaces --type merge --patch '{"spec":{"approved":true}}'
oc patch installplan install-xtlgj --namespace rhacs-operator --type merge --patch '{"spec":{"approved":true}}'
```
