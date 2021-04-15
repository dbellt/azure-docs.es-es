---
title: Requisitos del sistema
description: Enumera los requisitos del sistema para Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 02/03/2020
ms.topic: article
ms.custom: references_regions
ms.openlocfilehash: 789233ce1ede751276f965143716694c6feca3ca
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105032806"
---
# <a name="system-requirements"></a>Requisitos del sistema

En este capítulo se enumeran los requisitos mínimos del sistema para trabajar con *Azure Remote Rendering* (ARR).

## <a name="development-pc"></a>PC de desarrollo

* Windows 10 versión 1903 o posterior.
* Controladores gráficos actualizados.
* Opcional: [Descodificador de vídeo de hardware de H265](https://www.microsoft.com/p/hevc-video-extensions/9nmzlz57r3t7), si desea usar la versión preliminar local del contenido representado de forma remota (por ejemplo, en Unity).

> [!IMPORTANT]
> Windows Update no siempre entrega los controladores de GPU más recientes; consulte el sitio web del fabricante de la GPU para obtenerlos:
>
> * [Controladores AMD](https://www.amd.com/en/support)
> * [Controladores Intel](https://www.intel.com/content/www/us/en/support/detect.html)
> * [Controladores NVIDIA](https://www.nvidia.com/Download/index.aspx)

En la tabla siguiente se enumeran las GPU que admiten la descodificación de vídeo de hardware H265.

| Fabricante de la GPU | Modelos admitidos |
|-----------|:-----------|
| NVIDIA | Compruebe la **matriz de compatibilidad de NVDEC** [en la parte inferior de esta página](https://developer.nvidia.com/video-encode-decode-gpu-support-matrix). La GPU necesita un sí (YES) en la columna **H.265 4:2:0 8 bits**. |
| AMD | GPU con al menos la versión 6 del [descodificador de vídeo unificado](https://en.wikipedia.org/wiki/Unified_Video_Decoder#UVD_6) de AMD. |
| Intel | Skylake y CPU más recientes |

Aunque es posible que esté instalado el códec H265 correcto, las propiedades de seguridad de los archivos DLL del códec pueden producir errores de inicialización del códec. En la [guía de solución de problemas](../resources/troubleshoot.md#h265-codec-not-available) se describen los pasos para resolver este problema. El problema de DLL solo puede producirse cuando se usa el servicio en una aplicación de escritorio, por ejemplo, en Unity.

## <a name="devices"></a>Dispositivos

Actualmente, Azure Remote Rendering solo admite **HoloLens 2** y el Escritorio de Windows como dispositivos de destino. Consulte la sección sobre las [limitaciones de la plataforma](../reference/limits.md#platform-limitations).

Es importante usar el códec HEVC más reciente, ya que las versiones más recientes tienen mejoras significativas en la latencia. Para comprobar qué versión está instalada en el dispositivo:

1. Inicie **Microsoft Store**.
1. Haga clic en el botón **"..."** en la esquina superior derecha.
1. Seleccione **Descargas y actualizaciones**.
1. Busque en la lista las **extensiones de vídeo HEVC del fabricante del dispositivo**. Si este elemento no aparece en actualizaciones, la versión más reciente ya está instalada.
1. Asegúrese de que el códec de la lista tiene al menos la versión **1.0.21821.0**.
1. Haga clic en el botón **Obtener actualizaciones** y espere a que se instalen.

## <a name="network"></a>Red

Una conexión de red estable y de baja latencia es fundamental para una buena experiencia del usuario.

Consulte el capítulo dedicado a los [requisitos de red](../reference/network-requirements.md).

Para solucionar problemas de red, consulte la [Guía de solución de problemas](../resources/troubleshoot.md#unstable-holograms).

### <a name="network-firewall"></a>Firewall de red

### <a name="sdk-version--0176"></a>Versión del SDK >= 0.1.76

Las máquinas virtuales de Remote Rendering usan direcciones IP compartidas desde los siguientes intervalos IP:

| Nombre             | Region         | Prefijo IP         |
|------------------|:---------------|:------------------|
| Este de Australia   | australiaeast  | 20.53.44.240/28   |
| Este de EE. UU.          | estado         | 20.62.129.224/28  |
| Este de EE. UU. 2        | eastus2        | 20.49.103.240/28  |
| Japón Oriental       | japaneast      | 20.191.165.112/28 |
| Norte de Europa     | northeurope    | 52.146.133.64/28  |
| Centro-sur de EE. UU. | southcentralus | 20.65.132.80/28   |
| Sudeste de Asia   | southeastasia  | 20.195.64.224/28  |
| Sur de Reino Unido         | uksouth        | 51.143.209.144/28 |
| Oeste de Europa      | westeurope     | 20.61.99.112/28   |
| Oeste de EE. UU. 2        | westus2        | 20.51.9.64/28     |

Asegúrese de que los firewalls (en el dispositivo, dentro de los enrutadores, etc.) no bloqueen estos intervalos IP ni los puertos siguientes:

| Port              | Protocolo  | Allow    |
|-------------------|---------- |----------|
| 49152-65534       | TCP / UDP | Saliente |

#### <a name="sdk-version--0176"></a>Versión del SDK < 0.1.76

Asegúrese de que los firewalls (en el dispositivo, dentro de los enrutadores, etc.) no bloqueen los puertos siguientes:

| Port              | Protocolo | Allow    | Description |
|-------------------|----------|----------|-------------|
| 50051             | TCP      | Saliente | Conexión inicial (Protocolo de enlace HTTP) |
| 8266              | UDP      | Saliente | Transferencia de datos |
| 5000, 5433, 8443  | TCP      | Saliente | Necesario para la [herramienta ArrInspector](../resources/tools/arr-inspector.md)|


## <a name="software"></a>Software

El software siguiente debe estar instalado:

* La versión más reciente de **Visual Studio 2019** [(descargar)](https://visualstudio.microsoft.com/vs/older-downloads/)
* [Visual Studio Tools para Mixed Reality](/windows/mixed-reality/install-the-tools). En concreto, las instalaciones de *carga de trabajo* siguientes son obligatorias:
  * **Desarrollo para el escritorio con C++**
  * **Desarrollo de Plataforma universal de Windows (UWP)**
* **Windows SDK 10.0.18362.0** [(descargar)](https://developer.microsoft.com/windows/downloads/windows-10-sdk)
* **GIT** [(descargar)](https://git-scm.com/downloads)
* Opcional: para ver la secuencia de vídeo desde el servidor en un equipo de escritorio, necesita las **extensiones de vídeo de HEVC** [(vínculo de Microsoft Store)](https://www.microsoft.com/p/hevc-video-extensions/9nmzlz57r3t7). Asegúrese de que se ha instalado la última versión comprobando si hay actualizaciones en el almacén.

## <a name="unity"></a>Unity

Para el desarrollo con Unity, instale una versión actual de Unity 2019.3 o 2019.4 LTS [(descarga)](https://unity3d.com/get-unity/download). Se recomienda usar el centro de Unity para administrar instalaciones.
Asegúrese de incluir los siguientes módulos en la instalación de Unity:
* **UWP**: compatibilidad con la compilación de la Plataforma universal de Windows
* **IL2CPP**: compatibilidad con la compilación de Windows (IL2CPP)

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Representación de un modelo con Unity](../quickstarts/render-model.md)