---
title: 'Tutorial: Conexión a un servidor SQL de Azure mediante un punto de conexión privado de Azure: CLI de Azure'
description: Use este tutorial para aprender a crear un servidor SQL de Azure con un punto de conexión privado mediante la CLI de Azure.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: tutorial
ms.date: 11/03/2020
ms.author: allensu
ms.custom: fasttrack-edit, devx-track-azurecli
ms.openlocfilehash: 76cdf52788a0b7e7a462acc8c9e37baba0eb81b7
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107883875"
---
# <a name="tutorial-connect-to-an-azure-sql-server-using-an-azure-private-endpoint---azure-cli"></a>Tutorial: Conexión a un servidor SQL de Azure mediante un punto de conexión privado de Azure: CLI de Azure

Un punto de conexión privado de Azure es el bloque de creación fundamental para Private Link en Azure. Permite que los recursos de Azure, como las máquinas virtuales, se comuniquen con recursos de Private Link de manera privada.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una red virtual y un host bastión.
> * Cree una máquina virtual.
> * Crear un servidor SQL de Azure y un punto de conexión privado.
> * Probar la conectividad con el punto de conexión privado del servidor SQL.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Inicie sesión en Azure Portal y compruebe que la suscripción está activa, para lo que debe ejecutar `az login`.
* Compruebe la versión de la CLI de Azure en una ventana de terminal o de comandos, para lo que debe ejecutar `az --version`. Para saber cuál es la versión más reciente, consulte las [notas de la versión más reciente](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Si no tiene la versión más reciente, actualice la instalación, para lo que debe seguir las instrucciones que encontrará en la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create):

* Denominado **CreateSQLEndpointTutorial-rg**. 
* En la ubicación **eastus**.

```azurecli-interactive
az group create \
    --name CreateSQLEndpointTutorial-rg \
    --location eastus
```

## <a name="create-a-virtual-network-and-bastion-host"></a>Creación de una red virtual y un host bastión

En esta sección, creará una red virtual, una subred y un host bastión. 

El host bastión se utilizará para conectarse de forma segura a la máquina virtual a fin de probar el punto de conexión privado.

Cree una red virtual con [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create).

* Denominada **myVNet**.
* Con el prefijo de dirección **10.0.0.0/16**.
* Subred denominada **MyBackendSubnet**.
* Con el prefijo de subred **10.0.0.0/24**.
* En el grupo de recursos **CreateSQLEndpointTutorial-rg**.
* Ubicación de **eastus**.

```azurecli-interactive
az network vnet create \
    --resource-group CreateSQLEndpointTutorial-rg\
    --location eastus \
    --name myVNet \
    --address-prefixes 10.0.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.0.0.0/24
```

Actualice la subred para deshabilitar las directivas de red del punto de conexión privado con [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update):

```azurecli-interactive
az network vnet subnet update \
    --name myBackendSubnet \
    --resource-group CreateSQLEndpointTutorial-rg \
    --vnet-name myVNet \
    --disable-private-endpoint-network-policies true
```

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública para el host bastión:

* Cree una dirección IP pública con redundancia de zona estándar llamada **myBastionIP**.
* En **CreateSQLEndpointTutorial-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myBastionIP \
    --sku Standard
```

Use [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) para crear una subred bastión:

* Llamada **AzureBastionSubnet**.
* Con el prefijo de dirección **10.0.1.0/24**.
* En la red virtual **myVNet**.
* En el grupo de recursos **CreateSQLEndpointTutorial-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.0.1.0/24
```

Use [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) para crear un host bastión:

* Denominación **myBastionHost**.
* En **CreateSQLEndpointTutorial-rg**.
* Asociado a la dirección IP pública **myBastionIP**.
* Asociado a la red virtual **myVNet**.
* En la ubicación **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

El host de Azure Bastion tarda unos minutos en implementarse.

## <a name="create-test-virtual-machine"></a>Creación de una máquina virtual de prueba

En esta sección, creará una máquina virtual que se utilizará para probar el punto de conexión privado.

