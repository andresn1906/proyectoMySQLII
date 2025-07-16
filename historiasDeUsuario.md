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
-- (Empty Set) Ningún producto tiene una calificación menor al mínimo de su categoría.
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
SELECT DISTINCT pro.id AS IdProducto, pro.name AS Producto
FROM products pro
JOIN details_favorites df ON pro.id = df.product_id
JOIN favorites f ON df.favorite_id = f.id
WHERE pro.id NOT IN (
    SELECT cp.product_id
    FROM companyproducts cp
    WHERE cp.available_product = 1
);
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

01.3 Obtener el promedio de calificación por producto.
🔍 *Explicación para dummies*: La persona encargada de revisar el rendimiento quiere saber qué tan bien calificado está cada producto. Con *AVG(rating)* agrupado por *product_id*, puede verlo de forma resumida.

```sql
DELIMITER //
CREATE FUNCTION fn_promedio_calificacion_producto(
    p_product_id INT
)
RETURNS DECIMAL(3,1)
DETERMINISTIC
BEGIN
    DECLARE v_promedio DECIMAL(3,1);
    
    SELECT AVG(rating) INTO v_promedio
    FROM quality_products
    WHERE product_id = p_product_id;
    
    IF v_promedio IS NULL THEN
        RETURN 0.0; 
    ELSE
        RETURN ROUND(v_promedio, 1);
    END IF;
END //

DELIMITER ;

-- Uso de la función:
SELECT pro.name AS Producto,
fn_promedio_calificacion_producto(pro.id) AS CalificaciónPromedio
FROM products pro
ORDER BY CalificaciónPromedio DESC;
```

02.3 Contar cuántos productos ha calificado cada cliente.
🔍 Explicación: Aquí se quiere saber quiénes están activos opinando. Se usa *COUNT(*)* sobre rates, agrupando por *customer_id*.

```sql
DELIMITER //
CREATE FUNCTION fn_productos_calificados_por_cliente(
    p_customer_id INT 
)
RETURNS INT 
DETERMINISTIC
BEGIN
    DECLARE v_total_productos INT;
    
    SELECT COUNT(DISTINCT product_id) INTO v_total_productos
    FROM quality_products
    WHERE customer_id = p_customer_id;
    
    RETURN IFNULL(v_total_productos, 0);
END //

DELIMITER ;

-- Uso de la función:
SELECT 
c.id, c.name AS Cliente,
fn_productos_calificados_por_cliente(c.id) AS ProductosCalificados
FROM customers c
ORDER BY ProductosCalificados DESC;
```

03.3 Sumar el total de beneficios asignados por audiencia.
🔍 Explicación: El auditor busca cuántos beneficios tiene cada tipo de usuario. Con *COUNT(*)* agrupado por *audience_id* en *audiencebenefits*, lo obtiene.

```sql
DELIMITER //
CREATE FUNCTION fn_total_beneficios_audiencia(
    p_audience_id INT
)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_total_beneficios INT;
    
    SELECT COUNT(*) INTO v_total_beneficios
    FROM audiencebenefits
    WHERE audience_id = p_audience_id;
    
    RETURN IFNULL(v_total_beneficios, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT 
a.id, a.description AS Audiencia,
fn_total_beneficios_audiencia(a.id) AS TotalBeneficios
FROM audiences a
ORDER BY TotalBeneficios DESC;
```

04.3 Calcular la media de productos por empresa.
🔍 Explicación: El administrador quiere saber si las empresas están ofreciendo pocos o muchos productos. Cuenta los productos por empresa y saca el promedio con *AVG(cantidad)*.

```sql
DELIMITER //

CREATE FUNCTION fn_media_productos_por_empresa()
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE v_media DECIMAL(10,2);
    
    SELECT AVG(productos_por_empresa) INTO v_media
    FROM (
        SELECT COUNT(*) AS productos_por_empresa
        FROM companyproducts
        GROUP BY company_id
    ) AS conteo_productos;
    
    RETURN IFNULL(v_media, 0);
END //

DELIMITER ;

-- Uso de la función:
SELECT fn_media_productos_por_empresa() AS MediaProductosGeneral;
```

05.3 Contar el total de empresas por ciudad.
🔍 Explicación: La idea es ver en qué ciudades hay más movimiento empresarial. Se usa *COUNT(*)* en *companies*, agrupando por *city_id*.

```sql
DELIMITER //
CREATE FUNCTION fn_total_empresas_ciudad(p_city_code VARCHAR(10))
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_total INT;
    
    SELECT COUNT(*) INTO v_total
    FROM companies
    WHERE city_id = p_city_code;
    
    RETURN IFNULL(v_total, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT com.name AS Ciudad,
fn_total_empresas_ciudad(com.code) AS TotalEmpresas
FROM citiesormunicipalities com
WHERE 
    fn_total_empresas_ciudad(com.code) > 0;
```

06.3 Calcular el promedio de precios por unidad de medida.
🔍 Explicación: Se necesita saber si los precios son coherentes según el tipo de medida. Con *AVG(price)* agrupado por *unit_id*, se compara cuánto cuesta el litro, kilo, unidad, etc.

```sql
DELIMITER //
CREATE FUNCTION fn_promedio_precio_unidad(p_unit_id INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE v_promedio DECIMAL(10,2);
    
    SELECT AVG(cp.price) INTO v_promedio
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    WHERE p.unitofmeasure_id = p_unit_id;
    
    RETURN IFNULL(v_promedio, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT u.description AS UnidadDeMedida,
fn_promedio_precio_unidad(u.id) AS PrecioPromedio
FROM unitofmeasure u
WHERE fn_promedio_precio_unidad(u.id) > 0;
```

07.3 Contar cuántos clientes hay por ciudad.
🔍 Explicación: Con *COUNT(*)* agrupado por *city_id* en la tabla *customers*, se obtiene la cantidad de clientes que hay en cada zona.

```sql
DELIMITER //
CREATE FUNCTION fn_total_clientes_ciudad(p_city_code VARCHAR(10))
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_total INT;
    
    SELECT COUNT(*) INTO v_total
    FROM customers
    WHERE city_id = p_city_code;
    
    RETURN IFNULL(v_total, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT com.name AS Ciudad,
fn_total_clientes_ciudad(com.code) AS TotalClientes
FROM citiesormunicipalities com
WHERE fn_total_clientes_ciudad(com.code) > 0
ORDER BY TotalClientes DESC;
```

08.3 Calcular planes de membresía por periodo.
🔍 Explicación: Sirve para ver qué tantos planes están vigentes cada mes o trimestre. Se agrupa por periodo (*start_date*, *end_date*) y se cuenta cuántos registros hay.

```sql
DELIMITER //
CREATE FUNCTION fn_planes_membresia_por_periodo(p_period_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_total INT;
    
    SELECT COUNT(*) INTO v_total
    FROM membershipperiods
    WHERE period_id = p_period_id;
    
    RETURN IFNULL(v_total, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT pe.name AS Periodo,
fn_planes_membresia_por_periodo(pe.id) AS TotalPlanes
FROM periods pe
ORDER BY TotalPlanes DESC;
```

09.3 Ver el promedio de calificaciones dadas por un cliente a sus favoritos.
🔍 Explicación: El cliente quiere saber cómo ha calificado lo que más le gusta. Se hace un JOIN entre favoritos y calificaciones, y se saca *AVG(rating)*.
```sql
DELIMITER //
CREATE FUNCTION fn_promedio_calificaciones_favoritos(p_customer_id INT)
RETURNS DECIMAL(3,2)
DETERMINISTIC
BEGIN
    DECLARE v_promedio DECIMAL(3,2);
    
    SELECT AVG(r.rating) INTO v_promedio
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    JOIN rates r ON f.customer_id = r.customer_id AND f.company_id = r.company_id
    WHERE f.customer_id = p_customer_id;
    
    RETURN IFNULL(v_promedio, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT c.name AS Cliente,
fn_promedio_calificaciones_favoritos(c.id) AS PromedioCalificacionesFav
FROM customers c
WHERE fn_promedio_calificaciones_favoritos(c.id) > 0;
```
10.3 Consultar la fecha más reciente en que se calificó un producto.
🔍 Explicación: Busca el *MAX(created_at)* agrupado por producto. Así sabe cuál fue la última vez que se evaluó cada uno.

```sql
DELIMITER //
CREATE FUNCTION fn_ultima_calificacion_producto(p_product_id INT)
RETURNS DATETIME
DETERMINISTIC
BEGIN
    DECLARE v_fecha_reciente DATETIME;
    
    SELECT MAX(q.daterating) INTO v_fecha_reciente
    FROM quality_products q
    WHERE q.product_id = p_product_id;
    
    RETURN v_fecha_reciente;
END //
DELIMITER ;

-- Uso de la función:
SELECT pro.name AS Producto,
fn_ultima_calificacion_producto(pro.id) AS CalificaciónReciente
FROM products pro
WHERE fn_ultima_calificacion_producto(pro.id) IS NOT NULL
ORDER BY CalificaciónReciente DESC;
```

11.3 Obtener la desviación estándar de precios por categoría.
🔍 Explicación: Usando *STDDEV(price)* en *companyproducts* agrupado por *category_id*, se puede ver si hay mucha diferencia de precios dentro de una categoría.

