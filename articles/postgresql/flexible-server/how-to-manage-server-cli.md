---
title: 'Administración de un servidor mediante la CLI de Azure en Azure Database for PostgreSQL: Servidor flexible'
description: Obtenga información sobre cómo administrar un Servidor flexible de Azure Database for PostgreSQL desde la CLI de Azure.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 9a7e16bf85293a412baf5015af825377438ebb7b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778506"
---
# <a name="manage-an-azure-database-for-postgresql---flexible-server-by-using-the-azure-cli"></a>Administración de un servidor flexible de Azure Database for PostgreSQL mediante la CLI de Azure

> [!IMPORTANT]
> Servidor flexible de Azure Database for PostgreSQL está en versión preliminar.

En este artículo se muestra cómo administrar el servidor flexible implementado en Azure. Entre las tareas de administración se incluyen el escalado de proceso y almacenamiento, el restablecimiento de contraseñas de administración y la visualización de detalles del servidor.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar. 

Debe ejecutar la versión 2.0 o posterior de la CLI de Azure en local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Inicie sesión en la cuenta mediante el comando [az login](/cli/azure/reference-index#az_login). 

```azurecli-interactive
az login
```

Seleccione la suscripción mediante el comando [az account set](/cli/azure/account). Anote el valor **id** de la salida de **az login** para usarlo como valor del argumento **subscription** en el comando siguiente. Si tiene varias suscripciones, elija aquella en la que se debe facturar el recurso. Para identificar todas las suscripciones, use el comando [az account list](/cli/azure/account#az_account_list).

```azurecli
az account set --subscription <subscription id>
```

> [!Important]
> Si aún no ha creado un servidor flexible, debe hacerlo para seguir esta guía paso a paso.

## <a name="scale-compute-and-storage"></a>Escalado de proceso y almacenamiento

Puede escalar verticalmente con facilidad el nivel de proceso, los núcleos virtuales y el almacenamiento con el siguiente comando. Para obtener una lista de todas las operaciones de servidor que puede ejecutar, vea la introducción a [az postgres flexible-server](/cli/azure/postgres/flexible-server).

```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --sku-name Standard_D4ds_v3 --storage-size 6144
```

Estos son los detalles de los argumentos del código anterior:

**Configuración** | **Valor de ejemplo** | **Descripción**
---|---|---
name | mydemoserver | Escriba un nombre único para el servidor. El nombre del servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
resource-group | myresourcegroup | Especifique el nombre del grupo de recursos de Azure.
sku-name|Standard_D4ds_v3|Escriba el nombre del nivel de proceso y el tamaño. El valor sigue la convención *Standard_ {tamaño de máquina virtual}* en abreviatura. Para más información, consulte los [planes de tarifa](../concepts-pricing-tiers.md).
storage-size | 6144 | Especifique la capacidad de almacenamiento del servidor en megabytes. El mínimo es 5120, que aumenta en incrementos de 1024.

> [!IMPORTANT]
> El almacenamiento no se puede reducir verticalmente. 

## <a name="manage-postgresql-databases-on-a-server"></a>Administración de bases de datos de PostgreSQL en un servidor

Hay una serie de aplicaciones que se pueden usar para conectarse al servidor de Azure Database for PostgreSQL. Si el equipo cliente tiene PostgreSQL instalado, puede usar una instancia local de [psql](https://www.postgresql.org/docs/current/static/app-psql.html). Ahora se va a usar la herramienta de línea de comandos psql para conectarse al servidor de Azure Database for PostgreSQL.

1. Ejecute el siguiente comando **psql**:

   ```bash
   psql --host=<servername> --port=<port> --username=<user> --dbname=<dbname>
   ```

   Por ejemplo, el siguiente comando se conecta a la base de datos predeterminada denominada **postgres** en el servidor de PostgreSQL **mydemoserver.postgres.database.azure.com** con las credenciales de acceso. Cuando se le pida, escriba el elemento `<server_admin_password>` que ha elegido.
  
   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin --dbname=postgres
   ```

   Después de la conexión, la herramienta psql muestra un símbolo del sistema **postgres** donde escribir comandos SQL. En la salida de la conexión inicial aparece una advertencia si la versión de psql que usa difiere de la versión del servidor de Azure Database for PostgreSQL.

   Ejemplo de salida de psql:

   ```bash
   psql (11.3, server 12.1)
   WARNING: psql major version 11, server major version 12.
            Some psql features might not work.
   SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
   Type "help" for help.

   postgres=>
   ```

   > [!TIP]
   > Si el firewall no está configurado para permitir la dirección IP de su cliente, se produce el siguiente error:
   >
   > "psql: FATAL:  no pg_hba.conf entry for host `<IP address>`, user "myadmin", database "postgres", SSL on FATAL: SSL connection is required. Specify SSL options and retry."
   >
   > Confirme que la dirección IP del cliente está permitida en las reglas de firewall.

2. Escriba el comando siguiente en el símbolo del sistema para crear una base de datos en blanco denominada **postgresdb**:

    ```bash
    CREATE DATABASE postgresdb;
    ```

3. En el símbolo del sistema, ejecute el comando siguiente para cambiar las conexiones a la base de datos **postgresdb** recién creada:

    ```bash
    \c postgresdb
    ```

4. Escriba `\q` y seleccione Entrar para salir de psql.

En esta sección se ha conectado al servidor de Azure Database for PostgreSQL mediante psql y ha creado una base de datos de usuarios en blanco.

## <a name="reset-the-admin-password"></a>Restablecimiento de la contraseña de administrador

Puede cambiar la contraseña del rol de administrador con el siguiente comando:

```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!IMPORTANT]
> Elija una contraseña que tenga un mínimo de 8 caracteres y un máximo de 128. La contraseña debe contener caracteres de tres de las categorías siguientes: 
> - Letras del alfabeto inglés en mayúscula
> - Letras del alfabeto inglés en minúscula
> - Números
> - Caracteres no alfanuméricos

## <a name="delete-a-server"></a>Eliminación de un servidor

Para eliminar el servidor flexible de Azure Database for PostgreSQL, ejecute el comando [az postgres flexible-server delete](/cli/azure/postgres/flexible-server#az_postgresql_flexible_server_delete).

```azurecli-interactive
az postgres flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de los conceptos de copia de seguridad y restauración](concepts-backup-restore.md)
- [Ajuste y supervisión del servidor](concepts-monitoring.md)