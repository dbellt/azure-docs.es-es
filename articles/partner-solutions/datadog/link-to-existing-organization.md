---
title: 'Creación de un vínculo a una instancia de Datadog existente: soluciones de asociados de Azure'
description: En este artículo se describe cómo usar Azure Portal para crear un vínculo a una instancia existente de Datadog.
ms.service: partner-services
ms.topic: quickstart
ms.date: 05/28/2021
author: tfitzmac
ms.author: tomfitz
ms.custom: references_regions
ms.openlocfilehash: b142b7d6e34905fe02de2a7546f9265289016808
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110657168"
---
# <a name="quickstart-link-to-existing-datadog-organization"></a>Inicio rápido: Creación de un vínculo a una organización de Datadog existente

En este inicio rápido, se creará un vínculo a una organización existente de Datadog. Puede [crear una organización de Datadog o un vínculo a una ya existente](create.md).

## <a name="prerequisites"></a>Prerrequisitos

Antes de crear la primera instancia de Datadog en Azure, [configure el entorno](prerequisites.md). Estos pasos deben completarse antes de continuar con los pasos siguientes de este inicio rápido.

## <a name="find-offer"></a>Búsqueda de la oferta

Use Azure Portal para encontrar Datadog.

1. Abra [Azure Portal](https://portal.azure.com/) e inicie sesión.

1. Si ha visitado el **Marketplace** en una sesión reciente, seleccione el icono entre las opciones disponibles. En caso contrario, busque _Marketplace_.

    :::image type="content" source="media/link-to-existing-organization/marketplace.png" alt-text="Icono de Marketplace.":::

1. En Marketplace, busque **Datadog**.

1. En la pantalla de información general del plan, seleccione **Configurar + suscribir**.

   :::image type="content" source="media/link-to-existing-organization/datadog-app-2.png" alt-text="Aplicación Datadog en Azure Marketplace.":::

## <a name="link-to-existing-datadog-organization"></a>Vínculo a un organización de Datadog existente

El portal muestra una selección que le pregunta si desea crear una organización de Datadog o vincular una suscripción de Azure a una organización de Datadog existente.

Si va a crear un vínculo a una organización de Datadog existente, seleccione **Crear** en **Link Azure subscription to an existing Datadog organization** (Vincular una suscripción de Azure a una organización de Datadog existente).

:::image type="content" source="media/link-to-existing-organization/datadog-create-link-selection.png" alt-text="Creación o vinculación de una organización de Datadog" border="true":::

Puede vincular el nuevo recurso de Datadog en Azure a una organización de Datadog existente en **US3**.

El portal muestra un formulario para crear el recurso Datadog.

:::image type="content" source="media/link-to-existing-organization/link-to-existing.png" alt-text="Vincular a una organización de Datadog existente." border="true":::

Especifique los valores siguientes.

|Propiedad | Descripción
|:-----------|:-------- |
| Subscription | Seleccione la suscripción de Azure que desea usar para crear el recurso Datadog. Debe tener acceso de propietario. |
| Resource group | Especifique si desea crear un nuevo grupo de recursos o utilizar uno existente. Un [grupo de recursos](../../azure-resource-manager/management/overview.md#resource-groups) es un contenedor que almacena los recursos relacionados de una solución de Azure. |
| Nombre del recurso | Especifique un nombre para el recurso Datadog. Este nombre será el de la nueva organización de Datadog cuando se cree. |
| Location | Seleccione Oeste de EE. UU. 2. Actualmente, la única región admitida es Oeste de EE. UU. 2. |

Seleccione **Link to Datadog organization** (Vincular a una organización de Datadog). El vínculo abre una ventana de autenticación de Datadog. Inicie sesión en Datadog.

De forma predeterminada, Azure vincula la organización de Datadog actual con el recurso de Datadog. Si desea vincular a otra organización, seleccione la organización adecuada en la ventana de autenticación, como se muestra a continuación.

:::image type="content" source="media/link-to-existing-organization/select-datadog-organization.png" alt-text="Seleccionar la organización de Datadog adecuada para vincular" border="true":::

Seleccione **Siguiente: Métricas y registros** para configurar métricas y registros.

## <a name="configure-metrics-and-logs"></a>Configuración de métricas y registros

Use las etiquetas de recursos de Azure para configurar qué métricas y registros se envían a Datadog. Puede incluir o excluir las métricas y los registros de recursos concretos.

Las reglas de etiquetas para enviar **métricas** son:

- De forma predeterminada, las métricas se recopilan para todos los recursos, excepto las máquinas virtuales, los conjuntos de escalado de máquinas virtuales y los planes de App Service.
- Las máquinas virtuales, los conjuntos de escalado de máquinas virtuales y los planes de App Service con etiquetas *Include* envían métricas a Datadog.
- Las máquinas virtuales, los conjuntos de escalado de máquinas virtuales y los planes de App Service con etiquetas *Exclude* no envían métricas a Datadog.
- Si hay un conflicto entre las reglas de inclusión y exclusión, la exclusión tiene prioridad.

Las reglas de etiquetas para el envío de **registros** son:

- De forma predeterminada, se recopilan registros para todos los recursos.
- Los recursos de Azure con etiquetas *Include* envían registros a Datadog.
- Los recursos de Azure con etiquetas *Exclude* no envían registros a Datadog.
- Si hay un conflicto entre las reglas de inclusión y exclusión, la exclusión tiene prioridad.

Por ejemplo, en la siguiente captura de pantalla se muestra una regla de etiqueta en la que solo esas máquinas virtuales, los conjuntos de escalado de máquinas virtuales y los planes de App Service etiquetados como *Datadog = True* envían métricas a Datadog.

:::image type="content" source="media/link-to-existing-organization/config-metrics-logs.png" alt-text="Configurar registros y métricas" border="true":::.

Hay dos tipos de registros que se pueden emitir desde Azure a Datadog.

* **Registros de nivel de suscripción**: proporcionan información sobre las operaciones de los recursos en el [plano de control](../../azure-resource-manager/management/control-plane-and-data-plane.md). También se incluyen actualizaciones de los eventos de estado del servicio. Utilice el registro de actividad para determinar el qué, quién y cuándo de las operaciones de escritura (PUT, POST, DELETE). Hay un único registro de actividad para cada suscripción de Azure.

* **Registros de recursos de Azure**: proporcionan información sobre las operaciones que se realizaron en un recurso de Azure en el [plano de datos](../../azure-resource-manager/management/control-plane-and-data-plane.md). Por ejemplo, obtener un secreto de una Key Vault es una operación de plano de datos. Realizar una solicitud a una base de datos también es una operación de plano de datos. El contenido de estos registros de recurso varía según el servicio de Azure y el tipo de recurso.

Para enviar registros de nivel de suscripción a Datadog, seleccione **Send subscription activity logs** (Enviar registros de actividad de suscripción). Si esta opción se deja desactivada, ninguno de los registros de nivel de suscripción se envía a Datadog.

Para enviar registros de recursos de Azure a Datadog, seleccione **Send Azure resource logs for all defined resources** (Enviar registros de recursos para todos los recursos definidos). Los tipos de registros de recursos de Azure se enumeran en [Categorías admitidas en los registros de recursos de Azure](../../azure-monitor/essentials/resource-logs-categories.md).  Para filtrar el conjunto de recursos de Azure que envían registros a Datadog, use etiquetas de recursos de Azure.

Azure cobrará los registros enviados a Datadog. Para más información, consulte los [precios de los registros de la plataforma](https://azure.microsoft.com/pricing/details/monitor/) que se envían a los asociados de Azure Marketplace.

Una vez que haya completado la configuración de métricas y registros, seleccione **Siguiente: Inicio de sesión único**.

## <a name="configure-single-sign-on"></a>Configurar inicio de sesión único

Si va a vincular el recurso Datadog a una organización de Datadog existente, no puede configurar el inicio de sesión único en este paso. En su lugar, configure el inicio de sesión único después de crear el recurso Datadog. Para más información, consulte la sección sobre la [reconfiguración del inicio de sesión único](manage.md#reconfigure-single-sign-on).

:::image type="content" source="media/link-to-existing-organization/linking-sso.png" alt-text="Inicio de sesión único para establecer una vinculación con una organización de Datadog existente." border="true":::

Seleccione **Siguiente: Etiquetas**.

## <a name="add-custom-tags"></a>Adición de etiquetas personalizadas

Puede especificar etiquetas personalizadas para el nuevo recurso Datadog. Especifique los pares de nombre y valor de las etiquetas que se aplican al recurso Datadog.

:::image type="content" source="media/link-to-existing-organization/tags.png" alt-text="Agregue etiquetas personalizadas para el recurso Datadog." border="true":::

Cuando haya terminado de agregar etiquetas, seleccione **Siguiente: Revisar y crear**.

## <a name="review--create-datadog-resource"></a>Revisar y crear recurso Datadog

Revise las opciones y los términos de uso requeridos. Una vez completada la validación, seleccione **Crear**.

:::image type="content" source="media/link-to-existing-organization/review-create.png" alt-text="Revisar y crear recurso Datadog" border="true":::.

Azure implementa el recurso Datadog.

Cuando se complete el proceso, seleccione **Ir al recurso** para ver el recurso Datadog.

:::image type="content" source="media/link-to-existing-organization/go-to-resource.png" alt-text="Implementación de recurso Datadog." border="true":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración del recurso Datadog](manage.md)
