---
title: Habilitación del acceso privado con Private Link (versión preliminar) mediante la CLI
titleSuffix: Azure Digital Twins
description: Vea cómo habilitar el acceso privado de las soluciones de Azure Digital Twins con Private Link, mediante la CLI.
author: baanders
ms.author: baanders
ms.date: 02/09/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 47299467c51d128a228e4120eb8e0d0ff4deebc4
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110456468"
---
# <a name="enable-private-access-with-private-link-preview-azure-cli"></a>Habilitación del acceso privado con Private Link (versión preliminar): Azure CLI

[!INCLUDE [digital-twins-private-link-selector.md](../../includes/digital-twins-private-link-selector.md)]

En este artículo se describen las distintas formas de [habilitar Private Link con un punto de conexión privado para una instancia de Azure Digital Twins](concepts-security.md#private-network-access-with-azure-private-link-preview) (actualmente en versión preliminar). La configuración de un punto de conexión privado para la instancia de Azure Digital Twins le permite protegerla y eliminar la exposición pública, además de evitar la filtración de datos desde la instancia de [Azure Virtual Network (VNet)](../virtual-network/virtual-networks-overview.md).

En este artículo se describe el proceso que utiliza la [CLI de Azure](/cli/azure/what-is-azure-cli).

Estos son los pasos que se describen en este artículo: 
1. Activación de Private Link y configuración de un punto de conexión privado para una instancia de Azure Digital Twins.
1. Deshabilitación o habilitación de marcas de acceso a la red pública para restringir el acceso de la API únicamente a las conexiones de Private Link.

## <a name="prerequisites"></a>Requisitos previos

Para poder configurar un punto de conexión privado, necesitará una [red virtual de Azure (VNet)](../virtual-network/virtual-networks-overview.md)  en la que se pueda implementar el punto de conexión. Si todavía no tiene una red virtual, puede seguir uno de los [inicios rápidos](../virtual-network/quick-create-portal.md) de Azure Virtual Network para configurarla.

## <a name="manage-private-endpoints-for-an-azure-digital-twins-instance"></a>Administración de puntos de conexión privados para una instancia de Azure Digital Twins 

Al usar la [CLI de Azure](/cli/azure/what-is-azure-cli), puede configurar puntos de conexión privados con Private Link en una instancia de Azure Digital Twins que ya existe (no se puede agregar como parte de la creación de instancias). Después, puede seguir viéndolos y administrarlos mediante comandos adicionales de la CLI. 

>[!TIP]
> También puede configurar un punto de conexión de Private Link mediante el servicio Private Link, en lugar de hacerlo con la instancia de Azure Digital Twins. Esto también proporciona las mismas opciones de configuración y el mismo resultado final.
>
> Para más información sobre la configuración de recursos de Private Link, consulte la documentación de Azure Private Link para [Azure Portal](../private-link/create-private-endpoint-portal.md), la [CLI de Azure](../private-link/create-private-endpoint-cli.md), las [plantillas de Resource Manager](../private-link/create-private-endpoint-template.md) o [PowerShell](../private-link/create-private-endpoint-powershell.md).

### <a name="add-a-private-endpoint-to-an-existing-instance"></a>Adición de un punto de conexión privado a una instancia existente

Para crear un punto de conexión privado y vincularlo a una instancia de Azure Digital Twins, use el comando [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest&preserve-view=true#az_network_private_endpoint_create). Identifique la instancia de Azure Digital Twins mediante su identificador completo en el parámetro `--private-connection-resource-id`.

Este es un ejemplo que usa el comando para crear un punto de conexión privado, con solo los parámetros necesarios.

```azurecli-interactive
az network private-endpoint create --connection-name <private-link-service-connection> --name <name-for-private-endpoint> --resource-group <resource-group> --subnet <subnet-ID> --private-connection-resource-id "/subscriptions/<subscription-ID>/resourceGroups/<resource-group>/providers/Microsoft.DigitalTwins/digitalTwinsInstances/<Azure-Digital-Twins-instance-name>" 
```

Para obtener una lista completa de los parámetros obligatorios y opcionales, así como más ejemplos de creación de puntos de conexión privados, consulte la documentación de referencia del comando [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest&preserve-view=true#az_network_private_endpoint_create).

### <a name="manage-private-endpoint-connections-on-the-instance"></a>Administración de las conexiones de punto de conexión privado de la instancia

Una vez que se ha creado un punto de conexión privado para la instancia de Azure Digital Twins, puede usar los comandos [az dt network private-endpoint connection](/cli/azure/dt/network/private-endpoint/connection?view=azure-cli-latest&preserve-view=true) para continuar administrando las **conexiones** de punto de conexión privado relacionadas con la instancia. Estas son algunas de las operaciones:
* Mostrar una conexión de punto de conexión privado
* Establecer el estado de conexión del punto de conexión privado
* Eliminar la conexión del punto de conexión privado
* Mostrar todas las conexiones del punto de conexión privado de una instancia

Para más información y ejemplos, consulte la documentación de referencia [az dt network private-endpoint](/cli/azure/dt/network/private-endpoint?view=azure-cli-latest&preserve-view=true).

### <a name="manage-other-private-link-information-on-an-azure-digital-twins-instance"></a>Administración de información adicional de Private Link en una instancia de Azure Digital Twins

Puede obtener información adicional sobre el estado de Private Link de la instancia con el comando [az dt network private-link](/cli/azure/dt/network/private-link?view=azure-cli-latest&preserve-view=true). Estas son algunas de las operaciones:
* Enumerar los vínculos privados asociados a una instancia de Azure Digital Twins
* Mostrar un vínculo privado asociado a la instancia

Para más información y ejemplos, consulte la documentación de referencia del comando [az dt network private-link](/cli/azure/dt/network/private-link?view=azure-cli-latest&preserve-view=true).

## <a name="disable--enable-public-network-access-flags"></a>Habilitación o deshabilitación de las marcas de acceso a la red pública

Puede configurar la instancia de Azure Digital Twins para denegar todas las conexiones públicas y permitir solo las conexiones a través de puntos de conexión privados a fin de mejorar la seguridad de la red. Esta acción se realiza con una **marca de acceso a la red pública**. 

Esta directiva permite restringir el acceso de la API únicamente a las conexiones de Private Link. Cuando la marca de acceso a la red pública está establecida en *deshabilitada*, todas las llamadas API REST al plano de datos de la instancia de Azure Digital Twins desde la nube pública devolverán `403, Unauthorized`. Como alternativa, cuando la directiva se establece en *deshabilitada* y se realiza una solicitud a través de un punto de conexión privado, la llamada API se realizará correctamente.

En este artículo se indica cómo actualizar el valor de la marca de red mediante la [CLI de Azure](/cli/azure/) o la [herramienta de comandos ARMClient](https://github.com/projectkudu/ARMClient). Para obtener instrucciones sobre cómo hacerlo con Azure Portal, consulte la versión para [Azure Portal](how-to-enable-private-link-portal.md) en este artículo.

### <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

En la CLI de Azure, puede deshabilitar o habilitar el acceso a la red pública agregando un parámetro `--public-network-access` al comando `az dt create`. Aunque este comando también se puede utilizar para crear una nueva instancia, puede utilizarlo para editar las propiedades de una instancia existente proporcionándole el nombre de una instancia que ya exista. (Para más información acerca de este comando, consulte la [documentación de referencia](/cli/azure/dt?view=azure-cli-latest&preserve-view=true#az_dt_create) o las [instrucciones generales para configurar una instancia de Azure Digital Twins](how-to-set-up-instance-cli.md#create-the-azure-digital-twins-instance)).

Para **deshabilitar** el acceso a la red pública para una instancia de Azure Digital Twins, use el parámetro `--public-network-access` de la siguiente manera:

```azurecli-interactive
az dt create --dt-name <name-of-existing-instance> --resource-group <resource-group> --public-network-access Disabled
```

Para **habilitar** el acceso a la red pública en una instancia en la que está deshabilitado, use el siguiente comando parecido:

```azurecli-interactive
az dt create --dt-name <name-of-existing-instance> --resource-group <resource-group> --public-network-access Enabled
```

### <a name="usethe-armclientcommand-tool"></a>Uso de la herramienta de comandos ARMClient 

Con la [herramienta de comandos ARMClient](https://github.com/projectkudu/ARMClient), el acceso a la red pública se habilita o deshabilita mediante los comandos siguientes. 

Para **deshabilitar** el acceso a la red pública:
  
```cmd/sh
armclient login 

armclient PATCH /subscriptions/<your-Azure-subscription-ID>/resourceGroups/<your-resource-group>/providers/Microsoft.DigitalTwins/digitalTwinsInstances/<your-Azure-Digital-Twins-instance>?api-version=2020-12-01 "{ 'properties': { 'publicNetworkAccess': 'disabled' } }"  
```

Para **habilitar** el acceso a la red pública:  
  
```cmd/sh
armclient login 

armclient PATCH /subscriptions/<your-Azure-subscription-ID>/resourceGroups/<your-resource-group>/providers/Microsoft.DigitalTwins/digitalTwinsInstances/<your-Azure-Digital-Twins-instance>?api-version=2020-12-01 "{ 'properties': { 'publicNetworkAccess': 'enabled' } }"  
``` 

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre Private Link para Azure: 
* [¿Qué es el servicio Azure Private Link?](../private-link/private-link-service-overview.md)