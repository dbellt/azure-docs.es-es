---
author: baanders
description: archivo de inclusión para limpiar una instancia de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 2/4/2021
ms.author: baanders
ms.openlocfilehash: 0d8cc30c0511098caf7b6c47d7f7bd400dc32f1b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800100"
---
* **Si no necesita ninguno de los recursos que creó en este tutorial**, puede eliminar la instancia de Azure Digital Twins y todos los demás recursos de este artículo con el comando [az group delete](/cli/azure/group#az_group_delete). Esto permite eliminar todos los recursos de Azure de un grupo de recursos, así como el grupo en sí.
    
    > [!IMPORTANT]
    > La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados.
    
    Abra [Azure Cloud Shell](https://shell.azure.com) y ejecute el siguiente comando para eliminar el grupo de recursos y todo lo que contiene.
    
    ```azurecli-interactive
    az group delete --name <your-resource-group>
    ```