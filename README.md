# El-Sakila
El Sakila

## DDL

```mysql
DROP DATABASE IF EXISTS sakila;
CREATE DATABASE sakila;
USE sakila;

CREATE TABLE sakilacampus_film_text (
film_id SMALLINT AUTO_INCREMENT PRIMARY KEY,
title VARCHAR(255),
description TEXT);

CREATE TABLE sakilacampus_idioma (
id_idioma TINYINT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(20) NOT NULL,
ultima_actualizacion DATE);

CREATE TABLE sakilacampus_pais(
id_pais SMALLINT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(50) NOT NULL,
ultima_actualizacion TIMESTAMP);

CREATE TABLE sakilacampus_actor(
id_actor SMALLINT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(45) NOT NULL,
apellidos VARCHAR(45),
ultima_actualizacion TIMESTAMP);

CREATE TABLE sakilacampus_categoria(
id_categoria TINYINT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(25) NOT NULL,
ultima_actualizacion TIMESTAMP);

CREATE TABLE sakilacampus_ciudad(
id_ciudad SMALLINT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(50),
id_pais SMALLINT,
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_pais) REFERENCES sakilacampus_pais(id_pais));

CREATE TABLE sakilacampus_direccion(
id_direccion SMALLINT AUTO_INCREMENT PRIMARY KEY,
direccion VARCHAR(50),
direccion2 VARCHAR(50),
distrito VARCHAR(20),
id_ciudad SMALLINT,
codigo_postal VARCHAR(10),
telefono VARCHAR(20),
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_ciudad) REFERENCES sakilacampus_ciudad(id_ciudad));

/* no se pone foreign key a id_empleado_jefe debido a la jerarquia de las tablas (esta tabla debe crearse primero, pero al tener un id de la tabla empleado, tabla que se debe crear despues, no seria posible crear ambas tablas)*/

CREATE TABLE sakilacampus_almacen(
id_almacen TINYINT AUTO_INCREMENT PRIMARY KEY,
id_empleado_jefe TINYINT NOT NULL,
id_direccion SMALLINT,
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_direccion) REFERENCES sakilacampus_direccion(id_direccion));

-- PASSWORD es una palabra reservada en MYSQL

CREATE TABLE sakilacampus_empleado(
id_empleado TINYINT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(45),
apellido VARCHAR(45),
id_direccion SMALLINT,
imagen BLOB,
email VARCHAR(50),
id_almacen TINYINT,
activo TINYINT(1),
username VARCHAR(16),
password_empleado VARCHAR(40),
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_almacen) REFERENCES sakilacampus_almacen(id_almacen),
FOREIGN KEY (id_direccion) REFERENCES sakilacampus_direccion(id_direccion));

CREATE TABLE sakilacampus_cliente(
id_cliente SMALLINT AUTO_INCREMENT PRIMARY KEY,
id_almacen TINYINT,
nombre VARCHAR(45),
apellidos VARCHAR(45),
email VARCHAR(50),
id_direccion SMALLINT,
activo TINYINT,
fecha_creacion DATETIME,
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_almacen) REFERENCES sakilacampus_almacen(id_almacen),
FOREIGN KEY (id_direccion) REFERENCES sakilacampus_direccion(id_direccion));

CREATE TABLE sakilacampus_pelicula(
id_pelicula SMALLINT AUTO_INCREMENT PRIMARY KEY,
titulo VARCHAR(255),
description TEXT,
anyo_lanzamiento YEAR,
id_idioma TINYINT,
id_idioma_origina TINYINT,
duracion_alquiler TINYINT,
rental_rate DECIMAL(4,2),
duracion SMALLINT,
replacement_cost DECIMAL(5,2),
clasificacion ENUM('G','PG','PG-13','R','NC-17'),
caracteristicas_especiales SET('Trailers','Commentaries','Deleted Scenes','Behind The Scenes'),
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_idioma) REFERENCES sakilacampus_idioma(id_idioma),
FOREIGN KEY (id_idioma_origina) REFERENCES sakilacampus_idioma(id_idioma));

CREATE TABLE sakilacampus_pelicula_categoria(
id_pelicula SMALLINT,
id_categoria TINYINT,
FOREIGN KEY (id_pelicula) REFERENCES sakilacampus_pelicula(id_pelicula),
FOREIGN KEY (id_categoria) REFERENCES sakilacampus_categoria(id_categoria));

CREATE TABLE sakilacampus_pelicula_actor(
id_actor SMALLINT,
id_pelicula SMALLINT,
FOREIGN KEY (id_pelicula) REFERENCES sakilacampus_pelicula(id_pelicula),
FOREIGN KEY (id_actor) REFERENCES sakilacampus_actor(id_actor));

CREATE TABLE sakilacampus_inventario(
id_inventario MEDIUMINT AUTO_INCREMENT PRIMARY KEY,
id_pelicula SMALLINT,
id_almacen TINYINT,
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_pelicula) REFERENCES sakilacampus_pelicula(id_pelicula),
FOREIGN KEY (id_almacen) REFERENCES sakilacampus_almacen(id_almacen));

CREATE TABLE sakilacampus_alquiler(
id_alquiler INT AUTO_INCREMENT PRIMARY KEY,
fecha_alquiler DATETIME,
id_inventario MEDIUMINT,
id_cliente SMALLINT,
fecha_devolucion DATETIME,
id_empleado TINYINT,
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_inventario) REFERENCES sakilacampus_inventario(id_inventario),
FOREIGN KEY (id_cliente) REFERENCES sakilacampus_cliente(id_cliente),
FOREIGN KEY (id_empleado) REFERENCES sakilacampus_empleado(id_empleado));

CREATE TABLE sakilacampus_pago(
id_pago SMALLINT AUTO_INCREMENT PRIMARY KEY,
id_cliente SMALLINT,
id_empleado TINYINT,
id_alquiler INT,
total DECIMAL(5,2),
fecha_pago DATETIME,
ultima_actualizacion TIMESTAMP,
FOREIGN KEY (id_cliente) REFERENCES sakilacampus_cliente(id_cliente),
FOREIGN KEY (id_alquiler) REFERENCES sakilacampus_alquiler(id_alquiler),
FOREIGN KEY (id_empleado) REFERENCES sakilacampus_empleado(id_empleado));
```

