---
title: 'Problemas conocidos con las identidades administradas: Azure Active Directory'
description: Problemas conocidos con identidades administradas para recursos de Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.assetid: 2097381a-a7ec-4e3b-b4ff-5d2fb17403b6
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 04/08/2021
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 8d2b6323a15595e57e7e89c6a83f9f718422e1d7
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107226929"
---
# <a name="known-issues-with-managed-identities"></a>Problemas conocidos con las identidades administradas

En este artículo se analizan un par de problemas relacionados con las identidades administradas y cómo abordarlos. Las preguntas comunes sobre las identidades administradas se documentan en nuestro artículo de [preguntas más frecuentes](managed-identities-faq.md).
## <a name="vm-fails-to-start-after-being-moved"></a>La máquina virtual no se inicia tras el traslado 

Si mueve una máquina virtual en estado en ejecución desde un grupo de recursos o una suscripción, esta continúa ejecutándose durante el traslado. Sin embargo, después, si la máquina virtual se detiene y se reinicia, no se podrá iniciar. Este problema se produce porque la máquina virtual no está actualizando la referencia a la identidad referente a las identidades administradas para recursos de Azure, y continúa apuntando a ella en el grupo de recursos anterior.

**Solución alternativa** 
 
Desencadene una actualización en la máquina virtual para que pueda obtener los valores correctos para las identidades administradas para recursos de Azure. Puede hacer un cambio de propiedad de máquina virtual para actualizar la referencia a la identidad referente a las identidades administradas para recursos de Azure. Por ejemplo, puede establecer un nuevo valor de etiqueta en la máquina virtual con el siguiente comando:

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --set tags.fixVM=1
```
 
Este comando establece una nueva etiqueta "fixVM" con un valor de 1 en la máquina virtual. 
 
Al establecer esta propiedad, la máquina virtual se actualiza con el URI del recurso correcto de las identidades administradas para recursos de Azure y, después, debería poder iniciar la máquina virtual. 
 
Una vez que se inicia la máquina virtual, la etiqueta puede quitarse con el comando siguiente:

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --remove tags.fixVM
```

## <a name="transferring-a-subscription-between-azure-ad-directories"></a>Transferencia de una suscripción entre directorios de Azure AD

Las identidades administradas no se actualizan cuando una suscripción se mueve o transfiere a otro directorio. Como resultado, se interrumpe cualquier identidad administrada asignada por el sistema o por el usuario. 

Solución alternativa para identidades administradas en una suscripción que se ha movido a otro directorio:

 - Para las identidades administradas asignadas por el sistema, tiene que desactivarlas y volver a activarlas. 
 - Para las identidades administradas asignadas por el usuario, tiene que eliminarlas, volver a crearlas y adjuntarlas de nuevo a los recursos necesarios (por ejemplo, máquinas virtuales).

Para más información, consulte [Transferencia de una suscripción de Azure a otro directorio de Azure AD](../../role-based-access-control/transfer-subscription.md).


## <a name="next-steps"></a>Pasos siguientes

Puede revisar nuestro artículo en el que se enumeran los [servicios que admiten identidades administradas](services-support-managed-identities.md) y nuestras [preguntas más frecuentes](managed-identities-faq.md).
