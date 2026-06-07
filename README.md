# 🚀 WordPress en Kubernetes

## Migración del stack de WordPress desde Docker Compose a Kubernetes con GitOps y ArgoCD.


## 📋 Índice

- [Acerca del proyecto](#acerca-del-proyecto)
- [Arquitectura](#arquitectura)
- [Tecnologías](#tecnologías)
- [Requisitos](#requisitos)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Instalación manual](#instalación-manual)
- [GitOps con ArgoCD](#gitops-con-argocd)
- [Servicios y puertos](#servicios-y-puertos)
- [Comandos útiles](#comandos-útiles)
- [Próximos pasos](#próximos-pasos)
- [Licencia](#licencia)


## 🎯 Acerca del proyecto

Este proyecto es la migración completa del stack de WordPress desde Docker Compose a Kubernetes. Incluye:

- 🐬 MySQL 8 - Base de datos relacional
- 🌐 WordPress - CMS con 3 réplicas
- 🗄️ phpMyAdmin - Gestión visual de la base de datos
- ⚡ Redis - Caché en memoria para WordPress
- 🔄 **ArgoCD** - GitOps para despliegue continuo
- 📊 Grafana - (Próximamente) Monitoreo y visualización


## 🏗️ Arquitectura

```text
                    🌐 INTERNET
                         │
                    [Traefik]
                   (Ingress)
                    │     │
             80     │     │   80
                │   │     │   │
          [WordPress]     [phpMyAdmin]
          (3 réplicas)     (1 réplica)
                │               │
                └───────┬───────┘
                        │
                  [redis-service]
                       │
                  [mysql-service]
                       │
                     [MySQL]
                   (1 réplica)
                       │
                [volumen persistente]

                    ┌─────────────┐
                    │   ArgoCD    │
                    │  (GitOps)   │
                    └─────────────┘
                         │
                         ▼
                    GitHub Repo
```


## 🛠️ Tecnologías

| Categoría | Tecnologías |
|-----------|-------------|
| Orquestación | Kubernetes |
| Containerización | Docker |
| GitOps | ArgoCD |
| Base de datos | MySQL 8 |
| Caché | Redis Alpine |
| Frontend | WordPress, phpMyAdmin |
| Monitoreo | Grafana (próximamente) |
| Ingress Controller | Traefik |
| Almacenamiento | PersistentVolumeClaims |


## ✅ Requisitos

- Docker Desktop con Kubernetes habilitado
- kubectl instalado
- Helm (para instalar Traefik)
- 4GB+ de RAM disponibles para el cluster

Verificar Kubernetes:

```bash
kubectl cluster-info
kubectl get nodes
```


## 📁 Estructura del proyecto

```text
k8s-wordpress/
│
├── argocd-application.yaml      # Configuración de ArgoCD
├── secrets-example.yaml          # Template de secretos
├── README.md
│
└── k8s/
    ├── mysql/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── pvc.yaml
    │
    ├── wordpress/
    │   ├── namespace.yaml
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   ├── ingress.yaml
    │   └── pvc.yaml
    │
    ├── phpmyadmin/
    │   ├── deploy.yaml
    │   ├── service.yaml
    │   └── ingress.yaml
    │
    └── redis/
        └── pvc.yaml
        ├── service.yaml
        ├── deploy.yaml

```


## 🚀 Instalación manual

### 1. Crear el Namespace

```bash
kubectl apply -f k8s/wordpress/namespace.yaml
```

### 2. Aplicar el Secret

```bash
cp secrets-example.yaml secrets.yaml
# Editar secrets.yaml con tus valores reales (base64)
kubectl apply -f secrets.yaml -n wordpress-stack
```

### 3. Aplicar los PersistentVolumeClaims

```bash
kubectl apply -f k8s/mysql/pvc.yaml -n wordpress-stack
kubectl apply -f k8s/wordpress/pvc.yaml -n wordpress-stack
kubectl apply -f k8s/redis/pvc.yaml -n wordpress-stack
```

### 4. Aplicar los Deployments

```bash
kubectl apply -f k8s/mysql/deployment.yaml -n wordpress-stack
kubectl apply -f k8s/wordpress/deployment.yaml -n wordpress-stack
kubectl apply -f k8s/phpmyadmin/deploy.yaml -n wordpress-stack
kubectl apply -f k8s/redis/deploy.yaml -n wordpress-stack
```

### 5. Aplicar los Services

```bash
kubectl apply -f k8s/mysql/service.yaml -n wordpress-stack
kubectl apply -f k8s/wordpress/service.yaml -n wordpress-stack
kubectl apply -f k8s/phpmyadmin/service.yaml -n wordpress-stack
kubectl apply -f k8s/redis/service.yaml -n wordpress-stack
```

### 6. Aplicar los Ingress (para Traefik)

```bash
kubectl apply -f k8s/wordpress/ingress.yaml -n wordpress-stack
kubectl apply -f k8s/phpmyadmin/ingress.yaml -n wordpress-stack
```

### 7. Verificar que todo está corriendo

```bash
kubectl get pods -n wordpress-stack
kubectl get svc -n wordpress-stack
kubectl get ingress -n wordpress-stack
kubectl get pvc -n wordpress-stack
```

Resultado esperado:

```text
NAME                                     READY   STATUS
mysql-deployment-xxxxxxxxxx-xxxxx        1/1     Running
wordpress-deployment-xxxxxxxxxx-xxxxx    1/1     Running
wordpress-deployment-xxxxxxxxxx-xxxxx    1/1     Running
wordpress-deployment-xxxxxxxxxx-xxxxx    1/1     Running
phpmyadmin-deploy-xxxxxxxxxx-xxxxx       1/1     Running
redis-deployment-xxxxxxxxxx-xxxxx        1/1     Running
```


## 🔄 GitOps con ArgoCD

### ¿Qué es ArgoCD?

ArgoCD es una herramienta de despliegue continuo (GitOps) que sincroniza automáticamente tu cluster Kubernetes con la configuración almacenada en Git.

### Instalación de ArgoCD

```bash
# Crear namespace
kubectl create namespace argocd

# Instalar ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Configurar modo inseguro (para pruebas)
kubectl patch deployment argocd-server -n argocd --type='json' -p='[
  {"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--insecure"}
]'

# Crear Ingress para ArgoCD
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
spec:
  ingressClassName: traefik
  rules:
  - host: argocd.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 80
EOF

# Obtener contraseña inicial
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
echo ""
```

### Configurar la Aplicación en ArgoCD

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: wordpress-stack
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/MarioAran/k8s-wordpress
    targetRevision: main
    path: .
    directory:
      recurse: true
  destination:
    server: https://kubernetes.default.svc
    namespace: wordpress-stack
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

```bash
# Aplicar la configuración
kubectl apply -f argocd-application.yaml

# Verificar
kubectl get applications -n argocd
```

### Flujo de trabajo GitOps

```text
1. Developer hace git push
         ↓
2. GitHub Actions construye imagen
         ↓
3. ArgoCD detecta cambios (cada 3min)
         ↓
4. ArgoCD sincroniza manifiestos
         ↓
5. Kubernetes actualiza los pods
```


## 🌐 Servicios y puertos

| Servicio | Tipo | Puerto interno | Puerto externo | Dominio |
|----------|------|----------------|----------------|---------|
| wordpress-service | ClusterIP | 80 | 30080 (vía Traefik) | wordpress.local |
| phpmyadmin-service | ClusterIP | 80 | 30080 (vía Traefik) | phpmyadmin.local |
| mysql-service | ClusterIP | 3306 | - | - |
| redis-service | ClusterIP | 6379 | - | - |
| argocd-server | ClusterIP | 80 | 30080 (vía Traefik) | argocd.local |

### Accesos

| Aplicación | URL | Credenciales |
|-------------|-----|---------------|
| WordPress | http://wordpress.local:30080 | Configuración inicial |
| phpMyAdmin | http://phpmyadmin.local:30080 | root + contraseña MySQL |
| ArgoCD | http://argocd.local:30080 | admin + (contraseña) |

### Configurar hosts en Mac/Linux

```bash
sudo tee -a /etc/hosts <<EOF
192.168.1.59  wordpress.local
192.168.1.59  phpmyadmin.local
192.168.1.59  argocd.local
EOF
```


## 🔧 Comandos útiles

```bash
# Ver todos los pods en el namespace
kubectl get pods -n wordpress-stack

# Ver todos los servicios
kubectl get svc -n wordpress-stack

# Ver ingres
kubectl get ingress -n wordpress-stack

# Ver logs de un deployment
kubectl logs -f deployment/wordpress-deployment -n wordpress-stack

# Escalar WordPress a 5 réplicas
kubectl scale deployment wordpress-deployment --replicas=5 -n wordpress-stack

# Acceder a un pod específico
kubectl exec -it deployment/wordpress-deployment -n wordpress-stack -- bash

# Ver uso de recursos
kubectl top pods -n wordpress-stack

# Sincronizar ArgoCD manualmente
argocd app sync wordpress-stack

# Ver aplicaciones de ArgoCD
kubectl get applications -n argocd

# Eliminar todo el stack
kubectl delete namespace wordpress-stack
kubectl delete namespace argocd  # Opcional
```


## 📝 Próximos pasos (Pendientes)

- [ ] **Grafana** - Implementar monitoreo del cluster
- [ ] **Cert-manager** - Certificados SSL automáticos
- [ ] **Prometheus** - Métricas del cluster
- [ ] **Horizontal Pod Autoscaling** - Auto-escalado de WordPress
- [ ] **Velero** - Backups del cluster


## 📄 Licencia

MIT - Libre para aprender y reutilizar.


---

Hecho con ☕, 🐳 y 🔄 por Mario
