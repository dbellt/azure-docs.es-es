---
title: Creación de una oferta de Dynamics 365 for Customer Engagement y PowerApps en el marketplace comercial de Microsoft
description: Creación de una oferta de Dynamics 365 for Customer Engagement & PowerApps en Microsoft AppSource. Publique o venda su oferta en Azure Marketplace o mediante el programa Proveedor de soluciones en la nube (CSP).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: navits09
ms.author: navits
ms.date: 12/02/2020
ms.openlocfilehash: 1661fd78812bff14d9efddf9ef1c31a359aa959d
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107370"
---
# <a name="create-a-dynamics-365-for-customer-engagement--powerapps-offer"></a>Creación de una oferta de Dynamics 365 for Customer Engagement y PowerApps

En este tema se explica cómo crear una oferta de Dynamics 365 for Customer Engagement y PowerApps. Todas las aplicaciones de Dynamics 365 for Customer Engagement (PowerApps, Sales, Service, Project Service y Field Service) deben pasar por nuestro proceso de certificación, que comprueba la solución en busca de requisitos estándar, compatibilidad y prácticas adecuadas. La experiencia de evaluación permite a los usuarios implementar la solución en un entorno de Dynamics 365 en vivo.

Antes de comenzar, [cree una cuenta de marketplace comercial en el Centro de partners](../create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa Marketplace comercial.

>[!NOTE]
> Una vez publicada la oferta, las modificaciones en ella solo se actualizarán en el Centro de partners y en la tienda en línea después de volver a enviar la oferta para su publicación.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página Información general, seleccione **+ Nueva oferta** > **Dynamics 365 for Customer Engagement y PowerApps**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-dynamics-365-customer-engagement-powerapps.png)

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
- El identificador de oferta en combinación con el del editor debe tener una longitud de menos de 40 caracteres.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios. Por ejemplo, si el identificador del editor es `testpublisherid` y escribe **test-offer-1**, la dirección web de la oferta será `https://appsource.microsoft.com/product/dynamics-365/testpublisherid.test-offer-1`.
- Este identificador no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- Este nombre no se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-setup"></a>Configuración de la oferta

### <a name="alias"></a>Alias

