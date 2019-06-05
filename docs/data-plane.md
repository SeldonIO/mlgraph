# Data Plane for ML Graph

ML Graph adds two specific operations: Route and Merge. Users can provide custom implementations for these.

# Path Spec and Payloads

There will need to be path to make prediction and in future calls such as Feedback needed for reinforcement learning.

## API Path Specs

For [Tensorflow Http](https://www.tensorflow.org/tfx/serving/api_rest) the `predict` API is of the form:

```
POST http://host:port/v1/models/${MODEL_NAME}[/versions/${MODEL_VERSION}]:predict
```

For Seldon the predict API is of the form:

```
/api/v0.1/predictions
```

For a graph the need to specify a particular model does not make sense as a graph may contain many models therefore a path spec more like that of Seldon would seem to make sense.

## Payloads

Payloads could be either Tensorflow or Seldon as long as all nodes in the graph conform to this.

# Route

A user should provide a server that:

 * Accepts calls for their protocol,e.g., Tensorflow serving, or Seldon
 * The requests will contain a header `mlgraph/routable-nodes` with the names of valid nodes that this request can be routed to.
 * Returns a response with Header `mlgraph/route` with value the list of child node names the request should be routed to.



# Merge

A user should provide a server that:

 * Receives requests and eventually returns an aggregated request.
 * Upon receving the request if the merge should not be carried out then an empty reply should be returned.
 * Each request will contain a header `mlgraph/pending-requests` which will provide an upper bound on the number of requests that may still arrive for this transaction. It is an upper bound as other requests in this transaction may not have passed through all routing elements in the graph so it is uncertain if all routes to this node will actually be taken.


