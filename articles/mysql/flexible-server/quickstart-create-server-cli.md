---
title: 'Inicio rápido: Creación de un servidor mediante la CLI de Azure en Azure Database for MySQL con la opción Servidor flexible'
description: En este inicio rápido se describe cómo usar la CLI de Azure para crear una instancia de Azure Database for MySQL con la opción Servidor flexible en un grupo de recursos de Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/21/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 7addfc3a0d91b85c4d63afa4ee6a55b5202c3855
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770244"
---
# <a name="quickstart-create-an-azure-database-for-mysql-flexible-server-using-azure-cli"></a>Inicio rápido: Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure

En este inicio rápido se muestra cómo usar los comandos de la [CLI de Azure](/cli/azure/get-started-with-azure-cli) en [Azure Cloud Shell](https://shell.azure.com) para crear una instancia de Azure Database for MySQL con la opción Servidor flexible en cinco minutos. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL con la opción Servidor flexible está en versión preliminar pública.

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

```azurecli-interactive
az account set --subscription <subscription id>
```

## <a name="create-a-flexible-server"></a>Creación de un servidor flexible

Cree un [grupo de recursos de Azure](../../azure-resource-manager/management/overview.md) mediante el comando `az group create` y, después, cree un servidor flexible de MySQL en dicho grupo de recursos. Debe proporcionar un nombre único. En el ejemplo siguiente, se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `eastus2`.

```azurecli-interactive
az group create --name myresourcegroup --location eastus2
```

Cree un servidor flexible con el comando `az mysql flexible-server create`. Un servidor puede contener varias bases de datos. El siguiente comando crea un servidor con los valores predeterminados de servicio y los valores del [contexto local](/cli/azure/local-context) de la CLI de Azure: 

```azurecli-interactive
az mysql flexible-server create
```

El servidor creado tiene los siguientes atributos: 
- Nombre del servidor generado automáticamente, nombre de usuario de administrador, contraseña de administrador, nombre del grupo de recursos (si aún no se ha especificado en el contexto local) y de la misma ubicación que el grupo de recursos. 
- Valores predeterminados de servicio para las configuraciones de servidor restantes: nivel de proceso (Flexible), tamaño de proceso/SKU (B1MS), período de retención de copia de seguridad (7 días) y versión de MySQL (5.7)
- El método de conectividad predeterminado es el acceso privado (integración con red virtual) con una red virtual y una subred generadas automáticamente.

> [!NOTE] 
> El método de conectividad no se puede cambiar después de crear el servidor. Por ejemplo, si seleccionó *Acceso privado (integración con red virtual)* durante la creación, no podrá cambiar a *Acceso público (direcciones IP permitidas)* después de la creación. Se recomienda encarecidamente crear un servidor con acceso privado para acceder de forma segura a su servidor mediante la integración con la red virtual. Obtenga más información sobre el acceso privado en el [artículo de conceptos](./concepts-networking.md).

Si quiere cambiar algún valor predeterminado, consulte en la [documentación de referencia](/cli/azure/mysql/flexible-server)de la CLI de Azure la lista completa de parámetros configurables de la CLI. 

A continuación se incluye una salida de ejemplo: 

```json
Command group 'mysql flexible-server' is in preview. It may be changed/removed in a future release.
Creating Resource Group 'groupXXXXXXXXXX'...
Creating new vnet "serverXXXXXXXXXVNET" in resource group "groupXXXXXXXXXX"...
Creating new subnet "serverXXXXXXXXXSubnet" in resource group "groupXXXXXXXXXX" and delegating it to "Microsoft.DBforMySQL/flexibleServers"...
Creating MySQL Server 'serverXXXXXXXXX' in group 'groupXXXXXXXXXX'...
Your server 'serverXXXXXXXXX' is using sku 'Standard_B1ms' (Paid Tier). Please refer to https://aka.ms/mysql-pricing for pricing details
Creating MySQL database 'flexibleserverdb'...
Make a note of your password. If you forget, you would have to reset your password with 'az mysql flexible-server update -n serverXXXXXXXXX -g groupXXXXXXXXXX -p <new-password>'.
{
  "connectionString": "server=serverXXXXXXXXX.mysql.database.azure.com;database=flexibleserverdb;uid=secureusername;pwd=securepasswordstring",
  "databaseName": "flexibleserverdb",
  "host": "serverXXXXXXXXX.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.DBforMySQL/flexibleServers/serverXXXXXXXXX",
  "location": "East US 2",
  "password": "securepasswordstring",
  "resourceGroup": "groupXXXXXXXXXX",
  "skuname": "Standard_B1ms",
  "subnetId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.Network/virtualNetworks/serverXXXXXXXXXVNET/subnets/serverXXXXXXXXXSubnet",
  "username": "secureusername",
  "version": "5.7"
}
```

Si quiere cambiar algún valor predeterminado, consulte en la [documentación de referencia](/cli/azure/mysql/flexible-server) de la CLI de Azure la lista completa de parámetros configurables de la CLI. 

## <a name="create-a-database"></a>Crear una base de datos
Ejecute el siguiente comando para crear una base de datos, **newdatabase**, si aún no ha creado una.

```azurecli-interactive
az mysql flexible-server db create -d newdatabase
```

> [!NOTE]
> Las conexiones a Azure Database for MySQL se comunican a través del puerto 3306. Si intenta conectarse desde una red corporativa, es posible que no se permita el tráfico saliente a través del puerto 3306. En ese caso no podrá conectarse al servidor, salvo que el departamento de TI abra el puerto 3306.

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host.

```azurecli-interactive
az mysql flexible-server show --resource-group myresourcegroup --name mydemoserver
```

El resultado está en formato JSON. Tome nota de los valores de **fullyQualifiedDomainName** y **administratorLogin**. A continuación se muestra un ejemplo de salida de JSON: 

```json
{
  "administratorLogin": "myadminusername",
  "administratorLoginPassword": null,
  "delegatedSubnetArguments": {
    "subnetArmResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/mydemoserverVNET/subnets/mydemoserverSubnet"
  },
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/flexibleServers/mydemoserver",
  "location": "East US 2",
  "name": "mydemoserver",
  "publicNetworkAccess": "Disabled",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 0,
    "name": "Standard_B1ms",
    "tier": "Burstable"
  },
  "storageProfile": {
    "backupRetentionDays": 7,
    "fileStorageSkuName": "Premium_LRS",
    "storageAutogrow": "Disabled",
    "storageIops": 0,
    "storageMb": 10240
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/flexibleServers",
  "version": "5.7"
}
```

## <a name="connect-and-test-the-connection-using-azure-cli"></a>Conexión y prueba de la conexión mediante la CLI de Azure

El servidor flexible de Azure Database for MySQL le permite conectarse al servidor MySQL con el comando ```az mysql flexible-server connect``` de la CLI de Azure. Este comando permite probar la conectividad al servidor de bases de datos, crear una base de datos de inicio rápido y ejecutar consultas directamente en el servidor sin tener que instalar mysql.exe o MySQL Workbench.  También puede usar ejecutar el comando en modo interactivo para ejecutar varias consultas.

Ejecute el script siguiente para probar y validar la conexión a la base de datos desde el entorno de desarrollo.

```azurecli-interactive
az mysql flexible-server connect -n <servername> -u <username> -p <password> -d <databasename>
```
**Ejemplo**:
```azurecli-interactive
az mysql flexible-server connect -n mysqldemoserver1 -u dbuser -p "dbpassword" -d newdatabase
```
Debería ver la siguiente salida para una conexión correcta:

```output
Command group 'mysql flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Connecting to newdatabase database.
Successfully connected to mysqldemoserver1.
```
Si se produjo un error en la conexión, pruebe estas soluciones:
- Compruebe si el puerto 3306 está abierto en la máquina cliente.
- Compruebe si el nombre de usuario y la contraseña del administrador del servidor son correctos.
- Compruebe si ha configurado una regla de firewall para la máquina cliente.
- Compruebe si ha configurado el servidor con acceso privado en redes virtuales y asegúrese de que la máquina cliente se encuentre en la misma red virtual.

Ejecute el siguiente comando para ejecutar una consulta única con el argumento ```--querytext```, ```-q```.

```azurecli-interactive
az mysql flexible-server connect -n <server-name> -u <username> -p "<password>" -d <database-name> --querytext "<query text>"
```

**Ejemplo**:
```azurecli-interactive
az mysql flexible-server connect -n mysqldemoserver1 -u dbuser -p "dbpassword" -d newdatabase -q "select * from table1;" --output table
```
Para más información sobre el comando ```az mysql flexible-server connect```, consulte la documentación de [conexión y consulta](connect-azure-cli.md).

## <a name="connect-using-mysql-command-line-client"></a>Conexión mediante el cliente de línea de comandos mysql

Si ha creado el servidor flexible con acceso privado (integración con red virtual), deberá conectarse a su servidor desde un recurso en la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada con el servidor flexible. Para más información, consulte cómo se configura la [documentación de acceso privado](how-to-manage-virtual-network-portal.md).

Si ha creado el servidor flexible con acceso público (direcciones IP permitidas), puede agregar la dirección IP local a la lista de reglas de firewall del servidor. En la [documentación para la creación o administración de reglas de firewall](how-to-manage-firewall-portal.md) encontrará instrucciones paso a paso.

Puede usar [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) o [MySQL Workbench](./connect-workbench.md) para conectarse al servidor desde su entorno local. Azure Database for MySQL: servidor flexible admite la conexión de las aplicaciones cliente al servicio MySQL mediante la Seguridad de la capa de transporte (TLS), que anteriormente se denominaba Capa de sockets seguros (SSL). TLS es un protocolo estándar del sector que garantiza conexiones de red cifradas entre el servidor de bases de datos y las aplicaciones cliente, lo que le permite satisfacer los requisitos de cumplimiento. Para conectar con el servidor flexible de MySQL, deberá descargar el [certificado SSL público](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) para la comprobación de la entidad de certificación. Para más información sobre la conexión con conexiones cifradas o la deshabilitación de SSL, consulte la documentación [Conexión al servidor flexible de Azure Database for MySQL con conexiones cifradas](how-to-connect-tls-ssl.md).

En el ejemplo siguiente se muestra cómo conectarse al servidor flexible mediante la interfaz de la línea de comandos de MySQL. En primer lugar, instalará la línea de comandos de MySQL si aún no lo está. Descargará el certificado DigiCertGlobalRootCA necesario para las conexiones SSL. Use la configuración de la cadena de conexión --ssl-mode=REQUIRED para aplicar la comprobación del certificado TLS/SSL. Pase la ruta de acceso al archivo del certificado local al parámetro --ssl-ca. Reemplace los valores por el nombre real del servidor y la contraseña.

```bash
sudo apt-get install mysql-client
wget --no-check-certificate https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem
mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p --ssl-mode=REQUIRED --ssl-ca=DigiCertGlobalRootCA.crt.pem
```

Si ha aprovisionado un servidor flexible mediante el **acceso público**, también puede usar [Azure Cloud Shell](https://shell.azure.com/bash) para conectarse a dicho servidor flexible mediante el cliente MySQL preinstalado, como se muestra a continuación:

Para usar Azure Cloud Shell para conectarse a un servidor flexible, tendrá que permitir el acceso de red desde Azure Cloud Shell al servidor flexible. Para ello, en Azure Portal, puede ir a la hoja **Redes** del servidor flexible de MySQL y activar la casilla de la sección **Firewall** "Permitir el acceso público desde cualquier servicio de Azure a este servidor" que se muestra en la captura de pantalla siguiente y hacer clic en Guardar para conservar la configuración.

 > :::image type="content" source="./media/quickstart-create-server-portal/allow-access-to-any-azure-service.png" alt-text="Captura de pantalla que muestra cómo permitir el acceso de Azure Cloud Shell al servidor flexible de MySQL para la configuración de red de acceso público.":::
 
 
> [!NOTE]
> La comprobación de **Allow public access from any Azure service within Azure to this server** (Permitir el acceso público desde cualquier servicio de Azure a este servidor) solo debe usarse solo para desarrollo o pruebas. Configura el firewall para permitir conexiones desde las direcciones IP asignadas a cualquier servicio o recurso de Azure, incluidas las conexiones de las suscripciones de otros clientes.

Haga clic en **Probar** para iniciar el Azure Cloud Shell y use los siguientes comandos para conectarse a su servidor flexible. Use el nombre del servidor, el nombre de usuario y la contraseña en el comando. 

```azurecli-interactive
wget --no-check-certificate https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem
mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p --ssl=true --ssl-ca=DigiCertGlobalRootCA.crt.pem
```
> [!IMPORTANT]
> Al conectarse al servidor flexible mediante Azure Cloud Shell, necesitará usar el parámetro --ssl=true y no --ssl-mode=REQUIRED.
> La razón principal es que Azure Cloud Shell incluye el cliente de mysql.exe preinstalado por la distribución de MariaDB, que requiere el parámetro --ssl, mientras que el cliente de MySQL de la distribución de Oracle requiere el parámetro --ssl-mode.

Si ve el siguiente mensaje de error al conectarse a su servidor flexible después del comando anterior, significa que no ha configurado la regla de firewall mediante la casilla Allow public access from any Azure service within Azure to this server (Permitir el acceso público desde cualquier servicio de Azure a este servidor) mencionada antes o que la opción no se ha guardado. Vuelva a intentar configurar el firewall y reintente la operación.

ERROR 2002 (HY000): No es posible conectarse a un servidor MySQL en <servername> (115)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no necesita estos recursos para otra guía de inicio rápido o tutorial, puede eliminarlos con el siguiente comando:

```azurecli-interactive
az group delete --name myresourcegroup
```

Si solo quiere eliminar el servidor recién creado, puede ejecutar el comando `az mysql server delete`.

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes

>[!div class="nextstepaction"]
> [Conexión y consulta mediante la CLI de Azure](connect-azure-cli.md)
> [Conexión al servidor flexible de Azure Database for MySQL con conexiones cifradas](how-to-connect-tls-ssl.md) 
> [Compilación de una aplicación web PHP (Laravel) con MySQL](tutorial-php-database-app.md)
