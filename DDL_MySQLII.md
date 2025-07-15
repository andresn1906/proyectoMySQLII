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
    code VARCHAR(6) PRIMARY KEY,
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

CREATE TABLE IF NOT EXISTS companies (
    id VARCHAR(20) PRIMARY KEY,
    type_id INT NOT NULL,
    CONSTRAINT FK_type_idcompa FOREIGN KEY (type_id) REFERENCES typesidentifications(id),
    name VARCHAR(80) NOT NULL,
    category_id INT NOT NULL,
    CONSTRAINT FK_category_idcompa FOREIGN KEY (category_id) REFERENCES categories(id),
    address VARCHAR(80) NOT NULL,
    city_id VARCHAR(6) NOT NULL,
    CONSTRAINT FK_city_idcompa FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
    audience_id INT NOT NULL,
    CONSTRAINT FK_audience_idcompa FOREIGN KEY (audience_id) REFERENCES audiences(id),
    cellphone VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(80) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS companyproducts (
    company_id VARCHAR(20) NOT NULL,
    CONSTRAINT FK_company_idcompany FOREIGN KEY (company_id) REFERENCES companies(id),
    product_id INT NOT NULL,
    CONSTRAINT FK_product_idcompany FOREIGN KEY (product_id) REFERENCES products(id),
    price DOUBLE NOT NULL,
    unitmeasure_id INT NOT NULL,
    CONSTRAINT FK_unitmeasure_idcompany FOREIGN KEY (unitmeasure_id) REFERENCES unitofmeasure(id),
    PRIMARY KEY (company_id, product_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(80) NOT NULL,
    city_id VARCHAR(6) NOT NULL,
    CONSTRAINT FK_city_idcustomers FOREIGN KEY (city_id) REFERENCES citiesormunicipalities(code),
    audience_id INT NOT NULL,
    CONSTRAINT FK_audience_idcustomers FOREIGN KEY (audience_id) REFERENCES audiences(id),
    cellphone VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(80) UNIQUE NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(60) UNIQUE NOT NULL,
    detail TEXT NOT NULL,
    price DOUBLE NOT NULL,
    category_id INT(11) NOT NULL,
    CONSTRAINT FK_category_idprod FOREIGN KEY (category_id) REFERENCES categories(id),
    image VARCHAR(80) NOT NULL 
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
    PRIMARY KEY (product_id, customer_id, poll_id),
    daterating DATETIME NOT NULL,
    rating DOUBLE NOT NULL
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
    CONSTRAINT FK_product_iddetails FOREIGN KEY (product_id) REFERENCES products(id)
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
```
