---
title: Alta disponibilidad y recuperación ante desastres
titleSuffix: Azure Digital Twins
description: Describe las características de Azure y Azure Digital Twins que lo ayudarán a crear soluciones de Azure IoT de alta disponibilidad con funcionalidades de recuperación ante desastres.
author: baanders
ms.author: baanders
ms.date: 10/14/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 05713c5a27a8b42ce9b6967212bf1414c841788c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110474583"
---
# <a name="azure-digital-twins-high-availability-and-disaster-recovery"></a>Alta disponibilidad y recuperación ante desastres de Azure Digital Twins

Un área clave que hay que considerar para tener soluciones resistentes de IoT es la continuidad empresarial y recuperación ante desastres. El diseño de **alta disponibilidad (HA)** y **recuperación ante desastres (DR)** puede ayudarlo a definir y alcanzar los objetivos de tiempo de actividad adecuados para su solución.

En este artículo se describen las características de alta disponibilidad y recuperación ante desastres que ofrece específicamente el servicio Azure Digital Twins.

Azure Digital Twins admite estas opciones de características:
* *Alta disponibilidad dentro de una región*: redundancia integrada para cumplir con el tiempo de actividad del servicio.
* *Recuperación ante desastres entre regiones*: conmutación por error en una región de Azure emparejada geográficamente en caso de un error inesperado en el centro de datos.

