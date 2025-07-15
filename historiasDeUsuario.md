# # ProyectoMySQLII Andres Suarez - Ejercicios Historias de Usuario

### 1. Consultas SQL Especializadas:
01.1 Como analista, quiero listar todos los productos con su empresa asociada y el precio más bajo por ciudad.
```sql
SELECT pro.name AS Producto,
em.name AS Empresa,
com.name AS Ciudad,
cp.price AS PrecioMasBajo
FROM companyproducts cp
JOIN products pro ON cp.product_id = pro.id
JOIN companies em ON cp.company_id = em.id
JOIN citiesormunicipalities com ON em.city_id = com.code
WHERE (cp.product_id, em.city_id, cp.price) IN (
    SELECT 
        cp_2.product_id,
        em_2.city_id,
        MIN(cp_2.price)
    FROM companyproducts cp_2
    JOIN companies em_2 ON cp_2.company_id = em_2.id
    GROUP BY cp_2.product_id, em_2.city_id
)
ORDER BY producto, ciudad;
```
02.1 Como administrador, deseo obtener el top 5 de clientes que más productos han calificado en los últimos 6 meses.
```sql
SELECT cu.name AS cliente,
cu.email AS Correo,
COUNT(qp.product_id) AS ProdCalificados
FROM quality_products qp
JOIN customers cu ON qp.customer_id = cu.id
WHERE qp.daterating >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
GROUP BY cu.id, cu.name, cu.email
ORDER BY ProdCalificados DESC
LIMIT 5;
```
03.1 Como gerente de ventas, quiero ver la distribución de productos por categoría y unidad de medida.
```sql
SELECT ca.description AS CategoríaProducto,
um.description AS UnidadDeMedida,
COUNT(DISTINCT cp.product_id) AS CantidadProductos
FROM companyproducts cp
JOIN products pro ON cp.product_id = pro.id
JOIN categories ca ON pro.category_id = ca.id
JOIN unitofmeasure um ON cp.unitmeasure_id = um.id
GROUP BY CategoríaProducto, UnidadDeMedida
ORDER BY CategoríaProducto, UnidadDeMedida;
```
04.1 Como cliente, quiero saber qué productos tienen calificaciones superiores al promedio general.
```sql
SELECT pro.name AS Producto,
ROUND(AVG(qp.rating), 2) AS PromedioDeProductos
FROM quality_products qp
JOIN products pro ON qp.product_id = pro.id
GROUP BY pro.id, pro.name
HAVING AVG(qp.rating) > (
    SELECT AVG(rating) FROM quality_products
)
ORDER BY PromedioDeProductos DESC;
```
05.1 Como auditor, quiero conocer todas las empresas que no han recibido ninguna calificación.
```sql
SELECT em.id, em.name AS Empresa
FROM companies em
LEFT JOIN rates r ON em.id = r.company_id
LEFT JOIN quality_products qp ON em.id = qp.company_id
WHERE r.company_id IS NULL AND qp.company_id IS NULL; 
-- (Empty set) Todas las empresas han sido calificadas.
```
06.1 Como operador, deseo obtener los productos que han sido añadidos como favoritos por más de 10 clientes distintos.
```sql
SELECT df.product_id,
pro.name AS Producto,
COUNT(DISTINCT f.customer_id) AS TotalDeClientes
FROM details_favorites df
JOIN favorites f ON df.favorite_id = f.id
JOIN products pro ON df.product_id = pro.id
GROUP BY df.product_id, pro.name
HAVING COUNT(DISTINCT f.customer_id) > 10; 
-- (Empty set) Hay solamente 3 clientes que tienen menos de 3 productos añadidos como favoritos.
```
07.1 Como gerente regional, quiero obtener todas las empresas activas por ciudad y categoría.
```sql
SELECT em.id AS company_id, em.name AS Empresa,
ct.name AS Ciudad, ct.code AS CódigoCiudad,
cat.description AS Categoría
FROM companies em
JOIN categories cat ON em.category_id = cat.id
JOIN citiesormunicipalities ct ON em.city_id = ct.code
WHERE em.id IN (
    SELECT DISTINCT qp.company_id
    FROM quality_products qp
    JOIN customers cu ON qp.customer_id = cu.id
    WHERE cu.membership_active = 1
    UNION
    SELECT DISTINCT r.company_id
    FROM rates r
    JOIN customers cu ON r.customer_id = cu.id
    WHERE cu.membership_active = 1
)
ORDER BY Ciudad, Categoría;
```
08.1 Como especialista en marketing, deseo obtener los 10 productos más calificados en cada ciudad.
```sql
SELECT sub.city_id AS IdCiudad, sub.product_id,
pro.name AS Producto,
sub.avg_rating AS PromedioCalificación
FROM (
    SELECT 
        c.city_id,
        qp.product_id,
        AVG(qp.rating) AS avg_rating,
        RANK()OVER(PARTITION BY c.city_id ORDER BY AVG(qp.rating) DESC) AS rango
    FROM quality_products qp
    JOIN companies comp ON qp.company_id = comp.id
    JOIN customers c ON qp.customer_id = c.id
    GROUP BY c.city_id, qp.product_id
) AS sub
JOIN products pro ON sub.product_id = pro.id
WHERE sub.rango <= 10
ORDER BY sub.city_id, sub.rango;
```
09.1 Como técnico, quiero identificar productos sin unidad de medida asignada.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto, pro.detail AS Detalle, pro.price AS Precio, pro.category_id AS IdCategoría, pro.image AS URLImage
FROM products pro
LEFT JOIN companyproducts cp ON pro.id = cp.product_id
WHERE cp.unitmeasure_id IS NULL;
-- (Empty set) Todos los productos dados en el INSERT tienen unidad de medida.
```
10.1 Como gestor de beneficios, deseo ver los planes de membresía sin beneficios registrados.
```sql
SELECT ms.id, ms.name AS Membresía, ms.description AS Descripción
FROM memberships ms
LEFT JOIN membershipbenefits mb ON ms.id = mb.membership_id
WHERE mb.benefit_id IS NULL;
-- (Empty set) Cada plan dado en el INSERT tiene al menos un beneficio registrado.
```
11.1 Como supervisor, quiero obtener los productos de una categoría específica con su promedio de calificación.
```sql
-- Productos categoría 1:
SELECT pro.id AS IdProducto, pro.name AS Producto,
cat.description AS Categoría,
ROUND(AVG(qp.rating), 2) AS PromedioCalificación
FROM products pro
JOIN categories cat ON pro.category_id = cat.id
JOIN quality_products qp ON pro.id = qp.product_id
WHERE pro.category_id = 1 
GROUP BY IdProducto, Producto, Categoría;

