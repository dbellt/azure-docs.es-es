---
title: 'Funciones de plantillas: any'
description: Describe la función any que está disponible en Bicep para convertir tipos.
ms.topic: conceptual
author: tfitzmac
ms.author: tomfitz
ms.service: azure-resource-manager
ms.subservice: templates
ms.date: 03/02/2021
ms.openlocfilehash: b0cb51c9a79d1100de7f1ef32fe326eddcdd6dcc
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353476"
---
# <a name="any-function-for-bicep"></a>Función any para Bicep

Bicep admite una función llamada `any()` para resolver errores de tipo en el sistema de tipos de Bicep. Esta función se usa cuando el formato del valor que se proporciona no coincide con lo que espera el sistema de tipos. Por ejemplo, si la propiedad requiere un número, pero se debe proporcionar como cadena, por ejemplo `'0.5'`, use la función `any()` para suprimir el error indicado por el sistema de tipos.

Esta función no existe en el entorno en tiempo de ejecución de las plantillas de Azure Resource Manager. Solo se usa en Bicep y no se emite en el JSON para la plantilla compilada.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="any"></a>cualquiera

`any(value)`

Devuelve un valor que es compatible con cualquier tipo de datos.

### <a name="parameters"></a>Parámetros

| Parámetro | Obligatorio | Tipo | Descripción |
|:--- |:--- |:--- |:--- |
| value | Sí | Todos los tipos | Valor que se va a convertir en un tipo compatible. |

### <a name="return-value"></a>Valor devuelto

El valor en un formulario, que es compatible con cualquier tipo de datos.

### <a name="examples"></a>Ejemplos

En la plantilla de ejemplo siguiente se muestra cómo usar la función `any()` para proporcionar valores numéricos como cadenas.

```bicep
resource wpAci 'microsoft.containerInstance/containerGroups@2019-12-01' = {
  name: 'wordpress-containerinstance'
  location: location
  properties: {
    containers: [
      {
        name: 'wordpress'
        properties: {
          ...
          resources: {
            requests: {
              cpu: any('0.5')
              memoryInGB: any('0.7')
            }
          }
        }
      }
    ]
  }
}
```

La función sirve para cualquier valor asignado en Bicep. En el ejemplo siguiente se usa `any()` con una expresión ternaria como argumento.  

```bicep
publicIPAddress: any((pipId == '') ? null : {
  id: pipId
})
```

## <a name="next-steps"></a>Pasos siguientes

Para conocer usos más complejos de la función `any()`, consulte los ejemplos siguientes:

* [Recursos secundarios que requieren nombres específicos](https://github.com/Azure/bicep/blob/main/docs/examples/201/api-management-create-all-resources/main.bicep#L246)
* [Propiedad de recurso no definida en el tipo del recurso, aunque existe](https://github.com/Azure/bicep/blob/main/docs/examples/201/log-analytics-with-solutions-and-diagnostics/main.bicep#L26)

