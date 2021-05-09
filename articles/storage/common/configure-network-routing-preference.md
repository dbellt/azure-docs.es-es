---
title: Configuración de las preferencias de enrutamiento de red
titleSuffix: Azure Storage
description: Configure la preferencia de enrutamiento de red de la cuenta de Azure Storage para especificar cómo se enruta el tráfico de red a la cuenta desde los clientes a través de Internet.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 03/17/2021
ms.author: normesta
ms.reviewer: santoshc
ms.subservice: common
ms.openlocfilehash: e9d1f7f520a7613abde520c35fbacc7355973021
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108208228"
---
# <a name="configure-network-routing-preference-for-azure-storage"></a>Configuración de las preferencias de enrutamiento de red para Azure Storage

En este artículo se describe cómo puede configurar la preferencia de enrutamiento de red y los puntos de conexión específicos de la ruta para la cuenta de almacenamiento. 

La preferencia de enrutamiento de red le permite especificar cómo se enruta el tráfico de red a la cuenta desde los clientes a través de Internet. Los puntos de conexión específicos de la ruta son nuevos puntos de conexión que Azure Storage crea para la cuenta de almacenamiento. Estos puntos de conexión enrutan el tráfico a través de una ruta de acceso de su elección sin necesidad de cambiar su preferencia de enrutamiento predeterminada. Para más información, consulte [Preferencia de enrutamiento de red para Azure Storage](network-routing-preference.md).

## <a name="configure-the-routing-preference-for-the-default-public-endpoint"></a>Configuración de la preferencia de enrutamiento del punto de conexión público predeterminado

De forma predeterminada, la preferencia de enrutamiento del punto de conexión público de la cuenta de almacenamiento se establece en la red global de Microsoft. Puede elegir entre la red global de Microsoft y el enrutamiento de Internet como una preferencia de enrutamiento predeterminada del punto de conexión público de la cuenta de almacenamiento. Para más información acerca de la diferencia entre estos dos tipos de enrutamiento, consulte [Preferencia de enrutamiento de red para Azure Storage](network-routing-preference.md). 

### <a name="portal"></a>[Portal](#tab/azure-portal)

Para cambiar la preferencia de enrutamiento a enrutamiento de Internet:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Vaya a la cuenta de almacenamiento desde el portal.

3. En **Configuración**, elija **Redes**.

    > [!div class="mx-imgBorder"]
    > ![Opción del menú Redes](./media/configure-network-routing-preference/networking-option.png)

4.  En la pestaña **Firewalls y redes virtuales**, en **Enrutamiento de red**, cambie el valor **Preferencia de enrutamiento** a **Enrutamiento de Internet**.

