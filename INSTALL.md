On LXD & using Microk8s for learning

Create dataset in exsting pool
```
sudo zfs create -o mountpoint=/tank/k8s tank/k8s
```

Create image files for disks so we can attach - you could use iscsic whatever
```
for i in {1..3}; do
  lxc init ubuntu:20.04 --vm -c limits.cpu=6 -c limits.memory=12GB k8s$i
  lxc config device override k8s$i root size=100GB
  sudo truncate -s 300G /tank/k8s/k8s$i.img
  lxc config device add k8s$i zfs disk source=/tank/k8s/k8s$i.img
done
```

Create pool using /dev/sdb 300G on each k8s node that will use zfs k8s
```
apt-get install -y zfsutils-linux
zpool create zfspv-pool /dev/sdb
```

Create ZZFS driver and stroage class
```
curl -LO https://openebs.github.io/charts/zfs-operator.yaml && \
  sed -i 's/\/var\/lib\/kubelet\//\/var\/snap\/microk8s\/common\/var\/lib\/kubelet\//g' zfs-operator.yaml && \
  kubectl apply -f zfs-operator.yaml
kubectl apply -f https://raw.githubusercontent.com/jeremybusk/k8s-cluster/main/zfs-sc.yaml
```

Create PVC & Pod and do some testing
```
kubectl apply -f https://raw.githubusercontent.com/jeremybusk/k8s-cluster/main/pvc.yaml
kubectl apply -f https://raw.githubusercontent.com/jeremybusk/k8s-cluster/main/test-zfs-pod.yaml
kubectl get pod fio
kubectl exec -it fio -- sh
ubectl exec -it fio -- sh -c "echo test | tee -a /datadir/test.txt"
kubectl exec -it fio -- sh -c "cat /datadir/test.txt"
kubectl delete -f https://raw.githubusercontent.com/jeremybusk/k8s-cluster/main/test-zfs-pod.yaml
kubectl delete -f https://raw.githubusercontent.com/jeremybusk/k8s-cluster/main/pvc.yaml
```



# Misc Example Commands
```
lxc config set $host limits.cpu=8 limits.memory=16GB
```

Show pods with claims
```
kubectl get pods --all-namespaces -o=json | jq -c \
'.items[] | {name: .metadata.name, namespace: .metadata.namespace, claimName:.spec.volumes[] | select( has ("persistentVolumeClaim") ).persistentVolumeClaim.claimName }'
```
