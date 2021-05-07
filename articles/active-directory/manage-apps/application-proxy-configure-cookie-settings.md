---
title: 'Configuración de cookies para Application Proxy: Azure Active Directory | Microsoft Docs'
description: Azure Active Directory (Azure AD) cuenta con cookies de sesión y acceso para acceder a aplicaciones locales a través de Application Proxy. En este artículo, encontrará información sobre cómo usar y configurar los valores de las cookies.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 01/16/2019
ms.author: kenwith
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: 62afe97b44f45bc0b7aa12b33b6a65dd94ecf095
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108224569"
---
# <a name="cookie-settings-for-accessing-on-premises-applications-in-azure-active-directory"></a>Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory

Azure Active Directory (Azure AD) cuenta con cookies de sesión y acceso para acceder a aplicaciones locales a través de Application Proxy. Obtenga información sobre cómo usar la configuración de cookies de Application Proxy. 

## <a name="what-are-the-cookie-settings"></a>¿Qué son las configuraciones de cookies?

[Application Proxy](application-proxy.md) utiliza los siguientes valores de configuración para cookies de sesión y acceso.

| Configuración de cookies | Valor predeterminado | Descripción | Recomendaciones |
| -------------- | ------- | ----------- | --------------- |
| Usar cookie solo HTTP | **No** | **Sí** permite que Application Proxy incluya la marca HTTPOnly en los encabezados de respuesta HTTP. Esta marca proporciona ventajas de seguridad adicionales; por ejemplo, evita que el scripting del cliente (CSS) copie o modifique las cookies.<br></br><br></br>Antes de que se añadiera la compatibilidad con la configuración solo HTTP, Proxy de aplicación cifraba y transmitía las cookies a través de un canal TLS seguro para protegerlas contra las modificaciones. | Use **Sí** para obtener las ventajas de seguridad adicionales.<br></br><br></br>Use **No** para los agentes de usuario o cliente que necesiten acceder a la cookie de sesión. Por ejemplo, use **No** para un cliente RDP o MTSC que se conecta a un servidor de puerta de enlace de Escritorio remoto a través Application Proxy.|
| Usar cookies seguras | **No** | **Sí** permite que Application Proxy incluya la marca Secure en los encabezados de respuesta HTTP. Las cookies seguras mejoran la seguridad al transmitir las cookies a través de un canal protegido mediante TLS, como HTTPS. Esto impide que personas no autorizadas observen las cookies, ya que no se transmiten como texto no cifrado. | Use **Sí** para obtener las ventajas de seguridad adicionales.|
| Usar cookies persistentes | **No** | **Sí** permite que Application Proxy configure las cookies de acceso de forma que no expiren cuando se cierre el explorador web. La persistencia dura hasta que expire el token de acceso o hasta que el usuario elimine manualmente las cookies persistentes. | Use **No** debido al riesgo de seguridad asociado con mantener autenticados a los usuarios.<br></br><br></br>Se recomienda usar **Sí** únicamente para aplicaciones más antiguas que no pueden compartir cookies entre procesos. Es mejor actualizar la aplicación para controlar el uso compartido de cookies entre procesos, en lugar de usar las cookies persistentes. Por ejemplo, puede necesitar las cookies persistentes para permitir que un usuario abra documentos de Office en la vista de explorador desde un sitio de SharePoint. Sin cookies persistentes, esta operación puede producir un error si las cookies de acceso no se comparten entre el explorador, el proceso del explorador y el proceso de Office. |

## <a name="samesite-cookies"></a>Cookies SameSite
A partir de la versión Chrome 80 y, a la larga, en exploradores que use Chromium, las cookies que no especifican el atributo [SameSite](https://web.dev/samesite-cookies-explained) se tratarán como si estuvieran establecidas en **SameSite=Lax**. En el atributo SameSite se declara cómo se deben restringir las cookies a un contexto del mismo sitio. Cuando se establece en Lax, la cookie solo se envía a solicitudes del mismo sitio o a la navegación de nivel superior. Sin embargo, Application Proxy requiere que estas cookies se mantengan en el contexto de terceros para que los usuarios puedan iniciar sesión correctamente durante su sesión. Debido a esto, se realizan actualizaciones de las cookies de sesión y acceso de Application Proxy para evitar el impacto adverso de este cambio. Las actualizaciones incluyen:

* Establecer el atributo **SameSite** en **None**. Esto permite que las cookies de sesiones y acceso de Application Proxy se envíen correctamente en el contexto de terceros.
* Establecer la opción **Usar cookies seguras** en **Sí** como valor predeterminado. Chrome también requiere que las cookies especifiquen la marca Secure o se rechazará. Este cambio se aplicará a todas las aplicaciones existentes publicadas a través de Application Proxy. Observe que todas las cookies de acceso de Application Proxy siempre se han establecido en Secure y solo se transmiten a través de HTTPS. Este cambio solo se aplicará a las cookies de la sesión.

Estos cambios en las cookies de Application Proxy se implementarán durante el transcurso de las próximas semanas antes de la fecha de lanzamiento de Chrome 80.

Además si la aplicación back-end tiene cookies que deben estar disponibles en un contexto de terceros, debe participar explícitamente cambiando la aplicación para que use SameSite=None para estas cookies. Application Proxy convierte el encabezado Set-Cookie en sus direcciones URL y respetará la configuración de estas cookies que estableció la aplicación back-end.



## <a name="set-the-cookie-settings---azure-portal"></a>Configuración de los valores de cookies: Azure Portal
Para configurar los valores de las cookies en Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. Vaya a **Azure Active Directory** > **Aplicaciones empresariales** > **Todas las aplicaciones**.
3. Seleccione la aplicación para la que quiere habilitar las configuraciones de cookies.
4. Haga clic en **Application Proxy**.
5. En **Configuración adicional**, establezca la configuración de la cookie en **Sí** o **No**.
6. Haga clic en **Guardar** para aplicar los cambios. 

## <a name="view-current-cookie-settings---powershell"></a>Visualización de la configuración actual de cookies (PowerShell)

Para ver la configuración actual de cookies de la aplicación, use este comando de PowerShell:  

```powershell
Get-AzureADApplicationProxyApplication -ObjectId <ObjectId> | fl * 
```

## <a name="set-cookie-settings---powershell"></a>Establecimiento de la configuración de cookies (PowerShell)

En los siguientes comandos de PowerShell, ```<ObjectId>``` es el valor de ObjectId de la aplicación. 

**Cookie de solo HTTP** 

```powershell
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsHttpOnlyCookieEnabled $true 
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsHttpOnlyCookieEnabled $false 
```

**Cookie segura**

```powershell
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsSecureCookieEnabled $true 
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsSecureCookieEnabled $false 
```

**Cookies persistentes**

```powershell
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsPersistentCookieEnabled $true 
Set-AzureADApplicationProxyApplication -ObjectId <ObjectId> -IsPersistentCookieEnabled $false 
```
