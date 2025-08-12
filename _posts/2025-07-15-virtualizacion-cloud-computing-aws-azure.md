---
title: "Virtualización y Cloud Computing: De VMware a AWS, Arquitecturas para el Futuro"
date: 2025-07-15 11:45:00 +0100
categories: [Cloud Computing, Virtualización]
tags: [virtualization, cloud, aws, azure, vmware, docker, kubernetes, infrastructure, devops]
---

# Virtualización y Cloud Computing: De VMware a AWS, Arquitecturas para el Futuro

La **virtualización** y el **cloud computing** han transformado radicalmente la forma en que diseñamos, desplegamos y gestionamos la infraestructura de TI. Desde los centros de datos tradicionales hasta las arquitecturas serverless, esta evolución ha democratizado el acceso a recursos computacionales y ha habilitado la innovación a escala global.

## Fundamentos de la Virtualización

### ¿Qué es la Virtualización?

La **virtualización** es una tecnología que permite crear versiones virtuales de recursos físicos como servidores, sistemas operativos, dispositivos de almacenamiento o recursos de red. Esta abstracción permite un uso más eficiente de los recursos hardware y mayor flexibilidad operacional.

### Tipos de Virtualización

#### 1. Virtualización de Servidores

```bash
# Ejemplo con VirtualBox
# Crear una VM
VBoxManage createvm --name "Ubuntu-Server" --register

# Configurar memoria y storage
VBoxManage modifyvm "Ubuntu-Server" --memory 2048 --vram 128
VBoxManage createhd --filename "Ubuntu-Server.vdi" --size 20480

# Adjuntar storage
VBoxManage storagectl "Ubuntu-Server" --name "SATA Controller" --add sata
VBoxManage storageattach "Ubuntu-Server" --storagectl "SATA Controller" \
  --port 0 --device 0 --type hdd --medium "Ubuntu-Server.vdi"
```

#### 2. Virtualización de Contenedores

```dockerfile
# Dockerfile para aplicación web
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

#### 3. Virtualización de Red

```yaml
# Software Defined Network con Docker
version: '3.8'
services:
  web:
    image: nginx
    networks:
      - frontend
      - backend
  api:
    image: node:18
    networks:
      - backend
      - database
  db:
    image: postgres:13
    networks:
      - database

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
  database:
    driver: bridge
    internal: true  # Red aislada
```

### Hypervisors: El Corazón de la Virtualización

#### Tipo 1 (Bare Metal)

**VMware vSphere/ESXi**
```bash
# Comandos básicos de ESXi
# Listar VMs
vim-cmd vmsvc/getallvms

# Obtener estado de VM
vim-cmd vmsvc/power.getstate <vmid>

# Encender VM
vim-cmd vmsvc/power.on <vmid>

# Crear snapshot
vim-cmd vmsvc/snapshot.create <vmid> "snapshot-name" "description"
```

**Microsoft Hyper-V**
```powershell
# PowerShell para Hyper-V
# Crear nueva VM
New-VM -Name "TestServer" -MemoryStartupBytes 2GB -NewVHDPath "C:\VMs\TestServer.vhdx" -NewVHDSizeBytes 40GB

# Configurar VM
Set-VM -Name "TestServer" -ProcessorCount 2
Add-VMNetworkAdapter -VMName "TestServer" -SwitchName "External Switch"

# Iniciar VM
Start-VM -Name "TestServer"
```

#### Tipo 2 (Hosted)

**VirtualBox**
```bash
# Gestión de VMs con VBoxManage
# Listar VMs
VBoxManage list vms

# Iniciar VM en modo headless
VBoxManage startvm "Ubuntu-Server" --type headless

# Crear snapshot
VBoxManage snapshot "Ubuntu-Server" take "clean-install"

# Clonar VM
VBoxManage clonevm "Ubuntu-Server" --name "Ubuntu-Server-Clone" --register
```

## Evolución hacia Cloud Computing

### Modelos de Servicio en la Nube

#### Infrastructure as a Service (IaaS)

```bash
# AWS EC2 - Crear instancia
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --count 1 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-903004f8 \
  --subnet-id subnet-6e7f829e \
  --user-data file://my-user-data.txt

