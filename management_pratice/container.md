# 컨테이너 관련

## One pod Multi container

- 주로 로깅에서 많이 사용됨.
- 하나의 container를 보조하는 역할의 container들을 배치하는 전략.(사이드카)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-redis
  name: nginx-redis
spec:
  replicas:
    1  selector:
      matchLabels:
        app:
          nginx-redis  strategy: { }
  template:
    metadata:
      creationTimestamp:
        null      labels:
          app: nginx-redis
    spec:
      containers:
        - image: nginx
          name: nginx
          ports:
            - containerPort: 80
          resources: { }
        - image: redis
          name:
            redisstatus: { }
```

## Init container

- 포드 컨테이너 실행 전 초기화 역할을 하는 컨테이너 실행
- 완전히 초기화가 된 이후에야 main container가 실행
- 즉, main container가 어떤 dependency가 있는 경우 사용함.
- Init container가 실패하면, 성공할 떄 까지 pod를 재시작.(Policy에 따라 다름)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: [ 'sh', '-c', 'echo The app is running! && sleep 3600' ]
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command: [ 'sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done" ]
    - name: init-mydb
      image: busybox:1.28
      command: [ 'sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done" ]
```

- 위 yaml을 실행하고 관찰하면 init container로 설정된 두개가 완료 되기 전까지는 pod이 올라가지 않음.

```bash
kubectl -f init-container.yaml

# NAME        READY   STATUS     RESTARTS   AGE
# myapp-pod   0/1     Init:0/2   0          19s
```

- 따라서 init container에서 설정해 주었던 두 개의 서비스를 동작시켜줘야 함.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9377
```

```yaml
kubectl create -f svc-for-initcontainer.yaml

# NAME        READY   STATUS     RESTARTS   AGE
# myapp-pod   0/1     Init:0/2   0          5m58s
# myapp-pod   0/1     Init:1/2   0          6m16s
# myapp-pod   0/1     PodInitializing   0          6m17s
# myapp-pod   1/1     Running           0          6m18s
```

## Reousrce Allocation

### In Container

- 컨테이너의 리소스를 요청 또는 제한(requests and limits)
- cpu는 코어 단위(m, millicpu), memory는 바이트 단위.
- request의 경우 최소 요구조건인데, 주의할 점은 request가 요구한 리소스가 없을 경우 node에 올라가지 않기 때문에, 너무 과한 request는 피해야 함.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp:
    null  labels:
      app: nginx-resource
  name: nginx-resource
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-resource
  strategy: { }
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-resource
    spec:
      containers:
        - image: nginx
          name: nginx
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "1m"
              memory: "200Mi"
            limits:
              cpu: "2m"
              memory: "400Mi"
```

### In Namespace

- 포드나 컨테이너에서 사용하는 컴퓨팅 리소스 제한
- PersistentVolumeCalim 당 최소 최대 스토리지 사용량 제한
- 리소스 requests와 limits 사이의 비율 적용
- 컴퓨팅 리소스에 대한 default requests and limits을 설정.

```bash
kubectl create namespace limitrange-demo # name space 생성
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: limitrange-demo
spec:
  limits:
    - default: # default limit
        memory: 512Mi
      defaultRequest:
        memory: 256Mi
      type: Container # 컨테이너 당 제한
  # type: Pod # pod 당 제한
```

### Quota

- namespace 내에서 사용하는 리소스의 총량을 제한

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: limit-quota
  namespace: limitrange-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```