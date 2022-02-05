# Pod 관리

## DaemonSet

- ReplicaSet은 무작위 노드에 pod 생성
- DaemonSet은 각 노드에 하나의 pod만 생성

```yaml
apiVersion: v1
kind: DaemonSet
metadata:
  name: http-go-ds
spec:
  selector:
    matchLabels:
      app: http-go
  template:
    metadata:
      labels:
        app: http-go
    spec:
    tolerations:
      - key: node-role.kubernetes.io/master # Master node에도 daemonset을 설치하는 경우
        effect: NoSchedule
    containers:
      - name: http-go
      image: gasbugs/http-go
```

## Static pod

- kubelet이 직접 실행하는 pod, 각각의 노드에서 kublet에 의해 실행.
- 즉, 직접 kubectl을 통해 호출하는 것이 아닌 자동으로 실행됨.
- 기본 경로는 /etc/kubernetes/manifests
- 각 컴퐅넌트의 세부 사항 설정을 위해 여기 있는 파일을 수정하면, 자동으로 업데이트되 pod를 재구성.
- 기본적인 pod의 작성요령과 동일하고, 기본 경로로 설정된 위치에 yaml 파일을 옮겨주면 kubelet에 의해 자동으로 실행됨.