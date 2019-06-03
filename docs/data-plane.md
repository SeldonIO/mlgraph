# Data Plane for ML Graph

ML Graph adds two specific operations: Route and Merge. Users can provide custom implementations for these.

# Route

A user should provide a server that:

 * Extracts the child node names from the environment variable `MLGRAPH_CHILD_NODES`. This will contain the list of child node names. These are the acceptable names to return in routing calls. 
 * Accepts calls for their protocol,e.g., Tensorflow serving, or Seldon
 * Returns a response with Header `mlgraph-route` with value the list of child node names the request should be routed to.


# Merge

A user should provide a server that:

 * 