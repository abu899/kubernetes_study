# 컨테이너 환경 변수 전달 방법

- `ConfigMap` 과 `Secrets`를 사용한 실습
- 환경 변수가 자주 변경되는 상황에 적용하기 적절함.

## ConfigMap

```yaml
env:
  - name: env_value
  value: "Environment value"

env:
  - name: env_value
  valueFrom:
    configMapKeyRef: confgimap-name #ConfigMap 이용

env:
  - name: env_value
  valueFrom:
    secretKeyRef: secret-name # secrets 이용		
```

### 일반적인 yaml 작성

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
    - name: envar-demo-container
      image: gcr.io/google-samples/node-hello:1.0
      env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_FAREWELL
          value: "Such a sweet sorrow"
```

### ConfigMap 사용

```bash
#kubectl create configmap <map-name> <data-source> 
# 주로 file로 만들어서 import

echo -n 1234 > test # key-value 형태 text 생성
kubectl create configmap test-cm --from-file=test # configMap 생성
kubectl get configmap test-cm -o yaml # configmap yaml로 확인
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-config-map
  labels:
    purpose: demonstrate-envars
spec:
  containers:
    - name: envar-demo-container
      image: gcr.io/google-samples/node-hello:1.0
      env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_FAREWELL
          valueFrom:
      configMapKeyRef:
        name: test-cm # configMap name
        key: test # test: 1234
```

### ConfigMap을 활용한 디렉토리 마운트

```bash
kubectl create -f https://kubernetes.io/examples/configmap/configmap-multikeys.yaml
# kubernetes 예제 사용
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volumes-config-map
  labels:
    purpose: demonstrate-envars
spec:
  containers:
    - name: envar-demo-container
      image: gcr.io/google-samples/node-hello:1.0
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
```

- web 설정파일 같은 걸 담아두고 사용하는 경우 외부에서 handling 가능
- 앞서 환경변수(env)를 통해 설정하는 경우 pod를 재시작해야 환경변수가 적용 됨.
- 하지만 volume mount를 사용하면, 1분마다 refresh되면서 바로 데이터 확인 가능.

## Secret

- secret은 encoding 된 데이터를 저장.
- 비밀번호, OAuth 토큰 및 ssh키와 같은 민감한 정보

```bash
echo -n 'admin' > username
echo -n 'passwd' > password
kubectl create secret generic db-user-pw --from-file=username --from-file=password
# base64 encoding된 데이터 생성

kubectl get secret db-user-pw -o yaml # 데이터 확인
```

### Mysql password with secret

```shell
echo -n 'passw0rd!0' > DB_Password
kubectl create secret generic db-secret --from-file=DB_Password
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-secret
spec:
  containers:
    - name: mysql
      image: mysql:5.6
      ports:
        - containerPort: 3306
      env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_Password
```