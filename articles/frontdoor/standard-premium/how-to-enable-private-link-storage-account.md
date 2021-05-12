---
title: Conexión de Azure Front Door Prémium a un origen de cuenta de almacenamiento con Private Link
titleSuffix: Azure Private Link
description: Aprenda a conectar Azure Front Door Prémium a una cuenta de almacenamiento de forma privada.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 03/04/2021
ms.author: duau
ms.openlocfilehash: 991727c369b70fdd84507f6d32b6f8a3405fa4ac
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633562"
---
# <a name="connect-azure-front-door-premium-to-a-storage-account-origin-with-private-link"></a>Conexión de Azure Front Door Prémium a un origen de cuenta de almacenamiento con Private Link

Este artículo le guía por los pasos para configurar la SKU prémium de Azure Front Door a fin de conectarse al origen de la cuenta de almacenamiento de forma privada mediante el servicio Azure Private Link.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-private-link-to-a-storage-account"></a>Habilitación de Private Link para una cuenta de almacenamiento
 
En esta sección, asignará el servicio Private Link a un punto de conexión privado creado en una red privada de Azure Front Door. 

1. En el perfil de Azure Front Door Prémium, en *Settings* (Configurción), seleccione **Origin groups** (Grupos de origen).

1. Seleccione el grupo de origen que contiene el origen de la cuenta de almacenamiento para la que quiere habilitar Private Link.

1. Seleccione **+ Add an origin** (+ Agregar un origen) para agregar un nuevo origen de cuenta de almacenamiento, o seleccione un origen de cuenta de almacenamiento creado anteriormente en la lista.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-storage-account.png" alt-text="Captura de pantalla de la habilitación de Private Link para una cuenta de almacenamiento.":::

1. En **Select an Azure resource** (Seleccione un recurso de Azure), elija **In my directory** (En mi directorio). Seleccione o escriba los siguientes valores para configurar el sitio al que quiere que Azure Front Door Prémium se conecte de forma privada.

    | Configuración | Value |
    | ------- | ----- |
    | Region | Seleccione la región que sea la misma o la más cercana a su origen. |
    | Tipo de recurso | Seleccione **Microsoft.Storage/storageAccounts**. |
    | Resource | Seleccione la cuenta de almacenamiento. |
    | Subrecurso de destino | Puede seleccionar *blob* o *web*. |
    | Mensaje de solicitud | Personalice el mensaje o elija el predeterminado. |

1. Luego, seleccione **Add** (Agregar) para guardar la configuración.

## <a name="approve-private-endpoint-connection-from-the-storage-account"></a>Aprobación de la conexión del punto de conexión privado desde la cuenta de almacenamiento

1. Vaya a la cuenta de almacenamiento para la que configurará Private Link en la última sección. En **Configuración**, haga clic en **Redes**.

1. En **Redes**, seleccione **Conexiones de punto de conexión privado**. 

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/storage-account-configure-endpoint.png" alt-text="Captura de pantalla de la configuración de redes en una aplicación web.":::

1. Seleccione la solicitud de punto de conexión privado *pendiente* de Azure Front Door Prémium y elija **Aprobar**.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-pending-approval.png" alt-text="Captura de pantalla de la solicitud de punto de conexión privado de almacenamiento pendiente.":::

1. Una vez aprobada, debería ser similar a la captura de pantalla siguiente. La conexión tardará unos minutos en establecerse por completo. Ahora puede acceder a su cuenta de almacenamiento desde Azure Front Door Prémium.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-approved.png" alt-text="Captura de pantalla de la solicitud aprobada de punto de conexión de almacenamiento.":::

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el [servicio Private Link con la cuenta de almacenamiento](../../storage/common/storage-private-endpoints.md).
