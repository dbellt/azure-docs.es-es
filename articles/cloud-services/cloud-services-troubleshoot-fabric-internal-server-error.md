---
title: Solución de problemas de FabricInternalServerError o ServiceAllocationFailure al implementar un servicio en la nube (clásico) en Azure | Microsoft Docs
description: En este artículo se muestra cómo resolver una excepción FabricInternalServerError o ServiceAllocationFailure al implementar una servicio en la nube (clásico) en Azure.
services: cloud-services
author: mamccrea
ms.author: mamccrea
ms.service: cloud-services
ms.topic: troubleshooting
ms.date: 02/22/2021
ms.openlocfilehash: 0883178779179df2e531123b8a500c62d42530e4
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107877451"
---
# <a name="troubleshoot-fabricinternalservererror-or-serviceallocationfailure-when-deploying-a-cloud-service-classic-to-azure"></a>Solución de problemas de FabricInternalServerError o ServiceAllocationFailure al implementar un servicio en la nube (clásico) en Azure

En este artículo, solucionará los errores de asignación en los que el controlador de tejido no puede realizar la asignación al implementar un servicio en la nube de Azure (clásico).

Al implementar instancias en un servicio en la nube o agregar nuevas instancias de rol de trabajo o web, Microsoft Azure asigna recursos de proceso.

En ocasiones, es posible que reciba errores durante estas operaciones, incluso antes de alcanzar el límite de la suscripción de Azure.

> [!TIP]
> La información también puede ser útil si tiene pensado realizar la implementación de sus servicios.

## <a name="symptom"></a>Síntoma

En Azure Portal, vaya a su instancia de Cloud Services (clásico) y, en la barra lateral, seleccione *Registro de operaciones (clásico)* para ver los registros.

![En la imagen se muestra la hoja del registro de operaciones (clásico).](./media/cloud-services-troubleshoot-fabric-internal-server-error/cloud-services-troubleshoot-allocation-logs.png)

Al inspeccionar los registros de la instancia de Cloud Services (clásico), verá la siguiente excepción:

|Excepción  |Mensaje de error  |
|---------|---------|
FabricInternalServerError     |Error en la operación con el código "InternalError" y el mensaje de error "El servidor ha detectado un error interno. Vuelva a intentar realizar la solicitud".|
|ServiceAllocationFailure     |Error en la operación con el código "InternalError" y el mensaje de error "El servidor ha detectado un error interno. Vuelva a intentar realizar la solicitud".|

## <a name="cause"></a>Causa

*FabricInternalServerError* y *ServiceAllocationFailure* son excepciones que se pueden producir cuando el controlador de tejido no puede asignar instancias en el clúster. La causa principal varía si el servicio en la nube está **anclado** o **no anclado**.

- [**No anclado:** Errores de una primera implementación de un nuevo servicio en la nube](#not-pinned-to-a-cluster)
- [**Anclado:** Errores de servicios en la nube existentes](#pinned-to-a-cluster)

> [!NOTE]
> Cuando se implementa la primera instancia en un servicio en la nube (ya sea en fase de almacenamiento provisional o producción), el servicio en la nube se ancla a un clúster.
>
> Con el tiempo, los recursos de este grupo de recursos se pueden aprovechar por completo. Si un servicio en la nube realiza una solicitud de asignación de recursos adicionales cuando no hay suficientes recursos disponibles en el grupo de recursos anclado, la solicitud producirá un [error de asignación](cloud-services-allocation-failures.md).

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
