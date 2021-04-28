---
title: Autodiagnóstico de la red virtual de Azure Spring Cloud
description: Aprenda a autodiagnosticar y solucionar los problemas de ejecución de Azure Spring Cloud en la red virtual.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 01/25/2021
ms.custom: devx-track-java
ms.openlocfilehash: f2b2dcc7e22ce904035a6805d78e59ad14f9bff1
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108134796"
---
# <a name="self-diagnose-running-azure-spring-cloud-in-vnet"></a>Autodiagnóstico de la ejecución de Azure Spring Cloud en la red virtual
El diagnóstico de Azure Spring Cloud admite aplicaciones de solución de problemas interactivas que se ejecutan en redes virtuales sin configuración. El diagnóstico de Azure Spring Cloud los identifica y le guía por la información que ayuda a solucionarlos.

## <a name="navigate-to-the-diagnostics-page"></a>Vaya a la página de diagnósticos.
El procedimiento siguiente inicia el diagnóstico de aplicaciones en red.
1. Inicie sesión en Azure Portal.
1. Vaya a la página Información general de Azure Spring Cloud.
1. Seleccione **Diagnosticar y solucionar problemas** en el panel de navegación izquierdo.
1. Seleccione la tercera categoría, **Redes**.

   ![Título de autodiagnóstico](media/spring-cloud-self-diagnose-vnet/self-diagostic-title.png)

## <a name="view-a-diagnostic-report"></a>Visualización de un informe de diagnóstico
Después de hacer clic en la categoría **Redes**, puede ver dos problemas relacionados con la red específicos de la red virtual insertada en Azure Spring Cloud: **DNS Resolution** (Resolución de DNS) y **Required Outbound Traffic** (Tráfico saliente necesario).

   ![Opciones de autodiagnóstico](media/spring-cloud-self-diagnose-vnet/self-diagostic-dns-req-outbound-options.png)

Seleccione el problema de destino para ver el informe de diagnóstico. Se muestra un resumen de los diagnósticos, como: 

* *Se ha quitado el recurso*.
* *El recurso no está implementado en su propia red virtual*.

Algunos resultados contienen documentación relacionada. Las distintas subredes mostrarán los resultados por separado.

## <a name="dns-resolution"></a>Resolución DNS 
Si selecciona **DNS Resolution** (Resolución de DNS), los resultados indicarán si hay problemas de DNS con las aplicaciones.  Las aplicaciones en buen estado se enumeran de la manera siguiente:

* *Problemas de DNS resueltos sin problemas en la subred "subnet01"* .
* *Problemas de DNS resueltos sin problemas en la subred "subnet01"* .

El siguiente ejemplo de informe de diagnóstico indica que el estado de la aplicación es desconocido. El período de tiempo del informe no incluye la hora a la que se ha notificado el estado de mantenimiento.  Supongamos que la hora de finalización del contexto es *2021-03-03T04:20:00Z*. La marca de tiempo más reciente de las **representaciones de tabla de resolución de DNS** es *2021-03-03T03:39:00Z*, el día anterior. Es posible que no se haya enviado el registro de comprobación de estado debido a una red bloqueada. 

Los resultados de estado de mantenimiento desconocido contienen documentación relacionada.  Puede hacer clic en el corchete angular de apertura para ver la pantalla desplegable.

   ![DNS desconocido](media/spring-cloud-self-diagnose-vnet/self-diagostic-dns-unknown.png)

Si ha configurado incorrectamente el conjunto de registros de la zona DNS privada, obtendrá un resultado crítico, como: `Failed to resolve the Private DNS in subnet xxx`. 

En la lista desplegable **DNS Resolution Table Renderings** (Representaciones de tablas de resolución de DNS) encontrará información detallada de los mensajes en la que puede comprobar la configuración.

## <a name="required-outbound-traffic"></a>Tráfico saliente necesario 

Si selecciona **Required Outbound Traffic** (Tráfico saliente necesario), los resultados indicarán si hay problemas de tráfico saliente con las aplicaciones.  Las aplicaciones en buen estado se enumeran de la manera siguiente:

* *Tráfico saliente necesario resuelto sin problemas en la subred "subnet01".
* *Tráfico saliente necesario resuelto sin problemas en la subred "subnet02".

Si una subred está bloqueada por las reglas de firewall o del grupo de seguridad de red, y si no ha bloqueado el registro, encontrará los siguientes errores. Puede comprobar si se ha pasado por alto alguna [responsabilidad del cliente](./vnet-customer-responsibilities.md).
    
   ![Error del punto de conexión](media/spring-cloud-self-diagnose-vnet/self-diagostic-endpoint-failed.png)

Si no hay datos en `Required Outbound Traffic Table Renderings` al cabo de 30 minutos, el resultado será `health status unknown`. Es posible que la red esté bloqueada o que el servicio de registro esté inactivo.

   ![Punto de conexión de diagnóstico desconocido](media/spring-cloud-self-diagnose-vnet/self-diagostic-endpoint-unknown.png)

## <a name="see-also"></a>Consulte también
* [Autodiagnóstico de Azure Spring Cloud](./how-to-self-diagnose-solve.md)