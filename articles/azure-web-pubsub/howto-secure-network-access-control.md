---
title: Protección y control del acceso de red al punto de conexión del servicio Azure Web PubSub
description: Información general sobre cómo controlar el acceso de red del servicio Azure Web PubSub
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: how-to
ms.date: 03/25/2021
ms.openlocfilehash: 912de328332023e0e868a7ab0ac6088a2daeb33e
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167137"
---
# <a name="configure-network-access-control-for-azure-web-pubsub-service"></a>Configuración del control del acceso de red para el servicio Azure Web PubSub

El servicio Azure Web PubSub le permite proteger y controlar el nivel de acceso al punto de conexión de su servicio en función del tipo de solicitud y del subconjunto de redes usadas. Cuando se configuran las reglas de red, solo las aplicaciones que solicitan datos del conjunto especificado de redes pueden acceder a su servicio Azure Web PubSub.

El servicio Azure Web PubSub tiene un punto de conexión público accesible a través de Internet. También puede crear [puntos de conexión privados para Azure Web PubSub](howto-secure-private-endpoints.md). El punto de conexión privado asigna una dirección IP privada de su red virtual al servicio Azure Web PubSub, y protege todo el tráfico entre la red virtual y el servicio Azure Web PubSub a través de un vínculo privado. El control de acceso de red del servicio Azure Web PubSub permite controlar el acceso para los puntos de conexión públicos y privados.

Opcionalmente, puede permitir o denegar determinados tipos de solicitudes para los puntos de conexión públicos y para cada punto de conexión privado. 

Una aplicación que accede a un servicio Azure Web PubSub cuando las reglas de control de acceso de red están en vigor todavía requiere una autorización adecuada para la solicitud.

## <a name="scenario-a---no-public-traffic"></a>Escenario A: sin tráfico público

Para denegar completamente todo el tráfico público, primero debe configurar la regla de red pública para que no permita ningún tipo de solicitud. A continuación, debe configurar las reglas que conceden acceso al tráfico desde redes virtuales específicas. Esta configuración le permite crear un límite de red seguro para las aplicaciones.

## <a name="scenario-b---only-client-connections-from-public-network"></a>Escenario B: solo conexiones de cliente desde la red pública

En este escenario, puede configurar la regla de red pública para permitir solo conexiones de cliente desde esta red. Después, puede configurar reglas de red privada para permitir otros tipos de solicitudes originadas en una red virtual específica. Esta configuración oculta sus servidores de aplicaciones de la red pública y establece conexiones seguras entre los servidores de aplicaciones y el servicio Azure Web PubSub.

## <a name="managing-network-access-control"></a>Administración del control de acceso de red

Puede administrar el control de acceso de red para el servicio Azure Web PubSub a través de Azure Portal.

### <a name="azure-portal"></a>Portal de Azure

1. Vaya al servicio Azure Web PubSub que quiere proteger.

1. Seleccione el menú de configuración denominado **Control de acceso de red**.

    :::image type="content" source="./media/howto-secure-network-access-control/portal-network-access-control.png" alt-text="Control de acceso de red en Azure Portal.":::

1. Para editar la acción predeterminada, alterne el botón **Permitir/Denegar**.

    > [!TIP]
    > La acción predeterminada es la acción que se realiza cuando no hay ninguna coincidencia de regla de ACL. Por ejemplo, si la acción predeterminada es **Denegar**, se denegarán los tipos de solicitud que no se aprueben explícitamente.

1. Para editar la regla de red pública, seleccione los tipos de solicitudes permitidos en **Red pública**.

    :::image type="content" source="./media/howto-secure-network-access-control/portal-public-network.png" alt-text="Edición de la ACL de redes públicas en Azure Portal.":::

1. Para editar reglas de red de puntos de conexión privados, seleccione los tipos de solicitudes permitidos en cada una de las filas que aparecen en **Conexiones de punto de conexión privado**.

1. Seleccione **Guardar** para aplicar los cambios.
