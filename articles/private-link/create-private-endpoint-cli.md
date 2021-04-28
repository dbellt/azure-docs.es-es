---
title: 'Inicio rápido: Creación de un punto de conexión privado de Azure mediante la CLI de Azure'
description: Use este inicio rápido para aprender a crear un punto de conexión privado mediante la CLI de Azure.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 11/07/2020
ms.author: allensu
ms.openlocfilehash: 036052dc45b8d029dac6e137b3a878b75e6e015c
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107873404"
---
# <a name="quickstart-create-a-private-endpoint-using-azure-cli"></a>Inicio rápido: Creación de un punto de conexión privado mediante la CLI de Azure

Comience a trabajar con Azure Private Link usando un punto de conexión privado para conectarse de forma segura a una aplicación web de Azure.

En este inicio rápido, creará un punto de conexión privado para una aplicación web de Azure e implementará una máquina virtual para probar la conexión privada.  

Se pueden crear puntos de conexión privados para distintos tipos de servicios de Azure, como Azure SQL y Azure Storage.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Una aplicación web de Azure con un plan de servicio de aplicaciones de **nivel PremiumV2** o superior implementado en la suscripción de Azure.  
    * Para más información y ver un ejemplo, consulte [Inicio rápido: Creación de una aplicación web ASP.NET Core en Azure](../app-service/quickstart-dotnetcore.md). 
    * Para ver un tutorial detallado sobre cómo crear una aplicación web y un punto de conexión, consulte [Tutorial: Conexión a una aplicación web mediante un punto de conexión privado de Azure](tutorial-private-endpoint-webapp-portal.md).
* Inicie sesión en Azure Portal y compruebe que la suscripción está activa, para lo que debe ejecutar `az login`.
* Compruebe la versión de la CLI de Azure en una ventana de terminal o de comandos, para lo que debe ejecutar `az --version`. Para saber cuál es la versión más reciente, consulte las [notas de la versión más reciente](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Si no tiene la versión más reciente, actualice la instalación, para lo que debe seguir las instrucciones que encontrará en la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create):

* Denominado **CreatePrivateEndpointQS-rg**. 
* En la ubicación **eastus**.

```azurecli-interactive
az group create \
    --name CreatePrivateEndpointQS-rg \
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
* En el grupo de recursos **CreatePrivateEndpointQS-rg**.
* Ubicación de **eastus**.

```azurecli-interactive
az network vnet create \
    --resource-group CreatePrivateEndpointQS-rg\
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
    --resource-group CreatePrivateEndpointQS-rg \
    --vnet-name myVNet \
    --disable-private-endpoint-network-policies true
```

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública para el host bastión:

* Cree una dirección IP pública con redundancia de zona estándar llamada **myBastionIP**.
* En **CreatePrivateEndpointQS-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myBastionIP \
    --sku Standard
```

Use [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) para crear una subred bastión:

* Llamada **AzureBastionSubnet**.
* Con el prefijo de dirección **10.0.1.0/24**.
* En la red virtual **myVNet**.
* En el grupo de recursos **CreatePrivateEndpointQS-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.0.1.0/24
```

Use [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) para crear un host bastión:

* Denominación **myBastionHost**.
* En **CreatePrivateEndpointQS-rg**.
* Asociado a la dirección IP pública **myBastionIP**.
* Asociado a la red virtual **myVNet**.
* En la ubicación **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePrivateEndpointQS-rg \
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
* En **CreatePrivateEndpointQS-rg**.
* En la red **myVNet**.
* En la subred **myBackendSubnet**.
* Imagen del servidor **Win2019Datacenter**.

```azurecli-interactive
az vm create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myVM \
    --image Win2019Datacenter \
    --public-ip-address "" \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --admin-username azureuser
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección creará el punto de conexión privado.

