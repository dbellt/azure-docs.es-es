---
title: Guía de publicación para ofertas de contenedor en Azure Marketplace
description: En este artículo se describen los requisitos para publicar ofertas de contenedor en Azure Marketplace.
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 03/30/2021
ms.openlocfilehash: 43c569204f879e865eb5bd4f040c96e907e63abb
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144716"
---
# <a name="plan-an-azure-container-offer"></a>Planificación de una oferta de contenedor de Azure

Las ofertas de contenedor de Azure le ayudan a publicar la imagen de contenedor en Azure Marketplace. Use esta guía para comprender los requisitos para este tipo de oferta.

Las ofertas de contenedor de Azure son ofertas de transacciones que se implementan y facturan mediante Azure Marketplace. La opción que el usuario ve es **Obtener ahora**.

Use el tipo de oferta Contenedor de Azure cuando la solución sea una imagen de contenedor de Docker configurada como instancia de contenedor de Azure basada en Kubernetes.

> [!NOTE]
> Una instancia de Azure Container Instances es una instancia de Docker de entorno de ejecución que proporciona la forma más rápida y sencilla de ejecutar un contenedor en Azure, sin tener que administrar ninguna máquina virtual ni adoptar un servicio de nivel superior. Las instancias de contenedor se pueden implementar directamente en Azure u organizarse mediante Azure Kubernetes Services o el motor de Azure Kubernetes Service.  

## <a name="licensing-options"></a>Opciones de licencia

Estas son las opciones de licencia disponibles para las ofertas de contenedor de Azure:

| Opción de obtención de licencias | Proceso de transacción |
| --- | --- |
| Gratuito | Muestre la oferta a los clientes gratis. |
| BYOL | La opción de incorporación de licencias propias permite a los clientes aportar licencias de software existentes a Azure.\* |
|

\* Como editor, debe brindar soporte a todos los aspectos de la transacción de las licencias de software, incluidas, entre otras cosas, el pedido, la entrega, la medición, la facturación, el pago y el cobro.

## <a name="customer-leads"></a>Clientes potenciales

Al publicar una oferta en el marketplace comercial con el Centro de partners, querrá conectarla a un sistema de administración de relaciones con clientes (CRM). Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa. Es necesario conectarse a un CRM si desea habilitar una unidad de prueba; de lo contrario, la conexión a un CRM es opcional. El Centro de partners admite Tabla de Azure, Dynamics 365 Customer Engagement, Punto de conexión HTTPS, Marketo y Salesforce.

## <a name="legal-contracts"></a>Contratos legales

Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para los proveedores de software, Microsoft ofrece un contrato estándar que se puede usar con las ofertas del marketplace comercial. Al ofrecer el software bajo el contrato estándar, los clientes solo tienen que leerlo y aceptarlo, sin necesidad de crear términos y condiciones personalizados.

También puede usar sus propios términos y condiciones en lugar del contrato estándar. Los clientes deben aceptar estos términos para poder probar la oferta.

## <a name="offer-listing-details"></a>Detalles de la descripción de la oferta

> [!NOTE]
> No es necesario que el contenido de la descripción de la oferta esté en inglés si comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]".

Para facilitar la creación de la oferta, prepare estos elementos con antelación. Todos son necesarios excepto cuando se indica lo contrario.

- **Nombre**: este nombre aparecerá como título de la descripción de la oferta en el marketplace comercial. El nombre puede ser una marca comercial. No puede contener emojis (a menos que sean símbolos de marca comercial y copyright), y no puede tener más de 50 caracteres.
- **Resumen de los resultados de la búsqueda**: el propósito o la función de la oferta en una sola frase de 100 caracteres como máximo y sin saltos de línea. Se usa en los resultados de búsqueda de las ofertas del marketplace comercial.
- **Descripción breve**: detalles del propósito o la función de la oferta, escritos en texto sin formato sin saltos de línea. Aparecerá en la página de detalles de la oferta.
- **Descripción**: esta descripción se muestra en la información general de las ofertas del marketplace comercial. Considere la posibilidad de incluir una propuesta de valor, los beneficios principales, los compradores previstos, las asociaciones de categoría o sector, las oportunidades de compra en la aplicación, todas las divulgaciones necesarias y un vínculo para obtener más información. Este cuadro de texto tiene controles de editor de texto enriquecido para que su descripción sea más atractiva. Opcionalmente, use etiquetas HTML para aplicar formato.
- **Vínculo a la directiva de privacidad**: la dirección URL que lleva a la directiva de privacidad de su empresa. El usuario es responsable de garantizar que la aplicación cumpla con las leyes y los reglamentos de privacidad.
- **Vínculos útiles** (opcional): vínculos a varios recursos para los usuarios de su oferta. Por ejemplo, foros, preguntas más frecuentes y notas de la versión.
- **Información de contacto**
  - **Contacto de soporte técnico**: el nombre, el teléfono y el correo electrónico que los asociados de Microsoft se usarán cuando los clientes abran incidencias. Incluya la dirección URL del sitio web de soporte técnico.
  - **Contacto de ingeniería**: el nombre, el teléfono y el correo electrónico que Microsoft usará directamente cuando haya problemas con la oferta. Esta información de contacto no aparece en el marketplace comercial.
  - **Contacto del programa CSP** (opcional): el nombre, el teléfono y el correo electrónico si participa en el programa CSP, de modo que dichos asociados puedan ponerse en contacto con usted con cualquier pregunta. También puede incluir una dirección URL a sus materiales de marketing.