```sql
DELIMITER //
CREATE FUNCTION fn_desviacion_categoria(p_category_id INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE v_desviacion DECIMAL(10,2);
    
    SELECT STDDEV(cp.price) INTO v_desviacion
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    WHERE p.category_id = p_category_id;
    
    RETURN IFNULL(v_desviacion, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT cat.description AS Categoría,
fn_desviacion_categoria(cat.id) AS DesviacionEstándar,
(
    SELECT COUNT(cp.product_id)
    FROM products pro 
    JOIN companyproducts cp ON pro.id = cp.product_id 
    WHERE pro.category_id = cat.id) AS Productos
FROM categories cat
ORDER BY DesviacionEstándar DESC;
```

12.3 Contar cuántas veces un producto fue favorito.
🔍 Explicación: Con *COUNT(*)* en *details_favorites*, agrupado por *product_id*, se obtiene cuáles productos son los más populares entre los clientes.

```sql
DELIMITER //
CREATE FUNCTION fn_contar_favoritos_producto(p_product_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_total INT;
    
    SELECT COUNT(*) INTO v_total
    FROM details_favorites
    WHERE product_id = p_product_id;
    
    RETURN IFNULL(v_total, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT pro.name AS Producto,
fn_contar_favoritos_producto(pro.id) AS VecesFavorito
FROM products pro
WHERE fn_contar_favoritos_producto(pro.id) > 0
ORDER BY VecesFavorito DESC;
```

13.3 Calcular el porcentaje de productos evaluados.
🔍 Explicación: Cuenta cuántos productos hay en total y cuántos han sido evaluados *(rates)*. Luego calcula *(evaluados / total) x 100*.

```sql
DELIMITER //
CREATE FUNCTION fn_porcentaje_productos_evaluados()
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
    DECLARE v_total_productos INT;
    DECLARE v_productos_evaluados INT;
    DECLARE v_porcentaje DECIMAL(5,2);
    
    SELECT COUNT(*) INTO v_total_productos FROM products;
    
    SELECT COUNT(DISTINCT product_id) INTO v_productos_evaluados 
    FROM quality_products;
    
    SET v_porcentaje = (v_productos_evaluados * 100.0) / v_total_productos;
    
    RETURN IFNULL(v_porcentaje, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT fn_porcentaje_productos_evaluados() AS '% ProductosEvaluados';
```

14.3 Ver el promedio de rating por encuesta.
🔍 Explicación: Agrupa por *poll_id* en *rates*, y calcula el *AVG(rating)* para ver cómo se comportó cada encuesta.

```sql
DELIMITER //
CREATE FUNCTION fn_promedio_rating_encuesta(p_poll_id INT)
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    DECLARE v_promedio DECIMAL(3,2);
    DECLARE v_count INT;
    
    SELECT COUNT(*) INTO v_count FROM rates WHERE poll_id = p_poll_id;
    
    IF v_count = 0 THEN
        RETURN 'Sin datos';
    ELSE
        SELECT AVG(rating) INTO v_promedio FROM rates WHERE poll_id = p_poll_id;
        RETURN CONCAT(ROUND(v_promedio, 2), ' (', v_count, ' ratings)');
    END IF;
END //
DELIMITER ;

-- Uso de la función:
SELECT p.name AS Encuesta,
fn_promedio_rating_encuesta(p.id) AS 'Promedio de Rating'
FROM polls p
ORDER BY p.id;
```

15.3 Calcular el promedio y total de beneficios por plan.
🔍 Explicación: Agrupa por *membership_id* en *membershipbenefits*, y usa *COUNT(*)* y *AVG(beneficio)* si aplica (si hay ponderación).

```sql
DELIMITER //
CREATE FUNCTION fn_total_beneficios_plan(p_membership_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_total INT;
    
    SELECT COUNT(*) INTO v_total
    FROM membershipbenefits
    WHERE membership_id = p_membership_id;
    
    RETURN IFNULL(v_total, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT m.name AS Plan,
COUNT(mb.benefit_id) AS TotalBeneficios,
AVG(b.id) AS PromedioBeneficios
FROM memberships m
LEFT JOIN membershipbenefits mb ON m.id = mb.membership_id
LEFT JOIN benefits b ON mb.benefit_id = b.id
GROUP BY m.id, m.name
ORDER BY TotalBeneficios DESC;
```

16.3 Obtener media y varianza de precios por empresa.
🔍 Explicación: Se agrupa por *company_id* y se usa *AVG(price)* y *VARIANCE(price)* para saber qué tan consistentes son los precios por empresa.

```sql
DELIMITER //
CREATE FUNCTION fn_varianza_precios_empresa(p_company_id VARCHAR(20))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE v_varianza DECIMAL(10,2);
    
    SELECT VARIANCE(price) INTO v_varianza
    FROM companyproducts
    WHERE company_id = p_company_id;
    
    RETURN IFNULL(v_varianza, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT em.name AS NombreEmpresa,
COUNT(cp.product_id) AS CantidadProductos, ROUND(AVG(cp.price), 2) AS MediaPrecios, ROUND(VARIANCE(cp.price), 2) AS VarianzaPrecios
FROM companies em
LEFT JOIN companyproducts cp ON em.id = cp.company_id
GROUP BY em.id, em.name
HAVING COUNT(cp.product_id) > 1
ORDER BY VarianzaPrecios DESC;
```

17.3 Ver total de productos disponibles en la ciudad del cliente.
🔍 Explicación: Hace un *JOIN* entre *companies*, *companyproducts* y *citiesormunicipalities*, filtrando por la ciudad del cliente. Luego se cuenta.

```sql
DELIMITER //
CREATE FUNCTION fn_total_productos_ciudad_cliente(p_customer_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_total INT;
    DECLARE v_city_code VARCHAR(10);
    
    -- Obtener la ciudad del cliente
    SELECT city_id INTO v_city_code
    FROM customers
    WHERE id = p_customer_id;
    
    -- Contar productos disponibles en esa ciudad
    SELECT COUNT(DISTINCT cp.product_id) INTO v_total
    FROM companyproducts cp
    JOIN companies c ON cp.company_id = c.id
    WHERE c.city_id = v_city_code
    AND cp.available_product = TRUE;
    
    RETURN IFNULL(v_total, 0);
END //
DELIMITER ;

-- Uso de la función:
SELECT cu.name AS Cliente,
com.name AS CiudadCliente,
COUNT(DISTINCT cp.product_id) AS ProductosDisponibles,
IFNULL(GROUP_CONCAT(DISTINCT pro.name SEPARATOR ', '), 'No disponible en ciudad') AS ListaProductos
FROM customers cu
LEFT JOIN citiesormunicipalities com ON cu.city_id = com.code
LEFT JOIN companies c ON com.code = c.city_id
LEFT JOIN companyproducts cp ON c.id = cp.company_id AND cp.available_product = TRUE
LEFT JOIN products pro ON cp.product_id = pro.id
GROUP BY cu.id, cu.name, com.name;
```

18.3 Contar productos únicos por tipo de empresa.
🔍 Explicación: Agrupa por *company_type_id* y cuenta cuántos productos diferentes tiene cada tipo de empresa.

```sql
DELIMITER //

CREATE FUNCTION count_unique_products_by_company_type() 
RETURNS TEXT
DETERMINISTIC
BEGIN
    DECLARE result TEXT DEFAULT '';
    DECLARE done INT DEFAULT FALSE;
    DECLARE c_type VARCHAR(100);
    DECLARE p_count INT;
    DECLARE cur CURSOR FOR 
        SELECT 
            ct.name,
            COUNT(DISTINCT cp.product_id)
        FROM 
            company_types ct
        JOIN 
            companies c ON ct.id = c.typecompany_id
        JOIN 
            companyproducts cp ON c.id = cp.company_id
        GROUP BY 
            ct.id, ct.name;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN cur;
    
    read_loop: LOOP
        FETCH cur INTO c_type, p_count;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        IF result = '' THEN
            SET result = CONCAT(c_type, ': ', p_count, ' products');
        ELSE
            SET result = CONCAT(result, '; ', c_type, ': ', p_count, ' products');
        END IF;
    END LOOP;
    
    CLOSE cur;
    
    RETURN result;
END //

DELIMITER ;

-- Uso de la función:
SELECT ct.name AS 'Tipo de Empresa',
COUNT(DISTINCT cp.product_id) AS CantidadProductos
FROM company_types ct
JOIN companies c ON ct.id = c.typecompany_id
JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY ct.id, ct.name
ORDER BY COUNT(DISTINCT cp.product_id) DESC;
```

19.3 Ver total de clientes sin correo electrónico registrado.
🔍 Explicación: Filtra *customers WHERE email IS NULL* y hace un *COUNT(*)*. Esto ayuda a mejorar la base de datos para campañas.

```sql
DELIMITER //

CREATE FUNCTION count_customers_without_email() 
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE total INT;
    
    SELECT COUNT(*) INTO total
    FROM customers
    WHERE email IS NULL OR email = '';
    
    RETURN total;
END //

DELIMITER ;

-- Uso de la función:
SELECT count_customers_without_email() AS 'Cantidad de clientes sin email';
```

20.3 Empresa con más productos calificados.
🔍 Explicación: Hace un *JOIN* entre *companies*, *companyproducts*, y *rates*, agrupa por empresa y usa *COUNT(DISTINCT product_id)*, ordenando en orden descendente y tomando solo el primero.

