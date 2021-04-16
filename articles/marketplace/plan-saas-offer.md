---
title: Planeamiento de una oferta de SaaS en el marketplace comercial de Microsoft
description: Cómo planear una nueva oferta de software como servicio (SaaS) para publicarla o venderla en Microsoft AppSource, Azure Marketplace o mediante el programa del Proveedor de soluciones en la nube (CSP) por medio del programa del marketplace comercial del Centro de partners de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 03/26/2021
ms.openlocfilehash: 6f08fa0b2126112fa17fd61be6f44bb5cc6d5396
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106552162"
---
# <a name="how-to-plan-a-saas-offer-for-the-commercial-marketplace"></a>Planeamiento de una oferta de SaaS en el marketplace comercial

En este artículo se explican los distintos requisitos y opciones para publicar una oferta de software como servicio (SaaS) en el marketplace comercial de Microsoft. Las ofertas de SaaS permiten ofrecer soluciones de software, y conceder licencias para ellas, a los clientes mediante suscripciones en línea. Como publicador de SaaS, administrará y pagará por la infraestructura necesaria para respaldar el uso que los clientes hacen de la oferta. Este artículo le ayudará a preparar la oferta para publicarla en el marketplace comercial con el Centro de partners.

## <a name="listing-options"></a>Opciones de publicación

Cuando prepare la publicación de una nueva oferta de SaaS, debe decidir la opción de _publicación_. La opción de lista que elija determinará la información adicional que deberá proporcionar a medida que cree la oferta en el Centro de partners. La opción de publicación se define en la página **Configuración de la oferta**, según se explica en [Creación de una oferta de SaaS en el marketplace comercial](create-new-saas-offer.md).

En la tabla siguiente se muestran las opciones de publicación de las ofertas de SaaS en el marketplace comercial.

