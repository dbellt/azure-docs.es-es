---
title: Solicitud de acceso Just-In-Time
description: Describe cómo los editores de Azure Managed Applications solicitan acceso Just-In-Time a una aplicación administrada.
author: MSEvanhi
ms.topic: conceptual
ms.date: 09/25/2020
ms.author: evanhi
ms.openlocfilehash: ae7c4e61107518676b2cc8308904816d075ecf7e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111951486"
---
# <a name="enable-and-request-just-in-time-access-for-azure-managed-applications"></a>Habilitación y solicitud de acceso Just-In-Time para Azure Managed Applications

Los consumidores de la aplicación administrada pueden ser reacios a concederle acceso permanente al grupo de recursos administrados. Como editor de una aplicación administrada, puede establecer que los consumidores sepan exactamente cuando necesita acceder a los recursos administrados. Para ofrecer a los consumidores mayor control sobre la concesión de acceso a los recursos administrados, Azure Managed Applications proporciona una característica denominada acceso Just-In-Time (JIT). Esta funcionalidad actualmente está en su versión preliminar.

El acceso JIT permite solicitar acceso con privilegios elevados a recursos de la aplicación administrada con fines de solución de problemas o mantenimiento. Siempre tendrá acceso de solo lectura a los recursos, pero puede tener mayor acceso durante un período de tiempo específico.

El flujo de trabajo de concesión de acceso es el siguiente:

1. Agrega una aplicación administrada en Marketplace y especifica que el acceso JIT esté disponible.

1. Durante la implementación, el consumidor permite el acceso JIT a esa instancia de la aplicación administrada.

1. Después de la implementación, el consumidor puede cambiar la configuración del acceso JIT.

1. Envía una solicitud de acceso cuando se necesita para solucionar problemas o actualizar los recursos administrados.

1. El consumidor aprueba la solicitud.

Este artículo se centra en las acciones que los editores llevan a cabo para habilitar el acceso JIT y enviar solicitudes. Para obtener información acerca de la aprobación de solicitudes de acceso JIT, consulte el tema sobre la [aprobación del acceso Just-In-Time en Azure Managed Applications](approve-just-in-time-access.md).

## <a name="add-jit-access-step-to-ui"></a>Agregar el paso de acceso JIT a la interfaz de usuario

En el archivo CreateUiDefinition.json, incluya un paso que permita a los consumidores habilitar el acceso JIT. Para admitir la funcionalidad JIT para la oferta, agregue el siguiente contenido al archivo CreateUiDefinition.json.

En "steps":

```json
{
    "name": "jitConfiguration",
    "label": "JIT Configuration",
    "subLabel": {
        "preValidation": "Configure JIT settings for your application",
        "postValidation": "Done"
    },
    "bladeTitle": "JIT Configuration",
    "elements": [
        {
          "name": "jitConfigurationControl",
          "type": "Microsoft.Solutions.JitConfigurator",
          "label": "JIT Configuration"
        }
    ]
}
```

En "outputs":

```json
"jitAccessPolicy": "[steps('jitConfiguration').jitConfigurationControl]"
```

> [!NOTE]
> El acceso JIT está en versión preliminar. El esquema de configuración de JIT podría cambiar en futuras iteraciones.

## <a name="enable-jit-access"></a>Habilitar el acceso JIT

Al crear la oferta en el Centro de partners, asegúrese de habilitar el acceso JIT.

1. Inicie sesión en el portal de Marketplace comercial del [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/overview).

1. Para obtener instrucciones sobre cómo crear una nueva aplicación administrada, siga los pasos descritos en [Creación de una oferta de aplicación de Azure](../../marketplace/azure-app-offer-setup.md).

1. En la página **Configuración técnica**, seleccione la casilla **Habilitar acceso Just-In-Time (JIT)** .

   :::image type="content" source="./media/request-just-in-time-access/enable-just-in-time-access.png" alt-text="Habilitar acceso Just-In-Time":::

Ha agregado un paso de configuración JIT a la interfaz de usuario y habilitado el acceso JIT en la oferta de Marketplace comercial. Cuando los consumidores implementen la aplicación administrada, pueden [activar el acceso JIT para su instancia](approve-just-in-time-access.md#enable-during-deployment).

## <a name="request-access"></a>Solicitar acceso

Cuando necesite acceder a los recursos administrados del consumidor, envíe una solicitud para un rol, una hora y una duración específicos. A continuación, el consumidor debe aprobar la solicitud.

Para enviar una solicitud de acceso JIT:

1. Seleccione **Acceso de JIT** para la aplicación administrada a la que necesita acceder.

1. Seleccione **Roles elegibles** y, a continuación, **Activar** en la columna ACCIÓN del rol deseado.

   ![Activar la solicitud de acceso](./media/request-just-in-time-access/send-request.png)

1. En el formulario **Activar rol**, seleccione una hora de inicio y la duración que el rol deberá estar activo. Seleccione **Activar** para enviar la solicitud.

   ![Activar el acceso](./media/request-just-in-time-access/activate-access.png) 

1. Consulte las notificaciones para ver que la nueva solicitud de JIT se ha enviado correctamente al consumidor.

   ![Notification](./media/request-just-in-time-access/in-progress.png)

   Ahora, debe esperar a que el consumidor [apruebe su solicitud](approve-just-in-time-access.md#approve-requests).

1. Para ver el estado de todas las solicitudes JIT de una aplicación administrada, seleccione **Acceso de JIT** e **Historial de solicitudes**.

   ![Ver estado](./media/request-just-in-time-access/view-status.png)

## <a name="known-issues"></a>Problemas conocidos

El identificador de entidad de seguridad de la cuenta que solicita acceso JIT debe incluirse explícitamente en la definición de la aplicación administrada. La cuenta no puede incluirse solo a través de un grupo especificado en el paquete. Esta limitación se corregirá en una versión futura.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información acerca de la aprobación de solicitudes de acceso JIT, consulte el tema sobre la [aprobación del acceso Just-In-Time en Azure Managed Applications](approve-just-in-time-access.md).