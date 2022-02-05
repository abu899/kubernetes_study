# Rule Based Access Control(RBAC)

- 그룹 내 개별 사용자의 역할(Role)을 기반으로 리소스에 대한 Access를 제어
- [rbac.authorization.k8s.io](http://rbac.authorization.k8s.io) API를 사용하여 정의
- RBAC를 사용하여 Rule을 정의하려면 apiser에 —authorization-mode=RBAC 옵션이 필요.
- 관리자가 kubernetes api를 통해 정책을 동적으로 구성하기 위해 사용.
- 예를 들어 frontend, backend, application dev rule을 만들어두고, user를 만들어둔 rule에 binding 시켜서 각 rule에 정의된 권한을 사용하는 것.

## [rbac.authorization.k8s.io](http://rbac.authorization.k8s.io) API

- RBAC를 다루는 이 API는 총 4가지의 리소스를 컨트롤
    - Role: namespace에 종속적. 주로 사용자, 개발자 들에게 사용.
    - Role Binding
    - ClusterRole: cluster 전체에서 사용될 수 있는 권한. 관리자 또는 모니터링에게 사용.
    - ClusterRoleBinding
- `Role`과 `RoleBinding`의 차이
    - `Role`은 누가하는 것인지를 정의하지 않고 행위(rule)를 정의.
    - `RoleBinding`은 누가할 것인지를 정의하는 것.
    - `RoleBinding`은 `Role`을 정의하는 대신 참조할 `Role`을 정의한다(roleRef)
    - [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

<p align="center"><img src="./img/4_1.png" width="80%"></p>

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
  - apiGroups: [ "" ] # "" indicates the core API group
    resources: [ "pods" ]
    verbs: [ "get", "watch", "list" ]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
  # You can specify more than one "subject"
  - kind: User
    name: jane # "name" is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```