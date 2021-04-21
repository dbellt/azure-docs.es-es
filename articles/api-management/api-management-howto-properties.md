---
title: Cómo usar valores con nombre en las directivas de Azure API Management
description: Aprenda a usar valores con nombre en las directivas de Azure API Management. Los valores con nombre pueden contener cadenas literales, expresiones de directiva y secretos almacenados en Azure Key Vault.
services: api-management
documentationcenter: ''
author: vladvino
ms.service: api-management
ms.topic: article
ms.date: 02/09/2021
ms.author: apimpm
ms.openlocfilehash: b0e076f3b248942870ba58a51c85c3df1f1277a4
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750615"
---
# <a name="use-named-values-in-azure-api-management-policies"></a>Uso de valores con nombre en las directivas de Azure API Management

Las [directivas de API Management](api-management-howto-policies.md) constituyen una eficaz funcionalidad del sistema que permite al editor cambiar el comportamiento de la API a través de la configuración. Las directivas son una colección de declaraciones que se ejecutan secuencialmente en la solicitud o respuesta de una API. Las instrucciones de las directivas se pueden crear con valores de texto literal, expresiones de directiva y valores con nombre.

Los *valores con nombre* son una colección global de pares nombre-valor en cada instancia de API Management. No se impone ningún límite en el número de elementos de la colección. Los valores con nombre se pueden usar para administrar secretos y valores de cadena constantes en todas las directivas y la configuración de API. 

:::image type="content" source="media/api-management-howto-properties/named-values.png" alt-text="Valores con nombre en Azure Portal":::

## <a name="value-types"></a>Tipos de valor