```sql
DELIMITER //

CREATE FUNCTION get_company_with_most_rated_products() 
RETURNS VARCHAR(255)
DETERMINISTIC
BEGIN
    DECLARE result VARCHAR(255);
    
    SELECT CONCAT(name, ' (', COUNT(DISTINCT product_id), ' productos calificados)') INTO result
    FROM companies c
    JOIN quality_products qp ON c.id = qp.company_id
    GROUP BY c.id, c.name
    ORDER BY COUNT(DISTINCT product_id) DESC
    LIMIT 1;
    
    RETURN result;
END //

DELIMITER ;

-- Uso de la función:
SELECT em.name AS Empresa,
COUNT(DISTINCT qp.product_id) AS ProductosCalificados
FROM companies em
JOIN quality_products qp ON em.id = qp.company_id
GROUP BY em.id, em.name
ORDER BY COUNT(DISTINCT qp.product_id) DESC
LIMIT 1;
```

### 4. Procedimientos Almacenados:

01.4 Registrar una nueva calificación y actualizar el promedio.
🧠 Explicación: Este procedimiento recibe *product_id*, *customer_id* y *rating*, inserta la nueva fila en rates, y recalcula automáticamente el promedio en la tabla products (campo *average_rating*).

```sql
DELIMITER //

CREATE PROCEDURE sp_simple_register_rating(
    IN p_product_id INT,
    IN p_customer_id INT,
    IN p_rating DECIMAL(3,1))
BEGIN
    DECLARE v_company_id VARCHAR(20);
    DECLARE v_avg_rating DECIMAL(3,2);
    
    SELECT company_id INTO v_company_id 
    FROM companyproducts 
    WHERE product_id = p_product_id 
    LIMIT 1;
    
    IF p_rating < 1 OR p_rating > 5 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'La calificación debe estar entre 1 y 5';
    END IF;
    
    INSERT INTO quality_products (
        product_id, customer_id, poll_id, 
        company_id, daterating, rating
    ) VALUES (
        p_product_id, p_customer_id, 1,
        v_company_id, NOW(), p_rating
    )
    ON DUPLICATE KEY UPDATE
        rating = VALUES(rating),
        daterating = NOW();
    
    SELECT ROUND(AVG(rating), 2) INTO v_avg_rating
    FROM quality_products 
    WHERE product_id = p_product_id;
    
    SELECT p_product_id AS IdProducto,
    v_avg_rating AS PromedioCalificación,
    'Calificación registrada' AS Mensaje;
END //

DELIMITER ;

-- Uso con CALL:
CALL sp_simple_register_rating(1, 1, 4.5); 
```

02.4 Insertar empresa y asociar productos por defecto.
🧠 Explicación: Este procedimiento inserta una empresa en *companies*, y luego vincula automáticamente productos predeterminados en *companyproducts*.

```sql
DELIMITER $$

CREATE PROCEDURE Insertar_empresaConProductos (
    IN p_id VARCHAR(20),
    IN p_type_id INT,
    IN p_name VARCHAR(80),
    IN p_category_id INT,
    IN p_city_id VARCHAR(10),
    IN p_audience_id INT,
    IN p_cellphone VARCHAR(15),
    IN p_email VARCHAR(80),
    IN p_typecompany_id INT
)
BEGIN
    INSERT INTO companies (id, type_id, name, category_id, city_id, audience_id, cellphone, email, typecompany_id) VALUES (
    p_id, p_type_id, p_name, p_category_id, p_city_id, p_audience_id, p_cellphone, p_email, p_typecompany_id);

    INSERT INTO companyproducts (company_id, product_id, price, unitmeasure_id, available_product)
    SELECT p_id,                 
    pr.id, pr.price, pr.unitofmeasure_id, 
    TRUE                  
    FROM products pr;
    
END$$

DELIMITER ;

-- Uso con CALL:
CALL Insertar_empresaConProductos ('COMP4', 4, 'Claro', 3, '05001', 2, '3003202991', 'claro@gmail.com', 1);

-- Revisar INSERT:
SELECT * 
FROM companies 
WHERE id = 'COMP4';

SELECT cp.company_id AS CompañíaNueva,
pro.name AS Producto, u.description AS UnidadDeMedida
FROM companyproducts cp
JOIN products pro ON cp.product_id = pro.id
JOIN unitofmeasure u ON cp.unitmeasure_id = u.id
WHERE cp.company_id = 'COMP4';
```

03.4 Añadir producto favorito validando duplicados.
🧠 Explicación: Verifica si el producto ya está en favoritos (*details_favorites*). Si no lo está, lo inserta. Evita duplicaciones silenciosamente.

```sql
DELIMITER $$

CREATE PROCEDURE ProductoFavorito (
    IN p_favorite_id INT,
    IN p_product_id INT
)
BEGIN
    IF NOT EXISTS (
        SELECT 1
        FROM details_favorites
        WHERE favorite_id = p_favorite_id
        AND product_id = p_product_id
    ) THEN
        INSERT INTO details_favorites (favorite_id, product_id)
        VALUES (p_favorite_id, p_product_id);
    END IF;
END$$

DELIMITER ;

-- Uso con CALL:
CALL ProductoFavorito (1, 5);

-- Revisar INSERT:
SELECT * 
FROM details_favorites df
WHERE favorite_id = 1 AND product_id = 5;

SELECT * 
FROM details_favorites df;
```

04.4 Generar resumen mensual de calificaciones por empresa.
🧠 Explicación: Hace una consulta agregada con *AVG(rating)* por empresa, y guarda los resultados en una tabla de resumen tipo *resumen_calificaciones*.

```sql
DELIMITER $$

CREATE PROCEDURE ResumenCalificacionesMensual (
    IN p_anio INT,
    IN p_mes INT
)
BEGIN
    DELETE FROM resumen_calificaciones
    WHERE año = p_anio AND mes = p_mes;

    INSERT INTO resumen_calificaciones (company_id, Año, Mes, promedio_calificacion)
    SELECT company_id AS IdEmpresa,
    p_anio AS Año, p_mes AS Mes,
    AVG(rating) AS promedio_calificacion
    FROM quality_products
    WHERE YEAR(daterating) = p_anio AND MONTH(daterating) = p_mes
    GROUP BY company_id;
END$$

DELIMITER ;

-- Uso con CALL:
SELECT company_id, daterating, rating -- Verificar datos como fecha (año y mes) 
FROM quality_products 
LIMIT 10;

CALL ResumenCalificacionesMensual (2025, 7);

SELECT em.name AS Empresa,
r.año AS Año, r.mes AS Mes, r.promedio_calificacion AS PromedioCalificacion
FROM resumen_calificaciones r
JOIN companies em ON r.company_id = em.id
ORDER BY r.año DESC, r.mes DESC;
```

05.4 Calcular beneficios activos por membresía.
🧠 Explicación: Consulta *membershipbenefits* junto con *membershipperiods*, y devuelve una lista de beneficios vigentes según la fecha actual.

```sql
DELIMITER $$

CREATE PROCEDURE BeneficioasActivos ()
BEGIN
    SELECT mb.membership_id AS IdMembresía,
    m.name AS Membresía,
    b.description AS TipoBeneficio, b.detail AS DetalleBeneficio,
    a.description AS Audiencia,
    p.name AS Periodo
    FROM membershipbenefits mb
    JOIN memberships m ON mb.membership_id = m.id
    JOIN benefits b ON mb.benefit_id = b.id
    JOIN audiences a ON mb.audience_id = a.id
    JOIN periods p ON mb.period_id = p.id
    JOIN membershipperiods mp ON mb.membership_id = mp.membership_id AND mb.period_id = mp.period_id
    ORDER BY mb.membership_id, b.id;
END$$

DELIMITER ;

-- uso con CALL:
CALL BeneficioasActivos ();
```

06.4 Eliminar productos huérfanos.
🧠 Explicación: Elimina productos de la tabla *products* que no tienen relación ni en *rates* ni en *companyproducts*.

```sql
DELIMITER $$

CREATE PROCEDURE EliminarProductosHuerfanos()
BEGIN
    DELETE FROM products
    WHERE id NOT IN (SELECT DISTINCT product_id FROM companyproducts)
      AND id NOT IN (SELECT DISTINCT product_id FROM quality_products);
END$$

DELIMITER ;

-- uso con CALL:
CALL EliminarProductosHuerfanos();

-- Revisar PROCEDURE:
SELECT * 
FROM products pro   
WHERE id NOT IN (SELECT product_id FROM companyproducts) AND id NOT IN (SELECT product_id FROM quality_products);
-- (Empty set) Todos los productos tienen relación con *rates* y *companyproducts*.
```

07.4 Actualizar precios de productos por categoría
🧠 Explicación: Recibe un *categoria_id* y un factor (por ejemplo 1.05), y multiplica todos los precios por ese factor en la tabla *companyproducts*.

```sql
DELIMITER $$

CREATE PROCEDURE ActualizarPrecios (
    IN p_categoria_id INT,
    IN p_factor DECIMAL(5,2),
    IN p_usuario VARCHAR(50)
)
BEGIN
    INSERT INTO cambios_precios (
    producto_id, empresa_id,
    precio_anterior, precio_nuevo,
    categoria_id, factor, usuario
    )
    SELECT cp.product_id, cp.company_id, cp.price,
    cp.price * p_factor, 
    p.category_id, p_factor, p_usuario
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    WHERE p.category_id = p_categoria_id;

    UPDATE companyproducts cp
    JOIN products p ON cp.product_id = p.id
    SET cp.price = cp.price * p_factor
    WHERE p.category_id = p_categoria_id;
END$$

DELIMITER ;

-- Uso con CALL:
CALL ActualizarPrecios(1, 1.15, 'admin');

-- Verificar INSERTS:
SELECT * 
FROM cambios_precios 
ORDER BY fecha_cambio DESC;
```

