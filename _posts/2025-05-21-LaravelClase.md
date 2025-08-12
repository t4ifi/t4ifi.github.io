---
title: "Cliente-Servidor y API REST con Laravel"
date: 2025-05-21
categories: [Backend, Laravel]
tags: [Laravel, PHP, API, REST, HTTP, Cliente-Servidor]
---

## Introducción al modelo Cliente-Servidor

El modelo cliente-servidor es una arquitectura donde:

- El cliente solicita recursos o servicios.
- El servidor responde con los datos o resultados solicitados.

En una aplicación web, el cliente suele ser un navegador o una app móvil, y el servidor una API que gestiona la lógica y los datos.

## ¿Qué es una API?

Una API (Interfaz de Programación de Aplicaciones) permite que dos sistemas se comuniquen entre sí. Es como un puente que conecta aplicaciones, permitiendo el intercambio de información estructurada, generalmente en formato JSON o XML.

## API REST

REST (Representational State Transfer) es un estilo arquitectónico para diseñar servicios web. Sus principios clave son:

- Comunicación mediante HTTP.
- Uso de verbos HTTP para indicar la acción (GET, POST, PUT, PATCH, DELETE).
- Intercambio de datos estructurado, comúnmente en formato JSON.
- Acceso a recursos mediante URLs claras, como por ejemplo:  
  /api/productos/12 (acceder al producto con ID 12).

## Verbos HTTP y significados

- **GET**: Obtener recurso (por ejemplo, una lista de productos o los detalles de uno).
- **POST**: Crear recurso (enviar datos para crear un nuevo producto).
- **PUT**: Reemplazar recurso completo (actualizar todos los campos de un producto).
- **PATCH**: Modificar parcialmente (actualizar solo algunos campos).
- **DELETE**: Eliminar recurso (borrar un producto).

## Códigos de estado HTTP

- **200 OK**: Respuesta exitosa.
- **201 Created**: Recurso creado correctamente.
- **204 No Content**: Eliminación exitosa sin contenido de respuesta.
- **400 Bad Request**: Datos mal enviados.
- **401 Unauthorized**: No autorizado.
- **403 Forbidden**: Acceso prohibido.
- **404 Not Found**: Recurso no encontrado.
- **500 Internal Server Error**: Error en el servidor.

## Crear una API REST en Laravel

1. **Crear un nuevo proyecto Laravel**  
   Comando:  
   `composer create-project laravel/laravel api-rest-laravel`

2. **Configurar la base de datos en el archivo .env**  
   Ejemplo de configuración:

   ```
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=api_rest
   DB_USERNAME=root
   DB_PASSWORD=
   ```

3. **Crear el modelo con su migración**  
   Comando:  
   `php artisan make:model Producto -m`

   En el archivo de migración, asegurate de definir las columnas:
   - id (autoincremental)
   - nombre (string)
   - precio (decimal 8,2)
   - timestamps (created_at y updated_at)

   Después ejecutá:  
   `php artisan migrate`

4. **Crear el controlador tipo API**  
   Comando:  
   `php artisan make:controller Api/ProductoController --api`

5. **Definir las rutas en el archivo routes/api.php**

   ```php
   use App\Http\Controllers\Api\ProductoController;
   Route::apiResource('productos', ProductoController::class);
   ```

6. **Implementar el controlador con los siguientes métodos:**

   - index(): Devuelve todos los productos.
   - store(Request $request): Valida y guarda un nuevo producto.
   - show($id): Muestra un producto específico por ID.
   - update(Request $request, $id): Actualiza un producto existente.
   - destroy($id): Elimina un producto.

   **Validaciones:**
   - El nombre debe ser un string y obligatorio.
   - El precio debe ser numérico y obligatorio.

7. **Probar la API usando Postman o cURL:**

   - GET `/api/productos`: Lista todos los productos (200 OK).
   - POST `/api/productos`: Crea un nuevo producto (201 Created).
   - GET `/api/productos/{id}`: Muestra un producto (200 OK o 404 Not Found).
   - PUT `/api/productos/{id}`: Reemplaza un producto (200 OK o 404 Not Found).
   - PATCH `/api/productos/{id}`: Modifica parcialmente un producto (200 OK).
   - DELETE `/api/productos/{id}`: Elimina un producto (204 No Content).

## Conclusión

Laravel facilita enormemente la creación de APIs REST gracias a herramientas como:

- Eloquent ORM para interactuar con la base de datos.
- Validaciones automáticas con el objeto Request.
- Controladores con métodos predefinidos para operaciones CRUD.
- Sistema de rutas RESTful.

Al seguir los principios REST y usar correctamente los verbos y códigos HTTP, podés construir APIs robustas, escalables y fáciles de integrar con frontend en Vue, React o apps móviles.
---
**Andrés Nuñez - t4ifi**
