---
title: Supervisión del estado de instancias de App Service
description: Obtenga información sobre cómo supervisar el estado de las instancias de App Service mediante la comprobación de estado.
keywords: Azure App Service, aplicación web, comprobación de estado, tráfico de ruta, instancias correctas, ruta de acceso, supervisión,
author: msangapu-msft
ms.topic: article
ms.date: 12/03/2020
ms.author: msangapu
ms.openlocfilehash: 3cc8ba29629c36cc9fcb295b1cdd348fbcae1584
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109732909"
---
# <a name="monitor-app-service-instances-using-health-check"></a>Supervisión de instancias de App Service mediante la comprobación de estado

![Error de la comprobación de estado][2]

En este artículo se usa la comprobación de estado en Azure Portal para supervisar las instancias de App Service. La comprobación de estado aumenta la disponibilidad de la aplicación, ya que quita las instancias incorrectas. El [plan de App Service](./overview-hosting-plans.md) debe escalarse a dos o más instancias para usar la comprobación de estado. La ruta de acceso de comprobación de estado debe comprobar los componentes críticos de la aplicación. Por ejemplo, si la aplicación depende de una base de datos y de un sistema de mensajería, el punto de conexión de comprobación de estado debe conectarse a esos componentes. Si la aplicación no se puede conectar a un componente crítico, la ruta de acceso debe devolver un código de respuesta de nivel 500 para indicar que la aplicación tiene un estado incorrecto.

## <a name="what-app-service-does-with-health-checks"></a>Lo que hace App Service con las comprobaciones de estado

- Cuando se proporciona una ruta de acceso en la aplicación, la comprobación de estado hace ping en esta ruta de acceso a todas las instancias de la aplicación App Service en intervalos de 1 minuto.
- Si una instancia no responde con un código de estado entre 200 y 299 (inclusive) después de dos o más solicitudes, o no responde al ping, el sistema determina que su estado es incorrecto y lo quita.
- Después de la eliminación, la comprobación de estado sigue haciendo ping a la instancia incorrecta. Si sigue respondiendo sin éxito, App Service reinicia la máquina virtual subyacente en un esfuerzo para devolver la instancia a un estado correcto.
- Si una instancia permanece en mal estado durante una hora, se reemplaza por una nueva.
- Además, al escalar vertical u horizontalmente, App Service hace ping en la ruta de acceso de comprobación de estado para asegurarse de que las nuevas instancias están listas.

> [!NOTE]
> La comprobación de estado no sigue 302 redirecciones. A lo sumo, se reemplaza una instancia por hora, con un máximo de tres instancias al día por plan de App Service.
>

## <a name="enable-health-check"></a>Habilitación de la comprobación de estado

![Navegación por la comprobación de estado en Azure Portal][3]

- Para habilitar la comprobación de estado, vaya a Azure Portal y seleccione la aplicación App Service.
- En **Supervisión**, seleccione **Comprobación de estado**.
- Seleccione **Habilitar** y proporcione una ruta de acceso válida a una dirección URL en la aplicación, por ejemplo, `/health` o `/api/health`.
- Haga clic en **Save**(Guardar).

> [!CAUTION]
> Los cambios de configuración de la comprobación de estado reinician la aplicación. Para minimizar el impacto en las aplicaciones de producción, se recomienda [configurar los espacios de ensayo](deploy-staging-slots.md) y cambiar a producción.
>

### <a name="configuration"></a>Configuración

Además de configurar las opciones de la comprobación de estado, también puede configurar las siguientes [opciones de la aplicación](configure-common.md):

| Nombre del valor de configuración de la aplicación | Valores permitidos | Descripción |
|-|-|-|
|`WEBSITE_HEALTHCHECK_MAXPINGFAILURES` | 2 - 10 | Número máximo de errores de ping. Por ejemplo, si se establece en `2`, las instancias se quitarán después de `2` ping con error. Además, al escalar vertical u horizontalmente, App Service hace ping en la ruta de acceso de comprobación de estado para asegurarse de que las nuevas instancias están listas. |
|`WEBSITE_HEALTHCHECK_MAXUNHEALTHYWORKERPERCENT` | 0 - 100 | Para evitar saturar las instancias correctas, no se excluirán más de la mitad de las instancias. Por ejemplo, si un plan de App Service se escala a cuatro instancias y tres son incorrectas, se excluirán dos como máximo. Las otras dos instancias (una correcta y otra incorrecta) seguirán recibiendo solicitudes. En el peor de los casos, si todas las instancias están en mal estado, no se excluye ninguna. Para invalidar este comportamiento, establezca la configuración de la aplicación en un valor entre `0` y `100`. Un valor mayor significa que se eliminarán más instancias incorrectas (el valor predeterminado es 50). |

#### <a name="authentication-and-security"></a>Autenticación y seguridad

La comprobación de estado se integra con las características de autenticación y autorización de App Service. No se requiere ninguna configuración adicional si estas características de seguridad están habilitadas. Sin embargo, si usa un sistema de autenticación propio, la ruta de acceso de la comprobación de estado debe permitir el acceso anónimo. Si el sitio tiene habilitado Solo HTTP **S**, la solicitud de comprobación de estado se enviará a través de HTTP **S**.

A menudo, los equipos de desarrollo de grandes empresas necesitan cumplir los requisitos de seguridad para las API expuestas. Para proteger el punto de conexión de la comprobación de estado, primero debe usar características como [restricciones de IP](app-service-ip-restrictions.md#set-an-ip-address-based-rule), [certificados de cliente](app-service-ip-restrictions.md#set-an-ip-address-based-rule) o una red virtual para restringir el acceso a la aplicación. Puede proteger el punto de conexión de la comprobación de estado requiriendo que `User-Agent` de la solicitud entrante coincida con `HealthCheck/1.0`. El agente de usuario no se puede suplantar porque la solicitud ya estaría protegida por las características de seguridad anteriores.

## <a name="monitoring"></a>Supervisión

Después de proporcionar la ruta de acceso de la comprobación de estado de la aplicación, puede supervisar el estado del sitio mediante Azure Monitor. En la hoja **Comprobación de estado** del portal, haga clic en **Métricas** en la barra de herramientas superior. Se abre una nueva hoja donde puede ver el estado de mantenimiento histórico del sitio y crear una regla de alerta. Para obtener más información sobre la supervisión de sitios, [vea la guía sobre Azure Monitor](web-sites-monitor.md).

## <a name="limitations"></a>Limitaciones

No se debe habilitar la comprobación de estado en los sitios de funciones prémium. Debido al rápido escalado de las funciones prémium, las solicitudes de comprobación de estado pueden provocar fluctuaciones innecesarias en el tráfico HTTP. Las funciones prémium tienen sus propios sondeos de estado internos que se usan para informar sobre las decisiones de escalado.

## <a name="next-steps"></a>Pasos siguientes
- [Creación de una alerta de registro de actividades para supervisar todas las operaciones del motor de escalado automático en su suscripción](https://github.com/Azure/azure-quickstart-templates/tree/master/demos/monitor-autoscale-alert)
- [Creación de una alerta de registro de actividades para supervisar todas las operaciones erróneas de escalado automático y reducción horizontal en su suscripción](https://github.com/Azure/azure-quickstart-templates/tree/master/demos/monitor-autoscale-failed-alert)

[1]: ./media/app-service-monitor-instances-health-check/health-check-success-diagram.png
[2]: ./media/app-service-monitor-instances-health-check/health-check-failure-diagram.png
[3]: ./media/app-service-monitor-instances-health-check/azure-portal-navigation-health-check.png