08.4 Validar inconsistencia entre rates y quality_products.
🧠 Explicación: Busca calificaciones (*rates*) que no tengan entrada correspondiente en *quality_products*. Inserta el error en una tabla *errores_log*.

```sql
DELIMITER $$

CREATE PROCEDURE ValidarInconsistencia ()
BEGIN
    INSERT INTO errores_log (
        tipo_error,
        descripcion,
        entidad_origen,
        id_origen
    )
    SELECT 
        'Inconsistencia: rate sin quality',
        CONCAT('No existe en quality_products la calificación de cliente ', r.customer_id,
        ', empresa ', r.company_id,
        ', encuesta ', r.poll_id),
        'rates',
        NULL
    FROM rates r
    WHERE NOT EXISTS (
        SELECT 1
        FROM quality_products qp
        WHERE qp.customer_id = r.customer_id AND qp.company_id = r.company_id AND qp.poll_id = r.poll_id
    );
END$$

DELIMITER ;

-- Uso con CALL:
CALL ValidarInconsistencia();

-- Verificar INSERTS:
SELECT * 
FROM errores_log 
ORDER BY fecha_error DESC;
-- (Empty Set) No hay registros huérfanos, es decir, todos los *rates* tienen entrada en *quality_products*.
```

09.4 Asignar beneficios a nuevas audiencias.
🧠 Explicación: Recibe un *benefit_id* y *audience_id*, verifica si ya existe el registro, y si no, lo inserta en *audiencebenefits*.

```sql
DELIMITER $$

CREATE PROCEDURE AsignarBeneficio (
    IN p_benefit_id INT,
    IN p_audience_id INT
)
BEGIN
    IF NOT EXISTS (
        SELECT 1
        FROM audiencebenefits
        WHERE benefit_id = p_benefit_id
          AND audience_id = p_audience_id
    ) THEN
        INSERT INTO audiencebenefits (benefit_id, audience_id)
        VALUES (p_benefit_id, p_audience_id);
    END IF;
END$$

DELIMITER ;

-- Uso con CALL:
CALL AsignarBeneficio(1, 1);

-- Verificar INSERTS:
SELECT * 
FROM audiencebenefits
```

10.4 Activar planes de membresía vencidos con pago confirmado.
🧠 Explicación: Actualiza el campo *status* a '*ACTIVA*' en *membershipperiods* donde la fecha haya vencido pero el campo *pago_confirmado* sea *TRUE*.

```sql
DELIMITER $$

CREATE PROCEDURE ActivarMembresiasVencidasConPago()
BEGIN
    UPDATE membershipperiods
    SET status = 'ACTIVA'
    WHERE fecha_fin < CURDATE()
      AND pago_confirmado = TRUE
      AND (status IS NULL OR status <> 'ACTIVA');

    SELECT ROW_COUNT() AS membresias_activadas;
END$$

DELIMITER ;

-- Uso con CALL:
CALL ActivarMembresiasVencidasConPago();
```

11.4 Listar productos favoritos del cliente con su calificación.
🧠 Explicación: Consulta todos los productos favoritos del cliente y muestra el promedio de calificación de cada uno, uniendo *favorites*, *rates* y *products*.

```sql
DELIMITER $$

CREATE PROCEDURE ListarProductosFavoritosConCalificacion (
    IN p_customer_id INT
)
BEGIN
    SELECT 
        p.id AS producto_id,
        p.name AS nombre_producto,
        ROUND(AVG(qp.rating), 2) AS promedio_calificacion
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    JOIN products p ON df.product_id = p.id
    LEFT JOIN quality_products qp ON p.id = qp.product_id
    WHERE f.customer_id = p_customer_id
    GROUP BY p.id, p.name
    ORDER BY promedio_calificacion DESC;
END$$

DELIMITER ;

-- Uso con CALL:
CALL ListarProductosFavoritosConCalificacion(1);
```

12.4 Registrar encuesta y sus preguntas asociadas.
🧠 Explicación: Inserta la encuesta principal en *polls* y luego cada una de sus preguntas en otra tabla relacionada como *poll_questions*.

```sql
DELIMITER $$

CREATE PROCEDURE RegistrarEncuestaConPreguntas()
BEGIN
    DECLARE new_poll_id INT;

    INSERT INTO polls (name, description, isactive, categorypoll_id)
    VALUES ('Encuesta de Satisfacción 360', 'Evalúa experiencia general del cliente', 1, 1);

    SET new_poll_id = LAST_INSERT_ID();

    INSERT INTO poll_questions (poll_id, question)
    VALUES 
        (new_poll_id, '¿Qué tan satisfecho está con el producto?'),
        (new_poll_id, '¿Cómo califica la atención al cliente?'),
        (new_poll_id, '¿Qué mejoraría en el servicio?');
END$$

DELIMITER ;

-- Uso con CALL:
CALL RegistrarEncuestaConPreguntas();

-- Verificación:
SELECT * FROM polls ORDER BY id DESC;
SELECT * FROM poll_questions WHERE poll_id = LAST_INSERT_ID();
```

13.4 Eliminar favoritos antiguos sin calificaciones.
🧠 Explicación: Filtra productos favoritos que no tienen calificaciones recientes y fueron añadidos hace más de 12 meses, y los elimina de *details_favorites*.

```sql
DELIMITER $$

CREATE PROCEDURE EliminarFavoritosAntiguosSinCalificaciones()
BEGIN
    DELETE FROM details_favorites df
    WHERE df.created_at < CURDATE() - INTERVAL 12 MONTH
      AND df.product_id NOT IN (
          SELECT DISTINCT qp.product_id
          FROM quality_products qp
          WHERE qp.daterating >= CURDATE() - INTERVAL 12 MONTH
      );
END$$

DELIMITER ;

-- Uso con CALL:
CALL EliminarFavoritosAntiguosSinCalificaciones();

-- Verificación:
SELECT * 
FROM details_favorites 
WHERE created_at < CURDATE() - INTERVAL 12 MONTH;
```

14.4 Asociar beneficios automáticamente por audiencia.
🧠 Explicación: Inserta en *audiencebenefits* todos los beneficios que apliquen según una lógica predeterminada (por ejemplo, por tipo de usuario).

```sql
DELIMITER $$

CREATE PROCEDURE AsociarBeneficiosPorAudiencia()
BEGIN
    INSERT IGNORE INTO audiencebenefits (benefit_id, audience_id)
    SELECT id, 1
    FROM benefits
    WHERE description LIKE '%Cliente%';

    INSERT IGNORE INTO audiencebenefits (benefit_id, audience_id)
    SELECT id, 2
    FROM benefits
    WHERE description LIKE '%Empresa%';

    INSERT IGNORE INTO audiencebenefits (benefit_id, audience_id)
    SELECT id, 3
    FROM benefits
    WHERE description LIKE '%Distribuidor%';
END$$

DELIMITER ;

-- Uso con CALL:
CALL AsociarBeneficiosPorAudiencia();

-- Verificación:
SELECT ab.audience_id AS Identificador, a.description AS Audiencia, b.description AS Beneficio
FROM audiencebenefits ab
JOIN audiences a ON ab.audience_id = a.id
JOIN benefits b ON ab.benefit_id = b.id
ORDER BY ab.audience_id;
```

15.4 Historial de cambios de precio.
🧠 Explicación: Cada vez que se cambia un precio, el procedimiento compara el anterior con el nuevo y guarda un registro en una tabla *cambios_precios*.

```sql
DELIMITER $$

CREATE PROCEDURE ActualizarPrecioProducto (
    IN p_company_id VARCHAR(20),
    IN p_product_id INT,
    IN p_nuevo_precio DECIMAL(10,2),
    IN p_usuario VARCHAR(50)
)
BEGIN
    DECLARE v_precio_actual DECIMAL(10,2);
    DECLARE v_categoria_id INT;
    DECLARE v_factor DECIMAL(10,4);
    DECLARE v_existe INT;

    SELECT COUNT(*) INTO v_existe
    FROM companyproducts
    WHERE company_id = p_company_id AND product_id = p_product_id;

    IF v_existe = 1 THEN
        SELECT price INTO v_precio_actual
        FROM companyproducts
        WHERE company_id = p_company_id AND product_id = p_product_id;

        IF v_precio_actual <> p_nuevo_precio THEN
            SET v_factor = p_nuevo_precio / v_precio_actual;

            SELECT category_id INTO v_categoria_id
            FROM products
            WHERE id = p_product_id;

            UPDATE companyproducts
            SET price = p_nuevo_precio
            WHERE company_id = p_company_id AND product_id = p_product_id;

            INSERT INTO cambios_precios (
                producto_id, empresa_id, precio_anterior, precio_nuevo,
                categoria_id, factor, usuario
            ) VALUES (
                p_product_id, p_company_id, v_precio_actual, p_nuevo_precio,
                v_categoria_id, v_factor, p_usuario
            );
        END IF;
    END IF;
END$$

DELIMITER ;

-- Uso con CALL:
CALL ActualizarPrecioProducto('COMP1', 1, 16000.00, 'admin');

-- Verificación:
SELECT * FROM companyproducts 
WHERE company_id = 'COMP1' AND product_id = 1;

SELECT * FROM cambios_precios 
WHERE empresa_id = 'COMP1' AND producto_id = 1
ORDER BY fecha_cambio DESC;
```

16.4 Registrar encuesta activa automáticamente.
🧠 Explicación: Inserta una encuesta en *polls* con el campo *status* = 'activa' y una fecha de inicio en *NOW()*.

