---
title: Planeamiento de una oferta de Aplicación de Azure para el marketplace comercial
description: Aprenda a planear una nueva oferta de Aplicación de Azure para mostrarla o venderla en Azure Marketplace o mediante el programa de proveedores de soluciones en la nube (CSP) usando el portal del marketplace comercial en el Centro de partners de Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: 8120bcb030f5a1ff6948757d8f65e3a21d31898f
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955398"
---
# <a name="plan-an-azure-application-offer-for-the-commercial-marketplace"></a>Planeamiento de una oferta de Aplicación de Azure para el marketplace comercial

En este artículo se explican los distintos requisitos y opciones para publicar una oferta de Aplicación de Azure en el marketplace comercial de Microsoft.

## <a name="before-you-begin"></a>Antes de empezar

Para diseñar, compilar y probar ofertas de aplicaciones de Azure, se necesitan conocimientos técnicos de la plataforma de Azure y de las tecnologías que se usan para crear la oferta. El equipo de ingeniería debe tener conocimientos sobre las tecnologías de Microsoft siguientes:

- Conocimientos básicos de los [Servicios de Azure](https://azure.microsoft.com/services/).
- Cómo [diseñar y estructurar las aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/).
- Conocimientos prácticos de [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage#storage) y [redes de Azure](https://azure.microsoft.com/services/?filter=networking#networking).
- Conocimientos prácticos de [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/).
- Conocimientos prácticos de [JSON](https://www.json.org/).

### <a name="technical-documentation-and-resources"></a>Documentación y recursos técnicos

Mientras prepara su oferta de Aplicación de Azure para el marketplace comercial, revise los siguientes recursos.

- [Nociones sobre las plantillas de Azure Resource Manager](../azure-resource-manager/templates/syntax.md)
- Guías de inicio rápido:
    - [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/)
    - [Guía de procedimientos recomendados para plantillas de Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md)
    - [Publicación de definición de aplicación](../azure-resource-manager/managed-applications/publish-service-catalog-app.md)
    - [Implementación de la aplicación de catálogo de servicios](../azure-resource-manager/managed-applications/deploy-service-catalog-quickstart.md)
- Tutoriales:
    - [Creación de archivos de definición](../azure-resource-manager/managed-applications/publish-service-catalog-app.md)
- Ejemplos:
    - [CLI de Azure](../azure-resource-manager/managed-applications/cli-samples.md)
    - [Azure PowerShell](../azure-resource-manager/managed-applications/powershell-samples.md)
    - [Soluciones para aplicaciones administradas](../azure-resource-manager/managed-applications/sample-projects.md)

En el vídeo [Creación de plantillas de soluciones y aplicaciones administradas para Azure Marketplace](https://channel9.msdn.com/Events/Build/2018/BRK3603) se proporciona una completa introducción al tipo de oferta de aplicación de Azure:

- Qué tipos de oferta están disponibles
- Qué recursos técnicos se necesitan
- Cómo crear una plantilla de Azure Resource Manager
- Desarrollo y prueba de la interfaz de usuario de la aplicación
- Cómo publicar la oferta de la aplicación
- El proceso de revisión de la aplicación

### <a name="suggested-tools"></a>Herramientas sugeridas

Elija uno o ambos de los entornos de scripting siguientes para ayudar a administrar la aplicación de Azure:

- [Azure PowerShell](/powershell/azure/)
- [CLI de Azure](/cli/azure)

Se recomienda agregar las herramientas siguientes al entorno de desarrollo:

- [Explorador de Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md)
- [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    - Extensión: [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
    - Extensión: [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
    - Extensión: [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json)

Puede revisar las herramientas disponibles en la página [Herramientas de desarrollo de Azure](https://azure.microsoft.com/tools/). Si usa Visual Studio, visite [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="listing-options"></a>Opciones de publicación

Una vez publicada la oferta, sus opciones de descripción aparecen en forma de un botón en la esquina superior izquierda de la página de descripción de la oferta. Por ejemplo, en la siguiente captura de pantalla se muestra una página de descripción de la oferta en Azure Marketplace con el botón _Obtenerla ahora_. Si ha elegido ofrecer una versión de prueba, también se mostrará el botón _Versión de prueba_.

:::image type="content" source="media/create-new-azure-app-offer/azure-app-listing-page.png" alt-text="Se muestra una página de descripción en Azure Marketplace.":::

## <a name="test-drive"></a>Versión de prueba

Puede optar por habilitar una versión de prueba para la oferta de Aplicación de Azure que permita a los clientes probarla antes de adquirirla. Para más información sobre las versiones de prueba, consulte [¿Qué es una versión de prueba?](what-is-test-drive.md). Para información sobre la configuración de diferentes tipos de versiones de prueba, consulte [Configuración técnica de la versión de prueba](test-drive-technical-configuration.md).

También puede leer sobre los [procedimientos recomendados de la versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices) y descargar el PDF de [información general de las versiones de prueba ](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (asegúrese de que el bloqueador de elementos emergentes está desactivado).

> [!NOTE]
> Dado que todas las aplicaciones de Azure se implementan mediante una plantilla de Azure Resource Manager, el único tipo de versión de prueba disponible para una [aplicación de Azure es una versión de prueba basada en Azure Resource Manager](azure-resource-manager-test-drive.md).

## <a name="customer-leads"></a>Clientes potenciales

Debe conectar su oferta al sistema de administración de relaciones con clientes (CRM) para recopilar información del cliente. El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre y el identificador de la oferta, además de la tienda en línea donde la encontró, se envían al sistema CRM que haya configurado. El marketplace comercial admite una gran variedad de sistemas CRM, junto con la opción de usar una tabla de Azure o de configurar un punto de conexión HTTPS con Power Automate.

Puede agregar o modificar una conexión CRM en cualquier momento durante o después de la creación de la oferta. Para obtener instrucciones detalladas, vea [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

## <a name="categories-and-subcategories"></a>Categorías y subcategorías

Puede elegir una categoría como mínimo y dos como máximo para agrupar la oferta en las áreas de búsqueda apropiadas del marketplace comercial. Puede elegir hasta dos subcategorías por cada categoría principal y secundaria. Para ver la lista completa de categorías y subcategorías, consulte [Procedimientos recomendados para la descripción de ofertas](gtm-offer-listing-best-practices.md#categories).

## <a name="legal-contracts"></a>Contratos legales

Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para los proveedores de software, Microsoft ofrece un contrato estándar que se puede usar con las ofertas del marketplace comercial. Al ofrecer el software bajo el contrato estándar, los clientes solo tienen que leerlo y aceptarlo, sin necesidad de crear términos y condiciones personalizados.

Si decide usar el contrato estándar, tiene la opción de agregarle términos de modificación universales y hasta 10 modificaciones personalizadas. También puede usar sus propios términos y condiciones en lugar del contrato estándar. Estos detalles se administrarán en la página **Propiedades**. Para más información, consulte [Contrato estándar para el marketplace comercial de Microsoft](standard-contract.md).

> [!NOTE]
> Después de publicar una oferta con el contrato estándar en el marketplace comercial, no puede usar sus propios términos y condiciones personalizados. Solo puede elegir una de las dos opciones. O bien ofrecer su solución bajo el contrato estándar o según sus propios términos y condiciones. Si quiere modificar los términos del contrato estándar, puede hacerlo a través de las modificaciones al contrato estándar.

## <a name="offer-listing-details"></a>Detalles de la descripción de la oferta

Al crear una oferta de Aplicación de Azure en el Centro de partners, insertará texto, imágenes, vídeos opcionales y otros detalles en la página de descripción de la oferta. Esta es la información que verán los clientes cuando descubran la descripción de la oferta en Azure Marketplace, como se muestra en el ejemplo siguiente.

:::image type="content" source="media/create-new-azure-app-offer/example-azure-marketplace-app.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Categorías
3. Dirección de soporte técnico (vínculo)
4. Términos de uso
5. Dirección de la directiva de privacidad (vínculo)
6. Nombre de la oferta
7. Resumen
8. Descripción
9. Capturas de pantallas o vídeos

En la captura de pantalla siguiente se muestra cómo aparece la información de la oferta en Azure Portal:

[![Ilustración de cómo aparece esta oferta en Azure Portal.](media/create-new-azure-app-offer/example-virtual-machine-container-iot-edge-saas.png)](media/create-new-azure-app-offer/example-virtual-machine-container-iot-edge-saas.png#lightbox)

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Título
2. Descripción
3. Vínculos útiles
4. Capturas de pantalla

> [!NOTE]
> No es necesario que el contenido de la descripción de la oferta esté en inglés si comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]".

Para facilitar la creación de la oferta, prepare algunos de estos elementos con antelación. Los elementos siguientes son obligatorios a menos que se indique lo contrario.

- **Name**: este nombre aparecerá como título de la descripción de la oferta en el marketplace comercial. El nombre puede ser una marca comercial. No puede contener emojis (a menos que sean símbolos de marca comercial y copyright), y no puede tener más de 50 caracteres.
- **Resumen de los resultados de la búsqueda**: describa el propósito o la función de la oferta en una sola frase, en texto sin formato, de 100 caracteres como máximo y sin saltos de línea. Este resumen se usa en los resultados de búsqueda de las ofertas del marketplace comercial.
- **Descripción breve**: proporcione hasta 256 caracteres de texto sin formato. Este resumen aparecerá en la página de detalles de la oferta.
- **Descripción**: esta descripción se mostrará en la información general de las descripciones de ofertas de Azure Marketplace. Considere la posibilidad de incluir una propuesta de valor, las ventajas principales, los compradores previstos, las asociaciones de categoría o sector, las oportunidades de compra dentro de la aplicación, las necesidades o problemas que soluciona la oferta, todas las divulgaciones necesarias y un vínculo a más información.

    Este cuadro de texto tiene controles de editor de texto enriquecido que puede utilizar para que su descripción sea más atractiva. También puede usar etiquetas HTML para dar formato a la descripción. En este cuadro puede escribir hasta 3000 caracteres de texto, lo que incluye el marcado HTML y espacios. Puede encontrar otras sugerencias en [Escribir una excelente descripción de la aplicación](/windows/uwp/publish/write-a-great-app-description) y [Etiquetas HTML admitidas en las descripciones de las ofertas del marketplace comercial](supported-html-tags.md).

- **Palabras clave de búsqueda** (opcional): escriba al menos tres palabras clave de búsqueda que los clientes puedan usar para buscar su oferta en la tienda en línea. Para obtener mejores resultados, intente usar también estas palabras clave en la descripción. No es necesario incluir el **nombre** y **descripción** de la oferta. Ese texto se incluye automáticamente en la búsqueda.
- **Vínculo a la directiva de privacidad**: la dirección URL que lleva a la directiva de privacidad de su empresa. Debe proporcionar una directiva de privacidad válida y usted es responsable de garantizar que la aplicación cumple con las leyes y normativas de privacidad.
- **Vínculos útiles** (opcional): puede proporcionar vínculos a varios recursos para los usuarios de su oferta. Por ejemplo, foros, preguntas más frecuentes y notas de la versión.
- **Información de contacto**: debe designar los siguientes contactos de la organización:
  - **Contacto de soporte técnico**: proporcione el nombre, el teléfono y el correo electrónico de los asociados de Microsoft que se usarán cuando los clientes abran vales. También debe incluir la dirección URL del sitio web de soporte técnico.
  - **Contacto de ingeniería**: proporcione el nombre, el teléfono y el correo electrónico que Microsoft usará directamente cuando haya problemas con la oferta. Esta información de contacto no aparece en el marketplace comercial.
  - **Contacto del programa CSP** (opcional): proporcione el nombre, el teléfono y el correo electrónico si participa en el programa de proveedores de soluciones en la nube (CSP), de modo que dichos asociados puedan ponerse en contacto con usted si necesitan hacerle alguna pregunta. También puede incluir una dirección URL a sus materiales de marketing.
- **Elementos multimedia (logotipos)** : Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear un logotipo **Pequeño** y un logotipo **Mediano**. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.
  - Grande (de 216 x 216 a 350 x 350 píxeles, obligatorio)
  - Mediano (90 x 90 píxeles, opcional)
  - Pequeño (48 x 48 píxeles, opcional)

  Estos logotipos se usan en distintos lugares de las tiendas en línea:
  - El logotipo pequeño aparece en los resultados de búsqueda de Azure Marketplace.
  - El logotipo mediano aparece cuando se crea un recurso en Microsoft Azure.
  - El logotipo grande aparece en la página de descripción de la oferta de Azure Marketplace.

  Siga estas instrucciones para los logotipos:

  - El diseño de Azure tiene una paleta de colores simple. Utilice pocos colores primarios y secundarios en el logotipo.
  - Los colores del tema del portal son el blanco y el negro. No use estos colores como fondo de los logotipos. Use un color que haga que su logotipo destaque en el portal. Nosotros recomendamos usar colores primarios simples.
  - Si usa un fondo transparente, asegúrese de que el logotipo y el texto no son blancos, negros ni azules.
  - La apariencia del logotipo debe ser homogénea y evitar degradados en el logotipo o el fondo. No coloque texto en el logotipo, ni siquiera el nombre de su empresa o de la marca. Las imágenes borrosas harán que se rechace el envío.
  - Asegúrese de que el logotipo no quede estirado.

- **Elementos multimedia: capturas de pantalla** (opcional): se recomienda agregar capturas de pantallas que muestren el funcionamiento de la oferta. Puede agregar entre una y cinco capturas de pantallas que muestren el funcionamiento de la oferta, con los siguientes requisitos:
  - 1280 x 720 píxeles
  - Archivo .png
  - Se debe incluir un título
- **Elementos multimedia: vídeos** (opcional): puede agregar hasta cinco vídeos que muestren su oferta, con los siguientes requisitos:
  - Nombre
  - Dirección URL: solo se debe hospedar en YouTube o en Vimeo.
  - Miniatura: archivo .png de 1280 x 720

> [!NOTE]
> La oferta debe cumplir las [directivas de certificación del marketplace comercial](/legal/marketplace/certification-policies#100-general.md) para ser publicada en esta plataforma.

## <a name="preview-audience"></a>Público preliminar

Una audiencia preliminar puede tener acceso a la oferta antes de publicarla en las tiendas en línea a fin de probar la funcionalidad completa.

> [!NOTE]
> Una audiencia preliminar no es lo mismo que un plan privado. Un plan privado es el que solo está disponible para una audiencia determinada de su elección. Esto le permite negociar un plan personalizado con clientes específicos.

Especifique la audiencia preliminar mediante los identificadores de suscripción de Azure, junto con una descripción opcional de cada uno. Los clientes no pueden ver ninguno de estos campos.

## <a name="technical-configuration"></a>Configuración técnica

En el caso de las aplicaciones administradas que emiten eventos de medición mediante las [API del servicio de medición de Marketplace](marketplace-metering-service-apis.md), debe proporcionar la identidad que usará el servicio al emitir dichos eventos.

Esta configuración es necesaria si quiere usar [eventos de uso de Batch](marketplace-metering-service-apis.md#metered-billing-batch-usage-event). En caso de que quiera enviar [eventos de uso](marketplace-metering-service-apis.md#metered-billing-single-usage-event), también puede emplear el [servicio de metadatos de instancia](../active-directory/managed-identities-azure-resources/overview.md) para obtener el [token de portador de JSON Web Token (JWT)](partner-center-portal/pc-saas-registration.md#how-to-get-the-publishers-authorization-token).

- **Identificador de inquilino de Azure Active Directory** (obligatorio): En Azure Portal, debe [crear una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md) para que podamos validar que la conexión entre nuestros dos servicios está detrás de una comunicación autenticada. Para encontrar el [identificador de inquilino](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de la aplicación de Azure Active Directory (Azure AD), vaya a la hoja [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) de Azure Active Directory. En la columna **Nombre para mostrar**, seleccione la aplicación. A continuación, busque **Propiedades** y, luego, el **identificador de directorio (inquilino)** (por ejemplo `50c464d3-4930-494c-963c-1e951d15360e`).
- **Identificador de aplicación de Azure Active Directory** (obligatorio): También necesita su [identificador de la aplicación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) y una clave de autenticación. Para encontrar el identificador de la aplicación, vaya a la hoja [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) en su instancia de Azure Active Directory. En la columna **Nombre para mostrar**, seleccione la aplicación y, a continuación, busque el **identificador de aplicación (cliente)** (por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`). Para buscar la clave de autenticación, vaya a **Configuración** y seleccione **Claves**. Deberá proporcionar una descripción y una duración y, después, se proporcionara un valor numérico.

> [!NOTE]
> El identificador de aplicación de Azure se asociará a su identificador de editor y solo se podrá volver a usar en la cuenta de este editor.

## <a name="additional-sales-opportunities"></a>Oportunidades de venta adicionales

Puede optar por participar en los canales de marketing y ventas respaldados por Microsoft. Al crear la oferta en el Centro de partners, verá dos pestañas hacia el final del proceso:

- **Resell through CSPs** (Revender a través de los CSP): use esta opción para permitir que los asociados de los proveedores de soluciones en la nube (CSP) de Microsoft revendan la solución como parte de una oferta agrupada. Para más información, consulte [Programa del Proveedor de soluciones en la nube](./cloud-solution-providers.md).
- **Venta conjunta con Microsoft**: esta opción permite que los equipos de ventas de Microsoft consideren la solución idónea para la venta conjunta de IP al evaluar las necesidades de los clientes. Para obtener información detallada sobre cómo preparar la oferta para la evaluación, vea [Opción de venta conjunta en el marketplace comercial](./co-sell-configure.md). Para obtener más información sobre los requisitos de la venta conjunta de IP, vea [Requisitos para el estado de la venta conjunta](/legal/marketplace/certification-policies#3000-requirements-for-co-sell-status). Para más información sobre cómo comercializar su oferta a través de los canales de asociados de CSP de Microsoft, consulte [Proveedores de soluciones en la nube](cloud-solution-providers.md).

Para más información, consulte [Desarrollo de un negocio en la nube con Azure Marketplace](https://azuremarketplace.microsoft.com/sell).

## <a name="plans"></a>Planes

Las ofertas de Aplicación de Azure requieren al menos un plan. Un plan define el ámbito y los límites de la solución, así como los precios asociados, si corresponde. Puede crear varios planes para que su oferta brinde a sus clientes diferentes opciones técnicas y de precios.

Puede encontrar instrucciones generales sobre los planes, incluidos los modelos de precios y los planes privados en [Planes y precios de ofertas de marketplace comercial](plans-pricing.md). En las secciones siguientes se describe información adicional específica de los planes de Aplicación de Azure.

### <a name="types-of-plans"></a>Tipos de planes

Hay dos tipos de planes de la Aplicación de Azure: _plantilla de solución_ y _aplicación administrada_. Ambos tipos de planes admiten la automatización de la implementación y la configuración de una solución más allá de una sola máquina virtual (VM). Puede automatizar el proceso de suministrar varios recursos, incluidas VM, redes y recursos de almacenamiento, para proporcionar soluciones complejas, como soluciones IaaS. Ambos tipos de planes pueden emplear muchos tipos diferentes de recursos de Azure, incluidas, entre otras, VM.

- Los planes de **plantilla de solución** son uno de los principales mecanismos para publicar una solución en el marketplace comercial. Los planes de plantilla de solución no se pueden comercializar en el marketplace comercial, pero se pueden usar para implementar ofertas de VM de pago facturadas mediante el marketplace comercial. Use el tipo de plan de plantilla de solución cuando el cliente administre la solución y las transacciones se facturan a través de otro plan. Para más información sobre la creación de plantillas de solución, consulte [¿Qué es Azure Resource Manager?](../azure-resource-manager/management/overview.md)
- Los planes de **aplicación administrada** le permiten crear y entregar con facilidad a los clientes aplicaciones llave en mano totalmente administradas. Tienen las mismas funcionalidades que los planes de plantilla de solución, con algunas diferencias clave:
    - Los recursos se implementan en un grupo de recursos y están administrados por el editor de la aplicación. El grupo de recursos está presente en la suscripción del consumidor, pero una identidad en el inquilino del editor tiene acceso al grupo de recursos. 
    - Como editor, especifica el costo del soporte técnico continuo de la solución, y las transacciones se admiten a través del marketplace comercial.
 
    Use el tipo de plan de aplicación administrada cuando usted o su cliente requieran que un partner administre la solución, o cuando vayan a implementar una solución basada en suscripciones. Para más información sobre las ventajas y los tipos de aplicaciones administradas, consulte [Introducción a las aplicaciones administradas de Azure](../azure-resource-manager/managed-applications/overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Para planear una plantilla de solución, consulte [Planeamiento de una plantilla de solución para una oferta de Aplicación de Azure](plan-azure-app-solution-template.md).
- Para planear una aplicación administrada de Azure, consulte [Planeamiento de una aplicación administrada de Azure para una oferta de Aplicación de Azure](plan-azure-app-managed-app.md).