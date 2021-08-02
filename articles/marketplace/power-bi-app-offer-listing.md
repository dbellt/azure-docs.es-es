---
title: Configuración de los detalles de las descripciones de ofertas de aplicaciones de Power BI en Microsoft AppSource (Azure Marketplace)
description: Configure los detalles de las descripciones de ofertas de aplicaciones de Power BI en Microsoft AppSource (Azure Marketplace).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: keferna
ms.author: keferna
ms.date: 05/26/2021
ms.openlocfilehash: bc827cb42246129a507ee0894251b65d5c53a89c
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110631664"
---
# <a name="configure-power-bi-app-offer-listing-details"></a>Configuración de los detalles de las descripciones de ofertas de aplicaciones de Power BI

Esta página le permite definir los detalles de las ofertas, como el nombre de la oferta, la descripción, los vínculos, los contactos, los logotipos y las capturas de pantalla que aparecerán en Microsoft AppSource.

> [!NOTE]
> Proporcione los detalles de la descripción de la oferta solo en un idioma. No es necesario que estén en inglés si la descripción de la oferta comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable especificar una *dirección URL de vínculo útil* para ofrecer contenido en un idioma distinto del que se usa en el contenido de la descripción de la oferta.

A continuación se muestra un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/power-bi/example-power-bi-app.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

##### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
1. Productos
1. Categorías
1. Industrias
1. Dirección de soporte técnico (vínculo)
1. Términos de uso
1. Directiva de privacidad
1. Nombre de la oferta
1. Resumen de los resultados de la búsqueda
1. Descripción
1. Capturas de pantallas o vídeos

## <a name="marketplace-details"></a>Detalles del marketplace

El **nombre** que escriba aquí se muestra a los clientes como el título de la oferta. Este campo se ha rellenado previamente con el texto especificado en **Alias de oferta** cuando creó la oferta, pero puede cambiarlo. El nombre:

- puede incluir símbolos de marca comercial y copyright.
- Debe tener 50 caracteres o menos.
- No puede incluir emojis.

Proporcione una descripción breve de la oferta (100 caracteres como máximo), en el **resumen de resultados de la búsqueda**. Esta descripción se puede usar en los resultados de la búsqueda de Marketplace.

[!INCLUDE [Long description-1](includes/long-description-1.md)]

[!INCLUDE [Long description-2](includes/long-description-2.md)]

Use etiquetas HTML para formatear la descripción y hacer que sea más atractiva. Para obtener una lista de las etiquetas permitidas, consulte el artículo sobre las [etiquetas HTML admitidas](supported-html-tags.md).

#### <a name="helpprivacy-urls"></a>Direcciones URL de ayuda y privacidad

Escriba el **vínculo de ayuda de la aplicación** (URL), donde los clientes pueden obtener más información sobre su oferta. La dirección URL de ayuda no puede ser la misma que la dirección URL de soporte técnico.

Escriba el **vínculo a la directiva de privacidad** (URL) de su organización. Usted es el responsable no solo de garantizar que la aplicación cumple la normativa y la legislación en lo tocante a la privacidad, sino también de proporcionar una directiva de privacidad válida.

### <a name="contact-information"></a>Información de contacto

Indique el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP.

En la sección **Contacto de soporte técnico**, proporcione la **dirección URL de soporte técnico** en la que los asociados de CSP pueden encontrar soporte técnico para la oferta. La dirección URL de soporte técnico no puede ser la misma que la dirección URL de ayuda.

## <a name="supporting-documents"></a>Documentos relacionados

Proporcione aquí hasta tres documentos de marketing en PDF relacionados, como notas del producto, folletos, listas de comprobación o presentaciones (opcional).

## <a name="marketplace-media"></a>Elementos multimedia del marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato PNG. Las imágenes borrosas harán que se rechace el envío.

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

Agregue hasta cuatro vídeos que muestren la oferta (este paso es opcional). Deben estar hospedados en un servicio de vídeo externo. Escriba el nombre de cada vídeo, su dirección web y una imagen en miniatura en formato PNG del vídeo de 1280 x 720 píxeles.

Para ver más recursos sobre las listas de marketplace, consulte [Procedimientos recomendados para las listas de ofertas de marketplace](gtm-offer-listing-best-practices.md).

Seleccione **Guardar borrador** antes de pasar a la pestaña siguiente del menú de navegación de la izquierda, **Configuración técnica**.

## <a name="next-steps"></a>Pasos siguientes

- [Establecimiento de la configuración técnica de la oferta](power-bi-app-technical-configuration.md)
