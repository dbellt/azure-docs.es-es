---
title: Uso de Azure Policy para implementar la gobernanza y los controles de recursos de Azure Cosmos DB
description: Aprenda a usar Azure Policy para implementar la gobernanza y los controles de recursos de Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/23/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3f1fffb10bf47cab28d31bd3c180b12cd2adadde
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110680770"
---
# <a name="use-azure-policy-to-implement-governance-and-controls-for-azure-cosmos-db-resources"></a>Uso de Azure Policy para implementar la gobernanza y los controles de recursos de Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

[Azure Policy](../governance/policy/overview.md) ayuda a aplicar estándares de gobernanza de la organización, a valorar el cumplimiento de los recursos y a implementar correcciones automáticas. Algunos casos de uso comunes son la seguridad, la administración de costos y la coherencia de la configuración.

Azure Policy proporciona definiciones de directivas integradas Puede crear definiciones de directivas personalizadas para escenarios en los que no sirvan las definiciones de directivas integradas. Para más información, consulte la [documentación de Azure Policy](../governance/policy/overview.md).

> [!IMPORTANT]
> Azure Policy se aplica a los servicios de Azure en el nivel de proveedor de recursos. Los SDK de Cosmos DB pueden realizar la mayoría de las operaciones de administración en los recursos de base de datos, contenedor y rendimiento que pasan por alto el proveedor de recursos de Cosmos DB, de forma que se omiten las directivas creadas mediante Azure Policy. Para garantizar la aplicación de las directivas, consulte [Bloqueo en los SDK de Azure Cosmos DB para evitar cambios](role-based-access-control.md#prevent-sdk-changes).

## <a name="assign-a-built-in-policy-definition"></a>Asignación de una definición de directiva integrada

Las definiciones de directivas describen las condiciones de cumplimiento de los recursos y qué sucederá si se cumple una condición. Las _asignaciones_ de directiva se crean a partir de _definiciones_ de directivas. Puede usar definiciones de directivas integradas o personalizadas con los recursos de Azure Cosmos DB. Las asignaciones de directiva se limitan a un grupo de administración de Azure, una suscripción de Azure o un grupo de recursos, y se aplican a los recursos dentro del ámbito seleccionado. Opcionalmente, puede excluir recursos específicos del ámbito.

Puede crear asignaciones de directiva con [Azure Portal](../governance/policy/assign-policy-portal.md), [Azure PowerShell](../governance/policy/assign-policy-powershell.md), la [CLI de Azure](../governance/policy/assign-policy-azurecli.md) o [plantillas de ARM](../governance/policy/assign-policy-template.md).

Para crear una asignación de directiva a partir de una definición de directiva integrada para Azure Cosmos DB, siga los pasos descritos en el artículo [Creación de una asignación de directiva con Azure Portal](../governance/policy/assign-policy-portal.md).

En el paso para seleccionar una definición de directiva, escriba `Cosmos DB` en el campo de búsqueda para filtrar la lista de definiciones de directivas integradas disponibles. Seleccione una de las definiciones de directivas integradas disponibles y, luego, elija **Seleccionar** para continuar con la creación de la asignación de directiva.

> [!TIP]
> También puede usar los nombres de definición de directiva integrada que se muestran en el panel **Definiciones disponibles** con Azure PowerShell, la CLI de Azure o plantillas de ARM para crear asignaciones de directivas.

:::image type="content" source="./media/policy/available-definitions.png" alt-text="Búsqueda de definiciones de directivas integradas de Azure Cosmos DB":::

## <a name="create-a-custom-policy-definition"></a>Creación de una definición de directiva personalizada

En el caso de escenarios específicos en los que no sirvan las directivas integradas, puede crear [una definición de directiva personalizada](../governance/policy/tutorials/create-custom-policy-definition.md). Más adelante creará una _asignación_ de directiva a partir de la _definición_ de directiva personalizada.

### <a name="property-types-and-property-aliases-in-policy-rules"></a>Tipos y alias de propiedades en reglas de directiva

Use los [pasos de definición de directiva personalizada](../governance/policy/tutorials/create-custom-policy-definition.md) para identificar las propiedades de recursos y los alias de propiedad, que son necesarios para crear reglas de directiva.

Para identificar alias de propiedad específicos de Azure Cosmos DB, use el espacio de nombres `Microsoft.DocumentDB` con uno de los métodos que se muestran en el artículo sobre los pasos de definición de directivas personalizadas.

#### <a name="use-the-azure-cli"></a>Uso de la CLI de Azure:
```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Get Azure Policy aliases for namespace Microsoft.DocumentDB
az provider show --namespace Microsoft.DocumentDB --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
```

#### <a name="use-azure-powershell"></a>Uso de Azure PowerShell:
```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Use Get-AzPolicyAlias to list aliases for Microsoft.DocumentDB namespace
(Get-AzPolicyAlias -NamespaceMatch 'Microsoft.DocumentDB').Aliases
```

Estos comandos generan la lista de nombres de alias de propiedad para propiedades de Azure Cosmos DB. A continuación, se muestra un extracto de la salida:

```json
[
  "Microsoft.DocumentDB/databaseAccounts/sku.name",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*]",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*].id",
  "Microsoft.DocumentDB/databaseAccounts/isVirtualNetworkFilterEnabled",
  "Microsoft.DocumentDB/databaseAccounts/consistencyPolicy.defaultConsistencyLevel",
  "Microsoft.DocumentDB/databaseAccounts/enableAutomaticFailover",
  "Microsoft.DocumentDB/databaseAccounts/Locations",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*]",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*].locationName",
  "..."
]
```

Puede usar cualquiera de estos nombres de alias de propiedad en las [reglas de definición de directiva personalizada](../governance/policy/tutorials/create-custom-policy-definition.md#policy-rule).

El siguiente es un ejemplo de definición de directiva que comprueba si una cuenta de Azure Cosmos DB está configurada para varias ubicaciones de escritura. Una definición de directiva personalizada incluye dos reglas: una para comprobar el tipo específico de alias de propiedad y la otra para la propiedad específica del tipo, en este caso el campo que almacena la configuración de varias ubicaciones de escritura. Ambas reglas usan los nombres de alias.

```json
"policyRule": {
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.DocumentDB/databaseAccounts"
      },
      {
        "field": "Microsoft.DocumentDB/databaseAccounts/enableMultipleWriteLocations",
        "notEquals": true
      }
    ]
  },
  "then": {
    "effect": "Audit"
  }
}
```

Se pueden usar definiciones de directivas personalizadas para crear asignaciones de directiva de la misma manera que se usan las definiciones de directivas integradas.

## <a name="policy-compliance"></a>Cumplimiento de directivas

Una vez creadas las asignaciones de directiva, Azure Policy evalúa los recursos en el ámbito de la asignación. Se valora el _cumplimiento_ de cada recurso con la directiva. El _efecto_ especificado en la directiva se aplica luego a los recursos no compatibles.

Puede revisar los resultados de cumplimiento y los detalles de corrección en [Azure Portal](../governance/policy/how-to/get-compliance-data.md#portal), la [CLI de Azure](../governance/policy/how-to/get-compliance-data.md#command-line) o los [registros de Azure Monitor](../governance/policy/how-to/get-compliance-data.md#azure-monitor-logs).

En la captura de pantalla siguiente se muestran dos ejemplos de asignaciones de directiva.

Una asignación se basa en una definición de directiva integrada, que comprueba que los recursos de Azure Cosmos DB se implementan solo en las regiones de Azure permitidas. La compatibilidad de recursos muestra el resultado de la evaluación de las directivas (compatible o no compatible) para los recursos del ámbito.

La otra asignación se basa en una definición de directiva personalizada. Esta asignación comprueba que las cuentas de Cosmos DB están configuradas para varias ubicaciones de escritura.

Una vez implementadas las asignaciones de directiva, el panel de cumplimiento muestra los resultados de la evaluación. Tenga en cuenta que esta operación puede tardar hasta 30 minutos después de implementar una asignación de directiva. Además, los [análisis de evaluación de directivas se pueden iniciar a petición](../governance/policy/how-to/get-compliance-data.md#on-demand-evaluation-scan) inmediatamente después de crear asignaciones de directivas.

En la captura de pantalla se muestran los siguientes resultados de la evaluación de cumplimiento para las cuentas de Azure Cosmos DB del ámbito:

- Cero de dos cuentas cumple con una directiva por la que debe estar configurado el filtrado de Virtual Network (VNet).
- Cero de dos cuentas cumple con una directiva que requiere que la cuenta esté configurada para varias ubicaciones de escritura.
- Cero de dos cuentas cumple con una directiva por la que los recursos deben estar implementados en regiones de Azure permitidas.

:::image type="content" source="./media/policy/compliance.png" alt-text="Resultados de cumplimiento de las asignaciones de Azure Policy enumeradas":::

Para corregir los recursos no compatibles, consulte [Corregir los recursos no conformes con Azure Policy](../governance/policy/how-to/remediate-resources.md).

## <a name="next-steps"></a>Pasos siguientes

- [Revise las definiciones de directivas personalizadas de ejemplo para Azure Cosmos DB](https://github.com/Azure/azure-policy/tree/master/samples/CosmosDB), incluidas las directivas de filtrado de VNet y de varias ubicaciones de escritura mostradas anteriormente.
- [Creación de una asignación de directiva en Azure Portal](../governance/policy/assign-policy-portal.md)
- [Revisión de las definiciones de directivas integradas de Azure Policy para Azure Cosmos DB](./policy-reference.md)
