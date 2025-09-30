<!--
SPDX-FileCopyrightText: Red Hat

SPDX-License-Identifier: Apache-2.0
-->

# GitOps repository layout and setup

This guide describes the Git repository content required by the O‑Cloud Manager to deploy and configure a cluster. We recommend organizing the repo as follows. For concrete examples, see the sample content under [git-setup](./samples/git-setup/).

Recommended layout:

```text
git-root/
  clustertemplates/
    hardwareprofiles/                   # HardwareProfile CRs for hardware provisioning
    hardwaretemplates/                  # HardwareTemplate CRs for hardware provisioning
    inventory/                          # BareMetalHost inventory for sites
    version_4.Y.Z/                      # Version matches the OCP version to be installed
      sno-ran-du/
        clusterinstance-defaults-*.yaml # Defaults in ConfigMap for Cluster installation
        policytemplates-defaults-*.yaml # Defaults in ConfigMap for ACM policy configuration
        pull-secret.yaml                # Pull secret used for spoke installation
        extra-manifest/                 # Extra manifests installed on day0
        ns.yaml                         # Namespace where the ClusterTemplate is created
        sno-ran-du-v4-Y-Z-*.yaml        # ClusterTemplate(s) for this OCP version
    version_4.Y.Z+1/                    # Optional upgrade content
  policytemplates/
    version_4.Y.Z/                      # Version matches the OCP version to be installed
      sno-ran-du/
        ns.yaml                         # Namespace where policies are created
        msc-binding.yaml                # ManagedClusterSetBinding for policy placement
        sno-ran-du-pg-v4-Y-Z-*.yaml     # ACM PolicyGenerator(s) for this OCP version
      source-crs/                       # ZTP source CRs; keep in sync with OCP version
    version_4.Y.Z+1/                    # Optional upgrade content
```

Notes

* Keep content versioned by OCP release using the `version_4.Y.Z/` folders so templates and policies align with the target OCP.
* The content of the `extra-manifest` directory should be copied over from the [cnf-features-deploy extra-manifest](https://github.com/openshift-kni/cnf-features-deploy/tree/master/ztp/source-crs/extra-manifest) repo 
or extracted from the [ztp-site-generate](https://catalog.redhat.com/software/containers/openshift4/ztp-site-generate-rhel8/6154c29fd2c7f84a4d2edca1) container.
  * ArgoCD will assemble all the extra-manifests into a ConfigMap.
* The content of the `source-crs` directory should be copied over from the [cnf-features-deploy source-crs](https://github.com/openshift-kni/cnf-features-deploy/tree/master/ztp/source-crs/) repo
or extracted from the [ztp-site-generate](https://catalog.redhat.com/software/containers/openshift4/ztp-site-generate-rhel8/6154c29fd2c7f84a4d2edca1) container.
  * The ACM PGs will reference the source CRs to generate the ACM Policies that will be applied on the spoke cluster(s).
* Make sure to bring over the `extra-manifest` and `source-crs` corresponding to the OCP release provided in the ClusterTemplate CR.

## Preparation of ArgoCD applications

Preparing the hub cluster for provisioning clusters through the O-Cloud Manager is similar to the [ZTP way](https://github.com/openshift-kni/cnf-features-deploy/tree/master/ztp/gitops-subscriptions/argocd#preparation-of-hub-cluster-for-ztp).

Distinctions:

* The source path for the ArgoCD Application [clusters](https://github.com/openshift-kni/cnf-features-deploy/blob/master/ztp/gitops-subscriptions/argocd/deployment/clusters-app.yaml) should point to the [clustertemplates](./clustertemplates/) directory. Additionally, configure `spec.ignoreDifferences` for the `BareMetalHost` kind to ignore the following fields:
```yaml
spec:
  ignoreDifferences:
  - group: metal3.io
    jsonPointers:
    - /spec/preprovisioningNetworkDataName
    - /spec/online
    kind: BareMetalHost
```
* The source path for the ArgoCD Application [policies](https://github.com/openshift-kni/cnf-features-deploy/blob/master/ztp/gitops-subscriptions/argocd/deployment/policies-app.yaml) should point to the [policytemplates](./policytemplates/) directory.

* The following CRDs should be added to the [AppProject](https://github.com/openshift-kni/cnf-features-deploy/blob/master/ztp/gitops-subscriptions/argocd/deployment/app-project.yaml), under `spec.clusterResourceWhitelist`:

```yaml
  - group: clcm.openshift.io
    kind: ClusterTemplate
  - group: clcm.openshift.io
    kind: HardwareProfile
  - group: clcm.openshift.io
    kind: HardwareTemplate
```

* TALM is not used for the initial cluster installation and configuration, but it's still needed for cluster upgrades.
