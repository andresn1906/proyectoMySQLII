# ProyectoMySQLII Andres Suarez - Comandos DDL

### Base de datos
```sql
CREATE DATABASE proyectoCasn;
USE proyectoCasn;
```

### Tablas base de datos
```sql
CREATE TABLE IF NOT EXISTS categories_polls (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(80) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS polls (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(80) UNIQUE NOT NULL,
    description TEXT NOT NULL,
    isactive BOOLEAN NOT NULL,
    categorypoll_id INT NOT NULL,
    CONSTRAINT FK_categorypoll_idpolls FOREIGN KEY(categorypoll_id) REFERENCES categories_polls(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS poll_questions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    poll_id INT NOT NULL,
    CONSTRAINT FK_poll_id_pq FOREIGN KEY (poll_id) REFERENCES polls(id),
    question TEXT NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS countries (
    isocode VARCHAR(6) PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    alfaisotwo VARCHAR(2) UNIQUE NOT NULL,
    alfaisothree VARCHAR(4) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS subdivisioncategories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(40) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS stateorregions (
    code VARCHAR(6) PRIMARY KEY,
    name VARCHAR(60) UNIQUE NOT NULL,
    country_id VARCHAR(6) NOT NULL,
    CONSTRAINT FK_country_idstate FOREIGN KEY (country_id) REFERENCES countries(isocode),
    code3166 VARCHAR(10) UNIQUE NOT NULL,
    subdivision_id INT NOT NULL,
    CONSTRAINT FK_subdivision_idstate FOREIGN KEY (subdivision_id) REFERENCES subdivisioncategories(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS citiesormunicipalities (
    code VARCHAR(10) PRIMARY KEY,
    name VARCHAR(40) NOT NULL,
    statereg_id VARCHAR(6) NOT NULL,
    CONSTRAINT FK_statereg_idcities FOREIGN KEY (statereg_id) REFERENCES stateorregions(code)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS typesidentifications (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(60) UNIQUE NOT NULL,
    sufix VARCHAR(5) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(60) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS memberships (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(60) UNIQUE NOT NULL,
    description TEXT NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS periods (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(60) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS membershipperiods (
    membership_id INT(11) NOT NULL,
    CONSTRAINT FK_membership_idmembership FOREIGN KEY (membership_id) REFERENCES memberships(id),
    period_id INT(11) NOT NULL,
    CONSTRAINT FK_period_idmembership FOREIGN KEY (period_id) REFERENCES periods(id),
    fecha_fin DATE NOT NULL,
    pago_confirmado BOOLEAN DEFAULT FALSE,
    status VARCHAR(20) DEFAULT 'INACTIVA',
    PRIMARY KEY (membership_id, period_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS benefits (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(80) NOT NULL,
    detail TEXT NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS audiences (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(60) NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS unitofmeasure (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(60) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS audiencebenefits (
    audience_id INT NOT NULL,
    CONSTRAINT FK_audience_idaudience FOREIGN KEY (audience_id) REFERENCES audiences(id),
    benefit_id INT NOT NULL, 
    CONSTRAINT FK_benefit_idaudience FOREIGN KEY (benefit_id) REFERENCES benefits(id),
    PRIMARY KEY (audience_id, benefit_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS membershipbenefits (
    membership_id INT NOT NULL,
    CONSTRAINT FK_membership_idmembef FOREIGN KEY (membership_id) REFERENCES memberships(id),
    period_id INT NOT NULL,
    CONSTRAINT FK_period_idmembef FOREIGN KEY (period_id) REFERENCES periods(id),
    benefit_id INT NOT NULL,
    CONSTRAINT FK_benefit_idmembef FOREIGN KEY (benefit_id) REFERENCES benefits(id),
    audience_id INT NOT NULL,
    CONSTRAINT FK_audience_idmembef FOREIGN KEY (audience_id) REFERENCES audiences(id),
    PRIMARY KEY (membership_id, period_id, benefit_id, audience_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS company_types (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS companies (
    id VARCHAR(20) PRIMARY KEY,
    type_id INT NOT NULL,
    CONSTRAINT FK_type_idcompa FOREIGN KEY (type_id) REFERENCES typesidentifications(id),
    name VARCHAR(80) NOT NULL,
    category_id INT NOT NULL,
    CONSTRAINT FK_category_idcompa FOREIGN KEY (category_id) REFERENCES categories(id), 
    city_id VARCHAR(10) NOT NULL,
    CONSTRAINT FK_city_idcompa FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
    audience_id INT NOT NULL,
    CONSTRAINT FK_audience_idcompa FOREIGN KEY (audience_id) REFERENCES audiences(id),
    cellphone VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(80) UNIQUE NOT NULL,
    typecompany_id INT NOT NULL,
    CONSTRAINT FK_typecompany_idcompa FOREIGN KEY (type_id) REFERENCES company_types(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS resumen_calificaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    company_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_company_id_resumen FOREIGN KEY (company_id) REFERENCES companies(id),
    año INT NOT NULL,
    mes INT NOT NULL,
    promedio_calificacion DOUBLE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(60) UNIQUE NOT NULL,
    detail TEXT NOT NULL,
    price DOUBLE NOT NULL,
    category_id INT(11) NOT NULL,
    CONSTRAINT FK_category_idprod FOREIGN KEY (category_id) REFERENCES categories(id),
    image VARCHAR(80) NOT NULL,
    unitofmeasure_id INT NOT NULL,
    CONSTRAINT FK_unitmeasure_idprod FOREIGN KEY (unitofmeasure_id) REFERENCES unitofmeasure(id)
) ENGINE = INNODB;

CREATE TABLE poll_product (
    poll_id INT PRIMARY KEY,
    FOREIGN KEY (poll_id) REFERENCES polls(id),
    product_id INT NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS companyproducts (
    company_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_company_idcompany FOREIGN KEY (company_id) REFERENCES companies(id),
    product_id INT NOT NULL,
    CONSTRAINT FK_product_idcompany FOREIGN KEY (product_id) REFERENCES products(id),
    price DOUBLE NOT NULL,
    unitmeasure_id INT NOT NULL,
    CONSTRAINT FK_unitmeasure_idcompany FOREIGN KEY (unitmeasure_id) REFERENCES unitofmeasure(id),
    available_product BOOLEAN NOT NULL DEFAULT TRUE,
    PRIMARY KEY (company_id, product_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(80) NOT NULL,
    city_id VARCHAR(10) NOT NULL,
    CONSTRAINT FK_city_idcustomers FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
    audience_id INT NOT NULL,
    CONSTRAINT FK_audience_idcustomers FOREIGN KEY (audience_id) REFERENCES audiences(id),
    cellphone VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(80) UNIQUE NOT NULL,
    membership_active BOOLEAN NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS quality_products (
    product_id INT,
    CONSTRAINT FK_product_idquality FOREIGN KEY (product_id) REFERENCES products(id),
    customer_id INT NOT NULL,
    CONSTRAINT FK_customer_idquality FOREIGN KEY (customer_id) REFERENCES customers(id),
    poll_id INT NOT NULL,
    CONSTRAINT FK_poll_idquality FOREIGN KEY (poll_id) REFERENCES polls(id),
    company_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_company_idquality FOREIGN KEY (company_id) REFERENCES companies(id),
    daterating DATETIME NOT NULL,
    rating DOUBLE NOT NULL,
    PRIMARY KEY (product_id, customer_id, poll_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS favorites (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    CONSTRAINT FK_customer_idfav FOREIGN KEY (customer_id) REFERENCES customers(id),
    company_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_company_idfav FOREIGN KEY (company_id) REFERENCES companies(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS details_favorites (
    id INT PRIMARY KEY AUTO_INCREMENT,
    favorite_id INT NOT NULL,
    CONSTRAINT FK_favorite_iddetails FOREIGN KEY (favorite_id) REFERENCES favorites(id),
    product_id INT NOT NULL,
    CONSTRAINT FK_product_iddetails FOREIGN KEY (product_id) REFERENCES products(id),
    created_at DATE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS rates (
    customer_id INT NOT NULL,
    CONSTRAINT FK_customer_idrates FOREIGN KEY (customer_id) REFERENCES customers(id), 
    company_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_company_idrates FOREIGN KEY (company_id) REFERENCES companies(id),
    poll_id INT NOT NULL,
    CONSTRAINT FK_poll_idrates FOREIGN KEY (poll_id) REFERENCES polls(id),
    daterating DATETIME NOT NULL,
    rating DOUBLE NOT NULL,
    PRIMARY KEY (customer_id, company_id, poll_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS cambios_precios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    producto_id INT NOT NULL,
    CONSTRAINT FK_producto_historico FOREIGN KEY (producto_id) REFERENCES products(id),
    empresa_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_empresa_historico FOREIGN KEY (empresa_id) REFERENCES companies(id),
    precio_anterior DECIMAL(10,2) NOT NULL,
    precio_nuevo DECIMAL(10,2) NOT NULL,
    categoria_id INT NOT NULL,
    CONSTRAINT FK_categoria_historico FOREIGN KEY (categoria_id) REFERENCES categories(id),
    factor DECIMAL(10,2) NOT NULL,
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario VARCHAR(50)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS errores_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tipo_error VARCHAR(100) NOT NULL,
    descripcion TEXT NOT NULL,
    fecha_error TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    entidad_origen VARCHAR(50) NOT NULL,
    id_origen VARCHAR(50) NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS log_acciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tipo_accion VARCHAR(100),
    descripcion TEXT NOT NULL,
    customer_id INT NOT NULL,
    producto_id INT NOT NULL,
    fecha_accion DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS notificaciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    mensaje TEXT NOT NULL,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS bitacora (
    id INT AUTO_INCREMENT PRIMARY KEY,
    mensaje TEXT NOT NULL,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS log_cambios_encuestas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    poll_id INT NOT NULL,
    nuevo_estado VARCHAR(20) NOT NULL,
    fecha_cambio DATETIME DEFAULT CURRENT_TIMESTAMP,
    usuario VARCHAR(50) NOT NULL
) ENGINE = INNODB;

CREATE TABLE product_metrics (
    product_id INT PRIMARY KEY,
    avg_rating DOUBLE
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS products_backup LIKE products;
CREATE TABLE IF NOT EXISTS rates_backup LIKE rates;

CREATE TABLE IF NOT EXISTS user_reminders (
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    reminder_date DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id, product_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS favoritos_resumen (
    customer_id INT NOT NULL,
    total_favoritos INT NOT NULL,
    resumen_fecha DATE NOT NULL DEFAULT (CURRENT_DATE),
    PRIMARY KEY (customer_id, resumen_fecha)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS inconsistencias_fk (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tabla_afectada VARCHAR(100) NOT NULL,
    campo VARCHAR(100) NOT NULL,
    valor_erroneo VARCHAR(100) NOT NULL,
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS poll_responses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    poll_id INT NOT NULL,
    CONSTRAINT FK_poll_id_pr FOREIGN KEY (poll_id) REFERENCES polls(id),
    customer_id INT NOT NULL,
    CONSTRAINT FK_customer_id_pr FOREIGN KEY (customer_id) REFERENCES customers(id),
    question_id INT NOT NULL,
    CONSTRAINT FK_question_id_pr FOREIGN KEY (question_id) REFERENCES poll_questions(id),
    answer TEXT NOT NULL,
    response_date DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS auditorias_diarias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fecha DATE NOT NULL,
    total_productos INT NOT NULL,
    total_usuarios INT NOT NULL,
    total_empresas INT NOT NULL,
    registro_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS notificaciones_empresa (
    id INT AUTO_INCREMENT PRIMARY KEY,
    company_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_company_id_ne FOREIGN KEY (company_id) REFERENCES companies(id),
    product_id INT NOT NULL,
    CONSTRAINT FK_product_id_ne FOREIGN KEY (product_id) REFERENCES products(id),
    avg_rating DECIMAL(3,2) NOT NULL,
    fecha DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS estadisticas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fecha DATE NOT NULL DEFAULT (CURRENT_DATE),
    total_productos_activos INT NOT NULL,
    total_clientes INT NOT NULL,
    total_empresas INT NOT NULL,
    total_productos_disponibles INT NOT NULL,
    registro_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS resumen_categoria_uso (
    id INT AUTO_INCREMENT PRIMARY KEY,
    category_id INT NOT NULL,
    total_calificaciones INT NOT NULL,
    fecha_resumen DATE NOT NULL DEFAULT (CURRENT_DATE),
    FOREIGN KEY (category_id) REFERENCES categories(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS alertas_productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    alerta_fecha DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    mensaje VARCHAR(255) NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS inflacion_indice (
    id INT AUTO_INCREMENT PRIMARY KEY,
    indice DECIMAL(5,4) NOT NULL, 
    fecha_aplicacion DATE NOT NULL,
    comentario VARCHAR(100),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB;

CREATE TABLE customer_memberships (
    customer_id INT NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(id),
    membership_id INT NOT NULL,
    FOREIGN KEY (membership_id) REFERENCES memberships(id),
    PRIMARY KEY (customer_id, membership_id)
) ENGINE = INNODB;

SHOW TABLES;
```
