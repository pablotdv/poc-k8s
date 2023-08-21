wsl --import-in-place ubuntu-bkp Ubuntu-22.04.vhdx

# Instalando o Docker https://docs.docker.com/engine/install/ubuntu/

1. atualizando e instalando pacotes

```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

2. Adicionando chaves GPG oficiais do Docker

```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

3. Configurando o repositório

```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. atualizar indices dos pacotes apt

```
sudo apt-get update
```

# Instalando Docker https://docs.docker.com/engine/install/linux-postinstall/

1. Instalando ultima versão do Docker e suas dependências

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

2. Verificando instalação do Docker

```
sudo docker run hello-world
```

# Ações pós instalação do Docker

1. Criar um groupo docker 

```
sudo groupadd docker
```

2. Adicionar usuário atual ao grupo docker

```
sudo usermod -aG docker $USER
```

3. Finalizar wsl

```
wsl --shutdown
```

4. Verificando com usuário atual

```
docker run hello-world
```

# Instalando k8s https://minikube.sigs.k8s.io/docs/start/

1. baixando e instalando o minikube

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

2. Iniciando o cluster

```
minikube start
```

3. Instalando addons

```
minikube addons list
minikube addons enable metrics-server
minikube addons enable ingress
```

4. Iniciando o dashboard

```
minikube dashboard
```

# Instalando o kubectl https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

1. Baixando o binário

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

2. Instalando o binário

```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

# Instalando o dotnet

```
sudo snap install dotnet-sdk --classic
```

# Criando projeto .net

```
mkdir poc-k8s
cd poc-k8s
mkdir src/
cd src/
dotnet new webapi -n poc-k8s-csharp
```

Build da imagem

```
docker build -f src/poc-k8s-csharp/Dockerfile . -t poc-k8s-csharp
```

Rodando a imagem

```
docker run -p 8080:80 poc-k8s-csharp
```

```
minikube image load poc-k8s-csharp
```

# Manipulando objetos do k8s

## POD

1.  Criar k8s/pod.yaml com seguinte conteúdo
```
apiVersion: v1
kind: Pod
metadata:
  name: poc-k8s-csharp
  labels:
    name: poc-k8s-csharp
spec:    
  containers:  
  - name: poc-k8s-csharp  
    image: poc-k8s-csharp
    imagePullPolicy: IfNotPresent
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
```

2. Aplicar POD no cluster

```
kubectl apply -f k8s/pod.yaml
```

3. Verificando eventos de um POD

```
kubectl get pods
kubectl describe pod/poc-k8s-csharp
```

4. Acessando o POD criado

```
kubectl port-forward pod/poc-k8s-csharp 8080:80
```

5. Excluíndo o POD

```
kubectl delete -f k8s/pod.yaml
```

## Criando um deployment

1. Criar arquivo k8s/deployment.yaml com o conteúdo

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: poc-k8s-csharp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: poc-k8s-csharp
  template:
    metadata:
      labels:
        app: poc-k8s-csharp
    spec:      
      containers:
      - name: poc-k8s-csharp
        image: poc-k8s-csharp
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```

2. Applicando o deployment no cluster

```
kubectl apply -f k8s/deployment.yaml
```

3. Acessando o deployment

```
kubectl port-forward deployment/poc-k8s-csharp 8080:80
```
