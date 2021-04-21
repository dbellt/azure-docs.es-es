---
title: 'Inicio rápido: Creación de un servidor mediante la CLI de Azure en Azure Database for PostgreSQL: Servidor flexible'
description: En esta guía de inicio rápido se describe cómo usar la CLI de Azure para crear una instancia de Azure Database for PostgreSQL con la opción Servidor flexible en un grupo de recursos de Azure.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/22/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 320457365a36825564154c36ad843ef665fc2d2c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791560"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-flexible-server-using-azure-cli"></a>Inicio rápido: Administración de una instancia de Azure Database for PostgreSQL con la opción Servidor flexible mediante la CLI de Azure

En este inicio rápido se muestra cómo usar los comandos de la [CLI de Azure](/cli/azure/get-started-with-azure-cli) en [Azure Cloud Shell](https://shell.azure.com) para crear una instancia de Azure Database for PostgreSQL con la opción Servidor flexible en cinco minutos. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

> [!IMPORTANT] 
> Actualmente, Azure Database for PostgreSQL con la opción Servidor flexible se encuentra en versión preliminar.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

[Azure Cloud Shell](../../cloud-shell/overview.md) es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede abrir Cloud Shell en una pestaña independiente acudiendo a [https://shell.azure.com/bash](https://shell.azure.com/bash). Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, después, seleccione **Entrar** para ejecutarlos.

Si prefiere instalar y usar la CLI de forma local, en este inicio rápido se requiere la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Requisitos previos

Será preciso que inicie sesión en su cuenta mediante el comando [az login](/cli/azure/reference-index#az_login). Tenga en cuenta la propiedad **id**, que hace referencia al **identificador de suscripción** para su cuenta de Azure.

```azurecli-interactive
az login
```

Seleccione la suscripción específica en su cuenta mediante el comando [az account set](/cli/azure/account#az_account_set). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](/cli/azure/account#az_account_list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-a-flexible-server"></a>Creación de un servidor flexible

Cree un [grupo de recursos de Azure](../../azure-resource-manager/management/overview.md) mediante el comando `az group create` y, después, cree un servidor flexible de PostgreSQL en dicho grupo de recursos. Debe proporcionar un nombre único. En el ejemplo siguiente, se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Cree un servidor flexible con el comando `az postgres flexible-server create`. Un servidor puede contener varias bases de datos. El siguiente comando crea un servidor con los valores predeterminados de servicio y los valores del [contexto local](/cli/azure/local-context) de la CLI de Azure: 

```azurecli
az postgres flexible-server create
```

El servidor creado tiene los siguientes atributos: 
- Nombre del servidor generado automáticamente, nombre de usuario de administrador, contraseña de administrador, nombre del grupo de recursos (si aún no se ha especificado en el contexto local) y de la misma ubicación que el grupo de recursos. 
- Valores predeterminados de servicio para las configuraciones de servidor restantes: nivel de proceso (De uso general), tamaño de proceso/SKU (D2s_v3, 2 núcleos virtuales y 8 GB de RAM), período de retención de copia de seguridad (7 días) y versión de PostgreSQL (12)
- El método de conectividad predeterminado es el acceso privado (integración con red virtual) con una red virtual y una subred generadas automáticamente.

> [!NOTE] 
> El método de conectividad no se puede cambiar después de crear el servidor. Por ejemplo, si seleccionó *Acceso privado (integración con red virtual)* durante la creación, no podrá cambiar a *Acceso público (direcciones IP permitidas)* después de la creación. Se recomienda encarecidamente crear un servidor con acceso privado para acceder de forma segura a su servidor mediante la integración con la red virtual. Obtenga más información sobre el acceso privado en el [artículo de conceptos](./concepts-networking.md).

Si quiere cambiar los valores predeterminados, consulte la documentación de referencia de la CLI de Azure <!--FIXME --> para obtener la lista completa de parámetros configurables de la CLI. 

> [!NOTE]
> Las conexiones a Azure Database for PostgreSQL se comunican a través del puerto 5432. Si intenta conectarse desde una red corporativa, es posible que no se permita el tráfico saliente a través del puerto 5432. En ese caso, no podrá conectarse al servidor, salvo que el departamento de TI abra el puerto 5432.

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host.

```azurecli-interactive
az postgres flexible-server show --resource-group myresourcegroup --name mydemoserver
```

El resultado está en formato JSON. Tome nota de los valores de **fullyQualifiedDomainName** y **administratorLogin**.

<!--FIXME-->
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/flexibleServers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "name": "Standard_D2s_v3",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "publicAccess": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 131072
  },
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/flexibleServers",
  "userVisibleState": "Ready",
  "version": "12"
}
```

## <a name="connect-using-postgresql-command-line-client"></a>Conexión mediante el cliente de línea de comandos PostgreSQL

Dado que el servidor flexible se creó con la opción *Acceso privado (integración con red virtual)* , deberá conectarse a su servidor desde un recurso de la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada. 

Una vez creada la máquina virtual, puede aplicar SSH en la máquina e instalar la herramienta de línea de comandos **[psql](https://www.postgresql.org/download/)** .

Con psql, conéctese con el siguiente comando. Reemplace los valores por el nombre real del servidor y la contraseña. 

```bash
psql -h mydemoserver.postgres.database.azure.com -u mydemouser -p
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no necesita estos recursos para otra guía de inicio rápido o tutorial, puede eliminarlos con el siguiente comando:

```azurecli-interactive
az group delete --name myresourcegroup
```

Si solo quiere eliminar el servidor recién creado, puede ejecutar el comando `az postgres flexible-server delete`.

```azurecli-interactive
az postgres flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Implementación de una aplicación de Django con App Service y PostgreSQL](tutorial-django-app-service-postgres.md)
