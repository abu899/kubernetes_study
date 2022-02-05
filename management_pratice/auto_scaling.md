# Auto scaling

- 포드 스케일링의 방법
    - HPA: pod 자체의 갯수를 증가시킴- Horizontal Pod Auto Scaler
    - VPA: pod가 사용 가능한 리소스를 늘리는 방법 - Vertical Pod Auto Sccaler
    - CA: 클러스터 자체를 늘리는 방법(노드 추가)

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: default
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1beta
    kind: Deployment
    name: myapp
  targetCPUUtilizationPercentage: 30
```

- 자세한 실습은 gcp에서 다시 한번 진행할 예정.