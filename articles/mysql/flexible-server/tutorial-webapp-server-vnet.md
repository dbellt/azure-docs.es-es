---
title: 'Tutorial: Creación de un servidor flexible (versión preliminar) de Azure Database for MySQL y una aplicación web de Azure App Service en la misma red virtual'
description: Guía de inicio rápido para crear un servidor flexible (versión preliminar) de Azure Database for MySQL con una aplicación web en una red virtual
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 03/18/2021
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 13baf8f033338e242610d7b8c4eec14806cd5ec5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770028"
---
# <a name="tutorial-create-an-azure-database-for-mysql---flexible-server-preview-with-app-services-web-app-in-virtual-network"></a>Tutorial: Creación de un servidor flexible (versión preliminar) de Azure Database for MySQL con una aplicación web de App Services en una red virtual

> [!IMPORTANT]
> El servidor flexible de Azure Database for MySQL está actualmente en versión preliminar pública.

En este tutorial se muestra cómo crear una aplicación web de Azure App Service con el servidor flexible de MySQL (versión preliminar) dentro de una [red virtual](../../virtual-network/virtual-networks-overview.md).

En este tutorial, aprenderá a:
>[!div class="checklist"]
> * Crear un servidor flexible de MySQL en una red virtual
> * Crear una subred para delegar en App Service
> * Creación de una aplicación web
> * Adición de una aplicación web a la red virtual
> * Conexión a Postgres desde la aplicación web 

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

En este artículo es necesario que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, de forma local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Deberá iniciar sesión en su cuenta mediante el comando [az login](/cli/azure/reference-index#az_login). Anote la propiedad **id** de la salida del comando para el nombre de la suscripción correspondiente.

```azurecli
az login
```

Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account). Sustituya la propiedad **subscription ID** de la salida de **az login** de su suscripción en el marcador de posición de identificador de suscripción.

```azurecli
az account set --subscription <subscription ID>
```

## <a name="create-an-azure-database-for-mysql-flexible-server"></a>Creación de un servidor flexible de Azure Database for MySQL

Cree un servidor flexible privado dentro de una red virtual (VNet) con el siguiente comando:
```azurecli
az mysql flexible-server create --resource-group myresourcegroup --location westus2
```
Copie la cadena de conexión y el nombre de la red virtual recién creada. Este comando realiza las siguientes acciones, que pueden tardar algunos minutos:

- Cree el grupo de recursos si todavía no existe.
- Genera un nombre de servidor, si no se proporciona alguno.
- Crea una nueva red virtual para el nuevo servidor de MySQL. Anote el nombre de la red virtual y de la subred que se crearon para el servidor, ya que debe agregar la aplicación web a la misma red virtual.
- Crea un nombre de usuario administrador y una contraseña para el servidor, si no se proporciona alguna.
- Crea una base de datos vacía llamada **flexibleserverdb**.

> [!NOTE]
> Anote la contraseña que se generará si no proporciona alguna. Si olvida la contraseña, tendrá que restablecerla con el comando ``` az mysql flexible-server update```.

## <a name="create-subnet-for-app-service-endpoint"></a>Creación de una subred para un punto de conexión de App Service
Ahora es necesario tener una subred delegada en el punto de conexión de la aplicación web de App Service. Ejecute el siguiente comando para crear una nueva subred en la misma red virtual en la que se haya creado el servidor de bases de datos. 

```azurecli
az network vnet subnet create -g myresourcegroup --vnet-name VNETName --name webappsubnetName  --address-prefixes 10.0.1.0/24  --delegations Microsoft.Web/serverFarms --service-endpoints Microsoft.Web
```
Anote el nombre de la red virtual y el nombre de la subred después de este comando, ya que lo necesitará para agregar una regla de integración de red virtual para la aplicación web después de crearla. 

## <a name="create-a-web-app"></a>Creación de una aplicación web

En esta sección, creará un host de aplicación en la aplicación de App Service y lo conectará a la base de datos MySQL. Asegúrese de que se encuentra en la raíz del repositorio correspondiente al código de la aplicación en el terminal.

Creación de una aplicación de App Service (el proceso de host) con el comando az webapp up

```azurecli
az webapp up --resource-group myresourcegroup --location westus2 --plan testappserviceplan --sku P2V2 --name mywebapp
```

> [!NOTE]
> - Para el argumento --location, use la misma ubicación que usó para la base de datos en la sección anterior.
> - Reemplace _&lt;app-name>_ con un nombre único en todo Azure (el punto de conexión del servidor es https://\<app-name>.azurewebsites.net). Los caracteres permitidos para <app-name> son A-Z, 0-9 y -. Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.
> - App Service de nivel Básico no admite la integración con redes virtuales. Utilice el nivel Estándar o Premium. 

Este comando realiza las siguientes acciones, que pueden tardar algunos minutos:

- Cree el grupo de recursos si todavía no existe. (En este comando se usa el mismo grupo de recursos en el que se creó la base de datos anterior).
- Crea el plan de App Service ```testappserviceplan``` en el plan de tarifa Básico (B1), si no existe. --plan y --sku son opcionales.
- Cree la aplicación de App Service, si no existe.
- Habilite el registro predeterminado de la aplicación, si aún no está habilitado.
- Cargue el repositorio mediante la implementación del archivo ZIP con la automatización de compilación habilitada.

## <a name="add-the-web-app-to-the-virtual-network"></a>Adición de una aplicación web a la red virtual

Use el comando **az webapp vnet-integration** para agregar una integración de red virtual regional a una aplicación web. Reemplace _&lt;vnet-name>_ y _&lt;subnet-name_ por el nombre de red virtual y de subred que usa el servidor flexible.

```azurecli
az webapp vnet-integration add -g myresourcegroup -n  mywebapp --vnet VNETName --subnet webappsubnetName
```

## <a name="configure-environment-variables-to-connect-the-database"></a>Configuración de las variables de entorno para conectarse a la base de datos

Con el código ahora implementado en App Service, el paso siguiente consiste en conectar la aplicación al servidor flexible en Azure. El código de la aplicación espera encontrar información de la base de datos en una serie de variables de entorno. Para establecer variables de entorno en App Service, cree "valores de aplicación" mediante el comando ```az webapp config appsettings``` set.

```azurecli
az webapp config appsettings set --settings DBHOST="<mysql-server-name>.mysql.database.azure.com" DBNAME="flexibleserverdb" DBUSER="<username>" DBPASS="<password>"
```

- Reemplace _&lt;mysql-server-name>_ , _&lt;username>_ y _&lt;password>_ por el comando de servidor flexible recién creado.
- Reemplace _&lt;username>_ y _&lt;password>_ por las credenciales que el comando también ha generado automáticamente.
- El grupo de recursos y el nombre de la aplicación se extraen de los valores almacenados en caché del archivo .azure/config.
- El comando crea valores de configuración denominados DBHOST, DBNAME, DBUSER y DBPASS. Si el código de la aplicación usa un nombre diferente para la información de la base de datos, use esos nombres para los valores de la aplicación, tal como se mencionó en el código.

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie todos los recursos que creó en el tutorial mediante el siguiente comando. Este comando elimina todos los recursos del grupo de recursos.

```azurecli
az group delete -n myresourcegroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Asignación de un nombre DNS personalizado existente a Azure App Service](../../app-service/app-service-web-tutorial-custom-domain.md)