# Azure VM - Crear con CLI
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --size Standard_B1s
```

#### Platform as a Service (PaaS)

```yaml
# Google App Engine - app.yaml
runtime: nodejs18

env_variables:
  NODE_ENV: production
  DATABASE_URL: your-database-url

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.7
```

```json
// Azure App Service - Configuración
{
  "name": "mywebapp",
  "location": "East US",
  "properties": {
    "serverFarmId": "/subscriptions/.../serverfarms/myplan",
    "siteConfig": {
      "nodeVersion": "18.x",
      "appSettings": [
        {"name": "NODE_ENV", "value": "production"},
        {"name": "DATABASE_URL", "value": "your-connection-string"}
      ]
    }
  }
}
```

#### Software as a Service (SaaS)

Ejemplos: Microsoft 365, Google Workspace, Salesforce, Slack

### Modelos de Despliegue

#### 1. Nube Pública

**Ventajas:**
- Escalabilidad ilimitada
- Costo inicial bajo
- Mantenimiento gestionado
- Disponibilidad global

**Desventajas:**
- Menor control
- Preocupaciones de seguridad
- Costos variables

#### 2. Nube Privada

```yaml
# OpenStack - Configuración básica
# nova.conf
[DEFAULT]
transport_url = rabbit://guest:guest@controller:5672/
my_ip = 10.0.0.11
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api

[database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = PLACEMENT_PASS
```

#### 3. Nube Híbrida

```yaml
# Ejemplo de conectividad híbrida con Terraform
# AWS VPN Gateway
resource "aws_vpn_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags = {
    Name = "main-vpn-gateway"
  }
}

# Conexión VPN
resource "aws_vpn_connection" "main" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.main.id
  type                = "ipsec.1"
  static_routes_only  = true
}

# Rutas para el tráfico híbrido
resource "aws_vpn_connection_route" "office" {
  vpn_connection_id      = aws_vpn_connection.main.id
  destination_cidr_block = "192.168.1.0/24"
}
```

## Principales Proveedores de Cloud

### Amazon Web Services (AWS)

#### Servicios Fundamentales

```bash
# EC2 - Elastic Compute Cloud
# Listar instancias
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]'

# S3 - Simple Storage Service
# Crear bucket
aws s3 mb s3://my-unique-bucket-name

# Subir archivo
aws s3 cp myfile.txt s3://my-unique-bucket-name/

# RDS - Relational Database Service
aws rds create-db-instance \
  --db-instance-identifier mydbinstance \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password mypassword \
  --allocated-storage 20
```

#### Infrastructure as Code con CloudFormation

```yaml
# cloudformation-template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Web application infrastructure'

Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues: [t3.micro, t3.small, t3.medium]

Resources:
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0abcdef1234567890
      InstanceType: !Ref InstanceType
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for web server
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0

  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Scheme: internet-facing
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      Subnets:
        - subnet-12345678
        - subnet-87654321

Outputs:
  InstanceId:
    Description: Instance ID of the web server
    Value: !Ref WebServerInstance
  LoadBalancerDNS:
    Description: DNS name of the load balancer
    Value: !GetAtt ApplicationLoadBalancer.DNSName
```

### Microsoft Azure

#### Servicios Principales

```bash
# Azure CLI - Operaciones básicas
# Crear grupo de recursos
az group create --name myResourceGroup --location eastus

# Crear VM
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Crear App Service
az appservice plan create \
  --name myAppServicePlan \
  --resource-group myResourceGroup \
  --sku B1 \
  --is-linux

az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myUniqueAppName \
  --runtime "NODE|18-lts"
```

#### ARM Templates

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myVM"
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_B2s"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-11-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "azureuser",
          "linuxConfiguration": {
            "disablePasswordAuthentication": true,
            "ssh": {
              "publicKeys": [
                {
                  "path": "/home/azureuser/.ssh/authorized_keys",
                  "keyData": "[parameters('sshPublicKey')]"
                }
              ]
            }
          }
        }
      }
    }
  ]
}
```

