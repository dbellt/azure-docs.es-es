---
title: Implementación y configuración de Azure Firewall en una red híbrida con Azure Portal
description: En este artículo, aprenderá a implementar y configurar Azure Firewall mediante Azure Portal.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 04/29/2021
ms.author: victorh
customer intent: As an administrator, I want to control network access from an on-premises network to an Azure virtual network.
ms.openlocfilehash: 36605d6bf17c7652e7f21b89a83af08972765a30
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287610"
---
# <a name="deploy-and-configure-azure-firewall-in-a-hybrid-network-using-the-azure-portal"></a>Implementación y configuración de Azure Firewall en una red híbrida con Azure Portal

Cuando conecta la red local a una red virtual de Azure para crear una red híbrida, la capacidad de controlar el acceso a los recursos de la red de Azure es parte importante de un plan de seguridad global.

Puede usar Azure Firewall para controlar el acceso de red en una red híbrida con reglas que definen el tráfico de red que se permite o que se rechaza.

En este artículo se crearán tres redes virtuales:

- **VNet-Hub**: el firewall está en esta red virtual.
- **VNet-Spoke**: la red virtual Spoke representa la carga de trabajo ubicada en Azure.
- **VNet-Onprem**: la red virtual local representa una red local. En una implementación real, se puede conectar mediante una conexión VPN o ExpressRoute. Para simplificar, este procedimiento usa una conexión de puerta de enlace de VPN y una red virtual ubicada en Azure para representar una red local.

![Firewall en una red híbrida](media/tutorial-hybrid-ps/hybrid-network-firewall.png)

En este artículo aprenderá a:

> [!div class="checklist"]
> * Crear la red virtual del centro de firewall
> * Crear la red virtual de tipo hub-and-spoke
> * Crear la red virtual local
> * Configuración e implementación del firewall
> * Creación y conexión de las puertas de enlace de VPN
> * Emparejar las redes virtuales de tipo hub-and-spoke
> * Creación de las rutas
> * Creación de las máquinas virtuales
> * Probar el firewall

Si desea usar Azure PowerShell en su lugar para completar este procedimiento, consulte [Implementación y configuración de Azure Firewall en una red híbrida con Azure PowerShell](tutorial-hybrid-ps.md).

> [!NOTE]
> En este artículo se usan reglas de firewall clásicas para administrar el firewall. El método preferido es usar una [directiva de firewall](../firewall-manager/policy-overview.md). Para completar este procedimiento con una directiva de firewall, consulte [Tutorial: Implementación y configuración de Azure Firewall y una directiva en una red híbrida con Azure Portal](tutorial-hybrid-portal-policy.md).

## <a name="prerequisites"></a>Requisitos previos

Una red híbrida usa el modelo de arquitectura radial para enrutar el tráfico entre redes virtuales de Azure y redes locales. La arquitectura radial tiene los siguientes requisitos:

- Establezca **Use this virtual network's gateway or Route Server** (Usar la puerta de enlace o el servidor de rutas de esta red virtual) al emparejar el concentrador de la red virtual a un radio de la red virtual. En una arquitectura de red radial, el tránsito de una puerta de enlace permite que las redes virtuales de radio compartan la puerta de enlace de VPN en el centro, en lugar de implementar puertas de enlace de VPN en todas las redes virtuales de radio. 

   Además, las rutas a las redes virtuales conectadas a la puerta de enlace o a las redes locales se propagarán automáticamente a las tablas de enrutamiento de las redes virtuales emparejadas mediante el tránsito de la puerta de enlace. Para más información, consulte [Configuración del tránsito de la puerta de enlace de VPN para el emparejamiento de red virtual](../vpn-gateway/vpn-gateway-peering-gateway-transit.md).

