---
title: Detalles de la estructura de asignaciones de directivas
description: Describe la definición de asignación de directiva utilizada por Azure Policy para relacionar las definiciones de directiva y los parámetros con los recursos para su evaluación.
ms.date: 04/14/2021
ms.topic: conceptual
ms.openlocfilehash: 9de210b17264330e79ab5978a449e7a494054be2
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107535877"
---
# <a name="azure-policy-assignment-structure"></a>Estructura de asignaciones de Azure Policy

Azure Policy usa las asignaciones de directivas para definir qué recursos se asignan a qué directivas o iniciativas. La asignación de directivas puede determinar los valores de los parámetros para ese grupo de recursos en el momento de la asignación, lo que permite volver a usar las definiciones de directiva que se ocupan de las mismas propiedades de recursos con diferentes necesidades de cumplimiento.

Utilice JSON para crear una definición de directiva. La asignación de directivas contiene elementos para:

- nombre para mostrar
- description
- metadata
- modo de cumplimiento
- ámbitos excluidos
- definición de directiva
- mensajes de no cumplimiento
- parámetros

Por ejemplo, el siguiente archivo JSON muestra una asignación de directiva en el modo _DoNotEnforce_ con parámetros dinámicos:

```json
{
    "properties": {
        "displayName": "Enforce resource naming rules",
        "description": "Force resource names to begin with DeptA and end with -LC",
        "metadata": {
            "assignedBy": "Cloud Center of Excellence"
        },
        "enforcementMode": "DoNotEnforce",
        "notScopes": [],
        "policyDefinitionId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyDefinitions/ResourceNaming",
        "nonComplianceMessages": [
            {
                "message": "Resource names must start with 'DeptA' and end with '-LC'."
            }
        ],
        "parameters": {
            "prefix": {
                "value": "DeptA"
            },
            "suffix": {
                "value": "-LC"
            }
        }
    }
}
```

Todos los ejemplos de Azure Policy se encuentran en [Ejemplos de Azure Policy](../samples/index.md).

## <a name="display-name-and-description"></a>Nombre para mostrar y descripción

Use **displayName** y **description** para identificar la asignación de directiva y proporcionar el contexto para su uso con el conjunto específico de recursos. **displayName** tiene una longitud máxima de _128_ caracteres y **description** tiene una longitud máxima de _512_ caracteres.

## <a name="metadata"></a>Metadatos

La propiedad `metadata` opcional almacena información sobre la asignación de la directiva. Los clientes pueden definir las propiedades y los valores útiles para su organización en `metadata`. Aun así, hay algunas propiedades _comunes_ que se usan en Azure Policy. Cada propiedad `metadata` tiene un límite de 1024 caracteres.

### <a name="common-metadata-properties"></a>Propiedades de metadatos comunes

