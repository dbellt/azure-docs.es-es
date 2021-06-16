---
title: 'Tutorial sobre conexión de redes virtuales con emparejamiento de VNet: Azure Portal'
description: En este tutorial, aprenderá a conectar las redes virtuales con emparejamiento de redes virtuales mediante Azure Portal.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 01/22/2020
ms.author: kumud
ms.custom: ''
ms.openlocfilehash: 8a0dd8b20331d2f296767bc5628e901600f835e0
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078157"
---
# <a name="tutorial-connect-virtual-networks-with-virtual-network-peering-using-the-azure-portal"></a>Tutorial: Conexión de redes virtuales con emparejamiento de redes virtuales usando Azure Portal

Puede conectar redes virtuales entre sí con el emparejamiento de redes virtuales. Estas redes virtuales pueden estar en la misma región o en regiones diferentes (también conocidas como emparejamiento de VNET global). Una vez que las redes virtuales están emparejadas, los recursos de ambas se pueden comunicar entre sí con el mismo ancho de banda y la misma latencia que si estuvieran en la misma red virtual. En este tutorial aprenderá a:

> [!div class="checklist"]
> * Crear dos redes virtuales
> * Conectar dos redes virtuales con el emparejamiento de redes virtuales
> * Implementar una máquina virtual (VM) en cada red virtual
> * Comunicarse entre máquinas virtuales

Si lo prefiere, puede completar este tutorial con la [CLI de Azure](tutorial-connect-virtual-networks-cli.md) o [Azure PowerShell](tutorial-connect-virtual-networks-powershell.md).

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, debe tener una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-virtual-networks"></a>Creación de redes virtuales

1. En Azure Portal, seleccione **Crear un recurso**.
2. Seleccione **Redes** y **Red virtual**.
3. En la pestaña **Fundamentos**, escriba o seleccione la siguiente información y acepte los valores predeterminados en las siguientes opciones:

    |Configuración|Valor|
    |---|---|
    |Subscription| Seleccione su suscripción.|
    |Resource group| Haga clic en **Crear nuevo** y escriba *myResourceGroup*.|
    |Region| Seleccione **Este de EE. UU**.|
    |Nombre|myVirtualNetwork1|

4. En la pestaña **Direcciones IP**, escriba 10.0.0.0/16 en el campo **Espacio de direcciones**. Haga clic en el botón **Agregar subred** que aparece a continuación y escriba *Subnet1* en **Nombre de subred** y 10.0.0.0/24 en **Intervalo de direcciones de subred**.
5. Seleccione **Revisar y crear** y, luego, **Crear**.
   
5. Complete de nuevo los pasos del 1 al 5, con los cambios siguientes:

    |Configuración|Valor|
    |---|---|
    |Nombre|myVirtualNetwork2|
    |Espacio de direcciones|10.1.0.0/16|
    |Grupo de recursos| Seleccione **Usar existente** y después seleccione **myResourceGroup**.|
    |Nombre de subred | Subnet2|
    |Intervalo de direcciones de subred|10.1.0.0/24|

## <a name="peer-virtual-networks"></a>Emparejamiento de redes virtuales

1. En el cuadro Búsqueda que se encuentra en la parte superior de Azure Portal, escriba *MyVirtualNetwork1*. Cuando la opción **myVirtualNetwork1** aparezca en los resultados de la búsqueda, selecciónela.
2. Seleccione **Emparejamientos**, seleccione **Configuración** y, luego, seleccione **Agregar**, como se muestra en la imagen siguiente:

    ![Creación de emparejamiento](./media/tutorial-connect-virtual-networks-portal/create-peering.png)

3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**.

    |Configuración|Value|
    |---|---|
    |Nombre del emparejamiento de myVirtualNetwork1 a la red virtual remota|myVirtualNetwork1-myVirtualNetwork2: la primera vez que se cargue la página, verá aquí la frase "remote virtual network" (red virtual remota). Después de elegir la red virtual remota, la frase "remote virtual network" (red virtual remota) se reemplazará por el nombre de la red virtual remota.|
    |Subscription| Seleccione su suscripción.|
    |Virtual network|myVirtualNetwork2: para seleccionar la red virtual *myVirtualNetwork2*, seleccione **Red virtual** y, después, **myVirtualNetwork2 (myResourceGroup)**. Puede seleccionar una red virtuales de la misma región o de otra.|
    |Nombre del emparejamiento de myVirtualNetwork2 con myVirtualNetwork1|myVirtualNetwork2-myVirtualNetwork1|

    ![Configuración de emparejamiento](./media/tutorial-connect-virtual-networks-portal/peering-settings-bidirectional.png)

    El **ESTADO DE EMPAREJAMIENTO** es *Conectado*, como se muestra en la siguiente imagen:

    ![Estado de emparejamiento](./media/tutorial-connect-virtual-networks-portal/peering-status-connected.png)

    Si no ve el estado, actualice el explorador.

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree una máquina virtual en cada red virtual para que puedan comunicarse entre ellas en un paso posterior.

