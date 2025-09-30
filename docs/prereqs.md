# Prerequisites

## Hub Cluster Requirements

The O-Cloud Manager runs on an OpenShift hub cluster managed by Red Hat Advanced Cluster Management (ACM). Before you deploy the operator, ensure the following prerequisites are satisfied.

**Platform**

- OpenShift Container Platform 4.20.0-rc3 or newer

**Required operators and add‑ons on the hub**

- Advanced Cluster Management (ACM) v2.14 or newer
- Red Hat OpenShift GitOps Operator
- SiteConfig Operator (enabled in ACM)

  Enable SiteConfig in ACM by running the following command:

  ```console
  oc patch multiclusterhubs.operator.open-cluster-management.io multiclusterhub -n <ACM_NAMESPACE> --type json --patch '[{"op": "add", "path":"/spec/overrides/components/-", "value": {"name":"siteconfig","enabled": true}}]'
  ```

**Storage**

- A default StorageClass that supports ReadWriteOnce (RWO)
- Create a 20 Gi PersistentVolume for the operator’s internal database

**Observability**

- ACM Observability enabled on the hub cluster (required for dashboards and metrics used by this project)
- Follow the official guide to enable it: [Red Hat ACM Observability - Enabling the Observability service](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#enabling-observability)

## mTLS

## OAuth Server
