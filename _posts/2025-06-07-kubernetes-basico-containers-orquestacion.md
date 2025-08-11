---
title: "Kubernetes Básico: Containers y Orquestación para Principiantes"
date: 2025-06-07 09:30:00 +0100
categories: [DevOps, Kubernetes]
tags: [kubernetes, k8s, containers, orchestration, docker, devops, cluster]
---

# Kubernetes Básico: Containers y Orquestación para Principiantes

**Kubernetes** (K8s) es la plataforma de orquestación de contenedores más popular del mundo, desarrollada originalmente por Google y ahora mantenida por la Cloud Native Computing Foundation. Si bien puede parecer complejo al principio, entender sus conceptos fundamentales es esencial para cualquier profesional de DevOps moderno.

## ¿Qué es Kubernetes y Por Qué es Importante?

### El Problema que Resuelve

Imagina que tienes una aplicación web que funciona perfectamente en tu máquina local usando Docker. Ahora necesitas:

- **Ejecutarla en producción** con alta disponibilidad
- **Escalarla** cuando hay más tráfico
- **Actualizarla** sin downtime
- **Monitoreartla** y reiniciarla si falla
- **Balancear la carga** entre múltiples instancias

Hacer esto manualmente es complejo y propenso a errores. **Kubernetes automatiza todas estas tareas**.

### ¿Qué es un Orquestador?

Un **orquestador de contenedores** es como un director de orquesta que coordina múltiples músicos (contenedores) para crear una sinfonía armoniosa (aplicación funcionando).

```bash
# Sin orquestador (manual)
docker run -d --name app1 myapp:v1
docker run -d --name app2 myapp:v1
docker run -d --name app3 myapp:v1
# ¿Qué pasa si app2 se cae? ¿Cómo balanceo la carga?

# Con Kubernetes (automático)
kubectl create deployment myapp --image=myapp:v1 --replicas=3
# K8s se encarga de todo: health checks, load balancing, auto-restart
```

## Arquitectura de Kubernetes

### Componentes del Control Plane (Cerebro del Cluster)

#### 1. API Server
**El corazón de Kubernetes** - todos los comandos pasan por aquí.

```bash
# Cuando ejecutas esto:
kubectl get pods

# El API Server procesa la petición y responde
```

#### 2. etcd
**Base de datos distribuida** que almacena toda la configuración del cluster.

```bash
# etcd almacena información como:
# - ¿Cuántos pods debo tener de cada aplicación?
# - ¿En qué nodos están ejecutándose?
# - ¿Cuáles son las configuraciones de red?
```

#### 3. Scheduler
**Decide dónde ejecutar los pods** basándose en recursos disponibles.

```yaml
# El scheduler considera:
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

#### 4. Controller Manager
**Ejecuta controladores** que mantienen el estado deseado.

```bash
# Si tienes 3 replicas pero solo 2 están funcionando,
# el controller manager creará una nueva automáticamente
```

### Componentes del Nodo Worker

#### 1. kubelet
**Agente que ejecuta en cada nodo** - comunica con el control plane.

```bash
# kubelet es responsable de:
# - Descargar imágenes de contenedores
# - Ejecutar pods
# - Reportar salud del nodo
```

#### 2. kube-proxy
**Maneja la red** y el balanceo de carga.

```bash
# kube-proxy configura reglas de iptables para:
# - Enrutar tráfico a los pods correctos
# - Balancear carga entre replicas
```

#### 3. Container Runtime
**Ejecuta los contenedores** (Docker, containerd, etc.).

```bash
# El runtime pull las imágenes y ejecuta los contenedores
docker pull nginx:latest
docker run nginx:latest
```

## Conceptos Fundamentales

### 1. Pod - La Unidad Básica

Un **Pod** es la unidad más pequeña en Kubernetes. Contiene uno o más contenedores que comparten red y almacenamiento.

```yaml
# pod-simple.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-primer-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.20
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```bash
# Crear el pod
kubectl apply -f pod-simple.yaml

# Ver el pod
kubectl get pods

# Ver detalles
kubectl describe pod mi-primer-pod

# Ver logs
kubectl logs mi-primer-pod

# Eliminar el pod
kubectl delete pod mi-primer-pod
```

### 2. Deployment - Gestión de Aplicaciones

Un **Deployment** gestiona un conjunto de pods idénticos y garantiza que siempre estén ejecutándose.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3  # Queremos 3 copias
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
        livenessProbe:  # Verifica si el pod está vivo
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:  # Verifica si está listo para recibir tráfico
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

