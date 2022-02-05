# 수동 스케쥴링

- 원하는 pod를 원하는 노드에.
- 특수한 환경의 경우 특정 노드에서 실행되도록 pod를 제한
- 더 많은 제어가 필요한 몇가지 케이스
    - SSD가 있는 노드에서 pod가 실행되야 하는 경우
    - 특정 시스템을 위해 GPU 서비스가 필요한 경우
    - 서비스 성능을 극대화하기 위해 하나의 노드에 필요한 pod를 모두 배치해야 되는 경우

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-go
spec:
  containers:
    - name: http-go
      image: gasbugs/http-go
  noedeName: work1  # Node name을 지정!
```

## node selector를 활용

- 특정 하드웨어를 가진 노드에 pod를 실행하고자 하는 경우.
- 두 개 이상의 관리 영역을 가질 경우 편리.
- gpu, ssd 등.

```shell
kubectl label node <node_name> gpu=true
# node 자체에 role, labeling
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-go
spec:
  containers:
    - name: http-go
      image: gasbugs/http-go
  nodeSelector:
    gpu: "true"
```