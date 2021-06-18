---
title: Supervisión de Azure FXT Edge Filer
description: Cómo supervisar el estado del hardware de la caché de almacenamiento híbrida de Azure FXT Edge Filer
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: how-to
ms.date: 06/20/2019
ms.author: v-erkel
ms.openlocfilehash: 2ebc8ccb32034091e270bf85d7a9ac43becd0366
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111358540"
---
# <a name="monitor-azure-fxt-edge-filer-hardware-status"></a>Supervisión del estado del hardware de Azure FXT Edge Filer

El sistema de caché de almacenamiento híbrida de Azure FXT Edge Filer tiene varias luces de estado integradas en el chasis para ayudar a los administradores a entender cómo funciona el hardware.

## <a name="system-health-status"></a>Estado del sistema

Para supervisar las operaciones de caché a un nivel superior, use la página **Panel** del Panel de control del software, como se describe en la [Guía del panel del Panel de control](https://azure.github.io/Avere/legacy/dashboard/4_7/html/ops_dashboard_index.html)

## <a name="hardware-status-leds"></a>Luces LED de estado del hardware

En esta sección se explican las diversas luces de estado integradas en el hardware Azure FXT Edge Filer.

### <a name="hard-drive-status-leds"></a>Luces LED de estado de la unidad de disco duro

![imagen de la parte delantera de la unidad de disco duro, horizontal, con las etiquetas de llamada 2 (esquina superior izquierda), 1 (esquina inferior izquierda) y 3 (derecha)](media/fxt-monitor/fxt-drive-callouts.png)

Cada portadora de la unidad tiene dos LED de estado: un indicador de actividad (1) y un indicador de estado (2).

* El LED de actividad (1) se enciende cuando la unidad está en uso.
* El LED de estado (2) indica la condición de la unidad mediante los códigos de la tabla siguiente.

| Estado del LED de estado de la unidad              | Significado  |
|-------------------------------------|----------------------------------------------------------|
| Parpadea en verde dos veces por segundo      | Está identificando la unidad *o* <br> preparando la unidad para quitarla  |
| Desactivado (apagado)                         | El sistema no ha terminado de iniciarse *o* <br>la unidad está lista para quitarse |
| Parpadea en verde, en ámbar y se apaga       | Se predice un error de la unidad   |
| Parpadea en ámbar cuatro veces por segundo | Error de la unidad   |
| Luz fija verde                         | La unidad está en línea |

El lado derecho de la unidad (3) se etiqueta con la capacidad de la unidad y otra información.

Los números de unidad están impresos en el espacio entre las unidades. En Azure FXT Edge Filer, la unidad 0 es la unidad superior izquierda y la unidad 1 está directamente debajo de ella. La numeración continúa siguiendo ese modelo.

![foto de una bahía de disco duro en el chasis de FXT, que muestra los números de unidad y las etiquetas de capacidad](media/fxt-drives-photo.png)

## <a name="left-control-panel"></a>Panel de control izquierdo

El panel de control frontal izquierdo tiene varios indicadores LED de estado (1) y un indicador de estado del sistema grande encendido (2).

![panel de estado izquierdo, con los indicadores de estado etiquetados como 1 a la izquierda y la etiqueta 2 señalando la luz indicadora de estado del sistema grande de la derecha](media/fxt-monitor/fxt-control-panel-left.jpg)

### <a name="control-panel-status-indicators"></a>Indicadores de estado del panel de control

Los indicadores de estado de la izquierda muestran una luz ámbar fija si hay un error en ese sistema. En la tabla siguiente se describen posibles causas y soluciones para los errores.

Si sigue teniendo el error después de intentar estas soluciones, [póngase en contacto con el soporte técnico](fxt-support-ticket.md) para obtener ayuda.

| Icono | Descripción | Condición de error | Posibles soluciones |
|----------------|---------------|--------------------|----------------------|
| ![icono de la unidad](media/fxt-monitor/fxt-hd-icon.jpg) | Estado de la unidad | Error de la unidad | Compruebe el Registro de eventos del sistema para determinar si la unidad tiene un error, o bien <br>Ejecute la prueba de diagnóstico en línea correspondiente; reinicie el sistema y ejecute los diagnósticos insertados (ePSA), o bien <br>Si las unidades están configuradas en una matriz RAID, reinicie el sistema e introduzca el programa de utilidad de configuración del adaptador de host |
|![icono de temperatura](media/fxt-monitor/fxt-temp-icon.jpg) | Estado de la temperatura | Error térmico: por ejemplo, un ventilador ha fallado o la temperatura ambiente está fuera del intervalo | Compruebe las siguientes condiciones tratables: <br>Falta un ventilador o bien se ha estropeado <br>Se ha quitado la cobertura del sistema, la tapa del aire, la protección del módulo de memoria o el soporte de relleno trasero <br>La temperatura ambiente es demasiado alta <br>El flujo de aire externo está obstruido |
|![icono de electricidad](media/fxt-monitor/fxt-electric-icon.jpg) | Estado eléctrico | Error eléctrico: por ejemplo, el voltaje se encuentra fuera del intervalo o hay un error en la fuente de alimentación o en un regulador de voltaje |  Compruebe el registro de eventos del sistema o los mensajes del sistema para determinar el problema específico. Si hay un problema en la fuente de alimentación, compruebe el LED de estado de la fuente de alimentación y vuelva a colocarla si es necesario. |
|![icono de memoria](media/fxt-monitor/fxt-memory-icon.jpg) | Estado de la memoria | Error de memoria | Compruebe el registro de eventos del sistema o los mensajes del sistema para determinar la ubicación de la memoria que ha producido el error; vuelva a colocar el módulo de memoria. |
|![Icono de PCIe](media/fxt-monitor/fxt-pcie-icon.jpg) | Estado de PCIe | Error de la tarjeta PCIe | Reinicie el sistema; actualice los controladores de la tarjeta de PCIe; vuelva a instalar la tarjeta |

### <a name="system-health-status-indicator"></a>Indicador de estado del sistema

El botón grande iluminado a la derecha del panel de control izquierdo indica el estado general del sistema y también se usa como un indicador luminoso para localizar la unidad en el modo de identificador del sistema.

Presione el botón de identificador o estado del sistema para alternar entre el modo de identificador del sistema y el modo de estado del sistema.

|Estado del estado del sistema | Condición |
|-------------------------------------------|-----------------------------------------------|
| Azul fijo | Funcionamiento normal: el sistema está activado, funciona con normalidad y el modo de identificador del sistema no está activo. <br/>Presione el botón de identificador y estado del sistema si desea cambiar al modo de identificador del sistema. |
| Azul parpadeante | El modo de identificador del sistema está activo. Presione el botón de identificador y estado del sistema si desea cambiar al modo de estado del sistema. |
| Ámbar fijo | El sistema está en modo a prueba de errores. Si el problema persiste, [póngase en contacto con el Soporte técnico y servicio al cliente de Microsoft](fxt-support-ticket.md). |
| Ámbar parpadeante | Error del sistema. Compruebe el registro de eventos del sistema para consultar los mensajes de error específicos. Para obtener información acerca del evento y los mensajes de error generados por el firmware del sistema y los agentes que supervisan los componentes del sistema, consulte la página de búsqueda de códigos de error en qrl.dell.com. |
