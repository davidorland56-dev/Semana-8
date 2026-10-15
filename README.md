sql_code = """
-- Proyecto: Base de datos para una plataforma musical
-- SGBD: MySQL 8.0+
-- Orden de ejecución: ejecutar el archivo completo desde MySQL Workbench.
-- Nota de diseño: el diagrama no muestra una clave primaria para GRUPO ni REGISTRADO.
-- Se utiliza nombre como PK de grupo (aparece subrayado en el diagrama) y se agrega
-- id_registrado como PK técnica para identificar de forma única cada registro.
-- Se interpretan las relaciones N:M como tablas asociativas.

CREATE DATABASE IF NOT EXISTS plataforma_musical
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;

USE plataforma_musical;

-- 1. Entidad GRUPO: se crea primero porque ALBUM depende de ella.
CREATE TABLE grupo (
    nombre VARCHAR(120) NOT NULL,
    fecha_cre DATE,
    componente VARCHAR(255),
    CONSTRAINT pk_grupo PRIMARY KEY (nombre)
) ENGINE=InnoDB;

-- 2. Entidad USUARIO.
CREATE TABLE usuario (
    id_usuario BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    fecha_alta DATE NOT NULL,
    url VARCHAR(500),
    correo VARCHAR(254) NOT NULL,
    CONSTRAINT pk_usuario PRIMARY KEY (id_usuario),
    CONSTRAINT uq_usuario_correo UNIQUE (correo)
) ENGINE=InnoDB;

-- 3. Entidad REGISTRADO.
CREATE TABLE registrado (
    id_registrado BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    cuota DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    fecha_fin DATE,
    CONSTRAINT pk_registrado PRIMARY KEY (id_registrado),
    CONSTRAINT chk_registrado_cuota CHECK (cuota >= 0)
) ENGINE=InnoDB;

-- 4. Entidad ALBUM. Se interpreta que un grupo puede tener varios álbumes.
CREATE TABLE album (
    id_album BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    fecha_pub DATE,
    num_canciones INT UNSIGNED NOT NULL DEFAULT 0,
    titulo VARCHAR(200) NOT NULL,
    nombre_grupo VARCHAR(120) NOT NULL,
    CONSTRAINT pk_album PRIMARY KEY (id_album),
    CONSTRAINT fk_album_grupo FOREIGN KEY (nombre_grupo)
        REFERENCES grupo(nombre)
        ON UPDATE CASCADE ON DELETE RESTRICT,
    CONSTRAINT chk_album_num_canciones CHECK (num_canciones >= 0)
) ENGINE=InnoDB;

-- 5. Entidad CANCION.
CREATE TABLE cancion (
    id_cancion BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    publica BOOLEAN NOT NULL DEFAULT TRUE,
    titulo VARCHAR(200) NOT NULL,
    CONSTRAINT pk_cancion PRIMARY KEY (id_cancion)
) ENGINE=InnoDB;

-- 6. Relación N:M CONTIENE entre ALBUM y CANCION.
CREATE TABLE album_cancion (
    id_album BIGINT UNSIGNED NOT NULL,
    id_cancion BIGINT UNSIGNED NOT NULL,
    CONSTRAINT pk_album_cancion PRIMARY KEY (id_album, id_cancion),
    CONSTRAINT fk_album_cancion_album FOREIGN KEY (id_album)
        REFERENCES album(id_album)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_album_cancion_cancion FOREIGN KEY (id_cancion)
        REFERENCES cancion(id_cancion)
        ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE=InnoDB;

-- 7. Relación N:M STREAM entre CANCION y USUARIO, con atributo fecha.
CREATE TABLE stream (
    id_cancion BIGINT UNSIGNED NOT NULL,
    id_usuario BIGINT UNSIGNED NOT NULL,
    fecha DATETIME NOT NULL,
    CONSTRAINT pk_stream PRIMARY KEY (id_cancion, id_usuario, fecha),
    CONSTRAINT fk_stream_cancion FOREIGN KEY (id_cancion)
        REFERENCES cancion(id_cancion)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_stream_usuario FOREIGN KEY (id_usuario)
        REFERENCES usuario(id_usuario)
        ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE=InnoDB;

-- 8. Relación N:M DESCARGA entre CANCION y REGISTRADO,
--    con atributos n_veces y fecha.
CREATE TABLE descarga (
    id_cancion BIGINT UNSIGNED NOT NULL,
    id_registrado BIGINT UNSIGNED NOT NULL,
    fecha DATETIME NOT NULL,
    n_veces INT UNSIGNED NOT NULL DEFAULT 1,
    CONSTRAINT pk_descarga PRIMARY KEY (id_cancion, id_registrado, fecha),
    CONSTRAINT fk_descarga_cancion FOREIGN KEY (id_cancion)
        REFERENCES cancion(id_cancion)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_descarga_registrado FOREIGN KEY (id_registrado)
        REFERENCES registrado(id_registrado)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT chk_descarga_n_veces CHECK (n_veces >= 0)
) ENGINE=InnoDB;

-- Consultas de verificación de la estructura creada:
SHOW TABLES;
DESCRIBE grupo;
DESCRIBE usuario;
DESCRIBE registrado;
DESCRIBE album;
DESCRIBE cancion;
DESCRIBE album_cancion;
DESCRIBE stream;
DESCRIBE descarga;
"""

print("Código SQL cargado correctamente en la variable 'sql_code'.")
