---
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 09fa612e7e5c681da16bf19e94c626ae14a3b8a1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "77590719"
---
Al agregar discos de datos a una máquina virtual Linux, puede encontrar errores si un no existe un disco en LUN 0. Si va a agregar un disco manualmente mediante el comando `az vm disk attach -new` y especifica un LUN (`--lun`), en lugar de permitir que la plataforma Azure determine el LUN adecuado, asegúrese de que un disco existe o existirá en LUN 0. 

Considere el ejemplo siguiente que muestra un fragmento de código de salida de `lsscsi`:

```bash
[5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc 
[5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd 
```

Los dos discos de datos existen en LUN 0 y LUN 1 (la primera columna de la salida `lsscsi` detalla `[host:channel:target:lun]`). Ambos discos deben ser accesibles desde la VM. Si ha especificado manualmente el primer disco que se va a agregar en LUN 1 y el segundo disco en LUN 2, es posible que no vea los discos correctamente desde la máquina virtual.

> [!NOTE]
> El valor `host` de Azure es 5 en estos ejemplos, pero puede variar en función del tipo de almacenamiento que se seleccione.
> 
> 

Este comportamiento de disco no es un problema de Azure, sino la forma en la que el kernel de Linux cumple las especificaciones de SCSI. Cuando el kernel de Linux analiza el bus SCSI para los dispositivos conectados, un dispositivo debe encontrarse en LUN 0 para que el sistema siga analizando dispositivos adicionales. De esta forma:

* Revise la salida de `lsscsi` después de agregar un disco de datos para comprobar que tiene un disco en LUN 0.
* Si el disco no aparece correctamente en la máquina virtual, compruebe que existe el disco en LUN 0.

