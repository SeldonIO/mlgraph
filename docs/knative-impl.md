# KNative Implementation for MLGraph Proposal

The is a work-in-progress proposal of how a KNative implementation could be created for MLGraph.

There are two core operations that need to be applied:

  * Route: Route request to 1 or more subsequent nodes in the graph
  * Join: Join a set of responses from dependent nodes in the graph

Messages will be passed through the graph and be split and joined as defined by the specification.

## Route

We need to be able to allow algorithmic control over requests passing through the graph. The route operation will allow each request to be specified which of 0 or more child nodes the request proceeds to.

A KNative implementation is shown below:

![knative-route](./knative-route.png)

A Channel with a Subscription that applies meta data to the CloudEvent for each request that allows a subsequent Broker with Filters for each path to be used to direct requests. Headers would need to be added to the CloudEvent that will be matched by the Filters for each possible path to forward the request.

Notes:

 * An MLGraph component should provide to the router server the possible active paths that can be chosen
 
## Join
 
We need a component that will allow events that have passed through the previous graph nodes to be joined together. When a request is managed by the graph it will be given a unique ID. The role of the join component will be to join together events that have the same ID into a single event. A rough design is shown below:

![knative-join](./knative-join.png)

Notes:

 * An MLGraph component should provide to the join server whether the current event is the last for this request. This will allow the joiner server to produce a final joined result.


 
 
 