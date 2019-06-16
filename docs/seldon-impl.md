# Seldon Implementation

The MLGraph spec will replace the SeldonDeployment CRD as an evolution for non KNative customers who want more customization.

## Examples

Examples for existing SeldonDeployments and their converted MLGraph specs are shown below.


### Single Model

```YAML
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: seldon-model
spec:
  name: test-deployment
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - image: seldonio/mock_classifier:1.0
          name: classifier
    graph:
      children: []
      endpoint:
        type: REST
      name: classifier
      type: MODEL
    name: example
    replicas: 1
```

This would be converted to:

```YAML
apiVersion: serving.mlspec.org/v1alpha2
kind: MLGraph
metadata:
  name: test-deployment
spec:
  inline:
    - podspec:
        containers:
	  - name: classifier
            image: seldonio/mock_classifier:0.1
      replicas: 1              
  dag:
      name: classifier
```

### Complex Pipeline

An example derived fron the [Intel OpenVINO example pipeline](https://docs.seldon.io/projects/seldon-core/en/latest/examples/openvino_ensemble.html).

```YAML
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: openvino-model
  namespace: seldon
spec:
  name: openvino
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: imagenet-itransformer
          image: seldonio/openvino-demo-transformer:0.1
        - name: imagenet-otransformer
          image: seldonio/openvino-demo-transformer:0.1
        - name: imagenet-combiner
          image: seldonio/openvino-demo-combiner:0.1
        - name: prediction1
          image: seldonio/openvino-demo-prediction:0.2
        - name: prediction2
          image: seldonio/openvino-demo-prediction:0.2
    graph:
      name: imagenet-otransformer
      endpoint:
        type: GRPC
      type: OUTPUT_TRANSFORMER
      children:
      - name: imagenet-itransformer
        endpoint:
          type: GRPC
        type: TRANSFORMER
        children:
        - name: imagenet-combiner
          endpoint:
            type: GRPC
          type: COMBINER
          children:
          - name: prediction1
            endpoint:
              type: GRPC
            type: MODEL
            children: []
          - name: prediction2
            endpoint:
              type: GRPC
            type: MODEL
            children: []
    name: openvino
    replicas: 1
```

The above would be converted to:

```YAML
apiVersion: serving.mlspec.org/v1alpha2
kind: MLGraph
metadata:
  name: inline-example
  namespace: test
spec:
  inline:
    - podspec:
        containers:
          - name: imagenet-itransformer
            image: seldonio/openvino-demo-transformer:0.1
          - name: imagenet-otransformer
            image: seldonio/openvino-demo-transformer:0.1
          - name: prediction1
            image: seldonio/openvino-demo-prediction:0.2
          - name: prediction2
            image: seldonio/openvino-demo-prediction:0.2
          - name: imagenet-combiner
            image: seldonio/openvino-demo-combiner:0.1
      replicas: 1              
  dag:
    - name: imagenet-itransformer
    - name: prediction1
      dependencies: [imagenet-itransformer]
    - name: prediction2
      dependencies: [imagenet-itransformer]
    - name: imagenet-combiner
      dependencies: [prediction1, prediction2]
      merge:
          ensemble:
            implementation:
              name: imagenet-combiner
    - name: imagenet-otransformer
      dependencies: [imagenet-combiner]        
```

## Differences to SeldonDeployment

### No Parameters

The existing SeldonDeployment graphSpec has [parameters](https://github.com/SeldonIO/seldon-core/blob/686d53e164bc66f0f1eee7819654bb337d509bea/proto/seldon_deployment.proto#L131-L145). We should suggest user container's use configMaps or env variables to provide custom parameters to their components.

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

### No REST or gRPC type

It is assumed if gRPC then all components will provide a gRPC endpoint and the same for REST.




