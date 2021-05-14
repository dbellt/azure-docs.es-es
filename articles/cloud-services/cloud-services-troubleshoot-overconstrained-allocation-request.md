---
title: Solución de problemas de OverconstrainedAllocationRequest al implementar una instancia de Cloud Services (clásico) en Azure | Microsoft Docs
description: En este artículo se muestra cómo resolver una excepción OverconstrainedAllocationRequest al implementar una instancia de Cloud Services (clásico) en Azure.
services: cloud-services
documentationcenter: ''
author: mamccrea
ms.author: mamccrea
ms.service: cloud-services
ms.topic: troubleshooting
ms.date: 02/22/2021
ms.openlocfilehash: 1a5880107aaa414da42fe5e36e0cb3315071d8a0
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107877433"
---
# <a name="troubleshoot-overconstrainedallocationrequest-when-deploying-cloud-services-classic-to-azure"></a>Solución de problemas de OverconstrainedAllocationRequest al implementar una instancia de Cloud Services (clásico) en Azure

En este artículo, solucionará los errores de asignación restringida que impiden la implementación de Azure Cloud Services (clásico).

Al implementar instancias en un servicio en la nube o agregar nuevas instancias de rol de trabajo o web, Microsoft Azure asigna recursos de proceso.

En ocasiones, es posible que reciba errores durante estas operaciones, incluso antes de alcanzar el límite de la suscripción de Azure.

> [!TIP]
> La información también puede ser útil si tiene pensado realizar la implementación de sus servicios.

## <a name="symptom"></a>Síntoma

![En la imagen se muestra la hoja del registro de operaciones (clásico).](./media/cloud-services-troubleshoot-overconstrained-allocation-failed/cloud-services-troubleshoot-allocation-logs.png)

|Tipo de excepción  |Mensaje de error  |
|---------|---------|
|OverconstrainedAllocationRequest |No se puede aprovisionar el tamaño de la máquina virtual (o la combinación de tamaños de la máquina virtual) que se necesita en esta implementación debido a restricciones en las solicitudes de implementación. Si es posible, intente relajar las restricciones como los enlaces de red virtual, intente realizar la implementación en un servicio hospedado que no tenga ninguna otra implementación y en otro grupo de afinidad o sin grupo de afinidad, o bien intente realizar la implementación en otra región.|

## <a name="cause"></a>Causa

La causa principal varía si el servicio en la nube está **anclado** o **no anclado**.

- [**No anclado:** Errores de una primera implementación de un nuevo servicio en la nube (clásico)](#not-pinned-to-a-cluster)
- [**Anclado:** Errores de un servicio en la nube existente (clásico)](#pinned-to-a-cluster)

> [!NOTE]
> Cuando se implementa la primera instancia en un servicio en la nube (ya sea en fase de almacenamiento provisional o producción), el servicio en la nube se ancla a un clúster.
>
> Con el tiempo, los recursos del clúster se pueden aprovechar por completo. Si Cloud Services (clásico) realiza una solicitud de asignación de más recursos cuando no hay suficientes recursos disponibles en el clúster anclado, la solicitud genera un [error de asignación](cloud-services-allocation-failures.md).

## <a name="solution"></a>Solución

Siga las instrucciones para los errores de asignación en los escenarios siguientes.

### <a name="not-pinned-to-a-cluster"></a>No anclado a un clúster

La primera vez que se implementa un servicio en la nube (clásico), aún no se ha seleccionado el clúster, por lo que el servicio en la nube no está *anclado*. Azure puede tener un error de implementación porque:

- Ha seleccionado un tamaño determinado que no está disponible en la región.
- La combinación de tamaños que se necesitan en diferentes roles no está disponible en la región.

Cuando se produce un error de asignación en este escenario, el curso de acción recomendado es comprobar los tamaños disponibles en la región y cambiar el tamaño que se especificó anteriormente.

1. Puede comprobar los tamaños disponibles en una región en la página de [productos del servicio en la nube (clásico)](https://azure.microsoft.com/global-infrastructure/services/?products=cloud-services).

    > [!NOTE]
    > En la página de *productos* no se muestra la capacidad disponible. Para cualquier asignación nueva, Azure debe ser capaz de elegir el clúster óptimo de su región en ese momento dado.

1. Actualice el archivo de definición de servicio para el servicio en la nube (clásico) con objeto de especificar otro [tamaño de producto](cloud-services-sizes-specs.md#configure-sizes-for-cloud-services) de su región.

### <a name="pinned-to-a-cluster"></a>Anclado a un clúster

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
