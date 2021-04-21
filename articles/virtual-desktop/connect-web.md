---
title: 'Conexión a Windows Virtual Desktop con el cliente web: Azure'
description: Cómo conectarse a Windows Virtual Desktop mediante el cliente web.
author: Heidilohr
ms.topic: how-to
ms.date: 09/24/2019
ms.author: helohr
manager: femila
ms.openlocfilehash: 80a090abee45f9cb3ec6ee5406aad6abf1d73a59
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107725015"
---
# <a name="connect-to-windows-virtual-desktop-with-the-web-client"></a>Conexión a Windows Virtual Desktop con el cliente web

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/connect-web-2019.md).

El cliente web le permite acceder a los recursos de Windows Virtual Desktop desde un explorador web sin el largo proceso de instalación.

>[!NOTE]
>El cliente web no tiene compatibilidad con SO móvil actualmente.

## <a name="supported-operating-systems-and-browsers"></a>Exploradores y sistemas operativos compatibles

Aunque debería funcionar cualquier explorador compatible con HTML5, oficialmente se admiten los siguientes sistemas operativos y exploradores.

| Browser           | SO admitido                     | Notas               |
|-------------------|----------------------------------|---------------------|
| Microsoft Edge    | Windows                          |                     |
| Internet Explorer | Windows                          | Versión 11 o posterior |
| Apple Safari      | macOS                            |                     |
| Mozilla Firefox   | Windows, macOS, Linux            | Versión 55 o posterior |
| Google Chrome     | Windows, macOS, Linux, Chrome OS |                     |

## <a name="access-remote-resources-feed"></a>Acceso remoto a la fuente de recursos

En un explorador, vaya a la versión integrada de Windows Virtual Desktop del cliente web de Azure Resource Manager, en <https://rdweb.wvd.microsoft.com/arm/webclient>, e inicie sesión con su cuenta de usuario.

>[!NOTE]
>En cambio, si usa Windows Virtual Desktop (clásico) sin integración con Azure Resource Manager, conecte los recursos a <https://rdweb.wvd.microsoft.com/webclient>.
>
> Si utiliza el portal de US Gov, use <https://rdweb.wvd.azure.us/arm/webclient/index.html>.

>[!NOTE]
>Si ya ha iniciado sesión con una cuenta de Azure Active Directory diferente a la que quiere usar para Windows Virtual Desktop, debe cerrar la sesión o usar una ventana privada del explorador.

Después de iniciar sesión, debería ver una lista de recursos. Puede iniciar los recursos seleccionándolos de la misma forma en que lo haría con una aplicación normal en la pestaña **Todos los recursos**.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo usar el cliente web, consulte [Introducción al cliente web](/windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client).
