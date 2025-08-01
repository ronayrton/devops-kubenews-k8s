# maratona-devops-ia

Link para ter $ 200 na Digital Ocean

https://m.do.co/c/a939ecc60dfa


# Estrutura Inicial

```yaml
name: CI-CD

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
    CI:
        runs-on: ubuntu-latest
        steps:
            - run: echo "Obter o código"
            - run: echo "Executar o Docker Build"
            - run: echo "Enviar a Imagem Docker para o Docker Hub"

    CD:
        runs-on: ubuntu-latest
        
        steps:
            - run: echo "Obter o código"
            - run: echo "Configurar o Kubeconfig"
            - run: echo "Executar o apply"
```        


# Grafana

Obter a senha do admin
```        
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```        



### ✅ Tarefas do Projeto

Etapas                    
---------------------------
1. Criar `namespace.yaml`    
2. Criar `deployment.yaml`   
3. Criar `service.yaml`      
4. Testar local com Minikube 
5. Criar pasta `docs/`       
7. Escrever `README.md`      
8. Print do dashboard        



### 📦 Estrutura do Projeto devops-kubenews-k8s

```plaintext
devops-kubenews-k8s/
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── namespace.yaml
│
├── src/
│   └── [aplicação em Node ou outro, como no dia 1]
│
├── Dockerfile
├── .dockerignore
├── README.md
└── docs/
    ├── apresentacao.pdf
    └── imagens/
```