- Establezca **Use the remote virtual network's gateways or Route Server** (Usar las puertas de enlace o el servidor de rutas de la red virtual remota) al emparejar un radio de la red virtual al concentrador de la red virtual. Si se establece **Use the remote virtual network's gateways or Route Server** (Usar las puertas de enlace o el servidor de rutas de la red virtual remota) y también se establece **Use this virtual network's gateway or Route Server** (Usar la puerta de enlace o el servidor de rutas de esta red virtual) en el emparejamiento remoto, la red virtual del radio usa las puertas de enlace de la red virtual remota para el tránsito.
- Para enrutar el tráfico de la subred de radio a través del firewall de centro, puede usar una ruta definida por el usuario (UDR) que apunte al firewall con la opción **Virtual network gateway route propagation** (Deshabilitar la propagación de rutas de la puerta de enlace de red virtual) deshabilitada. La opción **Propagación de rutas de puerta de enlace de red virtual** deshabilitada evita la distribución de rutas a las subredes de radio. Esto evita que las rutas aprendidas entren en conflicto con la UDR. Si desea mantener la opción **Virtual network gateway route propagation** (Deshabilitar la propagación de rutas de la puerta de enlace de red virtual) habilitada, asegúrese de definir rutas específicas para el firewall con el fin de invalidar las que se publican desde el entorno local mediante el protocolo de puerta de enlace de borde.
- Configure una ruta definida por el usuario en la subred de la puerta de enlace del centro que apunte a la dirección IP del firewall como próximo salto para las redes de radio. No se requiere ninguna ruta definida por el usuario en la subred de Azure Firewall, ya que obtiene las rutas de BGP.

