---
title: "Introducción a las Bases de Datos"
date: 2025-05-17 12:00:00 +0000
categories: [Bases de Datos]
tags: [apuntes, relacionales, no-relacionales]
---

Una base de datos es un conjunto organizado de datos relacionados entre sí y almacenados de manera sistemática en una computadora o en otro sistema de almacenamiento. Estos datos están estructurados de manera que sea fácil recuperar, gestionar, y actualizar la información según sea necesario.

Las bases de datos se utilizan en una amplia gama de aplicaciones, desde simples listas de contactos hasta sistemas complejos utilizados por grandes empresas o instituciones gubernamentales. Proporcionan un medio eficiente para almacenar y acceder a grandes cantidades de información de manera estructurada, lo que facilita la gestión y la manipulación de los datos para realizar análisis, generar informes, tomar decisiones comerciales, y más.

<div style="text-align: center;">
  <img src="../assets/img/apuntes/mysql.png" alt="MySQL" style="width: 220px; margin-right: 30px;" />
  <img src="../assets/img/apuntes/mariadb.png" alt="MariaDB" style="width: 160px;" />
</div>

---

# Racionales y No Racionales

Una base de datos relacional (o racional) es un tipo de base de datos que organiza los datos en estructuras tabulares llamadas tablas. Está diseñada de acuerdo con el modelo relacional, que utiliza relaciones entre estas tablas para representar la información y las interacciones entre los datos. Las bases de datos relacionales utilizan lenguajes como SQL (Structured Query Language) para realizar consultas y manipular datos.

Por otro lado, una base de datos no relacional (también conocida como NoSQL, que significa "not only SQL" o "no solo SQL") es un tipo de base de datos que no sigue el modelo relacional tradicional. Estas bases de datos se diseñan para manejar grandes volúmenes de datos y diferentes tipos de datos de manera más flexible que las bases de datos relacionales.

Algunas diferencias clave entre una base de datos relacional y una no relacional incluyen:

1. **Estructura de datos:** En una base de datos relacional, los datos se organizan en tablas con filas y columnas, y las relaciones entre las tablas se establecen mediante claves primarias y externas. En una base de datos no relacional, los datos pueden almacenarse de diversas formas, como documentos, gráficos, columnas o claves-valor, según el tipo de base de datos NoSQL utilizada.
    
2. **Escalabilidad:** Las bases de datos NoSQL suelen ser más escalables horizontalmente, lo que significa que pueden manejar grandes volúmenes de datos distribuyendo la carga en múltiples servidores. En comparación, las bases de datos relacionales tienden a ser más rígidas en cuanto a escalabilidad.
    
3. **Flexibilidad:** Las bases de datos NoSQL ofrecen mayor flexibilidad en la estructura de datos, lo que permite manejar datos no estructurados o semiestructurados de manera más eficiente. Esto las hace adecuadas para aplicaciones que requieren adaptabilidad y cambios frecuentes en la estructura de datos.