-- Productos categoría 2:
SELECT pro.id AS IdProducto, pro.name AS Producto,
cat.description AS Categoría,
ROUND(AVG(qp.rating), 2) AS PromedioCalificación
FROM products pro
JOIN categories cat ON pro.category_id = cat.id
JOIN quality_products qp ON pro.id = qp.product_id
WHERE pro.category_id = 2 
GROUP BY IdProducto, Producto, Categoría;

-- Productos categoría 3:
SELECT pro.id AS IdProducto, pro.name AS Producto,
cat.description AS Categoría,
ROUND(AVG(qp.rating), 2) AS PromedioCalificación
FROM products pro
JOIN categories cat ON pro.category_id = cat.id
JOIN quality_products qp ON pro.id = qp.product_id
WHERE pro.category_id = 3 
GROUP BY IdProducto, Producto, Categoría;
```
12.1 Como asesor, deseo obtener los clientes que han comprado productos de más de una empresa.
```sql
SELECT qp.customer_id AS IdCliente, 
c.name AS Cliente, 
COUNT(DISTINCT qp.company_id) AS Empresas
FROM quality_products qp
JOIN customers c ON qp.customer_id = c.id
GROUP BY qp.customer_id, c.name
HAVING COUNT(DISTINCT qp.company_id) > 1;
-- (Empty set) Los clientes no han interactuado con más de una empresa. 
```
13.1 Como director, quiero identificar las ciudades con más clientes activos.
```sql
SELECT c.city_id AS IdCiudad, 
com.name AS Ciudad,
COUNT(c.id) AS TotalClientesActivos
FROM customers c
JOIN citiesormunicipalities com ON c.city_id = com.code
WHERE c.membership_active = 1
GROUP BY c.city_id, com.name
ORDER BY TotalClientesActivos DESC;
```
14.1 Como analista de calidad, deseo obtener el ranking de productos por empresa basado en la media de *quality_products*.
```sql
SELECT cp.company_id AS IdEmpresa,
em.name AS Empresa,
pro.id AS IdProducto, pro.name AS Producto,
ROUND(AVG(qp.rating), 2) AS PromedioCalificación,
RANK() OVER (PARTITION BY cp.company_id ORDER BY AVG(qp.rating) DESC) AS Ranking
FROM companyproducts cp
JOIN companies em ON cp.company_id = em.id
JOIN products pro ON cp.product_id = pro.id
JOIN quality_products qp ON pro.id = qp.product_id AND cp.company_id = qp.company_id
GROUP BY cp.company_id, em.name, pro.id, pro.name
ORDER BY cp.company_id, Ranking;
```
15.1 Como administrador, quiero listar empresas que ofrecen más de cinco productos distintos.
```sql
SELECT em.id AS IdEmpresa, em.name AS Empresa,
COUNT(DISTINCT cp.product_id) AS TotalProductos
FROM companies em
JOIN companyproducts cp ON em.id = cp.company_id
GROUP BY em.id, em.name
HAVING COUNT(DISTINCT cp.product_id) > 5;
```
16.1 Como cliente, deseo visualizar los productos favoritos que aún no han sido calificados.
```sql
SELECT df.product_id AS IdProducto,
pro.name AS Producto,
f.customer_id AS IdCliente,
c.name AS Cliente
FROM details_favorites df
JOIN favorites f ON df.favorite_id = f.id
JOIN products pro ON df.product_id = pro.id
JOIN customers c ON f.customer_id = c.id
LEFT JOIN quality_products qp 
  ON qp.product_id = df.product_id 
  AND qp.customer_id = f.customer_id