Escriba un nombre descriptivo que se usará para hacer referencia a esta oferta únicamente dentro del Centro de partners. Este nombre (previamente rellenado con lo que especificó al crear la oferta) no se usará en el marketplace y es diferente del nombre de la oferta que se muestra a los clientes. Si quiere actualizar el nombre de la oferta más adelante, vaya a la página [Offer Listing](#offer-listing) (Descripción de la oferta).

### <a name="setup-details"></a>Detalles de la configuración

En **¿Cómo quiere que los clientes potenciales interactúen con esta oferta?** , seleccione la opción que quiere usar para esta oferta.

- **Obtener ahora (gratis)** : la oferta se muestra a los clientes de forma gratuita.
- **Free trial (listing)** (Evaluación gratuita [descripción]): la oferta se muestra a los clientes con un vínculo a una evaluación gratuita. Las ofertas que incluyen evaluaciones gratuitas se crean, administran y configuran desde su servicio y no tienen suscripciones administradas por Microsoft.

    > [!NOTE]
    > Los tokens que la aplicación recibirá a través del vínculo de evaluación gratuita solo se pueden usar para obtener información de usuario mediante Azure Active Directory (Azure AD) para automatizar la creación de cuentas en la aplicación. No se admiten las cuentas de Microsoft para la autenticación con este token.

- **Contacto**: recopile información de contacto de los clientes mediante la conexión de su sistema de Administración de relaciones con los clientes (CRM). El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre e identificador de la oferta y el origen de Marketplace donde la ha encontrado, se envían al sistema CRM que se ha configurado. Para más información sobre cómo configurar el sistema CRM, consulte [Clientes potenciales](#customer-leads).

### <a name="test-drive"></a>Versión de prueba

Una versión de prueba es una excelente manera de presentar la oferta a clientes potenciales, ya que se les brinda la oportunidad de «probar antes de comprar», lo que aumenta la conversión y la generación de clientes potenciales altamente cualificados. Para obtener más información, comience con [Qué es la versión de prueba](../what-is-test-drive.md).

Para habilitar una versión de prueba durante un período de tiempo determinado, seleccione la casilla **Habilitar una versión de prueba**. Para eliminar la versión de prueba de la oferta, desactive esta casilla.

### <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

Para obtener más información, consulte [Introducción a la administración de clientes potenciales](./commercial-marketplace-get-customer-leads.md).

Seleccione **Guardar borrador** antes de continuar.

## <a name="properties"></a>Propiedades

Esta página le permite definir las categorías y los sectores que se usan para agrupar su oferta en el marketplace, la versión de la aplicación y los contratos legales que dan soporte a su oferta.

### <a name="categories"></a>Categorías

Seleccione categorías y subcategorías para colocar la oferta en las áreas de búsqueda adecuadas del marketplace. En la descripción de la oferta, asegúrese de indicar la forma en que la oferta admite estas categorías. Seleccione:

- Al menos una y un máximo de dos categorías, incluidas una categoría principal y una secundaria (opcional).
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no hay ninguna subcategoría aplicable a la oferta, seleccione **No aplicable**.

Vea la lista completa de categorías y subcategorías en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).

### <a name="industries"></a>Industrias

[!INCLUDE [Industry Taxonomy](includes/industry-taxonomy.md)]

### <a name="applicable-dynamics-365-products"></a>Productos de Dynamics 365 a los que se aplica

Seleccione todos los productos de Dynamics 365 a los que se aplica esta oferta.

### <a name="app-version"></a>Versión de la aplicación

Escriba el número de versión de la oferta. Los clientes verán esta versión en la página de detalles de la oferta.<!-- If you are only updating the version number due to marketing/descriptive changes, check the **Marketing only change** box. This option allows the offer to bypass the certification and provisioning stages.-->

### <a name="terms-and-conditions"></a>Términos y condiciones

Proporcione sus propios términos y condiciones legales aquí. También puede proporcionar la dirección donde se pueden encontrar. Los clientes deberán aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de continuar.

## <a name="offer-listing"></a>Descripción de la oferta

<!--This page displays the languages in which your offer will be listed. Currently, **English (United States)** is the only available option.

Define marketplace details for each language/market here, such as offer name, description, and images. Select the language/market name to provide this information.-->This page lets you define offer details such as offer name, description, links, and contacts.

> [!NOTE]
> Proporcione los detalles de la descripción de la oferta solo en un idioma. No es necesario que los detalles de la oferta estén en inglés si la descripción de la oferta comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable especificar una *dirección URL de vínculo útil* para ofrecer contenido en un idioma distinto del que se usa en el contenido de la descripción de la oferta.

A continuación se muestra un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):
<!-- update screen? -->
:::image type="content" source="media/example-azure-marketplace-d365-customer-engagement.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
1. Productos
1. Categorías
1. Dirección de soporte técnico (vínculo)
1. Términos de uso
1. Nombre de la oferta
1. Descripción
1. Capturas de pantallas o vídeos

### <a name="marketplace-details"></a>Detalles del marketplace

El **nombre** que escriba aquí se mostrará a los clientes como el título de la descripción de la oferta. Este campo se ha rellenado previamente con el texto introducido para **Alias de la oferta** cuando creó la oferta, pero puede cambiar este valor. Este nombre puede contener marcas comerciales (y puede incluir símbolos de marca comercial o copyright). El nombre no puede tener más de 50 caracteres ni incluir emojis.

Proporcione una descripción breve de la oferta (100 caracteres como máximo), en el **resumen de resultados de la búsqueda**. Esta descripción se puede usar en los resultados de la búsqueda de Marketplace.

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Rich text editor](./includes/rich-text-editor.md)]

Opcionalmente, puede escribir hasta tres **palabras clave de búsqueda** para ayudar a los clientes a buscar su oferta en el marketplace. Para obtener mejores resultados, intente usar también estas palabras clave en la descripción.

Si quiere permitir que los clientes sepan que su **aplicación funciona con productos específicos**, escriba hasta tres nombres de productos aquí.

### <a name="helpprivacy-urls"></a>Direcciones URL de ayuda y privacidad

Escriba el **vínculo de ayuda de la aplicación** (URL), donde los clientes pueden obtener más información sobre su oferta. La dirección URL de ayuda no puede ser la misma que la dirección URL de soporte técnico.

Escriba el **vínculo a la directiva de privacidad** (URL) de su organización. Usted es el responsable no solo de garantizar que la aplicación cumple la normativa y la legislación en lo tocante a la privacidad, sino también de proporcionar una directiva de privacidad válida.

### <a name="contact-information"></a>Información de contacto