### Google Cloud Platform (GCP)

#### Servicios Esenciales

```bash
# gcloud CLI - Operaciones básicas
# Configurar proyecto
gcloud config set project my-project-id

# Crear instancia de Compute Engine
gcloud compute instances create my-instance \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=ubuntu-2004-lts \
  --image-project=ubuntu-os-cloud

# Desplegar aplicación en App Engine
gcloud app deploy app.yaml

# Crear cluster de Kubernetes
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=e2-medium
```

## Containerización y Orquestación

### Docker en la Nube

```yaml
# docker-compose.yml para producción
version: '3.8'
services:
  web:
    image: myapp:latest
    ports:
      - "80:8080"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  database:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    deploy:
      placement:
        constraints: [node.role == manager]

volumes:
  postgres_data:
```

### Kubernetes en la Nube

#### Amazon EKS

```yaml
# eks-cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-west-2

nodeGroups:
  - name: ng-1
    instanceType: t3.medium
    desiredCapacity: 3
    minSize: 1
    maxSize: 5
    volumeSize: 20
    ssh:
      allow: true
      publicKeyName: my-key-pair

managedNodeGroups:
  - name: managed-ng-1
    instanceType: t3.medium
    minSize: 1
    maxSize: 5
    desiredCapacity: 3
    labels:
      role: worker
    tags:
      Environment: production
```

```bash
# Crear cluster EKS
eksctl create cluster -f eks-cluster.yaml

# Configurar kubectl
aws eks update-kubeconfig --region us-west-2 --name my-cluster
```

#### Google GKE

```bash
# Crear cluster GKE con autoescalado
gcloud container clusters create my-gke-cluster \
  --zone us-central1-a \
  --machine-type e2-medium \
  --num-nodes 3 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 10 \
  --enable-autorepair \
  --enable-autoupgrade

# Obtener credenciales
gcloud container clusters get-credentials my-gke-cluster --zone us-central1-a
```

#### Azure AKS

```bash
# Crear cluster AKS
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3 \
  --node-vm-size Standard_B2s \
  --enable-addons monitoring \
  --generate-ssh-keys

# Configurar kubectl
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## Arquitecturas Cloud-Native

### Microservicios

```yaml
# microservices-architecture.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: microservices

---
# User Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: microservices
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: mycompany/user-service:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: user-db-url
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
# API Gateway
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-gateway
  namespace: microservices
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - api.mycompany.com
    secretName: api-tls
  rules:
  - host: api.mycompany.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
```

### Serverless Computing

#### AWS Lambda

```python
# lambda_function.py
import json
import boto3
from decimal import Decimal

def lambda_handler(event, context):
    """
    Función Lambda para procesar pedidos
    """
    try:
        # Obtener datos del evento
        order_data = json.loads(event['body'])
        
        # Procesar pedido
        result = process_order(order_data)
        
        # Respuesta exitosa
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps(result, default=decimal_default)
        }
        
    except Exception as e:
        # Manejo de errores
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json'
            },
            'body': json.dumps({
                'error': str(e)
            })
        }

def process_order(order_data):
    """Lógica de procesamiento de pedidos"""
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('Orders')
    
    # Guardar pedido en DynamoDB
    response = table.put_item(
        Item={
            'order_id': order_data['order_id'],
            'customer_id': order_data['customer_id'],
            'items': order_data['items'],
            'total': Decimal(str(order_data['total'])),
            'status': 'pending'
        }
    )
    
    return {
        'message': 'Order processed successfully',
        'order_id': order_data['order_id']
    }

def decimal_default(obj):
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError
```

```yaml
# serverless.yml (Serverless Framework)
service: order-processing-api

provider:
  name: aws
  runtime: python3.9
  region: us-east-1
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:PutItem
        - dynamodb:GetItem
        - dynamodb:UpdateItem
      Resource:
        - arn:aws:dynamodb:us-east-1:*:table/Orders

functions:
  processOrder:
    handler: lambda_function.lambda_handler
    events:
      - http:
          path: /orders
          method: post
          cors: true
    environment:
      ORDERS_TABLE: Orders

