# Chaos Engineering Examples

## Example: CPU Stress Test

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: cpu-stress
spec:
  mode: one
  selector:
    labelSelectors:
      app: my-service
  stressors:
    cpu:
      workers: 2
      load: 50
  duration: '30s'
```
