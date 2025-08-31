# Descomplicando a Stack DevOps no Google Cloud Platform

> DevOps: A Ponte entre o Código e o Deploy

## Sobre o Treinamento

Este é um treinamento prático e imersivo que ensina como construir e gerenciar uma pipeline completa de DevOps no Google Cloud Platform. Você aprenderá desde a conteinerização até o deploy automatizado em Kubernetes, passando por Infrastructure as Code com Terraform.

### O que você vai aprender

- Conteinerização com Docker
- Infrastructure as Code com Terraform  
- Orquestração com Kubernetes no GKE
- Monitoramento e observabilidade
- CI/CD com GitHub Actions

### Para quem é este treinamento

- **Desenvolvedores** que querem entender o ciclo de vida de suas aplicações
- **Profissionais de Infraestrutura** que desejam modernizar suas práticas
- **Profissionais de TI** buscando conhecimentos em DevOps

## Pré-requisitos

### Conhecimento
- Linux e linha de comando (obrigatório)
- Docker (desejável)
- Programação básica (desejável)
- Terraform e Kubernetes (desejável)

### Ferramentas Necessárias
- [Google Cloud Platform Account](https://cloud.google.com/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Terraform](https://www.terraform.io/downloads)
- [Google Cloud CLI](https://cloud.google.com/sdk/docs/install)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Estrutura do Projeto

```
devops-app-gcp/
├── package.json              # Dependências Node.js
├── server.js                # Aplicação Express
├── public/
│   └── index.html           # Interface web
├── Dockerfile               # Container configuration
├── terraform/               # Infrastructure as Code
│   ├── main.tf                 # Recursos principais
│   ├── variables.tf            # Variáveis
│   ├── outputs.tf              # Outputs
│   ├── kubernetes.tf           # Cluster GKE
│   └── startup-script.sh       # Script de inicialização
└── k8s/                     # Kubernetes manifests
    ├── namespace.yaml
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

## Quick Start

### 1. Clone o repositório
```bash
git clone https://github.com/seu-usuario/devops-app-gcp.git
cd devops-app-gcp
```

### 2. Configure o ambiente
```bash
# Instalar dependências
npm install

# Configurar GCP
gcloud auth login
export PROJECT_ID="seu-projeto-$(date +%s)"
gcloud projects create $PROJECT_ID
gcloud config set project $PROJECT_ID

# Habilitar APIs necessárias
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable cloudbuild.googleapis.com
```

### 3. Build da aplicação
```bash
# Build da imagem Docker
docker build -t devops-app-gcp:v1.0 .

# Testar localmente
docker run -p 3000:3000 devops-app-gcp:v1.0
```

### 4. Deploy da infraestrutura
```bash
cd terraform
terraform init
terraform apply
```

## Dia 1: Docker e Containerização

### Aplicação Base

**package.json:**
```json
{
  "name": "devops-app-gcp",
  "version": "1.0.0",
  "description": "Aplicação de exemplo para treinamento DevOps no GCP",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

**server.js:**
```javascript
const express = require('express');
const path = require('path');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.static('public'));

app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

app.get('/api/info', (req, res) => {
  res.json({
    app: 'DevOps App GCP',
    version: '1.0.0',
    environment: process.env.NODE_ENV || 'development',
    pod: process.env.HOSTNAME || 'localhost'
  });
});

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

**public/index.html:**
```html



    
    
    DevOps App - GCP
    
        body { 
            font-family: Arial, sans-serif; 
            margin: 40px; 
            background: #f5f5f5; 
        }
        .container { 
            max-width: 600px; 
            margin: 0 auto; 
            background: white; 
            padding: 20px; 
            border-radius: 8px; 
            box-shadow: 0 2px 10px rgba(0,0,0,0.1); 
        }
        .status { 
            padding: 10px; 
            border-radius: 4px; 
            margin: 10px 0; 
        }
        .healthy { 
            background: #d4edda; 
            color: #155724; 
            border: 1px solid #c3e6cb; 
        }
    


    
        DevOps App no GCP
        Esta aplicação demonstra o pipeline completo DevOps no Google Cloud Platform.
        
        
            Status: Aplicação funcionando corretamente!
        
        
        Endpoints disponíveis:
        
            /health - Health check
            /api/info - Informações da aplicação
        
        
        
            fetch('/api/info')
                .then(response => response.json())
                .then(data => {
                    document.querySelector('.container').innerHTML += 
                        `<div class="status healthy">
                            <strong>Versão:</strong> ${data.version}<br>
                            <strong>Ambiente:</strong> ${data.environment}<br>
                            <strong>Pod/Host:</strong> ${data.pod}
                        </div>`;
                });
        
    


```

### Dockerfile Otimizado

```dockerfile
# Usar imagem oficial Node.js LTS
FROM node:18-alpine

# Definir diretório de trabalho
WORKDIR /app

# Copiar arquivos de dependências
COPY package*.json ./

# Instalar dependências
RUN npm ci --only=production && npm cache clean --force

# Copiar código da aplicação
COPY . .

# Criar usuário não-root para segurança
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Mudar propriedade dos arquivos
RUN chown -R nextjs:nodejs /app
USER nextjs

# Expor porta
EXPOSE 3000

# Definir variáveis de ambiente
ENV NODE_ENV=production
ENV PORT=3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (res) => { process.exit(res.statusCode === 200 ? 0 : 1) })"

# Comando para iniciar a aplicação
CMD ["node", "server.js"]
```

### Comandos Docker

```bash
# Build e execução
docker build -t devops-app-gcp:v1.0 .
docker run -d --name app -p 3000:3000 devops-app-gcp:v1.0

# Debugging
docker logs app
docker exec -it app sh

# Limpeza
docker stop app && docker rm app
```

## Dia 2: Terraform e Infrastructure as Code

### Arquivos Terraform

**terraform/variables.tf:**
```hcl
variable "project_id" {
  description = "ID do projeto GCP"
  type        = string
}

variable "region" {
  description = "Região do GCP"
  type        = string
  default     = "us-central1"
}

variable "zone" {
  description = "Zona do GCP"
  type        = string
  default     = "us-central1-a"
}

variable "machine_type" {
  description = "Tipo da máquina"
  type        = string
  default     = "e2-micro"
}
```

**terraform/main.tf:**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.84"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
  zone    = var.zone
}

# Rede VPC
resource "google_compute_network" "vpc_network" {
  name                    = "devops-network"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "devops-subnet"
  ip_cidr_range = "10.0.0.0/24"
  region        = var.region
  network       = google_compute_network.vpc_network.id
}

# Firewall rules
resource "google_compute_firewall" "allow_http" {
  name    = "allow-http"
  network = google_compute_network.vpc_network.name

  allow {
    protocol = "tcp"
    ports    = ["80", "3000", "22"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web-server"]
}

# Instância da VM
resource "google_compute_instance" "app_server" {
  name         = "devops-app-server"
  machine_type = var.machine_type
  zone         = var.zone

  tags = ["web-server"]

  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
      size  = 20
    }
  }

  network_interface {
    network    = google_compute_network.vpc_network.id
    subnetwork = google_compute_subnetwork.subnet.id
    
    access_config {
      # IP público efêmero
    }
  }

  metadata = {
    startup-script = file("${path.module}/startup-script.sh")
  }

  service_account {
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
  }
}
```

**terraform/outputs.tf:**
```hcl
output "instance_ip" {
  description = "IP público da instância"
  value       = google_compute_instance.app_server.network_interface[0].access_config[0].nat_ip
}

output "app_url" {
  description = "URL da aplicação"
  value       = "http://${google_compute_instance.app_server.network_interface[0].access_config[0].nat_ip}"
}

output "cluster_name" {
  description = "Nome do cluster GKE"
  value       = google_container_cluster.devops_cluster.name
}

output "cluster_location" {
  description = "Localização do cluster"
  value       = google_container_cluster.devops_cluster.location
}
```

**terraform/startup-script.sh:**
```bash
#!/bin/bash

# Atualizar sistema
sudo apt-get update

# Aguardar Docker inicializar
sleep 10

# Criar aplicação temporária para demonstração
mkdir -p /tmp/app/public
cat > /tmp/app/package.json << 'EOF'
{
  "name": "devops-app-gcp",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": { "start": "node server.js" },
  "dependencies": { "express": "^4.18.2" }
}
EOF

cat > /tmp/app/server.js << 'EOF'
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.static('public'));

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

app.get('/api/info', (req, res) => {
  res.json({
    app: 'DevOps App GCP',
    version: '1.0.0',
    environment: process.env.NODE_ENV || 'development',
    instance: process.env.HOSTNAME || 'localhost'
  });
});

app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
EOF

cat > /tmp/app/public/index.html << 'EOF'

DevOps App - GCP

DevOps App rodando no GCP!
Aplicação containerizada em VM do Google Compute Engine
Health Check | Info API

EOF

# Build da imagem
cd /tmp/app
sudo docker build -t devops-app-gcp:v1.0 .

# Executar container
sudo docker run -d \
  --name devops-app \
  --restart unless-stopped \
  -p 80:3000 \
  -p 3000:3000 \
  devops-app-gcp:v1.0

echo "Aplicação iniciada em $(date)" | sudo tee -a /var/log/app-startup.log
```

**terraform/terraform.tfvars:**
```hcl
project_id = "SEU_PROJECT_ID_AQUI"
region     = "us-central1"
zone       = "us-central1-a"
```

### Comandos Terraform

```bash
# Navegar para diretório terraform
cd terraform

# Inicializar Terraform
terraform init

# Planejar mudanças
terraform plan

# Aplicar infraestrutura
terraform apply

# Testar aplicação
APP_IP=$(terraform output -raw instance_ip)
curl http://$APP_IP/health
```

## Dia 3: Kubernetes no GKE

### Cluster GKE no Terraform

**terraform/kubernetes.tf:**
```hcl
# Cluster GKE
resource "google_container_cluster" "devops_cluster" {
  name     = "devops-cluster"
  location = var.region

  node_locations = [
    "${var.region}-a",
    "${var.region}-b"
  ]

  remove_default_node_pool = true
  initial_node_count       = 1

  network    = google_compute_network.vpc_network.name
  subnetwork = google_compute_subnetwork.subnet.name

  master_auth {
    client_certificate_config {
      issue_client_certificate = false
    }
  }
}

# Node Pool
resource "google_container_node_pool" "devops_nodes" {
  name       = "devops-node-pool"
  location   = var.region
  cluster    = google_container_cluster.devops_cluster.name
  node_count = 2

  node_config {
    preemptible  = true  # Custos reduzidos
    machine_type = "e2-medium"
    
    oauth_scopes = [
      "https://www.googleapis.com/auth/logging.write",
      "https://www.googleapis.com/auth/monitoring",
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    tags = ["gke-node"]
  }
  
  autoscaling {
    min_node_count = 1
    max_node_count = 4
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }
}
```

### Kubernetes Manifests

**k8s/namespace.yaml:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: devops-app
  labels:
    name: devops-app
```

**k8s/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-app
  namespace: devops-app
  labels:
    app: devops-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: devops-app
  template:
    metadata:
      labels:
        app: devops-app
    spec:
      containers:
      - name: devops-app
        image: gcr.io/PROJECT_ID/devops-app-gcp:v1.0  # Substituir PROJECT_ID
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi" 
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

**k8s/service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: devops-app-service
  namespace: devops-app
  labels:
    app: devops-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
    name: http
  selector:
    app: devops-app
```

**k8s/ingress.yaml:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: devops-app-ingress
  namespace: devops-app
  annotations:
    kubernetes.io/ingress.class: "gce"
spec:
  rules:
  - http:
      paths:
      - path: /*
        pathType: ImplementationSpecific
        backend:
          service:
            name: devops-app-service
            port:
              number: 80
```

### Deploy no GKE

```bash
# Aplicar mudanças no Terraform
terraform apply

# Configurar kubectl
gcloud container clusters get-credentials devops-cluster --region=us-central1

# Verificar conexão
kubectl cluster-info

# Build e push da imagem para Container Registry
docker tag devops-app-gcp:v1.0 gcr.io/$PROJECT_ID/devops-app-gcp:v1.0
docker push gcr.io/$PROJECT_ID/devops-app-gcp:v1.0

# Atualizar manifests com PROJECT_ID correto
sed -i "s/PROJECT_ID/$PROJECT_ID/g" k8s/deployment.yaml

# Aplicar manifests
kubectl apply -f k8s/

# Verificar deployments
kubectl get all -n devops-app

# Obter IP externo do LoadBalancer
kubectl get service devops-app-service -n devops-app

# Aguardar IP externo e testar
EXTERNAL_IP=$(kubectl get service devops-app-service -n devops-app -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$EXTERNAL_IP/health
```

## Comandos Úteis

### Docker
```bash
# Build e tag
docker build -t devops-app-gcp:v1.0 .
docker tag devops-app-gcp:v1.0 gcr.io/$PROJECT_ID/devops-app-gcp:v1.0

# Push para Container Registry
docker push gcr.io/$PROJECT_ID/devops-app-gcp:v1.0

# Executar localmente
docker run -p 3000:3000 devops-app-gcp:v1.0

# Debugging
docker exec -it  sh
docker logs 

# Limpeza local
docker stop 
docker rm 
docker rmi devops-app-gcp:v1.0
```

### Terraform
```bash
# Workflow básico
terraform init
terraform plan
terraform apply
terraform destroy

# Verificar estado
terraform show
terraform state list

# Outputs
terraform output
terraform output -raw instance_ip

# Formatação e validação
terraform fmt
terraform validate
```

### Kubernetes
```bash
# Contexts e clusters
kubectl config get-contexts
kubectl config use-context 

# Recursos
kubectl get all -n devops-app
kubectl describe deployment devops-app -n devops-app

# Logs
kubectl logs -f deployment/devops-app -n devops-app
kubectl logs  -n devops-app --previous

# Port forwarding
kubectl port-forward service/devops-app-service 8080:80 -n devops-app

# Scaling
kubectl scale deployment devops-app --replicas=5 -n devops-app

# Rollout
kubectl rollout status deployment/devops-app -n devops-app
kubectl rollout history deployment/devops-app -n devops-app
kubectl rollout undo deployment/devops-app -n devops-app
```

### GCP
```bash
# Projetos
gcloud projects list
gcloud config set project 

# Compute Engine
gcloud compute instances list
gcloud compute ssh 

# GKE
gcloud container clusters list
gcloud container clusters get-credentials  --region=

# Container Registry
gcloud container images list
gcloud container images delete 
```

## Monitoramento e Troubleshooting

### Health Checks
```bash
# Verificar aplicação local
curl http://localhost:3000/health
curl http://localhost:3000/api/info

# Verificar no GCP
curl http:///health
curl http:///api/info

# Kubernetes health
kubectl get pods -n devops-app
kubectl top pods -n devops-app
```

### Logs
```bash
# Docker logs
docker logs  --follow

# GCE logs
gcloud compute ssh  --command "sudo journalctl -u docker -f"

# Kubernetes logs
kubectl logs -f deployment/devops-app -n devops-app
kubectl logs  -n devops-app --previous
```

### Debugging
```bash
# Conectar ao container
docker exec -it  sh

# Conectar ao pod
kubectl exec -it  -n devops-app -- sh

# Verificar recursos
kubectl describe pod  -n devops-app
kubectl get events -n devops-app --sort-by='.lastTimestamp'
```

## Issues Comuns

### Docker build falha
```bash
# Verificar Dockerfile
cat Dockerfile

# Build com logs detalhados
docker build --no-cache -t devops-app-gcp:v1.0 .

# Verificar espaço em disco
docker system df
docker system prune
```

### Terraform apply falha
```bash
# Verificar credenciais
gcloud auth list
gcloud config list

# Verificar APIs habilitadas
gcloud services list --enabled

# Verificar quotas
gcloud compute project-info describe --project=$PROJECT_ID
```

### Pods em estado Pending
```bash
# Verificar events
kubectl get events -n devops-app --sort-by='.lastTimestamp'

# Verificar recursos dos nodes
kubectl top nodes
kubectl describe nodes
```

## Limpeza de Recursos

### Importante: Evitar Custos Desnecessários

```bash
# 1. Deletar recursos Kubernetes
kubectl delete namespace devops-app

# 2. Destruir infraestrutura Terraform
cd terraform
terraform destroy -auto-approve

# 3. Deletar imagens do Container Registry
gcloud container images delete gcr.io/$PROJECT_ID/devops-app-gcp:v1.0 --force-delete-tags

# 4. Verificar recursos restantes
gcloud compute instances list
gcloud container clusters list
gcloud compute networks list

# 5. Deletar projeto (opcional - remove tudo)
gcloud projects delete $PROJECT_ID
```

## Próximos Passos

### Após completar este treinamento:

#### 1. CI/CD Pipeline
- Implementar GitHub Actions
- Automatizar build e deploy
- Testes automatizados

#### 2. Monitoramento Avançado  
- Prometheus e Grafana
- Alerting e dashboards
- Application Performance Monitoring (APM)

#### 3. Segurança
- Vulnerability scanning
- Secret management
- RBAC no Kubernetes

#### 4. Otimização
- Resource limits e requests
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)

## Exercícios Práticos

### Exercício 1: Modificar a Aplicação
1. Adicione um novo endpoint `/api/version` que retorna apenas a versão
2. Modifique o health check para incluir informações do sistema
3. Reconstrua a imagem e faça o deploy

### Exercício 2: Otimizar o Terraform
1. Adicione tags padronizados em todos os recursos
2. Crie um IP estático para a VM
3. Configure backup automático do disco

### Exercício 3: Melhorar o Kubernetes
1. Adicione ConfigMaps para configuração
2. Implemente Secrets para dados sensíveis
3. Configure resource quotas no namespace

## Trilha de Treinamentos

1. **Linux Engineering** - Aprofundamento em administração de sistemas
2. **GitOps com ArgoCD** - Continuous delivery declarativa  
3. **Go Programming** - Linguagem popular no ecossistema DevOps

## Recursos Adicionais

### Documentação Oficial
- [Google Cloud Documentation](https://cloud.google.com/docs)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Terraform GCP Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)

### Ferramentas Úteis
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Docker Hub](https://hub.docker.com/)
- [Google Container Registry](https://cloud.google.com/container-registry)
- [Terraform Cloud](https://cloud.hashicorp.com/products/terraform)

### Comunidades
- [GKE Community](https://cloud.google.com/community/)
- [Cloud Native Computing Foundation](https://www.cncf.io/)
- [DevOps Brasil](https://github.com/devops-brasil)

## Contribuição

### Como contribuir:
1. Fork o projeto
2. Crie uma branch: `git checkout -b feature/nova-funcionalidade`
3. Commit suas mudanças: `git commit -m 'Adiciona nova funcionalidade'`
4. Push para a branch: `git push origin feature/nova-funcionalidade`
5. Abra um Pull Request

### Padrões do projeto:
- Use conventional commits
- Teste localmente antes do PR
- Documente mudanças significativas
- Mantenha compatibilidade com versões anteriores

## Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## Agradecimentos

- **LINUXtips** pelo treinamento base
- **Google Cloud** pela plataforma
- **Comunidade DevOps** pelas boas práticas
- **CNCF** pelos projetos open source

---

### Contato

- **Instrutor:** Camilla Martins
- **Organização:** LINUXtips
- **Website:** [linuxtips.io](https://linuxtips.io)