WHERE qp.product_id IS NULL;

```
17.1 Como desarrollador, deseo consultar los beneficios asignados a cada audiencia junto con su descripción.
```sql
SELECT ab.audience_id AS IdAudiencia, ab.benefit_id AS IdBeneficio,
a.description AS TipoAudiencia,
b.description AS Beneficio, b.detail AS Detalles
FROM audiencebenefits ab
JOIN audiences a ON ab.audience_id = a.id
JOIN benefits b ON ab.benefit_id = b.id;
```
18.1 Como operador logístico, quiero saber en qué ciudades hay empresas sin productos asociados.
```sql
SELECT em.id AS IdEmpresa, em.name AS Empresa,
com.name AS Ciudad
FROM companies em
JOIN citiesormunicipalities com ON em.city_id = com.code
LEFT JOIN companyproducts cp ON em.id = cp.company_id
WHERE cp.product_id IS NULL;
-- (Empty set) Cada una de las empresas tiene al menos un producto asociado. 
```
19.1 Como técnico, deseo obtener todas las empresas con productos duplicados por nombre.
```sql
SELECT cp.company_id AS IdEmpresa,em.name AS Empresa,
pro.name AS Producto,
COUNT(pro.id) AS TotalProductos
FROM companyproducts cp
JOIN products pro ON cp.product_id = pro.id
JOIN companies em ON cp.company_id = em.id
GROUP BY cp.company_id, em.name, pro.name
HAVING COUNT(pro.id) > 1;
-- (Empty set) No hay empresas con productos duplicados.
```
20.1 Como analista, quiero una vista resumen de clientes, productos favoritos y promedio de calificación recibido.
```sql
SELECT cu.id AS IdCliente, cu.name AS Cliente,
pro.id AS IdProducto,
pro.name AS ProductoFavorito,
ROUND(AVG(qp.rating), 2) AS PromedioCalificación
FROM customers cu
JOIN favorites f ON cu.id = f.customer_id
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products pro ON df.product_id = pro.id
LEFT JOIN quality_products qp ON pro.id = qp.product_id
GROUP BY cu.id, cu.name, pro.id, pro.name;
```

### 2. Subconsultas:

01.2 Como gerente, quiero ver los productos cuyo precio esté por encima del promedio de su categoría.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto, pro.price AS Precio,
c.description AS Categoría
FROM products pro
JOIN categories c ON pro.category_id = c.id
WHERE pro.price > (
    SELECT AVG(pro2.price)
    FROM products pro2
    WHERE pro2.category_id = pro.category_id
);
```
02.2 Como administrador, deseo listar las empresas que tienen más productos que la media de empresas.
```sql
SELECT cp.company_id AS IdEmpresa, 
em.name AS Empresa, 
COUNT(cp.product_id) AS TotalProductos
FROM companyproducts cp
JOIN companies em ON cp.company_id = em.id
GROUP BY cp.company_id, em.name
HAVING COUNT(cp.product_id) > (
    SELECT AVG(Productos) 
    FROM (
        SELECT COUNT(cp.product_id) AS Productos
        FROM companyproducts cp
        GROUP BY cp.company_id
    ) AS sub
);
```
03.2 Como cliente, quiero ver mis productos favoritos que han sido calificados por otros clientes.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto, pro.detail AS Detalle, pro.price AS Precio,
cat.description AS Categoría,
q.rating AS Calificación,
q.customer_id AS CalificaciónCliente
FROM (
    SELECT df.product_id
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    WHERE f.customer_id = 1
) AS favs
JOIN products pro ON favs.product_id = pro.id
JOIN categories cat ON pro.category_id = cat.id
JOIN quality_products q ON pro.id = q.product_id
WHERE q.customer_id <> 1;
-- (Empty Set) El producto favorito del cliente con ID 1 no ha sido calificado por un cliente diferente a él mismo.
```
04.2 Como supervisor, deseo obtener los productos con el mayor número de veces añadidos como favoritos.
```sql
SELECT 
pro.id AS IdProducto, pro.name AS Producto, pro.detail AS Detalle, pro.price AS Precio,
cat.description AS Categoría,
favs.VecesFavorito
FROM (
    SELECT df.product_id,
    COUNT(df.product_id) AS VecesFavorito
    FROM details_favorites df
    GROUP BY df.product_id
) AS favs
JOIN products pro ON favs.product_id = pro.id
JOIN categories cat ON pro.category_id = cat.id
ORDER BY favs.VecesFavorito DESC;
```
05.2 Como técnico, quiero listar los clientes cuyo correo no aparece en la tabla *rates* ni en *quality_products*.
```sql
SELECT 
c.id AS IdCliente, c.name AS Cliente, c.email AS Correo, c.cellphone AS Celular
FROM customers c
WHERE c.email NOT IN (
    SELECT DISTINCT r.email
    FROM rates rt
    JOIN customers r ON rt.customer_id = r.id
)
AND c.email NOT IN (
    SELECT DISTINCT q.email
    FROM quality_products qp
    JOIN customers q ON qp.customer_id = q.id
);
-- (Empty Set) Los correos de cada cliente se almacena en almenos una de las tablas "rates" o "quality_products".
```
06.2 Como gestor de calidad, quiero obtener los productos con una calificación inferior al mínimo de su categoría.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto,
cat.description AS Categoría,
qp.rating AS Calificación
FROM quality_products qp
JOIN products pro ON qp.product_id = pro.id
JOIN categories cat ON pro.category_id = cat.id
WHERE qp.rating < (
    SELECT MIN(qp2.rating)
    FROM quality_products qp2
    JOIN products p2 ON qp2.product_id = p2.id
    WHERE p2.category_id = pro.category_id
);
-- (Empty Set) Ninguno producto tiene una calificación menor al mínimo de su categoría.
```
07.2 Como desarrollador, deseo listar las ciudades que no tienen clientes registrados.
```sql
SELECT com.code AS CodigoCiudad, com.name AS Ciudad
FROM citiesormunicipalities com
WHERE com.code NOT IN (
    SELECT DISTINCT c.city_id
    FROM customers c
);
```
08.2 Como administrador, quiero ver los productos que no han sido evaluados en ninguna encuesta.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto, pro.detail AS Detalle, pro.price AS Precio,
cat.description AS Categoría
FROM products pro
JOIN categories cat ON pro.category_id = cat.id
WHERE pro.id NOT IN (
    SELECT DISTINCT qp.product_id
    FROM quality_products qp
);
```
09.2 Como auditor, quiero listar los beneficios que no están asignados a ninguna audiencia.
```sql
SELECT b.id AS IdBeneficio, b.description AS Beneficio, b.detail AS Detalle
FROM benefits b
WHERE b.id NOT IN (
    SELECT ab.benefit_id
    FROM audiencebenefits ab
);
-- (Empty Set) Todos los beneficios están asignados a cada una de las audiencia.
```
10.2 Como cliente, deseo obtener mis productos favoritos que no están disponibles actualmente en ninguna empresa.
```sql