Consulte la sección [Creación de rutas](#create-the-routes) en este artículo para ver cómo se crean estas rutas.

>[!NOTE]
>Azure Firewall debe tener conectividad directa a Internet. Si AzureFirewallSubnet aprende una ruta predeterminada a la red local mediante BGP, debe reemplazarla por una UDR 0.0.0.0/0 con el valor **NextHopType** establecido como **Internet** para mantener la conectividad directa a Internet.
>
>Azure Firewall puede configurarse para admitir la tunelización forzada. Para más información, consulte [Tunelización forzada de Azure Firewall](forced-tunneling.md).

>[!NOTE]
>El tráfico entre redes virtuales emparejadas directamente se enruta directamente aunque una ruta definida por el usuario apunte a Azure Firewall como puerta de enlace predeterminada. Para enviar tráfico de subred a subred al firewall en este escenario, una UDR debe contener el prefijo de red de la subred de destino de forma explícita en ambas subredes.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-the-firewall-hub-virtual-network"></a>Crear la red virtual del centro de firewall

En primer lugar, cree el grupo de recursos en el que se incluirán los recursos:

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En la página principal de Azure Portal, seleccione **Grupos de recursos** > **Agregar**.
3. En **Suscripción**, seleccione la suscripción.
1. En **Nombre del grupo de recursos**, escriba **FW-Hybrid-Test**.
2. En **Región**, seleccione **(EE. UU.) Este de EE. UU.** . Todos los recursos que cree después deben estar en la misma ubicación.
3. Seleccione **Revisar + crear**.
4. Seleccione **Crear**.

Ahora, cree la red virtual:

> [!NOTE]
> El tamaño de la subred AzureFirewallSubnet es /26. Para más información sobre el tamaño de la subred, consulte [Preguntas más frecuentes sobre Azure Firewall](firewall-faq.yml#why-does-azure-firewall-need-a--26-subnet-size).

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En **Redes**, seleccione **Red virtuales**.
1. Seleccione **Crear**.
1. En **Grupo de recursos**, seleccione **FW-Hybrid-Test**.
1. En **Nombre**, escriba **Vnet-Hub**.
1. Seleccione **Siguiente: Direcciones IP**.
1. En **Espacio de direcciones IPv4**, elimine la dirección predeterminada y escriba **10.5.0.0/16**.
1. En **Nombre de subred**, seleccione **Agregar subred**.
1. En **Nombre de subred**, escriba **AzureFirewallSubnet**. El firewall estará en esta subred y el nombre de la subred **debe** ser AzureFirewallSubnet.
1. En **Intervalo de direcciones de subred**, escriba **10.5.0.0/26**. 
1. Seleccione **Agregar**.
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**.

## <a name="create-the-spoke-virtual-network"></a>Crear la red virtual de tipo hub-and-spoke

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En **Redes**, seleccione **Red virtual**.
7. En **Grupo de recursos**, seleccione **FW-Hybrid-Test**.
1. En **Nombre**, escriba **VNet-Spoke**.
2. En **Región**, seleccione **(EE. UU.) Este de EE. UU.** .
3. Seleccione **Siguiente: Direcciones IP**.
4. En **Espacio de direcciones IPv4**, elimine la dirección predeterminada y escriba **10.6.0.0/16**.
6. En **Nombre de subred**, seleccione **Agregar subred**.
7. En **Nombre de subred**, escriba **SN-Workload**.
8. En **Intervalo de direcciones de subred**, escriba **10.6.0.0/24**. 
9. Seleccione **Agregar**.
10. Seleccione **Revisar y crear**.
11. Seleccione **Crear**.

## <a name="create-the-on-premises-virtual-network"></a>Crear la red virtual local

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En **Redes**, seleccione **Red virtual**.
7. En **Grupo de recursos**, seleccione **FW-Hybrid-Test**.
1. En **Nombre**, escriba **VNet-OnPrem**.
2. En **Región**, seleccione **(EE. UU.) Este de EE. UU.** .
3. Seleccione **Siguiente: Direcciones IP**
4. En **Espacio de direcciones IPv4**, elimine la dirección predeterminada y escriba **192.168.0.0/16**.
5. En **Nombre de subred**, seleccione **Agregar subred**.
7. En **Nombre de subred**, escriba **SN-Corp**.
8. En **Intervalo de direcciones de subred**, escriba **192.168.1.0/24**. 
9. Seleccione **Agregar**.
10. Seleccione **Revisar y crear**.
11. Seleccione **Crear**.

Ahora cree una segunda subred para la puerta de enlace.

1. En la página **VNet-Onprem**, seleccione **Subredes**.
2. Seleccione **+Subred**.
3. En **Nombre**, escriba **GatewaySubnet.**
4. En **Intervalo de direcciones de subred**, escriba **192.168.2.0/24**.
5. Seleccione **Aceptar**.

## <a name="configure-and-deploy-the-firewall"></a>Configuración e implementación del firewall

Ahora, implemente el firewall en la red virtual del concentrador de firewall.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En la columna de la izquierda, seleccione **Redes** y, a continuación, busque y seleccione **Firewall**.
4. En la página **Creación de un firewall**, utilice la tabla siguiente para configurar el firewall:

   |Configuración  |Value  |
   |---------|---------|
   |Suscripción     |\<your subscription\>|
   |Resource group     |**FW-Hybrid-Test** |
   |Nombre     |**AzFW01**|
   |Region     |**Este de EE. UU.**|
   |Administración del firewall|**Use reglas de firewall (clásicas) para administrar este firewall**.|
   |Elegir una red virtual     |**Usar existente**:<br> **VNet-hub**|
   |Dirección IP pública     |Agregar nueva: <br>**fw-pip**. |

5. Seleccione **Revisar + crear**.
6. Revise el resumen y seleccione **Crear** para crear el firewall.

   Esto tarda unos minutos en implementarse.
7. Una vez finalizada la implementación, vaya al grupo de recursos **FW-Hybrid-Test** y seleccione el firewall **AzFW01**.
8. Anote la dirección IP privada. Se usará más adelante al crear la ruta predeterminada.

### <a name="configure-network-rules"></a>Configuración de reglas de red

En primer lugar, agregue una regla de red para permitir el tráfico web.

1. En la página **AzFW01**, seleccione **Reglas**.
2. Seleccione la pestaña **Recopilación de reglas de red**.
3. Seleccione **Agregar una colección de reglas de red** .
4. En **Nombre**, escriba **RCNet01**.
5. En **Prioridad**, escriba **100**.
6. En **Acción de recopilación de reglas**, seleccione **Denegar**.
6. En **Reglas**, como **Nombre**, escriba **AllowWeb**.
8. Como **Tipo de origen**, seleccione **Dirección IP**.
9. Como **Origen**, escriba **192.168.1.0/24**.
7. En **Protocolo**, seleccione **TCP**.
1. En **Puertos de destino**, escriba **80**.
1. En **Tipo de destino**, seleccione **Dirección IP**.
1. En **Destino**, escriba **10.6.0.0/16**.

Ahora, agregue una regla para permitir el tráfico RDP.

En la segunda fila de la regla, escriba la siguiente información:

1. En **Nombre**, escriba **AllowRDP**.
3. Como **Tipo de origen**, seleccione **Dirección IP**.
4. Como **Origen**, escriba **192.168.1.0/24**.
2. En **Protocolo**, seleccione **TCP**.
1. En **Puertos de destino**, escriba **3389**.
1. En **Tipo de destino**, seleccione **Dirección IP**.
1. En **Destino**, escriba **10.6.0.0/16**.
1. Seleccione **Agregar**.

## <a name="create-and-connect-the-vpn-gateways"></a>Creación y conexión de las puertas de enlace de VPN

Las redes virtuales de centro y local están conectadas mediante puertas de enlace VPN.

### <a name="create-a-vpn-gateway-for-the-hub-virtual-network"></a>Creación de una puerta de enlace VPN para la red virtual de centro

Cree la puerta de enlace VPN para la red virtual de centro. Las configuraciones de red a red requieren un VpnType RouteBased. La creación de una puerta de enlace de VPN suele tardar 45 minutos o más, según la SKU de la puerta de enlace de VPN seleccionada.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En el cuadro de texto de búsqueda, escriba **puerta de enlace de red virtual**.
3. Seleccione **Puerta de enlace de red virtual** y, después, **Crear**.
4. En **Nombre**, escriba **GW-hub**.
5. En **Región**, seleccione la misma región que usó anteriormente.
6. En **Tipo de puerta de enlace** seleccione **VPN**.
7. En **Tipo de VPN** seleccione **Basada en rutas**.
8. En **SKU**, seleccione **Básico**.
9. En **Red virtual**, seleccione **VNet-hub**.
10. En **Dirección IP pública**, seleccione **Crear nuevo** y escriba **VNet-hub-GW-pip** como nombre.
11. Acepte el resto de valores predeterminados y seleccione **Revisar y crear**.
12. Revise la configuración y, después, seleccione **Crear**.

### <a name="create-a-vpn-gateway-for-the-on-premises-virtual-network"></a>Creación de una puerta de enlace VPN para la red virtual local

Cree la puerta de enlace VPN para la red virtual local. Las configuraciones de red a red requieren un VpnType RouteBased. La creación de una puerta de enlace de VPN suele tardar 45 minutos o más, según la SKU de la puerta de enlace de VPN seleccionada.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En el cuadro de texto de búsqueda, escriba **puerta de enlace de red virtual** y presione **Entrar**.
3. Seleccione **Puerta de enlace de red virtual** y, después, **Crear**.
4. En **Nombre**, escriba **GW-Onprem**.
5. En **Región**, seleccione la misma región que usó anteriormente.
6. En **Tipo de puerta de enlace** seleccione **VPN**.
7. En **Tipo de VPN** seleccione **Basada en rutas**.
8. En **SKU**, seleccione **Básico**.
9. En **Red virtual**, selecciona **VNet-Onprem**.
10. En **Dirección IP pública**, seleccione **Crear nuevo** y escriba **VNet-Onprem-GW-pip** como nombre.
11. Acepte el resto de valores predeterminados y seleccione **Revisar y crear**.
12. Revise la configuración y, después, seleccione **Crear**.

### <a name="create-the-vpn-connections"></a>Creación de las conexiones

Ahora puede crear las conexiones de VPN entre las puertas de enlace de centro y local.

En este paso va a crear la conexión entre la red virtual de centro y la red virtual local. Verá una clave compartida a la que se hace referencia en los ejemplos. Puede utilizar sus propios valores para la clave compartida. Lo importante es que la clave compartida coincida en ambas conexiones. Se tardará unos momentos en terminar de crear la conexión.

1. Abra el grupo de recursos **FW-Hybrid-Test** y seleccione la puerta de enlace **GW-hub**.
2. Seleccione **Conexiones** en la columna izquierda.
3. Seleccione **Agregar**.
4. En el nombre de la conexión, escriba **Hub-to-Onprem**.
5. En **Tipo de conexión**, seleccione **De VNet a VNet**.
6. En **Segunda puerta enlace de red virtual**, seleccione **GW-Onprem**.
7. En **Clave compartida (PSK)** , escriba **AzureA1b2C3**.
8. Seleccione **Aceptar**.

Cree la conexión de red entre la red virtual local y la red virtual de centro. Este paso es similar al anterior, excepto que se crea una conexión desde la red virtual local a la red virtual de centro. Asegúrese de que coincidan las claves compartidas. Después de unos minutos, se habrá establecido la conexión.

1. Abra el grupo de recursos **FW-Hybrid-Test** y seleccione la puerta de enlace **GW-Onprem**.
2. Seleccione **Conexiones** en la columna izquierda.
3. Seleccione **Agregar**.
4. Para el nombre de la conexión, escriba **Onprem-to-Hub**.
5. En **Tipo de conexión**, seleccione **De VNet a VNet**.
6. En **Segunda puerta enlace de red virtual**, seleccione **GW-hub**.
7. En **Clave compartida (PSK)** , escriba **AzureA1b2C3**.
8. Seleccione **Aceptar**.


#### <a name="verify-the-connection"></a>Comprobación de la conexión

Después de unos cinco minutos o más, el estado de ambas conexiones debe ser **conectado**.

![Conexiones de puerta de enlace](media/tutorial-hybrid-portal/gateway-connections.png)

## <a name="peer-the-hub-and-spoke-virtual-networks"></a>Emparejar las redes virtuales de tipo hub-and-spoke

Ahora, empareje las redes virtuales de tipo hub-and-spoke.

1. Abra el grupo de recursos **FW-Hybrid-Test** y seleccione la red virtual **VNet-hub**.
2. En la columna izquierda, seleccione **Emparejamientos**.
3. Seleccione **Agregar**.
4. En **Esta red virtual**:
 
   
   |Nombre del valor  |Value  |
   |---------|---------|
   |Nombre del vínculo de emparejamiento| HubtoSpoke|
   |Tráfico hacia la red virtual remota|   Permitir (predeterminado)      |
   |Tráfico reenviado desde la red virtual remota    |   Permitir (predeterminado)      |
   |Puerta de enlace de red virtual     |  Usar la puerta de enlace de esta red virtual       |
    
5. En **Red virtual remota**:

   |Nombre del valor  |Value  |
   |---------|---------|
   |Nombre del vínculo de emparejamiento | SpoketoHub|
   |Modelo de implementación de red virtual| Resource Manager|
   |Suscripción|\<your subscription\>|
   |Virtual network| VNet-Spoke
   |Tráfico hacia la red virtual remota     |   Permitir (predeterminado)      |
   |Tráfico reenviado desde la red virtual remota    |   Permitir (predeterminado)      |
   |Puerta de enlace de red virtual     |  Usar la puerta de enlace de la red virtual remota       |

5. Seleccione **Agregar**.

   :::image type="content" source="media/tutorial-hybrid-portal/firewall-peering.png" alt-text="Emparejamiento de Vnet":::

## <a name="create-the-routes"></a>Creación de las rutas

A continuación, cree un par de rutas:

- Una ruta desde la subred de puerta de enlace de concentrador a la subred de radio mediante la dirección IP del firewall.
- Una ruta predeterminada desde la subred de radio mediante la dirección IP del firewall.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En el cuadro de texto de búsqueda, escriba **tabla de rutas** y presione **Entrar**.
3. Seleccione **Tabla de rutas**.
4. Seleccione **Crear**.
6. Seleccione **FW-Hybrid-Test** en grupo de recursos.
8. En **Región**, seleccione la misma ubicación que usó anteriormente.
1. En el nombre, escriba **UDR-Hub-Spoke**.
9. Seleccione **Revisar + crear**.
10. Seleccione **Crear**.
11. Una vez creada la tabla de rutas, selecciónela para abrir la página de la tabla de rutas.
12. Seleccione **Rutas** en la columna izquierda.
13. Seleccione **Agregar**.
14. En el nombre de ruta, escriba **ToSpoke**.
15. En el prefijo de dirección, escriba **10.6.0.0/16**.
16. En el tipo del próximo salto, seleccione **Aplicación virtual**.
17. En la dirección del próximo salto, escriba la dirección IP privada del firewall que anotó anteriormente.
18. Seleccione **Aceptar**.

Ahora asocie la ruta a la subred.

1. En la página **UDR-Hub-Spoke - Routes**, seleccione **Subredes**.
2. Seleccione **Asociar**.
3. En **Red virtual**, seleccione **VNet-hub**.
1. En **Subred**, seleccione **GatewaySubnet**.
2. Seleccione **Aceptar**.

Ahora, cree la ruta predeterminada desde la subred radial.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En el cuadro de texto de búsqueda, escriba **tabla de rutas** y presione **Entrar**.
3. Seleccione **Tabla de rutas**.
5. Seleccione **Crear**.
7. Seleccione **FW-Hybrid-Test** en grupo de recursos.
8. En **Región**, seleccione la misma ubicación que usó anteriormente.
1. En el nombre, escriba **UDR-DG**.
4. En **Propagar la ruta de la puerta de enlace**, seleccione **No**.
5. Seleccione **Revisar + crear**.
6. Seleccione **Crear**.
7. Una vez creada la tabla de rutas, selecciónela para abrir la página de la tabla de rutas.
8. Seleccione **Rutas** en la columna izquierda.
9. Seleccione **Agregar**.
10. En el nombre de ruta, escriba **ToHub**.
11. En el prefijo de dirección, escriba **0.0.0.0/0**.
12. En el tipo del próximo salto, seleccione **Aplicación virtual**.
13. En la dirección del próximo salto, escriba la dirección IP privada del firewall que anotó anteriormente.
14. Seleccione **Aceptar**.

Ahora asocie la ruta a la subred.

1. En la página **UDR-DG - Routes**, seleccione **Subredes**.
2. Seleccione **Asociar**.
3. En **Red virtual**, seleccione **VNet-spoke**.
1. En **Subred**, seleccione **SN-Workload**.
2. Seleccione **Aceptar**.

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Ahora, cree la carga de trabajo de tipo hub-and-spoke y las máquinas virtuales locales, y colóquelas en las subredes adecuadas.

### <a name="create-the-workload-virtual-machine"></a>Creación de la máquina virtual de cargas de trabajo

Cree una máquina virtual en la red virtual radial, que ejecute IIS, sin ninguna dirección IP pública.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En la página **Popular**, seleccione **Windows Server 2016 Datacenter**.
3. Especifique estos valores para la máquina virtual:
    - **Grupo de recursos**: seleccione **FW-Hybrid-Test**.
    - **Nombre de la máquina virtual**: *VM-Spoke-01*.
    - **Región**: la misma región que usó anteriormente.
    - **Nombre de usuario**: \<type a user name\>.
    - **Contraseña**: \<type a password\>
4. En **Puertos de entrada públicos**, seleccione **Permitir los puertos seleccionados** y, después, seleccione **HTTP (80)** y **RDP (3389)** .
4. Seleccione **Siguiente: Discos**.
5. Acepte los valores predeterminados y seleccione **Siguiente: Redes**.
6. Seleccione **VNet-Spoke** para la red virtual y la subred es **SN-Workload**.
7. En **IP pública**, seleccione **Ninguno**. 
9. Seleccione **Siguiente: Administración**.
10. En **Diagnósticos de arranque**, seleccione **Deshabilitado**.
11. Seleccione **Revisar y crear**, revise la configuración en la página de resumen y, después, seleccione **Crear**.

### <a name="install-iis"></a>Instalación de IIS

1. En Azure Portal, abra Cloud Shell y asegúrese de que está establecido en **PowerShell**.
2. Ejecute el siguiente comando para instalar IIS en la máquina virtual; cambie la ubicación si es necesario:

   ```azurepowershell-interactive
   Set-AzVMExtension `
           -ResourceGroupName FW-Hybrid-Test `
           -ExtensionName IIS `
           -VMName VM-Spoke-01 `
           -Publisher Microsoft.Compute `
           -ExtensionType CustomScriptExtension `
           -TypeHandlerVersion 1.4 `
           -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell      Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
           -Location EastUS
   ```

### <a name="create-the-on-premises-virtual-machine"></a>Creación de la máquina virtual local

Se trata de una máquina virtual con una dirección IP pública a la que se puede conectar mediante Escritorio remoto. Desde allí, puede conectarse al servidor local a través del firewall.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En la página **Popular**, seleccione **Windows Server 2016 Datacenter**.
3. Especifique estos valores para la máquina virtual:
    - **Grupo de recursos**: seleccione el existente y, después, **FW-Hybrid-Test**.
    - **Nombre de máquina virtual** - *VM-Onprem*.
    - **Región**: la misma región que usó anteriormente.
    - **Nombre de usuario**: \<type a user name\>.
    - **Contraseña**: \<type a user password\>.
7. En **Puertos de entrada públicos**, seleccione **Permitir los puertos seleccionados** y, después, seleccione **RDP (3389)** .
4. Seleccione **Siguiente: Discos**.
5. Acepte los valores predeterminados y seleccione **Siguiente: Redes**.
6. Seleccione **VNet-Onprem** para red virtual y la subred es **SN-Corp**.
8. Seleccione **Siguiente: Administración**.
10. En **Diagnósticos de arranque**, seleccione **Deshabilitado**.
10. Seleccione **Revisar y crear**, revise la configuración en la página de resumen y, después, seleccione **Crear**.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="test-the-firewall"></a>Probar el firewall

1. En primer lugar, anote la dirección IP privada de la máquina virtual **VM-spoke-01**.

2. En Azure Portal, conéctese a la máquina virtual **VM-Onprem**.
<!---2. Open a Windows PowerShell command prompt on **VM-Onprem**, and ping the private IP for **VM-spoke-01**.

   You should get a reply.--->
3. Abra un explorador web en **VM-Onprem** y vaya a http://\<VM-spoke-01 private IP\>.

   Debería ver la página web de **VM-spoke-01**: ![Página web VM-Spoke-01](media/tutorial-hybrid-portal/VM-Spoke-01-web.png)

4. En la máquina virtual **VM-Onprem**, abra un escritorio remoto a **VM-spoke-01** en la dirección IP privada.

   Se realizará la conexión y podrá iniciar sesión.

Con ello, ha comprobado que las reglas de firewall funcionan:

<!---- You can ping the server on the spoke VNet.--->
- Puede examinar el servidor web en la red virtual de tipo hub-and-spoke.
- Puede conectarse al servidor en la red virtual de tipo hub-and-spoke mediante RDP.

A continuación, cambie la acción de recopilación de la regla de red del firewall a **Deny** (Denegar) para comprobar que las reglas del firewall funcionan según lo previsto.

1. Seleccione el firewall **AzFW01**.
2. Seleccione **Reglas**.
3. Seleccione la pestaña **Recopilación de reglas de red** y seleccione la colección de reglas **RCNet01**.
4. En **Acción**, seleccione **Denegar**.
5. Seleccione **Guardar**.

Cierre los escritorios remotos existentes antes de probar las reglas modificadas. Ahora, ejecute de nuevo las pruebas. Esta vez, todas producirán un error.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede conservar los recursos de firewall para probarlos más a fondo o, si ya no los necesita, eliminar el grupo de recursos **FW-Hybrid-Test** para eliminarlos todos.

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede supervisar los registros de Azure Firewall.

[Tutorial: Supervisión de los registros de Azure Firewall](./firewall-diagnostics.md)