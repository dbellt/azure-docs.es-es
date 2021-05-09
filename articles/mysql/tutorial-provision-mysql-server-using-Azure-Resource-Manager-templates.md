---
title: 'Tutorial: Creación de un servidor de Azure Database for MySQL con las plantillas de Azure Resource Manager'
description: En este tutorial se explica cómo aprovisionar y automatizar las implementaciones de servidores de Azure Database for MySQL mediante una plantilla de Azure Resource Manager.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: json
ms.topic: tutorial
ms.date: 12/02/2019
ms.custom: mvc
ms.openlocfilehash: 6fd070417cfa20c1a8f50c79915f4fcb2ec361f3
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108279863"
---
# <a name="tutorial-provision-an-azure-database-for-mysql-server-using-azure-resource-manager-template"></a>Tutorial: Aprovisionamiento de un servidor de Azure Database for MySQL mediante las plantillas de Azure Resource Manager

La [API de REST de Azure Database for MySQL](/rest/api/mysql/) permite a los ingenieros de DevOps automatizar e integrar el aprovisionamiento, la configuración y las operaciones de las bases de datos y los servidores MySQL administrados en Azure.  Además, con dicha API se pueden crear, enumerar, administrar y eliminar bases de datos y servidores MySQL en el servicio Azure Database for MySQL.

