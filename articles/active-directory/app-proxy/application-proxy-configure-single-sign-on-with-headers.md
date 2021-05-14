---
title: Inicio de sesión único basado en encabezados para aplicaciones locales con App Proxy de Azure AD
description: Aprenda a proporcionar inicio de sesión único para aplicaciones locales que estén protegidas con autenticación basada en encabezados.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 6b81ad97a99115ab9741b2200437774b502b4fa6
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108227225"
---
# <a name="header-based-single-sign-on-for-on-premises-apps-with-azure-ad-app-proxy"></a>Inicio de sesión único basado en encabezados para aplicaciones locales con App Proxy de Azure AD

Application Proxy de Azure Active Directory (Azure AD) admite de forma nativa el acceso de inicio sesión único a las aplicaciones que usan encabezados para la autenticación. Puede configurar los valores de encabezado que necesita la aplicación en Azure AD. Los valores de encabezado se envían a la aplicación a través de Application Proxy. Algunas de las ventajas que ofrece el uso de la compatibilidad nativa con la autenticación basada en encabezados mediante Application Proxy incluyen:  

* **Simplificación del acceso remoto a las aplicaciones locales**: App Proxy permite simplificar la arquitectura de acceso remoto existente. Puede reemplazar el acceso VPN a estas aplicaciones. También puede quitar dependencias de las soluciones de identidad locales para la autenticación. Los usuarios no van a notar ninguna diferencia al iniciar sesión para usar las aplicaciones corporativas. Podrán seguir trabajando desde cualquier lugar y en cualquier dispositivo.  

* **Ningún otro software ni cambios en las aplicaciones**: puede usar los conectores existentes de Application Proxy y no es necesario instalar ningún software adicional.  

