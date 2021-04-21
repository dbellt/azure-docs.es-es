---
title: 'Inicio rápido: Creación de una red virtual mediante Azure Portal'
titleSuffix: Azure Virtual Network
description: En este inicio rápido aprenderá a crear una red virtual mediante Azure Portal.
author: KumudD
ms.author: kumud
ms.date: 03/17/2021
ms.topic: quickstart
ms.service: virtual-network
ms.workload: infrastructure
ms.tgt_pltfrm: virtual-network
ms.devlang: na
tags:
- azure-resource-manager
ms.custom:
- mode-portal
ms.openlocfilehash: 43c45b43084656a45d2509ee2c7a4376cdc7c052
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107531188"
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-portal"></a>Inicio rápido: Creación de una red virtual mediante el Portal de Azure

En esta guía de inicio rápido aprenderá a crear una red virtual mediante Azure Portal. Va a implementar dos máquinas virtuales. Después, debe comunicarse de forma segura entre las máquinas virtuales y conectarse a las máquinas virtuales desde Internet. Una red virtual es el bloque de creación básico de una red privada en Azure. Permite que los recursos de Azure, como las máquinas virtuales, se comuniquen de manera segura entre sí y con Internet.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-virtual-network"></a>Creación de una red virtual

1. Seleccione **Crear un recurso** en la esquina superior izquierda del portal.

2. En el cuadro de búsqueda del portal, escriba **red virtual**. En los resultados de la búsqueda, seleccione **Redes virtuales**.

3. En la página **Red virtual**, seleccione **Crear**.

4. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

    | Parámetro | Value |
    | ------- | ----- |
    | **Detalles del proyecto** |   |
    | Suscripción | Seleccione su suscripción. |
    | Resource group | Seleccione **Crear nuevo**.  </br> Escriba **myResourceGroup**. </br> Seleccione **Aceptar**. |
    | **Detalles de instancia** |   |
    | Nombre | Escriba **myVNet**. |
    | Region | Seleccione **(EE. UU.) Este de EE. UU.** . |

    :::image type="content" source="./media/quick-create-portal/create-virtual-network.png" alt-text="Creación de una red virtual con Azure Portal" border="true":::

5. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

6. En **Espacio de direcciones IPv4**, seleccione el espacio de direcciones existente y cámbielo a **10.1.0.0/16**.

7. Seleccione **+ Agregar subred** y, a continuación, escriba **MySubnet** para el **nombre de subred** y **10.1.0.0/24** para **Intervalo de direcciones de subred**.

8. Seleccione **Agregar**.

9. Seleccione la pestaña **Seguridad** o elija el botón **Siguiente: Seguridad** situado en la parte inferior de la página.

10. En **BastionHost**, seleccione **Habilitar**. Escriba esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre del bastión | Escriba **myBastionHost**. |
    | Espacio de direcciones de AzureBastionSubnet | Escriba **10.1.1.0/24**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. </br> En **Nombre**, escriba **myBastionIP**. </br> Seleccione **Aceptar**. |

11. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

12. Seleccione **Crear**.

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree dos máquinas virtuales en la red virtual:

### <a name="create-the-first-vm"></a>Creación de la primera máquina virtual

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Configuración | Value                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroup**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVM1**. |
    | Region | Seleccione **(EE.UU.) Este de EE. UU.** |
    | Opciones de disponibilidad | Seleccione **No se requiere redundancia de la infraestructura** |
    | Imagen | Seleccione **Windows Server 2019 Datacenter**. |
    | Instancia de Azure Spot | Seleccione **No**. |
    | Size | Elija el tamaño de la máquina virtual o acepte la configuración predeterminada. |
    | **Cuenta de administrador** |  |
    | Nombre de usuario | Escriba un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmar contraseña | Vuelva a escribir la contraseña. |
    | **Reglas de puerto de entrada** |    |
    | Puertos de entrada públicos | Seleccione **Ninguno**. |
    |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Value |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | Seleccione **myVNet**. |
    | Subnet | Seleccione **mySubnet**. |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Básica**.|
    | Red de puertos de entrada públicos | Seleccione **Ninguno**. |
   
5. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.
  
6. Revise la configuración y, a continuación, seleccione **Crear**.

