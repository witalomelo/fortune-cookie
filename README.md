# Guia rápido: ArgoCD no projeto Fortune Cookie

Este documento reúne todos os comandos e etapas para configurar e usar o ArgoCD pela primeira vez em um projeto Kubernetes. Siga as seções abaixo para entender o fluxo completo, desde o ambiente até o deploy automatizado.

---

## 1. Organização do Projeto

```sh
cd projetos/devops/argoCD
ls -l
code .
mkdir -p k8s-manifests
cd k8s-manifests
ls -l
cd ..
```

## 1.1. Criação dos manifestos Kubernetes

Os manifestos são arquivos YAML que descrevem os recursos do seu projeto no Kubernetes, como Deployments, Services e Ingress. Eles devem ser criados dentro da pasta `k8s-manifests`.

Exemplo de criação dos arquivos:

```sh
touch k8s-manifests/backend-deployment.yaml
touch k8s-manifests/backend-service.yaml
touch k8s-manifests/frontend-deployment.yaml
touch k8s-manifests/frontend-service.yaml
touch k8s-manifests/ingress.yaml
```

Cada arquivo deve conter a definição do recurso correspondente. Por exemplo, um Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: rafaelvzago/fortune-frontend:1.1
          ports:
            - containerPort: 80
```

Repita o processo para os demais recursos (backend, services, ingress etc). Após criar e editar os manifestos, faça o commit e o push para o repositório Git.

## 2. Inicialização do Ambiente Kubernetes

```sh
minikube start
export KUBECONFIG=$HOME/.kube/config
mkdir $HOME/.kube
cd .kube
cat config
vim config
printenv
source ~/.zshrc
```

## 4. Comandos Kubernetes essenciais

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
kubectl get secrets -n argocd
kubectl get -n argocd secrets argocd-initial-admin-secret
kubectl get -n argocd secrets argocd-initial-admin-secret -o json
echo <senha_base64> | base64 -d
watch -n 1 kubectl get pods -n argocd
kubectl port-forward svc/argocd-server -n argocd 8080:443
kubectl port-forward svc/frontend 8081:80
kubectl get pods
```

## 5. Configuração do ArgoCD

### Instalação

```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Acesso à interface web

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
firefox localhost:8080
```

### Login via CLI

```sh
argocd login localhost:8080 --username admin --password <senha> --insecure
```

Obtenha a senha inicial com:

```sh
kubectl get -n argocd secrets argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Adicionando o repositório Git

```sh
argocd repo add git@github.com:user/fortune-cookie.git --ssh-private-key-path ~/.ssh/argocd_rsa
```

### Criando a aplicação ArgoCD

```sh
argocd app create fortune-cookie \
  --repo git@github.com:user/fortune-cookie.git \
  --path k8s-manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --revision argocd
```

### Criando a aplicação ArgoCD

Utilize o comando abaixo para criar a aplicação no ArgoCD, apontando para o repositório, pasta de manifestos, cluster e namespace desejados:

```sh
argocd app create fortune-cookie \
  --repo git@github.com:user/fortune-cookie.git \
  --path k8s-manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --revision argocd
```

**Explicação dos parâmetros:**

- `fortune-cookie`: nome da aplicação no ArgoCD
- `--repo`: URL do repositório Git
- `--path`: pasta onde estão os manifestos Kubernetes
- `--dest-server`: endereço do cluster Kubernetes
- `--dest-namespace`: namespace de destino
- `--revision`: branch ou tag do repositório

### Sincronizando a aplicação

```sh
argocd app sync fortune-cookie
```

### Histórico e rollback

```sh
argocd app history fortune-cookie
argocd app rollback fortune-cookie <id>
```

## 6. Comandos Git

```sh
git status
git add .
git add frontend-deployment.yaml
git commit -m "mensagem"
git commit -s -m "mensagem"
git push
git checkout -b argocd
```

## 7. Gerando e usando chave SSH para o GitHub

```sh
ssh-keygen -t rsa -b 4096 -C "<email>" -f ~/.ssh/argocd_rsa
ssh-add ~/.ssh/argocd_rsa
cat ~/.ssh/argocd_rsa.pub
ssh -T git@github.com
```

Use a chave pública gerada para adicionar ao GitHub e permitir acesso via SSH.

## 8. Outros comandos úteis

```sh
docker ps
top
clear
history
echo <senha_base64> | base64 -d
```

---
