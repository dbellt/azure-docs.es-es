---
title: Crea una oferta de prueba
description: Cómo crear una oferta de desarrollo independiente para probar la oferta de producción en el programa de Marketplace comercial en el Centro de partners de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 04/20/2021
ms.openlocfilehash: 318eebcca3053be2164a1fc52905f36f26b11a6d
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108331131"
---
# <a name="create-a-test-offer"></a>Crea una oferta de prueba

Para desarrollar en un entorno independiente de la oferta de producción, creará una oferta de prueba y desarrollo (DEV) independiente y una oferta de producción independiente (PROD). Para obtener información sobre las ventajas de usar una oferta de desarrollo independiente, consulte [Plan de una oferta de SaaS](plan-saas-offer.md#test-offer).

La mayoría de los ajustes se configurarán de la misma manera en las ofertas de desarrollo y producción. Por ejemplo, el lenguaje de marketing oficial y los activos, como capturas de pantalla y logotipos, deben ser los mismos. En los casos en los que la configuración es la misma, puede copiar y pegar los campos de los planes de la oferta de desarrollo a los planes de la oferta de producción.

En las secciones siguientes se describen las diferencias de configuración entre las ofertas de desarrollo y producción.

## <a name="offer-setup-page"></a>Página de configuración de la oferta

Se recomienda usar el mismo alias en el cuadro **Alias** de ambas ofertas, pero anexar "_test" al alias de la oferta de desarrollo. Por ejemplo, si el alias de la oferta de producción es "contososolution", el alias de la oferta de desarrollo debe ser "contososolution_test". De este modo, puede identificar fácilmente qué oferta de desarrollo de su oferta de producción.

En la sección **clientes potenciales**, use una tabla de Azure o un entorno de prueba de CRM para la oferta de desarrollo. Use el sistema de administración de clientes potenciales para la oferta de producción.

## <a name="properties-page"></a>Página de propiedades

Configure esta página de la misma manera que en las ofertas de desarrollo y de producción.

## <a name="offer-listing-page"></a>Página de descripción de la oferta

Configure esta página de la misma manera que en las ofertas de desarrollo y de producción.

## <a name="preview-audience"></a>Público preliminar

En la oferta de desarrollo, incluya los nombres principales de usuario de Azure Active Directory (AAD) o las direcciones de correo electrónico de cuenta Microsoft (MSA) de los desarrolladores y evaluadores, incluido usted mismo. El nombre principal de usuario de un usuario en AAD puede ser diferente del correo electrónico de dicho usuario. Por ejemplo, jane.doe@contoso.com no funcionará, pero janedoe@contoso.com si funcionará. Los usuarios que designen tendrán acceso a la oferta de desarrollo cuando comparta el vínculo de **versión preliminar** durante la fase de desarrollo y pruebas.

En la oferta de producción, incluya el nombre principal de usuario de Azure AD o el correo electrónico de la cuenta de Microsoft de los usuarios que validarán la oferta antes de seleccionar el **botón de Go Live** para publicar la oferta en directo.

## <a name="technical-configuration-page"></a>Página de configuración técnica

En esta tabla se describen las diferencias entre la configuración de las ofertas de desarrollo y las ofertas de producción.

***Tabla 1: Diferencias de configuración técnica***

| Configuración | Oferta de desarrollo | Oferta de producción |
| ------------ | ------------- | ------------- |
| Dirección URL de la página de aterrizaje | Escriba el punto de conexión de desarrollo/pruebas. | Escriba el punto de conexión de producción. |
| Webhook de conexión | Escriba el punto de conexión de desarrollo/pruebas. | Escriba el punto de conexión de producción. |
| Id. del inquilino de Azure Active Directory | Escriba el id. del inquilino de registro de la aplicación de prueba (id. de directorio de AAD). | Escriba el id. del inquilino de registro de la aplicación de producción. |
| Identificador de la aplicación de Azure Active Directory | Escriba el id. de la aplicación de registro de la aplicación de prueba (id. de cliente). | Escriba el id. de la aplicación de registro de la aplicación de producción. |
||||

## <a name="plan-overview-page"></a>Página de información general del plan

Al crear los planes, se recomienda usar el mismo _id. de Plan_ y el mismo _Nombre de plan_ en las ofertas de desarrollo y producción, excepto anexar el id. de plan en la oferta de desarrollo con **_test**. Por ejemplo, si el id. de Plan de la oferta de producción es "enterprise", el id. de plan de la oferta de desarrollo debe ser "enterprise_test". De este modo, puede identificar fácilmente qué oferta de desarrollo de su oferta de producción. Creará planes en la oferta de producción con los modelos de precios y precios que decida que son mejores para su oferta.

### <a name="plan-listing"></a>Lista del plan

En la pestaña **Descripción general del plan** >  **Lista de planes**, escriba la misma descripción del plan en los planes de desarrollo y producción.

### <a name="pricing-and-availability-page"></a>Página Precios y disponibilidad

En esta sección se proporciona una guía para completar la página **Descripción general del plan** >  **Precios y disponibilidad**.

#### <a name="markets"></a>Mercados

Seleccione los mismos mercados para las ofertas de desarrollo y producción.

#### <a name="pricing"></a>Precios

Use la oferta de desarrollo para experimentar con modelos de precios. Después de comprobar qué modelo de precios o modelos funcionan mejor, creará los planes en la oferta de producción con los modelos de precios y los precios que desee.

La oferta de desarrollo debe tener planes con un precio de cero o inferior en los planes. La oferta de producción tendrá los precios que desea cobrar a los clientes.

> [!IMPORTANT]
> Las compras realizadas en versión preliminar se procesarán para las ofertas de desarrollo y de producción. Si una oferta tiene un precio de $100 al mes, se cobrará a su empresa $100. Si esto ocurre, puede abrir una [incidencia de soporte técnico](support.md) y se le enviará un pago para la cantidad completa (y no se aplicará ningún honorario por el servicio de tienda).

#### <a name="pricing-model"></a>Modelo de precios

Use el mismo modelo de precios en los planes de las ofertas de desarrollo y producción. Por ejemplo, si el plan de la oferta de producción es una tarifa plana, con un término de facturación mensual, configure el plan en la oferta de desarrollo con el mismo modelo.

Para reducir el costo de probar los modelos de precios, incluidas las dimensiones del medidor personalizado de Marketplace, se recomienda configurar la sección de **Precios** de la pestaña **Precios y disponibilidad**, en la oferta de desarrollo con precios más bajos que la oferta de producción. Estas son algunas directrices que puede seguir al establecer los precios de los planes en la oferta de desarrollo.

***Tabla 2: Directrices de precios***

| Precio | Comentario |
| ------------ | ------------- |
| $0.00 | Establezca un costo total de transacción de cero para que no tenga ningún impacto financiero. Use este precio al realizar llamadas a las API de medición o para probar planes de compra en su oferta mientras desarrolla la solución. |
| $0.01 - $49.99 | Use este rango de precios para probar el análisis, los informes y el proceso de compra. |
| 50,00 $ - 100,00 $ | Use este rango de precios para probar el pago. Para obtener información sobre nuestra programación de pago, consulte [Programaciones y procesos de pago](/partner-center/payout-policy-details). |
|||

> [!IMPORTANT]
>  Para evitar que se le cobren honorarios de servicio de tienda en la prueba, abra una [incidencia de soporte técnico](support.md) en un plazo de 7 días desde la compra de la prueba.

#### <a name="free-trial"></a>Evaluación gratuita

No habilite una evaluación gratuita para la oferta de desarrollo.

## <a name="co-sell-with-microsoft-page"></a>Venta conjunta con la página de Microsoft

No configure la pestaña de **Venta conjunta con Microsoft** de la oferta de desarrollo.

## <a name="resell-through-csps"></a>Revender mediante los CSP

No configure la pestaña **Revender a través de CSP** de la oferta de desarrollo.

## <a name="next-steps"></a>Pasos siguientes

- [Prueba y publicación de ofertas de SaaS](test-publish-saas-offer.md)
