# Setting Up Red Hat Advanced Cluster Management with GitOps for Applications

## Using Policy Generator to Create a Policy

1. Download PolicyGenerator from [PolicyGenerator Releases](https://github.com/open-cluster-management-io/policy-generator-plugin/releases)
2. _(Optional)_ Add PolicyGenerator to your PATH
3. Create a `Subscription` manifest
```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-gitops-operator
  namespace: openshift-operators
spec:
  channel: stable
  name: openshift-gitops-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```
4. Create a `PolicyGenerator` configuration file
```
apiVersion: policy.open-cluster-management.io/v1
kind: PolicyGenerator
metadata:
  name: install-openshift-gitops
policyDefaults:
  namespace: policies
  placement:
    clusterSelectors:
      vendor: "OpenShift"
  remediationAction: enforce
policies:
  - name: install-openshift-gitops
    manifests:
      - path: openshift-gitops-subscription.yaml
```
5. Create a `kustomization.yaml` file
```
generators:
  - policy-generator-config.yaml
```
6. Run PolicyGenerator
```
$ /path/to/PolicyGenerator policy-generator-config.yaml
```
```yaml
---
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
    annotations:
        policy.open-cluster-management.io/categories: CM Configuration Management
        policy.open-cluster-management.io/controls: CM-2 Baseline Configuration
        policy.open-cluster-management.io/description: ""
        policy.open-cluster-management.io/standards: NIST SP 800-53
    name: install-openshift-gitops
    namespace: policies
spec:
    disabled: false
    policy-templates:
        - objectDefinition:
            apiVersion: policy.open-cluster-management.io/v1
            kind: ConfigurationPolicy
            metadata:
                name: install-openshift-gitops
            spec:
                object-templates:
                    - complianceType: musthave
                      objectDefinition:
                        apiVersion: operators.coreos.com/v1alpha1
                        kind: Subscription
                        metadata:
                            name: openshift-gitops-operator
                            namespace: openshift-operators
                        spec:
                            channel: latest
                            name: openshift-gitops-operator
                            source: redhat-operators
                            sourceNamespace: openshift-marketplace
                remediationAction: enforce
                severity: low
    remediationAction: enforce
---
apiVersion: apps.open-cluster-management.io/v1
kind: PlacementRule
metadata:
    name: placement-install-openshift-gitops
    namespace: policies
spec:
    clusterSelector:
        matchExpressions:
            - key: vendor
              operator: In
              values:
                - OpenShift
---
apiVersion: policy.open-cluster-management.io/v1
kind: PlacementBinding
metadata:
    name: binding-install-openshift-gitops
    namespace: policies
placementRef:
    apiGroup: apps.open-cluster-management.io
    kind: PlacementRule
    name: placement-install-openshift-gitops
subjects:
    - apiGroup: policy.open-cluster-management.io
      kind: Policy
      name: install-openshift-gitops
```

## Links
1. [Policy Generator Source](https://github.com/stolostron/policy-generator-plugin)
