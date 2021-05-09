---
title: Uso de SCP para mover archivos desde una máquina virtual y hacia allí
description: Traslado seguro de archivos a la máquina virtual Linux de Azure o desde ella mediante SCP y un par de claves de SSH.
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.workload: infrastructure
ms.topic: how-to
ms.date: 04/20/2021
ms.author: cynthn
ms.subservice: ''
ms.openlocfilehash: edfc44f79cff25486fde6326ac954fe5b575d846
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107816447"
---
# <a name="use-scp-to-move-files-to-and-from-a-linux-vm"></a>Uso de SCP para mover archivos desde una máquina virtual Linux y hacia allí 

Este artículo muestra cómo trasladar archivos desde su estación de trabajo hasta una máquina virtual Linux de Azure, o desde una máquina virtual Linux de Azure hasta la estación de trabajo, mediante Secure Copy (SCP). El traslado de archivos entre su estación de trabajo y una máquina virtual Linux, de forma rápida y segura, es una parte fundamental de la administración de la infraestructura de Azure. 

En este artículo, necesita una máquina virtual Linux implementada en Azure mediante [archivos de clave privada y pública de SSH](mac-create-ssh-keys.md). También necesita un cliente SCP para el equipo local. Se basa en SSH y se incluye en el shell de Bash predeterminado de la mayoría de los equipos Linux y Mac y en PowerShell.


## <a name="quick-commands"></a>Comandos rápidos

Copia de un archivo hacia arriba de la máquina virtual de Linux

```bash
scp file azureuser@azurehost:directory/targetfile
```

Copia de un archivo hacia abajo de la máquina virtual Linux

```bash
scp azureuser@azurehost:directory/file targetfile
```

## <a name="detailed-walkthrough"></a>Tutorial detallado

Como ejemplos, estamos trasladando un archivo de configuración de Azure a una máquina virtual Linux y extrayendo un directorio de archivos de registro, ambos con claves SCP y SSH.   

## <a name="ssh-key-pair-authentication"></a>Autenticación del par de claves de SSH

SCP usa SSH para el nivel de transporte. SSH administra la autenticación en el host de destino y traslada el archivo en un túnel cifrado proporcionado de forma predeterminada con SSH. Para la autenticación SSH, se pueden utilizar nombres de usuario y contraseñas. Sin embargo, se recomienda la autenticación clave pública y privada de SSH como procedimiento recomendado de seguridad. Una vez que SSH ha autenticado la conexión, SCP comienza a copiar el archivo. Con claves públicas y privadas de SSH y un `~/.ssh/config` configurado adecuadamente, la conexión de SCP puede establecerse usando únicamente un nombre del servidor (o dirección IP). Si solo tiene una clave SSH, SCP la busca en el directorio `~/.ssh/` y la usa de forma predeterminada para iniciar sesión en la máquina virtual.

Para más información acerca de cómo configurar las claves públicas y privadas de SSH y `~/.ssh/config`, consulte [Creación de claves SSH](mac-create-ssh-keys.md).

## <a name="scp-a-file-to-a-linux-vm"></a>SCP en un archivo para una máquina virtual Linux

Para el primer ejemplo, copiamos un archivo de configuración de Azure en una máquina virtual Linux que se usa para implementar la automatización. Como este archivo contiene las credenciales de la API de Azure, que incluyen secretos, la seguridad es importante. El túnel cifrado proporcionado por SSH protege el contenido del archivo.

El comando siguiente copia el archivo *.azure/config* local a una máquina virtual de Azure con el FQDN de *myserver.eastus.cloudapp.azure.com*. Si no tiene un [FQDN establecido,](../create-fqdn.md), también puede usar la dirección IP de la máquina virtual. El nombre de usuario administrador en la máquina virtual de Azure es *azureuser*. El archivo está dirigido al directorio */home/azureuser/* . Sustituya sus valores propios en este comando.

```bash
scp ~/.azure/config azureuser@myserver.eastus.cloudapp.com:/home/azureuser/config
```

## <a name="scp-a-directory-from-a-linux-vm"></a>SCP de un directorio desde una máquina virtual Linux

En este ejemplo, copiamos un directorio de archivos de registro de la máquina virtual Linux hasta la estación de trabajo. Un archivo de registro puede contener datos confidenciales o secretos, o no. Sin embargo, mediante SCP garantiza que el contenido de los archivos de registro está cifrado. El uso de SCP para transferir los archivos es la manera más fácil de obtener el directorio de registro y archivos hacia abajo en la estación de trabajo mientras se mantiene la seguridad.

El siguiente comando copia los archivos en el directorio */home/azureuser/logs/* en la máquina virtual de Azure al directorio /tmp local:

```bash
scp -r azureuser@myserver.eastus.cloudapp.com:/home/azureuser/logs/. /tmp/
```

La marca `-r` indica a SCP que copie de forma recursiva los archivos y directorios desde el punto del directorio enumerado en el comando.  Tenga también en cuenta que la sintaxis de línea de comandos es similar a un comando de copia de `cp`.

## <a name="next-steps"></a>Pasos siguientes

* [Administración de usuarios, SSH y comprobar o reparar discos en máquinas virtuales de Linux de Azure con la extensión VMAccess](../extensions/vmaccess.md?toc=/azure/virtual-machines/linux/toc.json)
