---
title: Ejecución de comandos en una instancia de contenedor en ejecución
description: Obtenga información sobre cómo ejecutar un comando en un contenedor actualmente en ejecución en Azure Container Instances
ms.topic: article
ms.date: 03/30/2018
ms.openlocfilehash: 42832910efff67f111c669793798d9ff0e413536
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790786"
---
# <a name="execute-a-command-in-a-running-azure-container-instance"></a>Ejecución de un comando en una instancia de contenedor de Azure en ejecución

Azure Container Instances permite ejecutar un comando en un contenedor en ejecución. Ejecutar un comando en un contenedor que ya inició resulta especialmente útil durante el desarrollo y la solución de problemas de las aplicaciones. El uso más común de esta característica es para iniciar un shell interactivo a fin de que pueda depurar problemas en un contenedor en ejecución.

## <a name="run-a-command-with-azure-cli"></a>Ejecución de un comando con la CLI de Azure

Ejecute un comando en un contenedor en ejecución con [az container exec][az-container-exec] en la [CLI de Azure][azure-cli]:

```azurecli
az container exec --resource-group <group-name> --name <container-group-name> --exec-command "<command>"
```

Por ejemplo, para iniciar un shell de Bash en un contenedor de Nginx:

```azurecli
az container exec --resource-group myResourceGroup --name mynginx --exec-command "/bin/bash"
```

En la salida de ejemplo siguiente, el shell de Bash se inicia en un contenedor Linux en ejecución, lo que proporciona un terminal en el que se ejecuta `ls`:

```output
root@caas-83e6c883014b427f9b277a2bba3b7b5f-708716530-2qv47:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@caas-83e6c883014b427f9b277a2bba3b7b5f-708716530-2qv47:/# exit
exit
Bye.
```

En este ejemplo, el símbolo del sistema se inicia en un contenedor Nanoserver en ejecución:

```azurecli
az container exec --resource-group myResourceGroup --name myiis --exec-command "cmd.exe"
```

```output
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\>dir
 Volume in drive C has no label.
 Volume Serial Number is 76E0-C852

 Directory of C:\

03/23/2018  09:13 PM    <DIR>          inetpub
11/20/2016  11:32 AM             1,894 License.txt
03/23/2018  09:13 PM    <DIR>          Program Files
07/16/2016  12:09 PM    <DIR>          Program Files (x86)
03/13/2018  08:50 PM           171,616 ServiceMonitor.exe
03/23/2018  09:13 PM    <DIR>          Users
03/23/2018  09:12 PM    <DIR>          var
03/23/2018  09:22 PM    <DIR>          Windows
               2 File(s)        173,510 bytes
               6 Dir(s)  21,171,609,600 bytes free

C:\>exit
Bye.
```

## <a name="multi-container-groups"></a>Grupos de varios contenedores

Si el [grupo de contenedores](container-instances-container-groups.md) tiene varios contenedores, como un contenedor de aplicación y un sidecar de registro, especifique el nombre del contenedor en el cual ejecutar el comando con `--container-name`.

Por ejemplo, en el grupo de contenedores *mynginx* hay dos contenedores, *nginx-app* y *logger*. Para iniciar un shell en el contenedor *nginx-app*:

```azurecli
az container exec --resource-group myResourceGroup --name mynginx --container-name nginx-app --exec-command "/bin/bash"
```

## <a name="restrictions"></a>Restricciones

Azure Container Instances actualmente permite iniciar un solo proceso con [az container exec][az-container-exec] y no es posible pasar argumentos de comando. Por ejemplo, no puede encadenar comandos como en `sh -c "echo FOO && echo BAR"` ni ejecutar `echo FOO`.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre otras herramientas para la solución de problemas y otros problemas comunes de implementación en [Solución de problemas de contenedor e implementación en Azure Container Instances](container-instances-troubleshooting.md).

<!-- LINKS - internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-container-exec]: /cli/azure/container#az_container_exec
[azure-cli]: /cli/azure