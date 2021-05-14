---
title: 'Depuración de los conectores de Application Proxy: Azure Active Directory'
description: Depure los problemas con los conectores de Application Proxy de Azure Active Directory (Azure AD).
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
ms.openlocfilehash: 730fba1b1a936a6277289b816412750764930469
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108187014"
---
# <a name="debug-application-proxy-connector-issues"></a>Depuración de problemas de los conectores de Application Proxy 

Este artículo lo ayudará a solucionar problemas con los conectores de Application Proxy de Azure Active Directory (Azure°AD). Si usa el servicio Application Proxy para acceder de manera remota a una aplicación web local pero tiene problemas para conectarse a la aplicación, use este diagrama de flujo para depurar los problemas con un conector. 

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se da por supuesto que tiene instalado el conector de Application proxy y que tiene problemas con él. Al solucionar problemas de Application Proxy, se recomienda empezar con este flujo de solución de problemas para determinar si los conectores de Application Proxy están configurados correctamente. Si sigue teniendo problemas para conectarse a la aplicación, siga el flujo de solución de problemas en el artículo sobre la [depuración de problemas de aplicación de Application Proxy](application-proxy-debug-apps.md).  


Para más información sobre Application Proxy y el uso de sus conectores, consulte:

- El artículo sobre el [acceso remoto a aplicaciones locales a través de Application Proxy](application-proxy.md).
- [Conectores de Application Proxy](application-proxy-connectors.md).
- [Instalación y registro de un conector](application-proxy-add-on-premises-application.md).
- [Solución de problemas y mensajes de error de Proxy de aplicación](application-proxy-troubleshoot.md)

## <a name="flowchart-for-connector-issues"></a>Diagrama de flujo para los problemas de conectores

Este diagrama de flujo lo guía a través de los pasos para depurar algunos de los problemas más comunes con los conectores. Para más información sobre cada paso, consulte la tabla que aparece después del diagrama de flujo.

![Diagrama de flujo que muestra los pasos para depurar un conector](media/application-proxy-debug-connectors/application-proxy-connector-debugging-flowchart.png)

| Paso | Acción | Descripción |
|---------|---------|---------|
|1 | Búsqueda del grupo de conectores asignado a la aplicación | Probablemente tiene un conector instalado en varios servidores, en cuyo caso los conectores se deben [asignar a grupos de conectores](application-proxy-connector-groups.md#assign-applications-to-your-connector-groups). Para más información acerca de los grupos de conectores, consulte [Publicación de aplicaciones en redes independientes y ubicaciones mediante grupos de conectores](application-proxy-connector-groups.md). |
|2 | Instalación del conector y asignación de un grupo | Si no tiene instalado un conector, consulte [Instalación y registro de un conector](application-proxy-add-on-premises-application.md#install-and-register-a-connector).<br></br> Si tiene problemas para instalar el conector, consulte [Problema al instalar el conector](application-proxy-connector-installation-problem.md).<br></br> Si el conector no está asignado a un grupo, consulte sobre la [asignación del conector a un grupo](application-proxy-connector-groups.md#create-connector-groups).<br></br>Si la aplicación no está asignada a un grupo de conectores, consulte sobre la [asignación de la aplicación a un grupo de conectores](application-proxy-connector-groups.md#assign-applications-to-your-connector-groups).|
|3 | Ejecución de una prueba de puertos en el servidor del conector | En el servidor del conector, ejecute una prueba de puertos con [telnet](/windows-server/administration/windows-commands/telnet) u otra herramienta para probar los puertos y compruebe si los puertos [443 y 80 están abiertos](application-proxy-add-on-premises-application.md#open-ports).|
|4 | Configuración de dominios y puertos | [Asegúrese de que los dominios y puertos estén configurados correctamente](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment) Para que un conector funcione como corresponde, hay determinados puertos que deben estar abiertos y direcciones URL a las que el servidor debe poder acceder. |
|5 | Comprobación del uso de un proxy de back-end | Compruebe si los conectores están usando servidores proxy de back-end o si los están omitiendo. Para detalles, consulte [Solución de problemas del proxy del conector y de conectividad del servicio](application-proxy-configure-connectors-with-proxy-servers.md#troubleshoot-connector-proxy-problems-and-service-connectivity-issues). |
|6 | Actualización del conector y del actualizador para usar el proxy de back-end | Si hay un proxy de back-end en uso, querrá asegurarse de que el conector use el mismo proxy. Para más información sobre la solución de problemas y la configuración de los conectores para que trabajen con servidores proxy, consulte [Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md). |
|7 | Carga de la dirección URL interna de la aplicación en el servidor del conector | En el servidor del conector, cargue la dirección URL interna de la aplicación. |
|8 | Comprobación de la conectividad de red interna | Hay un problema de conectividad en la red interna que este flujo de depuración no puede diagnosticar. Es necesario que se pueda acceder a la aplicación de manera interna para que los conectores funcionen. Puede habilitar y ver los registros de eventos de conector, tal como se describe en el artículo sobre los [conectores de Application Proxy](application-proxy-connectors.md#under-the-hood). |
|9 | Aumento del valor de tiempo de espera en back-end | En la **configuración adicional** de la aplicación, cambie la configuración de **Tiempo de espera de las aplicaciones de back-end** a **Long** (Largo). Consulte [Adición de una aplicación local a Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad). |
|10 | Si los problemas persisten, oriéntese a problemas de flujo específicos y revise los flujos de depuración de SSO y de la aplicación | Use el flujo de solución de problemas [Depuración de problemas de aplicación de Application Proxy](application-proxy-debug-apps.md). |

## <a name="next-steps"></a>Pasos siguientes


* [Publicación de aplicaciones en redes independientes y ubicaciones mediante grupos de conectores](application-proxy-connector-groups.md)
* [Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md)
* [Solución de problemas y mensajes de error de Proxy de aplicación](application-proxy-troubleshoot.md)
* [Cómo instalar de forma silenciosa el conector de Proxy de aplicación de Azure AD](application-proxy-register-connector-powershell.md)