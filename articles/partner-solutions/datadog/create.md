---
title: 'Creación de Datadog: soluciones para asociados de Azure'
description: En este artículo se describe cómo usar Azure Portal para crear una instancia de Datadog.
ms.service: partner-services
ms.topic: quickstart
ms.date: 05/05/2021
author: tfitzmac
ms.author: tomfitz
ms.custom: references_regions
ms.openlocfilehash: 3d9c364c1367986f4ebe47cc00b4cb8b2434b90c
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517467"
---
# <a name="quickstart-get-started-with-datadog"></a>Inicio rápido: Introducción a Datadog

En este inicio rápido, creará una instancia de Datadog. Puede crear una organización de Datadog o un vínculo a una organización de Datadog existente. Azure solo vincula a organizaciones de Datadog existentes en **US3**.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure Datadog, debe tener acceso de **propietario** en la suscripción de Azure. Asegúrese de que tiene el acceso adecuado antes de iniciar el programa de instalación.

Para usar la característica de inicio de sesión único de Lenguaje de marcado de aserción de seguridad dentro del recurso Datadog, debe configurar una aplicación empresarial. Para agregar una aplicación empresarial, se necesita uno de estos roles: Administrador global, Administrador de aplicaciones en la nube, Administrador de aplicaciones o propietario de la entidad de servicio.

Siga estos pasos para configurar la aplicación empresarial:

