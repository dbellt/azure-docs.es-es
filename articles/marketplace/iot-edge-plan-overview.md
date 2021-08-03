---
title: Creación y edición de planes para una oferta de módulo IoT Edge en Azure Marketplace
description: Cree y edite planes para una oferta de módulo IoT Edge en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: aarathin
ms.author: aarathin
ms.date: 05/21/2021
ms.openlocfilehash: 65284852a0e52c79339e31a54b829d63bb78adbf
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955626"
---
# <a name="create-and-edit-plans-for-an-iot-edge-module-offer"></a>Creación y edición de planes para una oferta de módulo IoT Edge

En esta página de información general podrá crear diferentes opciones de planes dentro de la misma oferta. Los planes (anteriormente denominados SKU) pueden diferir en cuanto a dónde están disponibles (Azure Global o Azure Government) y la imagen a la que hace referencia el plan. La oferta debe contener al menos un plan.

Puede crear hasta 100 planes para cada oferta, de los cuales hasta 45 pueden ser privados. Más información sobre los planes privados en [Ofertas privadas en el marketplace comercial de Microsoft](private-offers.md).

Después de crear un plan, la página de **Información general del plan** muestra:

- Nombres de los planes
- Modelo de precios
- Regiones de Azure (globales o gubernamentales)
- Estado de publicación actual
- Acciones disponibles

Las acciones disponibles en un plan varían en función del estado actual del plan. Entre ellas, las siguientes:

- **Elimine el borrador** si el estado del plan es un Borrador.
- **Detenga el plan de venta** si el estado el plan es Publicado.

## <a name="edit-a-plan"></a>Edición de un plan

Seleccione un **nombre** de plan para editar sus detalles.

## <a name="create-a-plan"></a>Creación de un plan

Para configurar un nuevo plan, seleccione **+ Crear nuevo plan**.

Escriba un **id. de plan** único para cada plan. Este identificador será visible para los clientes en la dirección web del producto. Use solo letras minúsculas y números, guiones o caracteres de subrayado, y un máximo de 50 caracteres. Recuerde que no se puede modificar el id. del plan después de seleccionar **Crear**.

Escriba un **nombre de plan**. Los clientes verán este nombre al decidir que plan van a seleccionar en su oferta. Cada plan de esta oferta debe tener un nombre único. Por ejemplo, puede usar el nombre de oferta **Windows Server** con los planes **Windows Server 2016** y **Windows Server 2019**.

Seleccione **Crear** y siga los pasos que se detallan a continuación.

## <a name="next-steps"></a>Pasos siguientes

- [+ Crear un nuevo plan](iot-edge-plan-setup.md) o
- Salir de la configuración del plan y continuar con la [venta conjunta opcional con Microsoft](./co-sell-overview.md) o
- [Revisar y publicar la oferta](review-publish-offer.md).