```sql
-- Modificar TABE polls:
ALTER TABLE polls 
ADD COLUMN status VARCHAR(20) DEFAULT 'inactiva',
ADD COLUMN fecha_inicio DATETIME DEFAULT NULL;


DELIMITER $$

CREATE PROCEDURE RegistrarEncuestaActiva (
    IN p_name VARCHAR(80),
    IN p_description TEXT,
    IN p_categorypoll_id INT
)
BEGIN
    INSERT INTO polls (
    name, description, isactive, status, fecha_inicio, categorypoll_id
    )
    VALUES (
    p_name, p_description, 1, 'activa', NOW(), p_categorypoll_id
    );
END$$

DELIMITER ;

-- Uso con CALL:
CALL RegistrarEncuestaActiva('Encuesta de Opinión', 'Recoge la percepción de los clientes sobre nuestros servicios.', 1);

-- Verificación:
SELECT * FROM polls
ORDER BY id DESC
LIMIT 1;
```

17.4 Actualizar unidad de medida de productos sin afectar ventas.
🧠 Explicación: Verifica si el producto no ha sido vendido, y si es así, permite actualizar su *unit_id*.

```sql
DELIMITER $$

CREATE PROCEDURE ActualizarUnidadMedidaSiNoUsado (
    IN p_product_id INT,
    IN p_nueva_unidad_id INT
)
BEGIN
    DECLARE v_usado INT DEFAULT 0;

    SELECT COUNT(*) INTO v_usado
    FROM quality_products
    WHERE product_id = p_product_id;

    IF v_usado = 0 THEN
        UPDATE products
        SET unitofmeasure_id = p_nueva_unidad_id
        WHERE id = p_product_id;

        SELECT 'Unidad de medida actualizada.' AS mensaje;
    ELSE
        SELECT 'No se puede actualizar: el producto ya ha sido utilizado en calificaciones o ventas.' AS mensaje;
    END IF;
END$$

DELIMITER ;

-- Uso con CALL:
CALL ActualizarUnidadMedidaSiNoUsado(5, 2);

-- Verificación:
SELECT id, name, unitofmeasure_id 
FROM products 
WHERE id = 5;
```

18.4 Recalcular promedios de calidad semanalmente.
🧠 Explicación: Hace un *AVG(rating)* agrupado por producto y lo actualiza en *products*.

```sql
-- Modificar TABLE products:
ALTER TABLE products ADD COLUMN promedio_calidad DOUBLE DEFAULT NULL;

DELIMITER $$

CREATE PROCEDURE RecalcularPromediosCalidad()
BEGIN
    UPDATE products p
    SET promedio_calidad = (
        SELECT ROUND(AVG(qp.rating), 2)
        FROM quality_products qp
        WHERE qp.product_id = p.id
    );
END$$

DELIMITER ;

-- Uso con CALL:
CALL RecalcularPromediosCalidad();

-- Verificación:
SELECT pro.id, pro.name AS Producto, pro.promedio_calidad AS PromedioCalidad
FROM products pro
ORDER BY PromedioCalidad DESC;
```

19.4 Validar claves foráneas entre calificaciones y encuestas.
🧠 Explicación: Busca registros en *rates* con *poll_id* que no existen en *polls*, y los reporta.

```sql
DELIMITER $$

CREATE PROCEDURE ValidarClavesCalificacionesEncuestas()
BEGIN
    INSERT INTO errores_log (
        tipo_error,
        descripcion,
        entidad_origen,
        id_origen
    )
    SELECT 
        'Poll_id inválido en rates',
        CONCAT('El poll_id = ', r.poll_id, ' no existe en polls.'),
        'rates',
        r.poll_id
    FROM rates r
    WHERE r.poll_id NOT IN (
        SELECT id FROM polls
    );
END$$

DELIMITER ;

-- Uso con CALL:
CALL ValidarClavesCalificacionesEncuestas();

-- Verificación:
SELECT * 
FROM errores_log
WHERE tipo_error = 'Poll_id inválido en rates'
ORDER BY fecha_error DESC;

-- (Empty Set) No hay ningún *poll_id* que se invalide en *rates*.
```

20.4 Generar el top 10 de productos más calificados por ciudad.
🧠 Explicación: Agrupa las calificaciones por ciudad (a través de la empresa que lo vende) y selecciona los 10 productos con más evaluaciones.

```sql
DELIMITER $$

CREATE PROCEDURE Top10ProductosPorCiudad()
BEGIN
    SELECT *
    FROM (
        SELECT
            ci.name AS ciudad,
            p.name AS producto,
            COUNT(qp.rating) AS total_calificaciones,
            RANK() OVER (PARTITION BY ci.code ORDER BY COUNT(qp.rating) DESC) AS ranking
        FROM quality_products qp
        JOIN products p ON qp.product_id = p.id
        JOIN companies c ON qp.company_id = c.id
        JOIN citiesormunicipalities ci ON c.city_id = ci.code
        GROUP BY ci.code, ci.name, p.id, p.name
    ) AS sub
    WHERE ranking <= 10
    ORDER BY ciudad, ranking;
END$$

DELIMITER ;

-- Uso con CALL:
CALL Top10ProductosPorCiudad();
```

### 5. Triggers:

01.5 Actualizar la fecha de modificación de un producto.
🧠 Explicación: Cada vez que se actualiza un producto, queremos que el campo *updated_at* se actualice automáticamente con la fecha actual *(NOW())*, sin tener que hacerlo manualmente desde la app.
🔁 Se usa un *BEFORE UPDATE*.

```sql
-- Modificar la TABLE *products*:
ALTER TABLE products
ADD COLUMN updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
ON UPDATE CURRENT_TIMESTAMP;

DELIMITER $$

CREATE TRIGGER before_update_products_timestamp
BEFORE UPDATE ON products
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END$$

DELIMITER ;

-- Verificación:
SELECT id, name, updated_at FROM products WHERE id = 1;
UPDATE products SET name = 'Arroz Integral' WHERE id = 1;
SELECT id, name, updated_at FROM products WHERE id = 1;
```

02.5 Registrar log cuando un cliente califica un producto.
🧠 Explicación: Cuando alguien inserta una fila en *rates*, el *trigger* crea automáticamente un registro en *log_acciones* con la información del cliente y producto calificado.
🔁 Se usa un *AFTER INSERT* sobre *rates*.

```sql
DELIMITER $$

CREATE TRIGGER after_insert_rates_log
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE v_product_id INT;

    SELECT product_id INTO v_product_id
    FROM quality_products
    WHERE poll_id = NEW.poll_id
      AND company_id = NEW.company_id
      AND customer_id = NEW.customer_id
    LIMIT 1;


    INSERT INTO log_acciones (
    tipo_accion,
    descripcion,
    customer_id,
    producto_id
    ) VALUES (
        'Calificación de producto',
        CONCAT('El cliente ', NEW.customer_id, ' calificó el producto ', v_product_id),
        NEW.customer_id,
        v_product_id
    );
END$$

DELIMITER ;

-- INSERTS:
INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating)
VALUES (1, 'COMP1', 1, NOW(), 4.5);

-- Verificación:
SELECT * 
FROM log_acciones 
ORDER BY fecha_accion DESC;
```

03.5 Impedir insertar productos sin unidad de medida.
🧠 Explicación: Antes de guardar un nuevo producto, el *trigger* revisa si *unit_id* es *NULL*. Si lo es, lanza un error con *SIGNAL*.
🔁 Se usa un *BEFORE INSERT*.

```sql
DELIMITER $$

CREATE TRIGGER before_insert_products_check_unit
BEFORE INSERT ON products
FOR EACH ROW
BEGIN
    IF NEW.unitofmeasure_id IS NULL THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se puede insertar un producto sin unidad de medida.';
    END IF;
END$$

DELIMITER ;

-- INSERT:
INSERT INTO products (name, unitofmeasure_id, detail, price, category_id, image) VALUES (
'Producto Válido', 1, 'Arroz blanco empacado', 3200.00, 1, 'arroz_blanco.jpg'
);

-- Verificación:
SELECT * 
FROM products
WHERE name = 'Producto Válido';
```

04.5 Validar calificaciones no mayores a 5.
🧠 Explicación: Si alguien intenta insertar una calificación de 6 o más, se bloquea automáticamente. Esto evita errores o trampa.
🔁 Se usa un *BEFORE INSERT*.

```sql
DELIMITER $$

CREATE TRIGGER before_insert_rates_limitar_rating
BEFORE INSERT ON rates
FOR EACH ROW
BEGIN
    IF NEW.rating > 5 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'La calificación no puede ser mayor a 5.';
    END IF;
END$$

DELIMITER ;

-- Verificación del ERROR:
INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating)
VALUES (1, 'COMP1', 3, NOW(), 6.5);

INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating)
VALUES (2, 'COMP1', 2, NOW(), 6.5); 
```

05.5 Actualizar estado de membresía cuando vence.
🧠 Explicación: Cuando se actualiza un periodo de membresía (*membershipperiods*), si *end_date* ya pasó, se puede cambiar el campo *status* a '*INACTIVA*'.
🔁 *AFTER UPDATE* o *BEFORE UPDATE* dependiendo de la lógica.

```sql
DELIMITER $$

CREATE TRIGGER before_update_membershipperiods_status
BEFORE UPDATE ON membershipperiods
FOR EACH ROW
BEGIN
    IF NEW.fecha_fin < CURDATE() THEN
        SET NEW.status = 'INACTIVA';
    END IF;
END$$

DELIMITER ;

-- Verificación:
UPDATE membershipperiods
SET fecha_fin = fecha_fin - INTERVAL 0 DAY 
WHERE membership_id = 1 AND period_id = 2;

SELECT membership_id, period_id, fecha_fin, status
FROM membershipperiods
WHERE membership_id = 1 AND period_id = 2;
```