- `assignedBy` (cadena): nombre descriptivo de la entidad de seguridad que creó la asignación.
- `createdBy` (cadena): el GUID de la entidad de seguridad que creó la asignación.
- `createdOn` (cadena): el formato de fecha y hora universal ISO 8601 de la hora de creación de la asignación.
- `parameterScopes`(objeto): colección de pares clave-valor donde la clave coincide con un nombre de parámetro configurado [strongType](./definition-structure.md#strongtype) y el valor define el ámbito de recurso usado en el portal para proporcionar la lista de recursos disponibles mediante la coincidencia de _strongType_. El portal establece este valor si el ámbito es diferente del ámbito de asignación. Si se establece, una edición de la asignación de directiva en el portal establece automáticamente el ámbito del parámetro en este valor. Sin embargo, el ámbito no está bloqueado en el valor y se puede cambiar a otro ámbito.

  El ejemplo siguiente de `parameterScopes` es para un parámetro _strongType_ denominado **backupPolicyId** que establece un ámbito para la selección de recursos cuando la asignación se edita en el portal.

  ```json
  "metadata": {
      "parameterScopes": {
          "backupPolicyId": "/subscriptions/{SubscriptionID}/resourcegroups/{ResourceGroupName}"
      }
  }
  ```

- `updatedBy` (cadena): nombre descriptivo de la entidad de seguridad que actualizó la asignación, si la hubiera.
- `updatedOn` (cadena): el formato de fecha y hora universal ISO 8601 de la hora de actualización de la asignación, si lo hubiera.

## <a name="enforcement-mode"></a>Modo de cumplimiento

La propiedad **enforcementMode** proporciona a los clientes la capacidad de probar el resultado de una directiva en los recursos existentes sin necesidad de iniciar el efecto de la directiva ni de desencadenar entradas en el [registro de actividad de Azure](../../../azure-monitor/essentials/platform-logs-overview.md). Este escenario se conoce comúnmente como "What If" y se alinea con las prácticas de implementación segura. **enforcementMode** es diferente del efecto [Deshabilitado](./effects.md#disabled), ya que ese efecto evita que se produzca la evaluación de recursos.

Esta propiedad tiene los siguientes valores:

|Mode |Valor JSON |Tipo |Corregir manualmente |Entrada de registro de actividad |Descripción |
|-|-|-|-|-|-|
|habilitado |Valor predeterminado |string |Sí |Sí |El efecto de la directiva se aplica durante la creación o actualización de recursos. |
|Disabled |DoNotEnforce |string |Sí |No | El efecto de la directiva no se aplica durante la creación o actualización de recursos. |

Si **enforcementMode** no se especifica en una definición de directiva o iniciativa, se usa el valor _Default_. [Las tareas de corrección](../how-to/remediate-resources.md) se pueden iniciar para las directivas [deployIfNotExists](./effects.md#deployifnotexists), incluso cuando **enforcementMode** está establecido en _DoNotEnforce_.

## <a name="excluded-scopes"></a>Ámbitos excluidos

El **ámbito** de la asignación incluye todos los contenedores de recursos secundarios y los recursos secundarios. Si la definición no debe aplicarse a un contenedor de recursos secundario o a un recurso secundario, se puede _excluir_ cada uno de ellos de la evaluación estableciendo **notScopes**. Esta propiedad es una matriz que permite excluir de la evaluación a uno o varios contenedores de recursos o recursos. **notScopes** se puede agregar o actualizar después de la creación de la asignación inicial.

> [!NOTE]
> Un recurso _excluido_ no es lo mismo que un recurso _exento_. Para más información, consulte [Descripción del ámbito en Azure Policy](./scope.md).

## <a name="policy-definition-id"></a>Identificador de la definición de directiva

Este campo debe ser el nombre de la ruta de acceso completa de una definición de directiva o una definición de iniciativa.
`policyDefinitionId` es una cadena y no una matriz. Se recomienda que, si a menudo se asignan varias directivas, se use una [iniciativa](./initiative-definition-structure.md) en su lugar.

## <a name="non-compliance-messages"></a>Mensajes de no cumplimiento

Para establecer un mensaje personalizado que describa por qué un recurso no es compatible con la definición de la directiva o de la iniciativa, establezca `nonComplianceMessages` en la definición de la asignación. Este nodo es una matriz de `message` entradas. Este mensaje personalizado es adicional al mensaje de error predeterminado para el no cumplimiento y es opcional.

> [!IMPORTANT]
> Los mensajes personalizados de no cumplimiento solo se admiten en definiciones o iniciativas con definiciones de [modos de Resource Manager](./definition-structure.md#resource-manager-modes).

```json
"nonComplianceMessages": [
    {
        "message": "Default message"
    }
]
```

Si la asignación es para una iniciativa, se pueden configurar diferentes mensajes para cada definición de directiva en la iniciativa. Los mensajes utilizan el valor `policyDefinitionReferenceId` configurado en la definición de la iniciativa. Para más información, consulte las [propiedades de las definiciones de directiva](./initiative-definition-structure.md#policy-definition-properties).

```json
"nonComplianceMessages": [
    {
        "message": "Default message"
    },
    {
        "message": "Message for just this policy definition by reference ID",
        "policyDefinitionReferenceId": "10420126870854049575"
    }
]
```

## <a name="parameters"></a>Parámetros

Este segmento de la asignación de directiva proporciona los valores para los parámetros definidos en la [definición de directiva o en la definición de iniciativa](./definition-structure.md#parameters). Este diseño permite reutilizar una definición de directiva o iniciativa con distintos recursos, así como comprobar si hay valores o resultados empresariales diferentes.

```json
"parameters": {
    "prefix": {
        "value": "DeptA"
    },
    "suffix": {
        "value": "-LC"
    }
}
```

En este ejemplo, los parámetros definidos anteriormente en la definición de directiva son `prefix` y `suffix`. Esta asignación de directiva determinada establece `prefix` en **DeptA** y `suffix` en **-LC**. La misma definición de directiva se puede volver a usar con un conjunto diferente de parámetros para otro departamento, lo que reduce la duplicación y la complejidad de las definiciones de directiva, a la vez que proporciona flexibilidad.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [estructura de la definición de directiva](./definition-structure.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