```
11.2 Como director, deseo consultar los productos vendidos en empresas cuya ciudad tenga menos de tres empresas registradas.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto,
cat.description AS Categoría,
em.name AS Empresa,
ci.name AS Ciudad
FROM companyproducts cp
JOIN products pro ON cp.product_id = pro.id
JOIN categories cat ON pro.category_id = cat.id
JOIN companies em ON cp.company_id = em.id
JOIN citiesormunicipalities ci ON em.city_id = ci.code
WHERE em.city_id IN (
    SELECT em2.city_id
    FROM companies em2
    GROUP BY em2.city_id
    HAVING COUNT(em2.id) < 3
);
```
12.2 Como analista, quiero ver los productos con calidad superior al promedio de todos los productos.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto, pro.price AS Precio,
cat.description AS Categoría,
AVG(qp.rating) AS PromedioCalidad
FROM products pro
JOIN quality_products qp ON pro.id = qp.product_id
JOIN categories cat ON pro.category_id = cat.id
GROUP BY pro.id, pro.name, pro.price, cat.description
HAVING AVG(qp.rating) > (
  SELECT AVG(qp.rating)
  FROM quality_products qp
);
```
13.2 Como gestor, quiero ver empresas que sólo venden productos de una única categoría.
```sql
SELECT em.id AS IdEmpresa, em.name AS Empresa,
cat.description AS Categoria
FROM companies em
JOIN companyproducts cp ON em.id = cp.company_id
JOIN products pro ON cp.product_id = pro.id
JOIN categories cat ON pro.category_id = cat.id
WHERE em.id IN (
    SELECT company_id
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    GROUP BY company_id
    HAVING COUNT(DISTINCT p.category_id) = 1
)
GROUP BY em.id, em.name, cat.description;
```
14.2 Como gerente comercial, quiero consultar los productos con el mayor precio entre todas las empresas.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto,
cp.price AS Precio,
em.name AS Empresa
FROM companyproducts cp
JOIN products pro ON cp.product_id = pro.id
JOIN companies em ON cp.company_id = em.id
WHERE cp.price = (
  SELECT MAX(price)
  FROM companyproducts
);
```
15.2 Como cliente, quiero saber si algún producto de mis favoritos ha sido calificado por otro cliente con más de 4 estrellas.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto, pro.detail AS Detalle, pro.price AS Precio,
qp.rating AS Calificación, qp.customer_id AS ClienteCalificador
FROM (
    SELECT df.product_id
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    WHERE f.customer_id = 1
) AS favs
JOIN quality_products qp ON favs.product_id = qp.product_id
JOIN products pro ON pro.id = favs.product_id
WHERE qp.customer_id <> 1
    AND qp.rating > 4;