06.5 Evitar duplicados de productos por empresa.
🧠 Explicación: Antes de insertar un nuevo producto en *companyproducts*, el *trigger* puede consultar si ya existe uno con el mismo *product_id* y *company_id*.
🔁 *BEFORE INSERT*.

```sql
DELIMITER $$

CREATE TRIGGER before_insert_companyproducts_no_duplicado
BEFORE INSERT ON companyproducts
FOR EACH ROW
BEGIN
    DECLARE v_existe INT;

    SELECT COUNT(*) INTO v_existe
    FROM companyproducts
    WHERE company_id = NEW.company_id
      AND product_id = NEW.product_id;

    IF v_existe > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Este producto ya está registrado para esta empresa.';
    END IF;
END$$

DELIMITER ;

-- INSERT y Verificación:
INSERT INTO companyproducts (company_id, product_id, price, unitmeasure_id, available_product)
VALUES ('COMP1', 9, 8500, 1, 1);

INSERT INTO companyproducts (company_id, product_id, price, unitmeasure_id, available_product)
VALUES ('COMP1', 9, 8500, 1, 1); -- Al ejecutar de nuevo, generará el error del TRIGGER:

-- Error:
ERROR 1644 (45000): Este producto ya está registrado para esta empresa.
```

07.5 Enviar notificación al añadir un favorito.
🧠 Explicación: Después de un *INSERT* en *details_favorites*, el *trigger* agrega un mensaje a una tabla notificaciones.
🔁 *AFTER INSERT*.

```sql
DELIMITER $$

CREATE TRIGGER after_insert_detailsfavorites_notificacion
AFTER INSERT ON details_favorites
FOR EACH ROW
BEGIN
    DECLARE v_customer_id INT;
    DECLARE v_product_name VARCHAR(100);

    SELECT f.customer_id INTO v_customer_id
    FROM favorites f
    WHERE f.id = NEW.favorite_id;

    SELECT p.name INTO v_product_name
    FROM products p
    WHERE p.id = NEW.product_id;

    INSERT INTO notificaciones (mensaje)
    VALUES (
        CONCAT('El cliente ', v_customer_id, ' añadió el producto "', v_product_name, '" a sus favoritos.')
    );
END$$

DELIMITER ;

-- INSERT:
INSERT INTO details_favorites (favorite_id, product_id, created_at)
VALUES (1, 4, CURDATE());

-- Verificación:
SELECT * FROM notificaciones ORDER BY fecha DESC;
```

08.5 Insertar fila en quality_products tras calificación.
🧠 Explicación: Al insertar una nueva calificación en *rates*, se crea automáticamente un registro en *quality_products* para mantener métricas de calidad.
🔁 *AFTER INSERT*.

```sql
DELIMITER $$

CREATE TRIGGER after_insert_rates_to_quality_products
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE v_product_id INT;

    SELECT product_id INTO v_product_id
    FROM quality_products
    WHERE poll_id = NEW.poll_id
      AND company_id = NEW.company_id
      AND customer_id = NEW.customer_id
    LIMIT 1;

    IF v_product_id IS NOT NULL THEN
        INSERT INTO quality_products (
            product_id,
            company_id,
            poll_id,
            customer_id,
            rating,
            daterating
        ) VALUES (
            v_product_id,
            NEW.company_id,
            NEW.poll_id,
            NEW.customer_id,
            NEW.rating,
            NEW.daterating
        );
    END IF;
END$$

DELIMITER ;

-- INSERT:
SELECT * 
FROM rates
WHERE customer_id = 2 AND company_id = 'COMP1' AND poll_id = 3;

-- Verificación:
SELECT * 
FROM quality_products
WHERE customer_id = 2 AND company_id = 'COMP1' AND poll_id = 3
ORDER BY daterating DESC;
-- (Empty Set) El *trigger* se ejecutó correctamente pero no se hizo ningún INSERT pues es el primero que se hace.
```

09.5 Eliminar favoritos si se elimina el producto.
🧠 Explicación: Cuando se borra un producto, el *trigger* elimina las filas en *details_favorites* donde estaba ese producto.
🔁 *AFTER DELETE* en products.

```sql
DELIMITER $$

CREATE TRIGGER after_delete_products_cleanup_favorites
AFTER DELETE ON products
FOR EACH ROW
BEGIN
    DELETE FROM details_favorites
    WHERE product_id = OLD.id;
END$$

DELIMITER ;

-- Verificación:
SELECT * 
FROM details_favorites 
WHERE product_id = 4;

SELECT * 
FROM details_favorites 
WHERE product_id = 4;
```

10.5 Bloquear modificación de audiencias activas.
🧠 Explicación: Si un usuario intenta modificar una audiencia que está en uso, el *trigger* lanza un error con *SIGNAL*.
🔁 *BEFORE UPDATE*.

```sql
-- Modificar TABLE audiences:
ALTER TABLE audiences ADD COLUMN status VARCHAR(20) DEFAULT 'ACTIVA';

DELIMITER $$

CREATE TRIGGER before_update_audiences_bloquear_activas
BEFORE UPDATE ON audiences
FOR EACH ROW
BEGIN
    IF OLD.status = 'ACTIVA' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se puede modificar una audiencia activa.';
    END IF;
END$$

DELIMITER ;

-- Verificación del ERROR:
UPDATE audiences
SET description = 'Audiencia Modificada'
WHERE id = 1;
```

11.5 Recalcular promedio de calidad del producto tras nueva evaluación.
🧠 Explicación: Después de insertar en *rates*, el *trigger* actualiza el campo *average_rating* del producto usando *AVG()*.
🔁 *AFTER INSERT*.

```sql
-- Modificar TABLE products (si no se ha hecho antes):
ALTER TABLE products ADD COLUMN promedio_calidad DOUBLE DEFAULT NULL;

DELIMITER $$

CREATE TRIGGER after_insert_rates_recalcular_promedio
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE v_product_id INT;

    SELECT product_id INTO v_product_id
    FROM quality_products
    WHERE customer_id = NEW.customer_id
      AND company_id = NEW.company_id
      AND poll_id = NEW.poll_id
    LIMIT 1;

    IF v_product_id IS NOT NULL THEN
        UPDATE products
        SET promedio_calidad = (
            SELECT ROUND(AVG(r.rating), 2)
            FROM rates r
            JOIN quality_products qp
              ON r.customer_id = qp.customer_id
             AND r.company_id = qp.company_id
             AND r.poll_id = qp.poll_id
            WHERE qp.product_id = v_product_id
        )
        WHERE id = v_product_id;
    END IF;
END$$

DELIMITER ;

-- Verificación:
DELETE FROM rates
WHERE customer_id = 3 AND company_id = 'COMP1' AND poll_id = 1;

INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating)
VALUES (3, 'COMP1', 1, NOW(), 4.7);

SELECT id, name, promedio_calidad
FROM products
WHERE id = 1;
```

12.5 Registrar asignación de nuevo beneficio.
🧠 Explicación: Cuando se hace *INSERT* en *membershipbenefits* o *audiencebenefits*, se agrega un log en *bitacora*.

```sql
-- Primer *TRIGGER*:
DELIMITER $$

CREATE TRIGGER after_insert_membershipbenefits_bitacora
AFTER INSERT ON membershipbenefits
FOR EACH ROW
BEGIN
    INSERT INTO bitacora (mensaje)
    VALUES (
        CONCAT('Se asignó el beneficio ', NEW.benefit_id, ' a la membresía ', NEW.membership_id)
    );
END$$

DELIMITER ;

-- Segundo *TRIGGER*:
DELIMITER $$

CREATE TRIGGER after_insert_audiencebenefits_bitacora
AFTER INSERT ON audiencebenefits
FOR EACH ROW
BEGIN
    INSERT INTO bitacora (mensaje)
    VALUES (
        CONCAT('Se asignó el beneficio ', NEW.benefit_id, ' a la audiencia ', NEW.audience_id)
    );
END$$

DELIMITER ;

-- INSERTS y Verificación:
INSERT INTO membershipbenefits (membership_id, period_id, benefit_id, audience_id)
VALUES (1, 1, 2, 1);

INSERT INTO audiencebenefits (audience_id, benefit_id)
VALUES (3, 1);

SELECT * 
FROM bitacora 
ORDER BY fecha DESC;
```

13.5 Impedir doble calificación por parte del cliente.
🧠 Explicación: Antes de insertar en *rates*, el *trigger* verifica si ya existe una calificación de ese *customer_id* y *product_id*.

```sql
DELIMITER $$

CREATE TRIGGER before_insert_rates_prevenir_doble_calificacion
BEFORE INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE v_product_id INT;

    SELECT product_id INTO v_product_id
    FROM quality_products
    WHERE customer_id = NEW.customer_id
      AND company_id = NEW.company_id
      AND poll_id = NEW.poll_id
    LIMIT 1;

    IF EXISTS (
        SELECT 1
        FROM quality_products qp
        JOIN rates r ON qp.customer_id = r.customer_id
                    AND qp.company_id = r.company_id
                    AND qp.poll_id = r.poll_id
        WHERE qp.product_id = v_product_id
          AND r.customer_id = NEW.customer_id
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'El cliente ya calificó este producto anteriormente.';
    END IF;
END$$

DELIMITER ;

-- Verificación:
INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating)
VALUES (3, 'COMP1', 2, NOW(), 4.2);
```