Use [az webapp list](/cli/azure/webapp#az_webapp_list) para colocar el identificador de recurso de la aplicación web que creó anteriormente en una variable de shell.

Use [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create) para crear la conexión y el punto de conexión:

* Denominado **myPrivateEndpoint**.
* En el grupo de recursos **CreatePrivateEndpointQS-rg**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* Conexión denominada **myConnection**.
* Su webapp **\<webapp-resource-group-name>** .

```azurecli-interactive
id=$(az webapp list \
    --resource-group <webapp-resource-group-name> \
    --query '[].[id]' \
    --output tsv)

az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group CreatePrivateEndpointQS-rg \
    --vnet-name myVNet --subnet myBackendSubnet \
    --private-connection-resource-id $id \
    --group-id sites \
    --connection-name myConnection  
```

## <a name="configure-the-private-dns-zone"></a>Configuración de la zona DNS privada

En esta sección, creará y configurará la zona DNS privada con [az network private-dns zone create](/cli/azure/network/private-dns/zone#az_network_private_dns_zone_create).  

Usará [az network private-dns link vnet create](/cli/azure/network/private-dns/link/vnet#az_network_private_dns_link_vnet_create) para crear el vínculo de red virtual en la zona DNS.

Creará un grupo de zona DNS con [az network private-endpoint dns-zone-group create](/cli/azure/network/private-endpoint/dns-zone-group#az_network_private_endpoint_dns_zone_group_create).

* Zona denominada **privatelink.azurewebsites.net**.
* En la red virtual **myVNet**.
* En el grupo de recursos **CreatePrivateEndpointQS-rg**.
* Vínculo DNS denominado **myDNSLink**.
* Asociado a **myPrivateEndpoint**.
* Grupo de zona llamado **MyZoneGroup**.

```azurecli-interactive
az network private-dns zone create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name "privatelink.azurewebsites.net"

az network private-dns link vnet create \
    --resource-group CreatePrivateEndpointQS-rg \
    --zone-name "privatelink.azurewebsites.net" \
    --name MyDNSLink \
    --virtual-network myVNet \
    --registration-enabled false

az network private-endpoint dns-zone-group create \
   --resource-group CreatePrivateEndpointQS-rg \
   --endpoint-name myPrivateEndpoint \
   --name MyZoneGroup \
   --private-dns-zone "privatelink.azurewebsites.net" \
   --zone-name webapp
```

## <a name="test-connectivity-to-private-endpoint"></a>Prueba de la conectividad con el punto de conexión privado

En esta sección, utilizará la máquina virtual creada en el paso anterior para conectarse al servidor SQL mediante el punto de conexión privado.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com) 
 
2. En el panel de navegación de la izquierda, seleccione **Grupos de recursos**.

3. Seleccione **CreatePrivateEndpointQS-rg**.

4. Seleccione **myVM**.

5. En la página de información general para **myVM**, seleccione **Conectar** y, luego, **Bastion**.

6. Seleccione el botón **Usar bastión** azul.

7. Especifique el nombre de usuario y la contraseña proporcionados durante la creación de la máquina virtual.

8. Abra Windows PowerShell en el servidor después de conectarse.

9. Escriba `nslookup <your-webapp-name>.azurewebsites.net`. Reemplace **\<your-webapp-name>** por el nombre de la aplicación web que creó en los pasos anteriores.  Recibirá un mensaje similar al que se muestra a continuación:

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mywebapp8675.privatelink.azurewebsites.net
    Address:  10.0.0.5
    Aliases:  mywebapp8675.azurewebsites.net
    ```

    Se devuelve la dirección IP privada **10.0.0.5** para el nombre de la aplicación web.  Esta dirección se encuentra en la subred de la red virtual que creó anteriormente.

10. En la conexión bastión a **myVM**, abra Internet Explorer.

11. Escriba la dirección URL de la aplicación web, **https://\<your-webapp-name>.azurewebsites.net**.

12. Recibirá la página de aplicación web predeterminada si no se ha implementado la aplicación:

    :::image type="content" source="./media/create-private-endpoint-portal/web-app-default-page.png" alt-text="Página de aplicación web predeterminada" border="true":::

13. Cierre la conexión con **myVM**.

## <a name="clean-up-resources"></a>Limpieza de recursos 
Cuando haya terminado con el punto de conexión privado y la máquina virtual, use [az group delete](/cli/azure/group#az_group_delete) para quitar el grupo de recursos y todos los recursos que contiene:

```azurecli-interactive
az group delete \
    --name CreatePrivateEndpointQS-rg
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado lo siguiente:

* Una red virtual y un host bastión.
* Una máquina virtual.
* Un punto de conexión privado para una aplicación web de Azure.

Usó la máquina virtual para probar la conectividad de forma segura a la aplicación web a través del punto de conexión privado.

Para más información sobre los servicios que admiten un punto de conexión privado, consulte:
> [!div class="nextstepaction"]
> [Disponibilidad de Private Link](private-link-overview.md#availability)
