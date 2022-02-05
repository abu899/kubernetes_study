# Deploy

## Packaging

```bash
helm package ./helm-test
```

- Chart directory를 package로 실행하면, .tgz 파일로 패키징 됨.
- 만약 파일의 무결성을 보장하고 싶다면, -sign option을 package와 함께 넘겨 signing 진행.
    - signing하게 되면 .prov(provenance file)이 같이 생성됨.
    - install 시 -verify option과 prov 파일을 이용해 무결성을 확인하고 변조되지 않은 경우 install을 진행.
- [Helm](https://v2.helm.sh/docs/developing_charts/#helm-provenance-and-integrity)


## Helm chart repository

- packaging 된 chart file을 서버를 통해 배포할 수 있음
- 만약 packing된 파일이 있는 directory를 repository로 설정하기 위해선 serve 명령어를 사용.

```bash
helm serve --repo-path .
```

- 위 명령어를 실행하면 현재 폴더에 있는 .tgz파일들을 바탕으로 index.yaml 파일을 만들어주고 자동으로 8879 port를 이용해 제공해 줌.
- index파일에는 repository에 있는 패키지들의 정보가 들어있음.

- serve를 이용하는 것이 아닌 외부 repository를 이용할 경우 별도로 index파일을 생성해줘야하며, 직접 repository url을지정하여 만들어 줘야함.

```bash
helm repo index . --url https://brettahn.github.io/helm-test
```

- local이나 외부에 만들어진 repo를 이용하는 방법은, 현재 helm-client의 repository list에 추가하는 것.

```bash
helm repo add helm-test-repo http://localhost:8879
```

## Chart Mueseum

- Open-source Helm Chart Repository
- 인증 기능을 제공하며 AWS, GCS 등을 스토리지로 사용 가능.
- 도커로 간단하게 설치 가능하며, repository 사용에 필요한 추가적인 기능들을 제공함.
- [ChartMuseum - Helm Chart Repository](https://chartmuseum.com/)