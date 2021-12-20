On LXD for learning

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
kubectl apply -f https://openebs.github.io/charts/zfs-operator.yaml
kubectl apply -f https://raw.githubusercontent.com/jeremybusk/k8s-cluster/main/zfs-sc.yaml
```



# Misc Example Commands
```
lxc config set $host limits.cpu=8 limits.memory=16GB
```
