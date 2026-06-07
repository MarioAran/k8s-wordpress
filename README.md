# 🚀 WordPress en Kubernetes

## Migración del stack de WordPress desde Docker Compose a Kubernetes.


## 📋 Índice

- [Acerca del proyecto](#acerca-del-proyecto)
- [Arquitectura](#arquitectura)
- [Tecnologías](#tecnologías)
- [Requisitos](#requisitos)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Instalación](#instalación)
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
- 📊 Grafana - (Próximamente) Monitoreo y visualización


## 🏗️ Arquitectura

```
                    🌐 INTERNET
                         │
                    [NodePort]
                    │         │
             30080  │         │  30081
                │   │         │   │
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
```


## 🛠️ Tecnologías

- | Categoría | Tecnologías |
- |-----------|-------------|
- | Orquestación | Kubernetes |
- | Containerización | Docker |
- | Base de datos | MySQL 8 |
- | Caché | Redis Alpine |
- | Frontend | WordPress, phpMyAdmin |
- | Monitoreo | Grafana (próximamente) |
- | Almacenamiento | PersistentVolumeClaims |


## ✅ Requisitos

- Docker Desktop con Kubernetes habilitado
- kubectl instalado
- 4GB+ de RAM disponibles para el cluster

Verificar Kubernetes:

```bash
kubectl cluster-info
kubectl get nodes
```


## 📁 Estructura del proyecto

```
k8s-wordpress/
│
├── argocd-application.yaml
├── secrets-example.yaml         # Template de secretos (renombrar a secrets.yaml)
│
├── k8s/
│   ├── mysql/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── pvc.yaml
│   │
│   ├── wordpress/
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── pvc.yaml
│   │
│   ├── phpmyadmin/
│   │   ├── deploy.yaml
│   │   └── service.yaml
│   │
│   └── redis/
│       ├── deploy.yaml
│       ├── service.yaml
│       └── pvc.yaml
│
└── grafana/                     [PENDIENTE]
   ├── deployment.yaml
   └── service.yaml
```


# 🚀 Instalación

## 1. Crear el Namespace

```bash
kubectl apply -f k8s/wordpress/namespace.yaml
```

## 2. Aplicar el Secret

```bash
cp secrets-example.yaml secrets.yaml
# Editar secrets.yaml con tus valores reales (base64)
kubectl apply -f secrets.yaml -n wordpress-stack
```

## 3. Aplicar los PersistentVolumeClaims

```bash
kubectl apply -f k8s/mysql/pvc.yaml -n wordpress-stack
kubectl apply -f k8s/wordpress/pvc.yaml -n wordpress-stack
kubectl apply -f k8s/redis/pvc.yaml -n wordpress-stack
```

## 4. Aplicar los Deployments

```bash
kubectl apply -f k8s/mysql/deployment.yaml -n wordpress-stack
kubectl apply -f k8s/wordpress/deploy.yaml -n wordpress-stack
kubectl apply -f k8s/phpmyadmin/deploy.yaml -n wordpress-stack
kubectl apply -f k8s/redis/deploy.yaml -n wordpress-stack
```

## 5. Aplicar los Services

```bash
kubectl apply -f k8s/mysql/service.yaml -n wordpress-stack
kubectl apply -f k8s/wordpress/service.yaml -n wordpress-stack
kubectl apply -f k8s/phpmyadmin/service.yaml -n wordpress-stack
kubectl apply -f k8s/redis/service.yaml -n wordpress-stack
```

## 6. Verificar que todo está corriendo

```bash
kubectl get pods -n wordpress-stack
kubectl get svc -n wordpress-stack
kubectl get pvc -n wordpress-stack
```

Resultado esperado:

```
NAME                                     READY   STATUS
mysql-deployment-xxxxxxxxxx-xxxxx        1/1     Running
wordpress-deployment-xxxxxxxxxx-xxxxx    1/1     Running
wordpress-deployment-xxxxxxxxxx-xxxxx    1/1     Running
wordpress-deployment-xxxxxxxxxx-xxxxx    1/1     Running
phpmyadmin-deploy-xxxxxxxxxx-xxxxx       1/1     Running
redis-deployment-xxxxxxxxxx-xxxxx        1/1     Running
```


## 🌐 Servicios y puertos

- | Servicio | Tipo | Puerto interno | Puerto externo |
- |----------|------|----------------|----------------|
- | wordpress-service | NodePort | 80 | 30080 |
- | phpmyadmin-service | NodePort | 80 | 30081 |
- | mysql-service | ClusterIP | 3306 | - |
- | redis-service | ClusterIP | 6379 | - |

## Accesos

- | Aplicación | URL |
- |-------------|-----|
- | WordPress | http://localhost:30080 |
- | phpMyAdmin | http://localhost:30081 |
- | Grafana | (próximamente) |


## 🔧 Comandos útiles

```bash
# Ver todos los pods en el namespace
kubectl get pods -n wordpress-stack

# Ver todos los servicios
kubectl get svc -n wordpress-stack

# Ver logs de un deployment
kubectl logs -f deployment/wordpress-deployment -n wordpress-stack

# Escalar WordPress a 5 réplicas
kubectl scale deployment wordpress-deployment --replicas=5 -n wordpress-stack

# Acceder a un pod específico
kubectl exec -it deployment/wordpress-deployment -n wordpress-stack -- bash

# Ver uso de recursos
kubectl top pods -n wordpress-stack

# Eliminar todo el stack
kubectl delete namespace wordpress-stack
```


# 📝 Próximos pasos (Pendientes)

## 🔲 Grafana

Falta implementar el deployment y service de Grafana para monitoreo del cluster.

Estructura pendiente:

```
grafana/
├── deployment.yaml
└── service.yaml
```


# 📄 Licencia

MIT - Libre para aprender y reutilizar.

---

Hecho con ☕ y 🐳 por Mario

[⬆ Volver arriba]