---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: c39c6d6530e94e8e84211b9950cd66f5b33c8c3c
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039438"
---
|Nombre |Descripción |Directivas |Versión |
|---|---|---|---|
|[\[Versión preliminar\]: Configurar Azure Defender para agentes SQL en máquinas virtuales](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitoring_AzureDefenderForSql.json) |Configure las máquinas virtuales para que instalen automáticamente los agentes de Azure Defender para SQL donde está instalado el agente de Azure Monitor. Security Center recopila eventos de los agentes y los usa para proporcionar alertas de seguridad y tareas de protección personalizadas (recomendaciones). Cree un grupo de recursos y un área de trabajo de Log Analytics en la misma región que la máquina. Esta directiva solo se aplica a las máquinas virtuales en algunas regiones. |2 |1.0.0-preview |
|[Configurar el agente de Azure Monitor en Linux Virtual Machines y asociarlo a una regla de recopilación de datos](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AMCS_LinuxPlatform_EnableDCR.json) |Implemente el agente de Azure Monitor para Linux Virtual Machines si la imagen de máquina virtual (SO) y la ubicación están en la lista definida y el agente no está instalado.  A continuación, implemente Asociación para vincular la máquina virtual a la regla de recopilación de datos especificada. La lista de imágenes de sistema operativo se actualiza con el tiempo a medida que aumenta la compatibilidad. |2 |1.0.0 |
|[Configurar el agente de Azure Monitor en Windows Virtual Machines y asociarlo a una regla de recopilación de datos](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AMCS_WindowsPlatform_EnableDCR.json) |Implemente el agente de Log Analytics para Windows Virtual Machines si la imagen de la máquina virtual (SO) está en la lista definida y el agente no está instalado.  A continuación, implemente Asociación para vincular la máquina virtual a la regla de recopilación de datos especificada. La lista de imágenes de sistema operativo se actualiza con el tiempo a medida que aumenta la compatibilidad. |2 |1.0.0 |
|[\[Versión preliminar\]: Implementación: configurar los requisitos previos para habilitar los agentes de Azure Monitor y de seguridad de Azure en máquinas virtuales](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitoring_Prerequisites.json) |Configure las máquinas para instalar automáticamente los agentes de seguridad de Azure y Azure Monitor. Security Center recopila eventos de los agentes y los usa para proporcionar alertas de seguridad y tareas de protección personalizadas (recomendaciones). Cree un grupo de recursos y un área de trabajo de Log Analytics en la misma región que la máquina para almacenar los registros de auditoría. Esta directiva solo se aplica a las máquinas virtuales en algunas regiones. |5 |1.0.0-preview |
|[Habilitar Azure Monitor para conjunto de escalado de máquinas virtuales](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitor_VMSS.json) |Se habilita Azure Monitor para los conjuntos de escalado de máquinas virtuales en el ámbito especificado (grupo de administración, suscripción o grupo de recursos). Usa el área de trabajo de Log Analytics como parámetro. Nota: Si establece upgradePolicy del conjunto de escalado en Manual, debe aplicar la extensión a todas las máquinas virtuales del conjunto mediante una llamada a la actualización. En la CLI, se usaría az vmss update-instances. |6 |1.0.1 |
|[Habilitar Azure Monitor para VM](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitor_VM.json) |Se habilita Azure Monitor para las máquinas virtuales en el ámbito especificado (grupo de administración, suscripción o grupo de recursos). Usa el área de trabajo de Log Analytics como parámetro. |10 |2.0.0 |
