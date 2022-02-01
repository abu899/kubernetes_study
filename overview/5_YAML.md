# YAML

## 기본 사용법

- 들여쓰기는 2칸 or 4칸
- 데이터는 key : value 형식의 map 형태.
- 배열은 - 로 표시
- 주석은 #으로 표시.
- 참, 거짓은 true, false 이외에 yes, no 지원
- 정수 또는 실수는 "" 없이 그대로 사용.
- 여러 줄을 사용할시 "|", "|-", ">" 사용. 순서대로 마지막 줄 바꿈, 마지막 줄 바꿈 제외, 중간에 들어가는 빈줄 제외.

```yaml
# comment
person:
  name: Brett Ahn
  job: Developer
  skills:
    - docker
    - kubernetes
  age: 33
  married: no
  newlines_sample: |
    number one line

    second line

    last line
```

[YAMLlint - The YAML Validator](http://www.yamllint.com/)

# Kubernetes에서 YAML 작성하기

### 필수 작성

- `apiVersion` - 이 오브젝트를 생성하기 위해 사용하고 있는 쿠버네티스 API 버전이 어떤 것인지
- `kind` - 어떤 종류의 오브젝트를 생성하고자 하는지(deployment, pod, service, job, ingress..)
- `metadata` - `이름` 문자열, `UID`, 그리고 선택적인 `네임스페이스`를 포함하여 오브젝트를 유일하게 구분지어 줄 데이터  (name, label, namespace, etc..)
    - name : 동일한 namespace 상에서 유일한값
    - labels : 특정 k8s object만 나열하거나 검색할때 유용하게 쓰이는 key-value쌍.
- `spec` - 오브젝트에 대해 어떤 상태를 의도하는지(container, volumn)
    - containers : pod에는 1개 이상의 container 포함 가능. containers에 원하는만큼 container 정의해서 넣으면됨
    - image : pull 받아올 docker 이미지 주소
    - replicas : 원하는 pod 갯수 정의
    - selector : controller가 어떤 pod를 감시해야하는지.
    - template : 새 pod를 런칭하는데 사용할 템플릿. selectors의 값이 template의 labels과 일치해야 관리되는 파드를 제대로 선택.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
  labels:
    app: pod-test
spec:
  containers:
    - name: pod-container
      image: where-to-image-pull
```

- 작성시 labels, containers 등 철자에 유의.
- metadata의 name은 pod의 name, containers의 name은 생성되는 container의 name이니 꼭 같을 필요는 없었음.
- containers, env와 같이 "-"로 시작하는 것은 배열로 표현한 것으로, 여러개가 들어올 수 있기에 항상 "-"로 시작.