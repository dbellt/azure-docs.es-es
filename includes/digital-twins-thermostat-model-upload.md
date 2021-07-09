---
author: baanders
description: archivo de inclusión para cargar un modelo en la instancia de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/1/2021
ms.author: baanders
ms.openlocfilehash: 9fcbfc0b0fccb5f130f0c554bac93748176387b7
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111438988"
---
El modelo tiene este aspecto:
:::code language="json" source="~/digital-twins-docs-samples/models/Thermostat.json":::

Para **cargar este modelo en la instancia del gemelo**, ejecute el siguiente comando de la CLI de Azure, que carga el modelo anterior como JSON insertado. Puede ejecutar el comando en [Azure Cloud Shell](../articles/cloud-shell/overview.md) en el explorador (use el entorno de **Bash**), o en su máquina si tiene la [CLI instalada localmente](/cli/azure/install-azure-cli).

```azurecli-interactive
az dt model create --dt-name <instance-name> --models '{  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",  "@type": "Interface",  "@context": "dtmi:dtdl:context;2",  "contents": [    {      "@type": "Property",      "name": "Temperature",      "schema": "double"    }  ]}' 
```