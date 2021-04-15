---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 10/16/2020
ms.author: larryfr
ms.openlocfilehash: 25e304daf75b60a7d037640d496ed0972581f13a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92204476"
---
Azure permite colocar _bloqueos_ en los recursos, de modo que no se puedan eliminar o sean de solo lectura. __El bloqueo de un recurso puede producir resultados inesperados.__ Algunas operaciones que no parecen modificar el recurso realmente requieren acciones que ha bloqueado el bloqueo. 

Por ejemplo, al aplicar un bloqueo de eliminación en el grupo de recursos del área de trabajo, se evitarán las operaciones de escalado de los clústeres de proceso de Azure ML.

Para obtener más información sobre el bloqueo de recursos, vea [Bloqueo de recursos para impedir cambios inesperados](../articles/azure-resource-manager/management/lock-resources.md).