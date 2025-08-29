# self-hosting

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd --create-namespace --kubeconfig /etc/rancher/k3s/k3s.yaml 
> kubeconfig cuz there were problems with connection:  
    helm install argocd argo/argo-cd --namespace argocd --create-namespace
    Error: INSTALLATION FAILED: Kubernetes cluster unreachable: Get "http://localhost:8080/version": dial tcp 127.0.0.1:8080: connect: connection refused
    
kubectl apply -f apps/infra/argo-app.yaml 

kubectl apply -f kustomizations/argocd/base/argo-projects.yaml -> no idea how this should be applied otherwise (picked up by argocd)