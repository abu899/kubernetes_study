# Helm CLI

## Test

- 테스트할 소스가 있는 root를 대상으로 문법적인 오류 검증을 먼저 진행.

```bash
helm lint ./helm-test
```

- 이후 템플릿에 벨류가 제대로 적용되는지 확인하는 작업 진행

```bash
helm template ./helm-test
# helm install --name helm-test-release --dry-run --debug ./helm-test
```

- helm template을 사용하면 tiller server에 연결없이 값이 채워지는지 확인 가능.
- tiller server에 접속하여 확인하는 경우 helm install을 사용하며, 서버에 같은 이름이나 같은 버젼이 있는지를 확인할 수 있는 장점이 있음.

## Install

- 설치는 helm install로 진행하면 name을 설정해줘야하며, 지정하지 않을시 임의로 생성됨.

```bash
helm install --name helm-test ./helm-test
```

- 위와 같이 설치가 진행되면 template으로 만들었던 deployment와 service가 동작함을 확인 가능
- 추가적으로 설치된 helm chart를 확인하려면

```bash
helm list # 설치된 helm chart 확인
```

## Upgrade

- helm install을 통해 설치된 리소스들을 Release라고 함.
- — name을 지정하여 설치하게되면 name에 해당하는 부분이 릴리즈명이 되고, 그 릴리즈를 업데이트 가능.

```bash
helm upgrade helm-test ./helm-test # install 시 사용한 이름.
```

- 릴리즈된 대상의 버젼이나 히스토리를 확인하려면 history를 사용

```bash
helm history helm-test
```

## Rollback

- history를 통해 릴리즈 대상에 대한 version을 확인할 수 있는데, 이전의 버젼으로 돌리고 싶다면 rollback 명령어 실행

```bash
helm rollback helm-test 1 # revision number
```

## Release

- 앞서 생성한 chart 한개로 하나의 클러스터에, 여러개의 어플리케이션을 설치하는 경우가 있음.
- 그러나 이렇게하는 경우 앞에 만든 템플릿으로는 에러가 생길 수 있는데, 템플릿의 리소스의 이름이 동일하기 때문

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: helm-{{ .Values.name }}-deployment # 리소스의 이름.
```

- 이런 상황을 방지하기 위해 리소스의 이름을 차트명 + 릴리즈명 형식으로 바꿔주면 유용.

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: helm-{{ .Chart.name }}-{{ .Release.name }}-deployment # 리소스의 이름.
```