---
author: gvashishtha
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: include
ms.date: 04/21/2021
ms.author: gopalv
ms.openlocfilehash: 0bac8d2b70ea1dc0dd624cae669f6b2f4c2e1c6e
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107952207"
---
El script de entrada recibe los datos enviados a un servicio web implementado y los pasa al modelo. Seguidamente, devuelve la respuesta del modelo al cliente. *El script es específico para su modelo*. El script de entrada debe entender los datos que el modelo espera y devuelve.

Estas son las dos tareas que debe realizar en el script de entrada:

1. Cargar el modelo (mediante una función llamada `init()`)
1. Ejecutar el modelo sobre los datos de entrada (mediante una función llamada `run()`)

Para la implementación inicial, use un script de entrada ficticio que imprima los datos que recibe.

```python
import json

def init():
    print('This is init')

def run(data):
    test = json.loads(data)
    print(f'received data {test}')
    return(f'test is {test}')

```
Guarde este archivo como `echo_score.py` dentro de un directorio denominado `source_dir`.

Por ejemplo, si un usuario llama al modelo con:

```bash
curl -X POST -d '{"this":"is a test"}' -H "Content-Type: application/json" http://localhost:6789/score
```

Se devuelve el siguiente valor:

```bash
"test is {'this': 'is a test'}"
```