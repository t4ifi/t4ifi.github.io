---
title: "Introducción a las Bases de Datos: Relacionales vs NoSQL - Guía Completa"
date: 2025-05-17 12:00:00 +0000
categories: [Bases de Datos, Fundamentos]
tags: [apuntes, relacionales, no-relacionales, mysql, mariadb, postgresql, mongodb, redis, sql, nosql]
---

## 🎯 ¿Qué es una Base de Datos?

Una **base de datos** es un conjunto organizado de datos relacionados entre sí y almacenados de manera sistemática en una computadora. Estos datos están estructurados para facilitar la **recuperación, gestión y actualización** de información según sea necesario.

> 💡 **Analogía simple:** Imagina una biblioteca. Los libros son los datos, las estanterías son las tablas, y el sistema de catalogación es el SGBD (Sistema Gestor de Base de Datos).

### ¿Por qué son importantes?

Las bases de datos se utilizan en:
- **E-commerce**: Catálogos de productos, usuarios, pedidos
- **Redes sociales**: Perfiles, publicaciones, conexiones
- **Banca**: Cuentas, transacciones, historial crediticio
- **Gobierno**: Registros civiles, sistemas de salud
- **Gaming**: Puntuaciones, perfiles de jugadores, inventarios

<div style="text-align: center;">
  <img src="../assets/img/apuntes/mysql.png" alt="MySQL" style="width: 220px; margin-right: 30px;" />
  <img src="../assets/img/apuntes/mariadb.png" alt="MariaDB" style="width: 160px;" />
</div>

---

## 🏗️ Bases de Datos Relacionales (SQL)

Las bases de datos relacionales organizan los datos en **tablas** (filas y columnas) y utilizan **relaciones** entre estas tablas para representar la información.

### Características principales:
- **Estructura rígida**: Esquema predefinido
- **ACID compliance**: Atomicidad, Consistencia, Aislamiento, Durabilidad
- **SQL**: Lenguaje estándar para consultas
- **Normalización**: Elimina redundancia de datos

### Ejemplo práctico: E-commerce

**Tabla usuarios:**
```sql
CREATE TABLE usuarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Tabla productos:**
```sql
CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(200) NOT NULL,
    precio DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    categoria_id INT,
    FOREIGN KEY (categoria_id) REFERENCES categorias(id)
);
```

**Tabla pedidos:**
```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    fecha_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

### Consultas típicas:

**Obtener pedidos de un usuario:**
```sql
SELECT p.id, p.total, p.fecha_pedido, u.nombre
FROM pedidos p
JOIN usuarios u ON p.usuario_id = u.id
WHERE u.email = 'juan@email.com';
```

**Productos más vendidos:**
```sql
SELECT pr.nombre, SUM(dp.cantidad) as total_vendido
FROM productos pr
JOIN detalle_pedidos dp ON pr.id = dp.producto_id
JOIN pedidos p ON dp.pedido_id = p.id
WHERE p.fecha_pedido >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY pr.id
ORDER BY total_vendido DESC
LIMIT 10;
```

### SGBD Relacionales populares:

| SGBD | Características | Uso típico |
|------|----------------|-------------|
| **MySQL** | Open source, rápido, fácil | Aplicaciones web, WordPress |
| **PostgreSQL** | Avanzado, extensible, JSON | Aplicaciones empresariales |
| **MariaDB** | Fork de MySQL, compatibilidad | Reemplazo drop-in de MySQL |
| **Oracle** | Empresarial, muy robusto | Grandes corporaciones |
| **SQL Server** | Microsoft, integración .NET | Entornos Windows |

---

## 🚀 Bases de Datos NoSQL

Las bases de datos **NoSQL** ("Not Only SQL") no siguen el modelo relacional tradicional. Están diseñadas para manejar **grandes volúmenes** de datos y **diferentes tipos** de datos de manera flexible.

### Tipos de NoSQL:

#### 1. **Documentos** (MongoDB, CouchDB)
Almacenan datos como documentos (JSON, BSON):

```javascript
// Documento de usuario en MongoDB
{
  "_id": ObjectId("..."),
  "nombre": "Juan Pérez",
  "email": "juan@email.com",
  "direcciones": [
    {
      "tipo": "casa",
      "calle": "Av. Principal 123",
      "ciudad": "Madrid"
    },
    {
      "tipo": "trabajo",
      "calle": "Calle Comercial 456", 
      "ciudad": "Madrid"
    }
  ],
  "preferencias": {
    "newsletter": true,
    "notificaciones": false
  }
}
```

**Consulta en MongoDB:**
```javascript
// Buscar usuarios de Madrid
db.usuarios.find({
  "direcciones.ciudad": "Madrid"
});

// Actualizar preferencias
db.usuarios.updateOne(
  { "email": "juan@email.com" },
  { "$set": { "preferencias.newsletter": false } }
);
```

#### 2. **Clave-Valor** (Redis, DynamoDB)
Estructura simple: una clave mapea a un valor:

```bash
# Redis examples
SET usuario:1001:nombre "Juan Pérez"
SET usuario:1001:email "juan@email.com"
HSET usuario:1001 nombre "Juan Pérez" email "juan@email.com" edad 30

# Obtener datos
GET usuario:1001:nombre
HGETALL usuario:1001

# Cache de sesión
SETEX sesion:abc123 3600 "datos_usuario_json"
```

