---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 06/10/2021
ms.author: crtreasu
ms.openlocfilehash: 8ad55355307cfc8656b6f6bf9b074d90daf7e2e8
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112006911"
---
El siguiente paso es descargar el paquete de Azure Object Anchors para Unity.

# <a name="download-with-web-browser"></a>[Descarga con explorador web](#tab/unity-package-web-ui)

Busque [aquí](https://aka.ms/aoa/unity-sdk/package) el paquete de Azure Object Anchors para Unity (`com.microsoft.azure.object-anchors.runtime`). Seleccione la versión que desee y descargue el paquete con el botón **Download** (Descargar).

# <a name="download-with-npm"></a>[Descarga con NPM](#tab/unity-package-npm)

Este paso requiere que <a href="https://www.npmjs.com/get-npm" target="_blank">NPM</a> esté instalado y disponible.

Ejecute el siguiente comando, reemplazando `<version_number>` por la versión de Azure Object Anchors que desea descargar:

```bash
npm pack com.microsoft.azure.object-anchors.runtime@<version_number> --registry https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry/
```

> [!NOTE]
> Para enumerar las versiones disponibles del paquete de Azure Object Anchors, ejecute lo siguiente:
>
> ```bash
> npm view com.microsoft.azure.object-anchors.runtime --registry https://pkgs.dev.azure.com/aipmr/MixedReality-Unity-Packages/_packaging/Unity-packages/npm/registry/ versions
> ```

El paquete de Azure Object Anchors se descargará en la carpeta donde se ejecutó el comando.

# <a name="install-with-mixed-reality-feature-tool-beta"></a>[Instalación con la herramienta de características de Mixed Reality (versión preliminar)](#tab/unity-package-mixed-reality-feature-tool)

Continúe con el paso siguiente. Usará la <a a href="/windows/mixed-reality/develop/unity/welcome-to-mr-feature-tool" target="_blank">herramienta de características de Mixed Reality</a> en un paso posterior.

---