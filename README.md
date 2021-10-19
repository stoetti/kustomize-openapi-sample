# kustomize-openapi-sample

This is a sample repo for an [issue](https://github.com/fluxcd/kustomize-controller/issues/436) created in the [kustomize-controller](https://github.com/fluxcd/kustomize-controller) of fluxcd.

## steps to reproduce

1. local kustomize version should be 4.3.0 assuming the cluster is running the current kustomize-controller version v0.14.1
   ```bash
   kustomize version
   kubectl get deployment -n=flux-system kustomize-controller -o jsonpath="{.spec.template.spec.containers[*].image}"
   ```
3. execute kustomize locally
    ```bash
   kustomize build ./deploy
    ```
   The output for the CustomPod-resource correctly shows the merged environment variables like this:
    ```yaml
    apiVersion: stoetti.github.com/v1
    kind: CustomPod
    metadata:
      name: test
    spec:
      podTemplate:
        spec:
          containers:
          - env:
            - name: env2
              value: value2
            - name: env1
              value: value1
            name: cont1
    ```
4. create a flux-kustomization referencing the deploy-directory
    ```bash
    flux create source git repo \
      --url=https://github.com/stoetti/kustomize-openapi-sample.git \
      --interval=10m \
      --branch=main
    flux create kustomization deploy \
      --source=repo \
      --path="./deploy" \
      --prune=true \
      --interval=10m
    ```
5. after the kustomization was reconciled load the final resource from the cluster
   ```bash
   kubectl get custompod test -o=yaml
   ```
   The output differs from the local output from step 2:
   ```yaml
   apiVersion: stoetti.github.com/v1
   kind: CustomPod
   metadata:
     labels:
       kustomize.toolkit.fluxcd.io/name: deploy
       kustomize.toolkit.fluxcd.io/namespace: flux-system
     name: test
     namespace: default
   spec:
     podTemplate:
       spec:
         containers:
         - env:
           - name: env2
             value: value2
           name: cont1
  ```
