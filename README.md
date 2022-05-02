# KubeVirtを使ったIaaS基盤構築

## Ubuntuのインストール
https://ubuntu.com/download/desktop

## Dockerのインストール
https://docs.docker.com/engine/install/ubuntu/#installation-methods
https://qiita.com/DQNEO/items/da5df074c48b012152ee

## K8Sのインストール
https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## minikubeのインストール
https://minikube.sigs.k8s.io/docs/start/

## kubevirt
https://kubevirt.io/

## testvm
```sh
minikube start

minikube addons enable kubevirt

kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"
```

## Virtctlインストール
```sh
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
chmod +x virtctl
sudo install virtctl /usr/local/bin
```

## pvc
https://komeiy.hatenablog.com/entry/2019/08/13/090000

```sh
PUBKEY=`cat ~/.ssh/id_rsa.pub`
sed -i "s%ssh-rsa.*%$PUBKEY%" vm1_pvc.yml
kubectl create -f vm1_pvc.yml

virtctl expose vmi vm1 --name vm1s --type NodePort --port 27017 --target-port 22
ssh 192.168.122.214 -p 31677 -l root
```

## ubuntu
https://livewyer.io/blog/2021/02/23/kubevirt-showcase-a-kubernetes-cluster-within-a-kubernetes-cluster/

```sh
export VERSION=$(curl -s https://github.com/kubevirt/containerized-data-importer/releases/latest | grep -o "v[0-9]\.[0-9]*\.[0-9]*")
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml

kubectl apply -f kubevirt-demo/examples/ubuntu/data-volume.yaml

PUBKEY=`cat ~/.ssh/id_rsa.pub`
sed -i "s%ssh-rsa.*%$PUBKEY%" kubevirt-demo/examples/ubuntu/vm.yaml
kubectl apply -f kubevirt-demo/examples/ubuntu/vm.yaml
```