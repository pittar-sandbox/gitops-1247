# GITOPS-1247: "managed-by" label changed by new Argo CD Instance

This repo is meant to reproduce [GITOPS-1247](https://issues.redhat.com/browse/GITOPS-1247).

## Problem  

In order to have the default cluster-level Argo CD instance that is installed by the OpenShift GitOps operator create more instances of Argo CD (for example, for dev teams), the new namespacde has to have the `argocd.argoproj.io/managed-by:` label set with a value of `openshift-gitops`.  Without this label, the default Argo CD can't create an `argocd` instance in that namespace.

The `openshift-gitops` default Argo CD instancde can create this new namespace with the correct label just fine.  The issue occurs when the new `argocd` instance is created.

As soon as the OpenShift GitOps operator starts to create the new Argo CD instance (for example, in a namespace called `developer-argocd`), the `managed-by` label gets changed to the name of this namespace (in this case, `developer-argocd`).

This will now cause an infinite "OutOfSync" loop for the Argo CD "Appliation" that is in the default Argo CD instance, as it will continually try to reset the `managed-by` label value to `openshift-gitops`, but the OpenShift GitOps operator will also continualy try to reset that same label value to `developer-argocd`.

## How To Reproduce This Issue

Start with a cluster that has the OpenShift GitOps (v1.2) operator installed.

Run the following command:
```
oc apply -k https://github.com/pittar-sandbox/gitops-1247/app
```

This will create a new Argo CD `Application` in the `openshift-gitops` namespace that is associated with the `default` project.  Argo CD will then:

1. Create a new namespace (`gitops-1247-argocd`) with the `managed-by` label set to `openshift-gitops`
2. Create a new instance of Argo CD in this namespace.

After a moment, you will notice that this Argo CD application in the main Argo CD instance deployed by OpenShift GitOps is continually "OutOfSync" as it fights with the operator to set the `managed-by` label value correctly.