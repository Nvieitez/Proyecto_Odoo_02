# Proyecto_Odoo_02
Este proyecto va a tratar los diferentes usos de PgAdmin para la modificación de datos dentro de Odoo, es necesario resaltar que para poder llevar a cabo los pasos descritos a continuación debemos tener instalado tanto PgAdmin como Odoo.

> [!TIP]
> En caso de necesitar instalar alguno de los dos componenetes, los pasos de instalación están en mi anterior repositoro "Proyecto_Odoo_01":
> https://github.com/Nvieitez/Proyecto_Odoo_01

Los pasos a seguir en este proyecto son:

    0. Pasos iniciales

    1. Creación de una tabla y sus tipos de datos
    2. Inserción de los primeros datos
    3. Impresión de las diferentes empresas
    4. Impresión de los diferentes contactos 
    5. Impresión de los diferentes proveedores
    6. Impresión de los diferentes datos filtrados
    7. Actualización de datos
    8. Eliminación de diferentes datos

# 0. Pasos iniciales
Antes de empezar con las consultas, primero debemos crear una nueva base de datos, que cuente como una instalación nueva, para ello accederemos al Odoo y en el apartado de inicio de sesión podremos acceder al Manager de las diferentes bases de datos.

Dentro podremos crear una nueva para este proyecto:

![Creación de la base de datos]()

> [!WARNING]
> Es importante recordar activar los datos "Demo" para no encontrarnos con la base de datos completamente vacía y sin tablas que podamos manejar.

Una vez creada correctamente la base de datos, debemos iniciar sesión dentro del Odoo y instalar en esta base de datos los módulos de contactos y facturación.

![Instalación de los módulos]()

Una vez terminado estos pasos, podemos proceder a pgadmin y realizar las consultas necesarias

# 1. Creación de una tabla y sus tipos de datos
El primer apartado va a tratar la creación de una tabla llamada "EmpresasFCT", la cual va a contener los siguientes datos:

    idEmpresa: Clave primaria
    nombre: Texto
    quiereAlumnos: Booleano
    numAlumnos: Número entero
    fechaContacto: Tipo fecha

Para ello, primero debemos iniciar sesión en nuestro PgAdmin y acceder a la base de datos que hemos creado anteriormente

![Base de datos abierta]()

Ahora si le damos click derecho a la propia base de datos podremos acceder a la consola de SQL, donde podremos introducir diferentes comandos para realizar diferentes cosas

![Consola de SQL]()

Para crear la tabla y los datos que se nos piden, debemos introducir la siguiente consulta de SQL:

```SQL
CREATE TABLE EmpresasFCT (
    idEmpresa SERIAL PRIMARY KEY,
    nombre VARCHAR(40),
    quiereAlumnos BOOLEAN,
    numAlumnos INTEGER,
    fechaContacto DATE
);
```

Esto creará la tabla y sus respectivos datos, y le daremos un límite de 40 caractéres al nombre

Ahora comprobaremos la creación de la tabla y sus datos en el panel izquierdo:

![Comprobación de la tabla]()

# 2. Inserción de los primeros datos
Este segundo apartado va a tratar la inserción de datos en la tabla que hemos creado reciente mente, para ello introducimos el siguiente SQL de igual manera que en apartado anterior:

```SQL
INSERT INTO EmpresasFCT (nombre, quiereAlumnos, numAlumnos, fechaContacto)
VALUES
('Empresa1', TRUE, 3, '2025-01-15'),
('Empresa2', FALSE, 0, '2025-02-10'),
('Empresa3', TRUE, 5, '2024-12-20'),
('Empresa4', TRUE, 2, '2024-11-30'),
('Empresa5', FALSE, 0, '2025-03-05');
```

Aquí introduje algunos datos de caracter aleatorio y genérico, para también facilitar la búsqueda de los respectivos datos, una vez creados vamos a comprobar que se hayan creado correctamente con la siguiente consulta: 

```SQL
SELECT * FROM public.empresasfct
ORDER BY idempresa ASC
```

![Comprobación de los datos introducidos]()

# 3. Impresión de las diferentes empresas
En este apartado imprimiremos las diferentes empresas ordenadas por fecha contacto, de modo que la primera empresa que se muestre tenga la fecha más reciente, para ello utilizaremos la siguiente consulta:

