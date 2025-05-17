---
title: "Consultas en Bases de Datos"
date: 2025-05-16 21:00:00 +0000
categories: [Bases de Datos, SQL]
tags: [sql, consultas, base de datos, comandos]
---

# ¿Qué son las Consultas?

Las consultas en bases de datos son instrucciones que se envían a la base de datos para realizar operaciones como recuperar datos, insertar nuevos registros, actualizar información existente o eliminar datos.

---

## Tipos de Consultas

### Relacionadas con la Definición de Datos

- **CREATE**: Se usa para la creación de tablas dentro de la base de datos.  
- **ALTER**: Se usa para modificar la estructura de una tabla.  
- **DROP**: Se usa para eliminar una tabla de la base de datos.  
- **TRUNCATE**: Se usa para borrar todo el contenido de una tabla.  

### Relacionadas con el Tratamiento de Datos

- **INSERT**: Se usa para insertar una o varias filas en una tabla.  
- **SELECT**: Se usa para buscar una o varias filas de una tabla.  
- **UPDATE**: Se usa para actualizar una o varias columnas de una o varias filas de una tabla.  
- **DELETE**: Se usa para eliminar una o varias filas de una tabla.  

> Las consultas relacionadas con el tratamiento de datos normalmente son las que casi siempre pueden ser vulneradas.

---

## Cómo Crear una Base de Datos

Para crear una base de datos en SQL usando comandos:

    CREATE DATABASE Nombre;

**Explicación**:

- `CREATE DATABASE`: Le indica al sistema que quieres crear una nueva base de datos.  
- `Nombre`: Reemplazalo con el nombre que quieras, por ejemplo: `mi_base_datos`.

<div style="text-align: center;">
  <img src="/assets/img/apuntes/Base1.jpeg" alt="crear base de datos" style="width: 100%; max-width: 500px;" />
</div>

---

## Cómo Crear Tablas

    CREATE TABLE usuarios (
      id INT PRIMARY KEY AUTO_INCREMENT,
      nombre VARCHAR(50) NOT NULL,
      email VARCHAR(100) UNIQUE NOT NULL,
      contraseña VARCHAR(255) NOT NULL
    );

**Explicación**:

- `id INT PRIMARY KEY AUTO_INCREMENT`: Crea un ID único para cada fila automáticamente.  
- `nombre VARCHAR(50) NOT NULL`: Guarda nombres de hasta 50 caracteres, sin permitir nulos.  
- `email VARCHAR(100) UNIQUE NOT NULL`: Correos únicos, hasta 100 caracteres, sin permitir nulos.  
- `contraseña VARCHAR(255) NOT NULL`: Guarda contraseñas sin permitir nulos.

### Comando `USE`

    USE nombre_de_la_base_de_datos;

Este comando se usa para seleccionar una base de datos específica.

---

## Cómo Agregar Registros

<div style="text-align: center;">
  <img src="/assets/img/apuntes/base2.jpeg" alt="insertar registro" style="width: 100%; max-width: 500px;" />
</div>

    INSERT INTO empleado (nombre, apellido, edad, username, contraseña)
    VALUES ("andres", "nuñez", 17, "andres.nuneez", "Andres12234");

**Explicación**:

- **INSERT INTO empleado**: Inserta datos en la tabla "empleado".  
- **(nombre, apellido, edad, username, contraseña)**: Columnas de destino.  
- **VALUES (...)**: Valores a insertar en cada columna.

---

## Consulta `SELECT`

    SELECT <columna o *> FROM <tabla> WHERE <condiciones>;

- `SELECT`: Selecciona columnas específicas o todas (`*`).  
- `FROM`: Indica la tabla.  
- `WHERE`: Filtra resultados (opcional).

---

## Consulta `UPDATE`

    UPDATE <tabla>
    SET columna1 = valor1, columna2 = valor2, ...
    WHERE condiciones;

- `UPDATE`: Indica la tabla a actualizar.  
- `SET`: Define qué columnas cambiar.  
- `WHERE`: Aplica condiciones específicas (opcional).

---

## Consulta `DELETE`

    DELETE FROM <tabla> WHERE condiciones;

- `DELETE FROM`: Elimina registros.  
- `WHERE`: Filtra qué registros eliminar (opcional).

---