| Opción de publicación | Proceso de transacción |
| ------------ | ------------- |
| Ponerse en contacto conmigo | El cliente se pone en contacto directamente a partir de la información de la descripción.``*`` |
| Evaluación gratuita | Al cliente se le redirige a la dirección URL de destino mediante Azure Active Directory (Azure AD).``*`` |
| Obténgalo ahora (de forma gratuita) | Al cliente se le redirige a la dirección URL de destino a través de Azure AD.``*`` |
| Venta mediante Microsoft  | Las ofertas vendidas a través de Microsoft se denominan ofertas _procesables_. Una oferta procesable es aquella en la que Microsoft facilita el intercambio de dinero por una licencia de software en nombre del editor. Microsoft factura las ofertas de SaaS con el modelo de precios elegido y administra las transacciones de los clientes en su nombre. Las cuotas de uso de la infraestructura de Azure se le facturan al asociado directamente. Al elegir el modelo de precio, es necesario considerar los costos de la infraestructura. Este aspecto se explica con más detalle a continuación en [Facturación de SaaS](#saas-billing).  |
|||

``*``Los editores son responsables de todos los aspectos de la transacción de las licencias de software, incluidas, entre otras cosas, el pedido, la entrega, la medición, la facturación, el pago y el cobro.

Para más información sobre estas opciones de publicación, consulte [Capacidades de transacción de marketplace comercial](marketplace-commercial-transaction-capabilities-and-considerations.md).

Una vez publicada la oferta, la opción de publicación elegida para ella aparece en forma de un botón en la esquina superior izquierda de la página de descripción de la oferta. Por ejemplo, en la siguiente captura de pantalla se muestra una página de descripción de la oferta en Azure Marketplace con el botón **Obtenerla ahora**.

![Se muestra la descripción de una oferta en la tienda en línea.](./media/saas/listing-options-saas.png)

## <a name="technical-requirements"></a>Requisitos técnicos

Los requisitos técnicos varían en función de la opción de publicación que elija para su oferta.

La opción _Ponerse en contacto conmigo_ no tiene requisitos técnicos. Tiene la opción de conectar un sistema de administración de relaciones con clientes (CRM) para administrar clientes potenciales. Esto se describe en la sección [Clientes potenciales](#customer-leads) que encontrará más adelante en este artículo.

Las opciones de publicación _Obtener ahora (gratis)_ , _Evaluación gratuita_ y _Venta mediante Microsoft_ presentan los siguientes requisitos técnicos:

- La aplicación de SaaS debe ser una solución para varios inquilinos.
- Puede permitir la autenticación de los usuarios tanto con cuentas de Microsoft (MSA) como de [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/).
- Debe crear una página de aterrizaje. Después de que un usuario compre su oferta, se le dirigirá a la página de aterrizaje. Esto les ayuda a completar cualquier aprovisionamiento o configuración adicional que sea necesaria. Puede encontrar instrucciones sobre la creación de la página de aterrizaje en estos artículos:
  - [Creación de la página de aterrizaje de su oferta de SaaS comercializable en el marketplace comercial](azure-ad-transactable-saas-landing-page.md)
  - [Creación de la página de aterrizaje de su oferta de SaaS gratuita o de evaluación en el marketplace comercial](azure-ad-free-or-trial-landing-page.md)

Estos requisitos técnicos adicionales se aplican solo a la opción de publicación _Venta mediante Microsoft_ (procesable):

- Se requiere Azure AD con la autenticación y la administración de identidades de inicio de sesión único (SSO) para el usuario comprador que accede a la página de aterrizaje. Para más información, consulte [Azure AD y ofertas de SaaS comercializables en el marketplace comercial](azure-ad-saas.md).
- Debe usar las [API de cumplimiento de SaaS](./partner-center-portal/pc-saas-fulfillment-api-v2.md) para la integración con Azure Marketplace y Microsoft AppSource. Debe exponer un servicio que pueda interactuar con la suscripción SaaS para crear, actualizar y eliminar una cuenta de usuario y un plan de servicio. Los cambios importantes en la API deben admitirse dentro de un plazo de 24 horas. Los cambios no importantes en la API se publicarán de forma periódica. En la documentación de las [API](./partner-center-portal/pc-saas-fulfillment-api-v2.md) hay diagramas y explicaciones detalladas que describen el uso de los campos recopilados.
- Debe crear al menos un plan para la oferta. El precio del plan se basa en el modelo de precios seleccionado antes de la publicación: _tarifa plana_ o _por usuario_. Se proporcionan más detalles sobre los [planes](#plans) más adelante en este artículo.
- El cliente puede cancelar su oferta en cualquier momento.

### <a name="technical-information"></a>Información técnica

Si va a crear una oferta procesable, deberá recopilar la siguiente información para la página **Configuración técnica**. Si en lugar de crear una oferta procesable decide procesar las transacciones de forma independiente, omita esta sección y vaya a [Versiones de prueba](#test-drives).

- **Dirección URL de la página de aterrizaje**: la dirección URL del sitio de SaaS (por ejemplo, `https://contoso.com/signup`) a la que se dirigirá a los usuarios después de adquirir la oferta en el marketplace comercial, que desencadena el proceso de configuración de la suscripción de SaaS recién creada. Esta dirección URL recibirá un token que se puede usar para llamar a las API de entrega y obtener los detalles de aprovisionamiento de la página de registro interactivo.

  Se llamará a esta dirección URL con el parámetro del token de identificación de compra de marketplace, que identifica de forma única la compra de SaaS del cliente específico. Este token se debe intercambiar por los detalles de la suscripción de SaaS correspondiente mediante la [API de resolución](./partner-center-portal/pc-saas-fulfillment-api-v2.md#resolve-a-purchased-subscription). Estos detalles y cualquier otro que quiera recopilar como parte de una página web interactiva del cliente se pueden usar para iniciar la experiencia de incorporación del cliente, que debe concluir en un momento dado con una llamada de activación en la API para iniciar el período de suscripción. En esta página, el usuario debe registrarse mediante la autenticación con un solo clic con Azure Active Directory (Azure AD).

  Esta dirección URL también se llamará con el parámetro del token de identificación de compra de marketplace cuando el cliente inicie la experiencia de SaaS administrada desde Azure Portal o el Centro de administración de Microsoft 365. Ambos flujos deben administrarse: la primera vez que se proporciona el token después de la compra de un nuevo cliente y cuando se proporciona de nuevo para un cliente existente que administra su solución de SaaS.

    La página de aterrizaje que configure debe estar en funcionamiento de forma ininterrumpida. Esta es la única vía por la que se le notificarán nuevas compras de las ofertas de SaaS realizadas en el marketplace comercial o las solicitudes de configuración de una suscripción activa a una oferta.

- **Webhook de conexión**: para todos los eventos asincrónicos que Microsoft tiene que enviarle (por ejemplo, si la suscripción a SaaS se ha cancelado), le pedimos que nos proporcione una URL de webhook de conexión. Se llamará a esta dirección URL para notificarle sobre el evento.

  El webhook que proporcione debe estar en funcionamiento de forma ininterrumpida. Tenga en cuenta que esta es la única vía por la que se le notificarán las actualizaciones de las suscripciones de SaaS de sus clientes adquiridas a través de Marketplace comercial.

  > [!NOTE]
  > Dentro de Azure Portal, es necesario que cree un [registro de aplicación de Azure Active Directory (Azure AD)](../active-directory/develop/howto-create-service-principal-portal.md) de un único inquilino. Use los detalles del registro de aplicación para autenticar la solución cuando llame a las API del marketplace. Para encontrar el [identificador de inquilino](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in), vaya a su instancia de Azure Active Directory, seleccione **Propiedades** y busque el número del identificador de directorio que se muestra. Por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`.

- **Id. de inquilino de Azure Active Directory**: (también conocido como identificador de directorio). Dentro de Azure Portal, es necesario [registrar una aplicación de Azure Active Directory (AD)](../active-directory/develop/howto-create-service-principal-portal.md) para poder agregarla a la lista de control de acceso (ACL) de la API a fin de garantizar que está autorizado para llamarla. Para encontrar el identificador de inquilino de la aplicación de Azure Active Directory (AD), vaya a la hoja [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) de Azure Active Directory. En la columna **Nombre para mostrar**, seleccione la aplicación. Luego, busque el número de **Id. de directorio (inquilino)** que se muestra (por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`).

- **Id. de aplicación de Azure Active Directory**: también necesitará el [identificador de la aplicación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in). Para obtener su valor, vaya a la hoja [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) de Azure Active Directory. En la columna **Nombre para mostrar**, seleccione la aplicación. Luego, busque el número de identificador de la aplicación (cliente) que se muestra (por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`).

  El identificador de la aplicación de Azure AD está asociado a su identificador de editor de la cuenta del Centro de partners. El mismo identificador de aplicación debe usarse con todas las ofertas de esa cuenta.

  > [!NOTE]
  > Si el publicador tiene dos o más cuentas diferentes en el Centro de partners, los detalles del registro de aplicación de Azure AD solo se pueden usar en una cuenta. No se admite el uso del mismo par de id. de inquilino e id. de la aplicación para una oferta en una cuenta de publicador diferente.

## <a name="test-drives"></a>Versiones de prueba
Se puede elegir habilitar una versión de prueba de la aplicación SaaS. Las versiones de prueba proporcionan a los clientes acceso a un entorno preconfigurado durante un número fijo de horas. Aunque puede habilitar las opciones de publicación en cualquier versión de prueba, esta característica conlleva requisitos adicionales. Para más información sobre las versiones de prueba, consulte [¿Qué es una versión de prueba?](what-is-test-drive.md). Para información sobre la configuración de diferentes tipos de versiones de prueba, consulte [Configuración técnica de la versión de prueba](test-drive-technical-configuration.md).

> [!TIP]
> Una versión de prueba es diferente de una [evaluación gratuita](plans-pricing.md#free-trials). Puede ofrecer una versión de prueba, una evaluación gratuita o ambas. Las dos proporcionan a los clientes la solución durante un período fijo. Sin embargo, una versión de prueba también incluye un práctico recorrido autoguiado por las principales características y ventajas del producto demostradas en un escenario de implementación real.

## <a name="customer-leads"></a>Clientes potenciales

Debe conectar su oferta al sistema de administración de relaciones con clientes (CRM) para recopilar información del cliente. El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre y el identificador de la oferta, además de la tienda en línea donde la encontró, se envían al sistema CRM que haya configurado. El marketplace comercial admite una gran variedad de sistemas CRM, junto con la opción de usar una tabla de Azure o de configurar un punto de conexión HTTPS con Power Automate.

Puede agregar o modificar una conexión CRM en cualquier momento durante o después de la creación de la oferta. Para obtener instrucciones detalladas, vea [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

## <a name="selecting-an-online-store"></a>Selección de una tienda en línea

Al publicar una oferta de SaaS, se mostrará en Microsoft AppSource, Azure Marketplace o ambos. Cada tienda en línea atiende los requisitos únicos de cada cliente. AppSource es para soluciones empresariales y Azure Marketplace para soluciones de TI. El tipo de oferta, las funcionalidades de transacción y la categoría determinarán dónde se publicará la oferta. Las categorías y subcategorías se asignan a cada tienda en línea en función del tipo de solución. 

Si la oferta de SaaS se encuentra en *ambas* categorías; es decir, es una solución de TI (Azure Marketplace) y una solución empresarial (AppSource), seleccione una categoría y una subcategoría aplicable a cada tienda en línea. Las ofertas publicadas en ambas tiendas en línea deben tener una propuesta de valor como solución de TI *y* como solución empresarial.

> [!IMPORTANT]
> Las ofertas de SaaS con [facturación medida](partner-center-portal/saas-metered-billing.md) están disponibles a través de Azure Marketplace y Azure Portal. Las ofertas de SaaS con planes privados exclusivamente están disponibles en Azure Portal.

| Facturación de uso medido | Plan público | Plan privado | Disponible en: |
|---|---|---|---|
| Sí             | Sí         | No           | Azure Marketplace y Azure Portal |
| Sí             | Sí         | Sí          | Azure Marketplace y Azure Portal* |
| Sí             | No          | Sí          | Azure Portal solamente |
| No              | No          | Sí          | Azure Portal solamente |
|||||

&#42; El plan privado de la oferta solo estará disponible en Azure Portal.

Por ejemplo, una oferta con facturación de uso medido y un plan privado solo (ningún plan público) lo comprarán los clientes en Azure Portal. Obtenga más información sobre [Ofertas privadas en el marketplace comercial de Microsoft](private-offers.md).

Para obtener información detallada acerca de cómo enumerar las opciones que admiten las tiendas en línea, consulte [Opciones de lista y precios de las tiendas en línea](determine-your-listing-type.md#listing-and-pricing-options-by-online-store). Para obtener más información sobre categorías y subcategorías, consulte [Categorías y subcategorías en Marketplace comercial](categories.md).

## <a name="legal-contracts"></a>Contratos legales

Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para los proveedores de software, Microsoft ofrece un contrato estándar que se puede usar con las ofertas del marketplace comercial. Al ofrecer el software bajo el contrato estándar, los clientes solo tienen que leerlo y aceptarlo, sin necesidad de crear términos y condiciones personalizados.

Si decide usar el contrato estándar, tiene la opción de agregarle términos de modificación universales y hasta 10 modificaciones personalizadas. También puede usar sus propios términos y condiciones en lugar del contrato estándar. Estos detalles se administrarán en la página **Propiedades**. Para más información, consulte [Contrato estándar para el marketplace comercial de Microsoft](standard-contract.md).

> [!NOTE]
> Después de publicar una oferta con el contrato estándar en el marketplace comercial, no puede usar sus propios términos y condiciones personalizados. Solo puede elegir una de las dos opciones. O bien ofrecer su solución bajo el contrato estándar o según sus propios términos y condiciones. Si quiere modificar los términos del contrato estándar, puede hacerlo a través de las modificaciones al contrato estándar.


## <a name="microsoft-365-integration"></a>Integración de Microsoft 365

La integración con Microsoft 365 permite que la oferta de SaaS proporcione experiencia conectada entre varias superficies de aplicación de Microsoft 365 a través de complementos gratuitos relacionados, como aplicaciones de Teams, complementos de Office y soluciones de SharePoint Framework. Puede ayudar a los clientes a detectar fácilmente todas las facetas de la solución E2E (servicio web + complementos relacionados) e implementarlas en un proceso proporcionando la siguiente información. 
  - Si la oferta de SaaS se integra con Microsoft Graph, proporcione el identificador de aplicación de Azure Active Directory (AAD) que usa la oferta de SaaS para la integración. Los administradores pueden revisar los permisos de acceso necesarios para el correcto funcionamiento de la oferta de SaaS como se establece en el identificador de aplicación de AAD y conceder acceso si se necesita el permiso de administrador avanzado en el momento de la implementación. 
    
     Si decide vender su oferta a través de Microsoft, es el mismo identificador de aplicación de AAD que ha registrado para usar en la página de aterrizaje para obtener la información básica de usuario necesaria para completar la activación de la suscripción de cliente. Para obtener instrucciones detalladas, consulte [Creación de la página de aterrizaje de su oferta de SaaS comercializable en el marketplace comercial](azure-ad-transactable-saas-landing-page.md). 
    
   -    Proporcione una lista de complementos relacionados que funcionen con la oferta de SaaS que desee vincular. Los clientes podrán detectar su solución de E2E en AppSource y los administradores pueden implementar tanto SaaS como todos los complementos relacionados que haya vinculado en el mismo proceso a través del centro de administración de Microsoft 365.
    
        Para vincular complementos relacionados, debe proporcionar el vínculo de AppSource del complemento, lo que significa que el complemento se debe publicar primero en AppSource. Los tipos de complementos admitidos que se pueden vincular son: aplicaciones de Teams, complementos de Office y soluciones de SharePoint Framework (SPFx). Cada complemento vinculado debe ser único para una oferta de SaaS. 

En el caso de los productos vinculados, la búsqueda en AppSource devolverá un resultado que incluye tanto SaaS como todos los complementos vinculados. El cliente puede desplazarse entre las páginas de detalles de producto de la oferta de SaaS y los complementos vinculados. Los administradores de TI pueden revisar e implementar los complementos de SaaS y vinculados dentro del mismo proceso a través de una experiencia integrada y conectada en el centro de administración de Microsoft 365. Para más información, consulte [Prueba e implementación de aplicaciones de Microsoft 365: administración de Microsoft 365](/microsoft-365/admin/manage/test-and-deploy-microsoft-365-apps).

### <a name="microsoft-365-integration-support-limitations"></a>Limitaciones de compatibilidad con la integración de Microsoft 365
La detección como una única solución de E2E es compatible con AppSource para todos los casos. Sin embargo, la implementación simplificada de la solución E2E como se describió anteriormente a través del centro de administración de Microsoft 365 no se admite en los siguientes escenarios:

   - El mismo complemento está vinculado a más de una oferta de SaaS.
   - La oferta de SaaS está vinculada a complementos, pero no se integra con Microsoft Graph y no se proporciona ningún identificador de aplicación de AAD.
  - La oferta de SaaS está vinculada a complementos, pero el identificador de aplicación de AAD proporcionado para la integración de Microsoft Graph se ha compartido en varias ofertas de SaaS.

 
## <a name="offer-listing-details"></a>Detalles de la descripción de la oferta

Al [crear una oferta de SaaS](create-new-saas-offer.md) en el Centro de partners, insertará texto, imágenes, vídeos opcionales y otros detalles en la página **Descripción de la oferta**. Esta es la información que verán los clientes cuando descubran la descripción de su oferta en el marketplace comercial, como se muestra en el ejemplo siguiente.

:::image type="content" source="./media/saas/example-saas-1.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

**Descripciones destacadas**

1. Logotipo
2. Categorías
3. Industrias
4. Dirección de soporte técnico (vínculo)
5. Términos de uso
6. Directiva de privacidad
7. Nombre de la oferta
8. Resumen
9. Descripción
10. Capturas de pantallas o vídeos
11. Documentos

En el ejemplo siguiente se muestra la descripción de una oferta en Azure Portal.

![Se muestra la descripción de una oferta en Azure Portal.](./media/example-managed-service-azure-portal.png)

**Descripciones de llamadas**

1. Título
1. Descripción
1. Vínculos útiles
1. Capturas de pantalla

> [!NOTE]
> No es necesario que el contenido de la descripción de la oferta esté en inglés si comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]".

Para facilitar la creación de la oferta, prepare algunos de estos elementos con antelación. Los elementos siguientes son obligatorios a menos que se indique lo contrario.

- **Name**: este nombre aparecerá como título de la descripción de la oferta en el marketplace comercial. El nombre puede ser una marca comercial. No puede contener emojis (a menos que sean símbolos de marca comercial y copyright), y no puede tener más de 50 caracteres.
- **Resumen de los resultados de la búsqueda**: describa el propósito o la función de la oferta en una sola frase de 100 caracteres como máximo y sin saltos de línea. Este resumen se usa en los resultados de búsqueda de las ofertas del marketplace comercial.
- **Descripción**: esta descripción se mostrará en la información general de las ofertas del marketplace comercial. Considere la posibilidad de incluir una propuesta de valor, los beneficios principales, los compradores previstos, las asociaciones de categoría o sector, las oportunidades de compra en la aplicación, todas las divulgaciones necesarias y un vínculo para obtener más información.

    Este cuadro de texto tiene controles de editor de texto enriquecido que puede utilizar para que su descripción sea más atractiva. También puede usar etiquetas HTML para dar formato a la descripción. En este cuadro puede escribir hasta 3000 caracteres de texto, incluido el marcado HTML. Consulte [Escribir una excelente descripción de la aplicación](/windows/uwp/publish/write-a-great-app-description) para encontrar más sugerencias.

- **Instrucciones de inicio** Si opta por vender su oferta a través de Microsoft (oferta procesable), este campo es obligatorio. Estas instrucciones ayudarán a los clientes a conectarse a su oferta de SaaS. Puede agregar hasta 3000 caracteres de texto y vínculos a documentación en línea más detallada.
- **Palabras clave de búsqueda** (opcional): escriba al menos tres palabras clave de búsqueda que los clientes puedan usar para buscar su oferta en las tiendas en línea. No es necesario incluir el **nombre** y la **descripción** de la oferta: ese texto se incluye automáticamente en la búsqueda.
- **Vínculo a la directiva de privacidad**: la dirección URL que lleva a la directiva de privacidad de su empresa. Debe proporcionar una directiva de privacidad válida y usted es responsable de garantizar que la aplicación cumple con las leyes y normativas de privacidad.
- **Información de contacto**: Debe proporcionar los siguientes contactos de la organización:
  - **Contacto de soporte técnico**: proporcione el nombre, el teléfono y el correo electrónico de los asociados de Microsoft que se usarán cuando los clientes abran vales. También debe incluir la dirección URL del sitio web de soporte técnico.
  - **Contacto de ingeniería**: proporcione el nombre, el teléfono y el correo electrónico que Microsoft usará directamente cuando haya problemas con la oferta. Esta información de contacto no aparece en el marketplace comercial.
  - **Contacto del programa CSP** (opcional): proporcione el nombre, el teléfono y el correo electrónico si participa en el programa CSP, de modo que dichos asociados puedan ponerse en contacto con usted con cualquier pregunta. También puede incluir una dirección URL a sus materiales de marketing.
- **Vínculos útiles** (opcional): puede proporcionar vínculos a varios recursos para los usuarios de su oferta. Por ejemplo, foros, preguntas más frecuentes y notas de la versión.
- **Documentos relacionados**: puede proporcionar hasta tres documentos orientados al cliente, como notas del producto, folletos, listas de comprobación o presentaciones de PowerPoint.
- **Elementos multimedia (logotipos)** : proporcione un archivo PNG para el logotipo **grande**. El Centro de partners lo usará para crear un logotipo **Pequeño** y un logotipo **Mediano**. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.

   - Grande (de 216 x 216 a 350 x 350 píxeles, obligatorio)
   - Mediano (90 x 90 píxeles, opcional)
   - Pequeño (48 x 48 píxeles, opcional)

  Estos logotipos se usan en distintos lugares de las tiendas en línea:

  - El logotipo pequeño aparece en los resultados de la búsqueda de Azure Marketplace, y en la página principal y en la página de resultados de búsqueda de Microsoft AppSource.
  - El logotipo mediano aparece cuando se crea un recurso en Microsoft Azure.
  - El logotipo grande aparece en la página de descripción de la oferta de Azure Marketplace y Microsoft AppSource.

- **Elementos multimedia (capturas de pantalla)** : debe agregar entre una y cinco capturas de pantallas que muestren el funcionamiento de la oferta, con los siguientes requisitos:
  - 1280 x 720 píxeles
  - Archivo .png
  - Se debe incluir un título
- **Elementos multimedia (vídeos)** (opcional): puede agregar hasta cuatro vídeos que muestren su oferta, con los siguientes requisitos:
  - Nombre
  - Dirección URL: solo se debe hospedar en YouTube o en Vimeo.
  - Miniatura: archivo .png de 1280 x 720

> [!Note]
> La oferta debe cumplir las [directivas de certificación del marketplace comercial](/legal/marketplace/certification-policies#100-general) generales y las [directivas de software como servicio](/legal/marketplace/certification-policies#1000-software-as-a-service-saas) que se van a publicar en el marketplace comercial.

> [!NOTE]
> Una audiencia preliminar no es lo mismo que un plan privado. Un plan privado es el que solo está disponible para una audiencia determinada de su elección. Esto le permite negociar un plan personalizado con clientes específicos. Para obtener más información, vea la próxima sección: Planes

Puede enviar invitaciones a direcciones de correo electrónico de cuentas de Microsoft (MSA) o de Azure Active Directory (Azure AD). Agregue hasta 10 direcciones de correo electrónico manualmente o importe hasta 20 con un archivo .csv. Aunque la oferta ya esté publicada, puede definir una audiencia preliminar para probar los cambios o las actualizaciones que realice en ella.

## <a name="plans"></a>Planes

Las ofertas procesables requieren al menos un plan. Un plan define el ámbito y los límites de la solución, así como los precios asociados. Puede crear varios planes para que su oferta brinde a sus clientes diferentes opciones técnicas y de precios. Si elige procesar las transacciones de forma independiente en lugar de crear una oferta procesable, la página **Planes** no es visible. En este caso, omita esta sección y vaya a [Oportunidades de venta adicionales](#additional-sales-opportunities).

Consulte [Planes y precios de ofertas de marketplace comercial](plans-pricing.md) para obtener instrucciones generales sobre los planes, incluidos los modelos de precios, las evaluaciones gratuitas y los planes privados. En las secciones siguientes se describe información adicional específica de las ofertas de SaaS.

### <a name="saas-pricing-models"></a>Modelos de precios de SaaS

Las ofertas de SaaS pueden usar dos modelos de precios con cada plan: _tarifa plana_ o _por usuario_. Todos los planes de la misma oferta deben estar asociados al mismo modelo de precios. Por ejemplo, una oferta no puede tener un plan de tarifa plana y un plan por usuario.

**Tarifa plana**: habilite el acceso a la oferta con unA única tarifa plana mensual o anual. A veces esto se denomina precios basados en puesto. Con este modelo de precios, puede definir opcionalmente planes de uso medido que emplean la API del servicio de medición del marketplace para cobrar a los clientes por el uso que no está cubierto por la tarifa plana. Para más información sobre la facturación del uso medido, consulte [Facturación según uso con el servicio de medición de marketplace](./partner-center-portal/saas-metered-billing.md). También debe usar esta opción si el comportamiento del servicio SaaS presenta ráfagas.

**Por usuario**: habilite el acceso a su oferta con un precio basado en el número de usuarios que pueden acceder a la oferta u ocupar los puestos. Con este modelo basado en el usuario, puede establecer el número mínimo y máximo de usuarios admitidos por el plan. Además, puede crear varios planes para configurar diferentes puntos de precio en función del número de usuarios. Estos campos son opcionales. Si no se seleccionan, se interpretará que el número de usuarios no tiene límite (el mínimo es 1 y el máximo es tantos como el sistema pueda admitir). Estos campos se pueden editar como parte de una actualización del plan.

> [!IMPORTANT]
> Una vez publicada la oferta, no se puede cambiar el modelo de precios. Además, todos los planes de la misma oferta deben compartir el mismo modelo de precios.

### <a name="saas-billing"></a>Facturación de SaaS

En el caso de aplicaciones SaaS que se ejecutan en la suscripción de Azure (del editor), el uso de la infraestructura se factura directamente; los clientes no ven las tarifas de uso de la infraestructura real. Debe agrupar las tarifas de uso de la infraestructura de Azure en el precio de la licencia de software para compensar el costo de la infraestructura que implementó para ejecutar la solución.

Las ofertas de aplicaciones SaaS que se venden a través de Microsoft admiten facturación mensual o anual según una cuota fija, por usuario o cargos de consumo por medio del [servicio de facturación medido](./partner-center-portal/saas-metered-billing.md). El marketplace comercial funciona según un modelo de agencia, en el que los editores establecen los precios y Microsoft factura a los clientes y paga los ingresos al editor, a la vez que retiene la cuota correspondiente a la agencia.

En el siguiente ejemplo se muestra un desglose de costos y pagos para demostrar el modelo de agencia. En este ejemplo, Microsoft factura USD 100 al cliente por su licencia de software y paga USD 80 al editor.

| Costo de licencia | 100 USD al mes |
| ------------ | ------------- |
| Costo de uso de Azure (D1/1 núcleo) | Facturado directamente al publicador, no al cliente |
| Microsoft factura al cliente | 100,00 USD al mes (el anunciante debe contar con los costos de paso a través o en que se incurran en el precio de la licencia) |
| **Microsoft factura** | **100 USD al mes** |
| Microsoft le paga el 80 % del costo de licencia<br>`*` En el caso de aplicaciones SaaS que cumplen los requisitos, Microsoft paga el 90 % del costo de la licencia| 80,00 USD al mes<br>``*`` 90,00 USD al mes |
|||

**`*` Tarifa reducida del servicio Marketplace**: en determinadas ofertas de SaaS que haya publicado en nuestro marketplace comercial, Microsoft reducirá su tarifa del servicio Marketplace del 20 % (como se describe en el Acuerdo de publicador de Microsoft) al 10 %. Para que su oferta sea apta, Microsoft debe haber designado sus ofertas como incentivadas para realizar una venta conjunta por IP de Azure. Para recibir el precio reducido del servicio Marketplace, se deben cumplir los criterios de idoneidad al menos cinco (5) días hábiles antes del final de cada mes natural. Una vez que se cumplan los criterios de idoneidad, se concederá el precio reducido del servicio a todas las transacciones vigentes el primer día del mes siguiente y se seguirá aplicando hasta que se pierda el estado Venta conjunta con incentivos de IP de Azure. Para obtener más información sobre la elegibilidad de la venta conjunta de IP, vea [Requisitos para el estado de la venta conjunta](/legal/marketplace/certification-policies#3000-requirements-for-co-sell-status). Las cuotas del servicio Marketplace reducidas también se aplican a VM y aplicaciones administradas con incentivos para realizar una venta conjunta de Azure IP, así como a cualquier otra oferta de IaaS procesable cualificada que esté disponible a través de Marketplace comercial.

## <a name="preview-audience"></a>Público preliminar

Una audiencia de versión preliminar puede acceder a la oferta antes de que se publique en las tiendas en línea. Pueden ver el aspecto que tendrá la oferta en el marketplace comercial y probar la funcionalidad de un extremo a otro antes de publicarla. 

En la página **Audiencia preliminar**, puede definir una audiencia preliminar limitada. Esta opción no está disponible si elige procesar las transacciones de forma independiente en lugar de vender su oferta mediante Microsoft. En este caso, puede omitir esta sección e ir a [Oportunidades de venta adicionales](#additional-sales-opportunities).

## <a name="test-offer"></a>Prueba de la oferta

Antes de publicar la oferta, debe usar la funcionalidad de versión preliminar para desarrollar la implementación técnica, probar y experimentar con diferentes modelos de precios.

Para desarrollar y probar la oferta de SaaS con la menor cantidad de riesgo, se recomienda crear una oferta de prueba y desarrollo (DEV) para experimentación y pruebas. La oferta de desarrollo será independiente de la oferta de producción (PROD).

Para evitar compras accidentales de la oferta de desarrollo, nunca debe presionar el botón **Publicar** para publicar dicha oferta.

![Muestra la página de Información general de la oferta para una oferta en el Centro de partners. Se muestran los vínculos de versión preliminar y el botón de Transmitir en directo. El vínculo Ver informe de validación también aparece en Validación automatizada.](./media/review-publish-offer/publish-status-saas.png)

Estas son algunas de las razones para crear una oferta de desarrollo independiente para que el equipo de desarrollo use para el desarrollo y las pruebas de la oferta de producción:

- Evitar cargos accidentales de clientes
- Evaluación de los modelos de precios
- No agregar planes que no tengan como destino clientes reales

### <a name="avoid-accidental-customer-charges"></a>Evitar cargos accidentales de clientes

Mediante el uso de una oferta de desarrollo en lugar de la oferta de producción y su tratamiento como entornos de desarrollo y producción, puede evitar cargos accidentales a los clientes.

Le recomendamos que registre dos aplicaciones de Azure AD diferentes para llamar a las API del marketplace. Los desarrolladores usarán una aplicación de Azure AD con la configuración de la oferta de desarrollo y el equipo de operaciones usará el registro de aplicación de producción. Al hacer esto, puede aislar el equipo de desarrollo de los errores involuntarios, como llamar a la API para cancelar la suscripción de un cliente que paga 100 000 USD al mes. También puede evitar el cargo de un cliente por uso medido que no consumió.

### <a name="evaluate-pricing-models"></a>Evaluación de los modelos de precios

La prueba de modelos de precios en la oferta de desarrollo reduce el riesgo cuando los desarrolladores experimentan con distintos modelos de precios.

Los publicadores pueden crear los planes que necesitan en la oferta de desarrollo para determinar qué modelo de precios funciona mejor para su oferta. Los desarrolladores pueden querer crear varios planes en la oferta de desarrollo para probar diferentes combinaciones de precios. Por ejemplo, puede crear planes con diferentes conjuntos de dimensiones personalizadas de uso medido. Puede crear un plan diferente con una combinación de una tarifa plana y dimensiones personalizadas de uso medido.

Para probar varias opciones de precios, debe crear un plan para cada modelo de precios único. Para más información, consulte [Planes](#plans).

### <a name="not-adding-plans-that-do-not-target-actual-customers"></a>No agregar planes que no tengan como destino clientes reales

Mediante el uso de una oferta de desarrollo para desarrollo y pruebas, puede reducir el desorden innecesario en la oferta de producción. Por ejemplo, no puede eliminar los planes que cree para probar diferentes modelos de precios o configuraciones técnicas (sin rellenar una incidencia de soporte técnico). Por lo tanto, al crear planes para realizar pruebas en la oferta de desarrollo, se reduce el desorden en la oferta de producción.

El desorden de la oferta de producción frustra a los equipos de productos y marketing, ya que esperan que todos los planes se dirijan a clientes reales. Especialmente en el caso de los equipos de gran tamaño inconexos, que quieren diferentes espacios aislados con los que trabajar, la creación de dos ofertas proporcionará dos entornos diferentes para desarrollo y producción. En algunos casos, puede que desee crear varias ofertas de desarrollo para admitir un equipo más grande que tenga distintas personas que ejecuten diferentes escenarios de prueba. Al permitir que los distintos miembros del equipo trabajen en la oferta de desarrollo independientemente de la oferta de producción, ayuda a mantener los planes de producción lo más cerca posible del entorno de producción.

La prueba de una oferta de desarrollo ayuda a evitar el límite de 30 dimensiones personalizadas de uso medido por oferta. Los desarrolladores pueden probar diferentes combinaciones de medidores en la oferta de desarrollo sin que ello afecte al límite de dimensiones personalizadas de uso medido en la oferta de producción.

## <a name="additional-sales-opportunities"></a>Oportunidades de venta adicionales

Puede optar por participar en los canales de marketing y ventas respaldados por Microsoft. Al crear la oferta en el Centro de partners, verá dos pestañas hacia el final del proceso:

- **Resell through CSPs** (Revender a través de los CSP): use esta opción para permitir que los asociados de los proveedores de soluciones en la nube (CSP) de Microsoft revendan la solución como parte de una oferta agrupada. Para más información sobre este programa, vea el [programa de proveedores de soluciones en la nube](cloud-solution-providers.md).

- **Venta conjunta con Microsoft**: esta opción permite que los equipos de ventas de Microsoft consideren la solución idónea para la venta conjunta de IP al evaluar las necesidades de los clientes. Para obtener más información sobre la elegibilidad de la venta conjunta, vea [Requisitos para el estado de la venta conjunta](/legal/marketplace/certification-policies#3000-requirements-for-co-sell-status). Para información detallada sobre cómo preparar la oferta para la evaluación, vea [Opción de venta conjunta en el Centro de partners](commercial-marketplace-co-sell.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de SaaS en el marketplace comercial](create-new-saas-offer.md)
- [Procedimientos recomendados para la publicación de ofertas](gtm-offer-listing-best-practices.md)
