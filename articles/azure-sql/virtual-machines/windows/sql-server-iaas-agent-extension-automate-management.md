---
title: ¿Qué es la extensión del Agente de IaaS de SQL Server?
description: En este artículo se describe cómo ayuda la extensión del Agente de IaaS de SQL Server a automatizar las tareas de administración específicas de SQL Server en las VM de Azure. Entre ellas se incluyen características como la copia de seguridad automatizada, la aplicación automatizada de revisiones, la integración de Azure Key Vault, la administración de licencias, la configuración de almacenamiento y la administración central de todas las instancias de VM con SQL Server.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: effe4e2f-35b5-490a-b5ef-b06746083da4
ms.service: virtual-machines-sql
ms.subservice: management
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/07/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: a80770013d373dbbd1ca672fbd2153731b604f5c
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112079560"
---
# <a name="automate-management-with-the-sql-server-iaas-agent-extension"></a>Automatización de la administración con la extensión del Agente de IaaS de SQL Server
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]


La extensión del Agente de IaaS de SQL Server (SqlIaasExtension) se ejecuta en SQL Server en Azure Virtual Machines (VM) para automatizar las tareas de administración. 

En este artículo se proporciona información general de la extensión. Para instalar la extensión de IaaS de SQL Server en SQL Server en las VM de Azure, consulte los artículos sobre [instalación automática](sql-agent-extension-automatic-registration-all-vms.md), [VM únicas](sql-agent-extension-manually-register-single-vm.md) o [VM en masa](sql-agent-extension-manually-register-vms-bulk.md). 

## <a name="overview"></a>Introducción

La extensión del agente de IaaS de SQL Server permite la integración en Azure Portal y, en función del modo de administración, desbloquea una serie de ventajas de las características para SQL Server en VM de Azure: 

