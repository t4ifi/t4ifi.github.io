---
title: "Contenedores Avanzado: Docker Compose y Kubernetes para DevOps"
date: 2025-06-24 14:30:00 +0100
categories: [DevOps, Contenedores]
tags: [docker, docker-compose, kubernetes, containers, orchestration, microservices, devops]
---

# Contenedores Avanzado: Docker Compose y Kubernetes para DevOps

En el mundo moderno del desarrollo y despliegue de aplicaciones, la **orquestación de contenedores** se ha convertido en una habilidad fundamental. Mientras que Docker nos permite crear y ejecutar contenedores individuales, las aplicaciones reales suelen consistir en múltiples servicios que necesitan trabajar juntos de manera coordinada.

## ¿Qué es la Orquestación de Contenedores?

La **orquestación de contenedores** es el proceso automatizado de desplegar, gestionar, escalar y conectar contenedores. Es como dirigir una orquesta donde cada músico (contenedor) debe tocar en el momento correcto y en armonía con los demás.

### Problemas que Resuelve la Orquestación

1. **Gestión de dependencias**: Asegurar que los servicios se inicien en el orden correcto
2. **Comunicación entre servicios**: Facilitar la comunicación segura entre contenedores
3. **Escalabilidad**: Aumentar o disminuir instancias según la demanda
4. **Alta disponibilidad**: Recuperación automática ante fallos
5. **Balanceo de carga**: Distribuir el tráfico entre múltiples instancias

## Docker Compose: Orquestación Simple y Eficaz

**Docker Compose** es una herramienta que permite definir y ejecutar aplicaciones Docker multi-contenedor usando un archivo YAML.

### Conceptos Fundamentales

#### Archivo docker-compose.yml

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/code
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      
  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Comandos Esenciales de Docker Compose

```bash
# Iniciar todos los servicios
docker-compose up -d

# Ver logs de todos los servicios
docker-compose logs -f

# Escalar un servicio específico
docker-compose up --scale web=3

# Detener y eliminar contenedores
docker-compose down

# Reconstruir imágenes
docker-compose build

# Ver estado de los servicios
docker-compose ps
```

### Ejemplo Práctico: Aplicación Web Completa

```yaml
version: '3.8'
services:
  # Aplicación web principal
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - database
      - cache
    environment:
      - NODE_ENV=production
      - DB_HOST=database
      - REDIS_URL=redis://cache:6379
    volumes:
      - ./uploads:/app/uploads
    networks:
      - app-network
    restart: unless-stopped
    
  # Base de datos
  database:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=secretpassword
      - MYSQL_DATABASE=myapp
      - MYSQL_USER=appuser
      - MYSQL_PASSWORD=userpassword
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    restart: unless-stopped
    
  # Cache Redis
  cache:
    image: redis:7-alpine
    networks:
      - app-network
    restart: unless-stopped
    
  # Proxy reverso
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    networks:
      - app-network
    restart: unless-stopped

networks:
  app-network:
    driver: bridge

volumes:
  mysql_data:
```

## Kubernetes: Orquestación Empresarial

**Kubernetes (K8s)** es una plataforma de orquestación de contenedores que automatiza el despliegue, escalado y gestión de aplicaciones contenerizadas a gran escala.

### Arquitectura de Kubernetes

#### Componentes del Control Plane

1. **API Server**: Punto de entrada para todas las operaciones
2. **etcd**: Base de datos distribuida que almacena el estado del cluster
3. **Controller Manager**: Ejecuta controladores que regulan el estado
4. **Scheduler**: Asigna pods a nodos según recursos disponibles

#### Componentes del Nodo

1. **kubelet**: Agente que ejecuta en cada nodo
2. **kube-proxy**: Maneja la red y el balanceo de carga
3. **Container Runtime**: Docker, containerd, etc.

### Objetos Fundamentales de Kubernetes

#### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
  labels:
    app: web
spec:
  containers:
  - name: web-container
    image: nginx:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.20
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### Comandos Esenciales de kubectl

```bash
# Ver información del cluster
kubectl cluster-info

# Listar nodos
kubectl get nodes

# Crear recursos desde archivo
kubectl apply -f deployment.yaml

# Ver pods
kubectl get pods -o wide

# Ver logs de un pod
kubectl logs pod-name -f

# Ejecutar comandos en un pod
kubectl exec -it pod-name -- /bin/bash

# Escalar deployment
kubectl scale deployment web-deployment --replicas=5

# Ver servicios
kubectl get services

# Describir un recurso
kubectl describe pod pod-name

# Eliminar recursos
kubectl delete -f deployment.yaml
```

### ConfigMaps y Secrets

#### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/myapp"
  redis_url: "redis://cache:6379"
  log_level: "info"
  config.json: |
    {
      "debug": true,
      "max_connections": 100
    }
```

#### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  # Los valores deben estar en base64
  database_password: cGFzc3dvcmQxMjM=
  api_key: YWJjZGVmZ2hpams=
```

### Ejemplo Completo: Aplicación Microservicios

```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: microservices-app

---
# Database Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
  namespace: microservices-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: "myapp"
        - name: POSTGRES_USER
          value: "user"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc

---
# Database Service
apiVersion: v1
kind: Service
metadata:
  name: database-service
  namespace: microservices-app
spec:
  selector:
    app: database
  ports:
  - port: 5432
    targetPort: 5432

---
# API Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: microservices-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: myapp/api:latest
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5

---
# API Service
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: microservices-app
spec:
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

## Mejores Prácticas

### Docker Compose

1. **Use variables de entorno**: Para configuraciones que cambien entre entornos
2. **Versiones específicas**: Evite `latest` en producción
3. **Healthchecks**: Implemente verificaciones de salud
4. **Secrets**: Use Docker secrets para información sensible
5. **Redes**: Cree redes personalizadas para aislar servicios

### Kubernetes

1. **Resource Limits**: Siempre defina limits y requests
2. **Probes**: Implemente liveness y readiness probes
3. **Labels**: Use labels consistentes para organización
4. **Namespaces**: Separe entornos y aplicaciones
5. **RBAC**: Implemente control de acceso basado en roles
6. **Backup**: Haga backup de etcd regularmente

## Comparación: Docker Compose vs Kubernetes

| Aspecto | Docker Compose | Kubernetes |
|---------|----------------|------------|
| **Complejidad** | Simple | Complejo |
| **Escalabilidad** | Limitada | Ilimitada |
| **Alta disponibilidad** | Básica | Avanzada |
| **Ecosistema** | Pequeño | Extenso |
| **Curva de aprendizaje** | Suave | Empinada |
| **Uso ideal** | Desarrollo, pequeñas aplicaciones | Producción empresarial |

## Monitoreo y Observabilidad

### Prometheus y Grafana

```yaml
# Prometheus
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: prometheus-config
          mountPath: /etc/prometheus
      volumes:
      - name: prometheus-config
        configMap:
          name: prometheus-config
```

## Conclusión

La orquestación de contenedores es fundamental en la arquitectura moderna de aplicaciones. **Docker Compose** es perfecto para desarrollo y aplicaciones simples, mientras que **Kubernetes** es la solución para entornos empresariales que requieren escalabilidad y alta disponibilidad.

La elección entre estas herramientas depende de:
- **Tamaño del equipo** y experiencia técnica
- **Complejidad** de la aplicación
- **Requisitos** de escalabilidad y disponibilidad
- **Presupuesto** y recursos disponibles

Dominar ambas tecnologías te permitirá adaptar la solución adecuada a cada escenario específico.

---

**Andrés Núñez**  