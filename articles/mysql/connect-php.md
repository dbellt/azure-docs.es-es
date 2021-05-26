---
title: 'Inicio rápido: Conexión mediante PHP: Azure Database for MySQL'
description: En este tutorial rápido se proporcionan ejemplos de código de PHP que se pueden usar para conectarse a Azure Database for MySQL y consultar datos en este servicio.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: f6d97417d4e1ac549c0aeb0311013455bb967a41
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110071909"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-mysql"></a>Inicio rápido: Uso de PHP para conectarse y consultar datos en Azure Database for MySQL
En este tutorial rápido se muestra cómo conectarse a una instancia de Azure Database for MySQL mediante una aplicación de [PHP](https://secure.php.net/manual/intro-whatis.php). Se indica cómo usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos en la base de datos.

## <a name="prerequisites"></a>Requisitos previos
Para esta guía de inicio rápido, necesitará lo siguiente:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free).
- [Inicio rápido: Creación de un servidor único de Azure Database for MySQL mediante Azure Portal](./quickstart-create-mysql-server-database-using-azure-portal.md). <br/> o la [CLI de Azure](./quickstart-create-mysql-server-database-using-azure-cli.md) si no tiene uno.
- En función de si usa el acceso público o privado, complete **UNA** de las acciones siguientes para habilitar la conectividad.

    |Acción| Método de conectividad|Guía paso a paso|
    |:--------- |:--------- |:--------- |
    | **Configurar reglas de firewall** | Público | [Portal](./howto-manage-firewall-using-portal.md) <br/> [CLI](./howto-manage-firewall-using-cli.md)|
    | **Configurar el punto de conexión de servicio** | Público | [Portal](./howto-manage-vnet-using-portal.md) <br/> [CLI](./howto-manage-vnet-using-cli.md)|
    | **Configuración del vínculo privado** | Private | [Portal](./howto-configure-privatelink-portal.md) <br/> [CLI](./howto-configure-privatelink-cli.md) |