### <a name="create-the-second-vm"></a>Creación de la segunda máquina virtual

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Configuración | Value                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroup**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVM2**. |
    | Region | Seleccione **(EE.UU.) Este de EE. UU.** |
    | Opciones de disponibilidad | Seleccione **No se requiere redundancia de la infraestructura** |
    | Imagen | Seleccione **Windows Server 2019 Datacenter**. |
    | Instancia de Azure Spot | Seleccione **No**. |
    | Size | Elija el tamaño de la máquina virtual o acepte la configuración predeterminada. |
    | **Cuenta de administrador** |  |
    | Nombre de usuario | Escriba un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmar contraseña | Vuelva a escribir la contraseña. |
    | **Reglas de puerto de entrada** |    |
    | Puertos de entrada públicos | Seleccione **Ninguno**. |
    |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Value |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | Seleccione **myVNet**. |
    | Subnet | Seleccione **mySubnet**. |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Básica**.|
    | Red de puertos de entrada públicos | Seleccione **Ninguno**. |
   
5. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.
  
6. Revise la configuración y, a continuación, seleccione **Crear**.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="connect-to-myvm1"></a>Conexión a myVM1

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la máquina virtual privada. Busque y seleccione **Máquinas virtuales**.

2. Seleccione el nombre de la máquina virtual privada **myVM1**.

3. En la barra de menús de la máquina virtual, seleccione **Conectar** y, a continuación, seleccione **Bastion**.

    :::image type="content" source="./media/quick-create-portal/connect-to-virtual-machine.png" alt-text="Conexión a myVM1 con Azure Bastion" border="true":::

4. En la página **Conectar**, seleccione el botón azul **Usar Bastion**.

5. En la página **Bastion**, escriba el nombre de usuario y la contraseña que ha creado para la máquina virtual previamente.

6. Seleccione **Conectar**.

## <a name="communicate-between-vms"></a>Comunicarse entre máquinas virtuales

1. En la conexión bastión de **myVM1**, abra PowerShell.

2. Escriba `ping myvm2`.

    Recibirá un mensaje similar a esta salida:

    ```powershell
    Pinging myvm2.cs4wv3rxdjgedggsfghkjrxuqf.bx.internal.cloudapp.net [10.1.0.5] with 32 bytes of data:
    Reply from 10.1.0.5: bytes=32 time=3ms TTL=128
    Reply from 10.1.0.5: bytes=32 time=1ms TTL=128
    Reply from 10.1.0.5: bytes=32 time=1ms TTL=128
    Reply from 10.1.0.5: bytes=32 time=1ms TTL=128

    Ping statistics for 10.1.0.5:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 1ms, Maximum = 3ms, Average = 1ms
    ```

3. Cierre la conexión bastión a **myVM1**.

4. Siga los pasos que se indican en [Conexión a myVM1](#connect-to-myvm1), pero conéctese a **myVM2**.

5. Abra PowerShell en **myVM2** y escriba `ping myvm1`.

    Recibirá un mensaje similar al siguiente:

    ```powershell
    Pinging myvm1.cs4wv3rxdjgedggsfghkjrxuqf.bx.internal.cloudapp.net [10.1.0.4] with 32 bytes of data:
    Reply from 10.1.0.4: bytes=32 time=1ms TTL=128
    Reply from 10.1.0.4: bytes=32 time=1ms TTL=128
    Reply from 10.1.0.4: bytes=32 time=1ms TTL=128
    Reply from 10.1.0.4: bytes=32 time=1ms TTL=128

    Ping statistics for 10.1.0.4:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 1ms, Maximum = 1ms, Average = 1ms
    ```

7. Cierre la conexión bastión a **myVM2**.

## <a name="clean-up-resources"></a>Limpieza de recursos

En esta guía de inicio rápido, ha creado una red virtual predeterminada y dos máquinas virtuales. 

Se ha conectado a una VM desde Internet y se ha comunicado de forma segura entre las dos VM.

Cuando haya terminado de usar la red virtual y las VM, elimine el grupo de recursos y todos los recursos que contiene:

1. Busque y seleccione **myResourceGroup**.

1. Seleccione **Eliminar grupo de recursos**.

1. Escriba **myResourceGroup** para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la configuración de red virtual, consulte [Crear, cambiar o eliminar una red virtual](manage-virtual-network.md).

Para más información sobre los tipos de comunicaciones de red de máquinas virtuales, consulte el artículo sobre el [filtrado del tráfico de red](tutorial-filter-network-traffic.md).
