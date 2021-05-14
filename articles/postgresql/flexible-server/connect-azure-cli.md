---
title: 'Inicio rápido: Conexión mediante la CLI de Azure a Azure Database for PostgreSQL Servidor flexible'
description: Esta guía de inicio rápido proporciona varias maneras de conectarse con la CLI de Azure a Azure Database for PostgreSQL Servidor flexible.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc, devx-track-azurecli
ms.topic: quickstart
ms.date: 03/06/2021
ms.openlocfilehash: 3017d10abc910233e0627037349c0dfd966fd88e
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107883803"
---
# <a name="quickstart-connect-and-query-with-azure-cli--with-azure-database-for-postgresql---flexible-server"></a>Inicio rápido: Conexión y consulta mediante la CLI de Azure a Azure Database for PostgreSQL Servidor flexible

> [!IMPORTANT]
> Actualmente Servidor flexible de Azure Database for PostgreSQL se encuentra en versión preliminar.

En este inicio rápido se muestra cómo puede conectarse a una instancia de Azure Database for PostgreSQL Servidor flexible mediante la CLI de Azure con el comando ```az postgres flexible-server connect```. Este comando permite probar la conectividad con el servidor de base de datos y ejecutar consultas. También puede ejecutar varias consultas mediante el modo interactivo. 

## <a name="prerequisites"></a>Requisitos previos
- Una cuenta de Azure. Si no tiene una, [obtenga la versión de evaluación gratuita](https://azure.microsoft.com/free/).
- Instale la última versión de la [CLI de Azure](/cli/azure/install-azure-cli) (2.20.0 o superior).
- Inicie sesión mediante la CLI de Azure on el comando ```az login```. 
- Active la persistencia de parámetros con ```az config param-persist on```. La persistencia de parámetros le ayudará a usar el contexto local sin tener que repetir muchos argumentos, como el grupo de recursos o la ubicación.

## <a name="create-an-postgresql-flexible-server"></a>Creación de un servidor flexible de PostgreSQL

Lo primero que crearemos es un servidor administrado de PostgreSQL. En [Azure Cloud Shell](https://shell.azure.com/), ejecute el siguiente script y anote el **nombre del servidor**, el **nombre de usuario** y la **contraseña** generados a partir de este comando.

```azurecli
az postgres flexible-server create --public-access <your-ip-address>
```
Puede proporcionar argumentos adicionales a este comando para personalizarlo. Consulte todos los argumentos del comando [az postgres flexible-server create](/cli/azure/postgres/flexible-server#az_postgres_flexible_server_create).

## <a name="view-all-the-arguments"></a>Visualización de todos los argumentos
Puede ver todos los argumentos de este comando con el argumento ```--help```. 

```azurecli
az postgresql flexible-server connect --help
```

## <a name="test-database-server-connection"></a>Prueba de la conexión al servidor de bases de datos
Puede probar y validar la conexión a la base de datos desde el entorno de desarrollo mediante el comando.

```azurecli
az postgres flexible-server connect -n <servername> -u <username> -p "<password>" -d <databasename>
```
**Ejemplo**: 
```azurecli
az postgres flexible-server connect -n postgresdemoserver -u dbuser -p "dbpassword" -d postgres
```
Verá en la salida si la conexión se realizó correctamente.
```output
Command group 'postgres flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Successfully connected to postgresdemoserver.
Local context is turned on. Its information is saved in working directory C:\mydir. You can run `az local-context off` to turn it off.
Your preference of  are now saved to local context. To learn more, type in `az local-context --help`
```

Si se produjo un error en la conexión, pruebe estas soluciones:
- Compruebe si el puerto 5432 está abierto en la máquina cliente.
- Compruebe si el nombre de usuario y la contraseña del administrador del servidor son correctos.
- Compruebe si ha configurado una regla de firewall para la máquina cliente.
- Compruebe si ha configurado el servidor con acceso privado en redes virtuales y asegúrese de que la máquina cliente se encuentre en la misma red virtual.

## <a name="run-single-query"></a>Ejecución de una sola consulta
Puede ejecutar una sola consulta con el comando mediante el argumento ```--querytext``` o ```-q```.

```azurecli
az postgres flexible-server connect -n <server-name> -u <username> -p "<password>" -d <database-name> -q "<query-text>"
```

**Ejemplo**: 
```azurecli
az postgresql flexible-server connect -n postgresdemoserver -u dbuser -p "dbpassword" -d flexibleserverdb -q "select * from table1;" --output table
```

Verá una salida como la que se muestra a continuación:

```output
Command group 'postgres flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Successfully connected to postgresdemoserver.
Ran Database Query: 'select * from table1;'
Retrieving first 30 rows of query output, if applicable.
Closed the connection postgresdemoserver.
Local context is turned on. Its information is saved in working directory C:\mydir. You can run `az local-context off` to turn it off.
Your preference of  are now saved to local context. To learn more, type in `az local-context --help`
Txt    Val
-----  -----
test   200
test   200
test   200
test   200
test   200
test   200
test   200
```

## <a name="run-multiple-queries-using-interactive-mode"></a>Ejecución de varias consultas mediante el modo interactivo
Puede ejecutar varias consultas mediante el modo **interactivo**. Para habilitar el modo interactivo, ejecute el siguiente comando.

```azurecli
az postgres flexible-server connect -n <servername> -u <username> -p "<password>" -d <databasename>
```

**Ejemplo**:

```azurecli
az postgresql flexible-server connect -n postgresdemoserver -u dbuser -p "dbpassword" -d flexibleserverdb --interactive
```

Verá la experiencia del shell de **psql**, como se muestra a continuación:

```bash
Command group 'postgres flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Password for earthyTurtle7:
Server: PostgreSQL 12.5
Version: 3.0.0
Chat: https://gitter.im/dbcli/pgcli
Home: http://pgcli.com
postgres> create database pollsdb;
CREATE DATABASE
Time: 0.308s
postgres> exit
Goodbye!
Local context is turned on. Its information is saved in working directory C:\sunitha. You can run `az local-context off` to turn it off.
Your preference of  are now saved to local context. To learn more, type in `az local-context --help`
```


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración del servidor](./how-to-manage-server-cli.md)