```bash
# Crear deployment
kubectl apply -f deployment.yaml

# Ver deployments
kubectl get deployments

# Ver pods creados por el deployment
kubectl get pods -l app=nginx

# Escalar la aplicación
kubectl scale deployment nginx-deployment --replicas=5

# Ver el escalado en tiempo real
kubectl get pods -w

# Actualizar la imagen
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Ver el rollout
kubectl rollout status deployment/nginx-deployment

# Ver historial de versiones
kubectl rollout history deployment/nginx-deployment

# Rollback a versión anterior
kubectl rollout undo deployment/nginx-deployment
```

### 3. Service - Exposición de Aplicaciones

Un **Service** expone un conjunto de pods a la red, proporcionando un punto de acceso estable.

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx  # Conecta con pods que tengan este label
  ports:
  - protocol: TCP
    port: 80        # Puerto del service
    targetPort: 80  # Puerto del contenedor
  type: ClusterIP   # Solo accesible dentro del cluster
```

#### Tipos de Services

```yaml
# ClusterIP (por defecto) - Solo interno
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080

---
# NodePort - Accesible desde fuera del cluster
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Puerto en cada nodo (30000-32767)

---
# LoadBalancer - Para cloud providers
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

```bash
# Crear service
kubectl apply -f service.yaml

# Ver services
kubectl get services

# Describir service
kubectl describe service nginx-service

# Probar conectividad interna
kubectl exec -it <pod-name> -- curl nginx-service

# Port-forward para testing local
kubectl port-forward service/nginx-service 8080:80
# Ahora puedes acceder en http://localhost:8080
```

## Ejemplo Práctico: Aplicación Web Completa

Vamos a desplegar una aplicación web completa con base de datos:

### 1. Base de Datos (MySQL)

```yaml
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password123"
        - name: MYSQL_DATABASE
          value: "webapp"
        - name: MYSQL_USER
          value: "appuser"
        - name: MYSQL_PASSWORD
          value: "apppass"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        emptyDir: {}  # En producción usa PersistentVolume

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
  type: ClusterIP
```

### 2. Aplicación Web

```yaml
# webapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:latest  # En la vida real sería tu aplicación
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: "mysql-service"
        - name: DATABASE_NAME
          value: "webapp"
        - name: DATABASE_USER
          value: "appuser"
        - name: DATABASE_PASSWORD
          value: "apppass"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer  # O NodePort para testing local
```

### 3. Desplegar Todo

```bash
# Aplicar los manifiestos
kubectl apply -f mysql-deployment.yaml
kubectl apply -f webapp-deployment.yaml

# Verificar que todo esté funcionando
kubectl get all

# Ver logs de la aplicación
kubectl logs -l app=webapp

# Verificar conectividad
kubectl exec -it deployment/webapp -- curl webapp-service
```

## ConfigMaps y Secrets

### ConfigMaps - Configuración No Sensible

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app_name: "Mi Aplicación Web"
  log_level: "info"
  max_connections: "100"
  database_host: "mysql-service"
  config.properties: |
    # Configuración de la aplicación
    debug=false
    max_users=1000
    cache_enabled=true
```

### Secrets - Información Sensible

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  # Los valores están en base64
  database_password: YXBwcGFzcw==  # apppass
  api_key: bXlfc2VjcmV0X2FwaV9rZXk=      # my_secret_api_key
```

```bash
# Crear secret desde línea de comandos
kubectl create secret generic app-secrets \
  --from-literal=database_password=apppass \
  --from-literal=api_key=my_secret_api_key

# Ver secrets (los valores no se muestran)
kubectl get secrets
kubectl describe secret app-secrets
```

### Usar ConfigMaps y Secrets en Pods

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-with-config
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp-config
  template:
    metadata:
      labels:
        app: webapp-config
    spec:
      containers:
      - name: webapp
        image: nginx:latest
        env:
        # Variables individuales desde ConfigMap
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: app_name
        # Variables desde Secret
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database_password
        # Cargar todas las variables del ConfigMap
        envFrom:
        - configMapRef:
            name: app-config
        # Montar archivos desde ConfigMap
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

## Comandos Esenciales de kubectl

### Información del Cluster

```bash
# Ver información del cluster
kubectl cluster-info

# Ver nodos del cluster
kubectl get nodes

# Ver información detallada de un nodo
kubectl describe node <node-name>

# Ver recursos disponibles
kubectl top nodes  # Requiere metrics-server
kubectl top pods
```

### Gestión de Recursos

```bash
# Listar recursos
kubectl get <resource>              # pods, services, deployments, etc.
kubectl get all                     # Todos los recursos principales
kubectl get pods -o wide            # Información adicional
kubectl get pods --all-namespaces   # En todos los namespaces

# Describir recursos (muy útil para debugging)
kubectl describe <resource> <name>
kubectl describe pod mi-pod
kubectl describe service mi-service

# Crear recursos
kubectl apply -f archivo.yaml       # Crear/actualizar desde archivo
kubectl create deployment nginx --image=nginx  # Crear directamente

# Editar recursos
kubectl edit deployment nginx        # Abre editor
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'  # Patch específico

# Eliminar recursos
kubectl delete -f archivo.yaml
kubectl delete deployment nginx
kubectl delete pod mi-pod --force --grace-period=0  # Forzar
```

