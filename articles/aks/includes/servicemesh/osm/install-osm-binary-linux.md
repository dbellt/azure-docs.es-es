---
author: phillipgibson
ms.topic: include
ms.date: 03/15/2021
ms.author: phillipgibson
ms.openlocfilehash: 6a4871b4aca10dddae46ed8f732090d9a258b3d6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468125"
---
## <a name="download-and-install-the-osm-client-binary"></a>Descarga e instalación del binario de cliente de OSM

En un shell basado en Bash en Linux o en el [Subsistema de Windows para Linux][install-wsl], utilice `curl` para descargar la versión de OSM y, a continuación, realice la extracción con `tar`, tal como se indica a continuación:

```bash
# Specify the OSM version that will be leveraged throughout these instructions
OSM_VERSION=v0.8.4

curl -sL "https://github.com/openservicemesh/osm/releases/download/$OSM_VERSION/osm-$OSM_VERSION-linux-amd64.tar.gz" | tar -vxzf -
```

El binario de cliente `osm` se ejecuta en la máquina cliente y permite administrar el controlador de OSM en el clúster de AKS. Use los comandos siguientes para instalar el binario de cliente `osm` de OSM en un shell basado en Bash en Linux o en el [Subsistema de Windows para Linux][install-wsl]. Estos comandos copian el binario de cliente `osm` en la ubicación del programa de usuario estándar en su `PATH`.

```bash
sudo mv ./linux-amd64/osm /usr/local/bin/osm
sudo chmod +x /usr/local/bin/osm
```

Puede comprobar que la biblioteca cliente `osm` se ha agregado correctamente a la ruta de acceso y su número de versión con el siguiente comando.

```
osm version
```

<!-- LINKS - external -->

[install-wsl]: /windows/wsl/install-win10
