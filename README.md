# Objetivo

O objetivo deste projeto foi construir uma arquitetura Kubernetes funcional e evoluir gradualmente em conceitos de Cloud Native e DevOps.

O laboratório começou com estudos básicos de Ingress e evoluiu para um ambiente completo contendo:

* WordPress
* MySQL
* Autoscaling Horizontal (HPA)
* Autoscaling Vertical (VPA)
* Monitoramento com Prometheus
* Observabilidade com Grafana
* Health Checks
* Segmentação de tráfego utilizando Ingress

---

# Arquitetura

```text
Browser
    │
    ▼
Port Forward / Ingress (Traefik)
    │
    ▼
Service WordPress
    │
    ▼
Pods WordPress
    │
    ▼
Service MySQL
    │
    ▼
Pod MySQL
```

---

# Tecnologias Utilizadas

* Kubernetes
* K3d / K3s
* kubectl
* Helm
* WordPress
* MySQL
* Prometheus
* Grafana
* Horizontal Pod Autoscaler (HPA)
* Vertical Pod Autoscaler (VPA)

---

# Services

Foram utilizados Services do tipo `ClusterIP` para permitir a comunicação interna entre os componentes da aplicação.

Exemplo:

```text
WordPress → mysql:3306
```

Mesmo que o Pod do MySQL seja recriado e receba um novo endereço IP, a comunicação continua funcionando através do Service.

---

# Secrets e ConfigMaps

As credenciais da aplicação foram externalizadas utilizando Kubernetes Secrets.

### Secrets

Responsáveis por armazenar informações sensíveis:

* Usuário do banco
* Senha do banco
* Credenciais do WordPress

### ConfigMaps

Responsáveis por armazenar configurações não sensíveis:

* Nome da aplicação
* Ambiente
* Configurações gerais

---

# Health Checks

Foram implementadas estratégias de verificação de saúde da aplicação.

## Readiness Probe

Responsável por verificar se a aplicação está pronta para receber tráfego.

Enquanto a aplicação não estiver pronta, o Kubernetes não encaminha requisições para o Pod.

## Liveness Probe

Responsável por verificar se a aplicação continua saudável após a inicialização.

Caso o container pare de responder, o Kubernetes realiza a reinicialização automática.

---

# HPA — Horizontal Pod Autoscaler

Foi configurado um HPA para escalar horizontalmente os Pods do WordPress com base na utilização de CPU.

Durante os testes foi possível observar:

```text
1 Pod → 3 Pods
```

Após a redução da carga:

```text
3 Pods → 1 Pod
```

Demonstrando a capacidade de escalabilidade automática da aplicação.

---

# VPA — Vertical Pod Autoscaler

Foi utilizado VPA para análise e recomendação de recursos.

Modo configurado:

```yaml
updateMode: "Off"
```

Neste modo o VPA não altera automaticamente os Pods.

Seu objetivo é fornecer recomendações de:

* CPU
* Memória

permitindo ajustes manuais mais precisos.

---

# Observabilidade

O monitoramento foi implementado utilizando Prometheus e Grafana.

## Prometheus

Responsável pela coleta das métricas do cluster e das aplicações.

## Grafana

Responsável pela visualização das métricas através de dashboards.

Foi possível acompanhar:

* Utilização de CPU
* Utilização de memória
* Requests e Limits
* Comportamento do HPA
* Estado dos Pods em tempo real

---

# Segmentação de Tráfego com Ingress

Como evolução do laboratório, foi implementada uma arquitetura com separação entre tráfego administrativo e tráfego público.

## WordPress Frontend

Deployment responsável pelo tráfego público.

* Deployment: `wordpress-frontend`
* Réplicas: `4 Pods`
* Service: `wordpress-frontend`

Host configurado:

```text
seusite.com.br
```

## WordPress Admin

Deployment responsável pelo tráfego administrativo.

* Deployment: `wordpress-admin`
* Réplicas: `1 Pod`
* Service: `wordpress-admin`

