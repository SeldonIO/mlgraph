# Data Plane for ML Graph

ML Graph adds two specific operations: Route and Merge. Users can provide custom implementations for these.

# Route

A user should provide a server that:

 * Accepts calls for their protocol,e.g., Tensorflow serving, or Seldon
 * The requests will contain a header `mlgraph/routable-nodes` with the names of valid nodes that this request can be routed to.
 * Returns a response with Header `mlgraph/route` with value the list of child node names the request should be routed to.



# Merge

There seem to be two options. A higher level interface where the list of payloads is provided or a low level one where each individual payload is received as available. For the later it is up to the function to decide when and if it does the merge operation.

 1. Function receives an aggregated list of payloads to merge
 1. Function receives each individual payload as it arrives with a header that indicates an estimation of how many more payloads are expected.

