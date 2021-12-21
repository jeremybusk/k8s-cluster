kubectl -f config.yaml apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
kubectl -f config.yaml apply -f https://raw.githubusercontent.com/jeremybusk/k8s-cluster/main/metallb/config.yaml \
  -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml
