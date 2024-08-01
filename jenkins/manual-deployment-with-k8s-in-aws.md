## How To Manually Deployment With K8s Architecture in AWS(eks)

#### Application backend Node, Frontend React, Database MongoCloud

```
aws

aws configure

eksctl create cluster -f .\cluster.yml

kubectl get pod

kubectl get nodes


kubectl apply -f .\k8s\client-deployment.yml

kubectl apply -f .\k8s\server-deployment.yml
```

#### Install choco and kubernetes-helm
```
// Ingress setup
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"


choco --version

choco install kubernetes-helm
```
#### Get Helm & Install RELEASE_NAME
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update

helm install [RELEASE_NAME(test-ingress)] ingress-nginx/ingress-nginx
```
#### Now Apply Ingress Service
```
//after install ingress apply ingress
kubectl apply -f .\k8s\ingress-service.yml

// get services
kubectl get services
```

### Delete All Deployment & Services
```
# Delete all deployments
kubectl delete deployments --all

# Delete all services
kubectl delete services --all

# Optionally, delete all resources in the default namespace
kubectl delete all --all -n default

# Delete node
kubectl delete node <node-name>

kubectl delete namespace <namespace-name>

```
### Ensuring a Clean Environment
```
# List all namespaces
kubectl get namespaces

# List pods in all namespaces
kubectl get pods --all-namespaces

# List deployments in all namespaces
kubectl get deployments --all-namespaces

# List services in all namespaces
kubectl get services --all-namespaces

```