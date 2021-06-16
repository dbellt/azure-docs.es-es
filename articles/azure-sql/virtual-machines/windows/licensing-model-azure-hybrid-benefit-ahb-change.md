---
title: Cambio del modelo de licencia de una VM de SQL en Azure
description: Aprenda a cambiar de licencia en una VM con SQL Server en Azure de la modalidad de "pago por uso" a "traiga su propia licencia" con la Ventaja híbrida de Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.subservice: management
ms.workload: iaas-sql-server
ms.date: 11/13/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8eca44fa7e5a23b38c8cfa7368aace1ea0a942eb
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080064"
---
# <a name="change-the-license-model-for-a-sql-virtual-machine-in-azure"></a>Cambio del modelo de licencia de una máquina virtual de SQL en Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]


En este artículo se describe cómo cambiar el modelo de licencia de una máquina virtual con SQL Server en Azure mediante la [extensión Agente de IaaS de SQL](./sql-server-iaas-agent-extension-automate-management.md).

## <a name="overview"></a>Información general

Hay tres modelos de licencia para una VM de Azure que hospeda SQL Server: pago por uso, Ventaja híbrida de Azure (AHB) y alta disponibilidad/recuperación ante desastres (HA/DR). Puede modificar el modelo de licencia de la VM con SQL Server mediante Azure Portal, la CLI de Azure o PowerShell. 

- El modelo de **pago por uso** significa que el costo por segundo de ejecutar la máquina virtual de Azure incluye el costo de la licencia de SQL Server.
- La [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) le permite usar su propia licencia de SQL Server con una máquina virtual que ejecuta SQL Server. 
- El tipo de licencia de **alta disponibilidad/recuperación ante desastres** se usa para la [réplica de alta disponibilidad/recuperación ante desastres gratuita](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure) en Azure. 

La Ventaja híbrida de Azure permite el uso de licencias de SQL Server con Software Assurance ("Licencia calificada") en Azure Virtual Machines. Con la Ventaja híbrida de Azure, los clientes no pagan por el uso de una licencia de SQL Server en una máquina virtual. Sin embargo, deben pagar por el costo del proceso en la nube subyacente (es decir, la tasa base), el almacenamiento y las copias de seguridad. También deben pagar por la E/S asociada a su uso de los servicios (según corresponda).

