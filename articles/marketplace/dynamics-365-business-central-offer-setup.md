---
title: Creación de ofertas de Dynamics 365 for Business Central en Microsoft AppSource (Azure Marketplace)
description: Cree una oferta de Dynamics 365 for Business Central en Microsoft AppSource (Azure Marketplace).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.date: 05/19/2021
ms.openlocfilehash: 43ce147cc1d8cb9cb6bcc389ad02cfcf71038e34
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110547951"
---
# <a name="how-to-create-a-dynamics-365-for-business-central-offer"></a>Creación de una oferta de Dynamics 365 for Business Central

En este artículo se describe cómo crear una oferta de [Dynamics 365 for Business Central](https://dynamics.microsoft.com/business-central/overview). Este tipo de oferta es un servicio de planeamiento de recursos empresariales (ERP) que admite una amplia variedad de procesos de negocios, como finanzas, operaciones, cadena de suministro, CRM, administración de proyectos y comercio electrónico. Todas las ofertas de Dynamics 365 deben pasar por nuestro proceso de certificación.

Antes de empezar, cree una cuenta de marketplace comercial en el [Centro de partners](create-account.md) y asegúrese de que esté inscrita en el programa del marketplace comercial.

## <a name="before-you-begin"></a>Antes de empezar

Revise el artículo [Planeamiento de una oferta de Dynamics 365](marketplace-dynamics-365.md). Se explican los requisitos técnicos de esta oferta y se muestra la información y los recursos que necesitará para crearla.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página Información general, seleccione **+ Nueva oferta** > **Dynamics 365 for Business Central**.

    :::image type="content" source="media/dynamics-365/new-offer-dynamics-365-business-central.png" alt-text="Opciones del menú del panel izquierdo y el botón &quot;Nueva oferta&quot;":::.

> [!IMPORTANT]
> Una vez publicada la oferta, las modificaciones que realice en ella en el Centro de partners aparecerán en Microsoft AppSource solo después de volver a publicarla. Asegúrese de volver a publicar siempre una oferta después de cambiarla.

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

Escriba un nombre descriptivo que se usará para hacer referencia a esta oferta únicamente dentro del Centro de partners. Este alias de oferta (rellenado previamente con lo que se haya especificado al crear la oferta) no se usará en el marketplace y es diferente del nombre de la oferta que se muestra a los clientes. Si quiere actualizar el nombre de la oferta más adelante, vaya a la página [Descripción de la oferta](dynamics-365-business-central-offer-listing.md).

## <a name="setup-details"></a>Detalles de la configuración

Seleccione el **tipo de paquete** que se aplica a su oferta: (Biz Central)

- Una aplicación de **complemento** amplía la experiencia y la funcionalidad existente de Dynamics 365 Business Central. Para obtener más información, consulte [Aplicaciones de complemento](/dynamics365/business-central/dev-itpro/developer/readiness/readiness-add-on-apps).
- Se puede usar una aplicación de **conexión** en el escenario en el que debe establecerse una conexión punto a punto entre Dynamics 365 Business Central y una solución o servicio de terceros. Para obtener más información, consulte [Aplicaciones de Connect](/dynamics365/business-central/dev-itpro/developer/readiness/readiness-connect-apps).

En **¿Cómo quiere que los clientes potenciales interactúen con esta oferta?** , seleccione la opción que quiere usar para esta oferta.

- **Obtener ahora (gratis)** : la oferta se muestra a los clientes de forma gratuita.
- **Free trial (listing)** (Evaluación gratuita [descripción]): la oferta se muestra a los clientes con un vínculo a una evaluación gratuita. Las ofertas que incluyen evaluaciones gratuitas se crean, administran y configuran desde su servicio y no tienen suscripciones administradas por Microsoft.

    > [!NOTE]
    > Los tokens que la aplicación recibirá a través del vínculo de evaluación gratuita solo se pueden usar para obtener información de usuario mediante Azure Active Directory (Azure AD) para automatizar la creación de cuentas en la aplicación. No se admiten las cuentas de Microsoft para la autenticación con este token.

- **Contacto**: recopile información de contacto de los clientes mediante la conexión de su sistema de Administración de relaciones con los clientes (CRM). El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre e identificador de la oferta y el origen de Marketplace donde la ha encontrado, se envían al sistema CRM que se ha configurado. Para más información sobre cómo configurar el sistema CRM, consulte [Clientes potenciales](#customer-leads).

## <a name="test-drive"></a>Versión de prueba

Una versión de prueba es una excelente manera de presentar la oferta a clientes potenciales, ya que se les brinda la oportunidad de «probar antes de comprar», lo que aumenta la conversión y la generación de clientes potenciales altamente cualificados. Para más información, consulte [¿Qué es la versión de prueba?](what-is-test-drive.md)

Para habilitar una versión de prueba durante un período de tiempo determinado, seleccione la casilla **Habilitar una versión de prueba**. Para eliminar la versión de prueba de la oferta, desactive esta casilla.

> [!TIP]
> Una versión de prueba es diferente de una evaluación gratuita. Puede ofrecer una versión de prueba, una evaluación gratuita o ambas. Las dos proporcionan a los clientes la solución durante un período fijo. Sin embargo, una versión de prueba también incluye un práctico recorrido autoguiado por las principales características y ventajas del producto demostradas en un escenario de implementación real.

## <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Connect lead management](includes/customer-leads.md)]

Para más información, consulte [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña del menú de navegación izquierdo, **Propiedades**.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las propiedades de la oferta](dynamics-365-business-central-properties.md)
- [Procedimientos recomendados para la publicación de ofertas](gtm-offer-listing-best-practices.md)