```SQL
SELECT * FROM EmpresasFCT
ORDER BY fechaContacto DESC;
```

Dando el siguiente resultado:

![Comprobación de las empresas]()

# 4. Impresión de los diferentes contactos
En este apartado imprimiremos los diferentes contactos registrados mostrando su nombre, la empresa comercial y deben de estar filtrados por su ciudad, siendo esta "Tracy", además deben ordenarse alfabéticamente, para ello utilizaremos la siguiente consulta:

```SQL
SELECT
	name,city,commercial_company_name 
FROM
	res_partner 
WHERE
	is_company=false AND city = 'Tracy' 
ORDER BY
	commercial_company_name ASC;
```

Dando el siguiente resultado:

![Resultado de la impresión]()

# 5. Impresión de los diferentes proveedores
En este apartado vamos a tratar la impresión de aquellos proveedores que han emitido un rembolso, se debe mostrar el nombre de la empresa, el número de factura, la fecha de la factura, y el total sin impuestos, para ello utilizamos la siguiente consulta:

```SQL
SELECT
    res_partner.name AS NombreEmpresa,
    account_move.name AS NumeroFactura,
    account_move.invoice_date AS FechaFactura,
    account_move.amount_total AS TotalConImpuestos,
    account_move.amount_untaxed AS TotalSinImpuestos
FROM
    account_move
JOIN
    res_partner ON account_move.partner_id = res_partner.id
WHERE
    account_move.move_type = 'in_refund'
ORDER BY
    account_move.invoice_date DESC;
```

El resultado debe ser el siguiente:

![Resultado de los proveedores]()

# 6. Impresión de los diferentes datos filtrados
Este apartado va a tratar el filtrado de datos a la hora de imprimirlos, para ellos mostraremos un listado de empresas que sean clientes y tengan más de dos facturas de venta confirmadas, mostrando su nombre, número de facturas, y el total sin impuestos

La consulta que he utilizado es:

```SQL
SELECT
    res_partner.name AS NombreEmpresa,
    COUNT(account_move.id) AS NumeroFacturas,
    SUM(account_move.amount_total) AS TotalConImpuestos,
    SUM(account_move.amount_untaxed) AS TotalSinImpuestos
FROM
    account_move
JOIN
    res_partner ON account_move.partner_id = res_partner.id
WHERE
    account_move.move_type = 'out_invoice' AND account_move.state = 'posted'
GROUP BY
    res_partner.name
HAVING
    COUNT(account_move.id) > 2;
```

Dando el resultado:

![Resultado de empresas]()

# 7. Actualización de los datos
Este apartado va a tratar la actualización de un dato ya creado, pasando de "@bilbao.example.com" a "@bilbao.bizkaia.neus", usando la consulta

```SQL
UPDATE res_partner
SET email = REPLACE(email, '@bilbao.example.com', '@bilbao.bizkaia.neus')
WHERE email LIKE '%@bilbao.example.com';
```

Podemos comprobar los datos cambiados con la consulta:

```SQL
SELECT name, email
FROM res_partner
WHERE email LIKE '%@bilbao.bizkaia.neus';
```

Dando el resultado:

![Resultado de la actualización]()

# 8. Eliminación de diferentes datos
Este apartado va a tratar la eliminación de diferentes datos pertenecientes a la empresa "Ready Mat"

Primero, vamos a comprobar que exista dentro de Odoo:

![Comprobación de Ready Mat]()

Ahora vamos a eliminar todos sus datos con la siguiente consulta:

```SQL
DELETE FROM res_partner
WHERE parent_id = (SELECT id FROM res_partner WHERE name = 'Ready Mat');
```

Podemos comprobar su eliminación dentro de PgAdmin con esta consulta:

```SQL
SELECT name, parent_id
FROM res_partner
WHERE parent_id = (SELECT id FROM res_partner WHERE name = 'Ready Mat');
```

Dando este resultado:

![Comprobación eliminación de los datos]()

Finalmente comprobamos dentro de Odoo:

![Comprobación eliminación Odoo]()

Ahora ya no tendrá empleados que se muestren ni en la base de datos ni en el propio Odoo
