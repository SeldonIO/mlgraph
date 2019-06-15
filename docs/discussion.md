# Discussion Points

## Should the spec contain a default/canary top level?

As the spec already contains experimentation via the ability to split (with traffic percentage to sub graphs) it would seem unnecessary. 


## Is Gitops available?

The spec contains both 'reference' fields such as [ReferenceSpec](./control-plane.md#InlineSpec) that point to existing resources and [InlineSpec](./control-plane.md#InlineSpec) to allow the specification of the implementation and its creation. For true gitops on a single MLGraph is only available via the later otherwise it is dependent on the using the set of resources referred to in the former.


## Can MLGraph nodes refer to other MLGraphs

This is available via [ReferenceSpec.custom](./control-plane.md#ReferenceSpec) that can refer to any running Kubernetes resource. It could be added explicitly in the same way as [`ReferenceSpec.kfservice`](./control-plane.md#ReferenceSpec).