También puede consultar la sección [Procedimientos recomendados](#best-practices) para ver instrucciones generales de Azure sobre el diseño de alta disponibilidad y recuperación ante desastres.

## <a name="intra-region-ha"></a>Alta disponibilidad dentro de una región
 
Azure Digital Twins ofrece alta disponibilidad dentro de una región mediante la implementación de redundancias en el servicio. Esto se refleja en el [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/digital-twins) para el tiempo de actividad. **No es necesario que los desarrolladores de una solución de Azure Digital Twins hagan ningún trabajo adicional para poder beneficiarse de estas características de alta disponibilidad.** Aunque Azure Digital Twins ofrece una garantía de tiempo de actividad razonablemente elevada, todavía es posible que ocurran errores transitorios, al igual que sucede con cualquier plataforma de informática distribuida. Se deben integrar directivas de reintento adecuadas en los componentes que interactúan con una aplicación en la nube para solucionar errores transitorios.

## <a name="cross-region-dr"></a>Recuperación ante desastres entre regiones

Podría haber algunas situaciones excepcionales cuando un centro de datos sufre interrupciones prolongadas debido a problemas con el suministro eléctrico u otros eventos en la región. Estos eventos son poco frecuentes, durante los cuales es posible que no sea útil la funcionalidad de alta disponibilidad dentro de una región que se describió anteriormente. Azure Digital Twins aborda esta situación con la conmutación por error iniciada por Microsoft.

La **conmutación por error iniciada por Microsoft** se lleva a cabo en situaciones excepcionales para conmutar por error todas las instancias de Azure Digital Twins desde una región afectada hasta la [región emparejada geográficamente](../best-practices-availability-paired-regions.md) correspondiente. Este proceso es una opción predeterminada (los usuarios no pueden omitirla) y no requiere ninguna intervención por parte del usuario. Microsoft se reserva el derecho a tomar una determinación sobre cuándo ejecutar esta opción. Este mecanismo no precisa de consentimiento del usuario antes de realizar la conmutación por error de la instancia del usuario.

>[!NOTE]
> Algunos servicios de Azure también proporcionan una opción adicional llamada **conmutación por error iniciada por el cliente**, que permite que los clientes inicien una conmutación por error solo para esa instancia, como la ejecución de un simulacro de recuperación ante desastres. Actualmente, Azure Digital Twins **no admite** este mecanismo. 

Si es importante que se mantengan todos los datos dentro de determinadas áreas geográficas, compruebe la ubicación de la [región emparejada geográficamente](../best-practices-availability-paired-regions.md#azure-regional-pairs) de la región en la que va a crear la instancia, para asegurarse de que cumpla los requisitos de residencia de datos.

>[!NOTE]
> Algunos servicios de Azure proporcionan una opción para que los usuarios configuren otra región para la conmutación por error, como una manera de cumplir los requisitos de residencia de datos. Actualmente, Azure Digital Twins **no admite** esta funcionalidad. 

## <a name="monitor-service-health"></a>Supervisar el estado del servicio

Como las instancias de Azure Digital Twins se conmutan por error y se recuperan, puede supervisar el proceso con la herramienta [Azure Service Health](../service-health/service-health-overview.md). Service Health realiza un seguimiento del estado de los servicios de Azure en diferentes regiones y suscripciones, y comparte las comunicaciones que afectan al servicio sobre las interrupciones y los tiempos de inactividad.

Durante un evento de conmutación por error, Service Health puede proporcionar una indicación de si el servicio está inactivo y de cuándo se realiza la copia de seguridad.

Para ver los eventos de Service Health...
1. Vaya a la página de [Service Health](https://portal.azure.com/?feature.customportal=false#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues) en Azure Portal (puede usar este vínculo o buscarlo en la barra de búsqueda del portal).
1. Use el menú de la izquierda para cambiar a la página del *Historial de Health*.
1. Busque un *nombre de problema* a partir de **Azure Digital Twins** y selecciónelo.

    :::image type="content" source="media/concepts-high-availability-disaster-recovery/navigate.png" alt-text="Captura de pantalla de Azure Portal donde se muestra la página &quot;Historial de estado&quot;. Se ha resaltado un problema llamado &quot;Azure Digital Twins: Oeste de Europa; mitigado&quot;." lightbox="media/concepts-high-availability-disaster-recovery/navigate.png":::

1. Para obtener información general sobre la interrupción, consulte la pestaña *Resumen*.

    :::image type="content" source="media/concepts-high-availability-disaster-recovery/summary.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la página &quot;Historial de estado&quot; con la pestaña &quot;Resumen&quot; resaltada. La pestaña muestra información general." lightbox="media/concepts-high-availability-disaster-recovery/summary.png":::
1. Para obtener más información y actualizaciones sobre el problema a lo largo del tiempo, consulte la pestaña *Issue updates* (Actualizaciones de problemas).

    :::image type="content" source="media/concepts-high-availability-disaster-recovery/issue-updates.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra la página &quot;Historial de estado&quot; con la pestaña &quot;Actualizaciones de problemas&quot; resaltada. La pestaña muestra el estado de las entradas." lightbox="media/concepts-high-availability-disaster-recovery/issue-updates.png":::


Tenga en cuenta que la información que se muestra en esta herramienta no es específica de una instancia de Azure Digital. Después de usar Service Health para comprender lo que está ocurriendo con el servicio Azure Digital Twins en una determinada región o suscripción, puede avanzar el proceso de supervisión un paso más mediante el uso de la [herramienta Resource Health](troubleshoot-resource-health.md) para profundizar en instancias específicas y ver si están afectadas.

## <a name="best-practices"></a>Procedimientos recomendados

Para conocer los procedimientos recomendados de alta disponibilidad y recuperación ante desastres, consulte estas instrucciones de Azure sobre el tema: 
* En el artículo de [orientación técnica de la continuidad empresarial de Azure](/azure/architecture/framework/resiliency/overview) se describe un marco general que ayuda a reflexionar sobre la continuidad empresarial y recuperación ante desastres. 
* El documento [Recuperación ante desastres y alta disponibilidad para aplicaciones de Azure](/azure/architecture/framework/resiliency/backup-and-recovery) proporciona una guía de arquitectura enfocada a estrategias para que las aplicaciones de Azure logren alta disponibilidad y recuperación ante desastres.

## <a name="next-steps"></a>Pasos siguientes 

Obtenga más información sobre cómo empezar a utilizar las soluciones de Azure Digital Twins:
 
* [¿Qué es Azure Digital Twins?](overview.md)
* [Inicio rápido: Exploración de un escenario de Azure Digital Twins de ejemplo mediante Azure Digital Twins Explorer](quickstart-azure-digital-twins-explorer.md)