### <a name="create-the-first-vm"></a>Creación de la primera máquina virtual

1. En Azure Portal, seleccione **Crear un recurso**.
2. Seleccione **Compute** y, después, seleccione **Windows Server 2016 Datacenter**. Puede seleccionar otro sistema operativo, pero en los pasos restantes se supone que seleccionó **Windows Server 2016 Datacenter**. 
3. Escriba o seleccione la siguiente información para **Aspectos básicos**, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Crear**:

    |Configuración|Valor|
    |---|---|
    |Grupo de recursos| Seleccione **Usar existente** y después seleccione **myResourceGroup**.|
    |Nombre|myVm1|
    |Location| Seleccione **Este de EE. UU**.|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Contraseña| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.yml?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm-).|
   
4. Seleccione un tamaño de máquina virtual para la opción **Tamaño**.
5. Seleccione los valores siguientes en **Redes**:

    |Configuración|Value|
    |---|---|
    |Virtual network| myVirtualNetwork1: si no está ya seleccionado, seleccione **Red virtual** y, luego, **myVirtualNetwork1**.|
    |Subnet| Subnet1: si aún no está seleccionado, seleccione **Subred** y, después, **Subnet1**.|
   
6. Seleccione **Redes**. Elija **Permitir los puertos seleccionados** en la opción **Puertos de entrada públicos**. Elija **RDP** para la opción **Seleccionar puertos de entrada** debajo de esto. 

7. Seleccione el botón **Revisar y crear** en la esquina inferior izquierda para iniciar la implementación de la máquina virtual.

### <a name="create-the-second-vm"></a>Creación de la segunda máquina virtual

Complete de nuevo los pasos del 1 al 6, con los cambios siguientes:

|Configuración|Valor|
|---|---|
|Nombre | myVm2|
|Virtual network | myVirtualNetwork2|

Las máquinas virtuales tardan unos minutos en crearse. No siga con los pasos restantes hasta que se creen ambas máquinas virtuales.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="communicate-between-vms"></a>Comunicarse entre máquinas virtuales

1. En el cuadro *Búsqueda* que se encuentra en la parte superior del portal, escriba *myVm1*. Seleccione **MyVm1** cuando aparezca en los resultados de búsqueda.
2. Cree una conexión a Escritorio remoto a la máquina virtual *myVm1* seleccionando **Conectar**, tal como se indica en la imagen siguiente:

    ![Conexión a la máquina virtual](./media/tutorial-connect-virtual-networks-portal/connect-to-virtual-machine.png)  

3. Para conectarse a la máquina virtual, abra el archivo RDP descargado. Cuando se le pida, seleccione **Conectar**.
4. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual (puede que deba seleccionar **Más opciones** y luego **Usar una cuenta diferente**, para especificar las credenciales que escribió cuando creó la máquina virtual). A continuación, seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Seleccione **Sí** para continuar con la conexión.
6. En un paso posterior, se usa ping para comunicarse con la máquina virtual *myVm2* desde la máquina virtual *myVm1*. Ping usa el Protocolo de mensajes de control de Internet (ICMP) que, de forma predeterminada, se deniega a través del Firewall de Windows. En la máquina virtual *myVm1*, habilite IMCP a través del Firewall de Windows para que pueda hacer ping en esta máquina virtual desde *myVm2* en un paso posterior, mediante PowerShell:

    ```powershell
    New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
    ```
    
    Aunque en este tutorial se usa ping para comunicarse entre máquinas virtuales, no se recomienda permitir que ICMP atraviese el Firewall de Windows en implementaciones de producción.

7. Para conectar con la máquina virtual *myVm2*, escriba el siguiente comando desde el símbolo del sistema en la máquina virtual *myVm1*:

    ```
    mstsc /v:10.1.0.4
    ```
    
8. Puesto que ha habilitado ping en *myVm1*, ahora puede hacer ping a él por dirección IP:

    ```
    ping 10.0.0.4
    ```
    
9. Desconecte las sesiones RDP para ambos *myVm1* y *myVm2*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesario, elimine el grupo de recursos y todos los recursos que contiene: 

1. Escriba *myResourceGroup* en el cuadro **Buscar** que se encuentra en la parte superior del portal. Seleccione **myResourceGroup** cuando lo vea en los resultados de la búsqueda.
2. Seleccione **Eliminar grupo de recursos**.
3. Escriba *myResourceGroup* para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información acerca del emparejamiento de red virtual](virtual-network-peering-overview.md).


