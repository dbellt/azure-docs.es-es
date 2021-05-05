---
title: Establecimiento de una configuración técnica del plan para una oferta de Azure Container en Microsoft AppSource.
description: Establecimiento de una configuración técnica del plan para una oferta de Azure Container en Microsoft AppSource.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: keferna
ms.author: keferna
ms.date: 04/21/2021
ms.openlocfilehash: bc720b4df62a06d4c635cc9998dd453bfb7683b1
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108127486"
---
# <a name="set-plan-technical-configuration-for-an-azure-container-offer"></a>Establecimiento de una configuración técnica para una oferta de Azure Container

Las imágenes de contenedor se deben hospedar en una instancia privada de [Azure Container Registry](https://azure.microsoft.com/services/container-registry/). Use esta página para proporcionar información de referencia para el repositorio de imágenes de contenedor de Azure Container Registry.

Una vez enviada la oferta, la imagen de contenedor se copia en Azure Marketplace en un registro de contenedor público específico. Todas las solicitudes de los usuarios de Azure que usan su módulo se atienden desde el registro de contenedor público de Azure Marketplace, no desde su registro de contenedor privado.

Puede elegir como destino varias plataformas y proporcionar varias versiones a la imagen de contenedor del módulo por medio de etiquetas. Para obtener más información sobre las etiquetas y el control de versiones, consulte [Preparación de los recursos técnicos de Azure Container](azure-container-technical-assets.md).

## <a name="image-repository-details"></a>Detalles del repositorio de imágenes

Proporcione el **id. de suscripción de Azure** en el que se indica el uso de recursos y los servicios se facturan para la instancia de Azure Container Registry que incluye la imagen de contenedor. Puede encontrar este identificador en la [página Suscripciones](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) de Azure Portal.

Proporcione el [**nombre del grupo de recursos**](../azure-resource-manager/management/manage-resource-groups-portal.md) que contiene la instancia de Azure Container Registry con la imagen de contenedor. El grupo de recursos debe ser accesible en el identificador de suscripción (anterior). Puede encontrar el nombre en la página [Grupos de recursos](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceGroups) en Azure Portal.

Proporcione el [**nombre de la instancia de Azure Container Registry**](../container-registry/container-registry-intro.md) que tiene su imagen de contenedor. El registro de contenedor debe estar presente en el grupo de recursos de Azure que proporcionó anteriormente. Proporcione solo el nombre del registro, no el nombre completo del servidor de inicio de sesión. Asegúrese de omitir la parte **azurecr.io** del nombre. Puede encontrar el nombre del registro en la [página Registros de contenedor](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.ContainerRegistry%2Fregistries) en Azure Portal.

Proporcione el [**Nombre de usuario de administrador de Azure Container Registry**](../container-registry/container-registry-authentication.md#admin-account) asociado a la instancia de Azure Container Registry que tiene su imagen de contenedor. El nombre de usuario y la contraseña (paso siguiente) son necesarios para asegurarse de que la empresa tiene acceso al registro. Para obtener el nombre de usuario y la contraseña de administrador, establezca la propiedad **admin-enabled** en **True** mediante la Interfaz de la línea de comandos (CLI) de Azure. Opcionalmente, puede establecer **Usuario administrador** en **Habilitar** en Azure Portal.

:::image type="content" source="media/azure-container/azure-create-12-update-container-registry-edit.png" alt-text="Muestra el cuadro de diálogo Actualizar registro de contenedor.":::

Proporcione la **contraseña de administrador de la instancia de Azure Container Registry** del nombre de usuario administrador asociado a la instancia de Azure Container Registry que tiene la imagen de contenedor. El nombre de usuario y la contraseña son necesarios para asegurarse de que la empresa tiene acceso al registro. Puede obtener la contraseña en Azure Portal desde **Container Registry** > **Claves de acceso** o bien con la CLI de Azure con el [comando show](/cli/azure/acr/credential#az-acr-credential-show).

:::image type="content" source="media/azure-container/azure-create-13-access-keys.png" alt-text="Muestra la pantalla de la clave de acceso en Azure Portal.":::

Proporcione el **nombre del repositorio de Azure Container Registry** que tiene su imagen. Especifique el nombre del repositorio cuando inserte la imagen en el registro. Para encontrar el nombre del repositorio, puede ir a [Container Registry](https://azure.microsoft.com/services/container-registry/) > **página Repositorios**. Para obtener más información, consulte [Visualización de los repositorios de registros de contenedor en Azure Portal](../container-registry/container-registry-repositories.md). Una vez establecido el nombre, no se puede cambiar. Use un nombre único para cada oferta de su cuenta.

## <a name="image-versions"></a>Versiones de la imagen

Los clientes deben poder obtener actualizaciones automáticamente de Azure Marketplace cuando publique una actualización. Si no quieren realizar la actualización, deben poder mantener una versión específica de la imagen. Para hacerlo, puede agregar nuevas etiquetas de imagen cada vez que realice una actualización en la imagen.

Seleccione **Agregar versión de imagen** para incluir una **etiqueta de imagen** que apunta a la versión más reciente de la imagen en todas las plataformas admitidas. Debe incluir al menos una etiqueta de versión (por ejemplo, que empiece por xx.xx.xx, donde xx es un número). Los clientes deben usar [etiquetas de manifiesto](https://github.com/estesp/manifest-tool) para dirigirse a varias plataformas. Todas las etiquetas a las que hace referencia una etiqueta de manifiesto también se deben agregar para que podamos cargarlas. Todas las etiquetas de manifiesto (excepto la etiqueta más reciente) deben comenzar por X.Y- o X.Y.Z-, donde X, Y, Z son números enteros. Por ejemplo, si una etiqueta más reciente apunta a `1.0.1-linux-x64`, `1.0.1-linux-arm32` y `1.0.1-windows-arm32`, estas seis etiquetas deben agregarse a este campo. Para obtener detalles sobre las etiquetas y el control de versiones, consulte [Preparación de los recursos técnicos de Azure Container](azure-container-technical-assets.md).

> [!TIP]
> No olvide agregar una etiqueta de prueba a la imagen para facilitar su identificación durante las pruebas.

<!-- possible future restore

## Samples

These examples show how the plan listing fields appear in different views.

These are the fields in Azure Marketplace when viewing plan details:

:::image type="content" source="media/azure-container/azure-create-10-plan-details-mtplc.png" alt-text="Illustrates the fields you see when viewing plan details in Azure Marketplace.":::

These are plan details on the Azure portal:

:::image type="content" source="media/azure-container/azure-create-11-plan-details-portal.png" alt-text="Illustrates plan details on the Azure portal.":::

Select **Save draft**, then **← Plan overview**  in the left-nav menu to return to the plan overview page.
-->
## <a name="next-steps"></a>Pasos siguientes

- Para **realizar una venta conjunta con Microsoft** (opcional), seleccione esta opción en el menú de navegación izquierdo. Para obtener más información, consulte [Involucración de asociados para realizar una venta conjunta](./co-sell-overview.md).
- Para **realizar una venta conjunta a través de CSP**  (Proveedores de soluciones en la nube; también opcional), seleccione esta opción en el menú de navegación de la izquierda. Para obtener más información, consulte [Realizar una venta conjunta a través de asociados de CSP](cloud-solution-providers.md).
- Si no está configurando ninguna de estas opciones o ya ha terminado, es el momento de [revisar y publicar la oferta](review-publish-offer.md).