Indique el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP.

En la sección **Contacto de soporte técnico**, proporcione la **dirección URL de soporte técnico** en la que los asociados de CSP pueden encontrar soporte técnico para la oferta. La dirección URL de soporte técnico no puede ser la misma que la dirección URL de ayuda.

### <a name="supporting-documents"></a>Documentos relacionados

Proporcione aquí al menos un documento de marketing relacionado (y un máximo de tres), como notas del producto, folletos, listas de comprobación o presentaciones, en formato PDF.

### <a name="marketplace-media"></a>Elementos multimedia del marketplace

Proporcione logotipos e imágenes que se usarán al mostrar la oferta a los clientes. Todas las imágenes deben estar en formato PNG.

[!INCLUDE [logo tips](../includes/graphics-suggestions.md)]

>[!NOTE]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="logos"></a>Logotipos

Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear otros tamaños necesarios. Opcionalmente, puede reemplazarlo por una imagen diferente.

Estos logotipos se usan en distintos lugares de la publicación:

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

[!INCLUDE [Logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Capturas de pantalla

Agregue capturas de pantallas que muestren el funcionamiento de la oferta. Se requiere al menos una captura de pantalla y puede agregar un máximo de cinco. Todas las capturas de pantallas deben tener un tamaño de 1280 x 720 píxeles y estar en formato PNG.

#### <a name="videos"></a>Vídeos

Opcionalmente, puede agregar hasta cuatro vídeos que muestren la oferta. Los vídeos deben estar hospedados en un sitio externo. Para cada uno de ellos, escriba el nombre del vídeo, su dirección y una imagen en miniatura del vídeo (1280 x 720 píxeles).

Para ver más recursos sobre las listas de marketplace, consulte [Procedimientos recomendados para las listas de ofertas de marketplace](../gtm-offer-listing-best-practices.md).

Seleccione **Guardar borrador** antes de continuar.

## <a name="availability"></a>Disponibilidad

Esta página le permite definir dónde y cómo hacer que la oferta esté disponible.

### <a name="markets"></a>Mercados

Para especificar los mercados en los que la oferta debe estar disponible, seleccione **Editar mercados** para mostrar la ventana emergente **Selección de mercado**.

Seleccione al menos un mercado. Elija **Seleccionar todo** para que su oferta esté disponible en cada mercado posible, o seleccione los mercados específicos que quiera agregar. Cuando haya terminado, seleccione **Guardar**.

Las selecciones aquí solo se aplican a las nuevas adquisiciones. Si algún usuario ya tiene su aplicación en un determinado mercado y posteriormente lo quita, las personas que ya tienen la oferta en dicho mercado podrán seguir utilizándola, pero ningún cliente nuevo en ese mercado podrá obtener su oferta.

> [!IMPORTANT]
> Es su responsabilidad cumplir los requisitos legales locales, incluso si esos requisitos no aparecen aquí ni en el Centro de partners. Incluso si selecciona todos los mercados, las leyes y restricciones locales u otros factores pueden impedir que se muestren determinadas ofertas en algunos países y regiones.

### <a name="preview-audience"></a>Público preliminar

Antes de publicar su oferta en la oferta de Marketplace más amplia, primero deberá ponerla a disposición de un **público preliminar** limitado. Escriba una **clave de ocultación** (cualquier cadena que use solo letras minúsculas o números) aquí. Los miembros del público preliminar pueden usar esta clave de ocultación como token para ver una versión preliminar de la oferta en Marketplace.

Después, cuando esté listo para que la oferta esté disponible y quite la restricción de versión preliminar, deberá quitar la **tecla de ocultación** y volver a publicarla.

Seleccione **Guardar borrador** antes de continuar.

## <a name="technical-configuration"></a>Configuración técnica

Esta página especifica los detalles técnicos usados para conectarse a la oferta. Esta conexión nos permite aprovisionar su oferta para el cliente final, en caso de que este elija adquirirla.

### <a name="offer-information"></a>Información de la oferta

El **modelo de licencia básica** determina cómo se asignan los clientes a la aplicación en el centro de administración de CRM. Seleccione **recurso** para las licencias que se basan en instancias o **Usuario** si se asigna una licencia por cada inquilino.

La casilla **Requiere salida de sitio a sitio y acceso al almacén seguro de CRM** permite configurar el acceso de salida del almacenamiento seguro de CRM o de servidor a servidor (S2S). Esta característica requiere una atención especial por parte del equipo de Dynamics 365 durante la fase de certificación. Microsoft se pondrá en contacto con usted para completar los pasos adicionales para admitir esta característica.

Deje **Dirección URL de configuración de aplicación** en blanco; se usará en un futuro.

### <a name="crm-package"></a>Paquete de CRM

En el campo **Dirección URL de la ubicación del paquete**, escriba la dirección URL de una cuenta de Azure Blob Storage que contenga el archivo .zip del paquete de CRM cargado. Incluya una clave SAS de solo lectura para permitir que Microsoft pueda recoger el paquete para su comprobación.

> [!IMPORTANT]
> Para evitar un bloque de publicación, asegúrese de que la fecha de expiración de la dirección URL del almacenamiento de blobs no haya expirado. Se puede revisar la fecha accediendo a la directiva. Se recomienda que el **Tiempo de expiración** sea al menos dentro de un mes.

Seleccione el cuadro **There is more than one CRM package in my package file** (Hay más de un paquete de CRM en el archivo de paquete), si procede. Si es así, asegúrese de incluir todos los paquetes en el archivo ZIP.

Para obtener información detallada sobre cómo compilar el paquete y actualizar su estructura, consulte [Paso 3: Crear un paquete AppSource para la aplicación](/powerapps/developer/common-data-service/create-package-app-appsource).

### <a name="crm-package-availability"></a>Disponibilidad del paquete de CRM

Seleccione **+ Agregar región** para especificar las regiones geográficas en las que el paquete de CRM estará disponible para los clientes. La implementación en las siguientes regiones soberanas requiere un permiso y una validación especiales durante el proceso de certificación: [Alemania](../../germany/index.yml), la [nube del US Gov](../../azure-government/documentation-government-welcome.md) y TIP.

De manera predeterminada, la **dirección URL de configuración de aplicación** que especificó anteriormente se usará para cada región. Si lo prefiere, puede especificar una dirección URL de configuración de aplicación independiente para una o varias regiones específicas.

Seleccione **Guardar borrador** antes de continuar.

<!-- ## Test drive technical configuration

This page lets you set up a demonstration ("test drive") that allows customers to try your offer before purchasing it. Learn more in [What is test drive](../what-is-test-drive.md).

To enable a test drive, select the **Enable a test drive** check box on the [Offer setup](#test-drive) tab. To remove test drive from your offer, clear this check box.

When you've finished setting up your test drive, select **Save draft** before continuing. -->

## <a name="supplemental-content"></a>Contenido complementario

Esta página le permite proporcionar información adicional que nos ayuda a validar la oferta. Esta información no se muestra a los clientes ni se publica en Marketplace.

### <a name="key-usage-scenario"></a>Escenario de uso clave

Cargue un archivo PDF que indique los escenarios de uso claves de la oferta. Es posible que nuestro equipo de validación compruebe todos los escenarios que se enumeran aquí antes de aprobar la oferta para Marketplace.

Seleccione **Guardar borrador** antes de continuar.

## <a name="publish"></a>Publicar

### <a name="submit-offer-to-preview"></a>Envío de la oferta para tener una versión preliminar

Cuando haya completado todas las secciones necesarias de la oferta, seleccione **Revisar y publicar** en la esquina superior derecha del portal.

Si es la primera vez que publica esta oferta, puede:

- Ver el estado de finalización de cada sección de la oferta.
    - **No iniciada**: la sección no se ha modificado y se debe completar.
    - **Incompleta**: la sección tiene errores que deben corregirse o se necesita más información. Vuelva a las secciones y actualícelas.
    - **Completa**: la sección está completa, se han proporcionado todos los datos necesarios y no hay ningún error. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- En la sección **Notas para la certificación**, envíe instrucciones para la realización de pruebas al equipo de certificación para asegurarse de que la aplicación se prueba correctamente, además de todas las notas adicionales que pueden resultar útiles para conocer la aplicación.
- Enviar la oferta para su publicación, para lo que debe seleccionar **Enviar**. Le enviaremos un correo electrónico cuando haya alguna versión preliminar de la oferta disponible para su revisión y aprobación. Para publicar la oferta para el público, vuelva al Centro de partners y seleccione **Publicar**.

## <a name="next-steps"></a>Pasos siguientes

- [Actualización de una oferta existente en Marketplace comercial](./update-existing-offer.md)
