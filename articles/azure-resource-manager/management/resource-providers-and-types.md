---
title: Proveedores de recursos y tipos de recursos
description: Describe los proveedores de recursos compatibles con Azure Resource Manager. Describe sus esquemas, versiones de API disponibles y las regiones que pueden hospedar los recursos.
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: d5936e1e4c80869495d5b71023e26264dc9a2cde
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319204"
---
# <a name="azure-resource-providers-and-types"></a>Tipos y proveedores de recursos de Azure

Al implementar los recursos, con frecuencia necesitará recuperar información sobre los tipos y proveedores de recursos. Por ejemplo, si quiere almacenar claves y secretos, trabajará con el proveedor de recursos Microsoft.KeyVault. Este proveedor de recursos ofrece un tipo de recurso denominado almacenes para crear el almacén de claves.

El nombre de un tipo de recurso tiene el formato: **{proveedor de recursos}/{tipo de recurso}** . El tipo de recurso para un almacén de claves es **Microsoft.KeyVault/vaults**.

En este artículo aprenderá a:

* Ver todos los proveedores de recursos de Azure
* Comprobar el estado de registro de un proveedor de recursos
* Registrar un proveedor de recursos
* Ver los tipos de recursos de un proveedor
* Ver las ubicaciones válidas de un tipo de recurso
* Ver las versiones de API válidas de un tipo de recurso

Puede llevar a cabo estos pasos en Azure Portal, Azure PowerShell o la CLI de Azure.

Para obtener una lista que asigna proveedores de recursos con servicios de Azure, consulte [Proveedores de recursos para servicios de Azure](azure-services-resource-providers.md).

## <a name="register-resource-provider"></a>Registro del proveedor de recursos

Antes de usar un proveedor de recursos, la suscripción de Azure debe registrarse para el proveedor de recursos. El registro configura su suscripción para así poder trabajar con el proveedor de recursos. Algunos proveedores de recursos están registrados de forma predeterminada. Para obtener una lista de los proveedores de recursos registrados de forma predeterminada, consulte [Proveedores de recursos para servicios de Azure](azure-services-resource-providers.md).

Otros proveedores de recursos están registrados automáticamente cuando toma determinadas medidas. Cuando implementa una plantilla de Azure Resource Manager, se registran automáticamente todos los proveedores de recursos necesarios. Cuando crea un recurso a través del portal, el proveedor de recursos normalmente se registra de manera automática. En otros escenarios, es posible que tenga que registrar manualmente un proveedor de recursos. 

En este artículo se muestra cómo comprobar el estado del registro de un proveedor de recursos y registrarlo según sea necesario. Debe tener permiso para realizar la operación `/register/action` para el proveedor de recursos. El permiso se incluye en los roles Colaborador y Propietario.

> [!IMPORTANT]
> Solo debe registrar un proveedor de recursos cuando esté listo para usarlo. El paso de registro permite mantener los privilegios mínimos dentro de la suscripción. Un usuario malintencionado no puede usar proveedores de recursos que no están registrados.

El código de aplicación no debe bloquear la creación de recursos para un proveedor de recursos con el estado de **registro**. Al registrar el proveedor de recursos, la operación se realiza de forma individual para cada región admitida. Para crear recursos en una región, el registro solo debe completarse en dicha región. Si no se bloquea el proveedor de recursos con el estado de registro, la aplicación puede continuar mucho antes que si se espera a que se completen todas las regiones.

No se puede anular el registro de un proveedor de recursos si todavía dispone de tipos de recursos de dicho proveedor en la suscripción.

## <a name="azure-portal"></a>Portal de Azure

### <a name="register-resource-provider"></a>Registro del proveedor de recursos

Para ver todos los proveedores de recursos y el estado de registro de su suscripción:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú de Azure Portal, busque **Suscripciones**. Seleccione el servicio entre las opciones disponibles.

   :::image type="content" source="./media/resource-providers-and-types/search-subscriptions.png" alt-text="Búsqueda de suscripciones":::

1. Seleccione la suscripción que quiere ver.

   :::image type="content" source="./media/resource-providers-and-types/select-subscription.png" alt-text="selección de suscripciones":::

1. En el menú de la izquierda, en **Configuración**, seleccione **Proveedores de recursos**.

   :::image type="content" source="./media/resource-providers-and-types/select-resource-providers.png" alt-text="Selección de proveedores de recursos":::

6. Busque el proveedor de recursos que desea registrar y seleccione **Registrar**. Para mantener los privilegios mínimos en su suscripción, registre solo los proveedores de recursos que esté listo para usar.

   :::image type="content" source="./media/resource-providers-and-types/register-resource-provider.png" alt-text="Registro de proveedores de recursos":::

### <a name="view-resource-provider"></a>Visualización del proveedor de recursos

Para ver información de un proveedor de recursos concreto:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú de Azure Portal, seleccione **Todos los servicios**.
3. En el cuadro **Todos los servicios**, escriba **Explorador de recursos** y, a continuación, seleccione **Explorador de recursos**.

    ![seleccione Todos los servicios](./media/resource-providers-and-types/select-resource-explorer.png)

4. Expanda **Proveedores**; para ello, seleccione la flecha derecha.

    ![Selección de proveedores](./media/resource-providers-and-types/select-providers.png)

5. Expanda un proveedor de recursos y el tipo de recurso que desea ver.

    ![Selección del tipo de recurso](./media/resource-providers-and-types/select-resource-type.png)

