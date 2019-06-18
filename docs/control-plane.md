# Control Plane

## MLGraph
An MLGraph joins together various atomic ML components into a graph.

## Specifying a Service
### Resource Definition
This top level resource definition is shared by all Kubernetes Custom Resources.

| Field       |  Value      | Description |
| ----------- | ----------- | ----------- |
| kind       | MLGraph                     | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#types-kinds) |
| apiVersion | mlgraph.org/v1alpha1 | [Read the Docs](https://kubernetes.io/docs/reference/using-api/api-overview/#api-versioning) |
| metadata   | Metadata         | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#metadata) |
| spec       | [Spec](#Spec)                 | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) |
| status     | [Status](#Status)             | [Read the Docs](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) |


### Spec
The Spec section of the resource definition encapsulates the desired state of the user's model serving resources. Changes made to the spec will be enacted upon the underlying servers in an eventually consistent manner. This infrastructure-as-code paradigm enables the coveted GitOps design pattern, where configurations are checked into git and may be reverted and applied to restore previous configurations. All fields are mutable and may be applied idempotently.

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| inline | List<[InlineSpec](#InlineSpec)> | A list of inline specs for implementing the nodes. (optional) |
| dag               | [DAGSpec](#DAGSpec) | Directed Acyclic Graph definition |
| analysis | [AnalysisSpec](#AnalysisSpec) | Configure analysis (optional) |

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
| name | *string | The name of the container in the inline spec that implements this node |
| ref | *[Reference](#ReferenceSpec)    | Reference to existing kubernetes resource |
| inline | *[InlineSpec](#InlineSpec) | inline implementation spec |


### ReferenceSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| kfservice | *string    | KFService name |
| custom | *corev1.ObjectReference    | Reference to an object that will be used to find target endpoint. Object must adhere to KNative Addressable pattern. |
| uri | *string | Reference to an endpoint |


### InlineSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| kfspec | [KFServiceSpec](https://github.com/kubeflow/kfserving/blob/master/docs/control-plane.md#spec) | KFServiceSpec |
| podspec | v1.PodSpec | A v1.PodSpec definiton |
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


### AnalysisSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| explanation | [ExplanationSpec](#ExplanationSpec) | model explanations |
| outliers | [OutlierSpec](#OutlierSpec) | request outlier monitoring |
| skew | [SkewSpec](#SkewSpec) | Concept drift/skew monitoring ] |
| bias | [BiasSpec](#BiasSpec) | Bias monitoring ] |
| reporting | [ReportingSpec](#ReportingSpec) | How to report results |

### ExplanationSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| method | string | Explanation method auto or method name |

| activation | float | What percetage of prediction calls will get explanations (default 0%) |
| reporting | [ReportingSpec](#ReportingSpec) | Where to send explanation results |

### OutlierSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| method | string | Outlier method auto or method name |
| activation | float | What percetage of prediction calls will get outlier detection (default 100%) |
| reporting | [ReportingSpec](#ReportingSpec) | Where to sent outlier alerts |

### SkewSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| method | string | Skew method auto or method name |
| activation | float | What percetage of reward calls will update skew component (default 100%) |
| reporting | [ReportingSpec](#ReportingSpec) | Where to send skew alerts |

### BiasSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| method | string | bias method auto or method name (defaults auto) |
| activation | float | What percetage of prediction calls will update bias component (default 100%) |
| reporting | [ReportingSpec](#ReportingSpec) | Where to send bias alerts |




## ReportingSpec

| Field       | Value       | Description |
| ----------- | ----------- | ----------- |
| ref  | *corev1.ObjectReference    | Reference to an object that will be used to send result. Should be addressable in KNative sense. | 

## Execution

* Nodes with no dependencies will be treated as input nodes
* Nodes which have no nodes that depend on them will be treated as output nodes

For each node:
  1. Apply merge if specified or pass traffic as received
  1. Run implementation if specified
  1. Apply route if specified or pass traffic to all children

