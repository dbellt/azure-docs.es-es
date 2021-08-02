---
title: Creación de una oferta de aplicación de Power BI en Microsoft AppSource (Azure Marketplace)
description: Cree una oferta de aplicación de Power BI en Microsoft AppSource (Azure Marketplace).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: keferna
ms.author: keferna
ms.date: 05/26/2021
ms.openlocfilehash: e86a97b0f4aa9fef24203128970b8d9cc857cd41
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955380"
---
# <a name="create-a-power-bi-app-offer"></a>Creación de una oferta de aplicación de Power BI

En este artículo se describe cómo crear una oferta de aplicación de Power BI. Todas las ofertas pasan por un proceso de certificación, que comprueba la solución en busca de requisitos estándar, compatibilidad y procedimientos adecuados.

Antes de empezar, cree una cuenta comercial de Marketplace en el [Centro de partners](./create-account.md) y asegúrese de que esté inscrita en el programa comercial de Marketplace.

## <a name="before-you-begin"></a>Antes de empezar

Revise [Planeamiento de una oferta de Power BI](marketplace-power-bi.md). En este artículo se explican los requisitos técnicos de esta oferta y se muestra la información y los recursos que necesitará para crearla.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página de información general, seleccione **+ Nueva oferta** > **Aplicación de Power BI**.

    :::image type="content" source="media/power-bi/new-offer-power-bi-app.png" alt-text="Opciones del menú del panel izquierdo y el botón &quot;Nueva oferta&quot;":::.

> [!IMPORTANT]
> Una vez publicada la oferta, las modificaciones que realice en ella en el Centro de partners aparecerán en Microsoft AppSource solo después de volver a publicarla. Asegúrese de volver a publicar siempre una oferta después de cambiarla.

Si no se muestra ni está habilitada la opción **Aplicación de Power BI**, la cuenta no tiene permiso para crear este tipo de oferta. Compruebe que cumple todos los [requisitos](marketplace-dynamics-365.md) para este tipo de oferta, incluido el registro para una cuenta de desarrollador.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Este identificador es visible para los clientes en la dirección web de la oferta y en las plantillas de Azure Resource Manager, si procede.
- Use solo letras minúsculas y números. El identificador puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si el identificador del editor es `testpublisherid` y escribe **test-offer-1**, la dirección web de la oferta será `https://appsource.microsoft.com/product/dynamics-365/testpublisherid.test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se utiliza en AppSource. Es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- Este nombre no se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta. En el Centro de partners, abra la página **Configuración de la oferta**.

## <a name="alias"></a>Alias

Escriba un nombre descriptivo que se usará para hacer referencia a esta oferta únicamente dentro del Centro de partners. Este alias de oferta (rellenado previamente con lo que se haya especificado al crear la oferta) no se usará en el marketplace y es diferente del nombre de la oferta que se muestra a los clientes. Si quiere actualizar el nombre de la oferta más adelante, vaya a la página [Descripción de la oferta](power-bi-app-offer-listing.md).

## <a name="setup-details"></a>Detalles de la configuración

Esta sección está en blanco y no es aplicable a aplicaciones de Power BI.

## <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Connect lead management](includes/customer-leads.md)]

Para más información, consulte [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña del menú de navegación izquierdo, **Propiedades**.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las propiedades de la oferta](power-bi-app-properties.md)