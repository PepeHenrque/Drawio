# Arquitetura Kubernetes e Manifestos de Implementação

Trabalho prático da disciplina sobre arquitetura e funcionamento do Kubernetes (Unidade 9).

**Integrantes:**
- Pedro Henrique
- Otávio

---

## 1. Visão Geral da Arquitetura

O Kubernetes é uma plataforma de orquestração de contêineres projetada para automatizar a implantação, o dimensionamento e o gerenciamento de aplicações distribuídas. O cluster é estruturado em duas camadas principais: o **Control Plane** (responsável pelo gerenciamento e estado do cluster) e os **Worker Nodes** (máquinas responsáveis pela execução das cargas de trabalho).

O diagrama visual da arquitetura encontra-se modelado no arquivo `kubernetes.drawio`, que pode ser visualizado diretamente no VS Code utilizando a extensão *Draw.io Integration*.

---

## 2. Descrição das Ferramentas e Componentes

### 2.1 Control Plane (Plano de Controle)

O Control Plane atua como o núcleo de controle do cluster, coordenando todas as operações e mantendo o estado desejado dos recursos:

- **kube-apiserver:** Ponto central de comunicação de todo o cluster. Expõe a API REST do Kubernetes e autentica, autoriza e valida todas as requisições enviadas tanto por usuários externos (via `kubectl`) quanto por componentes internos.
- **etcd:** Banco de dados distribuído do tipo chave-valor, consistente e altamente disponível. Armazena todo o estado do cluster, configurações e dados de persistência de todos os objetos.
- **kube-scheduler:** Componente responsável por monitorar Pods recém-criados que ainda não possuem um nó atribuído, selecionando o Worker Node mais adequado para alocação com base em requisitos de recursos, afinidades e restrições.
- **kube-controller-manager:** Executa os processos de controle em segundo plano. Monitora continuamente o estado atual do cluster e atua para convergí-lo ao estado desejado (inclui o Node Controller, ReplicaSet Controller e Endpoint Controller).

### 2.2 Worker Nodes (Nós de Trabalho)

Os Worker Nodes são os nós operacionais onde os contêineres são executados:

- **kubelet:** Agente em execução em cada nó do cluster. É responsável por se comunicar com o `kube-apiserver` e garantir que os contêineres descritos nas especificações dos Pods estejam saudáveis e em execução.
- **kube-proxy:** Responsável pelo gerenciamento das regras de rede em cada nó, encaminhando o tráfego destinado aos serviços e viabilizando o balanceamento de carga entre os Pods.
- **Container Runtime:** Software subjacente encarregado de gerenciar o ciclo de vida dos contêineres (baixar imagens, inicializar e finalizar contêineres). Exemplos: containerd, CRI-O, Docker.

### 2.3 Objetos e Ferramentas do Ecossistema

- **kubectl:** Ferramenta de linha de comando oficial para interação e envio de instruções declarativas (em formato YAML) para o `kube-apiserver`.
- **Pod:** Menor unidade implantável do Kubernetes. Representa uma instância de um processo no cluster e encapsula um ou mais contêineres que compartilham o mesmo namespace de rede (endereço IP e portas) e volumes de armazenamento.
- **Service:** Abstração que expõe uma aplicação em execução em um conjunto de Pods sob um ponto de acesso de rede estável (IP virtual e DNS), provendo balanceamento de carga. O tipo `NodePort` disponibiliza o serviço em uma porta estática em cada nó, possibilitando o acesso externo via navegador.
- **ReplicaSet:** Controlador cujo objetivo primário é garantir a disponibilidade de um número fixo de réplicas de Pods em execução contínua em qualquer momento determinado.

---

## 3. Manifestos de Implementação

### 3.1 Pod (`pod.yml`)

Implementa um Pod executando uma imagem própria publicada no Docker Hub:

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
    image: pepehenrque/my_app:latest
    ports:
    - containerPort: 80
```

### 3.2 Service (`service.yml`)

Implementa um Service do tipo `NodePort` para expor o Pod através da porta 30080:

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

## 4. Instruções de Implantação no Minikube

1. **Inicialização do cluster:**
   ```bash
   minikube start
   ```

2. **Aplicação dos manifestos:**
   ```bash
   kubectl apply -f pod.yml
   kubectl apply -f service.yml
   ```

3. **Verificação do status dos recursos:**
   ```bash
   kubectl get pods -o wide
   kubectl get services meu-service
   ```

4. **Acesso à aplicação:**
   ```bash
   minikube service meu-service
   ```