- **Elementos multimedia**
    - **Logotipos**: un archivo PNG para el logotipo **grande**. El Centro de partners lo usará para crear otros tamaños de logotipo necesarios. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.
    - **Capturas de pantalla**: al menos una y hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Las imágenes deben tener 1280 x 720 píxeles, en formato PNG, e incluir un título.
    - **Vídeos** (opcional): hasta cuatro vídeos que muestren la oferta. Incluya un nombre, una dirección URL para YouTube o Vimeo y una miniatura PNG de 1280 x 720 píxeles.

> [!Note]
> La oferta debe cumplir las [directivas de certificación del marketplace comercial](/legal/marketplace/certification-policies#100-general) para ser publicada en esta plataforma.

## <a name="preview-audience"></a>Público preliminar

Una audiencia preliminar puede tener acceso a la oferta antes de publicarla en las tiendas en línea a fin de probar la funcionalidad completa. En la página **Audiencia preliminar**, puede definir una audiencia preliminar limitada.

Puede enviar invitaciones a los identificadores de suscripción de Azure. Agregue hasta 10 identificadores manualmente o importe hasta 100 con un archivo .csv. Aunque la oferta ya esté publicada, puede definir una audiencia preliminar para probar los cambios o las actualizaciones que realice en ella.

## <a name="plans-and-pricing"></a>Planes y precios

Las ofertas de contenedor requieren al menos un plan. Un plan define el ámbito y los límites de la solución. Puede crear varios planes para que su oferta brinde a sus clientes diferentes opciones técnicas y de licencias. 

Los contenedores admiten dos modelos de licencia: Gratis o Traiga su propia licencia (BYOL). BYOL significa que facturará a sus clientes directamente y que Microsoft no le cobrará ningún cargo. Microsoft solo traspasa los honorarios de uso de la infraestructura de Azure. Para obtener más información, consulte [Capacidades de transacción de marketplace comercial](marketplace-commercial-transaction-capabilities-and-considerations.md).

## <a name="additional-sales-opportunities"></a>Oportunidades de venta adicionales

Puede optar por participar en los canales de marketing y ventas respaldados por Microsoft. Al crear la oferta en el Centro de partners, verá dos pestañas hacia el final del proceso:

- **Revender mediante los CSP**: permita que los asociados de los proveedores de soluciones en la nube (CSP) de Microsoft revendan la solución como parte de una oferta agrupada. Para más información sobre este programa, vea el [programa de proveedores de soluciones en la nube](cloud-solution-providers.md).
- **Venta conjunta con Microsoft**: permita que los equipos de ventas de Microsoft consideren la solución idónea para la venta conjunta de IP al evaluar las necesidades de los clientes. Para obtener más información sobre la elegibilidad de la venta conjunta, vea [Requisitos para el estado de la venta conjunta](/legal/marketplace/certification-policies). Para información detallada sobre cómo preparar la oferta para la evaluación, vea [Opción de venta conjunta en el Centro de partners](./co-sell-configure.md).

## <a name="container-offer-requirements"></a>Requisitos de la oferta de contenedor

| Requisito | Detalles |  
|:--- |:--- |  
| Facturación y medición | Admita el modelo de facturación gratuito o traiga su propia licencia (BYOL). |
| Imagen compilada a partir de un Dockerfile | Las imágenes de contenedor se deben basar en la especificación de imagen de Docker y se deben compilar a partir de un Dockerfile. Para obtener más información sobre cómo crear imágenes de Docker, consulte la sección "Uso" de [referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/#usage). |
| Hospedaje en un repositorio de Azure Container Registry | Las imágenes de contenedor se deben hospedar en un repositorio de Azure Container Registry. Para obtener más información sobre cómo trabajar con Azure Container Registry, consulte [Inicio rápido: Creación de un registro de contenedor privado con Azure Portal](../container-registry/container-registry-get-started-portal.md).<br><br> |
| Etiquetado de imágenes | Las imágenes de contenedor deben contener al menos 1 etiqueta (número máximo de etiquetas: 16). Para obtener más información sobre cómo etiquetar una imagen, consulte la página de `docker tag` en el sitio de [documentación de Docker](https://docs.docker.com/engine/reference/commandline/tag). |

## <a name="next-steps"></a>Pasos siguientes

- [Preparación de recursos técnicos](azure-container-technical-assets.md)