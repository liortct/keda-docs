+++
title = "Huawei Cloudeye"
layout = "scaler"
availability = "v1.1+"
maintainer = "Community"
description = "Scale applications based on a Huawei Cloudeye."
go_file = "huawei_cloudeye_scaler"
+++

### Trigger Specification

This specification describes the `huawei-cloudeye` trigger that scales based on a Huawei Cloudeye.

```yaml
triggers:
- type: huawei-cloudeye
  metadata:
    namespace: SYS.ELB
    metricName: mb_l7_qps
    dimensionName: lbaas_instance_id
    dimensionValue: 5e052238-0346-xxb0-86ea-92d9f33e29d2
    targetMetricValue: "100"
    minMetricValue: "1"
```

**Parameter list:**

- `namespace` - Namespace of the metric.The format is service.item; service and item must be strings, must start with a letter, can only contain 0-9/a-z/A-Z/_, the total length of service.item is 3, the maximum is 32.
- `metricName` - Name of the metric.
- `dimensionName` - Dimension name of the metric.
- `dimensionValue` - Dimension value of the metric.
- `targetMetricValue` - Target value for your metric.
- `minMetricValue` - Min value for your metric. If the actual value of the metric you get from cloudeye is less than the minimum value, then the scaler is not active.
- `metricCollectionTime` - Collection time of the metric. Equivalent to the earliest start time of the end time. (Default: `300`, Optional)
- `metricFilter` - Aggregation method of the metric. (Values: `max`, `min`, `average`, `sum`, `variance`, Default: `average`, Optional)
- `metricPeriod` - Granularity of the metric. (Default: `300`, Optional)

### Authentication Parameters

You can use `TriggerAuthentication` CRD to configure the authenticate by providing a set of IAM credentials.

**Credential based authentication:**

- `IdentityEndpoint` - Identity Endpoint.
- `ProjectID` - Project ID.
- `DomainID` - Id of domain.
- `Domain` - Domain.
- `Region` - Region.
- `Cloud` - Cloud name. (Default: `myhuaweicloud.com`, Optional)
- `AccessKey` - Id of the user.
- `SecretKey` - Access key for the user to authenticate with.

The user will need access to read data from Huawei Cloudeye.

### Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: keda-huawei-secrets 
  namespace: keda-test
data:
  IdentityEndpoint: <IdentityEndpoint>
  ProjectID: <ProjectID>
  DomainID: <DomainID>
  Region: <Region>
  Domain: <Domain>
  AccessKey: <AccessKey>
  SecretKey: <SecretKey>
--- 
apiVersion: keda.k8s.io/v1alpha1
kind: TriggerAuthentication
metadata:
  name: keda-trigger-auth-huawei-credential
  namespace: keda-test
spec:
  secretTargetRef:
  - parameter: IdentityEndpoint      # Required.
    name: keda-huawei-secrets        # Required.
    key: IdentityEndpoint            # Required.
  - parameter: ProjectID             # Required.
    name: keda-huawei-secrets        # Required.
    key: ProjectID                   # Required.
  - parameter: DomainID              # Required.
    name: keda-huawei-secrets        # Required.
    key: DomainID                    # Required.
  - parameter: Region                # Required.
    name: keda-huawei-secrets        # Required.
    key: Region                      # Required.
  - parameter: Domain                # Required.
    name: keda-huawei-secrets        # Required.
    key: Domain                      # Required.
  - parameter: AccessKey             # Required.
    name: keda-huawei-secrets        # Required.
    key: AccessKey                   # Required.
  - parameter: SecretKey             # Required.
    name: keda-huawei-secrets        # Required.
    key: SecretKey                   # Required.
---
apiVersion: keda.k8s.io/v1alpha1
kind: ScaledObject
metadata:
  name: huawei-cloudeye-scaledobject
  namespace: keda-test
  labels:
    test: nginx-deployment
spec:
  scaleTargetRef:
    deploymentName: nginx-deployment
  maxReplicaCount: 5
  minReplicaCount: 2
  triggers:
  - type: huawei-cloudeye
    metadata:
      namespace: SYS.ELB
      dimensionName: lbaas_instance_id
      dimensionValue: 5e052238-0346-47b0-xxea-92d9f33e29d2
      metricName: mb_l7_qps
      targetMetricValue: "100"
      minMetricValue: "1"
    authenticationRef:
      name: keda-trigger-auth-huawei-credential
```
