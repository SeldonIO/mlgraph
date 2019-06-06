# MLGraph

MLGraph defines a graph of machine learning components. The goal is to provide a simple machine learning focused 
specification for defining:

  * A/B Tests and multi-model experimentation
  * Advanced optimization with Multi-Armed Bandits
  * Model Ensembles
  
## Definitions
 
  * [Control plane](control-plane.md)
  * [Data plane](data-plane.md)

## Implementations
 
 There may be several implementations:
 
   * A synchronous implemtation provided by a graph orchestrator
       * A [seldon implementation](seldon-impl.md)
   * A [KNative implementation](knative-impl.md) for asynchronous cases 