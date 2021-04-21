---
title: Reemplazar el punto de entrada en la instancia de contenedor
description: Establecer una línea de comandos para reemplazar el punto de entrada de una imagen de contenedor cuando se implementa una instancia de contenedor de Azure
ms.topic: article
ms.date: 04/15/2019
ms.openlocfilehash: 5898decbf4108d48bb9e84019d659075b18fd043
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771090"
---
# <a name="set-the-command-line-in-a-container-instance-to-override-the-default-command-line-operation"></a>Establecimiento de la línea de comandos en una instancia de contenedor para invalidar la operación de línea de comandos predeterminada

Cuando cree una instancia de contenedor, puede especificar una línea de comandos para invalidar la instrucción de la línea de comandos predeterminada incorporada a la imagen de contenedor. Este comportamiento es similar al argumento de línea de comandos `--entrypoint` para `docker run`.

Igual que configurar [variables de entorno](container-instances-environment-variables.md) de instancias de contenedor, especificar una línea de comandos de inicio es útil para los trabajos por lotes donde es necesario preparar cada contenedor de forma dinámica con la configuración específica de la tarea.

## <a name="command-line-guidelines"></a>Instrucciones de línea de comandos

* De forma predeterminada, la línea de comandos especifica un *único proceso que inicia sin un shell* en el contenedor. Por ejemplo, la línea de comandos puede ejecutar un archivo ejecutable o un script de Python. El proceso puede especificar parámetros o argumentos adicionales.

* Para ejecutar varios comandos, inicie la línea de comandos mediante el establecimiento de un entorno de shell compatible con el sistema operativo del contenedor. Ejemplos:

  |Sistema operativo  |Shell predeterminado  |
  |---------|---------|
  |Ubuntu     |   `/bin/bash`      |
  |Alpine     |   `/bin/sh`      |
  |Windows     |    `cmd`     |

  Siga las convenciones del shell para combinar varios comandos para que se ejecuten en secuencia.

* Según la configuración del contenedor, es posible que deba establecer una ruta de acceso completa al ejecutable o a los argumentos de la línea de comandos.

* Establezca una [directiva de reinicio](container-instances-restart-policy.md) adecuada para la instancia de contenedor, según si la línea de comandos especifica una tarea de ejecución prolongada o una tarea de una sola ejecución. Por ejemplo, una directiva de reinicio de `Never` o `OnFailure` se recomienda para una tarea de una sola ejecución. 

* Si necesita información sobre el punto de entrada predeterminado establecido en una imagen de contenedor, use el comando de [inspección de imagen de docker](https://docs.docker.com/engine/reference/commandline/image_inspect/).

## <a name="command-line-syntax"></a>Sintaxis de la línea de comandos

La sintaxis de la línea de comandos varía según la herramienta o la API de Azure usada para crear las instancias. Si especifica un entorno de shell, observe también las convenciones de sintaxis del comando del shell.

* Comando [az container create][az-container-create]: Pase una cadena con el parámetro `--command-line`. Ejemplo: `--command-line "python myscript.py arg1 arg2"`.

* Cmdlet [New-AzureRmContainerGroup][new-azurermcontainergroup] de Azure PowerShell: Pase una cadena con el parámetro `-Command`. Ejemplo: `-Command "echo hello"`.

* Azure Portal: En la propiedad **Invalidación de comando** de la configuración del contenedor, proporcione una lista de cadenas separadas por comas sin comillas. Ejemplo: `python, myscript.py, arg1, arg2`. 

* Plantilla de Resource Manager, archivo YAML o uno de los SDK de Azure: Especifique la propiedad de la línea de comandos como una matriz de cadenas. Ejemplo: la matriz JSON `["python", "myscript.py", "arg1", "arg2"]` de una plantilla de Resource Manager. 

  Si está familiarizado con la sintaxis de [Dockerfile](https://docs.docker.com/engine/reference/builder/), este formato es similar al formato de *exec* de la instrucción CMD.

### <a name="examples"></a>Ejemplos

|    |  Azure CLI   | Portal | Plantilla | 
| ---- | ---- | --- | --- |
| **Comando único** | `--command-line "python myscript.py arg1 arg2"` | **Invalidación del comando**: `python, myscript.py, arg1, arg2` | `"command": ["python", "myscript.py", "arg1", "arg2"]` |
| **Varios comandos** | `--command-line "/bin/bash -c 'mkdir test; touch test/myfile; tail -f /dev/null'"` |**Invalidación del comando**: `/bin/bash, -c, mkdir test; touch test/myfile; tail -f /dev/null` | `"command": ["/bin/bash", "-c", "mkdir test; touch test/myfile; tail -f /dev/null"]` |

## <a name="azure-cli-example"></a>Ejemplo de la CLI de Azure

Por ejemplo, modifique el comportamiento de la imagen de contenedor [microsoft/aci-wordcount][aci-wordcount], que analiza el texto de la obra *Hamlet* de Shakespeare para encontrar con las palabras que se repiten con más frecuencia. En lugar de analizar *Hamlet*, puede establecer una línea de comandos que apunte a un origen de texto diferente.

Para ver la salida del contenedor [microsoft/aci-wordcount][aci-wordcount], cuando analice el texto predeterminado, ejecútela con el siguiente comando [az container create][az-container-create]. No se especifica ninguna línea de comandos de inicio, por lo que se ejecuta el comando del contenedor predeterminado. Con fines ilustrativos, este ejemplo establece [variables de entorno](container-instances-environment-variables.md) para buscar las tres palabras principales que tienen como mínimo cinco letras:

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer1 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --environment-variables NumWords=3 MinLength=5 \
    --restart-policy OnFailure
```

Una vez que el estado del contenedor se muestre como *Finalizado* (use [az container show][az-container-show] para comprobar el estado), muestre el registro con [az container logs][az-container-logs] para ver la salida.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer1
```

Salida:

```console
[('HAMLET', 386), ('HORATIO', 127), ('CLAUDIUS', 120)]
```

Ahora configure un segundo contenedor de ejemplo para analizar texto diferente mediante la especificación de una línea de comandos diferente. El script de Python ejecutado por el contenedor, *wordcount.py*, acepta una dirección URL como argumento y procesa el contenido de esa página en lugar del valor predeterminado.

Por ejemplo, para determinar las tres palabras principales que tienen cinco letras como mínimo en *Romeo y Julieta*:

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer2 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables NumWords=3 MinLength=5 \
    --command-line "python wordcount.py http://shakespeare.mit.edu/romeo_juliet/full.html"
```

Una vez más, cuando el contenedor muestra el estado *Terminado*, puede ver la salida en los registros del contenedor:

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer2
```

Salida:

```console
[('ROMEO', 177), ('JULIET', 134), ('CAPULET', 119)]
```

## <a name="next-steps"></a>Pasos siguientes

Los escenarios basados en tareas, como el procesamiento por lotes de un gran conjunto de datos con varios contenedores, pueden aprovechar líneas de comandos personalizadas en tiempo de ejecución. Para más información sobre cómo ejecutar contenedores basados en tareas, consulte [Ejecución de tareas en contenedores con directivas de reinicio](container-instances-restart-policy.md).

<!-- LINKS - External -->
[aci-wordcount]: https://hub.docker.com/_/microsoft-azuredocs-aci-wordcount

<!-- LINKS Internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-container-logs]: /cli/azure/container#az_container_logs
[az-container-show]: /cli/azure/container#az_container_show
[new-azurermcontainergroup]: /powershell/module/azurerm.containerinstance/new-azurermcontainergroup
[portal]: https://portal.azure.com
