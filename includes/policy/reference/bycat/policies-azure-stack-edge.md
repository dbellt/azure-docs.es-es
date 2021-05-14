---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/04/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 82536802a63c81bac1c3de58d3a059d59e5f09db
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108755301"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los dispositivos Azure Stack Edge deben usar cifrado doble](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb4ac1030-89c5-4697-8e00-28b5ba6a8811) |Para proteger los datos en reposo del dispositivo, asegúrese de que tienen cifrado doble, se controla el acceso a ellos y, una vez desactivado el dispositivo, se borran de los discos de datos de forma segura. El cifrado doble consiste en dos capas de cifrado: XTS-AES de BitLocker de 256 bits en los volúmenes de datos y cifrado integrado en los discos duros. Más información en la documentación de información general sobre seguridad del dispositivo Stark Edge en cuestión. |audit, deny, disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Stack%20Edge/AzureStackEdge_DoubleEncryption_Audit.json) |
