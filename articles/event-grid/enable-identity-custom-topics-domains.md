---
title: Habilitación de la identidad administrada en temas y dominios personalizados de Azure Event Grid
description: En este artículo se describe cómo habilitar una identidad de servicio administrada de un tema o un dominio personalizados de Azure Event Grid.
ms.topic: how-to
ms.date: 03/25/2021
ms.openlocfilehash: b93fd44282d9e19e7111dd52c73d8d4c01c67a10
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106278225"
---
# <a name="assign-a-system-managed-identity-to-an-event-grid-custom-topic-or-domain"></a>Asignación de una identidad administrada por el sistema a un tema o un dominio personalizados de Event Grid 
En este artículo se muestra cómo habilitar una identidad administrada por el sistema para un tema o un dominio personalizados de Event Grid. Para más información sobre las identidades administradas, consulte [¿Qué son las identidades administradas de recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="enable-identity-at-the-time-of-creation"></a>Habilitación de la identidad en el momento de la creación

### <a name="using-azure-portal"></a>Uso de Azure Portal
Puede habilitar una identidad asignada por el sistema para un tema o un dominio personalizados mientras la crea en Azure Portal. En la siguiente imagen se muestra cómo habilitar una identidad administrada por el sistema para un tema personalizado. Básicamente, se selecciona la opción **Enable system assigned identity** (Habilitar identidad asignada por el sistema) en la página de **opciones avanzadas** del Asistente para crear temas. También verá esta opción en la página de **opciones avanzadas** del Asistente para crear dominios. 

![Habilitación de la identidad durante la creación de un tema personalizado](./media/managed-service-identity/create-topic-identity.png)

### <a name="using-azure-cli"></a>Uso de la CLI de Azure
También puede usar la CLI de Azure para crear un tema o un dominio personalizados con una identidad asignada por el sistema. Use el comando `az eventgrid topic create` con el parámetro `--identity` establecido en `systemassigned`. Si no especifica un valor para este parámetro, se utiliza el valor predeterminado `noidentity`. 

```azurecli-interactive
# create a custom topic with a system-assigned identity
az eventgrid topic create -g <RESOURCE GROUP NAME> --name <TOPIC NAME> -l <LOCATION>  --identity systemassigned
```

Del mismo modo, puede usar el comando `az eventgrid domain create` para crear un dominio con una identidad administrada por el sistema.

## <a name="enable-identity-for-an-existing-custom-topic-or-domain"></a>Habilitación de una identidad para un tema o dominio personalizados existentes
En esta sección, aprenderá a habilitar una identidad administrada por el sistema para un tema o un dominio personalizados existentes. 

### <a name="using-azure-portal"></a>Uso de Azure Portal
El siguiente procedimiento muestra cómo habilitar una identidad administrada por el sistema para un tema personalizado. Los pasos para habilitar una identidad para un dominio son similares. 

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Busque **temas de Event Grid** en la barra de búsqueda de la parte superior.
3. Seleccione el **tema personalizado** para el que quiere habilitar la identidad administrada. 
4. Cambie a la pestaña **Identidad**. 
5. **Active** el conmutador para habilitar la identidad. 
1. Seleccione **Guardar** en la barra de herramientas para guardar la configuración. 

    :::image type="content" source="./media/managed-service-identity/identity-existing-topic.png" alt-text="Página de identidad para un tema personalizado"::: 

Puede usar pasos similares para habilitar una identidad de un dominio de Event Grid.

### <a name="use-the-azure-cli"></a>Uso de la CLI de Azure
Use el comando `az eventgrid topic update` con `--identity` establecido en `systemassigned` para habilitar la identidad asignada por el sistema para un tema personalizado existente. Si desea deshabilitar la identidad, especifique `noidentity` como valor. 

```azurecli-interactive
# Update the topic to assign a system-assigned identity. 
az eventgrid topic update -g $rg --name $topicname --identity systemassigned --sku basic 
```

El comando para actualizar un dominio existente es similar (`az eventgrid domain update`).


## <a name="next-steps"></a>Pasos siguientes
Agregue la identidad a un rol adecuado (por ejemplo, Remitente de los datos de Service Bus) en el destino (por ejemplo, una cola de Service Bus). Para los pasos detallados, consulte [Concesión a la identidad administrada el acceso al destino de Event Grid](add-identity-roles.md). 