Azure Resource Manager aprovecha la API REST subyacente para declarar y programar los recursos de Azure necesarios para implementaciones a escala y alinearlos con la infraestructura en cuanto concepto de código. La plantilla parametriza la configuración, el firewall, la red, la SKU y el nombre del recurso de Azure, lo que permite que se creen una vez y se utilicen varias.  Las plantillas de Azure Resource Manager se pueden crear fácilmente mediante [Azure Portal](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) o [Visual Studio Code](../azure-resource-manager/templates/quickstart-create-templates-use-visual-studio-code.md?tabs=CLI). Estas permiten automatizar la implementación, la estandarización y el empaquetado de aplicaciones, que se pueden integrar en la canalización de CI/CD de DevOps.  Por ejemplo, si desea implementar rápidamente una aplicación web con el servidor back-end de Azure Database for MySQL, puede realizar la implementación integral mediante esta [plantilla de inicio rápido](https://azure.microsoft.com/resources/templates/101-webapp-managed-mysql/) de la galería de GitHub.

En este tutorial, usará una plantilla de Azure Resource Manager y otras utilidades para aprender a hacer lo siguiente:

> [!div class="checklist"]
> * Creación de un servidor de Azure Database for MySQL con el punto de conexión de servicio de red virtual mediante la plantilla de Azure Resource Manager
> * Uso de la [herramienta de línea de comandos mysql](https://dev.mysql.com/doc/refman/5.6/en/mysql.html) para crear una base de datos
> * Carga de datos de muestra
> * Consultar datos
> * Actualización de datos

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-an-azure-database-for-mysql-server-with-vnet-service-endpoint-using-azure-resource-manager-template"></a>Creación de un servidor de Azure Database for MySQL con el punto de conexión de servicio de red virtual mediante la plantilla de Azure Resource Manager

Para obtener la referencia de la plantilla JSON de un servidor de Azure Database for MySQL, vaya a la referencia de la plantilla [de servidores Microsoft.DBforMySQL](/azure/templates/microsoft.dbformysql/servers). A continuación, se encuentra la plantilla JSON de ejemplo que puede usarse para crear un servidor que ejecuta Azure Database for MySQL con el punto de conexión de servicio de red virtual.
```json
{
  "apiVersion": "2017-12-01",
  "type": "Microsoft.DBforMySQL/servers",
  "name": "string",
  "location": "string",
  "tags": "string",
  "properties": {
    "version": "string",
    "sslEnforcement": "string",
    "administratorLogin": "string",
    "administratorLoginPassword": "string",
    "storageProfile": {
      "storageMB": "string",
      "backupRetentionDays": "string",
      "geoRedundantBackup": "string"
    }
  },
  "sku": {
    "name": "string",
    "tier": "string",
    "capacity": "string",
    "family": "string"
  },
  "resources": [
    {
      "name": "AllowSubnet",
      "type": "virtualNetworkRules",
      "apiVersion": "2017-12-01",
      "properties": {
        "virtualNetworkSubnetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnetName'))]",
        "ignoreMissingVnetServiceEndpoint": true
      },
      "dependsOn": [
        "[concat('Microsoft.DBforMySQL/servers/', parameters('serverName'))]"
      ]
    }
  ]
}
```
En esta solicitud, los valores que deben personalizarse son los siguientes:
+   `name`: especifique el nombre del servidor MySQL (sin el nombre de dominio).
+   `location`: especifique una región del centro de datos de Azure válida para el servidor MySQL. Por ejemplo, westus2.
+   `properties/version`: especifique la versión del servidor MySQL para implementar. Por ejemplo, 5.6 o 5.7.
+   `properties/administratorLogin`: especifique los datos de inicio de sesión del administrador de MySQL para el servidor. El nombre de inicio de sesión del administrador no puede ser azure_superuser, admin, administrator, root, guest o public.
+   `properties/administratorLoginPassword`: especifique la contraseña del usuario administrador de MySQL establecida anteriormente.
+   `properties/sslEnforcement`: especifique Enabled (Habilitado) o Disabled (Deshabilitado) para habilitar o deshabilitar sslEnforcement.
+   `storageProfile/storageMB`: especifique el tamaño máximo de almacenamiento aprovisionado necesario para el servidor en megabytes. Por ejemplo, 5120.
+   `storageProfile/backupRetentionDays`: especifique el período de retención de copia de seguridad en días. Por ejemplo, 7. 
+   `storageProfile/geoRedundantBackup`: especifique Enabled (Habilitado) o Disabled (Deshabilitado) según los requisitos de Geo-DR.
+   `sku/tier`: especifique el nivel Basic (Básico), GeneralPurpose (De uso general) o MemoryOptimized (Optimizada para memoria) de la implementación.
+   `sku/capacity`: especifique la capacidad del núcleo virtual. Los valores posibles son 2, 4, 8, 16, 32 o 64.
+   `sku/family`: especifique Gen5 para elegir la generación de hardware de la implementación de servidores.
+   `sku/name`: especifique TierPrefix_family_capacity. Por ejemplo, B_Gen5_1, GP_Gen5_16 o MO_Gen5_32. Para comprender cuáles son los valores válidos por región y nivel, consulte la documentación sobre [planes de tarifa](./concepts-pricing-tiers.md).
+   `resources/properties/virtualNetworkSubnetId`: especifique el identificador de Azure de la subred en la red virtual donde se debe colocar el servidor MySQL de Azure. 
+   `tags(optional)`: especifique las etiquetas opcionales; los pares clave-valor que usaría para clasificar los recursos para fines de facturación, etc.

Si desea crear una plantilla de Azure Resource Manager con el fin de automatizar las implementaciones de Azure Database for MySQL para su organización, la recomendación sería empezar desde el ejemplo [Plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.dbformysql/managed-mysql-with-vnet/azuredeploy.json) de la galería de GitHub de inicio rápido de Azure y crearlas a partir de esta. 

Si no está familiarizado con las plantillas de Azure Resource Manager y le gustaría probarlas, puede empezar siguiendo estos pasos:
+   Clone o descargue la [plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.dbformysql/managed-mysql-with-vnet/azuredeploy.json) de ejemplo de la galería de inicio rápido de Azure.  
+   Modifique azuredeploy.parameters.json para actualizar los valores de parámetro según sus preferencias y guarde el archivo. 
+   Use la CLI de Azure para crear el servidor MySQL de Azure mediante los siguientes comandos:

Puede usar Azure Cloud Shell en el explorador o instalar la CLI de Azure en su propio equipo para ejecutar los bloques de código de este tutorial.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

```azurecli-interactive
az login
az group create -n ExampleResourceGroup  -l "West US2"
az deployment group create -g $ ExampleResourceGroup   --template-file $ {templateloc} --parameters $ {parametersloc}
```

## <a name="get-the-connection-information"></a>Obtención de la información de conexión
Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host.
```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

El resultado está en formato JSON. Tome nota de los valores de **fullyQualifiedDomainName** y **administratorLogin**.
```json
{
  "administratorLogin": "myadmin",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus2",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
 "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-to-the-server-using-mysql"></a>Conexión al servidor con MySQL
Use la [herramienta de línea de comandos mysql](https://dev.mysql.com/doc/refman/5.6/en/mysql.html) para establecer una conexión con el servidor de Azure Database for MySQL. En este ejemplo, el comando es:
```cmd
mysql -h mydemoserver.database.windows.net -u myadmin@mydemoserver -p
```

## <a name="create-a-blank-database"></a>Crear una base de datos en blanco
Una vez conectado al servidor, cree una base de datos vacía.
```sql
mysql> CREATE DATABASE mysampledb;
```

En el símbolo del sistema, ejecute el comando siguiente para cambiar la conexión a esta base de datos recién creada:
```sql
mysql> USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>Creación de tablas en la base de datos
Ahora que sabe cómo conectarse a la base de datos de Azure Database for MySQL, complete algunas tareas básicas.

En primer lugar, cree una tabla y cárguela con algunos datos. Vamos a crear una tabla que almacena la información del inventario.
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

## <a name="load-data-into-the-tables"></a>Carga de datos en las tablas
Ahora que tiene una tabla, inserte algunos datos en ella. En la ventana de símbolo del sistema abierta, ejecute la consulta siguiente para insertar algunas filas de datos.
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Ahora tiene dos filas de datos de ejemplo en la tabla que creó anteriormente.

## <a name="query-and-update-the-data-in-the-tables"></a>Consulta y actualización de los datos en las tablas
Ejecute la consulta siguiente para recuperar información de la tabla de base de datos.
```sql
SELECT * FROM inventory;
```

También puede actualizar los datos en las tablas.
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

La fila se actualiza en consecuencia cuando se recuperan los datos.
```sql
SELECT * FROM inventory;
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, con lo que se eliminan los recursos que contiene.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), busque la opción **Grupos de recursos** y selecciónela.

2. En la lista de los grupos de recursos, elija el nombre del grupo de recursos.

3. En la página de **información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.

4. En el cuadro de diálogo de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

---

## <a name="next-steps"></a>Pasos siguientes
En este tutorial aprendió lo siguiente:
> [!div class="checklist"]
> * Creación de un servidor de Azure Database for MySQL con el punto de conexión de servicio de red virtual mediante la plantilla de Azure Resource Manager
> * Uso de la herramienta de línea de comandos mysql para crear una base de datos
> * Carga de datos de muestra
> * Consultar datos
> * Actualización de datos

> [!div class="nextstepaction"]
> [Conexión de aplicaciones a Azure Database for MySQL](./howto-connection-string.md)