# Recopilatorio de examenes de laboratorio individual - IISSI1

> Generado a partir del repositorio [examenes-lab-individual](https://github.com/IISSI1-IS-2025/examenes-lab-individual) (rama `main` excluida por ser la plantilla base). Cada seccion se corresponde con una rama/examen del repositorio; bajo cada apartado del enunciado se incluye el codigo de la solucion correspondiente cuando esta disponible en el repositorio.

## Indice

- [Modelo A](#modelo-a)
- [Modelo B](#modelo-b)
- [Modelo C](#modelo-c)
- [Segunda Convocatoria (Julio)](#segunda-convocatoria-julio)
- [Tercera Convocatoria (Octubre)](#tercera-convocatoria-octubre)

---

# Modelo A

*(rama: `modeloA`)*

**Si usted entrega sin haber sido verificada su identidad no podrá ser evaluado.**

## Tienda Online

Partiendo de la `tiendaOnline` vista durante los laboratorios descrita en el modelado conceptual siguiente:

![tiendaOnlineModeladoConceptual](https://github.com/user-attachments/assets/92eb4ba8-1ed8-488b-bb5b-448c0836fee6)

Las tablas y datos de prueba iniciales se encuentran en los ficheros `0.creacionTablas.sql` y `0.poblarBd.sql`.

Realice los siguientes ejercicios:

### 1. Creación de tabla. (1,5 puntos)

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.

Necesitamos conocer la garantía de nuestros productos. Para ello se propone la creación de una nueva tabla llamada `Garantias`. Cada producto tendrá como máximo una garantía (no todos los productos tienen garantía), y cada garantía estará relacionada con un producto.

Para cada garantía necesitamos conocer la fecha de inicio de la garantía, la fecha de fin de la garantía, si tiene garantía extendida o no.

Asegure que la fecha de fin de la garantía es posterior a la fecha de inicio.


**Solucion (`1.propuestaSolucionCreacionTabla.sql`):**

```sql
DROP TABLE IF EXISTS Garantias;
CREATE TABLE Garantias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    productoId INT NOT NULL,
    fechaInicio DATE NOT NULL,
    fechaFin DATE NOT NULL,
    esExtendida BOOLEAN NOT NULL,
    FOREIGN KEY (productoId) REFERENCES Productos(id),
    CHECK (fechaFin > fechaInicio),
    UNIQUE (productoId)
);

-- Ejemplo de inserción de datos
INSERT INTO Garantias (productoId, fechaInicio, fechaFin, esExtendida)
VALUES (1, '2024-01-01', '2026-01-01', TRUE),
       (2, '2024-03-15', '2025-03-15', FALSE),
       (3, '2024-05-10', '2027-05-10', TRUE);
```

### 2. Consultas SQL (DQL). (3 puntos)

Incluya su solución en el fichero `2.solucionConsultas.sql`.

#### 2.1. Devuelva el nombre del producto, nombre del tipo de producto, y precio unitario al que se vendieron los productos digitales (1 punto)

#### 2.2. Consulta que devuelva el nombre del empleado, el número de pedidos de más de 500 euros gestionados en este año y el importe total de cada uno de ellos, ordenados de mayor a menor importe gestionado. Los empleados que no hayan gestionado ningún pedido, también deben aparecer. (2 puntos)


**Solucion (`2.propuestaSolucionConsultas.sql`):**

```sql
-- 2.1
SELECT
    p.nombre AS nombre_producto,
    tp.nombre AS nombre_tipo_producto,
    lp.precio AS precio_unitario
FROM
    LineasPedido lp
JOIN
    Productos p ON lp.productoId = p.id
JOIN
    TiposProducto tp ON p.tipoProductoId = tp.id
WHERE
    tp.nombre = 'Digitales';


-- 2.2 Solución admisible con condición en LEFT JOIN
SELECT
    u.nombre AS nombre_empleado,
    COUNT(DISTINCT p.id) AS numero_pedidos_mas_de_500,
    SUM(lp.precio * lp.unidades) AS importe_total
FROM
    Empleados e
LEFT JOIN
    Usuarios u ON e.usuarioId = u.id
LEFT JOIN
    Pedidos p ON e.id = p.empleadoId AND YEAR(p.fechaRealizacion) = YEAR(CURDATE()) -- Filtrar pedidos del año actual
LEFT JOIN
    LineasPedido lp ON p.id = lp.pedidoId
GROUP BY
    u.id, u.nombre
HAVING
    SUM(lp.precio * lp.unidades) > 500 OR numero_pedidos_mas_de_500 = 0 -- Incluye empleados sin pedidos
ORDER BY
    importe_total DESC;


-- 2.2 Solución admisible con condición en HAVING (no aparecen empleados sin pedidos)
SELECT
    u.nombre AS nombre_empleado,
    COUNT(DISTINCT p.id) AS numero_pedidos_mas_de_500,
    SUM(lp.precio * lp.unidades) AS importe_total
FROM
    Empleados e
LEFT JOIN
    Usuarios u ON e.usuarioId = u.id
LEFT JOIN
    Pedidos p ON e.id = p.empleadoId
LEFT JOIN
    LineasPedido lp ON p.id = lp.pedidoId
WHERE
    YEAR(p.fechaRealizacion) = YEAR(CURDATE()) -- Filtrar pedidos realizados en el año actual
GROUP BY
    u.id, u.nombre
HAVING
    SUM(lp.precio * lp.unidades) > 500 OR numero_pedidos_mas_de_500 = 0 -- Incluye empleados con pedidos > 500
ORDER BY
    importe_total DESC;



-- 2.2 Solución exahustiva (considera casos raros)
SELECT
    u.nombre AS nombre_empleado,
    COUNT(DISTINCT CASE
                      WHEN YEAR(p.fechaRealizacion) = YEAR(CURDATE())
                           AND (SELECT SUM(lp2.precio * lp2.unidades)
                                FROM LineasPedido lp2
                                WHERE lp2.pedidoId = p.id) > 500
                      THEN p.id
                  END) AS numero_pedidos_mas_de_500,
    SUM(CASE
            WHEN YEAR(p.fechaRealizacion) = YEAR(CURDATE())
                 AND (SELECT SUM(lp2.precio * lp2.unidades)
                      FROM LineasPedido lp2
                      WHERE lp2.pedidoId = p.id) > 500
            THEN lp.precio * lp.unidades
            ELSE 0
        END) AS importe_total
FROM
    Empleados e
LEFT JOIN
    Usuarios u ON e.usuarioId = u.id
LEFT JOIN
    Pedidos p ON e.id = p.empleadoId
LEFT JOIN
    LineasPedido lp ON p.id = lp.pedidoId
GROUP BY
    u.id, u.nombre
HAVING
    numero_pedidos_mas_de_500 > 0
    OR numero_pedidos_mas_de_500 = 0 -- Incluye empleados sin pedidos
ORDER BY
    importe_total DESC;
```

### 3. Procedimiento. Actualizar precio de un producto y líneas de pedido no enviadas. (3,5 puntos)

Incluya su solución en el fichero `3.solucionProcedimiento.sql`.

Cree un procedimiento que permita actualizar el precio de un producto dado y que modifique los precios de las líneas de pedido asociadas al producto dado solo en aquellos pedidos que aún no hayan sido enviados. (1,5 puntos)

Asegure que el nuevo precio no sea un 50% menor que el precio actual y lance excepción si se da el caso con el siguiente mensaje: (1 punto)

`No se permite rebajar el precio más del 50%`.

Garantice que o bien se realizan todas las operaciones o bien no se realice ninguna. (1 punto)


**Solucion (`3.propuestaSolucionProcedimiento.sql`):**

```sql
DELIMITER //

CREATE PROCEDURE actualizar_precio_producto_lineas(
    IN p_productoId INT,
    IN p_nuevoPrecio DECIMAL(10,2)
)
BEGIN
    DECLARE v_precioActual DECIMAL(10,2);

    -- Iniciar transacción
    START TRANSACTION;

    -- Obtener el precio actual del producto
    SELECT precio INTO v_precioActual
    FROM Productos
    WHERE id = p_productoId;

    -- Si el producto no existe, lanzar un error
    IF v_precioActual IS NULL THEN
        ROLLBACK; -- Revertir cambios antes de salir
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Producto no encontrado';
    END IF;

    -- Verificar que el nuevo precio no sea un 50% menor
    IF p_nuevoPrecio < v_precioActual * 0.5 THEN
        ROLLBACK; -- Revertir cambios antes de salir
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El nuevo precio no puede ser un 50% menor que el actual';
    END IF;

    -- Actualizar el precio del producto
    UPDATE Productos
    SET precio = p_nuevoPrecio
    WHERE id = p_productoId;

    -- Actualizar las líneas de pedido de pedidos no enviados
    UPDATE LineasPedido
    SET precio = p_nuevoPrecio
    WHERE productoId = p_productoId
      AND pedidoId IN (
          SELECT id
          FROM Pedidos
          WHERE fechaEnvio IS NULL
      );

    -- Confirmar transacción
    COMMIT;
END//

DELIMITER ;
```

### 4. Trigger. (2 puntos)

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `t_asegurar_mismo_tipo_producto_en_pedidos` que impida que, a partir de ahora, un mismo pedido incluya productos físicos y digitales.


**Solucion (`4.propuestaSolucionTrigger.sql`):**

```sql
DELIMITER //

CREATE TRIGGER t_asegurar_mismo_tipo_producto_en_pedidos
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN
    DECLARE v_idTipoProductoNuevo INT;
    DECLARE v_existeOtroTipoProductoEnPedido BOOLEAN;

    -- Obtener el tipo del producto que se intenta insertar
    SELECT Productos.tipoProductoId
    INTO v_idTipoProductoNuevo
    FROM Productos p
    WHERE p.id = NEW.productoId;

    -- Verificar si el pedido ya tiene líneas y qué tipo de producto contiene
    SELECT EXISTS (
        SELECT *
        FROM LineasPedido lp
        JOIN Productos p ON lp.productoId = p.id
        WHERE lp.pedidoId = NEW.pedidoId
          AND p.tipoProductoId <> v_idTipoProductoNuevo
    ) INTO v_existeOtroTipoProductoEnPedido;

    -- Si el pedido ya tiene líneas, comparar los tipos y lanzar excepción si son diferentes
    IF v_existeOtroTipoProductoEnPedido THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'El pedido no puede incluir productos de tipos diferentes (físicos y digitales).';
    END IF;
END//

DELIMITER ;
```


---

# Modelo B

*(rama: `modeloB`)*

**Si usted entrega sin haber sido verificada su identidad no podrá ser evaluado.**

## Tienda Online

Partiendo de la `tiendaOnline` vista durante los laboratorios descrita en el modelado conceptual siguiente:

![tiendaOnlineModeladoConceptual](https://github.com/user-attachments/assets/92eb4ba8-1ed8-488b-bb5b-448c0836fee6)

Las tablas y datos de prueba iniciales se encuentran en los ficheros `0.creacionTablas.sql` y `0.poblarBd.sql`.

Realice los siguientes ejercicios:

### 1. Creación de tabla. (1,5 puntos)

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.


Necesitamos conocer los pagos que se realicen sobre los pedidos. Para ello se propone la creación de una nueva tabla llamada `Pagos`. Cada pedido podrá tener asociado varios pagos y cada pago solo corresponde con un pedido en concreto.

Para cada pago necesitamos conocer la fecha de pago, la cantidad pagada (que no puede ser negativa) y si el pago ha sido revisado o no (por defecto no estará revisado).


**Solucion (`1.propuestaSolucionCreacionTabla.sql`):**

```sql
CREATE TABLE Pagos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedidoId INT NOT NULL,
    fechaPago DATETIME NOT NULL,
    cantidadPagada DECIMAL(10, 2) NOT NULL CHECK (cantidadPagada >= 0),
    revisado BOOLEAN NOT NULL DEFAULT FALSE,
    FOREIGN KEY (pedidoId) REFERENCES Pedidos(id)
        ON DELETE CASCADE 
        ON UPDATE CASCADE
);
```

### 2. Consultas SQL (DQL). 3 puntos

Incluya su solución en el fichero `2.solucionConsultas.sql`.

#### 2.1. Devuelva el nombre del del empleado, la fecha de realización del pedido y el nombre del cliente de todos los pedidos realizados este mes. (1 puntos)

#### 2.2. Devuelva el nombre, las unidades totales pedidas y el importe total gastado de aquellos clientes que han realizado más de 5 pedidos en el último año. (2 puntos)


**Solucion (`2.propuestaSolucionConsultas.sql`):**

```sql
-- 2.1. Devuelva el nombre del empleado, la fecha de realización del pedido y el nombre del cliente de todos los pedidos realizados este mes.
SELECT 
    u.nombre AS nombre_empleado, 
    p.fechaRealizacion, 
    c.nombre AS nombre_cliente
FROM Pedidos p
LEFT JOIN Empleados e ON p.empleadoId = e.id
LEFT JOIN Usuarios u ON e.usuarioId = u.id
JOIN Clientes cl ON p.clienteId = cl.id
JOIN Usuarios c ON cl.usuarioId = c.id
WHERE MONTH(p.fechaRealizacion) = MONTH(CURDATE())
  AND YEAR(p.fechaRealizacion) = YEAR(CURDATE());


-- 2.2. Devuelva el nombre, las unidades totales pedidas y el importe total gastado de aquellos clientes que han realizado más de 5 pedidos en el último año.
SELECT 
    u.nombre AS nombre_cliente, 
    SUM(lp.unidades) AS unidades_totales,
    SUM(lp.unidades * lp.precio) AS importe_total
FROM Pedidos p
JOIN Clientes cl ON p.clienteId = cl.id
JOIN Usuarios u ON cl.usuarioId = u.id
JOIN LineasPedido lp ON p.id = lp.pedidoId
WHERE p.fechaRealizacion >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY u.id
HAVING COUNT(p.id) > 5;
```

### 3. Procedimiento. 3,5 puntos

Incluya su solución en el fichero `3.solucionProcedimiento.sql`.

Cree un procedimiento que permita crear un nuevo producto con posibilidad de que sea para regalo. Si el producto está destinado a regalo se creará un pedido con ese producto y costes 0€ para el cliente más antiguo. (1,5 puntos)

Asegure que el precio del producto para regalo no debe superar los 50 euros y lance excepción si se da el caso con el siguiente mensaje: (1 punto)

`No se permite crear un producto para regalo de más de 50€`.

Garantice que o bien se realizan todas las operaciones o bien no se realice ninguna. (1 punto)


**Solucion (`3.propuestaSolucionProcedimiento.sql`):**

```sql
DELIMITER //

CREATE PROCEDURE insertar_producto_y_regalos(
    IN p_nombre VARCHAR(255),
    IN p_descripcion VARCHAR(255),
    IN p_precio DECIMAL(10,2),
    IN p_tipoProductoId INT,
    IN p_esParaRegalo BOOLEAN
)
BEGIN
    -- Declaramos variables para datos del cliente
    DECLARE clienteMasAntiguoId INT;
    DECLARE clienteMasAntiguoDireccion VARCHAR(255);
    DECLARE clienteMasAntiguoCodigoPostal VARCHAR(10);
    
    -- Declaramos variable para guardar el ID del producto recién creado
    DECLARE v_nuevoProductoId INT;

    -- Verificar si el producto es para regalo y su precio
    IF p_esParaRegalo AND p_precio > 50 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se permite crear un producto para regalo de más de 50€';
    END IF;

    -- Iniciar transacción
    START TRANSACTION;

    -- 1. Insertar el producto
    INSERT INTO Productos (nombre, descripcion, precio, tipoProductoId)
    VALUES (p_nombre, p_descripcion, p_precio, p_tipoProductoId);

    -- 2. IMPORTANTE: Guardamos el ID del producto antes de hacer cualquier otro INSERT
    SET v_nuevoProductoId = LAST_INSERT_ID();

    -- Si es para regalo, crear un pedido para el cliente más antiguo
    IF p_esParaRegalo THEN
        -- Obtener cliente más antiguo
        SELECT id, direccionEnvio, codigoPostal INTO clienteMasAntiguoId, clienteMasAntiguoDireccion, clienteMasAntiguoCodigoPostal
        FROM Clientes
        ORDER BY id ASC
        LIMIT 1;

        -- 3. Crear pedido
        -- Al hacer este INSERT, LAST_INSERT_ID() cambiará al ID del nuevo pedido
        INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId, comentarios)
        VALUES (CURDATE(), clienteMasAntiguoDireccion, clienteMasAntiguoId, 'Pedido de regalo');

        -- 4. Agregar línea de pedido
        -- LAST_INSERT_ID() = ID del Pedido (recién creado arriba)
        -- v_nuevoProductoId = ID del Producto (guardado previamente)
        INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio)
        VALUES (LAST_INSERT_ID(), v_nuevoProductoId, 1, 0);
    END IF;

    -- Confirmar transacción
    COMMIT;
END //

DELIMITER ;
```

### 4. Trigger. 2 puntos

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `t_limitar_importe_pedidos_de_menores` que impida que, a partir de ahora, los pedidos realizados por menores superen los 500€.


**Solucion (`4.propuestaSolucionTrigger.sql`):**

```sql
DELIMITER //

CREATE TRIGGER t_limitar_importe_pedidos_de_menores
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN
    DECLARE clienteEdad INT;
    DECLARE sumaTotal DECIMAL(10, 2);

    -- Obtener la edad del cliente asociado al pedido
    SELECT TIMESTAMPDIFF(YEAR, c.fechaNacimiento, CURDATE()) INTO clienteEdad
    FROM Pedidos p
    JOIN Clientes c ON p.clienteId = c.id
    WHERE p.id = NEW.pedidoId;

    -- Calcular el importe total del pedido actual más la nueva línea
    SELECT SUM(lp.unidades * lp.precio) + (NEW.unidades * NEW.precio) INTO sumaTotal
    FROM LineasPedido lp
    WHERE lp.pedidoId = NEW.pedidoId;

    -- Verificar la restricción para menores
    IF clienteEdad < 18 AND sumaTotal > 500 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Los pedidos realizados por menores no pueden superar los 500€';
    END IF;
END //

DELIMITER ;
```


---

# Modelo C

*(rama: `modeloC`)*

> **Nota:** el repositorio no incluye soluciones para este examen (los ficheros `solucionX.sql` estan vacios). Las soluciones que aparecen a continuacion son una propuesta generada y verificada manualmente contra un MariaDB real (creacion de tabla, consultas, procedimiento y trigger probados con datos de prueba), no un fichero oficial del profesor.

**Si usted entrega sin haber sido verificada su identidad no podrá ser evaluado.**

## Tienda Online

Partiendo de la `tiendaOnline` vista durante los laboratorios descrita en el modelado conceptual siguiente:

![tiendaOnlineModeladoConceptual](https://github.com/user-attachments/assets/92eb4ba8-1ed8-488b-bb5b-448c0836fee6)

Las tablas y datos de prueba iniciales se encuentran en los ficheros `0.creacionTablas.sql` y `0.poblarBd.sql`.

Realice los siguientes ejercicios:

### 1. Creación de tabla. (1,5 puntos)

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.

Necesitamos conocer la opinión de nuestros clientes sobre nuestros productos. Para ello se propone la creación de una nueva tabla llamada `Valoraciones`. Cada valoración versará sobre un producto y será realizada por un solo cliente. Cada producto podrá ser valorado por muchos clientes. Cada cliente podrá realizar muchas valoraciones. Un cliente no puede valorar más de una vez un mismo producto.

Para cada valoración necesitamos conocer la puntuación de 1 a 5 (sólo se permiten enteros) y la fecha en que se realiza la valoración.


**Solucion (`propuesta1.solucionCreacionTabla.sql`):**

```sql
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE IF EXISTS Valoraciones;
SET FOREIGN_KEY_CHECKS = 1;

CREATE TABLE Valoraciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    productoId INT NOT NULL,
    clienteId INT NOT NULL,
    puntuacion INT NOT NULL CHECK (puntuacion BETWEEN 1 AND 5),
    fRealizacion DATE NOT NULL,
    FOREIGN KEY (productoId) REFERENCES Productos(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    FOREIGN KEY (clienteId) REFERENCES Clientes(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    UNIQUE(productoId, clienteId)
);
```

### 2. Consultas SQL (DQL). 3 puntos

Incluya su solución en el fichero `2.solucionConsultas.sql`.

#### 2.1. Devuelva el nombre del producto, el precio unitario y las unidades compradas para las 5 líneas de pedido con más unidades. (1 punto)

#### 2.3. Devuelva el nombre del empleado, la fecha de realización del pedido, el precio total del pedido y las unidades totales del pedido para todos los pedidos que de más 7 días de antigüedad desde que se realizaron. Si un pedido no tiene asignado empleado, también debe aparecer en el listado devuelto. (2 puntos)


**Solucion (`propuesta2.solucionConsultas.sql`):**

```sql
-- 2.1. Nombre del producto, precio unitario y unidades compradas
--      para las 5 líneas de pedido con más unidades.
SELECT p.nombre, lp.unidades, lp.precio
FROM LineasPedido lp
JOIN Productos p ON lp.productoId = p.id
ORDER BY lp.unidades DESC
LIMIT 5;

-- 2.3. Nombre del empleado, fecha de realización, precio total y unidades
--      totales de los pedidos con más de 7 días de antigüedad. Los pedidos
--      sin empleado asignado también deben aparecer (LEFT JOIN).
SELECT u.nombre AS nombre_empleado,
       p.fechaRealizacion,
       SUM(lp.precio * lp.unidades) AS precio_total,
       SUM(lp.unidades) AS unidades_totales
FROM Pedidos p
LEFT JOIN LineasPedido lp ON p.id = lp.pedidoId
LEFT JOIN Empleados e ON p.empleadoId = e.id
LEFT JOIN Usuarios u ON u.id = e.usuarioId
WHERE TIMESTAMPDIFF(DAY, p.fechaRealizacion, CURDATE()) > 7
GROUP BY p.id, u.nombre, p.fechaRealizacion;
```

### 3. Procedimiento. Bonificar pedido retrasado. 3,5 puntos

Incluya su solución en el fichero `3.solucionProcedimiento.sql`.

Cree un procedimiento que permita bonificar un pedido que se ha retrasado debido a la mala gestión del empleado a cargo. Recibirá un identificador de pedido, asignará a otro empleado como gestor y reducirá un 20% el precio unitario de cada línea de pedido asociada a ese pedido. (1,5 puntos)

Asegure que el pedido estaba asociado a un empleado y en caso contrario lance excepción con el siguiente mensaje: (1 punto)

`El pedido no tiene gestor`.

Garantice que o bien se realizan todas las operaciones o bien no se realice ninguna. (1 punto)


**Solucion (`propuesta3.solucionProcedimiento.sql`):**

```sql
DELIMITER //

DROP PROCEDURE IF EXISTS p_i_bonificar_pedido //
CREATE PROCEDURE p_i_bonificar_pedido (IN p_pedidoId INT, IN p_empleadoId INT)
BEGIN
    DECLARE v_empleado_actual INT;

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;

    START TRANSACTION;

    SELECT empleadoId INTO v_empleado_actual
    FROM Pedidos
    WHERE id = p_pedidoId
    FOR UPDATE;

    IF (v_empleado_actual IS NULL) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El pedido no tiene gestor.';
    END IF;

    UPDATE Pedidos
    SET empleadoId = p_empleadoId
    WHERE id = p_pedidoId;

    UPDATE LineasPedido
    SET precio = precio * 0.8
    WHERE pedidoId = p_pedidoId;

    COMMIT;
END //

DELIMITER ;
```

### 4. Trigger. 2 puntos

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `p_limitar_unidades_mensuales_de_productos_fisicos` que, a partir de este momento, impida la venta de más de 1000 unidades al mes de cualquier producto físico.


**Solucion (`propuesta4.solucionTrigger.sql`):**

```sql
DELIMITER //

DROP TRIGGER IF EXISTS p_limitar_unidades_mensuales_de_productos_fisicos //
CREATE TRIGGER p_limitar_unidades_mensuales_de_productos_fisicos
BEFORE INSERT ON LineasPedido
FOR EACH ROW
BEGIN
    DECLARE esFisico BOOLEAN;
    DECLARE nUnidades INT;
    DECLARE fechaPedido DATE;

    SELECT EXISTS (
        SELECT 1 FROM Productos p
        JOIN TiposProducto tp ON tp.id = p.tipoProductoId
        WHERE p.id = NEW.productoId AND tp.nombre = 'Físicos'
    ) INTO esFisico;

    IF (esFisico) THEN
        SELECT fechaRealizacion INTO fechaPedido
        FROM Pedidos WHERE id = NEW.pedidoId;

        SELECT IFNULL(SUM(lp.unidades), 0) INTO nUnidades
        FROM LineasPedido lp
        JOIN Pedidos pe ON pe.id = lp.pedidoId
        WHERE lp.productoId = NEW.productoId
          AND YEAR(pe.fechaRealizacion) = YEAR(fechaPedido)
          AND MONTH(pe.fechaRealizacion) = MONTH(fechaPedido);

        IF (nUnidades + NEW.unidades > 1000) THEN
            SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se pueden vender más de 1000 unidades de cualquier producto físico.';
        END IF;
    END IF;
END //

DELIMITER ;
```


### Script de pruebas (insercion de datos, trigger y procedimiento)

Casos de prueba verificados manualmente: inserciones validas e invalidas en `Valoraciones`, acumulacion de unidades hasta activar el trigger de limite mensual, y llamadas al procedimiento de bonificacion con y sin gestor asignado.

```sql
USE tiendaOnline;

-- ============================================================
-- IMPORTANTE: ejecuta primero 0.creacionTablas.sql, 0.poblarBd.sql
-- y tu script de soluciones (tabla Valoraciones + procedimiento +
-- trigger) antes de lanzar estas pruebas.
-- ============================================================


-- ============================================================
-- 1. PRUEBAS DE INSERCIÓN EN Valoraciones
-- ============================================================

-- 1.1 Inserción válida (cliente 1 valora producto 1 con un 5)
INSERT INTO Valoraciones (productoId, clienteId, puntuacion, fRealizacion)
VALUES (1, 1, 5, CURDATE());

-- 1.2 Inserción válida distinta (mismo cliente, otro producto)
INSERT INTO Valoraciones (productoId, clienteId, puntuacion, fRealizacion)
VALUES (2, 1, 3, CURDATE());

-- Comprobación: deben aparecer las 2 valoraciones insertadas
SELECT * FROM Valoraciones;

-- 1.3 DEBE FALLAR: el mismo cliente no puede valorar el mismo producto dos veces
--     (viola la restricción UNIQUE(productoId, clienteId))
INSERT INTO Valoraciones (productoId, clienteId, puntuacion, fRealizacion)
VALUES (1, 1, 2, CURDATE());

-- 1.4 DEBE FALLAR: puntuación fuera de rango (viola el CHECK 1-5)
INSERT INTO Valoraciones (productoId, clienteId, puntuacion, fRealizacion)
VALUES (3, 1, 6, CURDATE());

-- 1.5 OJO: NO da error. Al ser la columna INT, MariaDB redondea 2.5 a 3
--     silenciosamente antes de comprobar el CHECK (que sí se cumple con 3),
--     así que esta inserción se acepta con puntuacion=3. Compruébalo con el
--     SELECT de abajo. Si necesitas rechazar decimales de verdad, tendrías
--     que añadir una comprobación explícita en un trigger.
INSERT INTO Valoraciones (productoId, clienteId, puntuacion, fRealizacion)
VALUES (3, 1, 2.5, CURDATE());

SELECT * FROM Valoraciones WHERE productoId = 3 AND clienteId = 1;


-- ============================================================
-- 2. PRUEBAS DEL TRIGGER p_limitar_unidades_mensuales_de_productos_fisicos
-- ============================================================
-- Usamos una fecha "de laboratorio" (2030-01-15) que no tiene ventas
-- previas, para controlar el test sin interferencias de los datos
-- de 0.poblarBd.sql.

-- Comprobamos que el producto 1 (Smartphone) es de tipo 'Físicos'
SELECT p.id, p.nombre, tp.nombre AS tipo
FROM Productos p JOIN TiposProducto tp ON tp.id = p.tipoProductoId
WHERE p.id = 1;

-- 2.1 Pedido de prueba en enero de 2030
INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId)
VALUES ('2030-01-15', 'Dirección de prueba', 1);
SET @pedidoTriggerId = LAST_INSERT_ID();

-- 2.2 CASO QUE DEBE PASAR: 100 unidades (por debajo del límite de 1000)
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio)
VALUES (@pedidoTriggerId, 1, 100, 100);

-- 2.3 Repetimos con más pedidos del mismo mes hasta acumular 900
--     unidades en total (100 x 9 = 900), todas deben pasar sin problema
INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-16', 'Dirección de prueba', 1);
SET @p2 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p2, 1, 100, 100);

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-17', 'Dirección de prueba', 1);
SET @p3 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p3, 1, 100, 100);

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-18', 'Dirección de prueba', 1);
SET @p4 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p4, 1, 100, 100);

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-19', 'Dirección de prueba', 1);
SET @p5 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p5, 1, 100, 100);

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-20', 'Dirección de prueba', 1);
SET @p6 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p6, 1, 100, 100);

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-21', 'Dirección de prueba', 1);
SET @p7 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p7, 1, 100, 100);

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-22', 'Dirección de prueba', 1);
SET @p8 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p8, 1, 100, 100);

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-23', 'Dirección de prueba', 1);
SET @p9 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p9, 1, 100, 100);

-- Total acumulado esperado: 900 unidades. Comprobación:
SELECT IFNULL(SUM(lp.unidades), 0) AS unidadesVendidasEnero2030
FROM LineasPedido lp
JOIN Pedidos pe ON pe.id = lp.pedidoId
WHERE lp.productoId = 1 AND YEAR(pe.fechaRealizacion) = 2030 AND MONTH(pe.fechaRealizacion) = 1;

-- 2.4 CASO LÍMITE QUE DEBE PASAR: añadir 100 más -> total exacto 1000 (no debe fallar, el límite es "más de 1000")
INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-24', 'Dirección de prueba', 1);
SET @p10 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p10, 1, 100, 100);

-- 2.5 DEBE FALLAR: cualquier unidad adicional ese mismo mes ya supera 1000
INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-25', 'Dirección de prueba', 1);
SET @p11 = LAST_INSERT_ID();
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p11, 1, 1, 100);

-- 2.6 CASO QUE DEBE PASAR: un producto DIGITAL nunca está limitado,
--     aunque se vendan muchas unidades (usamos el producto id=3, revisa
--     antes con el SELECT de abajo que sea de tipo 'Digitales')
SELECT p.id, p.nombre, tp.nombre AS tipo
FROM Productos p JOIN TiposProducto tp ON tp.id = p.tipoProductoId
WHERE tp.nombre = 'Digitales';

INSERT INTO Pedidos (fechaRealizacion, direccionEntrega, clienteId) VALUES ('2030-01-26', 'Dirección de prueba', 1);
SET @p12 = LAST_INSERT_ID();
-- Sustituye el 3 por el id real de un producto digital si fuera distinto
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES (@p12, 3, 100, 9.99);


-- ============================================================
-- 3. PRUEBAS DEL PROCEDIMIENTO p_i_bonificar_pedido
-- ============================================================

-- 3.1 Localizamos un pedido SIN gestor asignado
SELECT id, empleadoId FROM Pedidos WHERE empleadoId IS NULL LIMIT 1;

-- 3.2 DEBE FALLAR con el mensaje exacto 'El pedido no tiene gestor.'
--     (sustituye 2 por el id devuelto arriba si es distinto)
CALL p_i_bonificar_pedido(2, 1);

-- 3.3 Localizamos un pedido CON gestor asignado, y anotamos precios antes
SELECT id, empleadoId FROM Pedidos WHERE empleadoId IS NOT NULL LIMIT 1;

SELECT id, pedidoId, precio FROM LineasPedido WHERE pedidoId = 1;  -- AJUSTA el pedidoId si hace falta

-- 3.4 Ejecutamos la bonificación asignando otro empleado (ajusta los ids)
CALL p_i_bonificar_pedido(1, 2);

-- 3.5 Comprobación: el precio de cada línea debe haberse reducido un 20%
--     y el pedido debe tener el nuevo empleadoId
SELECT id, pedidoId, precio FROM LineasPedido WHERE pedidoId = 1;
SELECT id, empleadoId FROM Pedidos WHERE id = 1;
```


---

# Segunda Convocatoria (Julio)

*(rama: `modelo-segunda-convocatoria-julio`)*


**Si usted entrega sin haber sido verificada su identidad no podrá ser evaluado.**

## Tienda Online

Partiendo de la `tiendaOnline` vista durante los laboratorios descrita en el modelado conceptual siguiente:

![tiendaOnlineModeladoConceptual](https://github.com/user-attachments/assets/92eb4ba8-1ed8-488b-bb5b-448c0836fee6)

Las tablas y datos de prueba iniciales se encuentran en los ficheros `0.1.creacionTablas.sql` y `0.2.poblarBd.sql`. Cree una base de datos y ejecute dichos scripts en el citado orden.

Realice los siguientes ejercicios:

### 1. Creación de tabla. 1 punto

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.

Necesitamos conocer las promociones de nuestros productos. Para ello se propone la creación de una nueva tabla llamada `Promociones`. Cada promoción solo estará relacionada con un producto, de manera que no todos los productos están promocionados.

Para cada promoción necesitamos conocer el descuento aplicado en forma de un valor mayor que 0 y menor o igual a 1, la fecha de inicio y fecha de fin de la promoción.

Asegure que la fecha de fin de la promoción es posterior a la fecha de inicio.


**Solucion (`soluciones.sql`):**

```sql
--------------------------------------
-- 1: Creación de la tabla Promociones

CREATE TABLE Promociones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    productoId INT NOT NULL,
    descuento DECIMAL(4,2) CHECK (descuento > 0 AND descuento <= 1), -- Descuento como % (0.20 = 20%)
    fechaInicio DATE NOT NULL,
    fechaFin DATE NOT NULL,
    CONSTRAINT fk_promocion_producto FOREIGN KEY (productoId) REFERENCES Productos(id),
    CONSTRAINT chk_fechas_validas CHECK (fechaInicio < fechaFin)
);
```

### 2. Inserción de datos. 1 punto

Incluya su solución en el fichero `2.solucionInsercionTabla.sql`.
A continuación, se detallan las promociones a ingresar:

* Para el producto con ID 1, que corresponde a un smartphone, se aplicaron estas promociones:
  * un 10% de descuento, vigente desde el 1 de enero de 2024 hasta el 15 de enero de 2024.
  * un 15% de descuento, vigente desde el 1 de enero de 2025 hasta el 15 de enero de 2025.
  * un 20% de descuento, vigente desde el 1 de junio de 2025 hasta el 30 de junio de 2025.
* Para el producto con ID 2, que corresponde a un laptop, tiene las siguientes promociones:
  * 30% de descuento, válida desde el 1 de julio de 2024 hasta el 31 de julio de 2024.
  * 20% de descuento, válida desde el 1 de julio de 2025 hasta el 31 de julio de 2025.
* Para el producto con ID 3, un libro electrónico, se ofrece un 10% de descuento, desde el 1 de julio de 2025 hasta el 31 de julio de 2025.
* El producto con ID 4, correspondiente a un videojuego, tiene programada una promoción del 40% de descuento, que estará vigente del 1 al 31 de agosto de 2025.
* El producto con ID 7, una película, tuvo una promoción del 5% de descuento desde el 1 hasta el 30 de junio de 2025.
* Por último, el producto con ID 9, una tableta gráfica, cuenta con una promoción activa del 10% de descuento, vigente del 1 al 31 de julio de 2025.


**Solucion (`soluciones.sql`):**

```sql
-------------------------------------------------------------------
-- 2.1: Inserción de datos en la tabla Promociones
INSERT INTO Promociones (productoId, descuento, fechaInicio, fechaFin) VALUES
(1, 0.10, '2024-01-01', '2024-01-15'), -- Smartphone (ya ha pasado)
(1, 0.15, '2025-01-01', '2025-01-15'), -- Smartphone (ya ha pasado)
(1, 0.20, '2025-06-01', '2025-06-30'), -- Smartphone (ya ha pasado)
(2, 0.30, '2024-07-01', '2024-07-31'), -- Laptop (ya ha pasado)
(2, 0.20, '2025-07-01', '2025-07-31'), -- Laptop (activo)
(3, 0.10, '2025-07-01', '2025-07-31'), -- Libro Electrónico (activo)
(4, 0.40, '2025-08-01', '2025-08-31'), -- Videojuego (próximamente)
(7, 0.05, '2025-06-01', '2025-06-30'), -- Película (ya ha pasado)
(9, 0.10, '2025-07-01', '2025-07-31'); -- Tableta gráfica (activo)
```

### 3. Consultas. 3,5 puntos

Incluya sus soluciones en el fichero `3.solucionConsultas.sql`.

#### 3.1. (1 punto)

Obtener los ids y nombres de productos que no tienen ninguna promoción asociada.

#### 3.2. (1,25 puntos)

Obtener las los productos con promociones activas. El resultado debe incluir el id y el nombre del producto, el precio sin promoción y el precio con la promoción aplicada.

#### 3.3 (1,25 puntos)

Obtener los productos ordenados por el número de promociones que tienen asociadas. El resultado debe incluir el id del producto, el nombre del producto y el número de promociones asociadas. Si un producto nunca ha sido promocionado también debe aparecer entre los resultados.


**Solucion (`soluciones.sql`):**

```sql
-----------------------------------------------
-- 3.1: 
SELECT p.id, p.nombre
FROM Productos p
LEFT JOIN Promociones pr ON p.id = pr.productoId
WHERE pr.id IS NULL;


--------------------------------------------------------------------------------------------------------------
-- 3.2: Obtener las promociones activas actualmente, junto con los productos implicados y precios finales tras aplicar descuento.
SELECT p.id,
       p.nombre,
       p.precio AS PrecioOriginal,
       ROUND(p.precio * (1 - pr.descuento), 2) AS PrecioFinal
FROM Productos p
JOIN Promociones pr ON p.id = pr.productoId
WHERE CURRENT_DATE BETWEEN pr.fechaInicio AND pr.fechaFin;

-----------------------------------------------------
-- 3.3

SELECT 
    p.id,
    p.nombre,
    COUNT(pr.id) AS cantidad_promociones
FROM Productos p
LEFT JOIN Promociones pr ON p.id = pr.productoId
GROUP BY p.id, p.nombre
ORDER BY cantidad_promociones DESC;
```

### 4. Trigger. 2 puntos

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `trg_actualizar_precio_pedido_promocion` que aplique las promociones activas cuando se realice un pedido.


**Solucion (`soluciones.sql`):**

```sql
------------------------------------------------------------------------------------------------------------
-- 4.1 Trigger: Al insertar una nueva línea de pedido, automáticamente aplica el precio con descuento si hay promoción activa.

DELIMITER //
CREATE OR REPLACE TRIGGER trg_actualizar_precio_pedido_promocion
BEFORE INSERT ON LineasPedido FOR EACH ROW 
BEGIN
	DECLARE descuentoActivo DECIMAL(4,2);
	 
    SELECT descuento INTO descuentoActivo
    FROM Promociones
    WHERE productoId = new.productoId
      AND CURRENT_DATE BETWEEN fechaInicio AND fechaFin;

    IF (descuentoActivo IS NOT NULL) THEN
        SET new.precio = new.precio * (1 - descuentoActivo);
    END IF;
END //
DELIMITER ;

----------------------------------------------
-- Caso de estudio para forzar el trigger

INSERT INTO Pedidos (fechaRealizacion, fechaEnvio, direccionEntrega, comentarios, clienteId, empleadoId) VALUES
('2025-07-08', '2025-07-09', '123 Calle Principal', 'Pedido con importe bajo', 1, 1);

INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES
(16, 1, 1, 15.99),      -- Pedido 16, Promoción pasada
(16, 2, 3, 29.99),      -- Pedido 16, Promoción activa
(16, 4, 3, 59.99);      -- Pedido 16, Promoción futura

-----------------------------------------------
```

### 5. Procedimiento. 2,5 puntos

Incluya su solución en el fichero `5.solucionProcedimiento.sql`.

Cree un procedimiento almacenado con transacción llamado `p_anularPedido` que reciba como parámetro un número de pedido. Deberá actualizar el campo de stock de aquellos productos que participaban del pedido, así como eliminar las líneas de pedido y el propio pedido.


**Solucion (`soluciones.sql`):**

```sql
-- 5.1 Procedimiento almacenado con transacción

DELIMITER //
CREATE PROCEDURE p_anularPedido(IN p_pedidoId INT)
BEGIN
	DECLARE EXIT HANDLER FOR SQLEXCEPTION
	BEGIN
		ROLLBACK;
		SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = "Error al anular el pedido";
	END;

    START TRANSACTION;

    -- 1. Actualizar el stock de los productos según las líneas del pedido
    UPDATE Productos p
    JOIN LineasPedido lp ON p.id = lp.productoId
    SET p.stock = p.stock + lp.unidades
    WHERE lp.pedidoId = p_pedidoId;

    -- 2. Eliminar las líneas del pedido
    DELETE FROM LineasPedido 
    WHERE pedidoId = p_pedidoId;

    -- 3. Eliminar el pedido
    DELETE FROM Pedidos
    WHERE id = p_pedidoId;

    COMMIT;
END //
DELIMITER ;

--------------------------------
-- Llamada al procedimiento

CALL p_anularPedido(16);
```


---

# Tercera Convocatoria (Octubre)

*(rama: `modelo-tercera-convocatoria-octubre`)*

**Si usted entrega sin haber sido verificada su identidad no podrá ser evaluado.**

## Tienda Online

Partiendo de la `tiendaOnline` vista durante los laboratorios descrita en el modelado conceptual siguiente:

![tiendaOnlineModeladoConceptual](https://github.com/user-attachments/assets/92eb4ba8-1ed8-488b-bb5b-448c0836fee6)

Las tablas y datos de prueba iniciales se encuentran en los ficheros `0.creacionTablas.sql` y `0.poblarBd.sql`.

Realice los siguientes ejercicios:

### 1. Creación de tabla. (2,5 puntos)

Incluya su solución en el fichero `1.solucionCreacionTabla.sql`.

Necesitamos guardar información sobre los envíos realizados por la empresa y el estado de los mismos. Para ello se propone la creación de una nueva tabla llamada `Envios`. Cada pedido se enviará en un solo envío, pero en cada envío se podrán incluir varios pedidos (al menos uno). **(1 punto)**

Además de la creación de la nueva tabla, adicionalmente tendrá que modificarse la tabla de pedidos de forma conveniente **(1 punto)** para cumplir con la tercera forma normal (3FN).

Para cada envío necesitamos conocer la fecha de creación del envío, la fecha de entrega (si la hay) y el estado del mismo, que puede ser "En preparación", "Enviado" o "Entregado". Asegure que la fecha de entrega (si la hay) es posterior a la fecha de envío. **(0,5 puntos)**

**NOTA:** el fichero `1.solucionCreacionTabla.sql` incluye comentarios y código para facilitar el ejercicio.


**Solucion (`soluciones.sql`):**

```sql
--------------------------------------
-- 1: Creación de la tabla 

DROP TABLE IF EXISTS lineaspedido;
DROP TABLE IF EXISTS pedidos;
DROP TABLE IF EXISTS Envios;


CREATE TABLE Envios (
	id INT PRIMARY KEY AUTO_INCREMENT,
   fechaEnvio DATE NOT NULL,
   fechaEntrega DATE,
   estadoEnvio VARCHAR(20) NOT NULL,
   CONSTRAINT Enum_estado_envio CHECK (estadoEnvio IN ('En preparación','Enviado','Entregado')),
   CONSTRAINT fechas CHECK (fechaentrega IS NULL OR fechaentrega >= fechaenvio)
);


CREATE TABLE Pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    fechaRealizacion DATE NOT NULL,
    envioId INT,
    direccionEntrega VARCHAR(255) NOT NULL,
    comentarios TEXT,
    clienteId INT NOT NULL,
    empleadoId INT,
    FOREIGN KEY (clienteId) REFERENCES Clientes(id) 
        ON DELETE RESTRICT 
        ON UPDATE RESTRICT,
    FOREIGN KEY (empleadoId) REFERENCES Empleados(id) 
        ON DELETE SET NULL 
        ON UPDATE CASCADE,
   FOREIGN KEY (envioId) REFERENCES Envios(id) 
        ON DELETE SET NULL 
        ON UPDATE CASCADE
);

CREATE TABLE LineasPedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedidoId INT NOT NULL,
    productoId INT NOT NULL,
    unidades INT NOT NULL CHECK (unidades > 0 AND unidades <= 100),
    precio DECIMAL(10, 2) NOT NULL CHECK (precio >= 0),
    FOREIGN KEY (pedidoId) REFERENCES Pedidos(id),
    FOREIGN KEY (productoId) REFERENCES Productos(id),
    UNIQUE (pedidoId, productoId)
);
```

### 2. Inserciones. (1 punto)

Incluya su solución en el fichero `2.solucionInsercion.sql`.

Inserte los siguientes envíos en la nueva tabla con las siguientes características:

* Envío 1: fecha de envío 23/10/2025, sin fecha de entrega, estado "En preparación". Asociado a los pedidos 1 y 2.
* Envío 2: fecha de envío 03/06/2010, fecha de entrega 05/06/2010, estado "Entregado". Asociado al pedido 3.
* Envío 3: fecha de envío 22/10/2025, sin fecha de entrega, estado "En preparación". Asociado al pedido 5.

Inserte de nuevo la información de los pedidos y líneas de pedido que se encuentran en el archivo `0.poblarBd.sql` ya que ha debido recrear las tablas de `pedidos` y `lineaspedido`, borrando todas las filas en el proceso. Para los pedidos, recuerde incluir la referencia a los envíos y modificar lo necesario para cumplir con la tercera forma normal (3FN).

**NOTA:** el fichero `2.solucionInsercion.sql` incluye comentarios y código para facilitar el ejercicio.


**Solucion (`soluciones.sql`):**

```sql
-------------------------------------------------------------------
-- 2: Inserción de datos
INSERT INTO envios (fechaEnvio, fechaEntrega, estadoEnvio) VALUES
	('2025-10-23', NULL, 'En preparación'),
	('2010-06-03', '2010-06-05', 'Entregado'),
	('2025-10-22', NULL, 'En preparación');

-- Insertar datos en la tabla Pedidos

INSERT INTO Pedidos (fechaRealizacion, envioId, direccionEntrega, comentarios, clienteId, empleadoId) VALUES
('2024-10-01', 1, '123 Calle Principal', 'Entregar en la puerta. AYUDA: Este pedido estará asociado al envío 1 en preparación', 1,  1),
('2024-10-02', 1, '456 Avenida Secundaria', 'Entregar en recepción. AYUDA: Este pedido estará asociado al envío 1 en preparación', 2, 1),
('2010-06-04', 2, '123 Calle VeteTuASaber', 'Cliente con movilidad reducida. AYUDA: Este pedido estará asociado al envío 2 entregado', 1,  2),

-- Insertar pedido con más de 5 productos sin envío
('2025-10-10', NULL, '123 Calle VeteTuASaber', 'Cliente con movilidad reducida. AYUDA: Este pedido no está asociado a ningún envío', 3,  3),
-- Insertar pedido con menos de 5 productos con envío en preparación
('2025-10-12', 3, '123 Calle En preparación', 'Cliente con movilidad reducida. AYUDA: Este pedido estará asociado al envío 3 en preparación', 3,  2);


-- Insertar datos en la tabla LineasPedido - Conjunto de inserciones correctas
-- Productos permitidos para todos los clientes
INSERT INTO LineasPedido (pedidoId, productoId, unidades, precio) VALUES
(1, 1, 1, 699.99),   -- Smartphone (permitido)
(1, 5, 2, 15.99),    -- Camiseta (permitido)
(1, 3, 1, 9.99),     -- Libro Electrónico (permitido)
(3, 2, 1, 1100.00),   -- Laptop (permitido)
(3, 8, 2, 29.99),    -- Audífono (permitido)
(3, 9, 1, 12.99),     -- Tableta (permitido)
(4, 2, 1, 1100.00),   -- Laptop (permitido)
(4, 8, 4, 29.99),    -- Audífono (permitido)
(4, 9, 2, 12.99),     -- Tableta (permitido)
(5, 9, 1, 13.99);     -- Tableta (permitido)
```

### 3. Consultas. (2,5 puntos)

Incluya su solución en el fichero `2.solucionConsultas.sql`.

#### 3.1. Consulta para listar el nombre de los usuarios (sin repeticiones) que tengan al menos un pedido sin entregar. (1 punto)

#### 3.2. Consulta para mostrar los envíos sin fecha de entrega que contengan pedidos de más de un cliente distinto ordenados por fecha de envío. Para cada envío necesitamos el id de envío, la fecha de envío y el número de clientes distintos del envío. (1,5 puntos)


**Solucion (`soluciones.sql`):**

```sql
-----------------------------------------------
-- 3: Consultas
-- 3.1: 
SELECT DISTINCT u.nombre
FROM usuarios u JOIN clientes c ON u.id=c.usuarioId
JOIN pedidos p ON p.clienteId=c.id
RIGHT JOIN envios e ON e.id=p.envioId
WHERE e.id IS NULL OR e.estadoEnvio NOT LIKE 'Entregado';

--------------------------------------------------------------------------------------------------------------
-- 3.2: 
SELECT envios.id AS envioId, envios.fechaEnvio, COUNT(DISTINCT pedidos.clienteId) AS numeroClientes
FROM envios
JOIN pedidos ON pedidos.envioId = envios.id
WHERE envios.fechaEntrega IS NULL
GROUP BY envios.id
HAVING numeroClientes > 1
ORDER BY envios.fechaEnvio
```

### 4. Procedimiento. Dar de alta un nuevo envío. (2 puntos)

Incluya su solución en el fichero `3.solucionProcedimiento.sql`.

Cree un procedimiento que permita dar de alta un nuevo envío, con fecha de envío la de hoy, y asignarle todos aquellos pedidos realizados hasta el día de ayer y que NO hayan sido incluídos en ningún otro envío aún. (1,5 puntos)

Garantice que o bien se realizan todas las operaciones o bien no se realice ninguna. (0,5 puntos)


**Solucion (`soluciones.sql`):**

```sql
------------------------------------------------------------------------------------------------------------
-- 5.1 Procedimiento almacenado con transacción

DELIMITER //
-- Cree un procedimiento que permita dar de alta un nuevo envío, con fecha de envío la de hoy y estado "En preparación"
-- y asigne todos aquellos pedidos realizados hasta el día de ayer y que NO hayan sido incluídos en ningún otro envío aún.
CREATE OR REPLACE PROCEDURE p_nuevoEnvio()
BEGIN
	DECLARE envio_id INT;

	DECLARE EXIT HANDLER FOR SQLEXCEPTION
	BEGIN
		ROLLBACK;
		SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = "Error al crear el envío";
	END;

    START TRANSACTION;

    -- 1. Crear el envío, guardando su id
   INSERT INTO envios (fechaEnvio, fechaEntrega, estadoEnvio) VALUES
	(CURDATE(), NULL, 'En preparación');
	SET envio_id = LAST_INSERT_ID();
    
    UPDATE Pedidos p
    SET p.envioId = envio_id
    WHERE p.fechaRealizacion < CURDATE() AND p.envioId IS NULL;

    COMMIT;
END //
DELIMITER ;

--------------------------------
-- Llamada al procedimiento
```

### 5. Trigger. (2 puntos)

Incluya su solución en el fichero `4.solucionTrigger.sql`.

Cree un trigger llamado `trg_asegurar_envio_pedidos_fisicos` que impida que un pedido sea enviado si no incluye productos físicos.


**Solucion (`soluciones.sql`):**

```sql
-----------------------------------------------
-- 5 Trigger: No se puede asignar un pedido a un envío si no contiene productos físicos.

DELIMITER //
CREATE OR REPLACE TRIGGER trg_asegurar_envio_pedidos_fisicos
BEFORE UPDATE ON Pedidos FOR EACH ROW 
BEGIN
	
	DECLARE fisicos INT;
	
	SELECT COUNT(*) INTO fisicos
	FROM pedidos p 
		JOIN LineasPedido lp ON lp.pedidoId = p.id
		JOIN productos ON productos.id = lp.productoId
		JOIN tiposproducto tp ON productos.tipoProductoId=tp.id 
	WHERE lp.pedidoId = NEW.id AND tp.nombre LIKE 'Físicos'
	GROUP BY p.id;
		 
   IF (fisicos <=0) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'El pedido no puede incluirse en ningún envío por no contener productos físicos.';
   END IF;
END //
DELIMITER ;
```
