# self-hosting

sudo systemctl stop k3s
sudo /usr/local/bin/k3s-uninstall.sh
sudo rm -rf /etc/rancher/k3s /var/lib/rancher/k3s /var/lib/kubelet /var/lib/etcd /var/lib/cni
sudo rm -rf /etc/cni /opt/cni /run/k3s /run/flannel /var/run/calico

curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable traefik" sh -

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

if traefik got installed:
kubectl delete deployment traefik -n kube-system
kubectl delete service traefik -n kube-system
kubectl delete configmap traefik -n kube-system
kubectl delete clusterrolebinding traefik
kubectl delete clusterrole traefik

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace

helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  --set installCRDs=true

<!-- helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd --create-namespace --kubeconfig /etc/rancher/k3s/k3s.yaml 
> kubeconfig cuz there were problems with connection:  
    helm install argocd argo/argo-cd --namespace argocd --create-namespace
    Error: INSTALLATION FAILED: Kubernetes cluster unreachable: Get "http://localhost:8080/version": dial tcp 127.0.0.1:8080: connect: connection refused
     -->

kubectl create namespace argocd

kubectl apply -k kustomizations/argocd/

kubectl apply -f apps/infra/argo-app.yaml 

kubectl apply -f apps/infra-aoa.yaml 

> kubectl -n argocd patch application argocd-self --type merge -p '{"status": {"operationState": {"operation": {"sync": {}}}}}'

> kubectl port-forward svc/argocd-server -n argocd 8080:80

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

-----

helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring -f charts/prometheus-stack/values.yaml --create-namespace --set installCRDs=true

Delete the Helm release but leave resources intact:
helm uninstall prometheus -n monitoring --keep-history
(⚠️ This leaves CRDs & resources, but Helm no longer manages them).

helm uninstall prometheus -n monitoring --keep-history=false --no-hooks

kubectl -n monitoring get job prometheus-stack-kube-prom-admission-create -o yaml | grep finalizers -A2
kubectl -n monitoring patch job prometheus-stack-kube-prom-admission-create -p '{"metadata":{"finalizers":[]}}' --type=merge
kubectl -n monitoring delete job prometheus-stack-kube-prom-admission-crea