-- (Empty Set) Mis productos favoritos(id = 1) no han sido calificados por otro cliente con más de 4 estrellas. 
```
16.2 Como operador, quiero saber qué productos no tienen imagen asignada pero sí han sido calificados.
```sql
SELECT pro.id AS IdProducto, pro.name AS Producto, pro.detail AS Detalle, pro.price AS Precio, pro.image AS Imagen
FROM products pro
WHERE (pro.image IS NULL OR pro.image = '')
    AND pro.id IN (
        SELECT DISTINCT qp.product_id
        FROM quality_products qp
);
-- (Empty Set) Todo producto calificado presenta imagen.
```
17.2 Como auditor, quiero ver los planes de membresía sin periodo vigente.
```sql
SELECT ms.id AS IdMembresia, ms.name AS Membresía, ms.description AS Descripcion
FROM memberships ms
WHERE ms.id NOT IN (
    SELECT DISTINCT mp.membership_id
    FROM membershipperiods mp
);
-- (Empty Set) Todas las membresías tienen un periodo vigente establecido.
```
18.2 Como especialista, quiero identificar los beneficios compartidos por más de una audiencia.
```sql
SELECT b.id AS Beneficio, b.description AS Descripcion
FROM benefits b
WHERE b.id IN (
    SELECT ab.benefit_id
    FROM audiencebenefits ab
    GROUP BY ab.benefit_id
    HAVING COUNT(DISTINCT ab.audience_id) > 1
);
-- (Empty Set) Cada beneficio está asignado a un solo tipo de audiencia.
```
19.2 Como técnico, quiero encontrar empresas cuyos productos no tengan unidad de medida definida.
```sql
SELECT em.id AS IdEmpresa, em.name AS Empresa
FROM companies em
WHERE em.id IN (
    SELECT cp.company_id
    FROM companyproducts cp
    JOIN products pro ON cp.product_id = pro.id
    WHERE pro.unitofmeasure_id IS NULL
);
-- (Empy Set) Todos los productos tienen una unidad de medida.
```
20.2 Como gestor de campañas, deseo obtener los clientes con membresía activa y sin productos favoritos.
```sql
SELECT cu.id AS IdCliente, cu.name AS NombreCliente
FROM customers cu
WHERE cu.membership_active = TRUE
AND cu.id NOT IN (
    SELECT f.customer_id
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
);
-- (Empty Set) Todos los clientes tienen un producto favorito y se encuentran con membresía activa.
```

### 3. Funciones Agregadas:

Obtener el promedio de calificación por producto
```sql

```
Contar cuántos productos ha calificado cada cliente
```sql

```
Sumar el total de beneficios asignados por audiencia
```sql

```
Calcular la media de productos por empresa
```sql

```
Contar el total de empresas por ciudad
```sql

```
Calcular el promedio de precios por unidad de medida
```sql

