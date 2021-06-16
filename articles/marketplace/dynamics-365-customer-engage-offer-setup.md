---
title: Creación de una oferta de Dynamics 365 for Customer Engagement & Power Apps en Microsoft AppSource (Azure Marketplace).
description: Creación de una oferta de Dynamics 365 for Customer Engagement & Power Apps en Microsoft AppSource (Azure Marketplace).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.date: 04/30/2021
ms.openlocfilehash: 060d24a331a7f15c4ee687c919fe8c2b7cdf2ca6
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971459"
---
# <a name="how-to-create-a-dynamics-365-for-customer-engagement--powerapps-offer"></a>Creación de una oferta de Dynamics 365 for Customer Engagement and Power Apps

En este artículo se explica cómo crear una oferta de Dynamics 365 for Customer Engagement and Power Apps. Todas las ofertas de Dynamics 365 deben pasar por nuestro proceso de certificación. La experiencia de evaluación permite a los usuarios implementar la solución en un entorno de Dynamics 365 en vivo.

Antes de empezar, cree una cuenta de marketplace comercial en el [Centro de partners](./create-account.md) y asegúrese de que esté inscrita en el programa del marketplace comercial.

## <a name="before-you-begin"></a>Antes de empezar

Revise el artículo [Planeamiento de una oferta de Dynamics 365](marketplace-dynamics-365.md). En este artículo se explican los requisitos técnicos de esta oferta y se muestra la información y los recursos que necesitará para crearla.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página de información general, seleccione **+ Nueva oferta** > **Dynamics 365 for Customer Engagement & Power Apps**.

    :::image type="content" source="media/dynamics-365/new-offer-dynamics-365-customer-engagement.png" alt-text="Captura de pantalla que muestra las opciones del menú del panel izquierdo y el botón &quot;Nueva oferta&quot; con Customer Engagement seleccionado.":::

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

Escriba un nombre descriptivo que se usará para hacer referencia a esta oferta únicamente dentro del Centro de partners. Este alias de oferta (rellenado previamente con lo que se haya especificado al crear la oferta) no se usará en el marketplace y es diferente del nombre de la oferta que se muestra a los clientes. Si quiere actualizar el nombre de la oferta más adelante, vaya a la página [Descripción de la oferta](dynamics-365-customer-engage-offer-listing.md).

## <a name="setup-details"></a>Detalles de la configuración

En **¿Cómo quiere que los clientes potenciales interactúen con esta oferta?** , seleccione la opción que quiere usar para esta oferta:

- **Habilitar la administración de licencias de aplicaciones a través de Microsoft**: administre las licencias de aplicación por medio de Microsoft. Para permitir que los clientes ejecuten la funcionalidad base de la aplicación sin una licencia y usen características prémium después de haber adquirido una licencia, active la casilla **Permitir que los clientes instalen mi aplicación incluso si no se han asignado licencias**. Si activa esta segunda casilla, debe configurar el paquete de la solución para que no exija una licencia.

    > [!NOTE]
    > No puede cambiar este valor después de publicar la oferta. Para obtener más información sobre este valor, vea [Administración de licencias de aplicaciones de terceros por medio de Microsoft](third-party-license.md).

- **Obtener ahora (gratis)** : la oferta se muestra a los clientes de forma gratuita.
- **Free trial (listing)** (Evaluación gratuita [descripción]): la oferta se muestra a los clientes con un vínculo a una evaluación gratuita. Las ofertas que incluyen evaluaciones gratuitas se crean, administran y configuran desde su servicio y no tienen suscripciones administradas por Microsoft.

    > [!NOTE]
    > Los tokens que la aplicación recibirá a través del vínculo de evaluación gratuita solo se pueden usar para obtener información de usuario mediante Azure Active Directory (Azure AD) para automatizar la creación de cuentas en la aplicación. No se admiten las cuentas de Microsoft para la autenticación con este token.

- **Contacto**: recopile información de contacto de los clientes mediante la conexión de su sistema de Administración de relaciones con los clientes (CRM). El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre e identificador de la oferta y el origen de Marketplace donde la ha encontrado, se envían al sistema CRM que se ha configurado. Para más información sobre cómo configurar el sistema CRM, consulte [Clientes potenciales](#customer-leads).

## <a name="test-drive"></a>Versión de prueba

Una versión de prueba es una excelente manera de exhibir la oferta a posibles clientes al concederles acceso a un entorno preconfigurado durante un número fijo de horas. La oferta de una versión de prueba tiene como resultado una mayor tasa de conversión y genera clientes potenciales muy cualificados. Para más información, comience con [¿Qué es una versión de prueba?](what-is-test-drive.md)

> [!TIP]
> Una versión de prueba es diferente de una evaluación gratuita. Puede ofrecer una versión de prueba, una evaluación gratuita o ambas. Las dos proporcionan a los clientes la solución durante un período fijo. Sin embargo, una versión de prueba también incluye un práctico recorrido autoguiado por las principales características y ventajas del producto demostradas en un escenario de implementación real.

Para habilitar una versión de prueba, active la casilla **Habilitar una versión de prueba** y seleccione **Tipo de versión de prueba**. La versión de prueba se configurará más adelante. Con la versión de prueba, también debe configurar la oferta en un sistema CRM para clientes potenciales (consulte la sección siguiente). Para eliminar la versión de prueba de la oferta, desactive esta casilla.

## <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Connect lead management](includes/customer-leads.md)]

Para más información, consulte [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña del menú de navegación izquierdo, **Propiedades**.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las propiedades de la oferta](dynamics-365-customer-engage-properties.md)
- [Procedimientos recomendados para la publicación de ofertas](gtm-offer-listing-best-practices.md)