5.  Haga clic en **Save**(Guardar).

    > [!div class="mx-imgBorder"]
    > ![Opción Enrutamiento de Internet](./media/configure-network-routing-preference/internet-routing-option.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Inicie sesión en la suscripción de Azure con el comando `Connect-AzAccount` y siga las instrucciones que aparecen en pantalla para autenticarse.

   ```powershell
   Connect-AzAccount
   ```

2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento que hospedará el sitio web estático.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

3. Para cambiar la preferencia de enrutamiento al enrutamiento de Internet, use el comando [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) y establezca el parámetro `--routing-choice` en `InternetRouting`.

   ```powershell
   Set-AzStorageAccount -ResourceGroupName <resource-group-name> `
    -AccountName <storage-account-name> `
    -RoutingChoice InternetRouting
   ```

   Reemplace el valor del marcador de posición `<resource-group-name>` por el nombre del grupo de recursos que contiene la cuenta de almacenamiento.

   Reemplace el valor del marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Inicie sesión en la suscripción de Azure.

   - Para iniciar Azure Cloud Shell, inicie sesión en [Azure Portal](https://portal.azure.com).

   - Para iniciar sesión en la instalación local de la CLI, ejecute el comando [az login](/cli/azure/reference-index#az_login):

     ```azurecli
     az login
     ```
2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento que hospedará el sitio web estático.

   ```azurecli
   az account set --subscription <subscription-id>
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

3. Para cambiar la preferencia de enrutamiento al enrutamiento de Internet, use el comando [az storage account update](/cli/azure/storage/account#az_storage_account_update) y establezca el parámetro `--routing-choice` en `InternetRouting`.

   ```azurecli
   az storage account update --name <storage-account-name> --routing-choice InternetRouting
   ```

   Reemplace el valor de marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

---

## <a name="configure-a-route-specific-endpoint"></a>Configuración de un punto de conexión específico de la ruta

También puede configurar un punto de conexión específico de la ruta. Por ejemplo, puede establecer la preferencia de enrutamiento para el punto de conexión predeterminado en *Enrutamiento de Internet* y, a continuación, publicar un punto de conexión específico de la ruta que permita el tráfico entre los clientes de Internet y la cuenta de almacenamiento a través de la red global de Microsoft.

Esta preferencia solo afecta al punto de conexión específico de la ruta. Esta preferencia no afecta a la preferencia de enrutamiento predeterminada.  

### <a name="portal"></a>[Portal](#tab/azure-portal)

1.  Vaya a la cuenta de almacenamiento desde el portal.

2.  En **Configuración**, elija **Redes**.

3.  En la pestaña **Firewalls y redes virtuales**, en **Publicar los puntos de conexión específicos de la ruta**, seleccione la preferencia de enrutamiento del punto de conexión específico de la ruta y, a continuación, haga clic en **Guardar**.

    En la imagen siguiente se muestra la opción de **Enrutamiento de red de Microsoft** seleccionada.

    > [!div class="mx-imgBorder"]
    > ![Opción Enrutamiento de red de Microsoft](./media/configure-network-routing-preference/microsoft-network-routing-option.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Para configurar un punto de conexión específico de la ruta, use el comando [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount). 

   - Para crear un punto de conexión específico de la ruta que use la preferencia de enrutamiento de red de Microsoft, establezca el parámetro `-PublishMicrosoftEndpoint` en `true`. 

   - Para crear un punto de conexión específico de la ruta que use la preferencia de enrutamiento de Internet, establezca el parámetro `-PublishInternetEndpointTo` en `true`.  

   En el ejemplo siguiente se crea un punto de conexión específico de la ruta que utiliza la preferencia de enrutamiento de red de Microsoft.

   ```powershell
   Set-AzStorageAccount -ResourceGroupName <resource-group-name> `
    -AccountName <storage-account-name> `
    -PublishMicrosoftEndpoint $true
   ```

   Reemplace el valor del marcador de posición `<resource-group-name>` por el nombre del grupo de recursos que contiene la cuenta de almacenamiento.

   Reemplace el valor del marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Para configurar un punto de conexión específico de la ruta, use el comando [az storage account update](/cli/azure/storage/account#az_storage_account_update). 

   - Para crear un punto de conexión específico de la ruta que use la preferencia de enrutamiento de red de Microsoft, establezca el parámetro `--publish-microsoft-endpoints` en `true`. 

   - Para crear un punto de conexión específico de la ruta que use la preferencia de enrutamiento de Internet, establezca el parámetro `--publish-internet-endpoints` en `true`.  

   En el ejemplo siguiente se crea un punto de conexión específico de la ruta que utiliza la preferencia de enrutamiento de red de Microsoft.

   ```azurecli
   az storage account update --name <storage-account-name> --publish-microsoft-endpoints true
   ```

   Reemplace el valor del marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

---

## <a name="find-the-endpoint-name-for-a-route-specific-endpoint"></a>Búsqueda del nombre de un punto de conexión específico de la ruta

Si configuró un punto de conexión específico de la ruta, puede encontrar el punto de conexión en las propiedades de la cuenta de almacenamiento.

### <a name="portal"></a>[Portal](#tab/azure-portal)

1.  En **Configuración**, elija **Propiedades**.

    > [!div class="mx-imgBorder"]
    > ![Opción de menú Propiedades](./media/configure-network-routing-preference/properties.png)

2.  El punto de conexión de **Enrutamiento de red de Microsoft** aparece en cada servicio que admite preferencias de enrutamiento. En esta imagen se muestra el punto de conexión para los servicios de blobs y archivos

    > [!div class="mx-imgBorder"]
    > ![Opción Enrutamiento de red de Microsoft para puntos de conexión específicos de la ruta](./media/configure-network-routing-preference/routing-url.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Para imprimir los puntos de conexión en la consola, use la propiedad `PrimaryEndpoints` del objeto de cuenta de almacenamiento.

   ```powershell
   Get-AzStorageAccount -ResourceGroupName <resource-group-name> -Name <storage-account-name>
   write-Output $StorageAccount.PrimaryEndpoints
   ```

   Reemplace el valor del marcador de posición `<resource-group-name>` por el nombre del grupo de recursos que contiene la cuenta de almacenamiento.

   Reemplace el valor del marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Para imprimir los puntos de conexión en la consola, use la propiedad [az storage account show](/cli/azure/storage/account#az_storage_account_show) del objeto de cuenta de almacenamiento.

   ```azurecli
   az storage account show -g <resource-group-name> -n <storage-account-name>
   ```

   Reemplace el valor del marcador de posición `<resource-group-name>` por el nombre del grupo de recursos que contiene la cuenta de almacenamiento.

   Reemplace el valor del marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

---

## <a name="see-also"></a>Vea también

- [Preferencia de enrutamiento de red](network-routing-preference.md)
- [Configuración de redes virtuales y firewalls de Azure Storage](storage-network-security.md)
- [Recomendaciones de seguridad para Blob Storage](../blobs/security-recommendations.md)
