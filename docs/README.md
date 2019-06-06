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

Possible implementations could be:

   * A [KNative implementation](knative-impl.md) 
   * A [Non Knative implementation](seldon-impl.md) as a transition from the current Seldon Core.

These two implementations could be a single operator which handles both.

