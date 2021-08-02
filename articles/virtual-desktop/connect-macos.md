---
title: 'Conexión a Azure Virtual Desktop desde macOS: Azure'
description: Procedimiento para conectarse a Azure Virtual Desktop mediante el cliente macOS.
author: Heidilohr
ms.topic: how-to
ms.date: 04/08/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: a77df2eabdb1ff5417ec0c3b7b8c6733f993a09b
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111756012"
---
# <a name="connect-to-azure-virtual-desktop-with-the-macos-client"></a>Conexión a Azure Virtual Desktop con el cliente macOS

> Se aplica a: macOS 10.12, o cualquier versión posterior

>[!IMPORTANT]
>Este contenido se aplica a Azure Virtual Desktop con objetos de Azure Resource Manager. Si usa Azure Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/connect-macos-2019.md).

Puede acceder a los recursos de Azure Virtual Desktop desde sus dispositivos macOS con nuestro cliente, que se puede descargar. En esta guía se le indicará cómo configurar el cliente.

## <a name="install-the-client"></a>Instalar el cliente

Para empezar, [descargue](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466?mt=12) e instale el cliente en su dispositivo macOS.

## <a name="subscribe-to-a-feed"></a>Suscripción a una fuente

Suscríbase a la fuente que le proporcionó el administrador para obtener la lista de recursos administrados que tiene a su disposición en el dispositivo macOS.

Para suscribirse a una fuente:

1. Seleccione **Agregar área de trabajo** en la página principal para conectarse al servicio y recuperar los recursos.
2. Escriba la dirección URL de la fuente. Puede ser una dirección URL o una dirección de correo electrónico:
   - Si eliges una dirección URL, usa la que te proporcionó el administrador. Normalmente, la dirección URL es <https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery>.
   - Para usar un correo electrónico, escribe tu dirección de correo: Esto indica al cliente que busque una dirección URL asociada a su dirección de correo electrónico si el administrador configuró el servidor de esa forma.
   - Para conectarse mediante el portal de US Gov, utilice <https://rdweb.wvd.azure.us/api/arm/feeddiscovery>.
3. Seleccione **Agregar**.
4. Inicie sesión con su cuenta de usuario cuando se le solicite.

Después de iniciar sesión, verás una lista de los recursos disponibles.

Una vez suscrito a una fuente, el contenido de la fuente se actualizará automáticamente de forma periódica. Pueden agregarse, cambiarse o quitarse recursos de acuerdo con los cambios hechos por el administrador.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el cliente macOS, consulte el documento [Introducción al cliente de macOS](/windows-server/remote/remote-desktop-services/clients/remote-desktop-mac/).
