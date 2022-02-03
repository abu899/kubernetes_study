# Backup & Restore

백업 리소스의 종류

- 포드의 정보 파일 YAML
- ETCD 데이터베이스
- Persistent Volume

## Backup

```bash
# pod의 정보 파일 yaml
kubectl get all --all-namespaces -o yaml > kube_backup.yaml

# etcd backup
# cacert, cert, key는 /etc/kubernets/pki/etcd 내에 존재
sudo ETCDTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
 --cacert /etc/kubernetes/pki/etcd/ca.crt \
 --cert /etc/kubernetes/pki/etcd/server.crt \
 --key /etc/kubernetes/pki/etcd/server.key \
 snapshot save sanpshotdb

```

## Restore

```bash
# etcd restore
sudo ETCDTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key \
--data-dir /var/lib/etcd
--initial-cluster='master=https://127.0.0.1:2380' \
--name=master
--initial-cluster-token token-value \
--initial-advertise-peer-urls https://127.0.0.1:2380 \
 snapshot restore sanpshotdb

#pod restore
kubectl create -f kube_backup.yaml
```