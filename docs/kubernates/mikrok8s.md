# mikrok8s
## Requierement 
* [snap](/debian/snap)
* sudo package
    # apt install sudo

## Install
    # snap install microk8s --classic
    # nft tcp dport 16443 counter accept comment "accept kube"

## Get konfig
    # microk8s config

## Get token
    # token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1) 
    # microk8s kubectl -n kube-system describe secret $token | grep token:

## Add addon
    # microk8s enable dashboard 
    # microk8s enable storage

## Add loadbalancer
    # microk8s enable metallb:PUBLICIP-PUBLICIP

## kubectl

### setup
    $ export KUBECONFIG=~/.kube/microk8s
    $ kubectl get pods -A
### dashboard
    $ microk8s kubectl describe secret -n kube-system microk8s-dashboard-token
    $ kubectl proxy
* [local dashboard](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#/workloads?namespace=default)

## Grafana
### install
    # microk8s enable prometheus
    # microk8s status

### access 
* port forwad (k9s)
* admin:prom-operator