# 🚀 WordPress en Kubernetes

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


## ✅ Requisitos

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
├── secrets.yaml
│
├── mysql/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── pvc.yaml
│
├── wordpress/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── pvc.yaml
│
├── phpmyadmin/
│   ├── deployment.yaml
│   └── service.yaml
│
├── redis/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── pvc.yaml
│
└── grafana/                 [PENDIENTE]
   ├── deployment.yaml
   └── service.yaml
```


# 🚀 Instalación

## 1. Aplicar el Secret

```bash
kubectl apply -f secrets.yaml
```

## 2. Aplicar los PersistentVolumeClaims

```bash
kubectl apply -f mysql/pvc.yaml
kubectl apply -f wordpress/pvc.yaml
kubectl apply -f redis/pvc.yaml
```

## 3. Aplicar los Deployments

```bash
kubectl apply -f mysql/deployment.yaml
kubectl apply -f wordpress/deployment.yaml
kubectl apply -f phpmyadmin/deployment.yaml
kubectl apply -f redis/deployment.yaml
```

## 4. Aplicar los Services

```bash
kubectl apply -f mysql/service.yaml
kubectl apply -f wordpress/service.yaml
kubectl apply -f phpmyadmin/service.yaml
kubectl apply -f redis/service.yaml
```

## 5. Verificar que todo está corriendo

```bash
kubectl get pods
kubectl get svc
kubectl get pvc
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


## 🌐 Servicios y puertos

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


## 🔧 Comandos útiles

```bash
Ver todos los pods
kubectl get pods

Ver todos los servicios
kubectl get svc

Ver logs de un deployment
kubectl logs -f deployment/wordpress-deployment

Escalar WordPress a 5 réplicas
kubectl scale deployment wordpress-deployment --replicas=5

Acceder a un pod específico
kubectl exec -it deployment/wordpress-deployment -- bash

Ver uso de recursos
kubectl top pods

Eliminar todo el stack
kubectl delete -f mysql/
kubectl delete -f wordpress/
kubectl delete -f phpmyadmin/
kubectl delete -f redis/
kubectl delete -f secrets.yaml
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