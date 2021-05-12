---
title: Planeamiento de una oferta de servicio administrado en el marketplace comercial de Microsoft
description: Procedimiento para planear una nueva oferta de servicio administrado para Azure Marketplace con el programa del marketplace comercial en el Centro de partners de Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.reviewer: anbene
ms.date: 12/23/2020
ms.openlocfilehash: ffa7e35bed1713a5ef9e04b59a8dbdb692b9630f
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109790962"
---
# <a name="how-to-plan-a-managed-service-offer-for-the-microsoft-commercial-marketplace"></a>Procedimiento para planear una oferta de servicio administrado en el marketplace comercial de Microsoft

En este artículo se presentan los requisitos para publicar una oferta de servicio administrado en el marketplace comercial de Microsoft por medio del Centro de partners.

Los servicios administrados son ofertas de Azure Marketplace que permiten la administración multiinquilino y entre inquilinos con Azure Lighthouse. Consulte [¿Qué es Azure Lighthouse?](../lighthouse/overview.md) para obtener más información. Cuando un cliente compra una oferta de servicios administrados, estos pueden delegar uno o varios grupos de recursos o suscripciones. A continuación, puede trabajar en esos recursos mediante las funcionalidades de [Administración de recursos delegados de Azure](../lighthouse/concepts/architecture.md) de Azure Lighthouse.

## <a name="eligibility-requirements"></a>Requisitos de elegibilidad

