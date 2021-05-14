---
title: Planeamiento de ofertas de Dynamics 365 para Microsoft AppSource
description: Planeamiento de ofertas de Dynamics 365 para Microsoft AppSource
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: vamahtan
ms.author: vamahtan
ms.date: 04/30/2021
ms.openlocfilehash: c0af6f3045a686f9ef73b7761b41e869d3ce9d6f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108766392"
---
# <a name="plan-a-microsoft-dynamics-365-offer"></a>Planeamiento de una oferta de Microsoft Dynamics 365

En este artículo se explican las distintas opciones y características de una oferta de Dynamics 365 en el marketplace comercial de Microsoft AppSource. AppSource incluye ofertas que se basan en Microsoft 365, Dynamics 365, Power BI, Power Apps y Power BI.

Antes de empezar, cree una cuenta de marketplace comercial en el [Centro de partners](./partner-center-portal/create-account.md) y asegúrese de que esté inscrita en el programa del marketplace comercial. Además, revise las [directrices y el proceso de publicación](/office/dev/store/submit-to-appsource-via-partner-center).

## <a name="licensing-options"></a>Opciones de licencia

Cuando prepare la publicación de una nueva oferta, debe decidir la opción de licencia. Esta opción determina la información adicional que deberá proporcionar más adelante cuando cree la oferta en el Centro de partners.

Estas son las opciones de licencia disponibles para las ofertas de Dynamics 365:

| Opción de obtención de licencias | Proceso de transacción |
| --- | --- |
| Ponerse en contacto conmigo | Recopile información de contacto de los clientes mediante la conexión de su sistema de Administración de relaciones con los clientes (CRM). El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre e identificador de la oferta y el origen de Marketplace donde la ha encontrado, se envían al sistema CRM que se ha configurado. Para obtener más información sobre cómo configurar CRM, consulte la sección **Clientes potenciales** de la página **Configuración de la oferta** del tipo de oferta. |
| Evaluación gratuita (anuncio) | Ofrezca a sus clientes una evaluación gratuita de uno, tres o seis meses. Las ofertas que incluyen evaluaciones gratuitas se crean, administran y configuran desde su servicio y no tienen suscripciones administradas por Microsoft. |
| Obténgalo ahora (de forma gratuita) | Muestre la oferta a los clientes gratis. |
| Obtenerlo ahora | Permite administrar las licencias de terceros en el Centro de partners.<br>Actualmente solo está disponible para el siguiente tipo de oferta:<ul><li>Dynamics 365 for Customer Engagement y Power Apps</li></ul><br>Para obtener más información sobre esta opción, vea [Administración de licencias de aplicaciones de terceros por medio de Microsoft](third-party-license.md). |
|||

## <a name="test-drive"></a>Versión de prueba

[!INCLUDE [Test drives section](includes/test-drives.md)]

## <a name="customer-leads"></a>Clientes potenciales

Al publicar una oferta en el marketplace comercial con el Centro de partners, conéctela a un sistema de administración de relaciones con clientes (CRM). Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa. Es necesario conectarse a un CRM si desea habilitar una versión de prueba; de lo contrario, la conexión a un CRM es opcional. El Centro de partners admite Tabla de Azure, Dynamics 365 Customer Engagement, Punto de conexión HTTPS, Marketo y Salesforce.

## <a name="legal"></a>Información legal

Cree los **términos y condiciones**. Los clientes deberán aceptar estos términos para poder probar la oferta. Microsoft tiene un contrato estándar, pero no se aplica a las ofertas de Dynamics 365.

## <a name="offer-listing-details"></a>Detalles de la descripción de la oferta

A continuación se muestra un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/dynamics-365/example-dynamics-365-operations.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

> [!NOTE]
> No es necesario que el contenido de la descripción de la oferta esté en inglés si comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]".

Para facilitar la creación de la oferta, prepare estos elementos con antelación. Todos son necesarios excepto cuando se indica lo contrario.

