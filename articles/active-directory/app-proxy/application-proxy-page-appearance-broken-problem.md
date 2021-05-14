---
title: La página de la aplicación no se muestra correctamente para la aplicación de Application Proxy
description: Instrucciones para cuando la página no se muestre correctamente en una aplicación de Application Proxy que se ha integrado con Azure Active Directory
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: troubleshooting
ms.date: 04/27/2021
ms.author: kenwith
ms.reviewer: asteen
ms.openlocfilehash: 96a6ddfe6bd6bd8cd742c5a005cb67126c9d7f5e
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108186854"
---
# <a name="application-page-does-not-display-correctly-for-an-application-proxy-application"></a>La página de aplicación no se muestra correctamente para una aplicación de proxy de aplicación

Este artículo lo ayuda a solucionar problemas con aplicaciones de proxy de aplicación de Azure Active Directory cuando va a la página y hay algo en ella que no parece correcto.

## <a name="overview"></a>Información general
Cuando se publica una aplicación de proxy de aplicación, solo son accesibles las páginas en la raíz al acceder a la aplicación. Si la página no se muestra correctamente, puede que falten algunos recursos de página en la dirección URL interna raíz utilizada para la aplicación. Para resolverlo, asegúrese de que ha publicado *todos* los recursos de la página como parte de la aplicación.

Para comprobar si la desaparición de recursos es el problema, abra la herramienta de seguimiento de red (como Fiddler o las herramientas de F12 en Internet Explorer o Microsoft Edge), cargue la página y busque errores 404. Eso indica las páginas que actualmente no se encuentran y que tiene que publicar.

Como ejemplo de este caso, suponga que ha publicado una aplicación de gastos mediante la dirección URL interna `http://myapps/expenses`, pero la aplicación utiliza la hoja de estilos `http://myapps/style.css`. En este caso, la hoja de estilos no está publicada en la aplicación, por lo que, cuando se carga, la aplicación de gastos genera un error 404 al intentar cargar style.css. En este ejemplo, el problema se resuelve publicando la aplicación con una dirección URL interna `http://myapp/`.

## <a name="problems-with-publishing-as-one-application"></a>Problemas con la publicación como aplicación

Si no es posible publicar todos los recursos en la misma aplicación, debe publicar varias aplicaciones y habilitar vínculos entre ellas.

Para ello, se recomienda utilizar la solución de [dominios personalizados](application-proxy-configure-custom-domain.md). Sin embargo, esta solución requiere que posea el certificado de su dominio y que las aplicaciones usen nombres de dominio completos (FQDN). Para ver otras opciones, consulte la [documentación sobre la solución de problemas de vínculos rotos](application-proxy-page-links-broken-problem.md).

## <a name="next-steps"></a>Pasos siguientes
[Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](../app-proxy/application-proxy-add-on-premises-application.md)