#### 3. **Columnas** (Cassandra, HBase)
Optimizadas para consultas por columnas:

```cql
-- Tabla de eventos en Cassandra
CREATE TABLE eventos (
    usuario_id UUID,
    timestamp timestamp,
    evento text,
    datos text,
    PRIMARY KEY (usuario_id, timestamp)
);

-- Insertar evento
INSERT INTO eventos (usuario_id, timestamp, evento, datos)
VALUES (uuid(), now(), 'login', '{"ip": "192.168.1.1"}');

-- Consultar eventos de usuario
SELECT * FROM eventos 
WHERE usuario_id = 123e4567-e89b-12d3-a456-426614174000
ORDER BY timestamp DESC;
```

#### 4. **Grafos** (Neo4j, Amazon Neptune)
Para datos interconectados:

```cypher
// Crear nodos y relaciones en Neo4j
CREATE (u1:Usuario {nombre: "Ana", email: "ana@email.com"})
CREATE (u2:Usuario {nombre: "Luis", email: "luis@email.com"})
CREATE (p1:Producto {nombre: "Laptop", precio: 1000})

CREATE (u1)-[:COMPRÓ {fecha: "2025-01-15"}]->(p1)
CREATE (u1)-[:AMIGO_DE]->(u2)

// Consultar amigos que compraron productos similares
MATCH (u:Usuario {email: "ana@email.com"})-[:AMIGO_DE]->(amigo)
MATCH (amigo)-[:COMPRÓ]->(producto)
RETURN amigo.nombre, producto.nombre
```

---

## ⚖️ Comparación: SQL vs NoSQL

| Aspecto | SQL (Relacional) | NoSQL |
|---------|------------------|-------|
| **Estructura** | Rígida (esquema fijo) | Flexible (esquema dinámico) |
| **Escalabilidad** | Vertical (más CPU/RAM) | Horizontal (más servidores) |
| **Consistencia** | ACID fuerte | Eventual (BASE) |
| **Consultas** | SQL estándar | APIs específicas |
| **Transacciones** | Completas ACID | Limitadas |
| **Casos de uso** | Sistemas críticos, finanzas | Big Data, tiempo real |

### ¿Cuándo usar SQL?

✅ **Usa SQL cuando:**
- Necesitas **transacciones ACID** (banca, finanzas)
- Datos con **relaciones complejas**
- **Consultas complejas** con JOINs
- **Esquema estable** y bien definido
- Necesitas **consistencia fuerte**

**Ejemplos:** Sistemas bancarios, ERP, CRM tradicional

### ¿Cuándo usar NoSQL?

✅ **Usa NoSQL cuando:**
- Manejas **grandes volúmenes** de datos
- Necesitas **escalabilidad horizontal**
- **Esquema flexible** (datos no estructurados)
- **Desarrollo ágil** con cambios frecuentes
- **Performance** sobre consistencia

**Ejemplos:** Redes sociales, IoT, gaming, analytics

---

## 🛠️ Instalación y primeros pasos

### MySQL en Ubuntu/Debian:

```bash
# Instalar
sudo apt update
sudo apt install mysql-server

# Configurar seguridad
sudo mysql_secure_installation

# Conectar
mysql -u root -p

# Crear base de datos
CREATE DATABASE mi_tienda;
USE mi_tienda;
```

### MongoDB en Ubuntu/Debian:

```bash
# Agregar repositorio
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Instalar
sudo apt update
sudo apt install mongodb-org

# Iniciar servicio
sudo systemctl start mongod
sudo systemctl enable mongod

# Conectar
mongosh
```

### Redis:

```bash
# Instalar
sudo apt install redis-server

# Iniciar
sudo systemctl start redis-server

# Conectar
redis-cli
```

---

## 🏢 Casos de uso profesionales

### E-commerce híbrido:
- **MySQL**: Usuarios, pedidos, inventario (consistencia crítica)
- **Redis**: Cache de sesiones, carrito de compras
- **MongoDB**: Catálogo de productos (esquema flexible)
- **Elasticsearch**: Búsqueda de productos

### Aplicación de análisis:
- **PostgreSQL**: Metadatos, configuración
- **ClickHouse**: Datos de eventos (columnar)
- **Redis**: Cache de consultas frecuentes

### Red social:
- **PostgreSQL**: Usuarios, autenticación
- **Neo4j**: Grafo de conexiones sociales  
- **MongoDB**: Posts, comentarios
- **Redis**: Timeline cache, notificaciones

---

## 🎯 Próximos pasos

1. **Practica con datos reales**: Crea bases de datos para proyectos personales
2. **Aprende SQL avanzado**: Subconsultas, funciones de ventana, procedimientos
3. **Experimenta con NoSQL**: Instala MongoDB y Redis
4. **Estudia rendimiento**: Índices, optimización de consultas
5. **Explora herramientas**: phpMyAdmin, MongoDB Compass, Redis GUI

💡 **Recuerda:** No existe una "mejor" base de datos. La elección depende de tus requisitos específicos de consistencia, escalabilidad, y tipo de datos.