14.5 Validar correos duplicados en clientes.
🧠 Explicación: Verifica, antes del *INSERT*, si el correo ya existe en la tabla *customers*. Si sí, lanza un error.

```sql
DELIMITER $$

CREATE TRIGGER before_insert_customers_cellphone_unico
BEFORE INSERT ON customers
FOR EACH ROW
BEGIN
    IF EXISTS (
        SELECT 1 FROM customers
        WHERE cellphone = NEW.cellphone
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Ya existe un cliente con este número de celular.';
    END IF;
END$$

DELIMITER ;
-- Verificación del ERROR:
INSERT INTO customers (name, city_id, audience_id, cellphone, email, membership_active) VALUES
('Andrés Suárez', '05001', 1, '3138430142', 'andresuarez@mail.com', 1); 
```

15.5 Eliminar detalles de favoritos huérfanos.
🧠 Explicación: Si se elimina un registro de *favorites*, se borran automáticamente sus detalles asociados.

```sql
DELIMITER $$

CREATE TRIGGER after_delete_favorites_cleanup_details
AFTER DELETE ON favorites
FOR EACH ROW
BEGIN
    DELETE FROM details_favorites
    WHERE favorite_id = OLD.id;
END$$

DELIMITER ;

-- Verificación:
DELETE FROM details_favorites 
WHERE favorite_id = 1;

SELECT * 
FROM details_favorites 
WHERE favorite_id = 1;
-- (Empty Set) Se ha eliminado el favorito correctamente.
```

16.5 Actualizar campo updated_at en companies.
🧠 Explicación: Como en productos, actualiza automáticamente la fecha de última modificación cada vez que se cambia algún dato.

```sql
-- Modificar la TABLE companies:
ALTER TABLE companies
ADD COLUMN updated_at DATETIME DEFAULT NULL;

DELIMITER $$

CREATE TRIGGER before_update_companies_set_updated_at
BEFORE UPDATE ON companies
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END$$

DELIMITER ;

-- Verficación:
UPDATE companies 
SET name = 'Empresa Actualizada' 
WHERE id = 'COMP1';

SELECT id, name, updated_at 
FROM companies 
WHERE id = 'COMP1';
```

17.5 Impedir borrar ciudad si hay empresas activas.
🧠 Explicación: Antes de hacer *DELETE* en *citiesormunicipalities*, el *trigger* revisa si hay empresas registradas en esa ciudad.

```sql
DELIMITER $$

CREATE TRIGGER before_delete_city_verificar_empresas
BEFORE DELETE ON citiesormunicipalities
FOR EACH ROW
BEGIN
    IF EXISTS (
        SELECT 1 FROM companies WHERE city_id = OLD.code
    ) THEN
        INSERT INTO errores_log (
            tipo_error,
            descripcion,
            entidad_origen,
            id_origen
        ) VALUES (
            'Eliminación bloqueada',
            CONCAT('Se intentó eliminar la ciudad "', OLD.name, '" (código ', OLD.code, ') con empresas registradas.'),
            'citiesormunicipalities',
            OLD.code
        );

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se puede eliminar la ciudad: existen empresas registradas en ella.';
    END IF;
END$$

DELIMITER ;


-- Verificación del ERROR:
SELECT * 
FROM companies 
WHERE city_id = '05001';

DELETE FROM citiesormunicipalities 
WHERE code = '05001';
```

18.5 Registrar cambios de estado en encuestas.
🧠 Explicación: Cada vez que se actualiza el campo *status* en *polls*, el *trigger* guarda la fecha, nuevo estado y usuario en un *log*.

```sql
DELIMITER $$

CREATE TRIGGER after_update_isactive_poll
AFTER UPDATE ON polls
FOR EACH ROW
BEGIN
    IF NOT (OLD.isactive <=> NEW.isactive) THEN
        INSERT INTO log_cambios_encuestas (poll_id, nuevo_estado, usuario)
        VALUES (NEW.id, NEW.isactive, 'sistema');
    END IF;
END$$

DELIMITER ;

-- Verificación con UPDATE:
UPDATE polls 
SET isactive = 0 
WHERE id = 1;

SELECT * 
FROM log_cambios_encuestas 
ORDER BY fecha_cambio DESC;
```

19.5 Sincronizar *rates* y *quality_products*.
🧠 Explicación: Inserta o actualiza la calidad del producto en *quality_products* cada vez que se inserta una nueva calificación.

```sql
DELIMITER $$

CREATE TRIGGER after_insert_rates_sync_quality_products
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE v_product_id INT;

    SELECT product_id INTO v_product_id
    FROM poll_product
    WHERE poll_id = NEW.poll_id;

    IF v_product_id IS NOT NULL THEN

        IF EXISTS (
            SELECT 1 FROM quality_products
            WHERE customer_id = NEW.customer_id
              AND company_id = NEW.company_id
              AND poll_id = NEW.poll_id
        ) THEN
            UPDATE quality_products
            SET rating = NEW.rating,
                daterating = NEW.daterating
            WHERE customer_id = NEW.customer_id
              AND company_id = NEW.company_id
              AND poll_id = NEW.poll_id;
        ELSE
            INSERT INTO quality_products (
                product_id, company_id, poll_id, customer_id, rating, daterating
            ) VALUES (
                v_product_id, NEW.company_id, NEW.poll_id, NEW.customer_id, NEW.rating, NEW.daterating
            );
        END IF;

    END IF;
END$$

DELIMITER ;

-- Verificación:

INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating) VALUES 
(3, 'COMP1', 1, NOW(), 4.8);

SELECT * 
FROM quality_products
WHERE customer_id = 3 AND company_id = 'COMP1' AND poll_id = 1;
```

20.5 Eliminar productos sin relación a empresas.
🧠 Explicación: Después de borrar la última relación entre un producto y una empresa (*companyproducts*), el *trigger* puede eliminar ese producto.

```sql
DELIMITER $$

CREATE TRIGGER after_delete_companyproducts_cleanup_product
AFTER DELETE ON companyproducts
FOR EACH ROW
BEGIN
    DECLARE v_uso_favoritos INT DEFAULT 0;
    DECLARE v_uso_calidad INT DEFAULT 0;
    DECLARE v_uso_empresas INT DEFAULT 0;

    SELECT COUNT(*) INTO v_uso_empresas
    FROM companyproducts
    WHERE product_id = OLD.product_id;

    SELECT COUNT(*) INTO v_uso_favoritos
    FROM details_favorites
    WHERE product_id = OLD.product_id;

    SELECT COUNT(*) INTO v_uso_calidad
    FROM quality_products
    WHERE product_id = OLD.product_id;

    IF v_uso_empresas = 0 AND v_uso_favoritos = 0 AND v_uso_calidad = 0 THEN
        DELETE FROM products
        WHERE id = OLD.product_id;
    END IF;
END$$

DELIMITER ;

-- Verificación:
DELETE FROM companyproducts 
WHERE company_id = 'COMP1' AND product_id = 1;

SELECT * 
FROM products 
WHERE id = 1;
```

### 6. Events:

01.6 Borrar productos sin actividad cada 6 meses.
🧠 Explicación: Algunos productos pueden haber sido creados pero nunca calificados, marcados como favoritos ni asociados a una empresa. Este evento eliminaría esos productos cada 6 meses.
🛠️ Se usaría un DELETE sobre products donde no existan registros en rates, favorites ni companyproducts.
📅 Frecuencia del evento: EVERY 6 MONTH.

```sql

```
02.6 Recalcular el promedio de calificaciones semanalmente
```sql

```
03.6 Actualizar precios según inflación mensual
```sql

```
04.6 Crear backups lógicos diariamente
```sql

```
05.6 Notificar sobre productos favoritos sin calificar
```sql

```
06.6 Revisar inconsistencias entre empresa y productos
```sql

```
07.6 Archivar membresías vencidas diariamente
```sql

```
08.6 Notificar beneficios nuevos a usuarios semanalmente
```sql

```
09.6 Calcular cantidad de favoritos por cliente mensualmente
```sql

```
10.6 Validar claves foráneas semanalmente
```sql

```
11.6 Eliminar calificaciones inválidas antiguas
```sql

```
12.6 Cambiar estado de encuestas inactivas automáticamente
```sql

```
13.6 Registrar auditorías de forma periódica
```sql

```
14.6 Notificar métricas de calidad a empresas
```sql

```
15.6 Recordar renovación de membresías
```sql

```
16.6 Reordenar estadísticas generales cada semana
```sql

```
17.6 Crear resúmenes temporales de uso por categoría
```sql

```
18.6 Actualizar beneficios caducados
```sql

```
19.6 Alertar productos sin evaluación anual
```sql

```
20.6 Actualizar precios con índice externo
```sql

```

### 7. Historias de Usuario con JOINs:

