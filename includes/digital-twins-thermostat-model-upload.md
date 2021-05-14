---
author: baanders
description: archivo de inclusión para cargar un modelo en la instancia de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/1/2021
ms.author: baanders
ms.openlocfilehash: 3a9ec169f91ebe048914c3ef2163e4fde7485389
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783681"
---
El modelo tiene este aspecto:
:::code language="json" source="~/digital-twins-docs-samples/models/Thermostat.json":::

Para **cargar este modelo en la instancia del gemelo**, ejecute el siguiente comando de la CLI de Azure, que carga el modelo anterior como JSON insertado. Puede ejecutar el comando en [Azure Cloud Shell](../articles/cloud-shell/overview.md) en el explorador (use el entorno de **Bash**), o en su máquina si tiene la CLI [instalada localmente](/cli/azure/install-azure-cli).

```azurecli-interactive
az dt model create --models '{  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",  "@type": "Interface",  "@context": "dtmi:dtdl:context;2",  "contents": [    {      "@type": "Property",      "name": "Temperature",      "schema": "double"    }  ]}' --dt-name {digital_twins_instance_name}
```