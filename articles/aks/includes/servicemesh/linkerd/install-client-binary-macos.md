---
author: paulbouwer
ms.topic: include
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 876e05d7b18ac193edbc9cf842ea2c1bf0555d54
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "77593750"
---
## <a name="download-and-install-the-linkerd-linkerd-client-binary"></a>Descarga e instalación del binario de cliente linkerd de Linkerd

En un shell basado en Bash en MacOS, use `curl` para descargar la versión de Linkerd como se indica a continuación:

```bash
# Specify the Linkerd version that will be leveraged throughout these instructions
LINKERD_VERSION=stable-2.6.0

curl -sLO "https://github.com/linkerd/linkerd2/releases/download/$LINKERD_VERSION/linkerd2-cli-$LINKERD_VERSION-darwin"
```

El binario de cliente `linkerd` se ejecuta en la máquina cliente y permite interactuar con la malla de servicio de Linkerd. Use los siguientes comandos para instalar el binario de cliente `linkerd` de Linkerd en un shell basado en Bash en MacOS. Estos comandos copian el binario de cliente `linkerd` en la ubicación del programa de usuario estándar en su `PATH`.

```bash
sudo cp ./linkerd2-cli-$LINKERD_VERSION-darwin /usr/local/bin/linkerd
sudo chmod +x /usr/local/bin/linkerd
```

Si desea completar la línea de comandos del binario de cliente `linkerd` de Linkerd, realice la siguiente configuración:

```bash
# Generate the bash completion file and source it in your current shell
mkdir -p ~/completions && linkerd completion bash > ~/completions/linkerd.bash
source ~/completions/linkerd.bash

# Source the bash completion file in your .bashrc so that the command-line completions
# are permanently available in your shell
echo "source ~/completions/linkerd.bash" >> ~/.bashrc
```
