---
title: Información general de la disponibilidad de Application Insights
description: Configure pruebas web periódicas para supervisar la disponibilidad y la capacidad de respuesta de la aplicación o el sitio web.
ms.topic: conceptual
ms.date: 05/04/2021
ms.openlocfilehash: 94fe55a5bce0e915a092fd32ad5acf6936d59e61
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111967014"
---
# <a name="availability-tests-overview"></a>Introducción a las pruebas de disponibilidad

Después de haber implementado la aplicación o sitio web, puede configurar pruebas periódicas para supervisar la disponibilidad y capacidad de respuesta. [Application Insights](./app-insights-overview.md) envía solicitudes web a la aplicación a intervalos regulares desde puntos de todo el mundo. Puede enviar una alerta si la aplicación no responde o si responde de manera demasiada lenta.

Puede configurar pruebas de disponibilidad para cualquier punto de conexión HTTP o HTTPS que sea accesible desde la red pública de Internet. No hace falta realizar cambios en el sitio web que está probando. De hecho, incluso no hace falta que sea un sitio de su propiedad. Puede probar la disponibilidad de una API de REST de la que depende su servicio.

## <a name="types-of-availability-tests"></a>Tipos de pruebas de disponibilidad

Hay tres tipos de pruebas de disponibilidad:

* [Prueba de ping de dirección URL:](monitor-web-app-availability.md) esta categoría tiene dos pruebas sencillas que puede crear a través del portal.
* [Prueba web de varios pasos](availability-multistep.md): una grabación de una secuencia de solicitudes web que se pueden reproducir para probar los escenarios más complejos. Las pruebas web de varios pasos se crean en Visual Studio Enterprise y se carga en el portal para su ejecución.
* [Pruebas de disponibilidad de seguimiento personalizado](/dotnet/api/microsoft.applicationinsights.telemetryclient.trackavailability): Si decide crear una aplicación personalizada para ejecutar pruebas de disponibilidad, puede usar el método `TrackAvailability()` para enviar los resultados a Application Insights.

> [!IMPORTANT]
> Tanto la [prueba de ping de URL](monitor-web-app-availability.md) como la [prueba web de varios pasos](availability-multistep.md) se basan en la infraestructura DNS de la Internet pública para resolver los nombres de dominio de los puntos de conexión de prueba. Esto significa que, si usa un DNS privado, debe asegurarse de que todos los nombres de dominio de la prueba también pueden resolverse mediante los servidores de nombres de dominio públicos o, cuando no sea posible, puede usar las [pruebas de disponibilidad de seguimiento personalizadas](/dotnet/api/microsoft.applicationinsights.telemetryclient.trackavailability) en su lugar.

**Puede crear hasta 100 pruebas de disponibilidad por recurso de Application Insights.**

## <a name="troubleshooting"></a>Solución de problemas

[Artículo de solución de problemas](troubleshoot-availability.md) dedicado.

## <a name="next-step"></a>Paso siguiente

* [Alertas de disponibilidad](availability-alerts.md)
* [Pruebas web de varios pasos](availability-multistep.md)
* [Pruebas de direcciones URL](monitor-web-app-availability.md)
* [Creación y ejecución de pruebas de disponibilidad personalizadas mediante Azure Functions.](availability-azure-functions.md)
* [Plantilla de Azure Resource Manager para pruebas web](/azure/templates/microsoft.insights/webtests?tabs=json)