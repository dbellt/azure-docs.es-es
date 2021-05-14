---
title: 'Enrutamiento del tráfico de red (tutorial): Azure Portal'
titlesuffix: Azure Virtual Network
description: En este tutorial, aprenderá a enrutar el tráfico de red con una tabla de rutas mediante Azure Portal.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/16/2021
ms.author: kumud
ms.openlocfilehash: 1d8a3bf8a17d04d98f605be3bcc01f47a61fdc89
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107906069"
---
# <a name="tutorial-route-network-traffic-with-a-route-table-using-the-azure-portal"></a>Tutorial: Enrutamiento del tráfico de red con una tabla de rutas mediante Azure Portal

De forma predeterminada, Azure enruta el tráfico entre todas las subredes de una red virtual. Sin embargo, puede crear sus propias rutas para invalidar las predeterminadas de Azure. Las rutas personalizadas resultan de utilidad si, por ejemplo, quiere enrutar el tráfico entre subredes por medio de una aplicación virtual de red (NVA). En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una aplicación virtual de red para enrutar el tráfico
> * Creación de una tabla de rutas
> * Creación de una ruta
> * Asociación de una tabla de rutas a una subred
> * Implementación de máquinas virtuales (VM) en subredes diferentes
> * Enrutamiento del tráfico desde una subred a otra a través de una aplicación virtual de red

