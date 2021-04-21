---
author: baanders
description: archivo de inclusión para cargar un modelo en la instancia de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/1/2021
ms.author: baanders
ms.openlocfilehash: add49cabaece1187cb9bcda3a94c92feb08b2439
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304310"
---
El modelo tiene este aspecto:
:::code language="json" source="~/digital-twins-docs-samples/models/Thermostat.json":::

Para **cargar este modelo en la instancia del gemelo**, ejecute el siguiente comando de la CLI de Azure, que carga el modelo anterior como JSON insertado. Puede ejecutar el comando en [Azure Cloud Shell](../articles/cloud-shell/overview.md) en el explorador (use el entorno de **Bash**), o en su máquina si tiene la CLI [instalada localmente](/cli/azure/install-azure-cli).

```azurecli-interactive
az dt model create --models '{  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",  "@type": "Interface",  "@context": "dtmi:dtdl:context;2",  "contents": [    {      "@type": "Property",      "name": "Temperature",      "schema": "double"    }  ]}' -n {digital_twins_instance_name}
```