## Consultas SQL

### 1. Encuentra el cliente que ha realizado la mayor cantidad de alquileres en los últimos 6 meses.

```mysql
SELECT sc.id_cliente, sc.nombre, COUNT(sa.id_alquiler) AS Numero_Alquileres
FROM sakilacampus_cliente sc
INNER JOIN sakilacampus_alquiler sa ON sc.id_cliente = sa.id_cliente
GROUP BY sc.id_cliente
ORDER BY Numero_Alquileres DESC;
```



### 2. Lista las cinco películas más alquiladas durante el último año.

```mysql
SELECT sp.id_pelicula, sp.titulo, COUNT(sa.id_alquiler) AS Numero_Alquileres
FROM sakilacampus_pelicula sp
INNER JOIN sakilacampus_inventario si ON sp.id_pelicula = si.id_pelicula
INNER JOIN sakilacampus_alquiler sa ON si.id_inventario = sa.id_inventario
GROUP BY  sp.id_pelicula
ORDER BY Numero_Alquileres DESC
LIMIT 5;
```



### 3. Obtén el total de ingresos y la cantidad de alquileres realizados por cada categoría de película.

```mysql
SELECT sc.nombre AS categoria, SUM(sp.total) AS total_ingresos, COUNT(sa.id_alquiler) AS cantidad_alquileres
FROM sakilacampus_pago sp
INNER JOIN sakilacampus_alquiler sa ON sp.id_alquiler = sa.id_alquiler
INNER JOIN sakilacampus_inventario si ON sa.id_inventario = si.id_inventario
INNER JOIN sakilacampus_pelicula sp ON si.id_pelicula = sp.id_pelicula
INNER JOIN sakilacampus_pelicula_categoria spc ON sp.id_pelicula = spc.id_pelicula
INNER JOIN sakilacampus_categoria sc ON spc.id_categoria = sc.id_categoria
GROUP BY sc.id_categoria;
```



### 4. Calcula el número total de clientes que han realizado alquileres por cada idioma disponible en un mes específico.

```mysql
SELECT si.nombre AS idioma, COUNT(DISTINCT sa.id_cliente) AS total_clientes
FROM sakilacampus_alquiler sa
INNER JOIN sakilacampus_inventario si ON sa.id_inventario = si.id_inventario
INNER JOIN sakilacampus_pelicula sp ON si.id_pelicula = sp.id_pelicula
INNER JOIN sakilacampus_idioma si ON sp.id_idioma = si.id_idioma
WHERE MONTH(sa.fecha_alquiler) = 4  -- cambiar por mes especifico
AND YEAR(sa.fecha_alquiler) = 2025  -- Cambiar por anyo requerido
GROUP BY sp.id_idioma;
```



### 5. Encuentra a los clientes que han alquilado todas las películas de una misma categoría.

```mysql
SELECT c.nombre AS cliente
FROM sakilacampus_cliente c
WHERE NOT EXISTS (
    SELECT 1
    FROM sakilacampus_pelicula_categoria pc
    WHERE NOT EXISTS (
        SELECT 1
        FROM sakilacampus_alquiler a
        INNER JOIN sakilacampus_inventario i ON a.id_inventario = i.id_inventario
        WHERE a.id_cliente = c.id_cliente
        AND i.id_pelicula = pc.id_pelicula))
GROUP BY c.id_cliente;
```



### 6. Lista las tres ciudades con más clientes activos en el último trimestre.

```mysql
SELECT ci.nombre AS ciudad, COUNT(DISTINCT cl.id_cliente) AS clientes_activos
FROM sakilacampus_cliente cl
INNER JOIN sakilacampus_direccion d ON cl.id_direccion = d.id_direccion
INNER JOIN sakilacampus_ciudad ci ON d.id_ciudad = ci.id_ciudad
WHERE cl.activo = 1
AND sa.fecha_alquiler >= DATE_SUB(CURDATE(), INTERVAL 3 MONTH)
GROUP BY ci.id_ciudad
ORDER BY clientes_activos DESC
LIMIT 3;
```