Según los [términos de los productos](https://www.microsoft.com/licensing/terms/productoffering/MicrosoftAzureServices/EAEAS) de Microsoft: "Los clientes deben indicar que usan Azure SQL Database (Instancia administrada, Grupo elástico y Base de datos única), Azure Data Factory, SQL Server Integration Services o Máquinas virtuales de SQL Server en la Ventaja híbrida de Azure para SQL Server al configurar cargas de trabajo en Azure".

Para indicar el uso de la Ventaja híbrida de Azure para SQL Server en una VM de Azure y cumplir los requisitos, dispone de tres opciones:

- Aprovisionar una máquina virtual mediante una imagen de SQL Server de "traiga su propia licencia" desde Azure Marketplace. Esta opción solo está disponible para los clientes con un Contrato Enterprise.
- Aprovisionar una máquina virtual con una imagen de SQL Server de "pago por uso" desde Azure Marketplace y activar la Ventaja híbrida de Azure
- Instalar automáticamente SQL Server en una VM de Azure, [realizar el registro con la extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) manualmente y activar la Ventaja híbrida de Azure.

El tipo de licencia de SQL Server se puede configurar cuando se aprovisiona la VM o en cualquier momento posterior. El cambio entre los modelos de licencia no genera ningún tiempo de inactividad, no reinicia la máquina virtual ni el servicio SQL Server, no agrega ningún costo adicional y es efectivo inmediatamente. De hecho, activar la Ventaja híbrida de Azure *reduce* el costo.

## <a name="prerequisites"></a>Prerrequisitos

El cambio del modelo de licencia de la máquina virtual con SQL Server tiene los siguientes requisitos: 

- Una [suscripción de Azure](https://azure.microsoft.com/free/).
- Una [VM con SQL Server](./create-sql-vm-portal.md) registrada con la [extensión Agente de IaaS de SQL](./sql-server-iaas-agent-extension-automate-management.md).
- [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) es un requisito para usar [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/). 


## <a name="change-license-model"></a>Cambio del modelo de licencia de una máquina virtual de SQL en Azure

# <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

El modelo de licencia se puede modificar directamente desde el portal: 

1. Abra [Azure Portal](https://portal.azure.com) y el [recurso de las máquinas virtuales de SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) para su VM con SQL Server. 
1. Seleccione **Configurar** en **Configuración**. 
1. Seleccione la opción **Ventaja híbrida de Azure** y active la casilla para confirmar que tiene una licencia de SQL Server con Software Assurance. 
1. Seleccione **Aplicar** en la parte inferior de la página **Configurar**. 

![Ventaja híbrida de Azure en el portal](./media/licensing-model-azure-hybrid-benefit-ahb-change/ahb-in-portal.png)


# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede usar la CLI de Azure para cambiar el modelo de licencia.  

Especifique los siguientes valores para el **tipo de licencia**:
- `AHUB` para la Ventaja híbrida de Azure
- `PAYG` para el pago por uso
- `DR` para activar la réplica de alta disponibilidad/recuperación ante desastres


```azurecli-interactive
# example: az sql vm update -n AHBTest -g AHBTest --license-type AHUB

az sql vm update -n <VMName> -g <ResourceGroupName> --license-type <license-type>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Puede usar PowerShell para cambiar el modelo de licencia.

Especifique los siguientes valores para el **tipo de licencia**:
- `AHUB` para la Ventaja híbrida de Azure
- `PAYG` para el pago por uso
- `DR` para activar la réplica de alta disponibilidad/recuperación ante desastres


```powershell-interactive
Update-AzSqlVM -ResourceGroupName <resource_group_name> -Name <VM_name> -LicenseType <license-type>
```

---

## <a name="remarks"></a>Observaciones

- Para que los clientes del Proveedor de soluciones en la nube de Azure (CSP) puedan usar la Ventaja híbrida de Azure, deben implementar primero una máquina virtual de "pago por uso" y, después, convertirla al modo "traiga su propia licencia" si tienen una instancia activa de Software Assurance.
- Si quita el recurso de máquina virtual con SQL, volverá a la configuración de licencia codificada de forma rígida de la imagen. 
- La posibilidad de cambiar el modelo de licencia es una característica de la extensión Agente de IaaS de SQL. Al implementar una imagen de Azure Marketplace desde Azure Portal, se registra automáticamente una VM con SQL Server con la extensión. Sin embargo, los clientes que instalen automáticamente SQL Server deberán [registrar su VM con SQL Server](sql-agent-extension-manually-register-single-vm.md) de forma manual. 
- Para agregar una VM con SQL Server a un conjunto de disponibilidad, es necesario volver a crear la máquina virtual. Por tanto, todas las máquinas virtuales que se agreguen a un conjunto de disponibilidad volverán a tener el tipo de licencia predeterminado de pago por uso. También será necesario habilitar de nuevo la Ventaja híbrida de Azure. 


## <a name="limitations"></a>Limitaciones

El cambio del modelo de licencia:
   - Solo está disponible para los clientes que cuentan con [Software Assurance](https://www.microsoft.com/en-us/licensing/licensing-programs/software-assurance-overview).
   - Solo se admite en las ediciones Standard Edition y Enterprise Edition de SQL Server. No se admiten los cambios de licencia para Express, Web ni Developer. 
   - Solo se admite en máquinas virtuales implementadas mediante el modelo de Azure Resource Manager. No se admiten las máquinas virtuales implementadas con el modelo clásico. 
   - Solo está disponible para las nubes públicas o Azure Government. 

> [!Note]
> Únicamente las licencias basadas en núcleo de SQL Server con Software Assurance o las licencias de suscripción son válidas para la Ventaja híbrida de Azure. Si usa una licencia Server + CAL para SQL Server y tiene Software Assurance, puede usar el modelo "traiga su propia licencia" para una imagen de máquina virtual de Azure SQL Server a fin de aprovechar la movilidad de licencias para estos servidores, pero no puede aprovechar las otras características de la Ventaja híbrida de Azure. 

## <a name="known-errors"></a>Errores conocidos

Revise los siguientes errores conocidos y sus soluciones. 

**No se encontró el recurso "Microsoft.SqlVirtualMachine/SqlVirtualMachines/\<resource-group>" en el grupo de recursos "\<resource-group>".**

Este error se produce al intentar cambiar el modelo de licencia en una VM con SQL Server que no se ha registrado con la extensión Agente de IaaS de SQL:

`The Resource 'Microsoft.SqlVirtualMachine/SqlVirtualMachines/\<resource-group>' under resource group '\<resource-group>' was not found. The property 'sqlServerLicenseType' cannot be found on this object. Verify that the property exists and can be set.`

Tendrá que registrar la suscripción con el proveedor de recursos y, después, [registrar la VM con SQL Server con la extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md). 



## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Preguntas más frecuentes de SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.yml)
* [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md)
* [Notas de la versión de SQL Server en Azure Virtual Machines](../../database/doc-changes-updates-release-notes.md)
* [Información general de la extensión Agente de IaaS de SQL](./sql-server-iaas-agent-extension-automate-management.md)
