---
author: wesmc7777
ms.service: redis-cache
ms.topic: include
ms.date: 04/02/2019
ms.author: wesmc
ms.openlocfilehash: 498a7ee28b9404d0733e4615f4df635a8c904b51
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186124"
---
### <a name="cacheskuname"></a>cacheSKUName

El plan de tarifa de la nueva instancia de Azure Cache for Redis.

```json
    "cacheSKUName": {
      "type": "string",
      "allowedValues": [
        "Basic",
        "Standard",
        "Premium"
      ],
      "defaultValue": "Basic",
      "metadata": {
        "description": "The pricing tier of the new Azure Cache for Redis."
      }
    },
```

La plantilla define los valores permitidos para este parámetro (Básico, Estándar o Premium) y asigna un valor predeterminado (Básico) si no se especifica ningún valor. Básico ofrece un único nodo con varios tamaños disponibles hasta 53 GB. Estándar proporciona dos nodos, principal/réplica, con varios tamaños disponibles de hasta 53 GB y un contrato de nivel de servicio del 99,9%.

### <a name="cacheskufamily"></a>cacheSKUFamily

La familia de la SKU.

```json
    "cacheSKUFamily": {
      "type": "string",
      "allowedValue/s": [
        "C",
        "P"
      ],
      "defaultValue": "C",
      "metadata": {
        "description": "The family for the sku."
      }
    },
```

### <a name="cacheskucapacity"></a>cacheSKUCapacity

El tamaño de la nueva instancia de Azure Cache for Redis.

Familias Básico y Estándar:

```json
    "cacheSKUCapacity": {
      "type": "int",
      "allowedValues": [
        0,
        1,
        2,
        3,
        4,
        5,
        6
      ],
      "defaultValue": 0,
      "metadata": {
        "description": "The size of the new Azure Cache for Redis instance. "
      }
    }
```

La capacidad de caché del valor Premium se define lo mismo, excepto que los valores permitidos van de 1 a 5 en lugar de 0 a 6.

La plantilla define los valores enteros permitidos para este parámetro (0 a 6 para las familias Básico y Estándar; 1 a 5 para la familia Premium). Si no se especifica ningún valor, la plantilla asigna un valor predeterminado de 0 para Básico y Estándar, 1 para Premium.

Los valores corresponden a los siguientes tamaños de caché:

| Value | Básico y Estándar<br>tamaño de caché | Premium<br>tamaño de caché |
| :---: | :------------------------------: | :-------------------: |
| 0     | 250 MB (valor predeterminado)                 | N/D                   |
| 1     | 1 GB                             | 6 GB (valor predeterminado)        |
| 2     | 2,5 GB                           | 13 GB                 |
| 3     | 6 GB                             | 26 GB                 |
| 4     | 13 GB                            | 53 GB                 |
| 5     | 26 GB                            | 120 GB                |
| 6     | 53 GB                            | N/D                   |