### Debugging y Troubleshooting

```bash
# Ver logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pod
kubectl logs -f <pod-name>                   # Follow (tiempo real)
kubectl logs --previous <pod-name>           # Logs de container anterior

# Ejecutar comandos en pods
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- curl localhost:8080
kubectl exec <pod-name> -- ps aux

# Port forwarding para testing
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80

# Copiar archivos
kubectl cp <pod-name>:/path/to/file /local/path
kubectl cp /local/file <pod-name>:/path/to/destination
```

## Namespaces - Organización y Aislamiento

Los **Namespaces** permiten organizar recursos en grupos lógicos:

```bash
# Ver namespaces
kubectl get namespaces

# Crear namespace
kubectl create namespace desarrollo
kubectl create namespace produccion

# Trabajar en un namespace específico
kubectl get pods -n desarrollo
kubectl apply -f deployment.yaml -n desarrollo

# Configurar namespace por defecto
kubectl config set-context --current --namespace=desarrollo
```

```yaml
# deployment-con-namespace.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
  namespace: desarrollo  # Especificar namespace
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
      - name: app
        image: nginx:latest
```

## Mejores Prácticas para Principiantes

### 1. Organización de Archivos

```bash
# Estructura recomendada
proyecto/
├── manifests/
│   ├── namespaces/
│   │   └── namespace.yaml
│   ├── deployments/
│   │   ├── frontend.yaml
│   │   └── backend.yaml
│   ├── services/
│   │   ├── frontend-service.yaml
│   │   └── backend-service.yaml
│   └── configs/
│       ├── configmap.yaml
│       └── secret.yaml
└── scripts/
    ├── deploy.sh
    └── cleanup.sh
```

### 2. Labels y Selectors Consistentes

```yaml
# Usar labels consistentes
metadata:
  labels:
    app: mi-aplicacion
    version: v1.2.3
    environment: production
    tier: frontend
```

### 3. Resource Limits

```yaml
# Siempre definir requests y limits
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### 4. Health Checks

```yaml
# Implementar probes de salud
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 5. Scripts de Automatización

```bash
#!/bin/bash
# deploy.sh

set -e

echo "Desplegando aplicación..."

# Aplicar manifiestos en orden
kubectl apply -f manifests/namespaces/
kubectl apply -f manifests/configs/
kubectl apply -f manifests/deployments/
kubectl apply -f manifests/services/

echo "Esperando que los pods estén listos..."
kubectl wait --for=condition=ready pod -l app=mi-aplicacion --timeout=300s

echo "Verificando estado del deployment..."
kubectl get all -l app=mi-aplicacion

echo "¡Despliegue completado!"
```

## Herramientas Útiles

### 1. kubectl Autocompletado

```bash
# Para bash
echo 'source <(kubectl completion bash)' >>~/.bashrc

# Para zsh
echo 'source <(kubectl completion zsh)' >>~/.zshrc

# Alias útil
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc
```

### 2. kubectx y kubens

```bash
# Instalar kubectx para cambiar contextos fácilmente
# Ubuntu/Debian
sudo apt install kubectx

# Cambiar entre clusters
kubectx

# Cambiar namespace
kubens desarrollo
```

### 3. k9s - Dashboard Terminal

```bash
# Instalar k9s
curl -sS https://webinstall.dev/k9s | bash

# Ejecutar
k9s
```

## Próximos Pasos

Una vez que domines estos conceptos básicos, puedes explorar:

1. **Persistent Volumes** - Almacenamiento persistente
2. **Ingress Controllers** - Gestión avanzada de tráfico HTTP/HTTPS
3. **Helm** - Gestor de paquetes para Kubernetes
4. **Operators** - Automatización de aplicaciones complejas
5. **Custom Resources** - Extender Kubernetes con tus propios recursos

## Conclusión

Kubernetes puede parecer abrumador al principio, pero dominar estos conceptos básicos te dará una base sólida. Recuerda:

- **Practica** regularmente con clusters locales (minikube, kind)
- **Empieza simple** y ve agregando complejidad gradualmente
- **Lee la documentación oficial** - es excelente
- **Únete a la comunidad** - hay muchos recursos y personas dispuestas a ayudar

Kubernetes es una herramienta poderosa que ha revolucionado la manera en que desplegamos y gestionamos aplicaciones. Con práctica y paciencia, se convertirá en una herramienta indispensable en tu arsenal de DevOps.

---

**Andrés Núñez**  