- **Nombre**: este nombre aparecerá como título de la descripción de la oferta en el marketplace comercial. El nombre puede ser una marca comercial. No puede contener emojis (a menos que sean símbolos de marca comercial y copyright), y no puede tener más de 50 caracteres.
- **Resumen de los resultados de la búsqueda**: el propósito o la función de la oferta en una sola frase de 100 caracteres como máximo y sin saltos de línea. Se usa en los resultados de la búsqueda de las ofertas del marketplace comercial.
- **Descripción**: esta descripción se muestra en la información general de las ofertas del marketplace comercial. Considere la posibilidad de incluir una propuesta de valor, los beneficios principales, los compradores previstos, las asociaciones de categoría o sector, las oportunidades de compra en la aplicación, todas las divulgaciones necesarias y un vínculo para obtener más información. Este cuadro de texto tiene controles de editor de texto enriquecido para que su descripción sea más atractiva. Opcionalmente, use etiquetas HTML para aplicar formato.
- **Palabras de búsqueda** (opcional): escriba un máximo de tres palabras clave de búsqueda que los clientes puedan usar para buscar su oferta. No incluya el **nombre** y la **descripción** de la oferta: ese texto se incluye automáticamente en la búsqueda.
- **Producto con el que funciona la aplicación** (opcional): los nombres de hasta tres productos con los que funciona la oferta.
- **Vínculos a la directiva de privacidad y la ayuda**: las direcciones URL de la ayuda de la oferta y la directiva de privacidad de la empresa. El usuario es responsable de garantizar que la aplicación cumpla con las leyes y los reglamentos de privacidad.
- **Información de contacto**
  - **Contacto de soporte técnico**: el nombre, el teléfono y el correo electrónico que los asociados de Microsoft se usarán cuando los clientes abran incidencias. Incluya la dirección URL del sitio web de soporte técnico.
  - **Contacto de ingeniería**: el nombre, el teléfono y el correo electrónico que Microsoft usará directamente cuando haya problemas con la oferta. Esta información de contacto no aparece en el marketplace comercial.
- **Documentos relacionados**: hasta tres documentos orientados al cliente en formato PDF, como notas del producto, folletos, listas de comprobación o presentaciones de PowerPoint.
- **Elementos multimedia**
    - **Logotipos**: un archivo PNG para el logotipo **grande**. El Centro de partners lo usará para crear otros tamaños de logotipo necesarios. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.
    - **Capturas de pantalla**: al menos una y hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Las imágenes deben tener 1280 x 720 píxeles, en formato PNG, e incluir un título.
    - **Vídeos** (opcional): hasta cuatro vídeos que muestren la oferta. Incluya un nombre, una dirección URL para YouTube o Vimeo y una miniatura PNG de 1280 x 720 píxeles.

> [!Note]
> La oferta debe cumplir las [directivas de certificación del marketplace comercial](/legal/marketplace/certification-policies#100-general) para ser publicada en esta plataforma.

## <a name="additional-sales-opportunities"></a>Oportunidades de venta adicionales

Puede optar por participar en los canales de marketing y ventas respaldados por Microsoft. Al crear la oferta en el Centro de partners, verá dos pestañas hacia el final del proceso para **realizar una venta conjunta con Microsoft**. esta opción permite que los equipos de ventas de Microsoft consideren la solución idónea para la venta conjunta de IP al evaluar las necesidades de los clientes. Consulte [Opción de venta conjunta en el Centro de partners](commercial-marketplace-co-sell.md) para información detallada sobre cómo preparar la oferta para la evaluación.

## <a name="next-steps"></a>Pasos siguientes

Después de analizar los elementos de planeamiento descritos anteriormente, seleccione uno de los siguientes elementos (también disponibles en la tabla de contenido de la izquierda) para empezar a crear la oferta.

| Guía de publicación    | Notas  |
| :------------------- | :-------------------|
| [Dynamics 365 for Operations](partner-center-portal/create-new-operations-offer.md) | Al realizar la compilación para Enterprise Edition, revise primero el [proceso y las directrices de publicación](/dynamics365/fin-ops-core/dev-itpro/lcs-solutions/lcs-solutions-app-source) adicionales. |
| [Dynamics 365 for Business Central](partner-center-portal/create-new-business-central-offer.md) |   |
| [Dynamics 365 for Customer Engagement y Power Apps](dynamics-365-customer-engage-offer-setup.md) | En primer lugar, revise [estos procesos y directrices de publicación](/dynamics365/customer-engagement/developer/publish-app-appsource) adicionales. |
| [Power BI](/azure/marketplace/partner-center-portal/create-power-bi-app-offer) | En primer lugar, revise [estos procesos y directrices de publicación](/power-bi/developer/office-store) adicionales. |
|||
