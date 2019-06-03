# Seldon Implementation

Add a new `dag` field that will use the [DAGSpec](control-plane.md#DAGSpec). This will be an alternative to the current `graph` field.

## Differences

### No Parameters

The existing SeldonDeployment graphSpec has [parameters](https://github.com/SeldonIO/seldon-core/blob/686d53e164bc66f0f1eee7819654bb337d509bea/proto/seldon_deployment.proto#L131-L145). We should suggest user container's use configMaps to provide parameters to their components.

### No Component Type

The current control plane has three sections:

  1. Merge
  2. Call main implementation
  3. Route

At present it is assumed the data plane path is passed to each component. So if the user calls:

```
/api/v0.1/predictions
```

Then this will be passed to each component at each step.



