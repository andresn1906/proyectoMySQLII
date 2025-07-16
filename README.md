# Proyecto MySQL II: Andrés Suárez Niño

## 📌 Descripción del Proyecto:
Este proyecto es una implementación de una base de datos MySQL para un sistema de gestión de encuestas, calificaciones de productos, membresías y beneficios. Incluye:
-   Estructura de base de datos completa (DDL) con tablas normalizadas.
-   Consultas SQL especializadas para análisis de datos.
-   Funciones almacenadas para cálculos y métricas.
-   Procedimientos almacenados para operaciones complejas.
-   Triggers para automatización y validación de datos.
-   Eventos programados para tareas recurrentes.
-   Historias de usuario implementadas con consultas SQL.

## 🗃️ Estructura de Archivos:
-   *DDL_MySQLII.md*: Contiene todos los comandos DDL para crear la base de datos y sus tablas.
-   *insertsMySQLII.md*: Contiene todos los INSERTS MANUALES para las tablas de la base de datos.
-   *historiasDeUsuario.md*: Implementación de 160 historias de usuario organizadas en 8 categorías.

## 🛠️ Configuración Inicial:
1. Creación de la Base de Datos:
    ```sql
        CREATE DATABASE proyectoCasn;
        USE proyectoCasn;
    ```
2. Ejecución de Scripts:
    -   Ejecutar todos los comandos DDL del archivo *DDL_MySQLII.md* para crear la estructura completa de la base de datos.

## 🔍 Características Principales:
### 📊 Consultas Especializadas:
-   Listado de productos con precios más bajos por ciudad.
-   Top 5 de clientes más activos.
-   Distribución de productos por categoría.
-   Productos con calificaciones superiores al promedio.

### 🧮 Funciones Almacenadas:
-   Cálculo de promedios ponderados.
-   Conteo de productos calificados por cliente.
-   Validación de membresías activas.
-   Generación de códigos únicos.

### ⚙️ Procedimientos Almacenados:
-   Registro de calificaciones con actualización automática de promedios.
-   Inserción de empresas con productos predeterminados.
-   Gestión de productos favoritos.
-   Actualización masiva de precios.

### ⚡ Triggers:
-   Validación de datos antes de inserción/actualización.
-   Registro automático de logs.
-   Sincronización entre tablas relacionadas.
-   Actualización de timestamps.

### ⏰ Eventos Programados:
-   Limpieza de productos inactivos.
-   Actualización de promedios semanales.
-   Ajuste de precios por inflación.
-   Generación de reportes periódicos.

## 📋 Ejemplos de Uso:
-   Consulta de Productos por Categoría.
    ```sql
        SELECT pro.name AS Producto, cat.description AS Categoría
        FROM products pro JOIN categories cat ON pro.category_id = cat.id;
    ```

-   Función para Promedio de Calificaciones
    ```sql
        SELECT fn_promedio_calificacion_producto(1) AS PromedioCalificación;
    ```

-   Procedimiento para Registrar Calificación   
    ```sql
        CALL sp_simple_register_rating(1, 1, 4.5);
    ```

## 📈 Diagrama Conceptual de la Base de Datos:
(Nota: Se recomienda generar un diagrama ER basado en las tablas definidas en el DDL)

Las principales entidades son:
-   *Productos*: Información de productos y sus categorías.
-   *Empresas*: Proveedores de productos.
-   *Clientes*: Usuarios que califican productos.
-   *Encuestas*: Mecanismo de recolección de calificaciones.
-   *Membresías*: Planes con beneficios para clientes.

## ✉️ Desarrolladores:
-   Andrés Suárez Niño - https://github.com/andresn1906
