# self-hosting

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

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
