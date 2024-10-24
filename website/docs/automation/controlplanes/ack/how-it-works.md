---
title: "How does ACK work?"
sidebar_position: 5
---



:::info
kubectl also contains useful `-oyaml` and `-ojson` flags which extract either the full YAML or JSON manifests of the deployment definition instead of the formatted output.
:::

This controller will watch for Kubernetes custom resources for DynamoDB such as `dynamodb.services.k8s.aws.Table` and will make API calls to the DynamoDB endpoint based on the configuration in these resources created. As resources are created, the controller will feed back status updates to the custom resources in the `Status` fields. For more information about the spec of the manifest, click [here](https://aws-controllers-k8s.github.io/community/reference/).

If you'd like to dive deeper into the mechanics of what objects and API calls the controller listens for, run:

```bash
$ kubectl get crd
```