resources:
  Resources:
    OrdersTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: Orders
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: order_id
            AttributeType: S
        KeySchema:
          - AttributeName: order_id
            KeyType: HASH
```

## Infrastructure as Code (IaC)

### Terraform

```hcl
# main.tf - Infraestructura multi-cloud
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# AWS Provider
provider "aws" {
  region = var.aws_region
}

# Azure Provider
provider "azurerm" {
  features {}
}

# AWS VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "main-vpc"
    Environment = var.environment
  }
}

# AWS Subnets
resource "aws_subnet" "public" {
  count = length(var.availability_zones)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "public"
  }
}

# AWS Application Load Balancer
resource "aws_lb" "main" {
  name               = "main-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id

  enable_deletion_protection = false

  tags = {
    Environment = var.environment
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                = "web-asg"
  vpc_zone_identifier = aws_subnet.public[*].id
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"

  min_size         = 2
  max_size         = 10
  desired_capacity = 3

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "web-instance"
    propagate_at_launch = true
  }
}

# Azure Resource Group
resource "azurerm_resource_group" "main" {
  name     = "myapp-resources"
  location = var.azure_location
}

# Azure Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "myapp-network"
  address_space       = ["10.1.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}
```

### Pulumi

```python
# __main__.py - Pulumi con Python
import pulumi
import pulumi_aws as aws
import pulumi_azure_native as azure
from pulumi_kubernetes import Provider as K8sProvider
import pulumi_kubernetes as k8s

# Configuración
config = pulumi.Config()
environment = config.get("environment", "dev")

# AWS EKS Cluster
cluster = aws.eks.Cluster(
    "my-cluster",
    version="1.24",
    role_arn=cluster_role.arn,
    vpc_config=aws.eks.ClusterVpcConfigArgs(
        subnet_ids=[subnet.id for subnet in subnets],
    ),
    depends_on=[
        cluster_role_policy_attachment,
        cluster_service_role_policy_attachment,
    ],
)

# Node Group
node_group = aws.eks.NodeGroup(
    "my-node-group",
    cluster_name=cluster.name,
    node_role_arn=node_group_role.arn,
    subnet_ids=[subnet.id for subnet in private_subnets],
    scaling_config=aws.eks.NodeGroupScalingConfigArgs(
        desired_size=3,
        max_size=6,
        min_size=1,
    ),
    instance_types=["t3.medium"],
    depends_on=[
        node_group_policy_attachment,
        node_group_cni_policy_attachment,
        node_group_registry_policy_attachment,
    ],
)

# Kubernetes Provider
k8s_provider = K8sProvider(
    "k8s-provider",
    kubeconfig=cluster.kubeconfig_json,
)

# Kubernetes Deployment
app_deployment = k8s.apps.v1.Deployment(
    "app-deployment",
    metadata=k8s.meta.v1.ObjectMetaArgs(
        name="my-app",
        labels={"app": "my-app"},
    ),
    spec=k8s.apps.v1.DeploymentSpecArgs(
        replicas=3,
        selector=k8s.meta.v1.LabelSelectorArgs(
            match_labels={"app": "my-app"},
        ),
        template=k8s.core.v1.PodTemplateSpecArgs(
            metadata=k8s.meta.v1.ObjectMetaArgs(
                labels={"app": "my-app"},
            ),
            spec=k8s.core.v1.PodSpecArgs(
                containers=[
                    k8s.core.v1.ContainerArgs(
                        name="app",
                        image="nginx:latest",
                        ports=[k8s.core.v1.ContainerPortArgs(container_port=80)],
                    )
                ],
            ),
        ),
    ),
    opts=pulumi.ResourceOptions(provider=k8s_provider),
)

# Exports
pulumi.export("cluster_name", cluster.name)
pulumi.export("kubeconfig", cluster.kubeconfig_json)
```

## Monitoreo y Observabilidad

### CloudWatch (AWS)

```python
# cloudwatch_monitoring.py
import boto3
import json
from datetime import datetime, timedelta

class CloudWatchMonitoring:
    def __init__(self):
        self.cloudwatch = boto3.client('cloudwatch')
        self.logs = boto3.client('logs')
        
    def create_custom_metric(self, namespace, metric_name, value, unit='Count'):
        """Crear métrica personalizada"""
        self.cloudwatch.put_metric_data(
            Namespace=namespace,
            MetricData=[
                {
                    'MetricName': metric_name,
                    'Value': value,
                    'Unit': unit,
                    'Timestamp': datetime.utcnow()
                }
            ]
        )
        
    def create_alarm(self, alarm_name, metric_name, namespace, threshold):
        """Crear alarma"""
        self.cloudwatch.put_metric_alarm(
            AlarmName=alarm_name,
            ComparisonOperator='GreaterThanThreshold',
            EvaluationPeriods=2,
            MetricName=metric_name,
            Namespace=namespace,
            Period=300,
            Statistic='Average',
            Threshold=threshold,
            ActionsEnabled=True,
            AlarmActions=[
                'arn:aws:sns:us-east-1:123456789012:my-topic'
            ],
            AlarmDescription='Alarm when metric exceeds threshold'
        )
        
    def query_logs(self, log_group, query, start_time, end_time):
        """Consultar logs con CloudWatch Insights"""
        response = self.logs.start_query(
            logGroupName=log_group,
            startTime=int(start_time.timestamp()),
            endTime=int(end_time.timestamp()),
            queryString=query
        )
        
        return response['queryId']
```

### Prometheus y Grafana

```yaml
# prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    rule_files:
      - "alert_rules.yml"

    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              - alertmanager:9093

    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']

      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)

