---
title: Recopilación de un seguimiento de red
description: Aprenda a obtener el seguimiento de red como ayuda para solucionar problemas.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: how-to
ms.date: 03/22/2021
ms.openlocfilehash: 6e92ab3c27113aad44b7bed2815b9c5050afd0ac
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167092"
---
# <a name="how-to-collect-a-network-trace"></a>Recopilación de un seguimiento de red

Si se encuentra con un problema, un seguimiento de red a veces puede proporcionar mucha información útil. En esta guía paso a paso se muestran las opciones disponibles para recopilar un seguimiento de red.

> [!WARNING]
> Un seguimiento de red incluye todo el contenido de cada mensaje que envía la aplicación. **Nunca** publique seguimientos de red sin procesar de las aplicaciones de producción en foros públicos como GitHub.

## <a name="collect-a-network-trace-with-fiddler"></a>Recopilar un seguimiento de red con Fiddler

Fiddler es una herramienta eficaz para recopilar seguimientos HTTP. Instálelo desde [telerik.com/fiddler](https://www.telerik.com/fiddler), inícielo y, luego, ejecute la aplicación y reproduzca el problema. Fiddler está disponible para Windows, macOS y Linux. 

Si se conecta mediante HTTPS, hay algunos pasos adicionales necesarios para asegurarse de que Fiddler pueda descifrar el tráfico HTTPS. Para más información, consulte la [documentación de Fiddler](https://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/DecryptHTTPS).

Una vez que recopile el seguimiento, si desea exportarlo, elija **Archivo** > **Guardar** > **Todas las sesiones** en la barra de menús.

## <a name="collect-a-network-trace-with-tcpdump-macos-and-linux-only"></a>Recopilación de un seguimiento de red con tcpdump (solo para macOS y Linux)

Este método funciona para todas las aplicaciones.

Para recopilar seguimientos TCP sin procesar mediante tcpdump, ejecute el comando siguiente desde un shell de comandos. Si recibe un error de permisos, es posible que tenga que ser `root` o escribir `sudo` como prefijo del comando:

```console
tcpdump -i [interface] -w trace.pcap
```

Reemplace `[interface]` por la interfaz de red en la que desea realizar la captura. Por lo general, es algo parecido a `/dev/eth0` (para la interfaz Ethernet estándar) o a `/dev/lo0` (para el tráfico de localhost). Para más información, consulte la página man `tcpdump` en el sistema host.

```console
man tcpdump
```

## <a name="collect-a-network-trace-in-the-browser-browser-based-apps-only"></a>Recopilación de un seguimiento de red en el explorador (solo para aplicaciones basadas en el explorador)

Las Herramientas de desarrollo de la mayoría de los exploradores tienen una pestaña "Red" que permite capturar la actividad de red entre el explorador y el servidor. 

### <a name="microsoft-edge-chromium"></a>Microsoft Edge (Chromium)

1. Abra [DevTools](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/)
    * Seleccione `F12` 
    * Seleccione `Ctrl`+`Shift`+`I` \(Windows/Linux\) o `Command`+`Option`+`I` \(macOS\)
    * Seleccione `Settings and more` y, luego, `More Tools > Developer Tools`  
1. Seleccione la pestaña `Network`
1. Si es necesario, actualice la página y reproduzca el problema
1. Seleccione `Export HAR...` en la barra de herramientas para exportar el seguimiento como archivo "HAR"

    :::image type="content" source="./media/howto-troubleshoot-network-trace/edge-export-network-trace.png" alt-text="Recopilación del seguimiento de red con Microsoft Edge":::

### <a name="google-chrome"></a>Google Chrome

1. Abra [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools)
    * Seleccione `F12` 
    * Seleccione `Ctrl`+`Shift`+`I` \(Windows/Linux\) o `Command`+`Option`+`I` \(macOS\)  
    * Seleccione `Customize and control Google Chrome` y, luego, `More Tools > Developer Tools`
1. Seleccione la pestaña `Network`
1. Si es necesario, actualice la página y reproduzca el problema
1. Seleccione `Export HAR...` en la barra de herramientas para exportar el seguimiento como archivo "HAR"

    :::image type="content" source="./media/howto-troubleshoot-network-trace/chrome-export-network-trace.png" alt-text="Recopilación del seguimiento de red con Google Chrome":::

### <a name="mozilla-firefox"></a>Mozilla Firefox

1. Abra las [Herramientas de desarrollo de Firefox](https://developer.mozilla.org/en-US/docs/Tools)
    * Seleccione `F12`
    * Seleccione `Ctrl`+`Shift`+`I` \(Windows/Linux\) o `Command`+`Option`+`I` \(macOS\) 
    * Seleccione `Open menu` y, luego, `Web Developer > Toggle Tools`
1. Seleccione la pestaña `Network`
1. Si es necesario, actualice la página y reproduzca el problema
1. Haga clic con el botón derecho en cualquier parte de la lista de solicitudes y elija "Guardar todo como HAR"

    :::image type="content" source="./media/howto-troubleshoot-network-trace/firefox-export-network-trace.png" alt-text="Recopilación del seguimiento de red con Mozilla Firefox":::

### <a name="safari"></a>Safari

1. Abra las [Herramientas de desarrollo web](https://developer.apple.com/safari/tools/)
    * Seleccione `Command`+`Option`+`I`
    * Seleccione el menú `Developer` y, luego, seleccione `Show Web Inspector` 
1. Seleccione la pestaña `Network`
1. Si es necesario, actualice la página y reproduzca el problema
1. Haga clic con el botón derecho en cualquier parte de la lista de solicitudes y elija "Guardar todo como HAR"