Este tutorial usa [Azure Portal](https://portal.azure.com). También puede usar la [CLI de Azure](tutorial-create-route-table-cli.md) o [Azure PowerShell](tutorial-create-route-table-powershell.md).

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, debe tener una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-virtual-network"></a>Creación de una red virtual

1. En el menú de Azure Portal, seleccione **Crear un recurso**. En Azure Marketplace, seleccione **Redes** > **Red virtual**, o bien, busque **Red virtual** en el cuadro de búsqueda.

2. Seleccione **Crear**.

2. En **Creación de una red virtual**, escriba o seleccione esta información:

    | Configuración | Valor |
    | ------- | ----- |
    | Subscription | Seleccione su suscripción.|
    | Resource group | Seleccione **Crear nuevo** y escriba **myResourceGroup**. </br> Seleccione **Aceptar**. |
    | Nombre | Escriba **myVirtualNetwork**. |
    | Location | Seleccione **(EE. UU.) Este de EE. UU.** .|

3. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

4. En **Espacio de direcciones IPv4**, seleccione el espacio de direcciones existente y cámbielo a **10.0.0.0/16**.

4. Seleccione **+ Agregar subred** y escriba **Public** en **Nombre de subred** y **10.0.0.0/24** en **Intervalo de direcciones de subred**.

5. Seleccione **Agregar**.

6. Seleccione **+ Agregar subred** y escriba **Private** en **Nombre de subred** y **10.0.1.0/24** en **Intervalo de direcciones de subred**.

7. Seleccione **Agregar**.

8. Seleccione **+ Agregar subred** y escriba **DMZ** en **Nombre de subred** y **10.0.2.0/24** en **Intervalo de direcciones de subred**.

9. Seleccione **Agregar**.

10. Seleccione la pestaña **Seguridad** o elija el botón **Siguiente: Seguridad** situado en la parte inferior de la página.

11. En **BastionHost**, seleccione **Habilitar**. Escriba esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre del bastión | Escriba **myBastionHost**. |
    | Espacio de direcciones de AzureBastionSubnet | Escriba **10.0.3.0/24**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. </br> En **Nombre**, escriba **myBastionIP**. </br> Seleccione **Aceptar**. |

12. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

13. Seleccione **Crear**.

## <a name="create-an-nva"></a>Creación de una aplicación virtual de red

Las aplicaciones virtuales de red (NVA) son máquinas virtuales que ayudan con las funciones de red, como la optimización del enrutamiento y del firewall. En este tutorial se supone que utiliza **Windows Server 2019 Datacenter**. Si lo desea puede seleccionar un sistema operativo diferente.

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Parámetro | Valor                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroup**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVMNVA**. |
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

    | Parámetro | Valor |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | Seleccione **myVirtualNetwork**. |
    | Subnet | Seleccione **DMZ**. |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Básica**.|
    | Red de puertos de entrada públicos | Seleccione **Ninguno**. |
   
5. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.
  
6. Revise la configuración y, a continuación, seleccione **Crear**.

## <a name="create-a-route-table"></a>Creación de una tabla de rutas

1. En el menú [Azure Portal](https://portal.azure.com) o en la página **Inicio**, seleccione **Crear un recurso**.

2. En el cuadro de búsqueda, escriba **Tabla de enrutamiento**. Cuando **Tabla de enrutamiento** aparezca en los resultados de la búsqueda, selecciónelo.

3. En la página **Tabla de enrutamiento**, seleccione **Crear**.

4. En **Crear tabla de rutas**, en la pestaña **Conceptos básicos**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** |   |
    | Subscription | Seleccione su suscripción.|
    | Resource group | Seleccione **myResourceGroup**. |
    | **Detalles de instancia** |    |
    | Region | Seleccione **Este de EE. UU**. |
    | Nombre | Escriba **myRouteTablePublic**. |
    | Propagar las rutas de la puerta de enlace | Seleccione **Sí**. |

    :::image type="content" source="./media/tutorial-create-route-table-portal/create-route-table.png" alt-text="Crear tabla de rutas, Azure Portal." border="true":::

5. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.

## <a name="create-a-route"></a>Creación de una ruta

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la tabla de rutas. Busque y seleccione **Tablas de rutas**.

2. Seleccione el nombre de la tabla de rutas **myRouteTablePublic**.

3. En la página de **myRouteTablePublic**, en la sección **Configuración**, seleccione **Rutas**.

4. En la página de rutas, seleccione el botón **+ Agregar**.

5. En **Agregar ruta**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre de ruta | Escriba **ToPrivateSubnet**. |
    | Prefijo de dirección | Escriba **10.0.1.0/24** (el intervalo de direcciones de la subred **Private** creada anteriormente). |
    | Tipo de próximo salto | Seleccione **Aplicación virtual**. |
    | Siguiente dirección de salto | Escriba **10.0.2.4** (una dirección dentro del intervalo de direcciones de la subred **DMZ**). |

6. Seleccione **Aceptar**.

## <a name="associate-a-route-table-to-a-subnet"></a>Asociación de una tabla de rutas a una subred

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la red virtual. Busque y seleccione **Redes virtuales**.

2. Seleccione el nombre de la red virtual **myVirtualNetwork**.

3. En la página de **myVirtualNetwork**, en la sección **Configuración**, seleccione **Subredes**.

4. En la lista de subredes de la red virtual, elija **Público**.

5. En **Tabla de rutas**, elija la tabla de rutas que ha creado, **myRouteTablePublic**. 

6. Seleccione **Guardar** para asociar la tabla de rutas a la subred **Public**.

    :::image type="content" source="./media/tutorial-create-route-table-portal/associate-route-table.png" alt-text="Asociar tabla de rutas, lista de subredes, red virtual, Azure Portal." border="true":::

## <a name="turn-on-ip-forwarding"></a>Habilitación del reenvío IP

A continuación, active el reenvío IP para la nueva máquina virtual NVA, **myVMNVA**. Cuando Azure envía tráfico a **myVMNVA**, si se destina a una dirección IP diferente, el reenvío de IP envía el tráfico a la ubicación correcta.

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la máquina virtual. Busque y seleccione **Máquinas virtuales**.

2. Seleccione el nombre de la máquina virtual **myVMNVA**.

3. En la página de información general de **myVMNVA**, en **Configuración**, seleccione **Redes**.

4. En la página **Redes** de **myVMNVA**, seleccione la interfaz de red junto a **Interfaz de red**.  El nombre de la interfaz comenzará por **myvmnva**.

    :::image type="content" source="./media/tutorial-create-route-table-portal/virtual-machine-networking.png" alt-text="Redes, aplicación virtual de red (NVA), máquina virtual, Azure Portal" border="true":::

5. En la página de información general de la interfaz de red, en **Configuración**, seleccione **Configuraciones de IP**.

6. En la página **Configuraciones de IP**, establezca **Reenvío IP** en **Habilitado** y seleccione **Guardar**.

    :::image type="content" source="./media/tutorial-create-route-table-portal/enable-ip-forwarding.png" alt-text="Habilitación del reenvío IP" border="true":::

## <a name="create-public-and-private-virtual-machines"></a>Creación de máquinas virtuales públicas y privadas

Cree una máquina virtual pública y una máquina virtual privada en la red virtual. Más adelante, las usará para ver que Azure enruta el tráfico de subred **Público** a la subred **Privada** mediante la aplicación virtual de red.


### <a name="public-vm"></a>Máquina virtual pública

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Parámetro | Valor                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroup**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVmPublic**. |
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

    | Parámetro | Valor |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | Seleccione **myVirtualNetwork**. |
    | Subnet | Seleccione **Público**. |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Básica**.|
    | Red de puertos de entrada públicos | Seleccione **Ninguno**. |
   
5. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.
  
6. Revise la configuración y, a continuación, seleccione **Crear**.

### <a name="private-vm"></a>Máquina virtual privada

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Parámetro | Valor                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroup**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVmPrivate**. |
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

    | Parámetro | Valor |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | Seleccione **myVirtualNetwork**. |
    | Subnet | Seleccione **Privado**. |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Básica**.|
    | Red de puertos de entrada públicos | Seleccione **Ninguno**. |
   
5. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.
  
6. Revise la configuración y, a continuación, seleccione **Crear**.

## <a name="route-traffic-through-an-nva"></a>Enrutamiento del tráfico a través de una aplicación virtual de red

### <a name="sign-in-to-private-vm"></a>Inicio de sesión en una máquina virtual privada

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la máquina virtual privada. Busque y seleccione **Máquinas virtuales**.

2. Seleccione el nombre de la máquina virtual privada **myVmPrivate**.

3. En la barra de menús de la máquina virtual, seleccione **Conectar** y, a continuación, seleccione **Bastion**.

4. En la página **Conectar**, seleccione el botón azul **Usar Bastion**.

5. En la página **Bastion**, escriba el nombre de usuario y la contraseña que ha creado para la máquina virtual previamente.

6. Seleccione **Conectar**.

### <a name="configure-firewall"></a>Configurar el firewall

En un paso posterior, usará la herramienta trace route para probar el enrutamiento. Trace route usa el Protocolo de mensajes de control de Internet (ICMP), que el Firewall de Windows deniega de manera predeterminada. 

Habilite ICMP a través del Firewall de Windows.

1. En la conexión bastión de **myVMPrivate**, abra PowerShell con privilegios de administrador.

2. Escriba este comando:

    ```powershell
    New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
    ```

    Estará usando el comando trace route para probar el enrutamiento en este tutorial. Para entornos de producción, no se recomienda permitir ICMP a través del Firewall de Windows.

### <a name="turn-on-ip-forwarding-within-myvmnva"></a>Activación del reenvío IP en myVMNVA

Ha [activado el reenvío IP](#turn-on-ip-forwarding) para la interfaz de red de la máquina virtual con Azure. El sistema operativo de la máquina virtual también tiene que reenviar el tráfico. 

Active el reenvío IP para **myVMNVA** con estos comandos.

1. Desde PowerShell en la máquina virtual **myVMPrivate**, abra una sesión de escritorio remoto en la máquina virtual **myVMNVA**:

    ```powershell
    mstsc /v:myvmnva
    ```

2. En PowerShell, en la máquina virtual **myVMNVA**, escriba este comando para activar el reenvío IP:

    ```powershell
    Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -Name IpEnableRouter -Value 1
    ```

3. Reinicie **myVMNVA**.

    ```powershell
    Restart-Computer
    ```

4. Una vez reiniciada **myVMNVA**, cree una sesión de escritorio remoto en **myVMPublic**. 
    
    Aún conectado a **myVMPrivate**, abra PowerShell y ejecute este comando:

    ```powershell
    mstsc /v:myvmpublic
    ```
5. En el escritorio remoto de **myVMPublic**, abra PowerShell.

6. Habilite ICMP mediante el Firewall de Windows con el comando siguiente:

    ```powershell
    New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
    ```

## <a name="test-the-routing-of-network-traffic"></a>Prueba del enrutamiento del tráfico de red

En primer lugar, vamos a probar el enrutamiento del tráfico desde la máquina virtual **myVMPublic** a la máquina virtual **myVMPrivate**.

1. En PowerShell, en **myVMPublic**, escriba este comando:

    ```powershell
    tracert myvmprivate
    ```

    La respuesta es similar a este ejemplo:

    ```powershell
    Tracing route to myvmprivate.q04q2hv50taerlrtdyjz5nza1f.bx.internal.cloudapp.net [10.0.1.4]
    over a maximum of 30 hops:

      1     1 ms     *        2 ms  myvmnva.internal.cloudapp.net [10.0.2.4]
      2     2 ms     1 ms     1 ms  myvmprivate.internal.cloudapp.net [10.0.1.4]

    Trace complete.
    ```

    * Puede ver que el primer salto es 10.0.2.4, que es la dirección IP privada de **myVMNVA**. 

    * El segundo salto es a la dirección IP privada de **myVMPrivate**: 10.0.1.4. 

    Anteriormente, agregó la ruta a la tabla de rutas **myRouteTablePublic** y la asoció a la subred *Pública*. Azure envió el tráfico mediante la aplicación virtual de red y no directamente a la subred *Private*.

2. Cierre la sesión de escritorio remoto a **myVMPublic**, que aún le deja conectado a **myVMPrivate**.

3. Abra PowerShell en **myVMPrivate** y escriba este comando:

    ```powershell
    tracert myvmpublic
    ```

    Este comando prueba el enrutamiento del tráfico de red desde la máquina virtual *myVmPrivate* a la máquina virtual *myVmPublic*. La respuesta es similar a este ejemplo:

    ```cmd
    Tracing route to myvmpublic.q04q2hv50taerlrtdyjz5nza1f.bx.internal.cloudapp.net [10.0.0.4]
    over a maximum of 30 hops:

      1     1 ms     1 ms     1 ms  myvmpublic.internal.cloudapp.net [10.0.0.4]

    Trace complete.
    ```

    Puede ver que Azure enruta el tráfico directamente desde *myVMPrivate* a *myVMPublic*. De forma predeterminada, Azure enruta el tráfico directamente entre subredes.

4. Cierre la sesión bastión a **myVMPrivate**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando el grupo de recursos ya no sea necesario, elimine **myResourceGroup** y todos los recursos que contiene:

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar el grupo de recursos. Busque y seleccione **Grupos de recursos**.

2. Selecione el nombre del grupo de recursos, **myResourceGroup**.

3. Seleccione **Eliminar grupo de recursos**.

4. En el cuadro de diálogo de confirmación, escriba **myResourceGroup** en **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS** y, después, seleccione **Eliminar**. 


## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha:

* Creado una tabla de rutas y la ha asociado a una subred.
* Creado una aplicación virtual de red sencilla que enrutó el tráfico desde una subred pública hasta una subred privada. 

Puede implementar diferentes aplicaciones virtuales de red preconfiguradas desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking), que proporcionan muchas funciones de red útiles. 

Para más información acerca del enrutamiento, consulte [Introducción al enrutamiento](virtual-networks-udr-overview.md) y [Administración de una tabla de rutas](manage-route-table.md).

Para filtrar tráfico en una red virtual, consulte:
> [!div class="nextstepaction"]
> [Tutorial: Filtrado del tráfico de red con un grupo de seguridad de red en Azure Portal](tutorial-filter-network-traffic.md)
