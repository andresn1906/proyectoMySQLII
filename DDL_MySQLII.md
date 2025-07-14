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
    CONSTRAINT FK_categorypoll_id FOREIGN KEY(categorypoll_id) REFERENCES categories_polls(id)
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
    id VARCHAR(6) PRIMARY KEY,
    name VARCHAR(60) UNIQUE NOT NULL,
    country_id VARCHAR(6) NOT NULL,
    CONSTRAINT FK_country_id FOREIGN KEY (country_id) REFERENCES countries(id),
    code3166 VARCHAR(10) UNIQUE NOT NULL,
    subdivision_id INT NOT NULL,
    CONSTRAINT FK_subdivision_id FOREIGN KEY (subdivision_id) REFERENCES subdivisioncategories(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS citiesormunicipalities (
    code VARCHAR(6) PRIMARY KEY,
    name VARCHAR(40) UNIQUE NOT NULL,
    statereg_id VARCHAR(6) NOT NULL,
    CONSTRAINT FK_statereg_id FOREIGN KEY (statereg_id) REFERENCES stateorregions(id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS typesidentifications (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(60) UNIQUE NOT NULL,
    sufix VARCHAR(5) UNIQUE NOT NULL,
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(60) UNIQUE NOT NULL,
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
    membership_id(11) INT NOT NULL,
    CONSTRAINT FK_membership_id FOREIGN KEY (membership_id) REFERENCES memberships(id),
    period_id INT(11) NOT NULL,
    CONSTRAINT FK_period_id FOREIGN KEY (period_id) REFERENCES periods(id),
    PRIMARY KEY (membership_id, period_id)
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS benefits (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(80) NOT NULL,
    detail TEXT NOT NULL
) ENGINE = INNODB;

CREATE TABLE IF NOT EXISTS benefits (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(80) NOT NULL,
    detail TEXT NOT NULL
) ENGINE = INNODB;
```