```
Contar cuántos clientes hay por ciudad
```sql

```
Calcular planes de membresía por periodo
```sql

```
Ver el promedio de calificaciones dadas por un cliente a sus favoritos
```sql

```
Consultar la fecha más reciente en que se calificó un producto
```sql

```
Obtener la desviación estándar de precios por categoría
```sql

```
Contar cuántas veces un producto fue favorito
```sql

```
Calcular el porcentaje de productos evaluados
```sql

```
Ver el promedio de rating por encuesta
```sql

```
Calcular el promedio y total de beneficios por plan
```sql

```
Obtener media y varianza de precios por empresa
```sql

```
Ver total de productos disponibles en la ciudad del cliente
```sql

```
Contar productos únicos por tipo de empresa
```sql

```
Ver total de clientes sin correo electrónico registrado
```sql

```
Empresa con más productos calificados
```sql

```

### 4. Procedimientos Almacenados:

Registrar una nueva calificación y actualizar el promedio
```sql

```
Insertar empresa y asociar productos por defecto
```sql

```
Añadir producto favorito validando duplicados
```sql

```
Generar resumen mensual de calificaciones por empresa
```sql

```
Calcular beneficios activos por membresía
```sql

```
Eliminar productos huérfanos
```sql

```
Actualizar precios de productos por categoría
```sql

```
Validar inconsistencia entre rates y quality_products
```sql

```
Asignar beneficios a nuevas audiencias
```sql

```
Activar planes de membresía vencidos con pago confirmado
```sql

```
Listar productos favoritos del cliente con su calificación
```sql

```
Registrar encuesta y sus preguntas asociadas
```sql

```
Eliminar favoritos antiguos sin calificaciones
```sql

```
Asociar beneficios automáticamente por audiencia
```sql

```
Historial de cambios de precio
```sql

```
Registrar encuesta activa automáticamente
```sql

```
Actualizar unidad de medida de productos sin afectar ventas
```sql

```
Recalcular promedios de calidad semanalmente
```sql

```
Validar claves foráneas entre calificaciones y encuestas
```sql

```
Generar el top 10 de productos más calificados por ciudad
```sql

```

### 5. Triggers:

Actualizar la fecha de modificación de un producto
```sql

```
Registrar log cuando un cliente califica un producto
```sql

```
Impedir insertar productos sin unidad de medida
```sql

```
Validar calificaciones no mayores a 5
```sql

```
Actualizar estado de membresía cuando vence
```sql

```
Evitar duplicados de productos por empresa
```sql

```
Enviar notificación al añadir un favorito
```sql

```
Insertar fila en quality_products tras calificación
```sql

```
Eliminar favoritos si se elimina el producto
```sql

```
Bloquear modificación de audiencias activas
```sql

```
Recalcular promedio de calidad del producto tras nueva evaluación
```sql

```
Registrar asignación de nuevo beneficio
```sql

```
Impedir doble calificación por parte del cliente
```sql

```
Validar correos duplicados en clientes
```sql

```
Eliminar detalles de favoritos huérfanos
```sql

```
Actualizar campo updated_at en companies
```sql

```
Impedir borrar ciudad si hay empresas activas
```sql

```
Registrar cambios de estado en encuestas
```sql

```
Sincronizar *rates* y *quality_products*
```sql

```
Eliminar productos sin relación a empresas
```sql

```

### 6. Events:

Borrar productos sin actividad cada 6 meses
```sql

```
Recalcular el promedio de calificaciones semanalmente
```sql

```
Actualizar precios según inflación mensual
```sql

```
Crear backups lógicos diariamente
```sql

```
Notificar sobre productos favoritos sin calificar
```sql

```
Revisar inconsistencias entre empresa y productos
```sql

```
Archivar membresías vencidas diariamente
```sql

```
Notificar beneficios nuevos a usuarios semanalmente
```sql

```
Calcular cantidad de favoritos por cliente mensualmente
```sql

```
Validar claves foráneas semanalmente
```sql

```
Eliminar calificaciones inválidas antiguas
```sql

```
Cambiar estado de encuestas inactivas automáticamente
```sql

```
Registrar auditorías de forma periódica
```sql

```
Notificar métricas de calidad a empresas
```sql

```
Recordar renovación de membresías
```sql

```
Reordenar estadísticas generales cada semana
```sql

```
Crear resúmenes temporales de uso por categoría
```sql

```
Actualizar beneficios caducados
```sql

```
Alertar productos sin evaluación anual
```sql

```
Actualizar precios con índice externo
```sql