Cree una máquina virtual con  [az vm create](/cli/azure/vm#az_vm_create). Cuando se le solicite, proporcione una contraseña que se usará como credenciales de inicio de sesión para la máquina virtual:

* Denominada **myVM**.
* En **CreateSQLEndpointTutorial-rg**.
* En la red **myVNet**.
* En la subred **myBackendSubnet**.
* Imagen del servidor **Win2019Datacenter**.

```azurecli-interactive
az vm create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myVM \
    --image Win2019Datacenter \
    --public-ip-address "" \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --admin-username azureuser
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-an-azure-sql-server"></a>Creación de un servidor SQL de Azure

En esta sección, va a crear un servidor SQL y una base de datos.

Use [az sql server create](/cli/azure/sql/server#az_sql_server_create) para crear un servidor SQL:

* Reemplace **\<sql-server-name>** por el nombre de servidor único.
* Reemplace **\<your-password>** por su contraseña.
* En **CreateSQLEndpointTutorial-rg**.
* En la región **eastus**.

```azurecli-interactive
az sql server create \
    --name <sql-server-name> \
    --resource-group CreateSQLEndpointTutorial-rg \
    --location eastus \
    --admin-user sqladmin \
    --admin-password <your-password>
```

Use [az sql db create](/cli/azure/sql/db#az_sql_db_create) para crear una base de datos:

* Llamada **myDataBase**.
* En **CreateSQLEndpointTutorial-rg**.
* Reemplace **\<sql-server-name>** por el nombre de servidor único.

```azurecli-interactive
az sql db create \
    --resource-group CreateSQLEndpointTutorial-rg  \
    --server <sql-server-name> \
    --name myDataBase \
    --sample-name AdventureWorksLT
```

## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección, creará el punto de conexión privado.

Use [az sql server list](/cli/azure/sql/server#az_sql_server_list) para colocar el identificador de recurso del servidor SQL en una variable del shell.

Use [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create) para crear el punto de conexión y la conexión:

* Llamado **myPrivateEndpoint**.
* En el grupo de recursos **CreateSQLEndpointTutorial-rg**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* Conexión llamada **myConnection**.

```azurecli-interactive
id=$(az sql server list \
    --resource-group CreateSQLEndpointTutorial-rg \
    --query '[].[id]' \
    --output tsv)

az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group CreateSQLEndpointTutorial-rg \
    --vnet-name myVNet --subnet myBackendSubnet \
    --private-connection-resource-id $id \
    --group-ids sqlServer \
    --connection-name myConnection  
```

## <a name="configure-the-private-dns-zone"></a>Configuración de la zona DNS privada

En esta sección, creará y configurará la zona DNS privada con [az network private-dns zone create](/cli/azure/network/private-dns/zone#az_network_private_dns_zone_create).  

Usará [az network private-dns link vnet create](/cli/azure/network/private-dns/link/vnet#az_network_private_dns_link_vnet_create) para crear el vínculo de red virtual en la zona DNS.

Creará un grupo de zona DNS con [az network private-endpoint dns-zone-group create](/cli/azure/network/private-endpoint/dns-zone-group#az_network_private_endpoint_dns_zone_group_create).

* Zona llamada **privatelink.database.windows.net**
* En la red virtual **myVNet**.
* En el grupo de recursos **CreateSQLEndpointTutorial-rg**.
* Vínculo DNS llamado **myDNSLink**.
* Asociado a **myPrivateEndpoint**.
* Grupo de zona llamado **MyZoneGroup**.

```azurecli-interactive
az network private-dns zone create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name "privatelink.database.windows.net"

az network private-dns link vnet create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --zone-name "privatelink.database.windows.net" \
    --name MyDNSLink \
    --virtual-network myVNet \
    --registration-enabled false

az network private-endpoint dns-zone-group create \
   --resource-group CreateSQLEndpointTutorial-rg \
   --endpoint-name myPrivateEndpoint \
   --name MyZoneGroup \
   --private-dns-zone "privatelink.database.windows.net" \
   --zone-name sql
```

## <a name="test-connectivity-to-private-endpoint"></a>Prueba de la conectividad con el punto de conexión privado

En esta sección, utilizará la máquina virtual creada en el paso anterior para conectarse al servidor SQL mediante el punto de conexión privado.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com) 
 
2. En el panel de navegación de la izquierda, seleccione **Grupos de recursos**.

3. Seleccione **CreateSQLEndpointTutorial-rg**.

4. Seleccione **myVM**.

5. En la página de información general para **myVM**, seleccione **Conectar** y, luego, **Bastion**.

6. Seleccione el botón **Usar bastión** azul.

7. Especifique el nombre de usuario y la contraseña proporcionados durante la creación de la máquina virtual.

8. Abra Windows PowerShell en el servidor después de conectarse.

9. Escriba `nslookup <sqlserver-name>.database.windows.net`. Reemplace **\<sqlserver-name>** por el nombre del servidor SQL que creó en los pasos anteriores.  Recibirá un mensaje similar al que se muestra a continuación:

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mysqlserver8675.privatelink.database.windows.net
    Address:  10.0.0.5
    Aliases:  mysqlserver8675.database.windows.net
    ```

    Se devuelve la dirección IP privada **10.0.0.5** del nombre del servidor SQL.  Esta dirección se encuentra en la subred de la red virtual que creó anteriormente.


10. Instale [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?preserve-view=true&view=sql-server-2017) en **myVM**.

11. Abra **SQL Server Management Studio**.

12. En **Conectar con el servidor**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Tipo de servidor | Seleccione **Motor de base de datos**.|
    | Nombre de servidor | Escriba **\<sql-server-name>.database.windows.net**. |
    | Autenticación | Seleccione **Autenticación de SQL Server**. |
    | Nombre de usuario | Escriba el nombre de usuario que especificó durante la creación del servidor. |
    | Contraseña | Escriba la contraseña que especificó durante la creación del servidor. |
    | Recordar contraseña | Seleccione **Sí**. |

13. Seleccione **Conectar**.

14. Examine las bases de datos en el menú izquierdo.

15. (Opcionalmente) Cree o consulte información de **mysqldatabase**.

16. Cierre la conexión bastión a **myVM**. 

## <a name="clean-up-resources"></a>Limpieza de recursos 

Cuando haya terminado mediante el punto de conexión privado, el servidor SQL y la máquina virtual, elimine el grupo de recursos y todos los recursos que contiene: 

```azurecli-interactive
az group delete \
    --name CreateSQLEndpointTutorial-rg
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado:

* Una red virtual y un host bastión.
* Una máquina virtual.
* Un servidor SQL de Azure con un punto de conexión privado.

Ha utilizado la máquina virtual para probar la conectividad de forma segura al servidor SQL mediante el punto de conexión privado.

Como paso siguiente, puede que también le interese el escenario de la arquitectura de **aplicación web con conectividad privada a Azure SQL Database**, que conecta una aplicación web situada fuera de la red virtual con el punto de conexión privado de una base de datos.
> [!div class="nextstepaction"]
> [Conectividad privada de una aplicación web a Azure SQL Database](/azure/architecture/example-scenario/private-web-app/private-web-app)
