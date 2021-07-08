---
title: Traslado de la autenticación de las aplicaciones de AD FS a Azure Active Directory
description: Obtenga información acerca de cómo usar Azure Active Directory para reemplazar los Servicios de federación de Active Directory (AD FS), lo que proporciona a los usuarios un inicio de sesión único para todas las aplicaciones.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 03/01/2021
ms.author: mtillman
ms.reviewer: baselden
ms.openlocfilehash: 616550188687823438187e1e09c90d5d3c6c6835
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078822"
---
# <a name="moving-application-authentication-from-active-directory-federation-services-to-azure-active-directory"></a>Traslado de la autenticación de las aplicaciones de los Servicios de federación de Active Directory (AD FS) a Azure Active Directory

[Azure Active Directory (Azure AD)](../fundamentals/active-directory-whatis.md) ofrece una plataforma de identidad universal que proporciona a los usuarios, asociados y clientes una identidad única para acceder a las aplicaciones y colaborar desde cualquier plataforma y dispositivo. Azure AD tiene un [conjunto completo de funcionalidades de administración de identidades](../fundamentals/active-directory-whatis.md). La estandarización de la autenticación y autorización de las aplicaciones en Azure AD ofrece estas ventajas.

> [!TIP]
> Este artículo se ha escrito para desarrolladores. Los jefes de proyecto y los administradores que planean trasladar una aplicación a Azure AD pueden leer [Migración de la autenticación de la aplicación a Azure AD](migrate-application-authentication-to-azure-active-directory.md).

## <a name="azure-ad-benefits"></a>Ventajas de Azure AD

Si tiene un directorio local que contiene cuentas de usuario, es probable que tenga muchas aplicaciones en las que los usuarios se deben autenticar. Cada una de estas aplicaciones está configurada para que los usuarios obtengan acceso mediante sus identidades.

Los usuarios también se pueden autenticar directamente con su instancia de Active Directory local. Los Servicios de federación de Active Directory (AD FS) son un servicio de identidad local basado en estándares. Extienden la capacidad de usar la funcionalidad del inicio de sesión único (SSO) entre socios comerciales de confianza a fin de que los usuarios no deban iniciar sesión por separado en cada aplicación. Esto se conoce como identidad federada.

Muchas organizaciones tienen software como servicio (SaaS) o aplicaciones de línea de negocio (LOB) personalizadas que están federadas directamente a AD FS, junto con aplicaciones basadas en Azure AD y Microsoft 365.

  ![Aplicaciones conectadas directamente en un entorno local](media/migrate-adfs-apps-to-azure/app-integration-before-migration-1.png)

> [!Important]
> Para aumentar la seguridad de las aplicaciones, el objetivo es tener un conjunto único de controles de acceso y directivas en los entornos locales y en la nube.

  ![Aplicaciones conectadas a través de Azure AD](media/migrate-adfs-apps-to-azure/app-integration-after-migration-1.png)

## <a name="types-of-apps-to-migrate"></a>Tipos de aplicaciones que se van a migrar

La migración de toda la autenticación de las aplicaciones a Azure AD es óptima, ya que ofrece un plano de control único para la administración de identidades y acceso.

Las aplicaciones pueden usar protocolos modernos o heredados para la autenticación. Cuando planee la migración a Azure AD, plantéese migrar primero las aplicaciones que usan protocolos de autenticación modernos (como SAML y Open ID Connect). Estas aplicaciones se pueden volver a configurar para autenticarse con Azure AD a través de un conector integrado desde la Galería de aplicaciones o al registrar la aplicación en Azure AD. Las aplicaciones que usan protocolos antiguos se pueden integrar con Application Proxy.

Para más información, consulte:

* [Uso de Azure AD Application Proxy para publicar aplicaciones locales para usuarios remotos](../app-proxy/what-is-application-proxy.md).
* [¿Qué es la administración de aplicaciones?](what-is-application-management.md)
* [Use el informe de actividades de aplicaciones de AD FS para migrar aplicaciones a Azure AD](migrate-adfs-application-activity.md).
* [Supervisión de AD FS mediante Azure AD Connect Health](../hybrid/how-to-connect-health-adfs.md).

### <a name="the-migration-process"></a>El proceso de migración

