# kustomize-openapi-sample

## steps to reproduce

1. apply the CRD
    ```bash
    kubectl apply -f custom-pod-crd.yaml
    ```
2. execute kustomize locally
    ```bash
   kustomize build ./deploy
    ```
   The output correctly shows the merged environment variables like this:
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
3. create a flux-kustomization referencing the deploy-directory
    ```bash
    flux create source git repo \
      --url=https://github.com/stoetti/kustomize-openapi-sample.git \
      --interval=10m \
      --branch=main
    flux create kustomization deploy \
      --source=repo \
      --path="./deploy" \
      --prune=true \
      --validation=client \
      --interval=10m
    ```
