---
title: 'Creación de una aplicación de Elastic: soluciones de asociados de Azure'
description: En este artículo se describe cómo usar Azure Portal para crear una instancia de Elastic.
ms.service: partner-services
ms.topic: quickstart
ms.date: 05/19/2021
author: tfitzmac
ms.author: tomfitz
ms.openlocfilehash: 456decb74534cfd3ca5bfbf966c57b4182814225
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952611"
---
# <a name="quickstart-get-started-with-elastic"></a>Inicio rápido: Introducción a Elastic

En esta guía de inicio rápido, usará Azure Portal para integrar una instancia de Elastic con sus soluciones de Azure.

## <a name="prerequisites"></a>Prerrequisitos

- Propietario de la suscripción: la integración de Elastic con Azure solo la pueden crear los usuarios que tengan acceso de _propietario_ en la suscripción de Azure. [Confirme que tiene el acceso adecuado](../../role-based-access-control/check-access.md) antes de iniciar el programa de instalación.
- Aplicación de inicio de sesión único: la capacidad de navegar automáticamente entre Azure Portal y Elastic Cloud está habilitada mediante el inicio de sesión único (SSO). Esta opción se habilita y activa automáticamente para todos los usuarios de Azure. 

## <a name="find-offer"></a>Búsqueda de la oferta

Use Azure Portal para encontrar la aplicación de Elastic.

1. En un explorador web, vaya a [Azure Portal](https://portal.azure.com/) e inicie sesión.

1. Si ha visitado el **Marketplace** en una sesión reciente, seleccione el icono entre las opciones disponibles. En caso contrario, busque _Marketplace_.

    :::image type="content" source="media/create/marketplace.png" alt-text="Icono de Marketplace.":::

1. Busque _Elastic_ y seleccione **Elasticsearch (Elastic Cloud)** en las ofertas disponibles.

1. Seleccione **Set up + subscribe** (Configurar y suscribirse).

   Seleccione :::image type="content" source="media/create/set-up.png" alt-text="Oferta":::.

## <a name="create-resource"></a>Crear el recurso

Una vez que haya seleccionado la oferta para Elastic, estará listo para configurar la aplicación.

1. En la página de aspectos básicos **Creación de recursos de Elastic**, proporcione los valores siguientes.

    :::image type="content" source="media/create/create-resource.png" alt-text="Formulario para configurar un recurso de Elastic":::.

    | Propiedad | Descripción |
    | ---- | ---- |
    | **Suscripción** | En la lista desplegable, seleccione una suscripción de Azure a la que tenga acceso como propietario. |
    | **Grupos de recursos** | Especifique si desea crear un grupo de recursos o utilizar uno existente. Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. Para más información, consulte [Información general del grupo de recursos de Azure](../../azure-resource-manager/management/overview.md). |
    | **Nombre de la cuenta de Elastic** | Proporcione el nombre de la cuenta de Elastic que desea crear. |
    | **Región** | Seleccione **Oeste de EE. UU. 2** o **Sur de Reino Unido**. Durante la versión preliminar, Elastic solo admite estas regiones. |
    | **Plan de precios** | **Pago por uso**. |
    | **Precio** | Se especifica en función del plan de Elastic seleccionado. |

   Una vez que haya terminado, seleccione **Next: Logs and Metrics** (Siguiente: Registros y métricas).

1. En **Logs & metrics** (Registros y métricas), especifique los registros que se enviarán a Elastic.

    :::image type="content" source="media/create/configure-logs.png" alt-text="Seleccione los registros que desea enviar":::.

   Hay dos tipos de registros que se pueden emitir desde Azure a Elastic.

   Los **registros de suscripción** proporcionan información sobre las operaciones de cada recurso de Azure de la suscripción desde el [plano de administración](../../azure-resource-manager/management/control-plane-and-data-plane.md). Los registros también proporcionan actualizaciones sobre los eventos de Service Health. Use el registro de actividad para determinar qué, quién y cuándo para las operaciones de escritura (PUT, POST y DELETE) en los recursos de la suscripción. Hay un único registro de actividad para cada suscripción de Azure.

   Los **registros de recursos de Azure** proporcionan información sobre las operaciones que se realizan dentro del [plano de datos](../../azure-resource-manager/management/control-plane-and-data-plane.md). Por ejemplo, obtener un secreto de un almacén de claves o realizar una solicitud a una base de datos son actividades del plano de datos. El contenido de estos registros de recurso varía según el servicio de Azure y el tipo de recurso. Los tipos de registros de recursos de Azure se enumeran en [Categorías admitidas en los registros de recursos de Azure](../../azure-monitor/essentials/resource-logs-categories.md).

   Para filtrar los recursos de Azure que envían registros a Elastic, use etiquetas de recursos. Las reglas de etiquetas para el envío de registros son:

   * De forma predeterminada, se recopilan registros para todos los recursos. 
   * Los recursos con etiquetas *Include* envían registros a Elastic. 
   * Los recursos con etiquetas *Exclude* no envían registros a Elastic. 
   * Si hay un conflicto entre las reglas de inclusión y exclusión, la exclusión tiene prioridad.
 
   Seleccione **Siguiente: Etiquetas** para configurar etiquetas para el nuevo recurso de Elastic.

1. En **Etiquetas**, agregue etiquetas personalizadas para el nuevo recurso de Elastic. Cada etiqueta consta de un nombre y un valor. Cuando haya terminado de agregar etiquetas, seleccione **Siguiente: Revisar y crear** para ir al paso final para la creación del recurso. 

   :::image type="content" source="media/create/add-tags.png" alt-text="Adición de etiquetas al recurso de Elastic":::

1. En **Revisar y crear**, se valida la configuración. Puede revisar las selecciones realizadas en los formularios anteriores. También puede revisar los términos de esta oferta.

   :::image type="content" source="media/create/review-validation.png" alt-text="Selecciones de revisión y validación":::

   Una vez que la validación se haya realizado correctamente y haya revisado los términos, seleccione **Crear**.

1. Azure inicia la implementación.

   :::image type="content" source="media/create/deployment-in-progress.png" alt-text="Estado de la implementación":::

1. Una vez finalizada la implementación, seleccione **Ir al recurso** para ver el recurso implementado.

    :::image type="content" source="media/create/deployment-complete.png" alt-text="Visualización del estado de la implementación":::


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración del recurso de Elastic](manage.md)