- [Creación de una base de datos y un usuario que no sea administrador](./howto-create-users.md?tabs=single-server)
- Instalación de la versión más reciente de PHP para su sistema operativo
    - [Instalación en Mac OS X](https://secure.php.net/manual/install.macosx.php)
    - [Instalación sobre sistemas Unix](https://secure.php.net/manual/install.unix.php)
    - [PHP en Windows](https://secure.php.net/manual/install.windows.php)

> [!NOTE]
> Estamos usando la biblioteca de [MySQLi](https://www.php.net/manual/en/book.mysqli.php) para administrar la conexión y consultar el servidor en este inicio rápido.

## <a name="get-connection-information"></a>Obtención de información sobre la conexión
Para obtener la información de conexión del servidor de bases de datos de Azure Portal, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Vaya a la página Azure Database for MySQL. Puede buscar **Azure Database for MySQL** y selecciónelo.
:::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/find-azure-mysql-in-portal.png" alt-text="Enlace de Azure Database for MySQL":::

2. Seleccione el servidor de MySQL (como **mydemoserver**).
3. En la página de **información general**, copie el nombre del servidor completo junto al **nombre del servidor** y el nombre de usuario administrador junto a **Nombre de inicio de sesión del administrador del servidor**. Para copiar el nombre del servidor o nombre de host, mantenga el cursor sobre él y seleccione el icono **Copiar**.

> [!IMPORTANT]
> - Si olvidó la contraseña, puede [restablecerla ](./howto-create-manage-server-portal.md#update-admin-password).
> - Reemplace los parámetros de **host, username, password** y **db_name** con sus propios valores**.

## <a name="step-1-connect-to-the-server"></a>Paso 1: Conexión al servidor
SSL está habilitado de manera predeterminada. Es posible que tenga que descargar el [certificado de SSL DigiCertGlobalRootG2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) para conectarse desde el entorno local. Este código llama a:
- [mysqli_init](https://secure.php.net/manual/mysqli.init.php) para inicializar MySQLi.
- [mysqli_ssl_set](https://www.php.net/manual/en/mysqli.ssl-set.php) para apuntar a la ruta de acceso del certificado SSL. Esto es necesario para el entorno local, pero no es necesario para la aplicación web de App Service o las máquinas virtuales de Azure.
- [mysqli_real_connect](https://secure.php.net/manual/mysqli.real-connect.php) para conectarse a MySQL.
- [mysqli_close](https://secure.php.net/manual/mysqli.close.php) para cerrar la conexión.


```php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin@mydemoserver';
$password = 'your_password';
$db_name = 'your_database';

//Initializes MySQLi
$conn = mysqli_init();

mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/DigiCertGlobalRootG2.crt.pem", NULL, NULL);

// Establish the connection
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin@mydemoserver', 'yourpassword', 'quickstartdb', 3306, NULL, MYSQLI_CLIENT_SSL);

//If connection failed, show the error
if (mysqli_connect_errno())
{
    die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```
[¿Tiene problemas? Háganoslo saber.](https://aka.ms/mysql-doc-feedback)

## <a name="step-2-create-a-table"></a>Paso 2: Creación de una tabla
Use el código siguiente para conectarse. Este código llama a:
- [mysqli_query](https://secure.php.net/manual/mysqli.query.php) para ejecutar la consulta.
```php
// Run the create table query
if (mysqli_query($conn, '
CREATE TABLE Products (
`Id` INT NOT NULL AUTO_INCREMENT ,
`ProductName` VARCHAR(200) NOT NULL ,
`Color` VARCHAR(50) NOT NULL ,
`Price` DOUBLE NOT NULL ,
PRIMARY KEY (`Id`)
);
')) {
printf("Table created\n");
}
```

## <a name="step-3-insert-data"></a>Paso 3: Insertar datos
Use el código siguiente para insertar datos mediante la instrucción SQL **INSERT**. Este código usa los métodos:
- [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) para crear una instrucción INSERT preparada.
- [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) para enlazar los parámetros de cada valor de columna insertada.
- [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php)
- [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php) para cerrar la instrucción mediante el método using.


```php
//Create an Insert prepared statement and run it
$product_name = 'BrandNewProduct';
$product_color = 'Blue';
$product_price = 15.5;
if ($stmt = mysqli_prepare($conn, "INSERT INTO Products (ProductName, Color, Price) VALUES (?, ?, ?)"))
{
    mysqli_stmt_bind_param($stmt, 'ssd', $product_name, $product_color, $product_price);
    mysqli_stmt_execute($stmt);
    printf("Insert: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
    mysqli_stmt_close($stmt);
}

```

## <a name="step-4-read-data"></a>Paso 4: Lectura de datos
Use el código siguiente para leer los datos mediante la instrucción SQL **SELECT**.  El código usa el método:
- [mysqli_query](https://secure.php.net/manual/mysqli.query.php) ejecuta la consulta **SELECT**.
- [mysqli_fetch_assoc](https://secure.php.net/manual/mysqli-result.fetch-assoc.php) para capturar las filas resultantes.

```php
//Run the Select query
printf("Reading data from table: \n");
$res = mysqli_query($conn, 'SELECT * FROM Products');
while ($row = mysqli_fetch_assoc($res))
 {
    var_dump($row);
 }

```


## <a name="step-5-delete-data"></a>Paso 5: Eliminación de datos
Use el código siguiente para eliminar filas mediante la instrucción SQL **DELETE**. El código usa los métodos:
- [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) para crear una instrucción Delete preparada.
- [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) enlaza los parámetros.
- [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php) ejecuta la instrucción Delete preparada.
- [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php) cierra la instrucción.

```php
//Run the Delete statement
$product_name = 'BrandNewProduct';
if ($stmt = mysqli_prepare($conn, "DELETE FROM Products WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 's', $product_name);
mysqli_stmt_execute($stmt);
printf("Delete: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar todos los recursos utilizados durante esta guía de inicio rápido, elimine el grupo de recursos con el siguiente comando:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Administración de un servidor de Azure Database for MySQL mediante Azure Portal](./howto-create-manage-server-portal.md)<br/>

> [!div class="nextstepaction"]
> [Administración con la CLI de Azure de una instancia de Azure Database for MySQL: Servidor único](./how-to-manage-single-server-cli.md)

[¿No encuentra lo que busca? Háganoslo saber.](https://aka.ms/mysql-doc-feedback)
