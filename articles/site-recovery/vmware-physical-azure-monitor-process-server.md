---
title: Supervisión del servidor de procesos de Azure Site Recovery
description: En este artículo se describe cómo supervisar el servidor de proceso de Azure Site Recovery utilizado para la recuperación ante desastres de servidor físico/VM de VMware.
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.openlocfilehash: 0dcee641a2de73238557a6a29173f71ee1c4312a
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106580568"
---
# <a name="monitor-the-process-server"></a>Supervisión del servidor de procesos

En este artículo se describe cómo supervisar el servidor de procesos de [Site Recovery](site-recovery-overview.md).

- El servidor de procesos se usa al configurar la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos locales en Azure.
- De forma predeterminada, el servidor de procesos se ejecuta en el servidor de configuración. Se instala de forma predeterminada al implementar el servidor de configuración.
- Opcionalmente, para escalar y controlar una cantidad mayor de máquinas replicadas y volúmenes de tráfico de replicación, puede implementar servidores de procesos de escalabilidad horizontal adicionales.

[Obtenga más información](vmware-physical-azure-config-process-server-overview.md) sobre el rol y la implementación de servidores de procesos.

## <a name="monitoring-overview"></a>Información general de la supervisión

Como el servidor de procesos tiene muchas funciones, especialmente en el almacenamiento en caché, la compresión y la transferencia de datos replicados en Azure, es importante supervisar el estado del servidor de proceso de manera continua.

Hay una serie de situaciones que normalmente afectan al rendimiento del servidor de procesos. Los problemas que afectan al rendimiento tendrá un efecto en cascada en el estado de la máquina virtual y, en última instancia, llevarán al servidor de procesos y sus máquinas replicadas a un estado crítico. A continuación se describen algunas situaciones:

- Una gran cantidad de máquinas virtuales usa un servidor de procesos que se acerca o supera las limitaciones recomendadas.
- Las máquinas virtuales que usan el servidor de procesos tienen una frecuencia de renovación elevada.
- El rendimiento de la red entre las máquinas virtuales y el servidor de procesos no es suficiente para cargar datos de replicación en el servidor de procesos.
- El rendimiento de la red entre el servidor de procesos y Azure no es suficiente para cargar datos de replicación desde el servidor de procesos a Azure.

Todos estos problemas pueden afectar al objetivo de punto de recuperación (RPO) de las máquinas virtuales. 

**¿Por qué?** Porque generar un punto de recuperación para una máquina virtual requiere que todos los discos de la máquina virtual tengan un punto común. Si un disco tiene una frecuencia de renovación elevada, la replicación es lenta o el servidor de procesos no es óptimo, la eficacia con la que se crean los puntos de recuperación se ve afectada.

## <a name="monitor-proactively"></a>Supervisar de forma proactiva

Para evitar problemas con el servidor de procesos, es importante:

- comprender los requisitos específicos para los servidores de procesos mediante [recomendaciones sobre la capacidad y el tamaño](site-recovery-plan-capacity-vmware.md#capacity-considerations), y asegúrese de que los servidores de procesos se implementan y ejecutan según las recomendaciones;
- supervisar alertas y solucionar problemas a medida que se producen para que los servidores de procesos se ejecuten de forma eficaz.


## <a name="process-server-alerts"></a>Alertas del servidor de procesos

El servidor de procesos genera una serie de alertas de estado que se resumen en la siguiente tabla.

**Tipo de alerta** | **Detalles**
--- | ---
![Healthy][green] | El servidor de procesos está conectado y en buen estado.
![Advertencia][yellow] | Uso de CPU > 80 % durante los últimos 15 minutos.
![Advertencia][yellow] | Uso de memoria > 80 % durante los últimos 15 minutos.
![Advertencia][yellow] | Espacio libre de la carpeta de caché < 30 % durante los últimos 15 minutos.
![Advertencia][yellow] | Site Recovery supervisa los datos pendientes y salientes cada cinco minutos y calcula que los datos de la caché del servidor de procesos no se pueden cargar en Azure en un plazo de 30 minutos.
![Advertencia][yellow] | Los servicios del servidor de procesos no se han ejecutado durante los últimos 15 minutos.
![Crítico][red] | Uso de CPU > 95 % durante los últimos 15 minutos.
![Crítico][red] | Uso de memoria > 95 % durante los últimos 15 minutos.
![Crítico][red] | Espacio libre de la carpeta de caché < 25 % durante los últimos 15 minutos.
![Crítico][red] | Site Recovery supervisa los datos pendientes y salientes cada cinco minutos y calcula que los datos de la caché del servidor de procesos no se pueden cargar en Azure en un plazo de 45 minutos.
![Crítico][red] | No hay latido del servidor de procesos desde hace 15 minutos.

![Clave de tabla](./media/vmware-physical-azure-monitor-process-server/table-key.png)

> [!NOTE]
> El estado de mantenimiento general del servidor de procesos se basa en la peor alerta generada.



## <a name="monitor-process-server-health"></a>Supervisión del estado de los servidores de procesos

Puede supervisar el estado de los servidores de procesos de la siguiente manera: 

1. Para supervisar el estado de replicación y el estado de una máquina replicada, así como el de su servidor de procesos, en Almacén > **Elementos replicados**, haga clic en la máquina que quiera supervisar.
2. En **Mantenimiento de la replicación**, puede supervisar el estado de mantenimiento de la máquina virtual. Haga clic en el estado para explorar en profundidad los detalles del error.

    ![Estado del servidor de procesos en el panel de la máquina virtual](./media/vmware-physical-azure-monitor-process-server/vm-ps-health.png)

4. En **Estado del servidor de proceso**, puede supervisar el estado del servidor de procesos. Explore los detalles en profundidad.

    ![Detalles del servidor de procesos en el panel de la máquina virtual](./media/vmware-physical-azure-monitor-process-server/ps-summary.png)

5. El estado también se puede supervisar mediante la representación gráfica de la página de la máquina virtual.
    - Un servidor de procesos de escalabilidad horizontal se resaltará en color naranja si hay advertencias asociadas y en color rojo si tiene algún problema crítico. 
    - Si el servidor de procesos se ejecuta en la implementación predeterminada en el servidor de configuración, el servidor de configuración se resaltará según corresponda.
    - Para explorar en profundidad, haga clic en el servidor de configuración o el servidor de procesos. Tenga en cuenta cualquier problema y recomendación de corrección.

También puede supervisar los servidores de procesos del almacén en **Infraestructura de Site Recovery**. En **Administrar la infraestructura de Site Recovery**, haga clic en **Servidores de configuración**. Seleccione el servidor de configuración asociado al servidor de procesos y explore en profundidad los detalles del servidor de procesos.


## <a name="next-steps"></a>Pasos siguientes

- Si tiene algún problema con los servidores de procesos, siga nuestra [guía de solución de problemas](vmware-physical-azure-troubleshoot-process-server.md).
- Si necesita más ayuda, puede publicar su pregunta en la [Página de preguntas y respuestas de Microsoft sobre Azure Site Recovery](/answers/topics/azure-site-recovery.html). 

[green]: ./media/vmware-physical-azure-monitor-process-server/green.png
[yellow]: ./media/vmware-physical-azure-monitor-process-server/yellow.png
[red]: ./media/vmware-physical-azure-monitor-process-server/red.png
