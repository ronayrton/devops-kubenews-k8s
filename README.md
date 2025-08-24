## ☸️ KubeNews com Kubernetes – Maratona DevOps + IA
Este repositório contém o Lab do Dia 2 da Maratona DevOps + IA com Fabricio Veronez, onde aplicamos na prática o uso de Kubernetes para orquestrar a aplicação KubeNews, promovendo escalabilidade, resiliência e automação declarativa.



##  Objetivos

- Criar e configurar um cluster Kubernetes local e na nuvem.
- Implantar uma aplicação em containers com escalabilidade e resiliência.
- Utilizar `kubectl` para gerenciar objetos do cluster.
- Entender a arquitetura do Kubernetes.
- Testar serviços do tipo LoadBalancer.
- Simular cenários de falha e recuperação automática (resiliência).
- Utilizar o Ask Gordon para análises do cluster.



##  Arquitetura Kubernetes

###  Cluster (Conjunto de máquinas)
- **Control Plane**
  - API Server
  - etcd
  - Scheduler
  - Controller Manager
- **Worker Node**
  - Kubelet
  - Kube-Proxy
  - Container Runtime (ContainerD, CRI-O)



##  Tecnologias Utilizadas

- Kubernetes (kubectl)
- Digital Ocean (Kubernetes as a Service)
- Minikube / Kind (para testes locais)
- Chocolatey (Instalação no Windows)
- Visual Studio Code



##  Componentes criados
| Objeto Kubernetes | Função                           |
| ----------------- | -------------------------------- |
| Deployment        | Gerencia pods e réplicas         |
| ReplicaSet        | Garante número constante de pods |
| Pod               | Unidade mínima de execução       |
| Service           | Expõe pods para acesso externo   |

 

##  Estrutura do Projeto devops-kubenews-k8s
```plaintext
devops-kubenews-k8s/
│
├── k8s/                         # Arquivos de definição Kubernetes (YAML)
│   └── deployment.yaml
├── src/                         # Código-fonte da aplicação
├── README.md                    # Documentação detalhada

```



##  Executando localmente com Minikube
```bash
# Inicia um cluster Kubernetes local usando o Minikube
minikube start  

# Aplica todos os manifests (YAML) que estão dentro da pasta "k8s/"
# Geralmente contém Deployments, Services, ConfigMaps etc.
kubectl apply -f k8s/  

# Exibe a URL de acesso externo para o Service chamado "kube-news-service"
# O Minikube cria um túnel local para acessar o serviço rodando no cluster
minikube service kube-news-service
```
Acesse via: http://localhost:PORTA


##  Executando na nuvem com DigitalOcean

1. Crie um cluster com pelo menos 2vCPU e 2GB RAM

2. Baixe o arquivo .kube/config e salve em:
  - Linux/macOS: ~/.kube/config
  - Windows: C:\Users\SEU_USUARIO\.kube\config

3. Deploy da Aplicação
```bash
kubectl apply -f deployment.yaml
kubectl get all
```



##  Manifesto deployment.yaml
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

##  Manifesto service.yaml
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

##  Escalabilidade e Resiliência
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

##  Rollback de Deploy
```bash
kubectl rollout history deployment kube-news-deployment
kubectl rollout undo deployment kube-news-deployment
```

##  Melhorias do Ambiente - Correções e Otimizações

### **Problemas Identificados e Soluções**

#### **1. Erro de Conexão com Banco de Dados**
**Problema:**
- Pods da aplicação em `CrashLoopBackOff`
- Erro: `ConnectionRefusedError: connect ECONNREFUSED 10.99.225.23:5432`
- Service do PostgreSQL apontando para IP incorreto

**Solução:**
- Recriação do service do PostgreSQL com selector correto
- Novo IP: `10.102.88.120:5432`
- Aplicação consegue conectar no banco

#### **2. Falta de Health Checks**
**Problema:**
- Deployments sem `livenessProbe` e `readinessProbe`
- Kubernetes não conseguia verificar saúde dos pods
- Tráfego sendo roteado para pods não saudáveis

