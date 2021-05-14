---
title: Solución de problemas de ConstrainedAllocationFailed al implementar una instancia de Cloud Services (clásico) en Azure | Microsoft Docs
description: En este artículo se muestra cómo resolver una excepción ConstrainedAllocationFailed al implementar una instancia de Cloud Services (clásico) en Azure.
services: cloud-services
author: mamccrea
ms.author: mamccrea
ms.service: cloud-services
ms.topic: troubleshooting
ms.date: 02/22/2021
ms.openlocfilehash: 4a92246f81ddd10840bb0dcf7edf2b01853fd148
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107876605"
---
# <a name="troubleshoot-constrainedallocationfailed-when-deploying-a-cloud-service-classic-to-azure"></a>Solución de problemas de ConstrainedAllocationFailed al implementar una instancia de Cloud Services (clásico) en Azure

En este artículo, solucionará los errores de asignación por los que Azure Cloud Services (clásico) no puede realizar la implementación debido a las restricciones de asignación.

Al implementar instancias en Cloud Services (clásico) o agregar nuevas instancias de rol de trabajo o web, Microsoft Azure asigna recursos de proceso.

En ocasiones, es posible que reciba errores durante estas operaciones, incluso antes de alcanzar el límite de la suscripción de Azure.

> [!TIP]
> La información también puede ser útil si tiene pensado realizar la implementación de sus servicios.

## <a name="symptom"></a>Síntoma

En Azure Portal, vaya a su instancia de Cloud Services (clásico) y, en la barra lateral, seleccione *Registro de operaciones (clásico)* para ver los registros.

![En la imagen se muestra la hoja del registro de operaciones (clásico).](./media/cloud-services-troubleshoot-constrained-allocation-failed/cloud-services-troubleshoot-allocation-logs.png)

Al inspeccionar los registros de la instancia de Cloud Services (clásico), verá la siguiente excepción:

|Tipo de excepción  |Mensaje de error  |
|---------|---------|
|ConstrainedAllocationFailed     |Error "`{Operation ID}`" en operaciones de Azure con el código Compute.ConstrainedAllocationFailed. Detalles: Error en la asignación; no se pudieron satisfacer las restricciones de la solicitud. La nueva implementación del servicio solicitada está enlazada a un grupo de afinidad, su destino es una instancia de Virtual Network o hay una implementación existente bajo este servicio hospedado. Todas estas condiciones restringen la nueva implementación a recursos específicos de Azure. Inténtelo de nuevo más tarde o pruebe a reducir el tamaño de la máquina virtual o el número de instancias de rol. También puede quitar, si es posible, las restricciones mencionadas o intentar realizar la implementación en otra región.|

## <a name="cause"></a>Causa

Cuando se implementa la primera instancia en un servicio en la nube (ya sea en fase de almacenamiento provisional o producción), el servicio en la nube se ancla a un clúster.

Con el tiempo, los recursos de este clúster se pueden aprovechar por completo. Si Cloud Services (clásico) realiza una solicitud de asignación de más recursos cuando no hay suficientes recursos disponibles en el clúster anclado, la solicitud genera un error de asignación. Para obtener más información, consulte los [problemas comunes de error de asignación](cloud-services-allocation-failures.md#common-issues).

## <a name="solution"></a>Solución

Los servicios en la nube existentes están *anclados* a un clúster. Todas las implementaciones posteriores para Cloud Services (clásico) tendrán lugar en el mismo clúster.

Cuando se produce un error de asignación en este escenario, el curso de acción recomendado es volver a implementarlo en una nueva instancia de Cloud Services (clásico) (y actualizar *CNAME*).

> [!TIP]
> Es probable que esta solución sea la más correcta, ya que permite a la plataforma elegir entre todos los clústeres de esa región.

> [!NOTE]
> Esta solución debe incurrir en tiempo de inactividad cero.

1. Implementar la carga de trabajo en una nueva instancia de Cloud Services (clásico)
    - Consulte la guía [Creación e implementación de un servicio en la nube de Azure (clásico)](cloud-services-how-to-create-deploy-portal.md) para obtener más instrucciones.

    > [!WARNING]
    > Si no quiere perder la dirección IP asociada con esta ranura de implementación, puede usar la [solución 3: conservar la dirección IP](cloud-services-allocation-failures.md#solutions).

1. Actualice *CNAME* o el registro *A* para que apunte el tráfico a la nueva instancia de Cloud Services (clásico).
    - Consulte la guía [Configuración de un nombre de dominio personalizado para Azure Cloud Services (clásico)](cloud-services-custom-domain-name-portal.md#understand-cname-and-a-records) para obtener más instrucciones.

1. Una vez que ya no se dirija el tráfico al sitio antiguo, puede eliminar la instancia anterior de Cloud Services (clásico).
    - Consulte la guía [Eliminación de implementaciones y un servicio en la nube (clásico)](cloud-services-how-to-manage-portal.md#delete-deployments-and-a-cloud-service) para obtener más instrucciones.
    - Para ver el tráfico de red en la instancia de Cloud Services (clásico), consulte [Introducción a la supervisión de servicios en la nube (clásico)](cloud-services-how-to-monitor.md).

Consulte [Solución de errores de asignación de Cloud Services (clásico) | Microsoft Docs](cloud-services-allocation-failures.md#common-issues) para obtener más pasos de corrección.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el contexto y las soluciones de errores de asignación:

> [!div class="nextstepaction"]
> [Errores de asignación: Cloud Services (clásico)](cloud-services-allocation-failures.md)

Si su problema con Azure no se trata en este artículo, visite los foros de Azure en [MSDN y Stack Overflow](https://azure.microsoft.com/support/forums/). Puede publicar su problema en ellos o [@AzureSupport en Twitter](https://twitter.com/AzureSupport). También puede enviar una solicitud de soporte técnico de Azure. Para enviar una solicitud de soporte técnico, en la página de [soporte técnico de Azure](https://azure.microsoft.com/support/options/), seleccione *Obtener soporte técnico*.