---
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
        - name: config
          mountPath: /etc/prometheus
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'
          - '--web.enable-lifecycle'
      volumes:
      - name: config
        configMap:
          name: prometheus-config
```

## Seguridad en la Nube

### Mejores Prácticas

#### Identity and Access Management (IAM)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket/*",
        "arn:aws:s3:::my-app-bucket"
      ]
    },
    {
      "Sid": "AllowEC2Describe",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeImages",
        "ec2:DescribeKeyPairs"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Network Security

```yaml
# Network Security Group (Azure)
apiVersion: v1
kind: ConfigMap
metadata:
  name: network-security-rules
data:
  rules.json: |
    {
      "securityRules": [
        {
          "name": "AllowHTTPS",
          "properties": {
            "protocol": "Tcp",
            "sourcePortRange": "*",
            "destinationPortRange": "443",
            "sourceAddressPrefix": "Internet",
            "destinationAddressPrefix": "*",
            "access": "Allow",
            "priority": 100,
            "direction": "Inbound"
          }
        },
        {
          "name": "DenyAll",
          "properties": {
            "protocol": "*",
            "sourcePortRange": "*",
            "destinationPortRange": "*",
            "sourceAddressPrefix": "*",
            "destinationAddressPrefix": "*",
            "access": "Deny",
            "priority": 4096,
            "direction": "Inbound"
          }
        }
      ]
    }
```

## Conclusión

La virtualización y el cloud computing han revolucionado la tecnología, ofreciendo:

### Beneficios Clave

1. **Escalabilidad**: Recursos bajo demanda
2. **Flexibilidad**: Múltiples opciones de despliegue
3. **Eficiencia**: Mejor utilización de recursos
4. **Innovación**: Acceso a tecnologías avanzadas
5. **Economía**: Modelos de pago por uso

### Consideraciones Futuras

- **Edge Computing**: Procesamiento cercano al usuario
- **Quantum Computing**: Computación cuántica en la nube
- **Sustainability**: Computación verde y eficiente
- **Multi-Cloud**: Estrategias distribuidas
- **AI/ML Integration**: Inteligencia artificial nativa

### Recomendaciones

1. **Evalúa** tus necesidades específicas
2. **Inicia** con proyectos pequeños
3. **Implementa** seguridad desde el diseño
4. **Monitorea** costos continuamente
5. **Mantente** actualizado con tendencias

El futuro de la tecnología está en la nube, y dominar estos conceptos es esencial para cualquier profesional de TI en la era digital.

---
**Andrés Nuñez - t4ifi**