### 7. Muestra las cinco categorías con menos alquileres registrados en el último año.

```mysql
SELECT sc.nombre AS categoria, COUNT(sa.id_alquiler) AS cantidad_alquileres
FROM sakilacampus_alquiler sa
INNER JOIN sakilacampus_inventario si ON sa.id_inventario = si.id_inventario
INNER JOIN sakilacampus_pelicula sp ON si.id_pelicula = sp.id_pelicula
INNER JOIN sakilacampus_pelicula_categoria spc ON sp.id_pelicula = spc.id_pelicula
INNER JOIN sakilacampus_categoria sc ON spc.id_categoria = sc.id_categoria
WHERE sa.fecha_alquiler >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY sc.id_categoria
ORDER BY cantidad_alquileres ASC
LIMIT 5;
```



### 8. Calcula el promedio de días que un cliente tarda en devolver las películas alquiladas.

```mysql
SELECT
    AVG(DATEDIFF(sa.fecha_devolucion, sa.fecha_alquiler)) AS promedio_dias_devolucion
FROM sakilacampus_alquiler sa
WHERE sa.fecha_devolucion IS NOT NULL;
```



### 9. Encuentra los cinco empleados que gestionaron más alquileres en la categoría de Acción.

### 10. Genera un informe de los clientes con alquileres más recurrentes.

### 11. Calcula el costo promedio de alquiler por idioma de las películas.

### 12. Lista las cinco películas con mayor duración alquiladas en el último año.

### 13. Muestra los clientes que más alquilaron películas de Comedia.

### 14. Encuentra la cantidad total de días alquilados por cada cliente en el último mes.

### 15. Muestra el número de alquileres diarios en cada almacén durante el último trimestre.

### 16. Calcula los ingresos totales generados por cada almacén en el último semestre.

### 17. Encuentra el cliente que ha realizado el alquiler más caro en el último año.

### 18. Lista las cinco categorías con más ingresos generados durante los últimos tres meses.

### 19. Obtén la cantidad de películas alquiladas por cada idioma en el último mes.

### 20. Lista los clientes que no han realizado ningún alquiler en el último año.

## Funciones SQL

### 1. TotalIngresosCliente(ClienteID, Año): Calcula los ingresos generados por un cliente en un año específico.

```mysql
DELIMITER //

CREATE FUNCTION totalingresoscliente (cliente_id SMALLINT, anyo_especifico YEAR)
RETURNS DECIMAL(10,2)
BEGIN
    DECLARE total_ingresos DECIMAL(10,2);

    SELECT SUM(sp.total) INTO total_ingresos
    FROM sakilacampus_pago sp
    WHERE sp.id_cliente = cliente_id 
    AND YEAR(sp.fecha_pago) = anyo_especifico;

    RETURN total_ingresos;
END//

DELIMITER ;
```



### 2. PromedioDuracionAlquiler(PeliculaID): Retorna la duración promedio de alquiler de una película específica.

```mysql
DELIMITER //

CREATE FUNCTION PromedioDuracionAlquiler(PeliculaID SMALLINT)
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
    DECLARE promedio DECIMAL(5,2);
    SELECT AVG(sa.duracion) INTO promedio
    FROM sakilacampus_alquiler sa
    INNER JOIN sakilacampus_inventario si ON sa.id_inventario = si.id_inventario
    WHERE si.id_pelicula = PeliculaID;
    RETURN promedio;
END //

DELIMITER ;
```



### 3. IngresosPorCategoria(CategoriaID): Calcula los ingresos totales generados por una categoría específica de películas.

### 4. DescuentoFrecuenciaCliente(ClienteID): Calcula un descuento basado en la frecuencia de alquiler del cliente.

### 5. EsClienteVIP(ClienteID): Verifica si un cliente es "VIP" basándose en la cantidad de alquileres realizados y los ingresos generados.

## Triggers

### 1. ActualizarTotalAlquileresEmpleado: Al registrar un alquiler, actualiza el total de alquileres gestionados por el empleado correspondiente.

no hay un dato de total de alquileres en empleado

### 2. AuditarActualizacionCliente: Cada vez que se modifica un cliente, registra el cambio en una tabla de auditoría.

no hay una tabla de auditorias para modificar un cliente

### 3. RegistrarHistorialDeCosto: Guarda el historial de cambios en los costos de alquiler de las películas.

no hay una tabla de logs para el historial de cambios en los costos de alquiler de las peliculas

### 4. NotificarEliminacionAlquiler: Registra una notificación cuando se elimina un registro de alquiler.

no hay una tabla en la que se registren las noticias de alquiler

### 5. RestringirAlquilerConSaldoPendiente: Evita que un cliente con saldo pendiente pueda realizar nuevos alquileres.

no hay tabla o campos necesarios para validar el saldo pendiente de un cliente
