---
title: Plantillas del Administrador de recursos de Azure
description: Plantillas de Resource Manager para su uso con los almacenes de Recovery Services y las características de Azure Backup
ms.topic: sample
ms.date: 01/31/2019
ms.custom: mvc
ms.openlocfilehash: f9db005cda1b63682f4e074818935e687a7f5636
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109737733"
---
# <a name="azure-resource-manager-templates-for-azure-backup"></a>Plantillas de Azure Resource Manager para Azure Backup

En la tabla siguiente se incluyen vínculos a las plantillas de Resource Manager para su uso con los almacenes de Recovery Services y las características de Azure Backup. Para obtener información sobre la sintaxis y las propiedades de JSON, consulte [Tipos de recursos Microsoft.RecoveryServices](/azure/templates/microsoft.recoveryservices/allversions).

| Plantilla | Descripción |
|---|---|
|**Almacén de Recovery Services** | |
| [Creación de un almacén de Recovery Services](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.recoveryservices/recovery-services-vault-create)| Cree un almacén de Recovery Services. El almacén se puede usar con Azure Backup y Azure Site Recovery. |
|**Copia de seguridad de máquinas virtuales**| |
| [Copia de seguridad de máquinas virtuales de Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-backup-vms) | Use el almacén de Recovery Services existente y la directiva de Backup para hacer una copia de seguridad de las máquinas virtuales de Resource Manager en el mismo grupo de recursos.|
| [Copia de seguridad de máquinas virtuales de IaaS en el almacén de Recovery Services](https://github.com/Azure/azure-quickstart-templates/tree/master/201-recovery-services-backup-classic-resource-manager-vms) | Plantilla para realizar la copia de seguridad de máquinas virtuales de los modelos clásico y de Resource Manager. |
| [Creación de una directiva de copia de seguridad semanal para máquinas virtuales de IaaS](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.recoveryservices/recovery-services-weekly-backup-policy-create) | La plantilla crea un almacén de Recovery Services y una directiva de copia de seguridad semanal que se usa para realizar la copia de seguridad de máquinas virtuales de los modelos clásico y de Resource Manager.|
| [Creación de una directiva de copia de seguridad diaria para máquinas virtuales de IaaS](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-daily-backup-policy-create) | La plantilla crea un almacén de Recovery Services y una directiva de copia de seguridad diaria que se usa para realizar la copia de seguridad de máquinas virtuales de los modelos clásico y de Resource Manager.|
| [Implementación de una máquina virtual de Windows Server con copia de seguridad habilitada](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.recoveryservices/recovery-services-create-vm-and-configure-backup) | La plantilla crea una máquina virtual de Windows Server y un almacén de Recovery Services con la directiva de copia de seguridad predeterminada habilitada.|
|**Supervisión de trabajos de copia de seguridad** |  |
| [Uso de los registros de Azure Monitor con Azure Backup](https://github.com/Azure/azure-quickstart-templates/tree/master/101-backup-oms-monitoring) | La plantilla implementa los registros de Azure Monitor con Azure Backup, lo cual le permite supervisar los trabajos de copia de seguridad y restauración, las alertas de copia de seguridad y el almacenamiento en la nube que se usa en los almacenes de Recovery Services.|  
|**Instancia de copia de seguridad de SQL Server en Azure VM** |  |
| [Instancia de copia de seguridad de SQL Server en Azure VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vm-workload-backup) | La plantilla crea un almacén de Recovery Services y una directiva de copia de seguridad específica para la carga de trabajo. Registra la máquina virtual con el servicio Azure Backup y configura la protección en esa máquina virtual. Actualmente, solo funciona para las imágenes de la galería SQL. |
|**Copia de seguridad de recursos compartidos de archivos de Azure** |  |
| [Copia de seguridad de recursos compartidos de archivos de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-backup-file-share) | Esta plantilla configura la protección para un recurso compartido de archivos de Azure existente al especificar los detalles correspondientes para la directiva de copia de seguridad y el almacén de Recovery Services. Opcionalmente, crea una directiva de copia de seguridad y un almacén de Recovery Services nuevos, y registra la cuenta de almacenamiento que contiene el recurso compartido de archivos en el almacén de Recovery Services. |
|   |   |
