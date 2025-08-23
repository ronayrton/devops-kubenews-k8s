☸️ KubeNews com Kubernetes – Maratona DevOps + IA
Este repositório contém o Lab do Dia 2 da Maratona DevOps + IA com Fabricio Veronez, onde aplicamos na prática o uso de Kubernetes para orquestrar a aplicação KubeNews, promovendo escalabilidade, resiliência e automação declarativa.

---

## 📌 Objetivos

- Criar e configurar um cluster Kubernetes local e na nuvem.
- Implantar uma aplicação em containers com escalabilidade e resiliência.
- Utilizar `kubectl` para gerenciar objetos do cluster.
- Entender a arquitetura do Kubernetes.
- Testar serviços do tipo LoadBalancer.
- Simular cenários de falha e recuperação automática (resiliência).
- Utilizar o Ask Gordon para análises do cluster.

---

## ☸️ Arquitetura Kubernetes

### 🔹 Cluster (Conjunto de máquinas)
- **Control Plane**
  - API Server
  - etcd
  - Scheduler
  - Controller Manager
- **Worker Node**
  - Kubelet
  - Kube-Proxy
  - Container Runtime (ContainerD, CRI-O)

---

## 🛠️ Tecnologias Utilizadas

- Kubernetes (kubectl)
- Digital Ocean (Kubernetes as a Service)
- Minikube / Kind (para testes locais)
- Chocolatey (Instalação no Windows)
- Visual Studio Code

---

## 🧱 Componentes criados
| Objeto Kubernetes | Função                           |
| ----------------- | -------------------------------- |
| Deployment        | Gerencia pods e réplicas         |
| ReplicaSet        | Garante número constante de pods |
| Pod               | Unidade mínima de execução       |
| Service           | Expõe pods para acesso externo   |

--- 

## 📦 Estrutura do Projeto devops-kubenews-k8s
```plaintext
devops-kubenews-k8s/
│
├── k8s/                         # Arquivos de definição Kubernetes (YAML)
│   └── deployment.yaml
│
├── src/                         # Código-fonte da aplicação
│
├── README.md                    # Documentação detalhada
└── docs/                        # Prints, apresentações, evidências
```

---

## 🚀 Executando localmente com Minikube
```bash
minikube start
kubectl apply -f k8s/
minikube service kube-news-service
```
Acesse via: http://localhost:PORTA


## ☁️ Executando na nuvem com DigitalOcean

1. Crie um cluster com pelo menos 2vCPU e 2GB RAM

2. Baixe o arquivo .kube/config e salve em:
  - Linux/macOS: ~/.kube/config
  - Windows: C:\Users\SEU_USUARIO\.kube\config

3. Deploy da Aplicação
```bash
kubectl apply -f deployment.yaml
kubectl get all
```
---


## 🧱 Manifesto deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-news-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kube-news-app
  template:
    metadata:
      labels:
        app: kube-news-app
    spec:
      containers:
      - name: kube-news
        image: <sua-imagem>
        ports:
        - containerPort: 8080
```

## 🌐 Criando o Service
- Expondo via LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kube-news-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kube-news-app
```

## 📈 Escalabilidade e Resiliência
**Escalabilidade**
- Aumentar número de réplicas:

```yaml
spec:
  replicas: 10
```

**Resiliência**
- Simular falha:

```bash
kubectl delete pod <nome-do-pod>
```
O pod será recriado automaticamente.

## 🧪 Rollback de Deploy
```bash
kubectl rollout history deployment kube-news-deployment
kubectl rollout undo deployment kube-news-deployment
```

## 🧹 Cleanup
```bash
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
```


## 📊 Análise do Cluster
Usar Ask Gordon para analisar a segurança e performance do cluster Kubernetes.


## ✍️ Autor
https://www.linkedin.com/in/ronayrton-rocha-13a872a8/
