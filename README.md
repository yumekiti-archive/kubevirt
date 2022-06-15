# KubeVirtを使ったIaaS基盤構築

## Ubuntuのインストール
https://ubuntu.com/download/desktop

## Dockerのインストール
https://docs.docker.com/engine/install/ubuntu/#installation-methods
https://qiita.com/DQNEO/items/da5df074c48b012152ee

# kubectl
https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/

## minikubeのインストール
https://minikube.sigs.k8s.io/docs/start/

## kubevirt
https://kubevirt.io/

## testvm
```sh
minikube start

minikube addons enable kubevirt

kubectl apply -f vm.yaml
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

## cdi
```sh
export VERSION=$(curl -s https://github.com/kubevirt/containerized-data-importer/releases/latest | grep -o "v[0-9]\.[0-9]*\.[0-9]*")
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml

kubectl create -f pvc_fedora.yml

kubectl get all -n cdi
```

## pvc
```sh
PUBKEY=`cat ~/.ssh/id_rsa.pub`
sed -i "s%ssh-rsa.*%$PUBKEY%" vm1_pvc.yml
kubectl create -f vm1_pvc.yml

virtctl console vm1

virtctl expose vmi vm1 --name=vm1-ssh --port=20222 --target-port=22 --type=NodePort

minikube ip
ssh -i ~/.ssh/id_rsa fedora@192.168.39.74 -p 32495
```

## トラブルシューティング
- kubectl get
    - リソースの一覧を表示
- kubectl describe
    - 単一リソースに関する詳細情報を表示
- kubectl logs
    - 単一Pod上の単一コンテナ内のログを表示
- kubectl exec
    - 単一Pod上の単一コンテナ内でコマンドを実行

## quickstart
```sh
minikube start

kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/v1.48.1/cdi-operator.yaml
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/v1.48.1/cdi-cr.yaml
kubectl create -f pvc_fedora.yml

PUBKEY=`cat ~/.ssh/id_rsa.pub`
sed -i "s%ssh-rsa.*%$PUBKEY%" vm1_pvc.yml
kubectl create -f vm1_pvc.yml
kubectl create -f vm1_svc.yml
```

## docker
```sh
eval $(minikube docker-env)
make up
docker build -t localhost:5000/quay.io/kubevirt/cirros-container-disk-demo .
docker push localhost:5000/quay.io/kubevirt/cirros-container-disk-demo

kubectl apply -f docker.yml
```