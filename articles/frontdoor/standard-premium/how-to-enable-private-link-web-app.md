---
title: Conexión de Azure Front Door Premium a un origen de aplicación web con Private Link
titleSuffix: Azure Private Link
description: Obtenga información sobre cómo conectar Azure Front Door Premium a una aplicación web de forma privada.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 02/18/2021
ms.author: duau
ms.openlocfilehash: d30b73d1e80273d766f4f94224798ddb04763195
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633257"
---
# <a name="connect-azure-front-door-premium-to-a-web-app-origin-with-private-link"></a>Conexión de Azure Front Door Premium a un origen de aplicación web con Private Link

Este artículo le guiará a través de la configuración del SKU de Azure Front Door Premium para conectarse a la aplicación web de forma privada mediante el servicio Azure Private Link.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Cree un servicio [Private Link](../../private-link/create-private-link-service-portal.md) para los servidores web de origen.

> [!Note]
> El punto de conexión privado está disponible en regiones públicas para las aplicaciones web de Windows de nivel PremiumV2, PremiumV3, las aplicaciones web de Linux y el plan de Azure Functions Premium (a veces llamado plan Elástico Premium).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-private-link-to-a-web-app-in-azure-front-door-premium"></a>Habilitación de Private Link para una aplicación web en Azure Front Door Premium
 
En esta sección, asignará el servicio Private Link a un punto de conexión privado creado en una red privada de Azure Front Door. 

1. En su perfil de Azure Front Door Premium, en *Configuración*, seleccione **Grupos de origen**.

1. Seleccione el grupo de origen que contiene el origen de la aplicación web para la que quiere habilitar Private Link.

1. Seleccione **+ Add an origin** (Agregar un origen) para agregar un nuevo origen de aplicación web o seleccione un origen de aplicación web creado anteriormente en la lista.

    :::image type="content" source="../media/how-to-enable-private-link-web-app/private-endpoint-web-app.png" alt-text="Captura de pantalla de la habilitación de Private Link a una aplicación web.":::

1. En **Seleccione un recurso de Azure**, seleccione **In my directory** (En mi directorio). Seleccione o escriba los siguientes valores para configurar el sitio al que quiere que Azure Front Door Prémium se conecte de forma privada.

    | Configuración | Value |
    | ------- | ----- |
    | Region | Seleccione la región que sea la misma o la más cercana a su origen. |
    | Tipo de recurso | Seleccione **Microsoft.Web/sites**. |
    | Resource | Seleccione **myPrivateLinkService**. |
    | Subrecurso de destino | sites |
    | Mensaje de solicitud | Personalice el mensaje o elija el predeterminado. |

1. Después, seleccione **Agregar** para guardar la configuración.

## <a name="approve-azure-front-door-premium-private-endpoint-connection-from-web-app"></a>Aprobación de la conexión de punto de conexión privado de Azure Front Door Premium desde la aplicación web

1. Vaya a la aplicación web para la que configura Private Link en la última sección. Seleccione **Redes** en **Configuración**.

1. En **Redes**, seleccione **Configurar las conexiones de punto de conexión privado**.

    :::image type="content" source="../media/how-to-enable-private-link-web-app/web-app-configure-endpoint.png" alt-text="Captura de pantalla de la configuración de redes en una aplicación web.":::

1. Seleccione la solicitud de punto de conexión privado *pendiente* de Azure Front Door Premium y seleccione **Aprobar**.

    :::image type="content" source="../media/how-to-enable-private-link-web-app/private-endpoint-pending-approval.png" alt-text="Captura de pantalla de la solicitud de punto de conexión privado pendiente.":::

1. Una vez aprobada, debería ser similar a la captura de pantalla siguiente. La conexión tardará unos minutos en establecerse por completo. Ahora puede acceder a la aplicación web desde Azure Front Door Premium. El acceso directo a la aplicación web desde la red pública de Internet se deshabilita una vez habilitado el punto de conexión privado.

    :::image type="content" source="../media/how-to-enable-private-link-web-app/private-endpoint-approved.png" alt-text="Captura de pantalla de la solicitud aprobada de punto de conexión.":::

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el [Servicio Private Link con Azure Web App](../../app-service/networking/private-endpoint.md).
