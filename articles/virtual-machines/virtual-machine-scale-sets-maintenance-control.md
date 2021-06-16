---
title: Información general sobre el control de mantenimiento para las actualizaciones de imágenes del sistema operativo en conjuntos de escalado de máquinas virtuales de Azure
description: Aprenda a controlar cuándo se implementan las actualizaciones automáticas de las imágenes del sistema operativo en los conjuntos de escalado de máquinas virtuales de Azure mediante el control de mantenimiento.
author: ju-shim
ms.service: virtual-machine-scale-sets
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/11/2020
ms.author: jushiman
ms.openlocfilehash: c97294833ea04e4af559008cf55d3aef7230ef81
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112072242"
---
# <a name="maintenance-control-for-azure-virtual-machine-scale-sets"></a>Vista previa: Controles de seguridad para conjuntos de escalado de máquinas virtuales de Azure 

Administre [actualizaciones automáticas de imágenes del sistema operativo](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) para los conjuntos de escalado de máquinas virtuales mediante el control de mantenimiento.

El control de mantenimiento le permite decidir cuándo se aplican las actualizaciones a los discos del sistema operativo en los conjuntos de escalado de máquinas virtuales gracias a una experiencia más sencilla y predecible. 

Mantener el trabajo de las configuraciones entre suscripciones y grupos de recursos.

Todo el flujo de trabajo se reduce a estos pasos: 
- Crear una configuración de mantenimiento.
- Asociar un conjunto de escalado de máquinas virtuales a una configuración de mantenimiento.
- Habilitar las actualizaciones automáticas del sistema operativo.


## <a name="limitations"></a>Limitaciones

- Las máquinas virtuales deben estar en un conjunto de escalado.
- El usuario debe tener acceso de **colaborador del recurso**.
- La duración del mantenimiento debe ser de cinco horas o más en la configuración de mantenimiento.
- La periodicidad de mantenimiento debe establecerse en "días" en la configuración de mantenimiento


## <a name="management-options"></a>Opciones de administración

Puede crear y administrar configuraciones de mantenimiento con cualquiera de estas opciones:

- [Azure PowerShell](virtual-machine-scale-sets-maintenance-control-powershell.md)
- [CLI de Azure](virtual-machine-scale-sets-maintenance-control-cli.md)
- [Azure Portal](virtual-machine-scale-sets-maintenance-control-portal.md)


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Control de mantenimiento del conjunto de escalado de máquinas virtuales mediante PowerShell](virtual-machine-scale-sets-maintenance-control-powershell.md)
