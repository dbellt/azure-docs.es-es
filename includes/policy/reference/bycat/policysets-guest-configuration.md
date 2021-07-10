---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 06/11/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 897a95014cdbc9f24e47cfe123fa443b330d5fb4
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112040363"
---
|Nombre |Descripción |Directivas |Versión |
|---|---|---|---|
|[Auditar las máquinas que tengan una configuración de seguridad de contraseña no segura](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordSettingsAINE.json) |Esta iniciativa implementa los requisitos de la directiva y audita las máquinas que cuenten con una configuración de seguridad de contraseña no segura. Para más información sobre las directivas de configuración de invitados, visite [https://aka.ms/gcpol](https://aka.ms/gcpol) |9 |1.0.0 |
|[Implementar los requisitos previos para habilitar directivas de configuración de invitado en máquinas virtuales](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json) |Esta iniciativa agrega una identidad administrada asignada por el sistema e implementa la extensión de configuración de invitado adecuada para la plataforma en las máquinas virtuales que se puedan supervisar mediante directivas de configuración de invitado. Este es un requisito previo para todas las directivas de configuración de invitado y se debe agregar al ámbito de asignación de directivas antes de usar cualquier directiva de configuración de invitado. Para más información sobre la configuración de invitado, visite [https://aka.ms/gcpol](https://aka.ms/gcpol). |4 |1.0.0 |
|[\[Versión preliminar\]: las máquinas Windows deben cumplir los requisitos de línea base de seguridad de Azure Compute](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_AzureBaseline.json) |Esta iniciativa audita las máquinas Windows con una configuración que no cumple la línea base de seguridad de Azure. Para obtener detalles, consulte: [https://aka.ms/gcpol](https://aka.ms/gcpol) |29 |2.0.1-preview |
