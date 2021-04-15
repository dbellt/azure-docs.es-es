---
author: phillipgibson
ms.topic: include
ms.date: 03/15/2021
ms.author: phillipgibson
ms.openlocfilehash: 1c7dbe9bb6cf28f4dee2c4e9aa036410fadbbddf
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106079953"
---
## <a name="download-and-install-the-osm-client-binary"></a>Descarga e instalación del binario de cliente de OSM

En un shell basado en Bash, utilice `curl` para descargar la versión de OSM y, a continuación, realice la extracción con `tar`, tal como se indica a continuación:

```bash
# Specify the OSM version that will be leveraged throughout these instructions
OSM_VERSION=v0.8.2

curl -sL "https://github.com/openservicemesh/osm/releases/download/$OSM_VERSION/osm-$OSM_VERSION-darwin-amd64.tar.gz" | tar -vxzf -
```

El binario de cliente `osm` se ejecuta en la máquina cliente y permite administrar el controlador de OSM en el clúster de AKS. Use los siguientes comandos para instalar el binario de cliente `osm` de OSM en un shell basado en Bash en macOS. Estos comandos copian el binario de cliente `osm` en la ubicación del programa de usuario estándar en su `PATH`.

```bash
sudo mv ./darwin-amd64/osm /usr/local/bin/osm
sudo chmod +x /usr/local/bin/osm
```

Puede comprobar que la biblioteca cliente `osm` se ha agregado correctamente a la ruta de acceso y su número de versión con el siguiente comando.

```
osm version
```