```

### 7. Historias de Usuario con JOINs:

 1.7 Ver productos con la empresa que los vende
```sql
SELECT 
    co.name AS empresa,
    p.name AS producto,
    cp.price AS precio
FROM companyproducts cp
INNER JOIN companies co ON cp.company_id = co.id
INNER JOIN products p ON cp.product_id = p.id;
```
 2.7 Mostrar productos favoritos con su empresa y categoría
```sql
SELECT 
    p.name AS producto,
    cat.description AS categoria,
    c.name AS empresa
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
JOIN categories cat ON p.category_id = cat.id
JOIN companyproducts cp ON cp.product_id = p.id
JOIN companies c ON cp.company_id = c.id;
```
 3.7 Ver empresas aunque no tengan productos
```sql
SELECT 
    C.name AS empresa
FROM companies c
LEFT JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY c.name
HAVING COUNT(cp.product_id) = 0;
 4.7 Ver productos que fueron calificados (o no)
SELECT 
    p.name AS producto,
    r.rating
FROM quality_products r
RIGHT JOIN products p ON p.id = r.product_id;
```
 5.7 Ver productos con promedio de calificación y empresa
```sql
SELECT 
    c.name AS empresa,
    p.name AS producto,
    AVG(qp.rating) AS promedio
FROM products p
JOIN companyproducts cp ON p.id = cp.product_id
JOIN companies c ON cp.company_id = c.id
JOIN quality_products qp ON qp.product_id = p.id AND qp.company_id = c.id
GROUP BY c.name, p.name;
```
 6.7 Ver clientes y sus calificaciones (si las tienen)
```sql
SELECT 
    cl.name AS cliente,
    r.rating AS calificacion
FROM customers cl
LEFT JOIN rates r ON cl.id = r.customer_id;
```
 7.7 Ver favoritos con la última calificación del cliente
```sql
SELECT
    c.name AS nombre_cliente,
    prod.name AS nombre_producto_fav,
    comp.name AS empresa,
    cat.description AS categoria,
    MAX(r.daterating) AS ultima_calificacion_fecha,
    r.rating AS calificacion
FROM customers c
JOIN favorites fav ON c.id = fav.customer_id
JOIN details_favorites df ON fav.id = df.favorite_id
JOIN products prod ON df.product_id = prod.id
JOIN companyproducts cp ON prod.id = cp.product_id
JOIN companies comp ON cp.company_id = comp.id
JOIN categories cat ON prod.category_id = cat.id
LEFT JOIN rates r ON c.id = r.customer_id AND comp.id = r.company_id
GROUP BY fav.id, prod.id, comp.id, r.rating;
```
 8.7 Ver beneficios incluidos en cada plan de membresía
```sql
SELECT
    m.name AS membresia,
    m.description AS descripcion,
    b.description AS beneficions,
    b.detail AS detalles
FROM membershipbenefits msb
JOIN memberships m ON m.id = msb.membership_id
JOIN benefits b ON b.id = msb.benefit_id;
```
 9.7 Ver clientes con membresía activa y sus beneficios
```sql

```
10.7 Ver ciudades con cantidad de empresas
```sql
SELECT
    cm.name AS Ciudad_Municipio,
    COUNT(c.id) AS Cantidad_Empresas
FROM citiesormunicipalities cm
JOIN companies c ON cm.code = c.city_id
GROUP BY(cm.code);
```
11.7 Ver encuestas con calificaciones
```sql
SELECT
    p.name AS encuesta,
    p.description AS descripcion,
    r.rating AS calificacion
FROM polls p
JOIN rates r ON p.id = r.poll_id;
```
12.7 Ver productos evaluados con datos del cliente
```sql
SELECT 
    p.name AS producto,
    c.name AS cliente,
    qp.daterating AS fecha_evaluacion,
    qp.rating AS calificacion
FROM quality_products qp
JOIN products p ON qp.product_id = p.id
JOIN customers c ON qp.customer_id = c.id;
```
13.7 Ver productos con audiencia de la empresa
```sql
SELECT 
    p.name AS producto,
    c.name AS empresa,
    a.description AS audiencia_objetivo
FROM products p
JOIN companyproducts cp ON p.id = cp.product_id
JOIN companies c ON cp.company_id = c.id
JOIN audiences a ON c.audience_id = a.id;
```
14.7 Ver clientes con sus productos favoritos
```sql
SELECT 
    c.name AS cliente,
    p.name AS producto_favorito
FROM customers c
JOIN favorites f ON f.customer_id = c.id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id;
```
15.7 Ver planes, periodos, precios y beneficios
```sql
SELECT
    m.name AS membresia,
    p.name AS periodo,
    b.description AS beneficio,
    b.detail AS detalle_beneficio
