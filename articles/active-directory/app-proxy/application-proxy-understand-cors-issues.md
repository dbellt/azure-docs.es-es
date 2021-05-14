---
title: Descripción y solución de los problemas de los conectores de CORS en Azure Active Directory Application Proxy
description: Se proporciona información de CORS en Azure Active Directory Application Proxy y cómo identificar y resolver problemas de CORS.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: troubleshooting
ms.date: 04/28/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: c9331acc65b5a3e4b1f0acdda4ecccb5a6ca3044
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230066"
---
# <a name="understand-and-solve-azure-active-directory-application-proxy-cors-issues"></a>Descripción y solución de los problemas de los conectores de CORS en Azure Active Directory Application Proxy

El [uso compartido de recursos entre orígenes (CORS)](https://www.w3.org/TR/cors/) a veces puede presentar desafíos para las aplicaciones y API que publica a través de Azure Active Directory Application Proxy. En este artículo se describen problemas y soluciones de CORS en Azure Active Directory Application Proxy.

En general, la seguridad del explorador impide que una página web realice solicitudes AJAX a otro dominio. Esta restricción se conoce como *directiva de mismo origen* y evita que un sitio malintencionado lea información confidencial de otro sitio. Sin embargo, a veces es posible que quiera permitir que otros llamen a su API web. CORS es una norma de W3C que permite que un servidor se relaje con la directiva del mismo origen, de forma que permita algunas solicitudes entre orígenes y rechace otras.

## <a name="understand-and-identify-cors-issues"></a>Comprender e identificar problemas de CORS

Dos direcciones URL tienen el mismo origen si tienen esquemas, hosts y puertos ([RFC 6454](https://tools.ietf.org/html/rfc6454)) idénticos, como:

-   http:\//contoso.com/foo.html
-   http:\//contoso.com/bar.html

Las siguientes direcciones URL tienen diferentes orígenes que las dos anteriores:

-   http:\//contoso.net - dominio diferente
-   http:\//contoso.com:9000/foo.html - puerto diferente
-   https:\//contoso.com/foo.html - esquema diferente
-   http:\//www.contoso.com/foo.html - subdominio diferente

La directiva de mismo origen impide que las aplicaciones accedan a recursos de otros orígenes, a menos que usen los encabezados de control de acceso correctos. Si los encabezados de CORS faltan o son incorrectos, no podrá realizar solicitudes entre orígenes. 

Puede identificar problemas de CORS mediante herramientas de depuración del explorador:

1. Inicie el explorador y vaya a la aplicación web.
1. Presione **F12** para que aparezca la consola de depuración.
1. Intente reproducir la transacción y revise el mensaje de la consola. Una infracción de CORS genera un error de la consola acerca del origen.

En la siguiente captura de pantalla, seleccionar el botón **Pruébelo** provocó un mensaje de error de CORS que https:\//corswebclient-contoso.msappproxy.net no se encontró en el encabezado de Access-Control-Allow-Origin.

![Problema de CORS](./media/application-proxy-understand-cors-issues/image3.png)

## <a name="cors-challenges-with-application-proxy"></a>Desafíos de CORS con Application Proxy

En el ejemplo siguiente se ilustra el típico escenario de CORS en Azure AD Application Proxy. El servidor interno hospeda un controlador de API web **CORSWebService** y un **CORSWebClient** que llama a **CORSWebService**. Hay una solicitud AJAX de **CORSWebClient** a **CORSWebService**.

![Solicitud del mismo origen local](./media/application-proxy-understand-cors-issues/image1.png)

La aplicación CORSWebClient funciona cuando se hospeda de forma local, pero no se carga o genera errores cuando se publica a través de Azure AD Application Proxy. Si publicó las aplicaciones CORSWebClient y CORSWebService por separado como aplicaciones diferentes a través de Application Proxy, las dos aplicaciones se hospedan en dominios diferentes. Una solicitud AJAX de CORSWebClient a CORSWebService es una solicitud entre orígenes y no se produce.

![Solicitud CORS de Application Proxy](./media/application-proxy-understand-cors-issues/image2.png)

## <a name="solutions-for-application-proxy-cors-issues"></a>Soluciones para problemas de CORS en Application Proxy

Puede resolver el problema anterior de CORS de varias maneras.

### <a name="option-1-set-up-a-custom-domain"></a>Opción 1: Configurar un dominio personalizado

Use un [dominio personalizado](./application-proxy-configure-custom-domain.md) de Azure AD Application Proxy para publicar desde el mismo origen, sin tener que realizar cambios en los orígenes de aplicación, el código ni los encabezados. 

### <a name="option-2-publish-the-parent-directory"></a>Opción 2: Publicar en el directorio primario

Publique el directorio primario de ambas aplicaciones. Esta solución funciona especialmente bien si tiene solo dos aplicaciones en el servidor web. En lugar de publicar cada aplicación por separado, puede publicar el directorio primario común, lo que genera el mismo origen.

En los ejemplos siguientes se muestra la página de Azure AD Application Proxy del portal para la aplicación CORSWebClient.  Cuando la **Dirección URL interna** está establecida en *contoso.com/CORSWebClient*, la aplicación no puede hacer solicitudes correctas al directorio *contoso.com/CORSWebService*, ya que son entre orígenes. 

![Publicar la aplicación de forma individual](./media/application-proxy-understand-cors-issues/image4.png)

En su lugar, establezca la **Dirección URL interna** para publicar el directorio primario, que incluye tanto el directorio *CORSWebClient* como el *CORSWebService*:

![Publicar el directorio primario](./media/application-proxy-understand-cors-issues/image5.png)

Las direcciones URL de aplicación resultantes resuelvan con eficacia el problema de CORS:

- https:\//corswebclient-contoso.msappproxy.net/CORSWebService
- https:\//corswebclient-contoso.msappproxy.net/CORSWebClient

### <a name="option-3-update-http-headers"></a>Opción 3: Actualizar los encabezados HTTP

Agregue un encabezado de respuesta HTTP personalizado en el servicio web para que coincida con la solicitud de origen. Para sitios web que se ejecutan en Internet Information Services (IIS), use el Administrador de IIS para modificar el encabezado:

![Agregar un encabezado de respuesta personalizado en el Administrador de IIS](./media/application-proxy-understand-cors-issues/image6.png)

Esta modificación no requiere ningún cambio de código. Puede verificarlo en los seguimientos de Fiddler:

**Publicar la adición de encabezado**\
HTTP/1.1 200 OK\
Cache-Control: no-cache\
Pragma: no-cache\
Content-Type: text/plain; charset=utf-8\
Expires: -1\
Vary: Accept-Encoding\
Servidor:  Microsoft-IIS/8.5 Microsoft-HTTPAPI/2.0\
**Access-Control-Allow-Origin: https\://corswebclient-contoso.msappproxy.net**\
X-AspNet-Version: 4.0.30319\
X-Powered-By: ASP.NET\
Content-Length: 17

### <a name="option-4-modify-the-app"></a>Opción 4: Modificar la aplicación

Puede cambiar la aplicación para admitir CORS agregando el encabezado Access-Control-Allow-Origin, con los valores adecuados. La manera de agregar el encabezado depende del lenguaje de programación de la aplicación. Cambiar el código es la opción menos recomendada, porque requiere más esfuerzo.

### <a name="option-5-extend-the-lifetime-of-the-access-token"></a>Opción 5: Ampliar la vigencia del token de acceso

Algunos problemas de CORS no se pueden resolver, por ejemplo, cuando la aplicación se redirige a *login.microsoftonline.com* para autenticarse y expira el token de acceso. Luego, la llamada de CORS produce un error. Una solución alternativa para este escenario es ampliar la duración del token de acceso, para evitar que expire durante la sesión de un usuario. Para más información sobre cómo hacerlo, consulte [Vigencia de tokens configurable en Azure AD](../develop/active-directory-configurable-token-lifetimes.md).

## <a name="see-also"></a>Consulte también
- [Tutorial: Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory](../app-proxy/application-proxy-add-on-premises-application.md) 
- [Planear una implementación de Azure AD Application Proxy](application-proxy-deployment-plan.md) 
- [Acceso remoto a aplicaciones locales mediante Azure Active Directory Application Proxy](application-proxy.md)