01.7 Ver productos con la empresa que los vende
```sql
SELECT em.name AS Empresa,
pro.name AS Producto,
cp.price AS Precio
FROM companyproducts cp
INNER JOIN companies em ON cp.company_id = em.id
INNER JOIN products pro ON cp.product_id = pro.id;
```
02.7 Mostrar productos favoritos con su empresa y categoría
```sql
SELECT pro.name AS Producto,
cat.description AS Categoría,
em.name AS empresa
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products pro ON df.product_id = pro.id
JOIN categories cat ON pro.category_id = cat.id
JOIN companyproducts cp ON cp.product_id = pro.id
JOIN companies em ON cp.company_id = em.id;
```
03.7 Ver empresas aunque no tengan productos
```sql
SELECT em.name AS Empresa
FROM companies em
LEFT JOIN companyproducts cp ON em.id = cp.company_id
GROUP BY em.name
HAVING COUNT(cp.product_id) = 0;
-- (Empty Set) Todas las empresas tienen productos.
```
04.7 Ver productos que fueron calificados (o no)
```sql
SELECT pro.name AS Producto,
r.rating AS Calificación
FROM quality_products r
RIGHT JOIN products pro ON pro.id = r.product_id;
```
05.7 Ver productos con promedio de calificación y empresa
```sql
SELECT em.name AS Empresa,
pro.name AS Producto,
AVG(qp.rating) AS PromedioCalificación
FROM products pro
JOIN companyproducts cp ON pro.id = cp.product_id
JOIN companies em ON cp.company_id = em.id
JOIN quality_products qp ON qp.product_id = pro.id AND qp.company_id = em.id
GROUP BY em.name, pro.name;
```
06.7 Ver clientes y sus calificaciones (si las tienen)
```sql
SELECT cl.name AS Cliente,
r.rating AS calificacion
FROM customers cl
LEFT JOIN rates r ON cl.id = r.customer_id;
```
07.7 Ver favoritos con la última calificación del cliente
```sql
SELECT c.name AS Cliente,
pro.name AS Producto,
comp.name AS Empresa,
cat.description AS Categoría,
MAX(r.daterating) AS FechaCalificación, r.rating AS Calificacion
FROM customers c
JOIN favorites fav ON c.id = fav.customer_id
JOIN details_favorites df ON fav.id = df.favorite_id
JOIN proucts pro ON df.prouct_id = pro.id
JOIN companyproucts cp ON pro.id = cp.prouct_id
JOIN companies comp ON cp.company_id = comp.id
JOIN categories cat ON pro.category_id = cat.id
LEFT JOIN rates r ON c.id = r.customer_id AND comp.id = r.company_id
GROUP BY fav.id, pro.id, comp.id, r.rating;
```
08.7 Ver beneficios incluidos en cada plan de membresía
```sql
SELECT m.name AS Membresía, m.description AS Descripción,
b.description AS Beneficio, b.detail AS Detalles
FROM membershipbenefits msb
JOIN memberships m ON m.id = msb.membership_id
JOIN benefits b ON b.id = msb.benefit_id;
```
09.7 Ver clientes con membresía activa y sus beneficios
```sql
SELECT cu.id AS IdCliente, cu.name AS Cliente,
m.name AS Membresia,
b.id AS IdBeneficio, b.description AS DescripcionBeneficio
FROM customers cu
JOIN membershipbenefits mb ON cu.audience_id = mb.audience_id
JOIN memberships m ON mb.membership_id = m.id
JOIN benefits b ON mb.benefit_id = b.id
WHERE cu.membership_active = TRUE;
```
10.7 Ver ciudades con cantidad de empresas
```sql
SELECT cm.name AS Ciudad,
COUNT(c.id) AS TotalEmpresas
FROM citiesormunicipalities cm
JOIN companies c ON cm.code = c.city_id
GROUP BY(cm.code);
```
11.7 Ver encuestas con calificaciones
```sql
SELECT po.name AS Encuesta, po.description AS Descripción,
r.rating AS Calificacion
FROM polls po
JOIN rates r ON po.id = r.poll_id;
```
12.7 Ver productos evaluados con datos del cliente
```sql
SELECT pro.name AS Producto,
c.name AS Cliente,
qp.daterating AS FechaCalificación, qp.rating AS Calificacion
FROM quality_products qp
JOIN products pro ON qp.product_id = pro.id
JOIN customers c ON qp.customer_id = c.id;
```
13.7 Ver productos con audiencia de la empresa
```sql
SELECT pro.name AS Producto,
em.name AS Empresa,
a.description AS DescripcioónAudiencia
FROM products pro
JOIN companyproducts cp ON pro.id = cp.product_id
JOIN companies em ON cp.company_id = em.id
JOIN audiences a ON em.audience_id = a.id;
```
14.7 Ver clientes con sus productos favoritos
```sql
SELECT c.name AS Cliente,
pro.name AS Producto
FROM customers c
JOIN favorites f ON f.customer_id = c.id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN products pro ON df.product_id = pro.id;
```
15.7 Ver planes, periodos, precios y beneficios
```sql
SELECT m.name AS Membresía,
pe.name AS Período,
b.description AS Beneficio, b.detail AS DetalleBeneficio
FROM membershipbenefits msb
JOIN memberships m ON m.id = msb.membership_id
JOIN benefits b ON b.id = msb.benefit_id
JOIN periods pe ON pe.id = msb.period_id;
```
16.7 Ver combinaciones empresa-producto-cliente calificados
```sql
SELECT c.name AS Cliente,
pro.name AS Producto,
em.name AS Empresa,
qp.rating AS calificacion
FROM quality_products qp
JOIN products pro ON qp.product_id = pro.id
JOIN companies em ON qp.company_id = em.id
JOIN customers c ON qp.customer_id = c.id;
```
17.7 Comparar favoritos con productos calificados
```sql
SELECT f.customer_id AS IdCliente,
pro.name AS Producto,
qp.rating AS Calificación,
qp.daterating AS Fecha
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products pro ON df.product_id = pro.id
JOIN quality_products qp ON qp.product_id = pro.id AND qp.customer_id = f.customer_id
WHERE f.customer_id = 1
ORDER BY Fecha DESC;
```
18.7 Ver productos ordenados por categoría
```sql
SELECT pro.name AS Producto,
cat.description AS Categoría
FROM products pro
JOIN categories cat ON cat.id = pro.category_id;
```
19.7 Ver beneficios por audiencia, incluso vacíos
```sql
SELECT a.description AS Audiencia,
b.description AS DescripciónBeneficio, b.detail AS Detalle
FROM audiences a
LEFT JOIN audiencebenefits ab ON a.id = ab.audience_id
LEFT JOIN benefits b ON ab.benefit_id = b.id;
```
20.7 Ver datos cruzados entre calificaciones, encuestas, productos y clientes
```sql
SELECT c.name AS Cliente,
pro.name AS Producto,
em.name AS Empresa,
po.name AS Encuesta,
qp.rating AS Calificación, qp.daterating AS Fecha
FROM quality_products qp
JOIN products pro ON qp.product_id = pro.id
JOIN polls po ON qp.poll_id = po.id
JOIN companies em ON qp.company_id = em.id
JOIN customers c ON qp.customer_id = c.id;
```

### 8. Historias de Usuario con Funciones Definidas por el Usuario (UDF):
01.8 Como analista, quiero una función que calcule el promedio ponderado de calidad de un producto basado en sus calificaciones y fecha de evaluación.
```sql

```
02.8 Como auditor, deseo una función que determine si un producto ha sido calificado recientemente (últimos 30 días).
```sql

```
03.8 Como desarrollador, quiero una función que reciba un *product_id* y devuelva el nombre completo de la empresa que lo vende.
```sql

```
04.8 Como operador, deseo una función que, dado un *customer_id*, me indique si el cliente tiene una membresía activa.
```sql

```
05.8 Como administrador, quiero una función que valide si una ciudad tiene más de X empresas registradas, recibiendo la ciudad y el número como parámetros.
```sql

```
06.8 Como gerente, deseo una función que, dado un rate_id, me devuelva una descripción textual de la calificación (por ejemplo, “Muy bueno”, “Regular”).
```sql

```
07.8 Como técnico, quiero una función que devuelva el estado de un producto en función de su evaluación (ej. “Aceptable”, “Crítico”).
```sql

```
08.8 Como cliente, deseo una función que indique si un producto está entre mis favoritos, recibiendo el *product_id* y mi *customer_id*.
```sql

```
09.8 Como gestor de beneficios, quiero una función que determine si un beneficio está asignado a una audiencia específica, retornando verdadero o falso.
```sql

```
10.8 Como auditor, deseo una función que reciba una fecha y determine si se encuentra dentro de un rango de membresía activa.
```sql

```
11.8 Como desarrollador, quiero una función que calcule el porcentaje de calificaciones positivas de un producto respecto al total.
```sql

```
12.8 Como supervisor, deseo una función que calcule la edad de una calificación, en días, desde la fecha actual.
```sql

```
13.8 Como operador, quiero una función que, dado un company_id, devuelva la cantidad de productos únicos asociados a esa empresa.
```sql

```
14.8 Como gerente, deseo una función que retorne el nivel de actividad de un cliente (frecuente, esporádico, inactivo), según su número de calificaciones.
```sql

```
15.8 Como administrador, quiero una función que calcule el precio promedio ponderado de un producto, tomando en cuenta su uso en favoritos.
```sql

```
16.8 Como técnico, deseo una función que me indique si un *benefit_id* está asignado a más de una audiencia o membresía (valor booleano).
```sql

```
17.8 Como cliente, quiero una función que, dada mi ciudad, retorne un índice de variedad basado en número de empresas y productos.
```sql

```
18.8 Como gestor de calidad, deseo una función que evalúe si un producto debe ser desactivado por tener baja calificación histórica.
```sql

```
19.8 Como desarrollador, quiero una función que calcule el índice de popularidad de un producto (combinando favoritos y ratings).
```sql

```
20.8 Como auditor, deseo una función que genere un código único basado en el nombre del producto y su fecha de creación.
```sql

```
