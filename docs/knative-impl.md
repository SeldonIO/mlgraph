# KNative Implementation for MLGraph Proposal

The is a work-in-progress proposal of how a KNative implementation could be created for MLGraph.

There are two core operations that need to be applied:

  * Fork: Route to 1 or more child nodes in the graph
  * Join: Join a set of responses from dependent nodes in the graph
  
 ## Fork

We need to be able to allow algorithmic control over requests passing through the graph. The fork operation will allow each request to be specified which 1 or more child nodes the request proceeds to. 

 This could be accomplished via a Channel with custom Subscription that applies meta data to the CloudEvent 
 for each request that allows a subsequent Broker with Filters for each path to be used.
 
![knative-fork](./knative-fork.png)
 
 
 ## Join
 
 TBD
 
 
 
 