# Secure

모든 통신을 TLS로

- 대부분의 엑세스는 Kube-apiserver를 통하지 않고서는 불가능.
- 엑세스 가능한 유저
    - File - user name & token
    - Service accounts
    - Certificates
    - External Authentication Providers - LDAP

## Accounts

- 사용자를 위한 user - 개발자 또는 devops 팀.
- 어플리케이션(pod 외)을 위한 service account - end-user

### Static Token

- apiserver를 시작할 떄 —token-auth-file=xxx.csv 전달.
- apiserver를 재시작 해야함.
- token, user name, user uid, group name(optional) 의 구조를 이룸.

```bash
kubectl config set-credentials user1 --token=password1 # id pw 등록!
kubectl config set-context user1-context --cluster=kubernetes \
--namespace=frontend --user=user1 # server를 선택하는 과정
# kubectl config use-context user1-context # user-1 context 사용
kubectl get pod --user user1
```

### Service Accounts

- pod에 별도 설정을 해주지 않는다면 기본적으로 service account가 생성됨.

```bash
kubectl create serviceaccount sa1 # service account 생성
```

- Pod에 spec.serviceAccount: sa1 와 같은 형식으로 지정.

## TLS

### SSL 통신 과정 이해

- Application layer와 Transport layer 사이에서 동작.
- 기존에 사용되는 application(HTTP, FTP..)을 지원하기 위해 사용.
- 데이터의 암호화, 데이터 무결성, 서버 인증 기능 등.

### Certificate Authority(CA)

- CA에서 public key와 certificate를 받음.
- client가 server에 접속하면, certificate가 있음을 확인하고
- server로 부터 certificate와 public key를 줌.
- client는 이를 가지고 CA에서 올바른지 확인.
- 확인이 완료되면 public key로 암호화된 session key를 server로 전달.
- server는 session key를 확인했음을 전달(Acknowlegement)
- 통신.

```bash
# kubernetes certificate 위치
sudo ls /etc/kubernetes/pki 
```

### Certification 확인하기

```bash
sudo openssl x509 -in <certificate_path> -text
```

- Issuer: 누가 인증서를 발급해주었는가
- Subject: 누가 발급 받았는가
- Validate: 유효기간

### 인증서 갱신

- [kubeadm을 사용한 인증서 관리](https://kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
- Check certificate expiration

```shell
# certification expiration 확인
kubeadm alpha certs check-expiration

# certification renewal
kubeadm alpha certs renew all
```

- Automatic certificate renewal
    - kubeadm은 control plane을 업그레이드하면 자동으로 모든 인증서 갱신.
- Manual certificate renewal

### TLS 인증서를 활용한 유저 생성

- static token은 password를 관리하기 어렵고, password를 잃어버리면 apiserver를 내리고 다시 setting 해줘야하기에 TLS를 활용.
- ca를 사용하여 직접 CSR(Certifcate Signing Request) 승인하기
    - 개인 키 생성.
    - private key를 기반으로 인증서 서명 요청하기
- 내부에서 직접 승인하는 경우 pki dir에 있는 ca.key와 ca.crt를 사용하여 승인.

```shell
# public key 생성
openssl genrsa -out brettahn.key 2048

# CSR 만들기
openssl req -new -key brettahn.key -out brettahn.csr \
-subj "/CN=brettahn/O=Seoul" # CN: 사용자 이름, O = Organization(그룹 이름)

# CA 이용하여 CSR 승인
openssl x509 -req -in brettahn.csr -CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out brettahn.crt -days 500

# crt 사용을 위한 등록
kubectl config set-credentials brettahn --client-certificate=.certs/brettahn.crt \
--client-key=.certs/brettahn.key

# user context 만들기
kubectl config set-config brettahn@kubernetes --cluster=kubernetes \
--user=brettahn --namespace=test

# 등록 확인
kubectl --context=brettahn-context get po
# Forbidden 떠야 정상
```

## Config

- TLS를 사용하거나 인증관련 정보를 kubectl로 조작할 수 있으나, 매번 명령어로 하기 불편.
- 이때 config를 이용하여 인증서나 user, context 등을 등록해두고 사용할 수 있음.

```bash
kubectl config view # config 내용 확인
```

### kube config의 구조

- ~/.kube/config 파일을 확인하면 3가지 부분으로 작성됨
    - cluster: 연결할 kubernetes cluster 정보 CA 정보 및 경로가 저장되어 있고, 서버의 주소가 적혀있음.
    - users: 사용할 권한을 가진 사용자. client-key와 certificate의 경로를 같이 적음.
    - contexts: user-name@cluster-name의 naming을 권장하며, cluster와 user의 조합

```bash
kubectl config use-context user@kube-cluster # config에 있는 context 중 하나를 사용.
```