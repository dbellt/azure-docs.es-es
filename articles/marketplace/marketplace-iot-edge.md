---
title: Ofertas del módulo de IoT Edge de Azure Marketplace
description: Obtenga información sobre la publicación de ofertas de módulo de IoT Edge en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 04/30/2021
ms.openlocfilehash: 70bb18984f5ed3ca6887fd01ec0e98e6f3d41595
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110547937"
---
# <a name="plan-an-iot-edge-modules-offer"></a>Planificación de una oferta de módulos de IoT Edge

La plataforma [Azure IoT Edge](https://azure.microsoft.com/services/iot-edge/) está respaldada por Microsoft Azure.  Esta plataforma permite a los usuarios implementar cargas de trabajo en la nube para ejecutarlas directamente en dispositivos IoT.  Un módulo IoT Edge puede ejecutar las cargas de trabajo sin conexión y realizar análisis de datos localmente. Este tipo de oferta ayuda a ahorrar ancho de banda, proteger datos locales y confidenciales y ofrecer un tiempo de respuesta de baja latencia.  Ahora tiene las opciones para aprovechar las ventajas de estas cargas de trabajo creadas previamente. Hasta ahora, solo estaban disponibles un puñado de soluciones propias de Microsoft.  Tenía que invertir tiempo y recursos en compilar sus propias soluciones personalizadas de IoT.

Con los [módulos de IoT Edge en Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1), ahora tenemos un único destino para que los publicadores expongan y vendan sus soluciones a la audiencia de IoT. Los desarrolladores de IoT pueden en última instancia buscar y adquirir capacidades para acelerar el desarrollo de sus soluciones.  

## <a name="key-benefits-of-iot-edge-modules-in-azure-marketplace"></a>Principales ventajas de los módulos de IoT Edge en Azure Marketplace

| **Para los publicadores**    | **Para los clientes (desarrolladores de IoT)**  |
| :------------------- | :-------------------|
| Llegue a millones de desarrolladores que buscan crear e implementar soluciones de IoT Edge.  | Cree una solución de IoT Edge con la confianza de usar de componentes probados y seguros. |
| Publique una vez y ejecute en cualquier hardware de IoT Edge que admita contenedores. | Reduzca el tiempo de comercialización al encontrar e implementar módulos de IoT Edge propios de Microsoft y de terceros para necesidades específicas. |
| Monetizar con opciones de facturación flexibles <ul><li>Gratis y Traiga su propia licencia (BYOL).</li></ul> | Realice compras con su elección de modelos de facturación.<ul><li>Gratis y Traiga su propia licencia (BYOL).</li></ul> |

## <a name="what-is-an-iot-edge-module"></a>¿Qué es un módulo IoT Edge?

Azure IoT Edge le permite implementar y administrar la lógica de negocios en el Edge a través de módulos. Los módulos de Azure IoT Edge son las unidades de cálculo más pequeñas que administra IoT Edge y pueden contener servicios de Microsoft (por ejemplo, Azure Stream Analytics), servicios de terceros o código propio específico de la solución. Para obtener más información sobre los módulos de IoT Edge, vea [Información sobre los módulos de Azure IoT Edge](../iot-edge/iot-edge-modules.md).

### <a name="what-is-the-difference-between-a-container-offer-type-and-an-iot-edge-module-offer-type"></a>¿Cuál es la diferencia entre un tipo de oferta de contenedor y un tipo de oferta de módulo de IoT Edge?

El tipo de oferta de módulo de IoT Edge es un tipo específico de contenedor que se ejecuta en un dispositivo de IoT Edge. Incluye valores de configuración predeterminados para ejecutarse en el contexto de IoT Edge y, opcionalmente, usa el SDK del módulo de IoT Edge que se integra con el tiempo de ejecución de IoT Edge.

## <a name="select-the-right-online-store"></a>Selección de la tienda en línea correcta

Los módulos de IoT Edge solo se publican en Azure Marketplace; no se aplica AppSource. Para obtener más información sobre las diferencias entre tiendas en línea, vea [Determinación de la opción de publicación](determine-your-listing-type.md).

## <a name="technical-requirements"></a>Requisitos técnicos

Los requisitos técnicos principales para que un módulo de IoT Edge pueda obtener la certificación y publicarse en Azure Marketplace se detallan en [Preparar los recursos técnicos del módulo IoT Edge](iot-edge-technical-asset.md).

## <a name="eligibility-prerequisites"></a>Requisitos previos de idoneidad

Todos los términos de los contratos de Microsoft Azure Marketplace y las directivas se aplican a las ofertas de módulo de IoT Edge.  Además, existen requisitos previos y requisitos técnicos para los módulos de IoT Edge.  

Para publicar un módulo de IoT Edge en Azure Marketplace, es preciso cumplir los siguientes requisitos previos:

- Acceso al Centro de partners. Para más información, consulte [Creación de una cuenta de marketplace comercial en el Centro de partners](create-account.md).
- Hospedar el módulo de IoT Edge en Azure Container Registry.
- Tener los metadatos del módulo IoT Edge listos como (lista no exhaustiva):
    - Un título
    - Una descripción (en formato HTML)
    - Una imagen de logotipo (en tamaños de 48 x 48 (opcional), 90 x 90 (opcional) y de 216 x 216 a 350 x 350 píxeles, todo en formato PNG)
    - Un término de uso y directiva de privacidad
    - Configuración predeterminada del módulo (ruta, propiedades deseadas del gemelo, createOptions, variables de entorno)
    - Documentación
    - Contactos de soporte técnico

## <a name="licensing-options"></a>Opciones de licencia

Estas son las opciones de licencia disponibles para las ofertas de contenedor de Azure:

| Opción de obtención de licencias | Proceso de transacción |
| --- | --- |
| Gratuito | Muestre la oferta a los clientes gratis. |
| BYOL | La opción de incorporación de licencias propias permite a los clientes aportar licencias de software existentes a Azure.\* |
|

\* Como editor, debe brindar soporte a todos los aspectos de la transacción de las licencias de software, incluidas, entre otras cosas, el pedido, la entrega, la medición, la facturación, el pago y el cobro.

## <a name="publishing-options"></a>Opciones de publicación

En todos los casos, los módulos de IoT Edge deben seleccionar la opción de publicación **Transact**.  Vea [Determinar la opción de publicación](determine-your-listing-type.md) para obtener más detalles sobre las opciones de publicación.  

## <a name="customer-leads"></a>Clientes potenciales

Al publicar una oferta en el marketplace comercial con el Centro de partners, conéctela a un sistema de administración de relaciones con clientes (CRM). Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa. Es necesario conectarse a un CRM si desea habilitar una versión de prueba; de lo contrario, la conexión a un CRM es opcional. El Centro de partners admite Tabla de Azure, Dynamics 365 Customer Engagement, Punto de conexión HTTPS, Marketo y Salesforce.

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
  - **Contacto del programa CSP** (opcional): el nombre, el teléfono y el correo electrónico si participa en el programa de proveedores de soluciones en la nube (CSP), de modo que dichos asociados puedan ponerse en contacto con usted si necesitan hacerle alguna pregunta. También puede incluir una dirección URL a sus materiales de marketing.
- **Elementos multimedia**
    - **Logotipos**: un archivo PNG para el logotipo **grande**. El Centro de partners lo usará para crear otros tamaños de logotipo necesarios. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.
    - **Capturas de pantalla**: al menos una y hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Las imágenes deben tener 1280 x 720 píxeles, en formato PNG, e incluir un título.
    - **Vídeos** (opcional): hasta cuatro vídeos que muestren la oferta. Incluya un nombre, una dirección URL para YouTube o Vimeo y una miniatura PNG de 1280 x 720 píxeles.

> [!Note]
> La oferta debe cumplir las [directivas de certificación del marketplace comercial](/legal/marketplace/certification-policies#100-general) para ser publicada en esta plataforma.

## <a name="next-steps"></a>Pasos siguientes

- Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear o completar la oferta.
- [Creación de una oferta de módulo IoT Edge](./iot-edge-offer-setup.md) en el Centro de partners.
