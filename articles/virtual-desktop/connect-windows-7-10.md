---
title: 'Conexión a Azure Virtual Desktop en Windows 10 o 7: Azure'
description: Cómo conectarse a Azure Virtual Desktop mediante el cliente de escritorio de Windows.
author: Heidilohr
ms.topic: how-to
ms.date: 09/22/2020
ms.author: helohr
manager: femila
ms.custom: template-how-to
ms.openlocfilehash: 32b858c4df24820dfb0f679a6da858e5807993c8
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111981663"
---
# <a name="connect-with-the-windows-desktop-client"></a>Conexión con el cliente de Escritorio de Windows

Para acceder a los recursos de Azure Virtual Desktop en dispositivos con Windows 10, Windows 10 IoT Enterprise y Windows 7, use el cliente de escritorio de Windows. 

> [!IMPORTANT]
> No es compatible con Window 8 ni Windows 8.1.
> 
> Solo admite objetos de Azure Resource Manager. Para admitir objetos sin Azure Resource Manager, consulte [Conexión con el cliente de escritorio de Windows (clásico)](./virtual-desktop-fall-2019/connect-windows-7-10-2019.md).
> 
> No es compatible con el cliente de Conexión de RemoteApp y Escritorio (RADC) ni con el cliente de Conexión a Escritorio remoto (MSTSC).

## <a name="install-the-windows-desktop-client"></a>Instalación del cliente de escritorio de Windows

Descargue el cliente en función de su versión de Windows:

- [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2068602)
- [Windows de 32 bits](https://go.microsoft.com/fwlink/?linkid=2098960)
- [Windows ARM64](https://go.microsoft.com/fwlink/?linkid=2098961)

Durante la instalación para determinar el acceso, seleccione una de estas opciones:

- **Instalar solo para usted**
- **Instalar para todos los usuarios de esta máquina** (requiere derechos de administrador)

Para iniciar el cliente después de la instalación, use el menú **Inicio** y busque **Escritorio remoto**.

## <a name="subscribe-to-a-workspace"></a>Suscripción a un área de trabajo

Para suscribirse a un área de trabajo, elija entre:

- Usar una cuenta profesional o educativa y hacer que el cliente detecte los recursos disponibles.
- Usar la dirección URL específica del recurso.

Para iniciar el recurso una vez suscrito, vaya al **Centro de conexiones** y haga doble clic en el recurso.

> [!TIP]
> Para iniciar un recurso desde el menú **Inicio**, puede buscar la carpeta con el nombre del área de trabajo o escribir el nombre del recurso en la barra de búsqueda.

### <a name="use-a-user-account"></a>Uso de una cuenta de usuario

1. Seleccione **Suscribirse** en la página principal.
1. Inicie sesión con su cuenta de usuario cuando se le solicite.

Los recursos agrupados por área de trabajo aparecerán en el **Centro de conexiones**.

   > [!NOTE]
   > De forma predeterminada, el cliente de Windows es Azure Virtual Desktop (clásico). 
   > 
   > Sin embargo, si el cliente detecta recursos de Azure Resource Manager adicionales, los agrega automáticamente o informa al usuario de que están disponibles.

### <a name="use-a-specific-url"></a>Uso de una dirección URL específica

1. Seleccione **Subscribe with URL** (Suscribirse con URL) en la página principal.
1. Escriba la *dirección URL del área de trabajo* o una *dirección de correo electrónico*:
   - Para la **URL del área de trabajo**, use la dirección URL proporcionada por el administrador.

   |Recursos disponibles|URL|
   |-|-|
   |Azure Virtual Desktop (clasico)|`https://rdweb.wvd.microsoft.com/api/feeddiscovery/webfeeddiscovery.aspx`|
   |Azure Virtual Desktop|`https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery`|
   |Azure Virtual Desktop (US Gov)|`https://rdweb.wvd.azure.us/api/arm/feeddiscovery`|
   
   - Para **Correo electrónico**, use su dirección de correo electrónico. 
      
   El cliente buscará la dirección URL asociada a su correo electrónico, siempre que el administrador haya habilitado la [detección de correo electrónico](/windows-server/remote/remote-desktop-services/rds-email-discovery).

1. Seleccione **Next** (Siguiente).
1. Inicie sesión con su cuenta de usuario cuando se le solicite.

Los recursos agrupados por área de trabajo aparecerán en el **Centro de conexiones**.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo usar el cliente, consulte [Introducción al cliente de escritorio de Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop/).

Si, como administrador, le interesa obtener más información sobre las características del cliente, consulte [Cliente de escritorio de Windows para administradores](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-admin).