* **Amplia lista de atributos y transformaciones disponibles**: todos los valores de encabezado disponibles se basan en notificaciones estándar que emite Azure AD. Todos los atributos y las transformaciones disponibles para [configurar notificaciones para aplicaciones SAML o OIDC](../develop/active-directory-saml-claims-customization.md#attributes) también están disponibles para su empleo como valores de encabezado. 

## <a name="pre-requisites"></a>Requisitos previos
Antes de empezar a trabajar con el inicio de sesión único para aplicaciones de autenticación basada en encabezados, asegúrese de que el entorno esté preparado con los siguientes valores y configuraciones:
- Debe habilitar Application Proxy e instalar un conector que tenga línea de sitio a las aplicaciones. Vea el tutorial [Adición de una aplicación local para el acceso remoto mediante Application Proxy](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad) para obtener más información sobre cómo preparar el entorno local, instalar y registrar un conector, y probarlo. 

## <a name="supported-capabilities"></a>Funcionalidades admitidas

En la tabla siguiente se indican las capacidades comunes necesarias para las aplicaciones de autenticación basada en encabezados que son compatibles con Application Proxy. 

|Requisito   |Descripción|
|----------|-----------|
|SSO federado |En el modo previo a la autenticación, todas las aplicaciones están protegidas con la autenticación de Azure AD y permiten a los usuarios el inicio de sesión único. |
|Acceso remoto |Application Proxy permite el acceso remoto a la aplicación. Los usuarios pueden acceder a la aplicación desde Internet en cualquier explorador que use el valor Dirección URL externa. Application Proxy no está diseñado para el uso de acceso corporativo. |
|Integración basada en encabezados |Application Proxy realiza la integración de SSO con Azure AD y luego pasa la identidad u otros datos de la aplicación, como encabezados HTTP, a la aplicación. |
|Autorización de aplicaciones |Se pueden especificar directivas comunes en función de la aplicación a la que se accede, la pertenencia a grupos del usuario y otras directivas. En Azure AD, las directivas se implementan mediante [acceso condicional](../conditional-access/overview.md). Las directivas de autorización de aplicaciones solo se aplican a la solicitud de autenticación inicial. |
|Autenticación de actualización a edición superior |Se pueden definir directivas para forzar autenticación adicional, por ejemplo, para obtener acceso a recursos confidenciales. |
|Autorización específica |Proporciona control de acceso en el nivel de dirección URL. Las directivas adicionales se pueden aplicar en función de la dirección URL a la que se accede. La dirección URL interna configurada para la aplicación define el ámbito de la aplicación al que se aplica la directiva. Se aplica la directiva configurada para la ruta de acceso más pormenorizada.  |

> [!NOTE] 
> En este artículo se habla de la conexión de aplicaciones de autenticación basada en encabezados a Azure AD mediante Application Proxy, que es el patrón recomendado. También hay un patrón de integración alternativo que usa PingAccess con Azure AD para habilitar la autenticación basada en encabezados. Para obtener más detalles, vea [Autenticación basada en encabezados para el inicio de sesión único con Application Proxy y PingAccess](application-proxy-ping-access-publishing-guide.md).

## <a name="how-it-works"></a>Cómo funciona

:::image type="content" source="./media/application-proxy-configure-single-sign-on-with-headers/how-it-works.png" alt-text="Funcionamiento del inicio de sesión único basado en encabezados con Application Proxy." lightbox="./media/application-proxy-configure-single-sign-on-with-headers/how-it-works.png":::

1. El administrador personaliza las asignaciones de atributos que requiere la aplicación en el portal de Azure AD. 
2. Cuando un usuario accede a la aplicación, Application Proxy garantiza que se haya autenticado mediante Azure AD. 
3. El servicio en la nube de Application Proxy es consciente de los atributos necesarios. Por lo tanto, el servicio captura las notificaciones correspondientes del token de identificación recibido durante la autenticación. Después, el servicio traduce los valores a los encabezados HTTP necesarios como parte de la solicitud al conector. 
4. Luego, la solicitud se pasa al conector que, a su vez, la pasa a la aplicación de back-end. 
5. La aplicación recibe los encabezados y puede usarlos según sea necesario. 

## <a name="publish-the-application-with-application-proxy"></a>Publicación de la aplicación con Application Proxy

1. Publique la aplicación según las instrucciones de  [Publicación de aplicaciones con Application Proxy](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).  
    - El valor Dirección URL interna determina el ámbito de la aplicación. Si configura el valor Dirección URL interna en la ruta de acceso raíz de la aplicación, todas las subrutas situadas debajo de la raíz reciben la misma configuración de encabezado y demás configuración de la aplicación. 
    - Cree una nueva aplicación para establecer otra asignación de usuario o configuración de encabezado para una ruta de acceso más pormenorizada que la aplicación que ha configurado. En la nueva aplicación, configure la dirección URL interna con la ruta de acceso específica que necesita y luego configure los encabezados concretos necesarios para esta dirección URL. Application Proxy siempre ajusta los valores de configuración a la ruta de acceso más pormenorizada establecida para una aplicación. 

2. Seleccione  **Azure Active Directory**  como  **método de autenticación previa**. 
3. Para asignar un usuario de prueba, vaya a **Usuarios y grupos** y asigne los usuarios y grupos adecuados. 
4. Abra un explorador y vaya a  **Dirección URL externa**  desde la configuración de Application Proxy. 
5. Compruebe que puede conectarse a la aplicación. Aunque pueda conectarse, aún no puede acceder a la aplicación, ya que los encabezados no están configurados. 

## <a name="configure-single-sign-on"></a>Configurar inicio de sesión único 
Antes de empezar a trabajar con el inicio de sesión único para aplicaciones basadas en encabezados, debe haber instalado un conector de Application Proxy que pueda acceder a las aplicaciones de destino. Si no es así, siga los pasos de  [Tutorial: Application Proxy de Azure AD](application-proxy-add-on-premises-application.md)  y luego vuelva aquí. 

1. Cuando la aplicación aparezca en la lista de aplicaciones empresariales, selecciónela y luego seleccione  **Inicio de sesión único**. 
2. Establezca el modo de inicio de sesión único en **Basado en encabezados**. 
3. En Configuración básica, **Azure Active Directory** se selecciona como valor predeterminado. 
4. Seleccione el lápiz de edición en Encabezados para configurar los encabezados que se van a enviar a la aplicación. 
5. Seleccione **Agregar nuevo encabezado**. Proporcione un **Nombre** para el encabezado, seleccione **Atributo** o **Transformación** y, en el menú desplegable, seleccione qué encabezado necesita la aplicación.  
    - Para obtener más información sobre la lista de atributos disponibles, vea [Personalización de notificaciones: atributos](../develop/active-directory-saml-claims-customization.md#attributes). 
    - Para obtener más información sobre la lista de transformaciones disponibles, vea [Personalización de notificaciones: transformaciones](../develop/active-directory-saml-claims-customization.md#claim-transformations). 
    - También puede agregar un **Encabezado de grupo** para enviar todos los grupos de los que forma parte un usuario, o los grupos asignados a la aplicación, como encabezado. Para obtener más información sobre la configuración de grupos como un valor, vea: [Configuración de notificaciones de grupos para aplicaciones](../hybrid/how-to-connect-fed-group-claims.md#add-group-claims-to-tokens-for-saml-applications-using-sso-configuration). 
6. Seleccione Guardar. 

## <a name="test-your-app"></a>Prueba de la aplicación 

Una vez realizados todos los pasos, la aplicación debería estar ejecutándose y disponible. Para probar la aplicación: 
1. Abra una nueva ventana normal o privada del explorador para asegurarse de que los encabezados previamente almacenados en caché se hayan borrado. A continuación, vaya a  **Dirección URL externa**  desde la configuración de Application Proxy.
2. Inicie sesión con la cuenta de prueba que asignó a la aplicación. Si puede cargar la aplicación e iniciar sesión en ella mediante SSO, ha terminado. 

## <a name="considerations"></a>Consideraciones

- Application Proxy se usa para proporcionar acceso remoto a las aplicaciones en el entorno local o en la nube privada. No se recomienda Application Proxy para controlar el tráfico que se origina internamente desde la red corporativa.
- El acceso a las aplicaciones de autenticación basada en encabezados se debe restringir solo al tráfico del conector u otra solución de autenticación basada en encabezados permitida. Esto se suele hacer mediante la restricción del acceso de red a la aplicación mediante una restricción de IP o firewall en el servidor de aplicaciones.

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es el inicio de sesión único?](../manage-apps/what-is-single-sign-on.md)
- [¿Qué es Application Proxy?](what-is-application-proxy.md)
- [Serie de guías de inicio rápido sobre la administración de aplicaciones](../manage-apps/view-applications-portal.md)