- **Ventajas de las características**: la extensión desbloquea varias ventajas de las características de automatización, como la administración del portal, la flexibilidad de las licencia, la copia de seguridad automatizada, la aplicación automatizada de revisiones, etc. Consulte [Ventajas de las características](#feature-benefits) más adelante en este artículo para obtener más información. 

- **Cumplimiento**: la extensión ofrece un método simplificado para satisfacer el requisito de notificar a Microsoft que se ha habilitado Ventaja híbrida de Azure, según se especifica en los términos del producto. Con este proceso no es necesario administrar formularios de registro de licencias para cada recurso.  

- **Gratis**: la extensión es totalmente gratuita en los tres modos de administración. No hay costos adicionales asociados con la extensión o con el cambio de los modos de administración. 

- **Administración de licencias simplificada**: La extensión simplifica la administración de licencias de SQL Server y permite identificar rápidamente las VM con SQL Server mediante Ventaja híbrida de Azure en [Azure Portal](manage-sql-vm-portal.md), PowerShell o la CLI de Azure: 

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell-interactive
   Get-AzSqlVM | Where-Object {$_.LicenseType -eq 'AHUB'}
   ```

   # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

   ```azurecli-interactive
   $ az sql vm list --query "[?sqlServerLicenseType=='AHUB']"
   ```
   ---


> [!IMPORTANT]
> La extensión del Agente de IaaS de SQL recopila datos con el fin de ofrecer a los clientes ventajas opcionales al usar SQL Server en Azure Virtual Machines. Microsoft no usará estos datos para auditorías de licencias sin el consentimiento previo del cliente. Para obtener más información, consulte [Complemento de privacidad de SQL Server](/sql/sql-server/sql-server-privacy#non-personal-data).


## <a name="feature-benefits"></a>Ventajas de las características 

La extensión del Agente de IaaS de SQL Server desbloquea una serie de ventajas de características para administrar la VM con SQL Server. 

En la siguiente tabla se detallan estas ventajas: 


| Característica | Descripción |
| --- | --- |
| **Administración de portal** | Desbloquea la [administración en el portal](manage-sql-vm-portal.md), lo que permite ver todas las VM con SQL Server en un solo lugar, así como habilitar y deshabilitar características específicas de SQL directamente desde el portal. <br/> Modo de administración: Ligero y completo.|  
| **Copia de seguridad automatizada** |Automatiza la programación de copias de seguridad de todas las bases de datos para la instancia predeterminada o una instancia con nombre [instalada de forma correcta](/azure/azure-sql/virtual-machines/windows/frequently-asked-questions-faq#administration) de SQL Server en la máquina virtual. Para más información, consulte [Automated Backup para SQL Server en Azure Virtual Machines (Resource Manager)](automated-backup-sql-2014.md). <br/> Modo de administración: Completo|
| **Aplicación de revisión automatizada** |Configura una ventana de mantenimiento durante la cual pueden tener lugar actualizaciones de seguridad importantes de Windows y SQL Server en la VM, de forma que pueda evitar las actualizaciones durante las horas punta de la carga de trabajo. Para más información, consulte [Automated Patching para SQL Server en Azure Virtual Machines (Resource Manager)](automated-patching.md). <br/> Modo de administración: Completo|
| **Integración de Azure Key Vault** |Permite instalar y configurar automáticamente Azure Key Vault en la máquina virtual SQL Server. Para más información, consulte [Configurar la integración de Azure Key Vault para SQL Server en Azure Virtual Machines (Resource Manager)](azure-key-vault-integration-configure.md). <br/> Modo de administración: Completo|
| **Visualización del uso del disco en el portal** | Permite ver una representación gráfica del uso del disco por parte de los archivos de datos de SQL en Azure Portal.  <br/> Modo de administración: Completo | 
| **Licencias flexibles** | Ahorre en el costo mediante la [transición sin problemas](licensing-model-azure-hybrid-benefit-ahb-change.md) de la característica traiga su propia licencia (también conocida como Ventaja híbrida de Azure) al modelo de licencias de pago por uso y viceversa. <br/> Modo de administración: Ligero y completo.| 
| **Versión o edición flexibles** | Si decide cambiar la [versión ](change-sql-server-version.md) o la [edición](change-sql-server-edition.md) de SQL Server, puede actualizar los metadatos en Azure Portal sin tener que volver a implementar toda la VM con SQL Server.  <br/> Modo de administración: Ligero y completo.| 


## <a name="management-modes"></a>Modos de administración

Puede optar por registrar la extensión IaaS de SQL en tres modos de administración: 

- **Ligero**: copia los archivos binarios de la extensión en la máquina virtual, pero no instala el agente y no reinicia el servicio SQL Server. El modo ligero solo admite el cambio del tipo de licencia y la edición de SQL Server, y proporciona administración limitada del portal. Use esta opción con máquinas virtuales con SQL Server con varias instancias o con aquellas que participan en una instancia de clúster de conmutación por error (FCI). El modo ligero es el modo de administración predeterminado cuando se usa la característica [registro automático](sql-agent-extension-automatic-registration-all-vms.md), o cuando no se especifica un tipo de administración durante el registro manual. Si se usa el modo ligero, no se produce ningún impacto en la memoria ni en la CPU, ni conlleva costos asociados. Se recomienda registrar la máquina virtual con SQL Server en el modo ligero primero y, posteriormente, actualizar al modo completo durante una ventana de mantenimiento programado. 

- **Completo**: instala el Agente de IaaS de SQL en la máquina virtual para ofrecer toda la funcionalidad, pero es necesario reiniciar el servicio SQL Server y se requieren los permisos de administrador del sistema. Úselo para administrar una VM con SQL Server con una sola instancia. El modo completo instala dos servicios de Windows que tienen un impacto mínimo en la memoria y la CPU (se pueden supervisar mediante el administrador de tareas). No hay ningún costo asociado al uso del modo de administración completa. 

- El modo **NoAgent** está dedicado a SQL Server 2008 y SQL Server 2008 R2 instalados en Windows Server 2008. Si se usa el modo NoAgent, ni la memoria ni la CPU resultan afectadas. No hay ningún costo asociado al uso del modo de administración NoAgent, el servicio SQL Server no se reinicia y no se instala un agente en la máquina virtual. 

Puede ver el modo actual del Agente de IaaS de SQL Server mediante Azure PowerShell: 

  ```powershell-interactive
  # Get the SqlVirtualMachine
  $sqlvm = Get-AzSqlVM -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName
  $sqlvm.SqlManagementType
  ```


## <a name="installation"></a>Instalación

Registre la VM con SQL Server con la extensión del Agente de IaaS de SQL Server para crear el **recurso** de _máquina virtual de SQL_ dentro de la suscripción, que es un recurso _independiente_ del recurso de máquina virtual. Al anular el registro de la VM con SQL Server de la extensión, se eliminará el **recurso** de _máquina virtual de SQL_, pero no se eliminará la máquina virtual real.

La implementación de una imagen de Azure Marketplace de una VM con SQL Server mediante Azure Portal registra automáticamente dicha máquina virtual con la extensión. Sin embargo, si elige instalar automáticamente SQL Server en una máquina virtual de Azure, o aprovisionar una máquina virtual de Azure desde un disco duro virtual personalizado, debe registrar la VM con SQL Server con la extensión del Agente de IaaS de SQL para descubrir las ventajas de las características. 

Al registrar la extensión en modo ligero, se copiarán los archivos binarios, pero no se instalará el agente en la máquina virtual. El agente se instala en la máquina virtual cuando la extensión se actualiza al modo de administración completa. 

Hay tres formas de registrarse con la extensión: 
- [Automáticamente para todas las VM actuales y futuras de una suscripción](sql-agent-extension-automatic-registration-all-vms.md)
- [Manualmente para una sola máquina virtual](sql-agent-extension-manually-register-single-vm.md)
- [Manualmente para varias máquinas virtuales de forma masiva](sql-agent-extension-manually-register-vms-bulk.md)

### <a name="named-instance-support"></a>Compatibilidad de las instancias con nombre

La extensión del Agente de IaaS de SQL Server funciona con una instancia con nombre de SQL Server si esa es la única instancia de SQL Server disponible en la máquina virtual. La extensión no se puede instalar en VM que tengan varias instancias de SQL Server con nombre si no hay ninguna instancia predeterminada en la VM. 

Para usar una instancia con nombre de SQL Server, implemente una máquina virtual de Azure, instale una única instancia con nombre de SQL Server y, luego, regístrela con la [extensión del Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md).

Como alternativa, para usar una instancia con nombre con una imagen de SQL Server de Azure Marketplace, siga estos pasos: 

   1. Implemente una VM SQL Server desde Azure Marketplace. 
   1. [Anule el registro](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) de la VM con SQL Server de la extensión del Agente de IaaS de SQL. 
   1. Desinstale SQL Server por completo de la máquina virtual SQL Server.
   1. Instale SQL Server con una instancia con nombre en la máquina virtual SQL Server. 
   1. [Registre la máquina virtual con la extensión del Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md#register-with-extension). 

## <a name="verify-status-of-extension"></a>Comprobación del estado de la extensión

Use Azure Portal o Azure PowerShell para comprobar el estado de la extensión. 

### <a name="azure-portal"></a>Azure portal

Compruebe que la extensión esté instalada en Azure Portal. 

Seleccione **Todas las configuraciones** en el panel de la máquina virtual y seleccione **Extensiones**. Debería aparecer la extensión **SqlIaasExtension**.

![Estado de la extensión del Agente de IaaS de SQL Server en Azure Portal](./media/sql-server-iaas-agent-extension-automate-management/azure-rm-sql-server-iaas-agent-portal.png)


### <a name="azure-powershell"></a>Azure PowerShell

También puede usar el cmdlet de Azure PowerShell **Get-AzVMSqlServerExtension**:

   ```powershell-interactive
   Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
   ```

El comando anterior confirma que el agente está instalado y proporciona información de estado general. Puede obtener información de estado específica sobre Automated Backup y Automated Patching con los siguientes comandos:

   ```powershell-interactive
    $sqlext = Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
    $sqlext.AutoPatchingSettings
    $sqlext.AutoBackupSettings
   ```


## <a name="limitations"></a>Limitaciones

La extensión del Agente de IaaS de SQL solo admite: 

- Máquinas virtuales con SQL Server implementadas mediante Azure Resource Manager. No se admiten las máquinas virtuales con SQL Server implementadas con el modelo clásico. 
- Las máquinas virtuales SQL Server implementadas en la nube pública o Azure Government. No se admiten las implementaciones en otras nubes públicas o privadas. 


## <a name="in-region-data-residency"></a>Residencia de datos en la región
La máquina virtual de Azure SQL y la extensión del agente de IaaS de SQL no mueven ni almacenan datos de clientes fuera de la región en la que se implementan.

## <a name="next-steps"></a>Pasos siguientes

Para instalar la extensión de IaaS de SQL Server en SQL Server en las VM de Azure, consulte los artículos sobre [instalación automática](sql-agent-extension-automatic-registration-all-vms.md), [VM únicas](sql-agent-extension-manually-register-single-vm.md) o [VM en masa](sql-agent-extension-manually-register-vms-bulk.md).

Para obtener más información sobre cómo ejecutar SQL Server en Azure Virtual Machines, consulte [¿Qué es SQL Server en Azure Virtual Machines?](sql-server-on-azure-vm-iaas-what-is-overview.md).

Para más información, consulte las [preguntas más frecuentes](frequently-asked-questions-faq.yml). 
