# Control Plane

## MLGraph
An MLGraph joins together various atomic ML components into a graph.

## Specifying a Service
### Resource Definition
This top level resource definition is shared by all Kubernetes Custom Resources.

| Field       |  Value      | Description |
| ----------- | ----------- | ----------- |
| kind       | MLGraph                     | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#types-kinds) |
| apiVersion | serving.mlpsepc.org/v1alpha1 | [Read the Docs](https://kubernetes.io/docs/reference/using-api/api-overview/#api-versioning) |
| metadata   | [Metadata](#Metadata)         | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#metadata) |
| spec       | [Spec](#Spec)                 | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) |
| status     | [Status](#Status)             | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) |

### Metadata
The Metadata section of the resource definition contains resource identification fields and specialized flags. Name and namespace are immutable and if changed, will result in the creation of a new resource alongside the old one. Label and annotations are mutable. Labels and annotations will be applied to all relevant subresources of the Service enabling users to apply the same grouping at the pod level and pass though any necessary annotations.

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| name        | String              | A name for your KFService. This will ultimately derive the internal and external URIs. |
| namespace   | String              | A namespace for your Service. This will ultimately derive the internal and external URIs. If missing, defaults to the namespace "default". |
| labels      | Map<String, String> | A set of key value pairs denoting arbitrary information (e.g integrate with MLOps Lifecycle systems or organize many Models). |
| annotations | Map<String, String> | A set of key value pairs used to enable specific features available in different kubernetes environments (e.g., Google Cloud ML Engine vs AzureML). |

### Spec
The Spec section of the resource definition encapsulates the desired state of the user's model serving resources. Changes made to the spec will be enacted upon the underlying servers in an eventually consistent manner. This infrastructure-as-code paradigm enables the coveted GitOps design pattern, where configurations are checked into git and may be reverted and applied to restore previous configurations. All fields are mutable and may be applied idempotently.

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| custom | List<[CustomSpec](#CustomSpec)> | A list of custom specs for implementing the nodes. |
| dag               | [DAGSpec](#DAGSpec) | The default traffic route serving a ModelSpec. |

### Status

TBD

### DAGSpec


| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| name | string   | A name for this graph node |
| dependencies | List<string> | A list of DAG names that this node depends on. (optional) |
| implementation | [ImplementationSpec](#ImplementationSpec) | The implementation for this node (optional) |
| route | [RouteSpec](#RouteSpec) | Optional spec for routing traffic to forward nodes. Default is to send all traffic to all children nodes (optional) |
| merge | [MergeSpec](#MergeSpec) | Optional spec for merging traffic from dependencies. Default is to pass traffic as it arrives. (optional) |

Notes:

  * A DAGSpec with no implementation is assumed to have a container in the custom spec with the same name.

### ImplementationSpec

Only one field must be defined.

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| name | string | The name of the container in the custom spec that implements this node |
| kfservice | string   | The name of a kubeflow kf service |
| svc | string | The FQDN name of a kubernetes service |

Notes:

  * Do we need to specify `modelName` for kfservice?

### CustomSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| spec | v1.PodSpec | A v1.PodSpec definiton |
| replicas | *int | Number of replicas (optional) |

### RouteSpec

Must be zero or 1 field. If no fields then the default is to send traffic to all children nodes.

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| split  | SplitSpec | Split traffic
| mab | [MABSpec](#MABSpec)  |  Apply Multi Armed Bandit Solver |
| implementation | [ImplementationSpec](#ImplmentationSpec) | Call custom function |


### MABSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| egreedy| Float | Apply egreedy algorithm with given epsilon value |


### SplitSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| traffic  | Map<String,Int> | Optional Map of node name to % of traffic to send to it |


### MergeSpec

Must contain zero or one field. If no fields then the default is to pass all traffic thru as it appears.

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| ensemble | [EnsembleSpec](#EnsembleSpec) | Ensemble traffic |
| implementation | [ImplementationSpec](#ImplementationSpec) | Call custom function |

### EnsembleSpec
If no fields specified then merge requests and average

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| weights  | Map<String,Float> |  Weight to apply to each traffic from each dependencies when merging |


## Execution

* Nodes with no dependencies will be treated as input nodes
* Nodes which have no nodes that depend on them will be treated as output nodes

For each node:
  1. Apply merge if specified or pass traffic as received
  1. Run implementation if specified
  1. Apply route if specified or pass traffic to all children

## Examples


### 3-Way Experiment to KFServices

Split traffic evenly between 3 models.

```YAML
apiVersion: serving.mlspec.org/v1alpha2
kind: MLGraph
metadata:
  name: experiment
spec:
  dag:
    - name: a
      route:
        split:
    - name: b
      dependencies: [a]
      implementation:
        kfservice: kfb
    - name: c
      dependencies: [a]
      implementation:
        kfservice: kfc
    - name: d
      dependencies: [a]
      implementation:
        kfservice: kfd
```

### E-Greedy Multi_Armed Bandit over 2 KFServices

```YAML
apiVersion: serving.mlspec.org/v1alpha2
kind: MLGraph
metadata:
  name: egreedy
spec:
  dag:
    - name: a
      route:
        mab:
          egreedy: 0.1
    - name: b
      dependencies: [a]
      implementation:
        kfservice: kfb
    - name: c
      dependencies: [a]
      implementation:
        kfservice: kfc
    - name: d
      dependencies: [a]
      implementation:
        kfservice: kfd
```

### Simple Ensembler for 3 KFServices

```YAML
apiVersion: serving.mlspec.org/v1alpha2
kind: MLGraph
metadata:
  name: ensemble
spec:
  dag:
    - name: a
      implementation:
        kfservice: kfa
    - name: b
      implementation:
        kfservice: kfb
    - name: c
      implementation:
        kfservice: kfc
    - name: d
      dependencies: [a, b, c]
      merge:
        ensemble:
```

### Single model with custom implementation

```YAML
apiVersion: serving.mlspec.org/v1alpha2
kind: MLGraph
metadata:
  name: test-deployment
spec:
  custom:
    - spec:
        containers:
          - name: classifier
            image: seldonio/mock_classifier:0.1
      replicas: 1              
  dag:
      name: classifier
```

### Complex Pipeline with custom implementation

```YAML
apiVersion: serving.mlspec.org/v1alpha2
kind: MLGraph
metadata:
  name: test-deployment
spec:
  custom:
    - spec:
        containers:
          - name: imagenet-itransformer
            image: seldonio/openvino-demo-transformer:0.1
          - name: imagenet-otransformer
            image: seldonio/openvino-demo-transformer:0.1
          - name: prediction1
            image: seldonio/openvino-demo-prediction:0.2
          - name: prediction2
            image: seldonio/openvino-demo-prediction:0.2
      replicas: 1              
  dag:
    - name: imagenet-itransformer
    - name: prediction1
      dependencies: [imagenet-itransformer]
    - name: prediction2
      dependencies: [imagenet-itransformer]
    - name: imagenet-combiner
      dependencies: [prediction1, prediction2]
      merge:
          ensemble:
            implementation:
              name: imagenet-combiner
    - name: imagenet-otransformer
      dependencies: [imagenet-combiner]        
```