1. Vaya al [Portal de Azure](https://portal.azure.com). Seleccione **Azure Active Directory**.
1. En el panel izquierdo, seleccione **Aplicaciones empresariales**.
1. Select **Nueva aplicación**.
1. En **Agregar desde la galería**, busque *Datadog*. Seleccione el resultado de la búsqueda y, después, seleccione **Agregar**.

   :::image type="content" source="media/create/datadog-azure-ad-app-gallery.png" alt-text="Aplicación Datadog en la galería empresarial de AAD." border="true":::

1. Una vez creada la aplicación, vaya a las propiedades del panel lateral. En **¿Asignación de usuarios?** , seleccione **No** y, después, **Guardar**.

   :::image type="content" source="media/create/user-assignment-required.png" alt-text="Establecer las propiedades de la aplicación Datadog" border="true":::

1. Vaya a **Inicio de sesión único** en el panel lateral. Luego, seleccione **SAML**.

   :::image type="content" source="media/create/saml-sso.png" alt-text="Autenticación SAML" border="true":::.

1. Seleccione **Sí** cuando se le pregunte si desea **guardar la configuración de inicio de sesión único**.

   :::image type="content" source="media/create/save-sso.png" alt-text="Guardar el inicio de sesión único para la aplicación Datadog" border="true":::

1. La configuración del inicio de sesión único ahora está completa.

## <a name="find-offer"></a>Búsqueda de la oferta

Use Azure Portal para encontrar Datadog.

1. Abra [Azure Portal](https://portal.azure.com/) e inicie sesión.

1. Si ha visitado el **Marketplace** en una sesión reciente, seleccione el icono entre las opciones disponibles. En caso contrario, busque _Marketplace_.

    :::image type="content" source="media/create/marketplace.png" alt-text="Icono de Marketplace.":::

1. En Marketplace, busque **Datadog**.

1. En la pantalla de información general del plan, seleccione **Configurar + suscribir**.

   :::image type="content" source="media/create/datadog-app.png" alt-text="Aplicación Datadog en Azure Marketplace.":::

## <a name="create-a-datadog-resource-in-azure"></a>Creación de un recurso Datadog en Azure.

El portal muestra un formulario para crear el recurso Datadog.

:::image type="content" source="media/create/datadog-create-resource.png" alt-text="Crear recurso Datadog" border="true":::

Especifique los valores siguientes.

|Propiedad | Descripción
|:-----------|:-------- |
| Subscription | Seleccione la suscripción de Azure que desea usar para crear el recurso Datadog. Debe tener acceso de propietario. |
| Resource group | Especifique si desea crear un nuevo grupo de recursos o utilizar uno existente. Un [grupo de recursos](../../azure-resource-manager/management/overview.md#resource-groups) es un contenedor que almacena los recursos relacionados de una solución de Azure. |
| Nombre del recurso | Especifique un nombre para el recurso Datadog. Este nombre será el de la nueva organización de Datadog cuando se cree. |
| Location | Seleccione Oeste de EE. UU. 2. Actualmente, la única región admitida es Oeste de EE. UU. 2. |
| Organización de Datadog | Para crear una organización de Datadog, seleccione **Nuevo**. Para establecer un vínculo a una organización de Datadog existente, seleccione **Existente**. |
| Plan de precios | Al crear una organización, selecciónelo en la lista de planes de Datadog disponibles. |
| Período de facturación | Mensual. |

Si va a crear una vinculación con una organización de Datadog existente, consulte la sección siguiente. En caso contrario, seleccione **Siguiente: Métricas y registros** y omita la sección siguiente.

## <a name="link-to-existing-datadog-organization"></a>Vínculo a un organización de Datadog existente

Puede vincular el nuevo recurso de Datadog en Azure a una organización de Datadog existente en **US3**.

Seleccione **Existente** en Organización de datos y, después, seleccione **Link to Datadog org** (Vincular a organización de Datadog).

:::image type="content" source="media/create/link-to-existing.png" alt-text="Vincular a una organización de Datadog existente." border="true":::

El vínculo abre una ventana de autenticación de Datadog. Inicie sesión en Datadog.

De forma predeterminada, Azure vincula la organización de Datadog actual con el recurso de Datadog. Si desea vincular a otra organización, seleccione la organización adecuada en la ventana de autenticación, como se muestra a continuación.

:::image type="content" source="media/create/select-datadog-organization.png" alt-text="Seleccionar la organización de Datadog adecuada para vincular" border="true":::

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

:::image type="content" source="media/create/config-metrics-logs.png" alt-text="Configurar registros y métricas" border="true":::.

Hay dos tipos de registros que se pueden emitir desde Azure a Datadog.

1. **Registros de nivel de suscripción**: proporcionan información sobre las operaciones de los recursos en el [plano de control](../../azure-resource-manager/management/control-plane-and-data-plane.md). También se incluyen actualizaciones de los eventos de estado del servicio. Utilice el registro de actividad para determinar el qué, quién y cuándo de las operaciones de escritura (PUT, POST, DELETE). Hay un único registro de actividad para cada suscripción de Azure.

1. **Registros de recursos de Azure**: proporcionan información sobre las operaciones que se realizaron en un recurso de Azure en el [plano de datos](../../azure-resource-manager/management/control-plane-and-data-plane.md). Por ejemplo, obtener un secreto de una Key Vault es una operación de plano de datos. Realizar una solicitud a una base de datos también es una operación de plano de datos. El contenido de estos registros de recurso varía según el servicio de Azure y el tipo de recurso.

Para enviar registros de nivel de suscripción a Datadog, seleccione **Send subscription activity logs** (Enviar registros de actividad de suscripción). Si esta opción se deja desactivada, ninguno de los registros de nivel de suscripción se envía a Datadog.

Para enviar registros de recursos de Azure a Datadog, seleccione **Send Azure resource logs for all defined resources** (Enviar registros de recursos para todos los recursos definidos). Los tipos de registros de recursos de Azure se enumeran en [Categorías admitidas en los registros de recursos de Azure](../../azure-monitor/essentials/resource-logs-categories.md).  Para filtrar el conjunto de recursos de Azure que envían registros a Datadog, use etiquetas de recursos de Azure.

Azure cobrará los registros enviados a Datadog. Para más información, consulte los [precios de los registros de la plataforma](https://azure.microsoft.com/pricing/details/monitor/) que se envían a los asociados de Azure Marketplace.

Una vez que haya completado la configuración de métricas y registros, seleccione **Siguiente: Inicio de sesión único**.

## <a name="configure-single-sign-on"></a>Configurar inicio de sesión único

Si su organización usa Azure Active Directory como proveedor de identidades, puede establecer el inicio de sesión único desde Azure Portal a Datadog. Si su organización usa otro proveedor de identidades o no desea establecer el inicio de sesión único en este momento, puede omitir esta sección.

Si va a vincular el recurso Datadog a una organización de Datadog existente, no puede configurar el inicio de sesión único en este paso. En su lugar, configure el inicio de sesión único después de crear el recurso Datadog. Para más información, consulte la sección sobre la [reconfiguración del inicio de sesión único](manage.md#reconfigure-single-sign-on).

:::image type="content" source="media/create/linking-sso.png" alt-text="Inicio de sesión único para establecer una vinculación con una organización de Datadog existente." border="true":::

Para establecer el inicio de sesión único a través de Azure Active Directory, active la casilla **Habilitar inicio de sesión único a través de Azure Active Directory**.

Azure Portal se recupera la aplicación Datadog adecuada de Azure Active Directory. La aplicación coincide con la aplicación de Enterprise que especificó en un paso anterior.

Seleccione el nombre de la aplicación de Datadog.

:::image type="content" source="media/create/sso.png" alt-text="Habilitar el inicio de sesión único en Datadog." border="true":::

Seleccione **Siguiente: Etiquetas**.

## <a name="add-custom-tags"></a>Adición de etiquetas personalizadas

Puede especificar etiquetas personalizadas para el nuevo recurso Datadog. Especifique los pares de nombre y valor de las etiquetas que se aplican al recurso Datadog.

:::image type="content" source="media/create/tags.png" alt-text="Agregue etiquetas personalizadas para el recurso Datadog." border="true":::

Cuando haya terminado de agregar etiquetas, seleccione **Siguiente: Revisar y crear**.

## <a name="review--create-datadog-resource"></a>Revisar y crear recurso Datadog

Revise las opciones y los términos de uso requeridos. Una vez completada la validación, seleccione **Crear**.

:::image type="content" source="media/create/review-create.png" alt-text="Revisar y crear recurso Datadog" border="true":::.

Azure implementa el recurso Datadog.

Cuando se complete el proceso, seleccione **Ir al recurso** para ver el recurso Datadog.

:::image type="content" source="media/create/go-to-resource.png" alt-text="Implementación de recurso Datadog." border="true":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración del recurso Datadog](manage.md)
