---
title: Eliminación de un proyecto de Azure Migrate
description: En este artículo, aprenderá a eliminar un proyecto de Azure Migrate mediante Azure Portal.
author: ms-psharma
ms.author: panshar
ms.manager: abhemraj
ms.topic: how-to
ms.date: 10/22/2019
ms.openlocfilehash: 8c94bb23f5d514fef5cdacb855657efdf5219631
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714747"
---
# <a name="delete-an-azure-migrate-project"></a>Eliminación de un proyecto de Azure Migrate

En este artículo se describe cómo eliminar un proyecto de [Azure Migrate](./migrate-services-overview.md).


## <a name="before-you-start"></a>Antes de comenzar

Antes de eliminar un proyecto:

- Cuando se elimina un proyecto, el proyecto y los metadatos de la máquina detectada también se eliminan.
- Si ha adjuntado un área de trabajo de Log Analytics a la herramienta Server Assessment para el análisis de dependencias, decida si quiere eliminar el área de trabajo. 
    - El área de trabajo no se elimina automáticamente. Elimínela manualmente.
    - Compruebe para qué se usa un área de trabajo antes de eliminarla. La misma área de trabajo de Log Analytics se puede usar para varios escenarios.
    - Antes de eliminar el proyecto, puede encontrar un vínculo al área de trabajo en **Azure Migrate - Servers** > **Azure Migrate - Server Assessment**, en **Área de trabajo de OMS**.
    - Para eliminar un área de trabajo después de eliminar un proyecto, busque el área de trabajo en el grupo de recursos correspondiente y siga [estas instrucciones](../azure-monitor/logs/delete-workspace.md).


## <a name="delete-a-project"></a>Eliminación de un proyecto


1. En Azure Portal, abra el grupo de recursos en el que se ha creado el proyecto.
2. En la página del grupo de recursos, seleccione **Mostrar tipos ocultos**.
3. Seleccione el proyecto y los recursos asociados que desea eliminar.
    - El tipo de recurso de los proyectos de Azure Migrate es **Microsoft.Migrate/migrateprojects**.
    - En la siguiente sección, revise los recursos creados para la detección, la evaluación y la migración en un proyecto de Azure Migrate.
    - Si el grupo de recursos solo contiene el proyecto de Azure Migrate, puede eliminar todo el grupo de recursos.
    - Si quiere eliminar un proyecto de la versión anterior de Azure Migrate, los pasos son los mismos. El tipo de recurso para estos proyectos es **Proyecto de migración**.


## <a name="created-resources"></a>Recursos creados

En estas tablas, se resumen los recursos creados para la detección, la evaluación y la migración en un proyecto de Azure Migrate.

> [!NOTE]
> Elimine el almacén de claves con precaución, ya que puede contener claves de seguridad.

### <a name="projects-with-public-endpoint-connectivity"></a>Proyectos con conectividad de punto de conexión público

#### <a name="vmwarephysical-server"></a>Servidores físicos o de VMware

**Recurso** | **Tipo**
--- | ---
"Appliancename"kv | Almacén de claves
"Appliancename"site | Microsoft.OffAzure/VMwareSites
"ProjectName" | Microsoft.Migrate/migrateprojects
"ProjectName"project | Microsoft.Migrate/assessmentProjects
"ProjectName"rsvault | Almacén de Recovery Services
"ProjectName"-MigrateVault-* | Almacén de Recovery Services
migrateappligwsa* | Cuenta de almacenamiento
migrateapplilsa* | Cuenta de almacenamiento
migrateapplicsa* | Cuenta de almacenamiento
migrateapplikv* | Almacén de claves
migrateapplisbns* | Espacio de nombres de Service Bus

#### <a name="hyper-v-vm"></a>Máquina virtual de Hyper-V

**Recurso** | **Tipo**
--- | ---
"ProjectName" | Microsoft.Migrate/migrateprojects
"ProjectName"project | Microsoft.Migrate/assessmentProjects
HyperV*kv | Almacén de claves
HyperV*Site | Microsoft.OffAzure/HyperVSites
"ProjectName"-MigrateVault-* | Almacén de Recovery Services

<br/>
En las tablas siguientes se resumen los recursos creados por Azure Migrate para detectar, evaluar y migrar servidores a través de una red privada mediante [Azure Private Link](./how-to-use-azure-migrate-with-private-endpoints.md).

### <a name="projects-with-private-endpoint-connectivity"></a>Proyectos con conectividad de punto de conexión privado

#### <a name="vmware-vms---agentless-migrations"></a>Migración sin agentes de máquinas virtuales de VMware

**Tipo** | **Recurso** | **Punto de conexión privado <br/>** |
--- | --- | ---
Microsoft.Migrate/migrateprojects | "ProjectName" | "ProjectName"\*pe 
Sitio de detección (sitio maestro) | "ProjectName"*mastersite | "ProjectName"\*mastersite\*pe 
Microsoft.Migrate/assessmentProjects | "ApplianceName"*project | "ApplianceName"\*project\*pe 
Almacén de claves | "ProjectName"*kv | "ProjectName"\*kv\*pe
Microsoft.OffAzure/VMwareSites | "ApplianceName"*site | N/D
Almacén de Recovery Services | "ApplianceName"*vault | N/D
Cuenta de almacenamiento | "ApplianceName"*usa | "ApplianceName"\*usa\*pe
Almacén de Recovery Services | "ProjectName"-MigrateVault-* | N/D
Cuenta de almacenamiento | migrateappligwsa* | N/D
Cuenta de almacenamiento | migrateapplilsa* | N/D
Almacén de claves | migrateapplikv* | N/D
Espacio de nombres de Service Bus | migrateapplisbns* | N/D

#### <a name="hyper-v-vms"></a>Máquinas virtuales de Hyper-V 

**Tipo** | **Recurso** | **Punto de conexión privado <br/>** |
--- | --- | ---
Microsoft.Migrate/migrateprojects | "ProjectName" | "ProjectName"\*pe 
Sitio de detección (sitio maestro) | "ProjectName"*mastersite | "ProjectName"\*mastersite\*pe 
Microsoft.Migrate/assessmentProjects | "ApplianceName"*project | "ApplianceName"\*project\*pe 
Almacén de claves | "ProjectName"*kv | "ProjectName"\*kv\*pe
Microsoft.OffAzure/HyperVSites | "ApplianceName"*site | N/D
Almacén de Recovery Services | "ProjectName"-MigrateVault-* | "ProjectName"-MigrateVault-*pe

#### <a name="physical-servers--aws-vms--gcp-vms"></a>Servidores físicos, máquinas virtuales de AWS o máquinas virtuales de GCP 

**Tipo** | **Recurso** | **Punto de conexión privado <br/>** |
--- | --- | ---
Microsoft.Migrate/migrateprojects | "ProjectName" | "ProjectName"\*pe 
Sitio de detección (sitio maestro) | "ProjectName"*mastersite | "ProjectName"\*mastersite\*pe 
Microsoft.Migrate/assessmentProjects | "ApplianceName"*project | "ApplianceName"\*project\*pe 
Almacén de claves | "ProjectName"*kv | "ProjectName"\*kv\*pe
Microsoft.OffAzure/serversites | "ApplianceName"*site | N/D
Almacén de Recovery Services | "ProjectName"-MigrateVault-* | "ProjectName"-MigrateVault-*pe


## <a name="next-steps"></a>Pasos siguientes

Aprenda a agregar más herramientas de [valoración](how-to-assess.md) y [migración](how-to-migrate.md). 