6. El Administrador de recursos se admite en todas las regiones, pero puede que los recursos que implementa no se admitan en todas las regiones. Además, puede haber limitaciones en su suscripción que le impidan utilizar algunas regiones que admiten el recurso. El explorador de recursos muestra las ubicaciones válidas para el tipo de recurso.

    ![Vista de las ubicaciones](./media/resource-providers-and-types/show-locations.png)

7. La versión de API se corresponde a una versión de operaciones de API de REST que se publican por el proveedor de recursos. Conforme un proveedor de recursos habilite nuevas características, publicará una nueva versión de la API de REST. El explorador de recursos muestra las versiones de API válidas para el tipo de recurso.

    ![Vista de las versiones de API](./media/resource-providers-and-types/show-api-versions.png)

## <a name="azure-powershell"></a>Azure PowerShell

Para ver todos los proveedores de recursos de Azure y el estado de registro de su suscripción, use:

```azurepowershell-interactive
Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
```

Que devuelve resultados similares a:

```output
ProviderNamespace                RegistrationState
-------------------------------- ------------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

Para ver todos los proveedores de recursos registrados para la suscripción, use:

```azurepowershell-interactive
 Get-AzResourceProvider -ListAvailable | Where-Object RegistrationState -eq "Registered" | Select-Object ProviderNamespace, RegistrationState | Sort-Object ProviderNamespace
```

Para mantener los privilegios mínimos en su suscripción, registre solo los proveedores de recursos que esté listo para usar. Para registrar un proveedor de recursos, use:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
```

Que devuelve resultados similares a:

```output
ProviderNamespace : Microsoft.Batch
RegistrationState : Registering
ResourceTypes     : {batchAccounts, operations, locations, locations/quotas}
Locations         : {West Europe, East US, East US 2, West US...}
```

Para ver información de un proveedor de recursos concreto, use:

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Batch
```

Que devuelve resultados similares a:

```output
{ProviderNamespace : Microsoft.Batch
RegistrationState : Registered
ResourceTypes     : {batchAccounts}
Locations         : {West Europe, East US, East US 2, West US...}

...
```

Para ver los tipos de recursos de un proveedor, use:

```azurepowershell-interactive
(Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes.ResourceTypeName
```

Que devuelve:

```output
batchAccounts
operations
locations
locations/quotas
```

La versión de API se corresponde a una versión de operaciones de API de REST que se publican por el proveedor de recursos. Conforme un proveedor de recursos habilite nuevas características, publicará una nueva versión de la API de REST.

Para obtener las versiones de API de un tipo de recurso, use:

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).ApiVersions
```

Que devuelve:

```output
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

El Administrador de recursos se admite en todas las regiones, pero puede que los recursos que implementa no se admitan en todas las regiones. Además, puede haber limitaciones en su suscripción que le impidan utilizar algunas regiones que admiten el recurso.

Para obtener las ubicaciones compatibles con un tipo de recurso, use:

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

Que devuelve:

```output
West Europe
East US
East US 2
West US
...
```

## <a name="azure-cli"></a>Azure CLI

Para ver todos los proveedores de recursos de Azure y el estado de registro de su suscripción, use:

```azurecli-interactive
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

Que devuelve resultados similares a:

```output
Provider                         Status
-------------------------------- ----------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

Para ver todos los proveedores de recursos registrados para la suscripción, use:

```azurecli-interactive
az provider list --query "sort_by([?registrationState=='Registered'].{Provider:namespace, Status:registrationState}, &Provider)" --out table
```

Para mantener los privilegios mínimos en su suscripción, registre solo los proveedores de recursos que esté listo para usar. Para registrar un proveedor de recursos, use:

```azurecli-interactive
az provider register --namespace Microsoft.Batch
```

Que devuelve un mensaje que indica que el registro está en curso.

Para ver información de un proveedor de recursos concreto, use:

```azurecli-interactive
az provider show --namespace Microsoft.Batch
```

Que devuelve resultados similares a:

```output
{
    "id": "/subscriptions/####-####/providers/Microsoft.Batch",
    "namespace": "Microsoft.Batch",
    "registrationsState": "Registering",
    "resourceTypes:" [
        ...
    ]
}
```

Para ver los tipos de recursos de un proveedor, use:

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[*].resourceType" --out table
```

Que devuelve:

```output
Result
---------------
batchAccounts
operations
locations
locations/quotas
```

La versión de API se corresponde a una versión de operaciones de API de REST que se publican por el proveedor de recursos. Conforme un proveedor de recursos habilite nuevas características, publicará una nueva versión de la API de REST.

Para obtener las versiones de API de un tipo de recurso, use:

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].apiVersions | [0]" --out table
```

Que devuelve:

```output
Result
---------------
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

El Administrador de recursos se admite en todas las regiones, pero puede que los recursos que implementa no se admitan en todas las regiones. Además, puede haber limitaciones en su suscripción que le impidan utilizar algunas regiones que admiten el recurso.

Para obtener las ubicaciones compatibles con un tipo de recurso, use:

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" --out table
```

Que devuelve:

```output
Result
---------------
West Europe
East US
East US 2
West US
...
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la creación de plantillas del Administrador de recursos, consulte [Creación de plantillas del Administrador de recursos de Azure](../templates/template-syntax.md). 
* Para ver los esquemas de plantilla de proveedor de recursos, consulte [Referencia de plantilla](/azure/templates/).
* Para obtener una lista que asigna proveedores de recursos con servicios de Azure, consulte [Proveedores de recursos para servicios de Azure](azure-services-resource-providers.md).
* Para ver las operaciones de un proveedor de recursos, consulte [API de REST de Azure](/rest/api/).
