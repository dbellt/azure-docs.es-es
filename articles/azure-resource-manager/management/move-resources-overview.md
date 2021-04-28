---
title: Traslado de recursos de Azure entre grupos de recursos, suscripciones y regiones
description: Introducción a los tipos de recursos de Azure que se pueden mover entre grupos de recursos, suscripciones o regiones.
ms.topic: conceptual
ms.date: 04/16/2021
ms.openlocfilehash: 646c0892ac3609fc531e91f4bfb9c1de6a61bc38
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107730394"
---
# <a name="move-azure-resources-across-resource-groups-subscriptions-or-regions"></a>Traslado de recursos de Azure entre grupos de recursos, suscripciones y regiones

Los recursos de Azure se pueden mover a un nuevo grupo de recursos o suscripción, o entre regiones.

## <a name="move-resources-across-resource-groups-or-subscriptions"></a>Traslado de recursos entre grupos de recursos o suscripciones

Puede mover recursos de Azure a otra suscripción de Azure o a otro grupo de recursos de la misma suscripción. Puede usar Azure Portal, Azure PowerShell, la CLI de Azure o la API REST para trasladar recursos. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).

### <a name="upgrade-a-subscription"></a>Actualización de una suscripción

Si quiere actualizar la suscripción de Azure (por ejemplo, para cambiar de gratuita a de pago por uso) deberá convertir su suscripción.

- Para actualizar a una evaluación gratuita, consulte [Actualización de la suscripción de Microsoft Imagine Azure o la prueba gratuita al plan de pago por uso](../../cost-management-billing/manage/upgrade-azure-subscription.md).
- Para cambiar a una cuenta de pago por uso, consulte [Cambie la suscripción de pago por uso de Azure a otra oferta](../../cost-management-billing/manage/switch-azure-offer.md).

Si no puede convertir la suscripción, [cree una solicitud de soporte técnico de Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Seleccione **Administración de suscripciones** para el tipo de problema.

## <a name="move-resources-across-regions"></a>Traslado de recursos entre regiones

Las geografías, las regiones y las zonas de disponibilidad de Azure conforman la base de la infraestructura global de Azure. Las [geografías](https://azure.microsoft.com/global-infrastructure/geographies/) de Azure suelen contener dos o más [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/). Una región es un área dentro de una geografía que contiene zonas de disponibilidad y varios centros de datos.

Después de implementar recursos en una región específica de Azure, hay muchos motivos por los que podría querer mover recursos a una región diferente.

- **Alineación con el lanzamiento de una región**: mueva los recursos a una región de Azure que acaba de incorporar y que no estaba disponible previamente.
- **Alineación con los servicios o las características**: mueva recursos para aprovechar las ventajas de los servicios o las características que están disponibles en una región específica.
- **Respuesta a desarrollos empresariales**: mueva recursos a una región en respuesta a cambios empresariales, como fusiones o adquisiciones.
- **Alineación para proximidad**: mueva recursos a una región que sea local para la empresa.
- **Cumplimiento de requisitos de los datos**: traslade los recursos para la alineación con los requisitos de residencia de los datos o las necesidades de clasificación de los datos. [Más información](https://azure.microsoft.com/mediahandler/files/resourcefiles/achieving-compliant-data-residency-and-security-with-azure/Achieving_Compliant_Data_Residency_and_Security_with_Azure.pdf).
- **Respuesta a requisitos de implementación**: mueva recursos que se implementaron por error, o como respuesta a necesidades de capacidad.
- **Respuesta a una retirada**: mueva recursos debido a la retirada de regiones.

### <a name="move-resources-with-resource-mover"></a>Traslado de recursos con Resource Mover

Puede trasladar recursos a una región diferente con [Azure Resource Mover](../../resource-mover/overview.md). Resource Mover proporciona:

- Un solo centro para trasladar recursos entre regiones.
- Menores tiempo de traslado y complejidad. Todo lo que necesita está en una única ubicación.
- Una experiencia sencilla y coherente para trasladar distintos tipos de recursos de Azure.
- Una manera fácil de identificar las dependencias entre los recursos que desee trasladar. Esta identificación le ayuda a mover los recursos relacionados de forma conjunta, de modo que todo funcione según lo previsto en la región de destino después del traslado.
- La limpieza automática de los recursos de la región de origen, si desea eliminarlos después del traslado.
- Pruebas. Puede probar un traslado y, a continuación, descartarlo si no quiere completarlo.

Puede trasladar recursos a otra región con un par de métodos diferentes:

- **Comience a mover recursos de un grupo de recursos**: Con este método, se comienza el traslado de región dentro de un grupo de recursos. Después de seleccionar los recursos que quiere trasladar, el proceso continúa en el centro de Resource Mover para comprobar las dependencias de los recursos y coordinar el proceso de traslado. [Más información](../../resource-mover/move-region-within-resource-group.md).
- **Comience a trasladar recursos directamente desde el centro de Resource Mover**: Con este método, se empieza el proceso de traslado de la región directamente en el centro. [Más información](../../resource-mover/tutorial-move-region-virtual-machines.md).

## <a name="next-steps"></a>Pasos siguientes

- Para comprobar si un tipo de recurso admite el traslado, consulte [Compatibilidad con la operación de traslado para recursos](move-support-resources.md).
- Para obtener más información sobre el proceso de traslado entre regiones, consulte [Acerca del proceso de traslado](../../resource-mover/about-move-process.md).