Para publicar una oferta de servicio administrado, debe haber obtenido una competencia Gold o Silver de Microsoft en la plataforma en la nube. Esta competencia demuestra a los clientes su experiencia. Para más información, consulte las [competencias de Microsoft Partner Network](https://partner.microsoft.com/membership/competencies).

Las ofertas deben cumplir todas las [directivas de certificación del marketplace comercial](/legal/marketplace/certification-policies) aplicables para que se publiquen en Azure Marketplace.

## <a name="customer-leads"></a>Clientes potenciales

Debe conectar su oferta al sistema de administración de relaciones con clientes (CRM) para recopilar información del cliente. El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre y el identificador de la oferta, además de la tienda en línea donde la encontró, se envían al sistema CRM que haya configurado. El marketplace comercial admite varios tipos de sistemas CRM, junto con la opción de usar una tabla de Azure o de configurar un punto de conexión HTTPS con Power Automate.

Puede agregar o modificar una conexión CRM en cualquier momento durante o después de la creación de la oferta. Para obtener instrucciones detalladas, vea [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

## <a name="legal-contracts"></a>Contratos legales

En la página Propiedades del Centro de partners, se le pedirá que proporcione **términos y condiciones** para el uso de la oferta. Puede especificar sus términos directamente en el Centro de partners o proporcionar la dirección URL donde se puedan consultar. Los clientes deberán aceptar estos términos y condiciones para poder comprar la oferta.

## <a name="offer-listing-details"></a>Detalles de la descripción de la oferta

Al crear la oferta de servicio administrado en el Centro de partners, deberá proporcionar el texto, las imágenes, los documentos y otros detalles de la oferta. Esto es lo que verán los clientes cuando encuentren su oferta en Azure Marketplace. Observe el ejemplo siguiente:

:::image type="content" source="media/example-managed-service.png" alt-text="Imagen de cómo aparece una oferta de servicio administrado en Azure Marketplace.":::

**Descripciones destacadas**

1. Logotipo
1. Nombre
1. Descripción breve
1. Categorías
1. Contratos legales y directiva de privacidad
1. Descripción
1. Capturas de pantallas o vídeos
1. Vínculos útiles

Este es un ejemplo de cómo aparece la descripción de la oferta en Azure Portal:

:::image type="content" source="media/example-managed-service-azure-portal.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Portal.":::

**Descripciones destacadas**

1. Nombre
2. Descripción
3. Vínculos útiles
4. Capturas de pantallas o vídeos

> [!NOTE]
> Si la oferta está en un idioma distinto del inglés, la descripción de esta podrá estar en tal idioma, pero la descripción deberá comenzar o finalizar con la frase en inglés "This service is available in &lt;idioma del contenido de la oferta>". También puede proporcionar documentos complementarios en un idioma distinto al utilizado en la descripción de la oferta.

Para facilitar la creación de la oferta, prepare algunos de estos elementos con antelación. Los elementos siguientes son obligatorios a menos que se indique lo contrario.

**Nombre**: aparecerá como título de la descripción de la oferta en el marketplace comercial. El nombre puede ser una marca comercial. No puede contener emojis (a menos que sean símbolos de marca comercial y copyright), y no puede tener más de 50 caracteres.

**Resumen de resultados de búsqueda**: describa el propósito u objetivo de la oferta en 100 caracteres como máximo. Este resumen se usa en los resultados de búsqueda de la oferta del marketplace comercial. No debe ser idéntico al título. Considere la posibilidad de incluir las palabras clave principales para SEO.

**Descripción breve**: proporcione una descripción breve de la oferta (256 caracteres como máximo). Se mostrará en la descripción de la oferta en Azure Portal.

**Descripción**: describa la oferta en 3000 caracteres como máximo. esta descripción se mostrará en la oferta del marketplace comercial. Considere la posibilidad de incluir una propuesta de valor, las ventajas clave, las asociaciones de la categoría o el sector, y cualquier otra información necesaria.

Estas son algunas sugerencias para escribir la descripción:

* Describa claramente la propuesta de valor de la oferta en las primeras frases, incluidas:
    * Tipo de usuario que se beneficia de la oferta.
    * Necesidades o problemas del cliente que aborda la oferta.
* Recuerde que estas primeras frases podrían mostrarse en los resultados de la búsqueda.
* Use vocabulario específico del sector.

Puede usar etiquetas HTML para dar formato a la descripción. Para obtener información sobre el formato HTML, consulte el artículo sobre las [etiquetas HTML admitidas en las descripciones de las ofertas del marketplace comercial](./supported-html-tags.md).

**Vínculo de la directiva de privacidad**: proporcione una dirección URL a la directiva de privacidad, hospedada en su sitio. Usted es responsable no solo de garantizar que la oferta cumpla la normativa y la legislación en lo relativo a la privacidad, sino también de proporcionar una directiva de privacidad válida.

**Vínculos útiles** (opcional): cargue documentos en línea complementarios sobre su oferta.

**Información de contacto**: proporcione el nombre, la dirección de correo electrónico y el número de teléfono de dos personas de la empresa (usted mismo puede ser una de esas personas). Dichas personas serán un contacto de soporte técnico y otro de ingeniería. Usaremos esta información para comunicarnos con usted sobre la oferta. Esta información no se muestra a los clientes, pero se proporcionará a los partners de Proveedor de soluciones en la nube (CSP).

**Support URLs** (Direcciones URL de soporte técnico) (opcional): si tiene sitios web de soporte técnico para clientes globales de Azure o clientes de Azure Government, proporcione las direcciones URL.

**Marketplace media – logos** (Elementos multimedia de Marketplace: logotipos): proporcione un archivo PNG para el logotipo de tamaño grande de la oferta. El Centro de partners lo usará para crear logotipos de tamaño mediano y pequeño. Opcionalmente, puede reemplazar estos logotipos por una imagen diferente más adelante.

* El logotipo grande (entre 216 x 216 y 350 x 350 píxeles) aparece en la descripción de la oferta en Azure Marketplace.
* El logotipo mediano (90 x 90 píxeles) se muestra cuando se crea un recurso.
* El logotipo pequeño (48 x 48 píxeles) se usa en los resultados de búsqueda de Azure Marketplace.

Siga estas instrucciones para los logotipos:

* Asegúrese de que la imagen no quede estirada.
* El diseño de Azure tiene una paleta de colores simple. Utilice pocos colores primarios y secundarios en el logotipo.
* Los colores de Azure Portal son el blanco y el negro. No los use como fondo de su logotipo. Se recomiendan colores primarios sencillos que permitan destacar el logotipo.
* Si usa un fondo transparente, asegúrese de que el logotipo y el texto no son blancos, negros ni azules.
* La apariencia del logotipo debe ser homogénea. Evite los degradados. No coloque texto en el logotipo, ni siquiera el nombre de su empresa o de la marca.

**Marketplace media – screenshots** (Elementos multimedia de Marketplace: capturas de pantalla) (opcional): agregue hasta cinco imágenes que muestren cómo funciona la oferta. Todas las imágenes deben tener el tamaño 1280 x 720 píxeles y estar en formato PNG.

**Marketplace media – videos** (Elementos multimedia de Marketplace: vídeos) (opcional): cargue hasta cinco vídeos que muestren la oferta. Los vídeos deben estar hospedados en YouTube o Vimeo y deben tener una miniatura (archivo PNG de 1280 x 720).

## <a name="preview-audience"></a>Público preliminar

El público preliminar puede acceder a la oferta para probarla antes de que se publique en Azure Marketplace. En la página **Preview audience** (Público preliminar) del Centro de partners, puede definir un público preliminar limitado.

> [!NOTE]
> Una audiencia preliminar no es lo mismo que un plan privado. Un plan privado es el que solo está disponible para una audiencia determinada de su elección. Esto le permite negociar un plan personalizado con clientes específicos.

Puede enviar invitaciones a direcciones de correo electrónico de cuentas de Microsoft (MSA) o de Azure Active Directory (Azure AD). Agregue hasta 10 direcciones de correo electrónico manualmente o importe hasta 20 con un archivo .csv. Aunque la oferta ya esté publicada, puede definir una audiencia preliminar para probar los cambios o las actualizaciones que realice en ella.

## <a name="plans-and-pricing"></a>Planes y precios

Las ofertas de servicio administrado requieren al menos un plan. Un plan define el ámbito y los límites de la solución, así como los precios asociados, si corresponde. Puede crear varios planes para que su oferta brinde a sus clientes diferentes opciones técnicas y de precios. Encontrará instrucciones generales sobre los planes, incluidos los planes privados, en [Planes y precios de las ofertas del marketplace comercial](plans-pricing.md).

Los servicios administrados solo admiten un modelo de precios: **Traiga su propia licencia (BYOL)** . Esto significa que facturará a sus clientes directamente y que Microsoft no le cobrará ningún cargo.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una oferta de servicio administrado](./create-managed-service-offer.md)
* [Procedimientos recomendados para la publicación de ofertas](./gtm-offer-listing-best-practices.md)