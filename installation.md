## Docker and kube, and Kserve on AWS Setup:

1. Login to AWS console.
2. Create IAM user with AdministratorAccess
3. Export the credentials in your AWS CLI by running "aws configure"
4. Create a s3 bucket
5. Create EC2 machine (Ubuntu) & add Security groups 5000 port
6. You may add below in ~/.bashrc , once updated you need to re-enter the terminel or bash
```bash
alias k='kubectl'
```
Run the following command on EC2 machine
```bash
sudo apt update && sudo apt upgrade -y;
sudo apt install unzip net-tools -y ;
```

## Install aws CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## Then set aws credentials
aws configure

## Install UV
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
exit bash, then re-enter
```

## Install docker
https://docs.docker.com/engine/install/ubuntu/
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker $USER
exit bash, then re-enter

## Install kind
https://kind.sigs.k8s.io/docs/user/quick-start/#installation
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# sudo mv kind /usr/local/bin
# sudo chmod +x /usr/local/bin/kind

## Install kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl

## Git clone
- git clone https://github.com/Kapil987/Kapil987-mlops-end-to-end2.git
- cd kind

## Kind commands
kind create cluster --config cluster-config.yml
kind get clusters
<!-- kind delete clusters clusterName -->

### Install Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh

#  latest available version
curl -s https://api.github.com/repos/kserve/kserve/releases/latest | grep -oP '"tag_name": "\K[^"]+'

### Install Cert Manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.20.0/cert-manager.yaml
# kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml

k get pods -n cert-manager # Make sure the pods are running
```

### Install KServe CRDs
<!-- helm uninstall kserve -n kserve 2>/dev/null
helm uninstall kserve-crd -n kserve
kubectl get crds | grep kserve.io | awk '{print $1}' | xargs kubectl delete crd -->

```bash
kubectl create namespace kserve

helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd \
  --version v0.16.0 \
  -n kserve \
  --wait

helm list -A

```

### Install KServe controller
```bash
helm upgrade --install kserve oci://ghcr.io/kserve/charts/kserve \
  --version v0.16.0 \
  -n kserve \
  --set kserve.controller.deploymentMode=RawDeployment

kubectl get pods -n kserve -w

Ensure below
kserve-controller-manager → 2/2 Running
kserve-webhook-server → Running (if exists)

kubectl get endpoints -n kserve
```


### KServe Deployment

```bash

kubectl apply -f k8s/serviceaccount.yaml
kubectl apply -f k8s/inference.yaml

kubectl get inferenceservice -n churn-model
kubectl get inferenceservice churn-predictor -n churn-model -w
```

⚠️ Ensure AWS credentials are updated in `serviceaccount.yaml`.


## 🔁 GitOps with ArgoCD

```bash
kubectl create namespace argocd

kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl apply -f argocd/application.yaml

kubectl port-forward svc/argocd-server -n argocd 8080:443

kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

## Add secrets in github
images/1.add_secrets.png

## Kubectl commands
kubectl apply -f deployment.yaml
kubectl get svc
kubectl describe po PODName -n namespace
kubectl port-forward svc/flask-service 8080:80 --address 0.0.0.0 

## AWS
make sure to open NodePort in security group of you ec2


## Docker commands
docker build -t dockerHubRepoName/ImageName:tagName pathToDockerFile
docker build -t kapil0123/myapp .

docker build -t myapp .
docker build -t myimage:latest -f MyDockerfile .
docker images
docker run --rm --name contaienrName -p 80:5000 ImageName:TagName # --rm to create temporary container
docker run --rm --name test -p 80:5000 kapil0123/myapp


docker ps -a
docker inspect
docker network ls
docker volume ls
docker system prune


## Troubleshooting
alias k='kubectl'
k delete -f deploy.yml

kubectl run test --rm -it --image=busybox -- sh
wget -qO- http://flask-service

kind delete clusters demo1

ubuntu@ip-172-31-26-169:~/15_Docker_Kubernetes/app$ kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
flask-service   NodePort    10.96.29.170   <none>        80:30007/TCP   2m2s
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        33m
ubuntu@ip-172-31-26-169:~/15_Docker_Kubernetes/app$

- Error
ubuntu@ip-172-31-30-0:~$ helm upgrade --install kserve oci://ghcr.io/kserve/charts/kserve \
  --version v0.16.0 \
  -n kserve \
  --set kserve.controller.deploymentMode=RawDeployment
Release "kserve" does not exist. Installing it now.
Pulled: ghcr.io/kserve/charts/kserve:v0.16.0
Digest: sha256:46d78a58fff65902213a7254ec16520286baa81d33340c4a03f5263d846e2124
Error: Internal error occurred: failed calling webhook "clusterservingruntime.kserve-webhook-server.validator": failed to call webhook: Post "https://kserve-webhook-server-service.kserve.svc:443/validate-serving-kserve-io-v1alpha1-clusterservingruntime?timeout=10s": dial tcp 10.96.133.27:443: connect: connection refused
Internal error occurred: failed calling webhook "clusterservingruntime.kserve-webhook-server.validator": failed to call webhook: Post "https://kserve-webhook-server-service.kserve.svc:443/validate-serving-kserve-io-v1alpha1-clusterservingruntime?timeout=10s": dial tcp 10.96.133.27:443: connect: connection refused
Internal error occurred: failed calling webhook "clusterservingruntime.kserve-webhook-server.validator": failed to call webhook: Post "https://kserve-webhook-server-service.kserve.svc:443/validate-serving-kserve-io-v1alpha1-clusterservingruntime?timeout=10s": dial tcp 10.96.133.27:443: connect: connection refused
ubuntu@ip-172-31-30-0:~$

- sol: try installing again after 1 minute

- if Error while replacing the inferance.yaml
yq e '.metadata.labels."model-version" = env(MODEL_VERSION)' -i k8s/inference.yaml