**Solução:**
```yaml
# PostgreSQL Health Checks
livenessProbe:
  exec:
    command:
    - pg_isready
    - -U
    - kubedevnews
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  exec:
    command:
    - pg_isready
    - -U
    - kubedevnews
  initialDelaySeconds: 5
  periodSeconds: 5

# Aplicação Health Checks
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

#### **3. Falta de Persistência de Dados**
**Problema:**
- PostgreSQL sem volume persistente
- Dados perdidos a cada reinicialização do pod

**Solução:**
```yaml
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

# Volume Mount no PostgreSQL
volumeMounts:
- name: postgres-storage
  mountPath: /var/lib/postgresql/data
volumes:
- name: postgres-storage
  persistentVolumeClaim:
    claimName: postgres-pvc
```

#### **4. Selector Incorreto no Service**
**Problema:**
- Service com selector `tier: backend` que não existia
- Roteamento falhando para o PostgreSQL

**Solução:**
```yaml
selector:
  app: postgres  #Selector correto
```

### **Arquivos de Deployment Separados**

Para facilitar manutenção, criamos arquivos individuais:

- `k8s/postgres-deployment.yaml` - Deployment do PostgreSQL
- `k8s/postgres-pvc.yaml` - Volume persistente
- `k8s/postgres-service.yaml` - Service do PostgreSQL
- `k8s/kube-news-service.yaml` - Service da aplicação
- `k8s/app-deployment.yaml` - Deployment da aplicação

### **Comandos de Aplicação**

```bash
# Aplicar cada recurso separadamente
kubectl apply -f k8s/postgres-pvc.yaml
kubectl apply -f k8s/postgres-deployment.yaml
kubectl apply -f k8s/postgres-service.yaml
kubectl apply -f k8s/kube-news-service.yaml
kubectl apply -f k8s/app-deployment.yaml
Kubectl apply -f k8s
```

# Verificar status
```bash
kubectl get pods
kubectl get services
kubectl logs <nome-do-pod>
```

### **Benefícios das Melhorias**

- **Alta Disponibilidade**: Health checks garantem pods saudáveis
- **Persistência**: Dados preservados entre reinicializações
- **Conectividade**: Service funcionando corretamente
- **Monitoramento**: Probes para observabilidade
- **Resiliência**: Recuperação automática de falhas

##  Cleanup Ambiente Local

### **Limpeza Completa do Ambiente**

Para remover todos os recursos criados no cluster Kubernetes:

```bash
# Remover deployments
kubectl delete -f k8s/app-deployment.yaml
kubectl delete -f k8s/postgres-deployment.yaml
```
# Remover services
kubectl delete -f k8s/postgres-service.yaml
kubectl delete -f k8s/kube-news-service.yaml

# Remover volume persistente (ATENÇÃO: dados serão perdidos)
kubectl delete -f k8s/postgres-pvc.yaml
```

### **Verificação da Limpeza**

```bash
# Verificar se todos os recursos foram removidos
kubectl get all
kubectl get pvc
kubectl get pv

# Verificar se não há pods órfãos
kubectl get pods --all-namespaces
```

### **Limpeza Rápida (Todos os Recursos)**

```bash
# Remover todos os recursos de uma vez
kubectl delete -f k8s/

# Ou deletar por tipo
kubectl delete deployment --all
kubectl delete service --all
kubectl delete pvc --all
```

### **Reset Completo do Cluster (Minikube)**

```bash
# Parar o cluster
minikube stop

# Deletar o cluster
minikube delete

# Iniciar um novo cluster limpo
minikube start
```

### **Notas Importantes**

- **Cuidado**: Deletar PVCs remove permanentemente os dados do banco
- **Rolling Update**: Deployments fazem rolling update, não delete imediato
- **Verificação**: Sempre verifique se todos os recursos foram removidos
- **Backup**: Faça backup dos dados antes de limpar se necessário


##  Autor
https://www.linkedin.com/in/ronayrton-rocha-13a872a8/
