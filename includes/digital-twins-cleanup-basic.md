---
author: baanders
description: archivo de inclusión para limpiar una instancia de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 2/4/2021
ms.author: baanders
ms.openlocfilehash: 0783f25813fa8e4a65a33975017e2b2d03a4dd2c
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110075491"
---
* **Si no necesita ninguno de los recursos que creó en este tutorial**, puede eliminar la instancia de Azure Digital Twins y todos los demás recursos de este artículo con el comando [az group delete](/cli/azure/group?view=azure-cli-latest&preserve-view=true#az_group_delete). Esto permite eliminar todos los recursos de Azure de un grupo de recursos, así como el grupo en sí.
    
    > [!IMPORTANT]
    > La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados.
    
    Abra [Azure Cloud Shell](https://shell.azure.com) y ejecute el siguiente comando para eliminar el grupo de recursos y todo lo que contiene.
    
    ```azurecli-interactive
    az group delete --name <your-resource-group>
    ```