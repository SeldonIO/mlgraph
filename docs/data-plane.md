# Data Plane for ML Graph

ML Graph adds two specific operations: Route and Merge. Users can provide custom implementations for these.

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


