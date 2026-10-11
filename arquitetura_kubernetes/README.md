# ☸️ Arquitetura Kubernetes & Implementação com Minikube

> **Atividade Prática — Unidade 9: Kubernetes**  
> **Integrantes da Dupla:** Pedro e Otávio  
> **Repositório:** [Link do GitHub da Dupla]  

---

## 📌 1. Visão Geral da Arquitetura do Kubernetes

O Kubernetes (K8s) é uma plataforma de código aberto para automação de implantação, escalonamento e gerenciamento de aplicações em contêineres. O cluster é dividido em duas partes fundamentais: o **Control Plane (Plano de Controle)** e os **Worker Nodes (Nós de Execução)**.

O diagrama completo e editável desta arquitetura está disponível no arquivo [`kubernetes.drawio`](kubernetes.drawio) (abra no VS Code com a extensão *Draw.io Integration*).

```
+-------------------------------------------------------------------------------+
|                             CONTROL PLANE (MASTER)                           |
|  +--------------------+   +-------------------+   +------------------------+  |
|  |   kube-apiserver   |<->|       etcd        |   | kube-controller-manager|  |
|  +--------------------+   +-------------------+   +------------------------+  |
|           ^                         ^                          ^              |
|           |                 +-----------------+                |              |
|           +---------------->| kube-scheduler  |<---------------+              |
|                             +-----------------+                               |
+-------------------------------------------------------------------------------+
       | (Comunicação REST/TLS)
       v
+-------------------------------------------------------------------------------+
|                                WORKER NODE                                    |
|  +--------------------+   +-------------------+   +------------------------+  |
|  |      kubelet       |   |    kube-proxy     |   |   Container Runtime    |  |
|  +--------------------+   +-------------------+   +------------------------+  |
|           |                                                    |              |
|           v                                                    v              |
|     +-----------+         +-------------------+          +-----------+        |
|     |  Service  |<------->|   Pod (Pod.yml)   |<-------->| Container |        |
|     | (NodePort)|         |   Labels: meu-app |          | (Port 80) |        |
|     +-----------+         +-------------------+          +-----------+        |
+-------------------------------------------------------------------------------+
```

---

## 🛠️ 2. Descrição dos Componentes e Ferramentas

### 🧠 Control Plane (Nó Mestre)
Responsável pelas decisões globais sobre o cluster (ex: agendamento), detecção e resposta a eventos.
* **`kube-apiserver`**: É a porta de entrada do Control Plane. Expõe a API REST do Kubernetes e processa todas as requisições internas e externas (recebidas via `kubectl`).
* **`etcd`**: Banco de dados chave-valor distribuído e altamente confiável. Armazena todo o estado do cluster, configurações e dados dos objetos.
* **`kube-scheduler`**: Monitora Pods recém-criados que ainda não possuem nó atribuído e seleciona o melhor Worker Node para executá-los com base em recursos disponíveis.
* **`kube-controller-manager`**: Executa processos de controle em segundo plano para manter o estado atual do cluster igual ao estado desejado (ex: Node Controller, ReplicaSet Controller, Endpoint Controller).

---

### 💻 Worker Nodes (Nós de Trabalho)
Máquinas (físicas ou virtuais) onde as aplicações em contêineres realmente rodam.
* **`kubelet`**: Agente principal que roda em cada nó. Garante que os contêineres descritos nos PodSpecs estejam em execução e saudáveis.
* **`kube-proxy`**: Gerencia as regras de rede em cada nó, permitindo a comunicação entre Pods e viabilizando o funcionamento dos **Services** tanto interna quanto externamente.
* **`Container Runtime`**: O software responsável por baixar as imagens e rodar os contêineres (ex: *containerd*, *Docker*, *CRI-O*).

---

### 📦 Objetos e Ferramentas do Kubernetes
* **`kubectl`**: Interface de linha de comando (CLI) usada pelos administradores e desenvolvedores para enviar instruções e manifestos YAML ao `kube-apiserver`.
* **`Pod`**: A menor e mais básica unidade de execução no Kubernetes. Encapsula um ou mais contêineres que compartilham rede (IP e portas), armazenamento e especificações de execução.
* **`Service`**: Abstração que define um conjunto de Pods e uma política de acesso a eles. Fornece um IP e DNS estáveis, garantindo balanceamento de carga mesmo se os Pods forem recriados com novos IPs.
  * **ClusterIP**: Apenas acessível internamente no cluster.
  * **NodePort**: Abre uma porta dedicada em cada nó do cluster (faixa 30000-32767), permitindo acesso externo direto pelo navegador.
  * **LoadBalancer**: Integração com balanceadores de carga em nuvem.
* **`ReplicaSet`**: Objeto responsável por garantir que um número específico de réplicas de um Pod esteja sempre em execução, oferecendo tolerância a falhas.

---

## 📄 3. Códigos YAML Desenvolvidos

### `pod.yml` (Implementação do Pod)
Manifesto simples para subir um Pod utilizando uma imagem customizada do Docker Hub:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
  labels:
    app: meu-app
spec:
  containers:
  - name: container-app
    image: pedrorodrigues/meu-app:latest # Substituir pela imagem do seu Docker Hub
    ports:
    - containerPort: 80
```

### `service.yml` (Implementação do Service)
Manifesto para expor o Pod externamente via **NodePort** na porta `30080`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  type: NodePort
  selector:
    app: meu-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

---

## 🚀 4. Passo a Passo de Execução no Minikube e Coleta de Artefatos

### Passo 1: Criar sua Imagem Própria e subir no Docker Hub (Requisito Mínimo)
No terminal da sua máquina:
```bash
# 1. Crie uma pasta temporaria com um index.html
mkdir app-docker && cd app-docker
echo "<h1>Kubernetes Rodando - Dupla: Pedro e Otavio</h1>" > index.html

# 2. Crie o Dockerfile simples
echo "FROM nginx:alpine" > Dockerfile
echo "COPY index.html /usr/share/nginx/html/index.html" >> Dockerfile

# 3. Faça build e push para o Docker Hub
docker build -t SEU_USUARIO_DOCKERHUB/meu-app:latest .
docker login
docker push SEU_USUARIO_DOCKERHUB/meu-app:latest
```

### Passo 2: Iniciar o Minikube
```bash
minikube start
```

### Passo 3: Aplicar os Manifestos do Kubernetes
```bash
kubectl apply -f pod.yml
kubectl apply -f service.yml
```

### Passo 4: Verificar a Execução (Comandos para Artefatos/Prints)
```bash
# Listar status do Pod (deve estar com STATUS: Running)
kubectl get pods -o wide

# Listar status do Service (deve exibir o NodePort 30080)
kubectl get services meu-service

# Descrever o Pod e confirmar o pull da imagem
kubectl describe pod meu-pod
```

### Passo 5: Acessar pelo Navegador (Artefato de Navegação)
No Minikube, execute o comando abaixo para abrir a URL no navegador:
```bash
minikube service meu-service
```


