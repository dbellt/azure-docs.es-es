---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 12/06/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 4732abacc80a24d268f0e5cb7d100079e5a8d90b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105104412"
---
Cree una Conexión a Escritorio remoto para conectarse a una máquina virtual implementada en la red virtual. La mejor manera de verificar que puede conectarse a la máquina virtual es hacerlo mediante su dirección IP privada, en lugar del nombre de equipo. Con este método prueba si puede conectarse, no si la resolución de nombres está configurada correctamente. 

1. Busque la dirección IP privada de la máquina virtual. Para buscar la dirección IP privada de una máquina virtual, consulte sus propiedades en Azure Portal o use PowerShell.
2. Verifique que está conectado a su red virtual mediante la conexión VPN de punto a sitio. 
3. Para abrir la Conexión a Escritorio remoto, escriba *RDP* o *Conexión a Escritorio remoto* en el cuadro de búsqueda de la barra de tareas y, después, seleccione **Conexión a Escritorio remoto**. También puede abrir esta opción con el comando **mstsc** de PowerShell. 
3. En **Conexión a Escritorio remoto**, escriba la dirección IP privada de la máquina virtual. Si es necesario, seleccione **Mostrar opciones** para ajustar más parámetros adicionales y, después, conéctese.

### <a name="to-troubleshoot-an-rdp-connection-to-a-vm"></a>Solución de problemas de una conexión de RDP a una máquina virtual

Si tiene problemas para conectarse a una máquina virtual a través de su conexión VPN, puede comprobar varios factores. 

- Compruebe que la conexión VPN se ha establecido correctamente.
- Compruebe que se conecta a la dirección IP privada de la máquina virtual.
- Escriba **ipconfig** para comprobar la dirección IPv4 asignada al adaptador de Ethernet en el equipo desde el que intenta conectarse. Se crea un espacio de direcciones superpuesto si la dirección IP está dentro del intervalo de direcciones de la red virtual a la que se va a conectar o dentro del intervalo de direcciones de su VPNClientAddressPool. Cuando el espacio de direcciones se superpone de esta manera, el tráfico de red no llega a Azure, sino que se mantiene en la red local.
- Si puede conectarse a la máquina virtual mediante la dirección IP privada, pero no el nombre del equipo, compruebe que ha configurado el DNS correctamente. Para más información acerca de cómo funciona la resolución de nombres para las máquinas virtuales, consulte [Resolución de nombres para las máquinas virtuales e instancias de rol](../articles/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).
- Compruebe que el paquete de configuración de cliente de VPN se generó después de que se especificaran las direcciones IP del servidor DNS para la red virtual. Si actualizó las direcciones IP de servidor DNS, genere un nuevo paquete de configuración de cliente de VPN e instálelo.

Para más información acerca de la solución de problemas, consulte [Solución de problemas de conexiones del Escritorio remoto a una máquina virtual de Azure](/troubleshoot/azure/virtual-machines/troubleshoot-rdp-connection).