Durante el proceso de trasladar la autenticación de las aplicaciones a Azure AD, pruebe tanto las aplicaciones como la configuración. Se recomienda seguir usando los entornos de prueba existentes para las pruebas de migración al trasladarse al entorno de producción. Si actualmente no hay disponible un entorno de prueba, puede configurar uno con [Azure App Service](https://azure.microsoft.com/services/app-service/) o [Azure Virtual Machines](https://azure.microsoft.com/free/virtual-machines/search/?OCID=AID2000128_SEM_lHAVAxZC&MarinID=lHAVAxZC_79233574796345_azure%20virtual%20machines_be_c__1267736956991399_kwd-79233582895903%3Aloc-190&lnkd=Bing_Azure_Brand&msclkid=df6ac75ba7b612854c4299397f6ab5b0&ef_id=XmAptQAAAJXRb3S4%3A20200306231230%3As&dclid=CjkKEQiAhojzBRDg5ZfomsvdiaABEiQABCU7XjfdCUtsl-Abe1RAtAT35kOyI5YKzpxRD6eJS2NM97zw_wcB), dependiendo de la arquitectura de la aplicación.

Puede optar por configurar un inquilino de Azure AD de prueba independiente en el que desarrollará las configuraciones de las aplicaciones.

El proceso de migración puede tener el aspecto siguiente:

#### <a name="stage-1--current-state-the-production-app-authenticates-with-ad-fs"></a>Fase 1: Estado actual: la aplicación de producción se autentica con AD FS

  ![Fase de migración 1 ](media/migrate-adfs-apps-to-azure/stage1.jpg)

#### <a name="stage-2--optional-point-a-test-instance-of-the-app-to-the-test-azure-ad-tenant"></a>Fase 2: (opcional) Apunte una instancia de prueba de la aplicación al inquilino de Azure AD de prueba

Actualice la configuración para que apunte la instancia de prueba de la aplicación a un inquilino de Azure AD de prueba y haga los cambios que sean necesarios. La aplicación se puede probar con los usuarios en el inquilino de Azure AD de prueba. Durante el proceso de desarrollo, puede usar herramientas como [Fiddler](https://www.telerik.com/fiddler) para comparar y comprobar solicitudes y respuestas.

Si no es factible configurar un inquilino de prueba independiente, omita esta fase y apunte una instancia de prueba de la aplicación a su inquilino de Azure AD de producción, tal como se describe en la fase 3.

  ![Fase de migración 2 ](media/migrate-adfs-apps-to-azure/stage2.jpg)

#### <a name="stage-3--point-a-test-instance-of-the-app-to-the-production-azure-ad-tenant"></a>Fase 3: Apunte una instancia de prueba de la aplicación al inquilino de Azure AD de producción

Actualice la configuración para que apunte la instancia de prueba de la aplicación al inquilino de producción de Azure AD. Ahora puede probar con los usuarios del inquilino de producción. Si es necesario, revise la sección de este artículo que habla sobre la transición de usuarios.

  ![Fase de migración 3 ](media/migrate-adfs-apps-to-azure/stage3.jpg)

#### <a name="stage-4--point-the-production-app-to-the-production-azure-ad-tenant"></a>Fase 4: Apunte la aplicación de producción al inquilino de Azure AD de producción

Actualice la configuración de la aplicación de producción para que apunte al inquilino de Azure AD de producción.

  ![Fase de migración 4 ](media/migrate-adfs-apps-to-azure/stage4.jpg)

 Las aplicaciones que se autentican con AD FS pueden usar grupos de Active Directory para los permisos. Use la [sincronización de Azure AD Connect](../hybrid/how-to-connect-sync-whatis.md) para sincronizar los datos de identidad entre el entorno local y Azure AD antes de empezar la migración. Compruebe esos grupos y pertenencias antes de la migración para que pueda conceder acceso a los mismos usuarios cuando se migre la aplicación.

### <a name="line-of-business-apps"></a>Aplicaciones de línea de negocio

Las aplicaciones de línea de negocio son aquellas que su organización desarrolló o que son un producto estándar empaquetado. Entre los ejemplos se incluyen aplicaciones basadas en aplicaciones de SharePoint y Windows Identity Foundation (no SharePoint Online).

Las aplicaciones de línea de negocio que usan OAuth 2.0, OpenID Connect o WS-Federation se pueden integrar con Azure AD como [registros de aplicación](../develop/quickstart-register-app.md). Integre aplicaciones personalizadas que usan SAML 2.0 o WS-Federation como [aplicaciones que no son de la galería](add-application-portal.md) en la página de las aplicaciones empresariales en [Azure Portal](https://portal.azure.com/).

## <a name="saml-based-single-sign-on"></a>Inicio de sesión único basado en SAML

Las aplicaciones que usan SAML 2.0 para la autenticación se pueden configurar para el [inicio de sesión único (SSO) basado en SAML](what-is-single-sign-on.md). Con el SSO basado en SAML, puede asignar usuarios a roles de aplicación específicos según las reglas que defina en las notificaciones SAML.

Para configurar una aplicación SaaS para el inicio de sesión único basado en SAML, consulte [Inicio rápido: Configuración del inicio de sesión único basado en SAML](add-application-portal-setup-sso.md).

  ![Capturas de pantallas de usuario de SSO de SAML ](media/migrate-adfs-apps-to-azure/sso-saml-user-attributes-claims.png)

Muchas aplicaciones SaaS tienen un [tutorial específico de la aplicación](../saas-apps/tutorial-list.md) que lo guía por la configuración del SSO basado en SAML.

  ![tutorial de la aplicación](media/migrate-adfs-apps-to-azure/app-tutorial.png)

Algunas aplicaciones se pueden migrar fácilmente. Las aplicaciones con requisitos más complejos, como las notificaciones personalizadas, pueden requerir configuración adicional en Azure AD o Azure AD Connect. Para obtener información acerca de las asignaciones de notificaciones admitidas, consulte [Personalización de las notificaciones emitidas en tokens para una determinada aplicación de un inquilino (versión preliminar)](../develop/active-directory-claims-mapping.md).

Cuando asigne los atributos, considere estas limitaciones:

* No todos los atributos que se pueden emitir en AD FS se muestran en Azure AD como atributos que se van a emitir a los tokens de SAML, incluso si esos atributos están sincronizados. Cuando edita el atributo, la lista desplegable **Valor** muestra los distintos atributos que están disponibles en Azure AD. Compruebe la configuración de los [temas de sincronización de Azure AD Connect](../hybrid/how-to-connect-sync-whatis.md) para asegurarse de que los atributos necesarios (por ejemplo, **samAccountName**) se sincroniza con Azure AD. Puede usar los atributos de extensión para emitir cualquier notificación que no forme parte del esquema de usuario estándar en Azure AD.
* En los escenarios más comunes, solo la notificación **NameID** y otras notificaciones de identificador de usuario comunes son necesarias para una aplicación. Para determinar si se requieren notificaciones adicionales, examine qué notificaciones está emitiendo desde AD FS.
* No todas las notificaciones pueden emitirse, porque algunas notificaciones está protegidas en Azure AD.
* La capacidad de usar tokens SAML cifrados está ahora en versión preliminar. Consulte el procedimiento de [personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales](../develop/active-directory-saml-claims-customization.md).

### <a name="software-as-a-service-saas-apps"></a>Aplicaciones de software como servicio (SaaS)

Si los usuarios inician sesión en aplicaciones SaaS como Salesforce, ServiceNow o Workday y están integradas con AD FS, está usando el inicio de sesión federado para las aplicaciones SaaS.

Es posible que la mayoría de las aplicaciones SaaS esté configurada en Azure AD. Microsoft tiene muchas conexiones preconfiguradas a aplicaciones SaaS de la [Galería de aplicaciones de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps), lo que simplifica la transición. Las aplicaciones de SAML 2.0 se pueden integrar con Azure AD a través de la Galería de aplicaciones de Azure AD o como [aplicaciones que no son de la galería](add-application-portal.md).

Las aplicaciones que usan OAuth 2.0 u OpenID Connect se pueden integrar de forma parecida con Azure AD a los [registros de aplicación](../develop/quickstart-register-app.md). Las aplicaciones que usan protocolos heredados puede usar [Azure AD Application Proxy](../app-proxy/application-proxy.md) para autenticarse con Azure AD.

Si tiene algún problema para incorporar las aplicaciones SaaS, puede ponerse en contacto con el [alias de soporte técnico de la integración de aplicaciones SaaS](mailto:SaaSApplicationIntegrations@service.microsoft.com).

### <a name="saml-signing-certificates-for-sso"></a>Certificados de firma de SAML para SSO

Los certificados de firma son una parte importante de cualquier implementación de SSO. Azure AD crea los certificados de firma para establecer el SSO federado basado en SAML para las aplicaciones SaaS. Una vez que agregue aplicaciones de la galería o que no son de la galería, configurará la aplicación agregada con la opción de SSO federado. Consulte [Administrar certificados para inicio de sesión único federado en Azure Active Directory](manage-certificates-for-federated-single-sign-on.md).

### <a name="saml-token-encryption"></a>Cifrado de tokens SAML

Tanto AD FS como Azure AD ofrecen cifrado de tokens; es decir, la capacidad de cifrar las aserciones de seguridad de SAML que se dirigen a las aplicaciones. Las aserciones se cifran con una clave pública y la aplicación receptora las descifra con la clave privada correspondiente. Al configurar el cifrado de tokens, se cargan los archivos de certificado X.509 para proporcionar las claves públicas.

Para obtener información acerca del cifrado de tokens SAML de Azure AD y cómo configurarlo, consulte [Configuración del cifrado de tokens SAML de Azure AD](howto-saml-token-encryption.md).  

> [!NOTE]
> El cifrado de tokens es una característica Premium de Azure Active Directory (Azure AD). Para más información sobre las ediciones, las características y los precios de Azure AD, consulte [Precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="apps-and-configurations-that-can-be-moved-today"></a>Aplicaciones y configuraciones que se pueden migrar actualmente

Las aplicaciones que se pueden mover fácilmente en la actualidad incluyen las aplicaciones SAML 2.0 que usan el conjunto estándar de notificaciones y elementos de configuración. Estos elementos estándar son:

* Nombre principal del usuario
* Dirección de correo electrónico
* Nombre propio
* Surname
* Atributo alternativo de **NameID** de SAML, lo que incluye el atributo de correo de Azure AD, el prefijo de correo, el identificador de empleado, los atributos de extensión 1 a 15 o el atributo **SamAccountName** local. Para más información, consulte [Edición de la notificación NameIdentifier](../develop/active-directory-saml-claims-customization.md).
* Notificaciones personalizadas.

Lo siguiente requiere pasos de configuración adicionales para migrar a Azure AD:

* Autorización personalizada o reglas de autenticación multifactor (MFA) en AD FS. Se configuran mediante la característica [Acceso condicional de Azure AD](../conditional-access/overview.md).
* Aplicaciones con varios puntos de conexión de URL de respuesta. Se configuran en Azure AD mediante PowerShell o la interfaz de Azure Portal.
* Aplicaciones de WS-Federation, como las aplicaciones de SharePoint, que requieren tokens de la versión 1.1 de SAML. Puede configurarlas manualmente con PowerShell. También puede agregar una plantilla genérica integrada previamente para aplicaciones de SharePoint y SAML 1.1 de la galería. Se admite el protocolo SAML 2.0.
* Reglas de transformaciones de emisión de notificaciones complejas. Para información sobre las asignaciones de notificaciones compatibles, consulte:
   *  [Asignación de notificaciones en Azure Active Directory](../develop/active-directory-claims-mapping.md).
   * [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure Active Directory](../develop/active-directory-saml-claims-customization.md).

### <a name="apps-and-configurations-not-supported-in-azure-ad-today"></a>Aplicaciones y configuraciones no admitidas actualmente en Azure AD

Las aplicaciones que necesitan ciertas funcionalidades no se pueden migrar en estos momentos.

#### <a name="protocol-capabilities"></a>Funcionalidades de protocolo

Las aplicaciones que necesitan las funcionalidades de protocolo siguientes no se pueden migrar en estos momentos:

* Compatibilidad con el patrón ActAs de WS-Trust
* Resolución de artefactos SAML
* Comprobación de la firma de las solicitudes SAML firmadas ‎
  > [!Note]
  > Se aceptan solicitudes firmadas, pero no se comprueba la firma.

  ‎Dado que Azure AD solo devuelve el token a los puntos de conexión preconfigurados en la aplicación, es probable que la comprobación de la firma no sea necesaria en la mayoría de los casos.

#### <a name="claims-in-token-capabilities"></a>Funcionalidades de notificaciones en token

Las aplicaciones que necesitan las funcionalidades de notificación en token siguientes no se pueden migrar en estos momentos.

* Notificaciones de almacenes de atributos que no son el directorio de Azure AD, a menos que los datos estén sincronizados con Azure AD. Para más información, consulte [Introducción a la API de sincronización de Azure AD](/graph/api/resources/synchronization-overview).
* Emisión de atributos de varios valores de directorio. Por ejemplo, en este momento no se puede emitir una notificación de varios valores para las direcciones de proxy.

## <a name="map-app-settings-from-ad-fs-to-azure-ad"></a>Asignación de la configuración de una aplicación de AD FS a Azure AD

La migración requiere la evaluación de cómo está configurada la aplicación en el entorno local y, a continuación, la asignación de esa configuración a Azure AD. AD FS y Azure AD funcionan de forma similar, de modo que los conceptos de configuración de direcciones URL e identificadores de confianza de inicio y cierre de sesión se aplican en ambos casos. Documente las opciones de configuración de AD FS de las aplicaciones para que pueda configurarlas fácilmente en Azure AD.

### <a name="map-app-configuration-settings"></a>Asignación de las opciones de configuración de aplicaciones

En la tabla siguiente se describen algunas de las asignaciones más comunes de configuración entre una relación de confianza para usuario autenticado de AD FS y una aplicación empresarial de Azure AD:

* AD FS: busque la configuración en la relación de confianza para usuario autenticado de AD FS correspondiente a la aplicación. Haga clic con el botón derecho en el usuario de confianza y seleccione Propiedades.
* Azure AD: la configuración se establece dentro de [Azure Portal](https://portal.azure.com/) en las propiedades de SSO de cada aplicación.

| Opción de configuración| AD FS| Configuración en Azure AD| Token SAML |
| - | - | - | - |
| **Dirección URL de inicio de sesión de aplicación** <p>Dirección URL para que el usuario inicie sesión en la aplicación en un flujo de SAML iniciado por el proveedor de servicios (SP).| N/D| Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| N/D |
| **Dirección URL de respuesta de la aplicación** <p>Dirección URL de la aplicación desde la perspectiva del proveedor de identidades (IdP). El IdP envía aquí el usuario y el token después de que el usuario inicia sesión en el IdP.  ‎Esto también se conoce como el **punto de conexión del consumidor de aserción de SAML**.| Seleccione la pestaña **Puntos de conexión**| Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| Elemento Destino en el token SAML. Valor de ejemplo: `https://contoso.my.salesforce.com` |
| **Dirección URL de cierre de sesión de la aplicación** <p>Esta es la dirección URL a la que se envían las solicitudes de limpieza de cierre de sesión cuando un usuario cierra la sesión de una aplicación. El IdP envía la solicitud para también cerrar la sesión del usuario de todas las demás aplicaciones.| Seleccione la pestaña **Puntos de conexión**| Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| N/D |
| **Identificador de la aplicación** <p>Este es el identificador de la aplicación desde la perspectiva del IdP. El valor de la dirección URL de inicio de sesión se usa a menudo (pero no siempre) como identificador.  ‎En ocasiones, la aplicación lo llama el "identificador de entidad".| Seleccione la pestaña **Identificadores**|Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| Se asigna al elemento **Audiencia** del token SAML. |
| **Metadatos de federación de la aplicación** <p>Se trata de la ubicación de los metadatos de federación de la aplicación. El IdP la usa para actualizar automáticamente valores de configuración específicos, como puntos de conexión o certificados de cifrado.| Seleccione la pestaña **Supervisión**| N/D Azure AD no admite el consumo directo de metadatos de federación de aplicaciones. Puede importar manualmente los metadatos de federación.| N/D |
| **Identificador de usuario / identificador de nombre** <p>Atributo que se usa para indicar de forma única la identidad de usuario de Azure AD o AD FS para la aplicación.  Por lo general, este atributo es el UPN o la dirección de correo electrónico del usuario.| Reglas de notificación. En la mayoría de los casos, la regla de notificación emite una notificación con un tipo que termina con **NameIdentifier**.| Puede encontrar el identificador bajo el encabezado **Atributos y notificaciones de usuario**. De manera predeterminada, se usa el UPN.| Se asigna al elemento **NameID** en el token SAML. |
| **Otras notificaciones** <p>Entre los ejemplos de otra información de notificación que se envía habitualmente desde el IdP a la aplicación se incluyen el nombre, el apellido, la dirección de correo electrónico y la pertenencia a grupos.| En AD FS, se encuentra como otras reglas de notificación en el usuario de confianza.| Puede encontrar el identificador bajo el encabezado **Atributos y notificaciones de usuario**. Seleccione **Ver** para editar todos los demás atributos de usuario.| N/D |

### <a name="map-identity-provider-idp-settings"></a>Asignación de la configuración del proveedor de identidades (IdP)

Configure las aplicaciones para que apunten a Azure AD en vez de AD FS para SSO. Aquí nos centraremos en las aplicaciones SaaS que usan el protocolo SAML. Sin embargo, este concepto se extiende también a las aplicaciones de línea de negocio personalizadas.

> [!NOTE]
> Los valores de configuración de Azure AD siguen el patrón en el que el identificador del inquilino de Azure reemplaza {tenant-ID} y el identificador de la aplicación reemplaza {Application-ID}. Esta información la encuentra en [Azure Portal](https://portal.azure.com/) en **Azure Active Directory > Propiedades**:

* Seleccione el id. de directorio para ver el id. del inquilino.
* Seleccione el id. de aplicación para ver el id. de la aplicación.

 En un nivel alto, asigne los elementos de configuración de las aplicaciones SaaS clave siguientes a Azure AD.

| Elemento| Valor de configuración |
| - | - |
| Emisor del proveedor de identidades| https:\//sts.windows.net/{tenant-id}/ |
| URL de inicio de sesión del proveedor de identidades| [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) |
| URL de cierre de sesión del proveedor de identidades| [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) |
| Ubicación de los metadatos de federación| [https://login.windows.net/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={application-id}](https://login.windows.net/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={application-id}) |

### <a name="map-sso-settings-for-saas-apps"></a>Asignación de la configuración de SSO para aplicaciones SaaS

Las aplicaciones SaaS deben saber dónde enviar las solicitudes de autenticación y cómo validar los tokens recibidos. En la tabla siguiente se describen los elementos para configurar los valores de inicio de sesión único en la aplicación y sus valores o ubicaciones dentro de AD FS y Azure AD.

| Opción de configuración| AD FS| Configuración en Azure AD |
| - | - | - |
| **Dirección URL de inicio de sesión del IdP** <p>Dirección URL de inicio de sesión del IdP desde la perspectiva de la aplicación (donde se redirige al usuario para el inicio de sesión).| La dirección URL de inicio de sesión de AD FS es el nombre del servicio de federación de AD FS seguido de "/adfs/ls/". <p>Por ejemplo: `https://fs.contoso.com/adfs/ls/`| Reemplace {tenant id} por el id. del inquilino. <p> ‎Para las aplicaciones que usan el protocolo SAML-P: [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) <p>‎Para las aplicaciones que usan el protocolo WS-Federation: [https://login.microsoftonline.com/{tenant-id}/wsfed](https://login.microsoftonline.com/{tenant-id}/wsfed) |
| **Dirección URL de cierre de sesión del IdP**<p>Dirección URL de cierre de sesión del IdP desde la perspectiva de la aplicación (donde se redirige al usuario cuando elige cerrar sesión en la aplicación).| La dirección URL de cierre de sesión es la misma que la dirección URL de inicio de sesión o la misma dirección URL seguida de "wa=wsignout1.0". Por ejemplo: `https://fs.contoso.com/adfs/ls/?wa=wsignout1.0`| Reemplace {tenant id} por el id. del inquilino.<p>Para las aplicaciones que usan el protocolo SAML-P:<p>[https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) <p> ‎Para las aplicaciones que usan el protocolo WS-Federation: [https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0](https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0) |
| **Certificado de firma de tokens**<p>El IdP usa la clave privada del certificado para firmar los tokens emitidos. Se comprueba que el token procede del mismo IdP que se ha configurado como de confianza en la aplicación.| El certificado de firma de token de AD FS se encuentra en Administración de AD FS, en **Certificados**.| Lo encontrará en Azure Portal en las **Propiedades del inicio de sesión único** de la aplicación, bajo el encabezado **Certificado de firma de SAML**. Desde allí puede descargar el certificado para cargarlo en la aplicación.  <p>Si la aplicación tiene más de un certificado, todos los certificados se pueden encontrar en el archivo XML de metadatos de federación. |
| **Identificador/ "emisor"**<p>Identificador del IdP desde la perspectiva de la aplicación (también llamado "Identificador de emisor").<p>‎En el token SAML, el valor aparece como el elemento Emisor.| Por lo general, el identificador de AD FS es el identificador del servicio de federación en Administración de AD FS, bajo **Servicio > Modificar las propiedades del Servicio de federación**. Por ejemplo: `http://fs.contoso.com/adfs/services/trust`| Reemplace {tenant id} por el id. del inquilino.<p>https:\//sts.windows.net/{tenant-id}/ |
| **Metadatos de federación del IdP**<p>Ubicación de los metadatos de federación disponibles públicamente del IdP. (Algunas aplicaciones usan metadatos de federación como alternativa a la configuración de direcciones URL, identificadores y certificados de firma de token de forma individual por el administrador).| Puede encontrar la dirección URL de metadatos de federación de AD FS en Administración de AD FS en **Servicio > Puntos de conexión > Metadatos > Tipo: Metadatos de federación**. Por ejemplo: `https://fs.contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`| El valor correspondiente para Azure AD sigue el patrón [https://login.microsoftonline.com/{TenantDomainName}/FederationMetadata/2007-06/FederationMetadata.xml](https://login.microsoftonline.com/{TenantDomainName}/FederationMetadata/2007-06/FederationMetadata.xml). Reemplace {TenantDomainName} por el nombre del inquilino con el formato "contoso.onmicrosoft.com."   <p>Para más información, consulte [Metadatos de federación](../azuread-dev/azure-ad-federation-metadata.md). |

## <a name="represent-ad-fs-security-policies-in-azure-ad"></a>Representación de las directivas de seguridad de AD FS en Azure AD

Cuando mueva la autenticación de las aplicaciones a Azure AD, cree asignaciones desde las directivas de seguridad existentes a sus variantes equivalentes o alternativas disponibles en Azure AD. Asegurarse de que estas asignaciones se pueden realizar mientras se cumplan los estándares de seguridad necesarios por los propietarios de la aplicación hace que el resto de la migración de la aplicación sea mucho más fácil.

Para cada ejemplo de regla se muestra se ve la regla en AD FS, el código equivalente en el lenguaje de la regla AD FS y cómo se asigna a Azure AD.

### <a name="map-authorization-rules"></a>Asignación de reglas de autorización

A continuación se muestran ejemplos de varios tipos de reglas de autorización en AD FS y cómo asignarlas a Azure AD.

#### <a name="example-1-permit-access-to-all-users"></a>Ejemplo 1: Permitir el acceso a todos los usuarios

Permita el acceso a todos los usuarios en AD FS:

  ![Captura de pantalla que muestra el cuadro de diálogo Configurar el inicio de sesión único con SAML.](media/migrate-adfs-apps-to-azure/permit-access-to-all-users-1.png)

Esto se asigna a Azure AD de una de las maneras siguientes:

1. Establezca la opción **Asignación de usuarios necesaria** en **No**.

    ![edite la directiva de control de acceso para aplicaciones SaaS ](media/migrate-adfs-apps-to-azure/permit-access-to-all-users-2.png)

    > [!Note]
    > Establecer la opción **Asignación de usuarios necesaria** a **Sí** requiere que los usuarios estén asignados a la aplicación para que obtengan acceso. Cuando se establece en **No**, todos los usuarios tienen acceso. Este cambios no controla lo que los usuarios ven en la experiencia **Aplicaciones**.

1. En la pestaña **Usuarios y grupos**, asigne la aplicación al grupo automático **Todos los usuarios**. Debe [habilitar los grupos dinámicos](../enterprise-users/groups-create-rule.md) en el inquilino de Azure AD para que el grupo predeterminado **Todos los usuarios** esté disponible.

    ![Mis aplicaciones SaaS en Azure AD ](media/migrate-adfs-apps-to-azure/permit-access-to-all-users-3.png)

#### <a name="example-2-allow-a-group-explicitly"></a>Ejemplo 2: Permitir un grupo de manera explícita

Autorización explícita de un grupo en AD FS:

  ![Captura de pantalla que muestra el cuadro de diálogo Editar regla para la regla de notificaciones para permitir administradores de dominio.](media/migrate-adfs-apps-to-azure/allow-a-group-explicitly-1.png)

Para asignar esta regla a Azure AD, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), [cree un grupo de usuarios](../fundamentals/active-directory-groups-create-azure-portal.md) que se corresponderá con el grupo de usuarios de AD FS.
1. Asigne permisos de aplicación al grupo:

    ![Agregar asignación ](media/migrate-adfs-apps-to-azure/allow-a-group-explicitly-2.png)

#### <a name="example-3-authorize-a-specific-user"></a>Ejemplo 3: Autorizar a un usuario específico

Autorización explícita de un usuario en AD FS:

  ![Captura de pantalla que muestra el cuadro de diálogo Editar regla para permitir un usuario especifico para una regla de notificaciones, con un tipo de notificación entrante de SID principal.](media/migrate-adfs-apps-to-azure/authorize-a-specific-user-1.png)

Para asignar esta regla a Azure AD, haga lo siguiente:

* En [Azure Portal](https://portal.azure.com/), agregue un usuario a la aplicación a través de la pestaña Agregar asignación de la aplicación, tal como se muestra a continuación:

    ![Mis aplicaciones SaaS en Azure ](media/migrate-adfs-apps-to-azure/authorize-a-specific-user-2.png)

### <a name="map-multi-factor-authentication-rules"></a>Asignación de reglas de autenticación multifactor

Una implementación local de [Multi-Factor Authentication (MFA)](../authentication/concept-mfa-howitworks.md) y AD FS sigue funcionando después de la migración, ya que está federada con AD FS. Sin embargo, considere la posibilidad de migrar a las funcionalidades de MFA integradas de Azure que están asociadas a los flujos de trabajo de acceso condicional de Azure AD.

A continuación se muestran ejemplos de tipos de reglas de MFA en AD FS y cómo se pueden asignar a Azure AD en función de condiciones distintas.

Configuración de reglas de MFA en AD FS:

  ![Captura de pantalla que muestra las condiciones de Azure AD en Azure Portal.](media/migrate-adfs-apps-to-azure/mfa-settings-common-for-all-examples.png)

#### <a name="example-1-enforce-mfa-based-on-usersgroups"></a>Ejemplo 1: Aplicación de MFA en función de usuarios o grupos

El selector de usuarios o grupos es una regla que permite aplicar MFA en cada grupo (SID de grupo) o usuario (SID principal). Además de las asignaciones de usuarios o grupos, todas las casillas adicionales de la interfaz de usuario de la configuración de AD FS MFA funcionan como reglas adicionales que se evalúan después de aplicar la regla de usuarios o grupos.

Especifique las reglas de MFA para un usuario o un grupo en Azure AD:

1. Cree una [directiva de acceso condicional nueva](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json).
1. Seleccione **Asignaciones**. Agregue los usuarios o los grupos en los que quiere aplicar MFA.
1. Configure las opciones de **controles de acceso**, tal como se muestra a continuación:

    ‎![Captura de pantalla que muestra el panel Concesión, en el que puede conceder acceso.](media/migrate-adfs-apps-to-azure/mfa-users-groups.png)

 #### <a name="example-2-enforce-mfa-for-unregistered-devices"></a>Ejemplo 2: Aplicación de MFA para dispositivos no registrados

Especifique las reglas de MFA para los dispositivos no registrados en Azure AD:

1. Cree una [directiva de acceso condicional nueva](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json).
1. Establezca las **Asignaciones** en **Todos los usuarios**.
1. Configure las opciones de **controles de acceso**, tal como se muestra a continuación:

    ![Captura de pantalla que muestra el panel Concesión, en el que puede conceder acceso y especificar otras restricciones.](media/migrate-adfs-apps-to-azure/mfa-unregistered-devices.png)

Cuando se establece la opción **Para varios controles** en **Requerir uno de los controles seleccionados**, significa que si el usuario cumple con cualquiera de las condiciones especificadas por la casilla, se le concede acceso a la aplicación.

#### <a name="example-3-enforce-mfa-based-on-location"></a>Ejemplo 3: Aplicación de MFA en función de la ubicación

Especifique las reglas de MFA en función de la ubicación de un usuario en Azure AD:

1. Cree una [directiva de acceso condicional nueva](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json).
1. Establezca las **Asignaciones** en **Todos los usuarios**.
1. [Configure ubicaciones con nombre en Azure AD](../conditional-access/location-condition.md). De lo contrario, se confía en la federación desde dentro de la red corporativa.
1. Configure las **Reglas de condiciones** para especificar las ubicaciones para las que quiere aplicar MFA.

    ![Captura de pantalla que muestra el panel Ubicaciones para las reglas de condiciones.](media/migrate-adfs-apps-to-azure/mfa-location-1.png)

1. Configure las opciones de **controles de acceso**, tal como se muestra a continuación:

    ![Asignación de directivas de control de acceso](media/migrate-adfs-apps-to-azure/mfa-location-2.png)

### <a name="map-emit-attributes-as-claims-rule"></a>Asignación de atributos de emisión como regla de notificaciones

Emita atributos como regla de notificaciones en AD FS:

  ![Captura de pantalla que muestra el cuadro de diálogo Editar regla para emitir atributos como notificaciones.](media/migrate-adfs-apps-to-azure/map-emit-attributes-as-claims-rule-1.png)

Para asignar la regla en Azure AD, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), seleccione **Aplicaciones empresariales** y, a continuación, **Inicio de sesión único** para ver la configuración de inicio de sesión basado en SAML:

    ![Captura de pantalla que muestra la página Inicio de sesión único para su aplicación empresarial.](media/migrate-adfs-apps-to-azure/map-emit-attributes-as-claims-rule-2.png)

1. Seleccione **Editar** (opción resaltada) para modificar los atributos:

    ![Esta es la página para editar los atributos y las notificaciones de usuario](media/migrate-adfs-apps-to-azure/map-emit-attributes-as-claims-rule-3.png)

### <a name="map-built-in-access-control-policies"></a>Asignación de directivas de control de acceso integradas

Directivas de control de acceso integradas en AD FS 2016:

  ![Control de acceso integrado de Azure AD](media/migrate-adfs-apps-to-azure/map-built-in-access-control-policies-1.png)

Para implementar directivas integradas en Azure AD, use una [directiva de acceso condicional nueva](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json) y configure los controles de acceso, o bien puede usar el diseñador de directivas personalizadas en AD FS 2016 para configurar las directivas de control de acceso. El editor de reglas tiene una lista exhaustiva de las opciones Permitir y Excepto que puede ayudarlo a realizar todo tipo de permutaciones.

  ![Directivas de control de acceso de Azure AD](media/migrate-adfs-apps-to-azure/map-built-in-access-control-policies-2.png)

En esta tabla, se enumeran algunas opciones Permitir y Excepto y cómo se asignan a Azure AD.

| Opción | ¿Cómo se configura la opción Permitir en Azure AD?| ¿Cómo se configura la opción Excepto en Azure AD? |
| - | - | - |
| Desde una red específica| Se asigna a [Ubicación con nombre](../conditional-access/location-condition.md) en Azure AD| Use la opción **Excluir** para las [ubicaciones de confianza](../conditional-access/location-condition.md) |
| Desde grupos específicos| [Establezca una asignación de usuario o grupos](assign-user-or-group-access-portal.md)| Use la opción **Excluir** en Usuarios y Grupos |
| Desde dispositivos con un nivel de confianza específico| Establezca esta opción desde el control **Estado del dispositivo** en Asignaciones -> Condiciones| Use la opción **Excluir** en la condición de estado del dispositivo e incluya **Todos los dispositivos** |
| Con notificaciones específicas en la solicitud| No se puede migrar esta configuración| No se puede migrar esta configuración |

Este es un ejemplo de cómo configurar la opción Excluir para las ubicaciones de confianza en Azure Portal:

  ![Captura de pantalla de la asignación de directivas de control de acceso](media/migrate-adfs-apps-to-azure/map-built-in-access-control-policies-3.png)

## <a name="transition-users-from-ad-fs-to-azure-ad"></a>Transición de usuarios de AD FS a Azure AD

### <a name="sync-ad-fs-groups-in-azure-ad"></a>Sincronización de grupos de AD FS en Azure AD

Cuando se asignan reglas de autorización, las aplicaciones que se autentican con AD FS pueden usar los grupos de Active Directory para los permisos. En tal caso, use [Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771) para sincronizar estos grupos con Azure AD antes de migrar las aplicaciones. Asegúrese de comprobar esos grupos y pertenencias antes de la migración para que pueda conceder acceso a los mismos usuarios cuando se migre la aplicación.

Para más información, consulte [Requisitos previos para usar atributos de grupo sincronizados desde Active Directory](../hybrid/how-to-connect-fed-group-claims.md).

### <a name="set-up-user-self-provisioning"></a>Configuración del aprovisionamiento automático de usuarios

Algunas aplicaciones SaaS admiten la capacidad de aprovisionar automáticamente a los usuarios que inician sesión por primera vez en la aplicación. En Azure AD, el término aprovisionamiento de aplicaciones hace referencia a la creación automática de identidades y roles de usuario en las aplicaciones de nube ([SaaS](https://azure.microsoft.com/overview/what-is-saas/)) a las que los usuarios necesitan acceder. Los usuarios que se migren ya tienen una cuenta en la aplicación SaaS. Los usuarios nuevos que se agreguen después de la migración deben aprovisionarse. Pruebe el [aprovisionamiento de aplicaciones SaaS](../app-provisioning/user-provisioning.md) una vez que se migre la aplicación.

### <a name="sync-external-users-in-azure-ad"></a>Sincronización de los usuarios externos en Azure AD

Los usuarios externos existentes se pueden configurar de las siguientes dos maneras en AD FS:

* **Usuarios externos con una cuenta local dentro de la organización**: siga usando estas cuentas de la misma manera en que funcionan las cuentas de usuario internas. Estas cuentas de usuario externas tienen un nombre principal dentro de la organización, aunque el correo electrónico de la cuenta puede apuntar externamente. A medida que la migración progresa, puede aprovechar las ventajas que [Azure AD B2B](../external-identities/what-is-b2b.md) ofrece al migrar estos usuarios para que usen su propia identidad corporativa cuando dicha identidad está disponible. Esto simplifica el proceso de inicio de sesión para esos usuarios, ya que a menudo inician sesión con un inicio de sesión corporativo propio. También se facilita la administración de la organización, porque no tiene que administrar las cuentas para usuarios externos.
* **Identidades externas federadas**: si actualmente se está federando con una organización externa, hay algunos enfoques que puede usar:
  * [Incorpore usuarios de colaboración B2B de Azure Active Directory en Azure Portal](../external-identities/add-users-administrator.md). Puede enviar de manera proactiva invitaciones de colaboración B2B desde el portal administrativo de Azure AD a la organización asociada para que cada miembro siga usando las aplicaciones y los recursos que acostumbra usar.
  * [Cree un flujo de trabajo de registro de B2B de autoservicio](../external-identities/self-service-portal.md) que genere una solicitud para usuarios individuales en la organización asociada mediante la API de invitación de B2B.

Independientemente de la configuración de los usuarios externos existentes, es probable que tengan permisos asociados con su cuenta, ya sea en la pertenencia a grupos o en permisos específicos. Evalúe si estos permisos se deben migrar o limpiar. Las cuentas dentro de su organización que representan a un usuario externo se deben deshabilitar una vez que el usuario ha migrado a una identidad externa. El proceso de migración se debe describir con sus socios comerciales, ya que puede ocurrir una interrupción en su capacidad de conectarse con los recursos.

## <a name="migrate-and-test-your-apps"></a>Migración y prueba de las aplicaciones

Siga el proceso de migración que se detalla en este artículo. Luego, vaya a [Azure Portal](https://aad.portal.azure.com/) para probar si la migración se realizó correctamente.

Siga estas instrucciones:

1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** y busque la aplicación en la lista.
1. Seleccione **Administrar** > **Usuarios y grupos** para asignar al menos un usuario o grupo a la aplicación.
1. Seleccione **Administrar** > **Acceso condicional**. Revise la lista de directivas y asegúrese de que no está bloqueando el acceso a la aplicación con una [directiva de acceso condicional](../conditional-access/overview.md).

En función de cómo configure la aplicación, compruebe que el inicio de sesión único funciona correctamente.

| Tipo de autenticación| Prueba |
| :- | :- |
| OAuth / OpenID Connect| Seleccione **Aplicaciones empresariales > Permisos** y asegúrese de haber dado su consentimiento a la aplicación desde la configuración del usuario correspondiente a la aplicación.|
| SSO basado en SAML | Use el botón [Probar la configuración de SAML](debug-saml-sso-issues.md) que está en **Inicio de sesión único**. |
| SSO basado en contraseñas |  Descargue e instale la [Extensión de inicio de](../user-help/my-apps-portal-end-user-access.md)[-](../user-help/my-apps-portal-end-user-access.md)[sesión seguro de Aplicaciones](../user-help/my-apps-portal-end-user-access.md). Esta extensión lo ayudará a iniciar cualquiera de las aplicaciones en la nube de su organización que requiera que se use un proceso de inicio de sesión único. |
| Proxy de aplicación | Asegúrese de que el conector esté en ejecución y que esté asignado a la aplicación. Visite la [Solución de problemas y mensajes de error de Proxy de aplicación](../app-proxy/application-proxy-troubleshoot.md) para obtener más ayuda. |

> [!NOTE]
> Las cookies del entorno de AD FS anterior persisten en las máquinas de usuario. Estas cookies pueden causar problemas con la migración, ya que los usuarios podrían dirigirse al entorno de inicio de sesión de AD FS anterior, en lugar de al inicio de sesión nuevo de Azure AD. Es posible que tenga que borrar las cookies del explorador del usuario de manera manual o mediante un script. También puede usar System Center Configuration Manager o una plataforma similar.

### <a name="troubleshoot"></a>Solución de problemas

Si hay algún error en la prueba de las aplicaciones migradas, solucionar los problemas podría ser el primer paso antes de revertir a los usuarios de confianza de AD FS existentes. Consulte [Depuración del inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](debug-saml-sso-issues.md).

### <a name="rollback-migration"></a>Reversión de la migración

Si se produce un error en la migración, se recomienda dejar los usuarios de confianza existentes en los servidores de AD FS y quitar el acceso a los usuarios de confianza. Esto lee permite realizar una reversión rápida durante la implementación si es necesario.

### <a name="employee-communication"></a>Comunicación de empleados

Aunque la ventana de interrupción planeada puede ser mínima, de todos modos debe planear la comunicación de estos plazos a los empleados de forma proactiva, a la vez que se migra de AD FS a Azure AD. Asegúrese de que la experiencia de la aplicación tenga un botón de comentarios o punteros al departamento de soporte técnico en caso de problemas.

Una vez completada la implementación, puede informar a los usuarios de la implementación correcta y recordarles los pasos que deben seguir.

* Indique a los usuarios que usen la página [Aplicaciones](https://myapps.microsoft.com) para acceder a todas las aplicaciones migradas.
* Recuerde a los usuarios que es posible que necesiten actualizar la configuración de MFA.
* Si se implementa el autoservicio de restablecimiento de contraseña, es posible que los usuarios necesiten actualizar o comprobar sus métodos de autenticación. Consulte las plantillas de comunicación con el usuario final de [MFA](https://aka.ms/mfatemplates) y [SSPR](https://aka.ms/ssprtemplates).

### <a name="external-user-communication"></a>Comunicación de usuarios externos

Por lo general, este grupo de usuarios es el que se ve afectado de forma más crítica en caso de problemas. Esto es especialmente cierto si su postura de seguridad dicta un conjunto diferente de reglas de acceso condicional o perfiles de riesgo para los asociados externos. Asegúrese de que los asociados externos conocen la programación de migración a la nube y que tienen un período de tiempo durante el cual se les anima a participar en una implementación piloto que prueba todos los flujos exclusivos de la colaboración externa. Por último, asegúrese de que tienen una manera de acceder al departamento de soporte técnico en caso de problemas.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Migración de la autenticación de aplicaciones a Azure AD](https://aka.ms/migrateapps/whitepaper).
* Configure el [acceso condicional](../conditional-access/overview.md) y [MFA](../authentication/concept-mfa-howitworks.md).
* Pruebe un ejemplo de código paso a paso: [Cuaderno de estrategias de migración de aplicaciones de AD FS a Azure AD para desarrolladores](https://aka.ms/adfsplaybook).