---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/04/2020
ms.author: alkohli
ms.openlocfilehash: 2b01b4f11ac7777075848287626e021b89254acb
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108758307"
---
Conéctese a la máquina virtual Windows mediante el Protocolo de escritorio remoto (RDP) a través de la dirección IP que pasó durante la creación de la máquina virtual.

1. En su cliente, abra RDP. 
1. Vaya a **Inicio** y escriba **mstsc**.
1. En el panel **Conexión a Escritorio remoto**, escriba la dirección IP de la máquina virtual y las credenciales de acceso que usó en el archivo de parámetros de la plantilla de máquina virtual. A continuación, seleccione **Conectar**.

   ![Captura de pantalla del panel Conexión a Escritorio remoto para conectarse a través de RDP a la máquina virtual de Windows.](media/azure-stack-edge-gateway-connect-vm-windows/connect-vm-rdp-1.png)

   > [!NOTE]
   > Es posible que tenga que aprobar la conexión a una máquina que no sea de confianza. 

Ahora ha iniciado sesión en la máquina virtual que se ejecuta en el dispositivo. 
