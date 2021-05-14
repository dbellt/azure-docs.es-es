---
title: Conexión de Azure Front Door Prémium a un origen de equilibrador de carga interno con Private Link
titleSuffix: Azure Private Link
description: Aprenda a conectar Azure Front Door Prémium a un equilibrador de carga interno.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 03/16/2021
ms.author: duau
ms.openlocfilehash: 80013ca5d4d20366672eda417862ff7bb1309b0c
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633166"
---
# <a name="connect-azure-front-door-premium-to-an-internal-load-balancer-origin-with-private-link"></a>Conexión de Azure Front Door Prémium a un origen de equilibrador de carga interno con Private Link

Este artículo le guía por los pasos para configurar la SKU prémium de Azure Front Door a fin de conectarse al origen de equilibrador de carga interno mediante el servicio Azure Private Link.

## <a name="prerequisites"></a>Requisitos previos

Cree [un servicio Private Link](../../private-link/create-private-link-service-portal.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-private-link-to-an-internal-load-balancer"></a>Habilitación de Private Link para un equilibrador de carga interno
 
En esta sección, asignará el servicio Private Link a un punto de conexión privado creado en una red privada de Azure Front Door. 

1. En el perfil de Azure Front Door Prémium, en *Settings* (Configurción), seleccione **Origin groups** (Grupos de origen).

1. Seleccione el grupo de origen en el que quiere habilitar Private Link para el equilibrador de carga interno.

1. Seleccione **+ Add an origin** (+ Agregar un origen) para agregar un origen de equilibrador de carga interno.

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/private-endpoint-internal-load-balancer.png" alt-text="Captura de pantalla que muestra cómo habilitar Private Link para un equilibrador de carga interno.":::

1. En **Select an Azure resource** (Seleccione un recurso de Azure), elija **In my directory** (En mi directorio). Seleccione o escriba los siguientes valores para configurar el sitio al que quiere que Azure Front Door Prémium se conecte de forma privada.

    | Configuración | Value |
    | ------- | ----- |
    | Region | Seleccione la región que sea la misma o la más cercana a su origen. |
    | Tipo de recurso | Seleccione **Microsoft.Network/privateLinkServices**. |
    | Resource | Seleccione la instancia de Private link relacionada con el equilibrador de carga interno. |
    | Target sub resource (Subrecurso de destino) | déjelo en blanco. |
    | Mensaje de solicitud | Personalice el mensaje o elija el predeterminado. |

1. Después, seleccione **Agregar** y, seguidamente, **Actualizar** para guardar la configuración.

## <a name="approve-private-endpoint-connection-from-the-storage-account"></a>Aprobación de la conexión del punto de conexión privado desde la cuenta de almacenamiento

1. Vaya a Private Link Center y seleccione **Servicios Private Link**. A continuación, seleccione el nombre del vínculo privado.

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/list.png" alt-text="Captura de pantalla de la lista de vínculos privados.":::

1. En **Configuración**, seleccione *Conexiones de punto de conexión privado*.

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/overview.png" alt-text="Captura de pantalla de la página Información general del vínculo privado.":::

1. Seleccione la solicitud de punto de conexión privado *pendiente* de Azure Front Door Prémium y elija **Aprobar**.

    :::image type="content" source="../media/how-to-enable-private-link-internal-load-balancer/private-endpoint-pending-approval.png" alt-text="Captura de pantalla de aprobación pendiente del vínculo privado.":::

1. Una vez aprobada, debería ser similar a la captura de pantalla siguiente. La conexión tardará unos minutos en establecerse por completo. Ahora puede acceder al equilibrador de carga interno desde Azure Front Door Prémium.

    :::image type="content" source="../media/how-to-enable-private-link-storage-account/private-endpoint-approved.png" alt-text="Captura de pantalla de la solicitud de vínculo privado aprobada.":::

## <a name="next-steps"></a>Pasos siguientes

Aprenda sobre el [servicio Private Link](../../private-link/private-link-service-overview.md).