FROM membershipbenefits msb
JOIN memberships m ON m.id = msb.membership_id
JOIN benefits b ON b.id = msb.benefit_id
JOIN periods p ON p.id = msb.period_id;
```
16.7 Ver combinaciones empresa-producto-cliente calificados
```sql
SELECT 
    cu.name AS cliente,
    p.name AS producto,
    c.name AS empresa,
    qp.rating AS calificacion
FROM quality_products qp
JOIN products p ON qp.product_id = p.id
JOIN companies c ON qp.company_id = c.id
JOIN customers cu ON qp.customer_id = cu.id;
```
17.7 Comparar favoritos con productos calificados
```sql
SELECT
    f.customer_id,
    p.name AS producto,
    qp.rating AS calificacion,
    qp.daterating AS fecha
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
JOIN quality_products qp ON qp.product_id = p.id AND qp.customer_id = f.customer_id
WHERE f.customer_id = 1
ORDER BY fecha DESC;
```
18.7 Ver productos ordenados por categoría
```sql
SELECT
    p.name AS producto,
    c.description AS categoria
FROM products p
JOIN categories c ON c.id = category_id;
```
19.7 Ver beneficios por audiencia, incluso vacíos
```sql
SELECT 
    a.description AS audiencia,
    b.description AS beneficio,
    b.detail AS detalle
FROM audiences a
LEFT JOIN audiencebenefits ab ON a.id = ab.audience_id
LEFT JOIN benefits b ON ab.benefit_id = b.id;
```
20.7 Ver datos cruzados entre calificaciones, encuestas, productos y clientes
```sql
SELECT 
    cu.name AS cliente,
    p.name AS producto,
    c.name AS empresa,
    poll.name AS encuesta,
    qp.rating AS calificacion,
    qp.daterating AS fecha
FROM quality_products qp
JOIN products p ON qp.product_id = p.id
JOIN polls poll ON qp.poll_id = poll.id
JOIN companies c ON qp.company_id = c.id
JOIN customers cu ON qp.customer_id = cu.id;
```

### 8. Historias de Usuario con Funciones Definidas por el Usuario (UDF):
Como analista, quiero una función que calcule el promedio ponderado de calidad de un producto basado en sus calificaciones y fecha de evaluación.
Como auditor, deseo una función que determine si un producto ha sido calificado recientemente (últimos 30 días).
Como desarrollador, quiero una función que reciba un product_id y devuelva el nombre completo de la empresa que lo vende.
Como operador, deseo una función que, dado un customer_id, me indique si el cliente tiene una membresía activa.
Como administrador, quiero una función que valide si una ciudad tiene más de X empresas registradas, recibiendo la ciudad y el número como parámetros.
Como gerente, deseo una función que, dado un rate_id, me devuelva una descripción textual de la calificación (por ejemplo, “Muy bueno”, “Regular”).
Como técnico, quiero una función que devuelva el estado de un producto en función de su evaluación (ej. “Aceptable”, “Crítico”).
Como cliente, deseo una función que indique si un producto está entre mis favoritos, recibiendo el product_id y mi customer_id.
Como gestor de beneficios, quiero una función que determine si un beneficio está asignado a una audiencia específica, retornando verdadero o falso.
Como auditor, deseo una función que reciba una fecha y determine si se encuentra dentro de un rango de membresía activa.
Como desarrollador, quiero una función que calcule el porcentaje de calificaciones positivas de un producto respecto al total.
Como supervisor, deseo una función que calcule la edad de una calificación, en días, desde la fecha actual.
Como operador, quiero una función que, dado un company_id, devuelva la cantidad de productos únicos asociados a esa empresa.
Como gerente, deseo una función que retorne el nivel de actividad de un cliente (frecuente, esporádico, inactivo), según su número de calificaciones.
Como administrador, quiero una función que calcule el precio promedio ponderado de un producto, tomando en cuenta su uso en favoritos.
Como técnico, deseo una función que me indique si un benefit_id está asignado a más de una audiencia o membresía (valor booleano).
Como cliente, quiero una función que, dada mi ciudad, retorne un índice de variedad basado en número de empresas y productos.
Como gestor de calidad, deseo una función que evalúe si un producto debe ser desactivado por tener baja calificación histórica.
Como desarrollador, quiero una función que calcule el índice de popularidad de un producto (combinando favoritos y ratings).
Como auditor, deseo una función que genere un código único basado en el nombre del producto y su fecha de creación.
