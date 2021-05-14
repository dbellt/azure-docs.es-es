---
title: Configuración de los detalles de la descripción de una oferta de contenedor de Azure en Microsoft AppSource
description: Configure los detalles de la descripción de una oferta de contenedor de Azure en Microsoft AppSource.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: keferna
ms.author: keferna
ms.date: 03/30/2021
ms.openlocfilehash: 0b9b96c4cbc2484597e51463a6b7a4ecfaf9e950
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107892535"
---
# <a name="configure-azure-container-offer-listing-details"></a>Configuración de los detalles de la descripción de una oferta de contenedor de Azure

Esta página le permite definir los detalles de la oferta, como el nombre de la oferta, la descripción, los vínculos, los contactos, los logotipos y las capturas de pantalla.

> [!NOTE]
> Proporcione los detalles de la descripción de la oferta solo en un idioma. No es necesario que estén en inglés si la descripción de la oferta comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable especificar una *dirección URL de vínculo útil* para ofrecer contenido en un idioma distinto del que se usa en el contenido de la descripción de la oferta.

## <a name="marketplace-details"></a>Detalles del marketplace

El **nombre** que escriba aquí se muestra a los clientes como el título de la oferta. Este campo se ha rellenado previamente con el texto especificado en **Alias de oferta** cuando creó la oferta, pero puede cambiarlo. El nombre:

- puede incluir símbolos de marca comercial y copyright.
- Debe tener 50 caracteres o menos.
- No puede incluir emojis.

Proporcione una descripción breve de la oferta (100 caracteres como máximo), en el **resumen de resultados de la búsqueda**. Esta descripción se puede usar en los resultados de la búsqueda de Marketplace.

Proporcione una **breve descripción** de la oferta (hasta 256 caracteres). Aparecerá en los resultados de la búsqueda y en la página de detalles de la oferta.

[!INCLUDE [Long description-1](includes/long-description-1.md)]

[!INCLUDE [Long description-2](includes/long-description-2.md)]

Use etiquetas HTML para formatear la descripción y hacer que sea más atractiva. Para obtener una lista de las etiquetas permitidas, consulte el artículo sobre las [etiquetas HTML admitidas](supported-html-tags.md).

Escriba la dirección web (URL) de la directiva de privacidad de la organización. Asegúrese de que la oferta cumple con las leyes y regulaciones de privacidad. También debe publicar una directiva de privacidad válida en su sitio web.

## <a name="useful-links"></a>Vínculos útiles

Proporcione documentos en línea complementarios sobre su oferta. Puede agregar hasta 25 vínculos. Para agregar un vínculo, seleccione **+ Agregar un vínculo** y, a continuación, rellene los campos siguientes:

- **Nombre**: los clientes lo verán en la página de detalles de la oferta.
- **Vínculo** (dirección URL): escriba un vínculo para que los clientes vean el documento en línea. El vínculo debe empezar por http:// o https://.

### <a name="contact-information"></a>Información de contacto

Proporcione el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico**, un **contacto de ingeniería** o un **contacto del Programa del proveedor de soluciones en la nube**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP.

En la sección **Contacto de soporte técnico**, proporcione el **sitio web de soporte** donde los clientes de Azure Global y Azure Government (si procede) pueden ponerse en contacto con el equipo de soporte técnico.

## <a name="marketplace-media"></a>Elementos multimedia del marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato PNG. Las imágenes borrosas harán que se rechace el envío.

[!INCLUDE [logo tips](includes/graphics-suggestions.md)]

>[!NOTE]
>Si tiene problemas al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

### <a name="logos"></a>Logotipos

Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear otros tamaños necesarios. Opcionalmente, puede reemplazarlo por una imagen diferente.

Estos logotipos se usan en distintos lugares de la publicación:

[!INCLUDE [logos-appsource-only](includes/logos-appsource-only.md)]

[!INCLUDE [Logo tips](includes/graphics-suggestions.md)]

### <a name="screenshots"></a>Capturas de pantalla

Agregue entre una y cinco capturas de pantalla que muestren el funcionamiento de la oferta. Todas las capturas de pantallas deben tener un tamaño de 1280 x 720 píxeles y estar en formato PNG. Agregue un título para cada captura de pantalla.

### <a name="videos"></a>Vídeos

Agregue hasta cinco vídeos que muestren la oferta (este paso es opcional). Deben estar hospedados en un servicio de vídeo externo. Escriba el nombre de cada vídeo, su dirección web y una imagen en miniatura en formato PNG del vídeo de 1280 x 720 píxeles.

Para ver más recursos sobre las listas de marketplace, consulte [Procedimientos recomendados para las listas de ofertas de marketplace](gtm-offer-listing-best-practices.md).

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña del menú de navegación izquierdo, **Vista previa de la audiencia**.
<!-- #### Offer examples

The following examples show how the offer listing fields appear in different places of the offer.

This shows search results in Azure Marketplace:

[![Illustrates the search results in Azure Marketplace](media/azure-container/azure-create-7-search-results-mkt-plc-small.png)](media/azure-container/azure-create-7-search-results-mkt-plc.png#lightbox)

This shows the **Offer listing** page in Azure portal:

:::image type="content" source="media/azure-container/azure-create-8-offer-listing-portal.png" alt-text="Illustrates the Offer listing page in Azure portal.":::

This shows search results in Azure portal:

[![Illustrates the search results in Azure portal.](media/azure-container/azure-create-9-search-results-portal-small.png)](media/azure-container/azure-create-9-search-results-portal.png#lightbox) -->

## <a name="next-steps"></a>Pasos siguientes

- [Establecimiento de la audiencia de la versión preliminar de la oferta](azure-container-preview-audience.md)