|Tipo  |Descripción  |
|---------|---------|
|Normal     |  Cadena literal o expresión de directiva     |
|Secreto     |   Cadena literal o expresión de directiva cifrada por API Management      |
|[Key vault](#key-vault-secrets)     |  Identificador de un secreto almacenado en una instancia de Azure Key Vault.      |

Los valores normales o los secretos pueden contener [expresiones de directiva](./api-management-policy-expressions.md). Por ejemplo, la expresión `@(DateTime.Now.ToString())` devuelve una cadena que contiene la fecha y la hora actuales.

Para obtener más información sobre los atributos de los valores con nombre, consulte la [referencia de la API REST](/rest/api/apimanagement/2020-06-01-preview/namedvalue/createorupdate) de API Management.

## <a name="key-vault-secrets"></a>Secretos de un almacén de claves

Los valores de los secretos se pueden almacenar como cadenas cifradas en API Management (secretos personalizados) o mediante referencia a secretos en [Azure Key Vault](../key-vault/general/overview.md). 

Se recomienda el uso de secretos de Key Vault, ya que esto ayuda a mejorar la seguridad de API Management:

* Los secretos almacenados en almacenes de claves se pueden reutilizar entre servicios
* Se pueden aplicar [directivas de acceso](../key-vault/general/security-overview.md#privileged-access) pormenorizadas a los secretos
* Los secretos actualizados en el almacén de claves se rotan automáticamente en API Management. Después de la actualización en el almacén de claves, un valor con nombre en API Management se actualiza en un plazo de 4 horas. También puede actualizar manualmente el secreto mediante Azure Portal o a través de la API REST de administración.

### <a name="prerequisites-for-key-vault-integration"></a>Requisitos previos para la integración de un almacén de claves

1. Para conocer los pasos para crear un almacén de claves, vea [Inicio rápido: Creación de un almacén de claves mediante Azure Portal](../key-vault/general/quick-create-portal.md).
1. Habilite una [identidad administrada](api-management-howto-use-managed-service-identity.md) asignada por el sistema o por el usuario en la instancia de API Management.
1. Asigne una [directiva de acceso del almacén de claves](../key-vault/general/assign-access-policy-portal.md) a la identidad administrada con permisos para obtener y enumerar los secretos del almacén. Para agregar la directiva:
    1. En el portal, vaya al almacén de claves.
    1. Seleccione **Configuración > Directivas de acceso > +Agregar directiva de acceso**.
    1. Seleccione **Permisos de los secretos**, seleccione **Obtener** y **Enumerar**.
    1. En **Seleccionar la entidad de seguridad**, seleccione el nombre de recurso de la identidad administrada. Si usa una identidad asignada por el sistema, la entidad de seguridad es el nombre de la instancia de API Management.
1. Cree un secreto o importe uno al almacén de claves. Consulte [Quickstart: Establecimiento y recuperación de un secreto de Azure Key Vault mediante Azure Portal](../key-vault/secrets/quick-create-portal.md).

Para usar el secreto del almacén de claves, [agregue o edite un valor con nombre](#add-or-edit-a-named-value) y especifique un tipo de **almacén de claves**. Seleccione el secreto del almacén de claves.

[!INCLUDE [api-management-key-vault-network](../../includes/api-management-key-vault-network.md)]

## <a name="add-or-edit-a-named-value"></a>Adición o edición de un valor con nombre

### <a name="add-a-key-vault-secret"></a>Adición de un secreto de almacén de claves

Consulte [Requisitos previos para la integración de un almacén de claves](#prerequisites-for-key-vault-integration).

> [!CAUTION]
> Al usar un secreto del almacén de claves en API Management, tenga cuidado de no eliminar el secreto, el almacén de claves ni la identidad administrada que se usa para acceder al almacén de claves.

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. En **API**, seleccione **Calores con nombre** >  **+Agregar**.
1. Escriba un identificador de **Nombre** y escriba un **Nombre para mostrar** que se use para hacer referencia a la propiedad en las directivas.
1. En **Tipo de valor**, seleccione **Almacén de claves**.
1. Escriba el identificador de un secreto de almacén de claves (sin versión) o elija **Seleccionar** para seleccionar un secreto de un almacén de claves.
    > [!IMPORTANT]
    > Si especifica usted mismo el identificador de un secreto del almacén de claves, asegúrese de que no tenga la información de versión. De lo contrario, el secreto no se rotará automáticamente en API Management después de una actualización en el almacén de claves.
1. En **Client identity** (Identidad del cliente), seleccione una identidad asignada por el sistema o una identidad administrada existente asignada por el usuario. Obtenga información sobre cómo [agregar o modificar identidades administradas en el servicio de API Management](api-management-howto-use-managed-service-identity.md).
    > [!NOTE]
    > La identidad necesita permisos para obtener y enumerar los secretos del almacén de claves. Si aún no ha configurado el acceso al almacén de claves, API Management se lo pedirá para poder configurar automáticamente la identidad con los permisos necesarios.
1. Agregue una o varias etiquetas opcionales para ayudar a organizar los valores con nombre y, a continuación, elija **Guardar**.
1. Seleccione **Crear**.

    :::image type="content" source="media/api-management-howto-properties/add-property.png" alt-text="Adición de un valor de secreto del almacén de claves":::

### <a name="add-a-plain-or-secret-value"></a>Adición de un valor normal o secreto

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. En **API**, seleccione **Calores con nombre** >  **+Agregar**.
1. Escriba un identificador de **Nombre** y escriba un **Nombre para mostrar** que se use para hacer referencia a la propiedad en las directivas.
1. En **Tipo de valor**, seleccione **Normal** o **Secreto**.
1. En **Valor**, escriba una cadena o expresión de directiva.
1. Agregue una o varias etiquetas opcionales para ayudar a organizar los valores con nombre y, a continuación, elija **Guardar**.
1. Seleccione **Crear**.

Una vez creado el valor con nombre, selecciónelo para editarlo. Si cambia el nombre para mostrar, las directivas que hagan referencia a ese valor con nombre se actualizarán automáticamente para utilizar el nuevo nombre para mostrar.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para empezar a usar la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Para agregar un valor con nombre, use el comando [az apim nv create](/cli/azure/apim/nv#az_apim_nv_create):

```azurecli
az apim nv create --resource-group apim-hello-word-resource-group \
    --display-name "named_value_01" --named-value-id named_value_01 \
    --secret true --service-name apim-hello-world --value test
```

Una vez creado un valor con nombre, puede actualizarlo mediante el comando [az apim nv update](/cli/azure/apim/nv#az_apim_nv_update). Para ver todos los valores con nombre, ejecute el comando [az apim nv list](/cli/azure/apim/nv#az_apim_nv_list):

```azurecli
az apim nv list --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --output table
```

Para ver los detalles del valor con nombre que creó en este ejemplo, ejecute el comando [az apim nv show](/cli/azure/apim/nv#az_apim_nv_show):

```azurecli
az apim nv show --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --named-value-id named_value_01
```

Este ejemplo es un valor secreto. El comando anterior no devuelve el valor. Para ver el valor, ejecute el comando [az apim nv show-secret](/cli/azure/apim/nv#az_apim_nv_show_secret):

```azurecli
az apim nv show-secret --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --named-value-id named_value_01
```

Para eliminar un valor con nombre, use el comando [az apim nv delete](/cli/azure/apim/nv#az_apim_nv_delete):

```azurecli
az apim nv delete --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --named-value-id named_value_01
```

---

## <a name="use-a-named-value"></a>Uso de un valor con nombre

En los ejemplos de esta sección se usan los valores con nombre que aparecen en la tabla siguiente.

| Nombre               | Value                      | Secreto | 
|--------------------|----------------------------|--------|---------|
| ContosoHeader      | `TrackingId`                 | Falso  | 
| ContosoHeaderValue | ••••••••••••••••••••••     | True   | 
| ExpressionProperty | `@(DateTime.Now.ToString())` | Falso  | 

Para utilizar un valor con nombre en una directiva, coloque su nombre para mostrar dentro de un par doble de llaves (como `{{ContosoHeader}}`), de la misma forma que se muestra en el ejemplo siguiente:

```xml
<set-header name="{{ContosoHeader}}" exists-action="override">
  <value>{{ContosoHeaderValue}}</value>
</set-header>
```

En este ejemplo, `ContosoHeader` se utiliza como el nombre de un encabezado en una directiva `set-header`, y `ContosoHeaderValue` se utiliza como valor de ese encabezado. Cuando esta directiva se evalúa durante una solicitud o responde a la pasarela de API Management, `{{ContosoHeader}}` y `{{ContosoHeaderValue}}` se reemplazan por sus respectivos valores.

Los valores con nombre se pueden utilizar como atributo completo o como valores de elemento (tal como se muestra en el ejemplo anterior), pero también se pueden insertar o combinar con parte de una expresión de texto literal, como en el ejemplo siguiente:  

```xml
<set-header name = "CustomHeader{{ContosoHeader}}" ...>
```

Los valores con nombre también pueden contener expresiones de directiva. En el ejemplo siguiente, se usa la expresión `ExpressionProperty`.

```xml
<set-header name="CustomHeader" exists-action="override">
    <value>{{ExpressionProperty}}</value>
</set-header>
```

Cuando se evalúa esta directiva, se reemplaza `{{ExpressionProperty}}` por su valor, `@(DateTime.Now.ToString())`. Puesto que el valor es una expresión de directiva, la expresión se evalúa y la directiva continúa con su ejecución.

Para probarlo en Azure Portal o en el [portal para desarrolladores](api-management-howto-developer-portal.md), puede llamar a una operación que tenga dentro de su ámbito una directiva con valores con nombre. En el ejemplo siguiente, se llama a una operación con las dos directivas `set-header` de ejemplo anteriores con valores con nombre. Tenga en cuenta que la respuesta contiene dos encabezados personalizados que se han configurado mediante directivas con valores con nombre.

:::image type="content" source="media/api-management-howto-properties/api-management-send-results.png" alt-text="Prueba de la respuesta de la API":::

Si busca en el [seguimiento de API](api-management-howto-api-inspector.md) de salida una llamada que incluya las dos directivas de ejemplo anteriores con valores con nombre, verá las dos directivas `set-header` con los valores con nombre insertados, así como la evaluación de la expresión de directiva para el valor con nombre que la contiene.

:::image type="content" source="media/api-management-howto-properties/api-management-api-inspector-trace.png" alt-text="Seguimiento de API Inspector":::

> [!CAUTION]
> Si una directiva hace referencia a un secreto en Azure Key Vault, el valor del almacén de claves será visible para los usuarios que tengan acceso a las suscripciones habilitadas para el [seguimiento de solicitudes de API](api-management-howto-api-inspector.md).

Aunque los valores con nombre pueden contener expresiones de directiva, no pueden contener otros valores con nombre. En caso de usarse texto que contenga una referencia a valores con nombre para un valor, como `Text: {{MyProperty}}`, dicha referencia no se resolverá ni reemplazará.

## <a name="delete-a-named-value"></a>Eliminación de un valor con nombre

Para eliminar un valor con nombre, seleccione el nombre y, a continuación, seleccione **Eliminar** en el menú contextual ( **…** ).

> [!IMPORTANT]
> Si alguna directiva de API Management hace referencia al valor con nombre, no podrá eliminarlo hasta que lo quite de todas las directivas que lo usen.

## <a name="next-steps"></a>Pasos siguientes

-   Obtenga más información sobre cómo trabajar con directivas
    -   [Directivas de Azure API Management](api-management-howto-policies.md)
    -   [Referencia de la directiva](./api-management-policies.md)
    -   [Expresiones de directiva](./api-management-policy-expressions.md)

[api-management-send-results]: ./media/api-management-howto-properties/api-management-send-results.png

