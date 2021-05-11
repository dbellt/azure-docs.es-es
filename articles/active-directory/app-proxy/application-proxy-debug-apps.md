---
title: 'Depuración de las aplicaciones de Application Proxy: Azure Active Directory'
description: Depure los problemas con las aplicaciones de Application Proxy de Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: troubleshooting
ms.date: 04/27/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 058ff3bc61794f74d2068bd99b93df007495bedf
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108187030"
---
# <a name="debug-application-proxy-application-issues"></a>Depuración de problemas con las aplicaciones de Application Proxy 

Este artículo lo ayudará a solucionar problemas con las aplicaciones de Application Proxy de Azure Active Directory (Azure°AD). Si usa el servicio Application Proxy para acceder de manera remota a una aplicación web local pero tiene problemas para conectarse a la aplicación, use este diagrama de flujo para depurar los problemas con una aplicación. 

## <a name="before-you-begin"></a>Antes de empezar

Cuando solucione problemas con Application Proxy, se recomienda que empiece con los conectores. En primer lugar, siga el flujo de solución de problemas que aparece en [Depuración de problemas de los conectores de Application Proxy](application-proxy-debug-connectors.md) para asegurarse de que los conectores de Application Proxy estén correctamente configurados. Si sigue con problemas, vuelva a este artículo para solucionar problemas con una aplicación.  

Para más información sobre Application Proxy, consulte:

- El artículo sobre el [acceso remoto a aplicaciones locales a través de Application Proxy](application-proxy.md).
- [Conectores de Application Proxy](application-proxy-connectors.md).
- [Instalación y registro de un conector](application-proxy-add-on-premises-application.md).
- [Solución de problemas y mensajes de error de Proxy de aplicación](application-proxy-troubleshoot.md)

## <a name="flowchart-for-application-issues"></a>Diagrama de flujo de los problemas de aplicaciones

Este diagrama de flujo lo guía a través de los pasos para depurar algunos de los problemas más comunes para conectarse a la aplicación. Para más información sobre cada paso, consulte la tabla que aparece después del diagrama de flujo.

![Diagrama de flujo que muestra los pasos para depurar una aplicación](media/application-proxy-debug-apps/application-proxy-apps-debugging-flowchart.png)

| Paso | Acción | Descripción |
|---------|---------|---------|
|1 | Abra un explorador, acceda a la aplicación y escriba sus credenciales | Trate de usar sus credenciales para iniciar sesión en la aplicación y revise por si hay algún error relacionado con el usuario, como [This corporate app can't be accessed](application-proxy-sign-in-bad-gateway-timeout-error.md) ("No se puede acceder a esta aplicación empresarial"). |
|2 | Compruebe la asignación de un usuario a la aplicación | Asegúrese de que la cuenta de usuario tiene permiso para acceder a la aplicación desde dentro de la red corporativa y, luego, intente iniciar sesión en la aplicación siguiendo los pasos que aparecen en [Prueba de la aplicación](application-proxy-add-on-premises-application.md#test-the-application). Si los problemas de inicio de sesión continúan, consulte [Solución de errores de inicio de sesión](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context).  |
|3 | Abra un explorador e intente acceder a la aplicación | Si se produce inmediatamente un error, revise si Application Proxy está configurado correctamente. Para información detallada sobre mensajes de error específicos, consulte [Solución de problemas y mensajes de error de Application Proxy](application-proxy-troubleshoot.md).  |
|4 | Compruebe la configuración del dominio personalizado o solucione el error | Si no aparece la página, asegúrese de que el dominio personalizado esté configurado correctamente. Para ello, revise [Uso de dominios personalizados](application-proxy-configure-custom-domain.md).<br></br>Si la página no se carga y aparece un mensaje de error, consulte [Solución de problemas y mensajes de error de Application Proxy](application-proxy-troubleshoot.md) para solucionar el error. <br></br>Si los mensajes de error tardan más de 20 segundos en aparecer, es porque podría haber un problema de conectividad. Vaya al artículo de solución de problemas [Depuración de los conectores de Application Proxy](application-proxy-debug-connectors.md).  |
|5 | Si siguen los problemas, vaya a la depuración de conectores | Es posible que haya un problema de conectividad entre el proxy y el conector o entre el conector y el back-end. Vaya al artículo de solución de problemas [Depuración de los conectores de Application Proxy](application-proxy-debug-connectors.md). |
|6 | Publique todos los recursos, revise todas las herramientas para desarrolladores del explorador y corrija los vínculos | Asegúrese de que la ruta de publicación incluye todas las imágenes, los scripts y las hojas de estilo necesarias para la aplicación. Consulte [Adición de una aplicación local a Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad) para información detallada. <br></br>Use las herramientas para desarrolladores del explorador (herramientas F12 en Internet Explorer o Microsoft Edge) y revise si hay problemas en la publicación, tal como se describe en [La página de aplicación no se muestra correctamente](application-proxy-page-appearance-broken-problem.md). <br></br>Revise las opciones para resolver vínculos rotos en [Los vínculos de la página no funcionan](application-proxy-page-links-broken-problem.md). |
|7 | Compruebe la latencia de la red | Si la página se carga lentamente, conozca algunas formas de minimizar la latencia de red en [Consideraciones para reducir la latencia](application-proxy-network-topology.md#considerations-for-reducing-latency). | 
|8 | Consulte más ayuda para solucionar problemas | Si los problemas continúan encuentre otros artículos para la solución de problemas en la [documentación para solucionar problemas de Application Proxy](application-proxy-troubleshoot.md). |

## <a name="next-steps"></a>Pasos siguientes


* [Publicación de aplicaciones en redes independientes y ubicaciones mediante grupos de conectores](application-proxy-connector-groups.md)
* [Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md)
* [Solución de problemas y mensajes de error de Proxy de aplicación](application-proxy-troubleshoot.md)
* [Cómo instalar de forma silenciosa el conector de Proxy de aplicación de Azure AD](application-proxy-register-connector-powershell.md)