Host configurado:

```text
admin.seusite.com.br
```

---

## Fluxo de Roteamento

```text
admin.seusite.com.br
        ↓
Traefik Ingress
        ↓
wordpress-admin Service
        ↓
1 Pod WordPress

seusite.com.br
        ↓
Traefik Ingress
        ↓
wordpress-frontend Service
        ↓
4 Pods WordPress
```

---

## Endpoints

Os Endpoints são criados automaticamente pelo Kubernetes a partir dos Selectors dos Services.

Exemplo:

```text
wordpress-frontend
├── 10.42.2.69:80
├── 10.42.2.70:80
├── 10.42.2.71:80
└── 10.42.2.72:80
```

Esses Endpoints representam os Pods disponíveis atrás do Service e são utilizados para distribuição das requisições.

---

# Ingress Labs

Os manifests presentes no diretório `ingress/` foram utilizados inicialmente para estudos de:

* Ingress
* Roteamento HTTP
* Traefik
* NGINX Ingress Controller

Esses manifests representam a evolução inicial do aprendizado antes da implementação da arquitetura principal baseada em WordPress e MySQL.

---

# Como Subir o Ambiente

## 1. Iniciar o Cluster

```bash
k3d cluster start lab-k8s
```

## 2. Aplicar os Manifests

```bash
kubectl apply -f namespace/

kubectl apply -R -f configmap/

kubectl apply -R -f mysql/

kubectl apply -R -f wordpress/

kubectl apply -R -f ingress/
```

---

# Validação

## Verificar Pods

```bash
kubectl get pods -n dev
```

## Verificar Services

```bash
kubectl get svc -n dev
```

## Verificar HPA

```bash
kubectl get hpa -n dev
```

## Verificar VPA

```bash
kubectl get vpa -n dev
```

## Verificar Ingress

```bash
kubectl get ingress -n dev
```

## Verificar Endpoints

```bash
kubectl get endpoints -n dev
```


### Segmentação de Tráfego com Kubernetes

O objetivo deste desafio foi separar o tráfego administrativo e público da aplicação utilizando recursos nativos do Kubernetes.

Para isso, foram criados dois Deployments independentes:

### WordPress Frontend

Responsável pelo tráfego público da aplicação.

* Deployment: `wordpress-frontend`
* Réplicas: `4 Pods`
* Service: `wordpress-frontend`

### WordPress Admin

Responsável pelo acesso administrativo da aplicação.

* Deployment: `wordpress-admin`
* Réplicas: `1 Pod`
* Service: `wordpress-admin`

---

### Roteamento com Ingress (Traefik)

Foi configurado um Ingress utilizando o Traefik para realizar o roteamento baseado em hostname.

```text
admin.seusite.com.br
        ↓
wordpress-admin Service
        ↓
1 Pod WordPress

seusite.com.br
        ↓
wordpress-frontend Service
        ↓
4 Pods WordPress
```

---

### Como os Services encontram os Pods

Cada Service utiliza labels e selectors para localizar automaticamente os Pods correspondentes.

Exemplo:

```yaml
selector:
  app: wordpress-frontend
```

O Kubernetes identifica todos os Pods com essa label e cria automaticamente os Endpoints do Service.

---

### Endpoints

Os Endpoints representam os Pods disponíveis atrás de um Service.

Exemplo:

```text
wordpress-frontend
├── 10.42.2.69:80
├── 10.42.2.70:80
├── 10.42.2.71:80
└── 10.42.2.72:80
```

Esses Endpoints são utilizados pelo Kubernetes para distribuir as requisições entre os Pods do frontend.

---

### Resultado

Com essa arquitetura foi possível:

* Separar o tráfego administrativo do tráfego público.
* Escalar o frontend independentemente da área administrativa.
* Implementar roteamento baseado em hostname utilizando Ingress e Traefik.
* Demonstrar a comunicação entre Deployments, Services, Endpoints e Ingress dentro do Kubernetes.

