
## How to run

```bash
kubectl apply -f operator.yaml
kubectl apply -f cluster.yaml
```

## Customize

To tweak options in the cluster crd, edit the `cluster.yaml` file.

Please check that all options in cluster.yaml are compatible with your cluster. (eg placement)
