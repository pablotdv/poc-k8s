wsl --import-in-place ubuntu-bkp Ubuntu-22.04.vhdx

# Instalando o Docker 

fonte: https://docs.docker.com/engine/install/ubuntu/

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

# Instalando Docker 

fonte: https://docs.docker.com/engine/install/linux-postinstall/

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

# Instalando k8s 

fonte: https://minikube.sigs.k8s.io/docs/start/

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

# Instalando o kubectl 

fonte: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

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

Criar Dockerfile

```
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["src/poc-k8s-csharp/poc-k8s-csharp.csproj", "poc-k8s-csharp/"]
RUN dotnet restore "poc-k8s-csharp/poc-k8s-csharp.csproj"
COPY src/ .
WORKDIR "/src/poc-k8s-csharp"
RUN dotnet build "poc-k8s-csharp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "poc-k8s-csharp.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "poc-k8s-csharp.dll"]
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

## Criando hpa

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: poc-k8s-csharp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: poc-k8s-csharp
  minReplicas: 5
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

```
kubectl apply -f k8s/hpa.yaml
```

# Criando service

```
apiVersion: v1
kind: Service
metadata:
  name: poc-csharp
spec:
  selector:
    app: poc-csharp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

# Criando ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: poc-k8s-csharp
  labels:
    name: poc-k8s-csharp
spec:
  rules:
  - host: poc-k8s-csharp.com.br
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: poc-k8s
            port: 
              number: 80
```

1. Acessando ingress

```
minikube tunnel
```

2. No windows, adicionar aos hosts

```
127.0.0.1 poc-k8s-csharp.com.br
```

# Configurando git credentials do windows no wsl

```
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-wincred.exe"
```

Instalando o helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Gerando template com o helm

```
helm template charts/ > local.yaml
```

Definindo label selector no node do minikube
```
kubectl label nodes minikube app=whatsbusiness
```
