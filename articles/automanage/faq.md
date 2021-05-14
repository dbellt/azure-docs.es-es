---
title: Preguntas más frecuentes sobre Azure Automanage para máquinas virtuales
description: Respuestas a las preguntas más frecuentes sobre Azure Automanage para máquinas virtuales.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: troubleshooting
ms.date: 02/22/2021
ms.author: deanwe
ms.openlocfilehash: f5a9ff7661fda372631d1bb912b1c137b37c7e07
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363366"
---
# <a name="frequently-asked-questions-for-azure-automanage-for-vms"></a>Preguntas más frecuentes sobre Azure Automanage para máquinas virtuales

En este artículo se proporcionan respuestas a algunas de las preguntas más comunes sobre [Azure Automanage para máquinas virtuales](automanage-virtual-machines.md).

Si su problema con Azure no se trata en este artículo, visite los foros de Azure en [MSDN y Stack Overflow](https://azure.microsoft.com/support/forums/). Puede publicar su problema en ellos o [@AzureSupport en Twitter](https://twitter.com/AzureSupport). También puede enviar una solicitud de soporte técnico de Azure. Para enviar una solicitud de soporte técnico, en la [página de Soporte técnico de Azure](https://azure.microsoft.com/support/options/), seleccione **Obtener soporte técnico**.


## <a name="azure-automanage-for-virtual-machines"></a>Azure Automanage para máquinas virtuales

**¿Qué requisitos previos se necesitan para habilitar Azure Automanage?**

Los requisitos previos para habilitar Azure Automanage son los siguientes:
- [Versiones de Windows Server](automanage-windows-server.md#supported-windows-server-versions) y [distribuciones de Linux](automanage-linux.md#supported-linux-distributions-and-versions) compatibles
- Las máquinas virtuales deben estar en una región admitida.
- El usuario debe tener los permisos correctos.
- Solo máquinas virtuales que no son de conjunto de escalado.
- Automanage no admite suscripciones de espacio aislado en este momento

**¿Qué permiso de Azure RBAC es necesario para habilitar Automanage?**

Si va a habilitar Automanage en una máquina virtual con una cuenta de Automanage existente, necesita el rol de colaborador en el grupo de recursos donde reside la máquina virtual.

Si usa una cuenta de Automanage nueva al habilitarlo, debe tener el rol de propietario o de administrador de acceso de usuario y colaborador para la suscripción.


**¿Qué regiones se admiten?**

[Aquí](./automanage-virtual-machines.md#supported-regions) se encuentra la lista completa de regiones admitidas.


**¿Qué funcionalidades automatiza Azure Automanage?**

Automanage inscribe, configura y supervisa los servicios mencionados [aquí](automanage-virtual-machines.md) durante todo el ciclo de vida de la máquina virtual.

**¿Funciona Azure Automanage con máquinas virtuales habilitadas para Azure Arc?**

Actualmente Automanage no admite máquinas virtuales habilitadas para Arc.

**¿Se pueden personalizar configuraciones en Azure Automanage?**

Los clientes pueden personalizar la configuración de servicios específicos, como la retención de Azure Backup, a través de las preferencias de configuración. Consulte la documentación que aparece [aquí](automanage-virtual-machines.md#customizing-an-environment-using-preferences) para ver la lista completa de las configuraciones que se pueden modificar.


**¿Azure Automanage funciona con máquinas virtuales Linux y Windows?**

Sí, consulte las [versiones de Windows Server](automanage-windows-server.md#supported-windows-server-versions) y [distribuciones de Linux](automanage-linux.md#supported-linux-distributions-and-versions) compatibles.


**¿Se puede aplicar Automanage de manera selectiva a un solo conjunto de máquinas virtuales?**

Automanage se puede habilitar simplemente haciendo clic y señalando máquinas virtuales nuevas y existentes. Automanage también se pueden deshabilitar en cualquier momento.


**¿Azure Automanage admite máquinas virtuales en un conjunto de escalado de máquinas virtuales?**

No, actualmente Azure Automanage no admite máquinas virtuales en un conjunto de escalado de máquinas virtuales.


**¿Cuánto cuesta Azure Automanage?**

Azure Automanage está disponible sin costo adicional en la versión preliminar pública. Los recursos adjuntos de Azure, como Azure Backup, supondrán un costo.


**¿Se puede aplicar Automanage a través de Azure Policy?**

Sí, tenemos una directiva integrada que aplicará automáticamente Automanage a todas las máquinas virtuales dentro del ámbito que se defina. También se especificará el entorno de configuración (Desarrollo/pruebas o Producción) junto con su cuenta de Automanage. Obtenga más información [aquí](virtual-machines-policy-enable.md) sobre cómo habilitar Automanage a través de Azure Policy.


**¿Qué es una cuenta de Automanage?**

Una cuenta de Automanage es una identidad de servicio administrado (MSI) que proporciona el contexto de seguridad o la identidad bajo la cual se producen las operaciones automatizadas.


**Al habilitar Automanage, ¿afecta a las máquinas virtuales adicionales además de a las máquinas virtuales seleccionadas?**

Si la máquina virtual está vinculada a un área de trabajo de Log Analytics existente, se volverá a usar esa área de trabajo para aplicar estas soluciones: Change Tracking, Inventory y Update Management. Todas las máquinas virtuales conectadas a esa área de trabajo tendrán habilitadas esas soluciones.


**¿Puedo cambiar el entorno de mi máquina virtual?**

En este momento, tendrá que deshabilitar Automanage en esa máquina virtual y habilitarlo de nuevo con el entorno y las preferencias que quiera.


**Si una máquina virtual ya está configurada para un servicio, como Update Management, ¿se volverá a configurar con Automanage?**
No, no se volverá a configurar con Automanage. Se iniciará la supervisión del desfase de los recursos asociados a ese servicio.


**¿Por qué una máquina virtual tiene un estado de error en el portal de Automanage?**

Si ve el estado *Error*, puede solucionar problemas de la implementación de varias maneras diferentes:
* Vaya a **Grupos de recursos**, seleccione el grupo de recursos, haga clic en **Implementaciones** y consulte el estado *Erróneo* que aparece ahí junto con los detalles del error.
* Vaya a **Suscripciones**, seleccione el grupo de recursos, haga clic en **Implementaciones** y vea ahí el estado *Error* con sus detalles.
* También puede visitar el registro de actividad de una máquina virtual, que contendrá una entrada para "crear o actualizar asignaciones de perfil de configuración". Esto también puede contener más detalles sobre la implementación.

**¿Cómo se puede obtener ayuda para solucionar problemas de Automanage?**

Se puede presentar una [incidencia de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). En la opción **Servicio**, busque y seleccione *Automanage* en la sección *Supervisión y administración*.


## <a name="next-steps"></a>Pasos siguientes

Intente habilitar Automanage para máquinas virtuales en Azure Portal.

> [!div class="nextstepaction"]
> [Habilitación de Automanage para máquinas virtuales en Azure Portal](quick-create-virtual-machines-portal.md)