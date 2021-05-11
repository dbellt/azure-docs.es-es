---
title: Creación de planes de Dynamics 365 for Customer Engagement & Power Apps en Microsoft AppSource (Azure Marketplace)
description: Configure los planes de ofertas de Dynamics 365 for Customer Engagement & PowerApps si decide habilitar la oferta para la administración de aplicaciones de terceros.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.date: 04/30/2021
ms.openlocfilehash: 6c6863c8f044d1354fa1ed6ebea8b679994fc6d2
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108322313"
---
# <a name="create-dynamics-365-for-customer-engagement--power-apps-plans"></a>Creación de planes de Dynamics 365 for Customer Engagement & Power Apps

Si ha habilitado la administración de licencias de aplicaciones para la oferta, la pestaña **Planes** aparece como se muestra en la captura de pantalla siguiente. De lo contrario, vaya a [Configuración técnica de una oferta de Dynamics 365 for Customer Engagement & Power Apps](dynamics-365-customer-engage-technical-configuration.md).

:::image type="content" source="./media/third-party-license/plan-tab.png" alt-text="Captura de pantalla de la pestaña Información general del plan para una oferta de Dynamics 365 for Customer Engagement & Power Apps habilitada para licencias de aplicaciones de terceros.":::

Si la oferta tiene habilitada la administración de licencias de aplicaciones, debe definir al menos un plan. Puede crear una variedad de planes con diferentes opciones dentro de la misma oferta. Estos planes (a veces denominados SKU) pueden diferir en lo tocante a monetización o niveles de servicio. Más adelante, asignará los identificadores de servicio de estos planes en el paquete de solución para habilitar una comprobación de licencia en tiempo de ejecución por parte de la plataforma de Dynamics con respecto a estos planes. Asignará el identificador de servicio de cada plan del paquete de solución. Esto permite a la plataforma de Dynamics ejecutar una comprobación de licencia con estos planes.

## <a name="create-a-plan"></a>Creación de un plan

1. Cerca de la parte superior de la página **Información general del plan**, seleccione **+ Crear nuevo plan**.
1. En el cuadro de diálogo que aparece, en el cuadro **Id. de plan**, escriba un identificador de plan único. Use solo hasta 50 caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. No se puede modificar el identificador del plan después de seleccionar **Crear**.
1. En el cuadro **Nombre del plan**, escriba un nombre para este plan. Use un máximo de 50 caracteres.
1. Seleccione **Crear**.

## <a name="define-the-plan-listing"></a>Definición de la lista de planes

En la pestaña **Lista del plan**, puede definir el nombre y la descripción del plan como desee que aparezcan en el marketplace comercial. Esta información se mostrará en la página de lista de Microsoft AppSource.

1. En el cuadro **Nombre del plan**, se muestra el nombre que proporcionó anteriormente para este plan. Puede cambiarlo en cualquier momento. Este nombre aparecerá en el marketplace comercial como el título del plan de software de la oferta.
1. En el cuadro **Descripción del plan**, explique lo que hace que este plan de software sea único y las diferencias con respecto a otros planes de software de su oferta. Esta descripción puede contener hasta 500 caracteres.
1. Seleccione **Guardar borrador** y, en la parte superior izquierda, seleccione **Información general del plan**.

    :::image type="content" source="./media/third-party-license/bronze-plan.png" alt-text="Captura de pantalla que muestra el vínculo Información general del plan en la página Lista de planes de una oferta en el Centro de partners.":::

1. Para crear otro plan para esta oferta, en la parte superior de la página **Información general del plan**, seleccione **+ Crear nuevo plan**. A continuación, repita los pasos descritos en la sección [Creación de un plan](#create-a-plan). En caso contrario, si ha terminado de crear planes, vaya a la sección siguiente: Copia de los identificadores de servicio.

## <a name="copy-the-service-ids"></a>Copia de los identificadores de servicio

Debe copiar el identificador de servicio de cada plan que creó para poder asignarlo al paquete de solución en el paso siguiente.

- Para cada plan que haya creado, copie el identificador de servicio en un lugar seguro. Lo agregará al paquete de solución en el paso siguiente. El identificador de servicio se muestra en la página **Información general del plan** con el formato `ISV name.offer name.plan ID`. Por ejemplo, Fabrikam.F365.bronze.

    :::image type="content" source="./media/third-party-license/service-id.png" alt-text="Captura de pantalla de la página Información general del plan. El identificador de servicio del plan está resaltado.":::

## <a name="add-service-ids-to-your-solution-package"></a>Agregar los identificadores de servicio al paquete de solución

1. Agregue los identificadores de servicio que copió en el paso anterior al paquete de solución. Para obtener información sobre cómo hacerlo, consulte [Incorporación de metadatos de licencia a la solución](https://go.microsoft.com/fwlink/?linkid=2162161&clcid=0x409) y [Creación de un paquete de AppSource para la aplicación](/powerapps/developer/data-platform/create-package-app-appsource).
1. Después de crear el archivo ZIP del paquete CRM, cárguelo en Azure Blob Storage. Deberá proporcionar la dirección URL de SAS de la cuenta de Azure Blob Storage que contiene el archivo ZIP del paquete CRM cargado.

## <a name="next-steps"></a>Pasos siguientes

- Vaya a [Configuración técnica de una oferta de Dynamics 365 for Customer Engagement & Power Apps](dynamics-365-customer-engage-technical-configuration.md) para cargar el paquete de solución en la oferta.
