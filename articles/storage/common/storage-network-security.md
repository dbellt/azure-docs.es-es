---
title: Configuración de redes virtuales y firewalls de Azure Storage | Microsoft Docs
description: Configure la seguridad de red por capas para la cuenta de almacenamiento mediante los firewalls de Azure Storage y Azure Virtual Network.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 03/16/2021
ms.author: normesta
ms.reviewer: santoshc
ms.subservice: common
ms.openlocfilehash: 51e0205fddf9c3e6e3d622465204e647c0099f53
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715833"
---
# <a name="configure-azure-storage-firewalls-and-virtual-networks"></a>Configuración de redes virtuales y firewalls de Azure Storage

Azure Storage proporciona un modelo de seguridad por capas. Este modelo le permite proteger y controlar el nivel de acceso a las cuentas de almacenamiento que exigen sus aplicaciones y entornos empresariales, en función del tipo y el subconjunto de redes o recursos usados. Cuando se configuran las reglas de red, solo las aplicaciones que solicitan datos en el conjunto especificado de redes o a través del conjunto especificado de recursos de Azure pueden acceder a una cuenta de almacenamiento. Puede limitar el acceso a su cuenta de almacenamiento a las solicitudes procedentes de direcciones IP especificadas, intervalos IP, subredes de Azure Virtual Network (VNet) o instancias de recursos de algunos servicios de Azure.

Las cuentas de almacenamiento tienen un punto de conexión público accesible a través de Internet. También puede crear [puntos de conexión privados para la cuenta de almacenamiento](storage-private-endpoints.md), lo que asigna una dirección IP privada de la red virtual a la cuenta de almacenamiento, y protege todo el tráfico entre la red virtual y la cuenta de almacenamiento a través de un vínculo privado. El firewall de almacenamiento de Azure proporciona control de acceso para el punto de conexión público de la cuenta de almacenamiento. También puede usar el firewall para bloquear todo el acceso a través del punto de conexión público al usar puntos de conexión privados. La configuración del firewall de almacenamiento también permite seleccionar servicios de la plataforma de Azure de confianza para acceder a la cuenta de almacenamiento de forma segura.

Una aplicación que accede a una cuenta de almacenamiento cuando las reglas de red están en vigor aún requiere la autorización adecuada para la solicitud. La autorización es compatible con las credenciales de Azure Active Directory (Azure AD) (para blobs y colas), con una clave de acceso de cuenta válida o un token de SAS.

> [!IMPORTANT]
> La activación de las reglas de firewall para la cuenta de almacenamiento bloquea las solicitudes entrantes para los datos de forma predeterminada, a menos que las solicitudes procedan de un servicio que funcione en una instancia de Azure Virtual Network (VNet) o desde direcciones IP públicas permitidas. Las solicitudes que bloquean incluyen aquellas de otros servicios de Azure, desde Azure Portal, desde los servicios de registro y de métricas, etc.
>
> Puede conceder acceso a los servicios de Azure que funcionan desde una VNet al permitir el tráfico de la subred que hospeda la instancia de servicio. Puede habilitar también un número limitado de escenarios mediante el mecanismo de excepciones que se describe más adelante. Para acceder a los datos de la cuenta de almacenamiento mediante Azure Portal, deberá estar en una máquina situada dentro del límite de confianza (IP o red virtual) que haya configurado.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="scenarios"></a>Escenarios

Para proteger la cuenta de almacenamiento, primero debe configurar una regla para denegar el acceso al tráfico desde todas las redes (incluido el tráfico de Internet) en el punto de conexión público de forma predeterminada. A continuación, debe configurar las reglas que conceden acceso al tráfico desde redes virtuales específicas. También puede configurar reglas para conceder acceso al tráfico desde intervalos de direcciones IP de Internet públicos seleccionados, lo que permite habilitar conexiones desde clientes locales o específicos de Internet. Esta configuración le permite crear un límite de red seguro para las aplicaciones.

Puede combinar reglas de firewall que permitan el acceso desde redes virtuales específicas y desde intervalos de direcciones IP públicas en la misma cuenta de almacenamiento. Las reglas de firewall de almacenamiento se pueden aplicar a cuentas de almacenamiento existentes o al crear nuevas cuentas de almacenamiento.

Las reglas de firewall de almacenamiento se aplican al punto de conexión público de una cuenta de almacenamiento. No se necesitan reglas de acceso de firewall para permitir el tráfico de los puntos de conexión privados de una cuenta de almacenamiento. El proceso de aprobación de la creación de un punto de conexión privado concede acceso implícito al tráfico desde la subred que hospeda el punto de conexión privado.

Se aplican reglas de red en todos los protocolos de red para Azure Storage, incluidos REST y SMB. Para tener acceso a los datos mediante herramientas como Azure Portal, el Explorador de Storage y AZCopy, deben configurarse reglas de red explícitas.

Una vez que se apliquen las reglas de red, se aplicarán a todas las solicitudes. Los tokens de SAS que conceden acceso a una dirección IP específica sirven para limitar el acceso del titular del token, pero no conceden un acceso nuevo más allá de las reglas de red configuradas.

El tráfico de disco de máquina virtual (incluidas las operaciones de montaje y desmontaje y las operaciones de E/S de disco) no se ve afectado por las reglas de red. El acceso de REST a los blobs en páginas está protegido por las reglas de red.

Las cuentas de almacenamiento clásicas no admiten firewalls y redes virtuales.

Puede usar discos no administrados en cuentas de almacenamiento con reglas de red aplicadas para crear copias de seguridad y restaurar máquinas virtuales mediante la creación de una excepción. Este proceso se documenta en la sección [Administración de excepciones](#manage-exceptions) de este artículo. Las excepciones del firewall no se aplican a los discos administrados, puesto que ya son administrados por Azure.

## <a name="change-the-default-network-access-rule"></a>Modificación de la regla de acceso de red predeterminada

De forma predeterminada, las cuentas de almacenamiento aceptan conexiones de clientes en cualquier red. Para limitar el acceso a redes seleccionadas, primero debe cambiar la acción predeterminada.

> [!WARNING]
> La realización de cambios en reglas de red puede afectar a la capacidad de las aplicaciones de conexión a Azure Storage. Si se establece la regla de red predeterminada en **denegar**, se bloquea el acceso a los datos, a no ser que se apliquen también las reglas de red específicas para **conceder** acceso. Asegúrese de conceder acceso a las redes permitidas con reglas de red antes de cambiar la regla predeterminada para denegar el acceso.

### <a name="managing-default-network-access-rules"></a>Administración de las reglas de acceso de red predeterminadas

Puede administrar las reglas predeterminadas de acceso a redes para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a la cuenta de almacenamiento que quiere proteger.

2. Realice la selección en el menú de configuración denominado **Redes**.

3. Para denegar el acceso de forma predeterminada, elija permitir el acceso desde **Redes seleccionadas**. Para permitir el tráfico desde todas las redes, elija permitir el acceso desde **Todas las redes**.

4. Seleccione **Guardar** para aplicar los cambios.

<a id="powershell"></a>

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

2. Visualice el estado de la regla predeterminada para la cuenta de almacenamiento.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").DefaultAction
    ```

3. Establezca la regla predeterminada para denegar el acceso de red de forma predeterminada.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Deny
    ```

4. Establezca la regla predeterminada para permitir el acceso de red de forma predeterminada.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Allow
    ```

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

2. Visualice el estado de la regla predeterminada para la cuenta de almacenamiento.

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.defaultAction
    ```

3. Establezca la regla predeterminada para denegar el acceso de red de forma predeterminada.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Deny
    ```

4. Establezca la regla predeterminada para permitir el acceso de red de forma predeterminada.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --default-action Allow
    ```
---

## <a name="grant-access-from-a-virtual-network"></a>Concesión de acceso desde una red virtual

Puede configurar las cuentas de almacenamiento para permitir el acceso solo desde subredes específicas. Las subredes permitidas pueden pertenecer a una red virtual de la misma suscripción o a las de una suscripción diferente, incluidas las suscripciones que pertenecen a un inquilino de Azure Active Directory diferente.

Habilite un [punto de conexión de servicio](../../virtual-network/virtual-network-service-endpoints-overview.md) para Azure Storage dentro de la red virtual. El punto de conexión de servicio enruta el tráfico desde la red virtual a través de una ruta de acceso óptima al servicio Azure Storage. Las identidades de la red virtual y la subred también se transmiten con cada solicitud. Luego, los administradores pueden configurar reglas de red para la cuenta de almacenamiento que permitan que se reciban solicitudes desde subredes específicas en una red virtual. Los clientes a los que se concedió acceso a través de estas reglas de red deben seguir cumpliendo los requisitos de autorización de la cuenta de almacenamiento para acceder a los datos.

Cada cuenta de almacenamiento admite un máximo de 200 reglas de red virtual, que se pueden combinar con [reglas de red IP](#grant-access-from-an-internet-ip-range).

### <a name="available-virtual-network-regions"></a>Regiones de red virtual disponibles

En general, los puntos de conexión de servicio funcionan entre redes virtuales e instancias de servicio en la misma región de Azure. Cuando se usan los puntos de conexión de servicio con Azure Storage, este ámbito crece para incluir la [región emparejada](../../best-practices-availability-paired-regions.md). Los puntos de conexión de servicio permiten la continuidad durante una conmutación por error regional, así como un acceso a las instancias de almacenamiento con redundancia geográfica de solo lectura (RA-GRS). Las reglas de red que conceden acceso de una red virtual a una cuenta de almacenamiento también conceden acceso a cualquier instancia de RA-GRS.

Al planear la recuperación ante desastres durante una interrupción regional, debe crear las redes virtuales en la región emparejada por adelantado. Habilite puntos de conexión de servicio para Azure Storage, con reglas de red que concedan acceso desde estas redes virtuales alternativas. A continuación, aplique estas reglas a las cuentas de almacenamiento con redundancia geográfica.

> [!NOTE]
> Los puntos de conexión de servicio no se aplican al tráfico fuera de la región de la red virtual y el par de región designado. Solo puede aplicar reglas de red que concedan acceso desde redes virtuales a las cuentas de almacenamiento en la región principal de una cuenta de almacenamiento o en la región emparejada designada.

### <a name="required-permissions"></a>Permisos necesarios

Para aplicar una regla de red virtual a una cuenta de almacenamiento, el usuario debe tener permisos apropiados para las subredes que se van a agregar. Un [colaborador de la cuenta de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-account-contributor) o un usuario al que se haya concedido permiso para la [operación del proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md#microsoftnetwork) `Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action` a través de un rol personalizado de Azure puede aplicar una regla.

La cuenta de almacenamiento y las redes virtuales a las que se concedió acceso pueden estar en distintas suscripciones, incluidas suscripciones que formen parte del un inquilino de Azure AD diferente.

> [!NOTE]
> La configuración de reglas que conceden acceso a subredes en redes virtuales que forman parte de un inquilino de Azure Active Directory diferente solo se admiten actualmente a través de PowerShell, la CLI y las API REST. Estas reglas no se pueden configurar mediante Azure Portal, aunque se puedan ver en el portal.

### <a name="managing-virtual-network-rules"></a>Administración de reglas de red virtual

Puede administrar las reglas de red virtual para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a la cuenta de almacenamiento que quiere proteger.

2. Realice la selección en el menú de configuración denominado **Redes**.

3. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

4. Para conceder acceso a una red virtual con una nueva regla de red, en **Redes virtuales**, seleccione **Agregar red virtual existente**, seleccione las opciones **Redes virtuales** y **Subredes** y, luego, seleccione **Agregar**. Para crear una nueva red virtual y concederle acceso, seleccione **Agregar nueva red virtual**. Proporcione la información necesaria para crear la nueva red virtual y, luego, seleccione **Crear**.

    > [!NOTE]
    > Si un punto de conexión de servicio para Azure Storage no se ha configurado previamente para la red virtual y las subredes seleccionadas, se puede configurar como parte de esta operación.
    >
    > Actualmente, solo se muestran las redes virtuales que pertenecen al mismo inquilino de Azure Active Directory para su selección durante la creación de la regla. Para conceder acceso a una subred de una red virtual que pertenece a otro inquilino, use PowerShell, la CLI o la API REST.

5. Para quitar una regla de red virtual o subred, seleccione **...** para abrir el menú contextual de la red virtual o la subred y seleccione **Quitar**.

6. Seleccione **Guardar** para aplicar los cambios.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

2. Enumere las reglas de red virtual.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").VirtualNetworkRules
    ```

3. Habilite el punto de conexión de servicio para Azure Storage en una red virtual y subred existentes.

    ```powershell
    Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.0.0.0/24" -ServiceEndpoint "Microsoft.Storage" | Set-AzVirtualNetwork
    ```

4. Agregue una regla de red para una red virtual y subred.

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

    > [!TIP]
    > Para agregar una regla de red para una subred de una red virtual que pertenezca a otro inquilino de Azure AD, use un parámetro completo **VirtualNetworkResourceId** con el formato "/subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name".

5. Quite una regla de red para una red virtual y subred.

    ```powershell
    $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

2. Enumere las reglas de red virtual.

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query virtualNetworkRules
    ```

3. Habilite el punto de conexión de servicio para Azure Storage en una red virtual y subred existentes.

    ```azurecli
    az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.Storage"
    ```

4. Agregue una regla de red para una red virtual y subred.

    ```azurecli
    subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

    > [!TIP]
    > Para agregar una regla para una subred de una red virtual que pertenezca a otro inquilino de Azure AD, use un identificador de subred completo con el formato "/subscriptions/\<subscription-ID\>/resourceGroups/\<resourceGroup-Name\>/providers/Microsoft.Network/virtualNetworks/\<vNet-name\>/subnets/\<subnet-name\>".
    >
    > Puede usar el parámetro **subscription** para recuperar el identificador de subred de una red virtual que pertenezca a otro inquilino de Azure AD.

5. Quite una regla de red para una red virtual y subred.

    ```azurecli
    subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

---

## <a name="grant-access-from-an-internet-ip-range"></a>Concesión de acceso desde un intervalo IP de Internet

Puede usar las reglas de red IP para permitir el acceso desde intervalos específicos de direcciones IP de la red pública de Internet mediante la creación de reglas de red IP. Cada cuenta de almacenamiento admite hasta 200 reglas. Estas reglas conceden acceso a servicios específicos basados en Internet y redes locales, y bloquean el tráfico de Internet general.

Las restricciones siguientes se aplican a los intervalos de direcciones IP.

- Las reglas de red IP solo se permiten para direcciones IP de la **red pública de Internet**. 

  No se permiten intervalos de direcciones IP reservados para redes privadas (tal y como se define en [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3)) en las reglas de IP. Las redes privadas incluyen direcciones que comienzan por _10.*_ , _172.16.*_  - _172.31.*_ y _192.168.*_ .

- Debe proporcionar los intervalos de dirección de Internet permitidos mediante la [notación CIDR](https://tools.ietf.org/html/rfc4632) en formato *16.17.18.0/24* o como direcciones IP individuales en formato *16.17.18.19*. 

- Los intervalos de dirección pequeños con tamaños de prefijos "/31" o "/32" no son compatibles. Estos intervalos se deberían configurar utilizando reglas de direcciones IP individuales. 

- Solo se admiten direcciones IPV4 para la configuración de reglas de firewall de almacenamiento.

Las reglas de red IP no se pueden usar en los siguientes casos:

- Para restringir el acceso a los clientes de la misma región de Azure que la cuenta de almacenamiento.
  
  Las reglas de red IP no tienen ningún efecto en las solicitudes que proceden de la misma región de Azure que la cuenta de almacenamiento. Use [reglas de red virtual](#grant-access-from-a-virtual-network) para permitir solicitudes de la misma región. 

- Para restringir el acceso a los clientes de una [región emparejada](../../best-practices-availability-paired-regions.md) que se encuentran en una red virtual que tiene un punto de conexión de servicio.

- Para restringir el acceso a los servicios de Azure implementados en la misma región que la cuenta de almacenamiento.

  Los servicios implementados en la misma región que la cuenta de almacenamiento usan direcciones IP privadas de Azure para la comunicación. Por lo tanto, no puede restringir el acceso a servicios específicos de Azure en función de su intervalo de direcciones IP salientes públicas.

### <a name="configuring-access-from-on-premises-networks"></a>Configuración del acceso desde redes locales

Para conceder acceso desde las redes locales a la cuenta de almacenamiento con una regla de red IP, debe identificar las direcciones IP orientadas a Internet que usa su red. Para obtener ayuda, póngase en contacto con el administrador de red.

Si usa [ExpressRoute](../../expressroute/expressroute-introduction.md) desde las instalaciones para pares públicos o el emparejamiento de Microsoft, tiene que identificar las direcciones IP de NAT que se usan. Para el emparejamiento público, cada circuito ExpressRoute usa de forma predeterminada dos direcciones IP de NAT que se aplican al tráfico del servicio de Azure cuando el tráfico entra en la red troncal de Microsoft Azure. Para el emparejamiento de Microsoft, el cliente o el proveedor de servicios proporciona las direcciones IP de NAT que se utilizan. Para permitir el acceso a los recursos de servicio, tiene que permitir estas direcciones IP públicas en la configuración del firewall de IP de recursos. Para encontrar las direcciones de IP de circuito de ExpressRoute de los pares públicos, [abra una incidencia de soporte técnico con ExpressRoute](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) a través de Azure Portal. Obtenga más información sobre [NAT para ExpressRoute para emparejamiento público y de Microsoft.](../../expressroute/expressroute-nat.md#nat-requirements-for-azure-public-peering)

### <a name="managing-ip-network-rules"></a>Administración de reglas de red IP

Puede administrar las reglas de red IP para las cuentas de almacenamiento a través de Azure Portal, PowerShell o CLIv2.

#### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a la cuenta de almacenamiento que quiere proteger.

2. Realice la selección en el menú de configuración denominado **Redes**.

3. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

4. Para conceder acceso al intervalo IP de Internet, escriba la dirección IP o el intervalo de direcciones (en formato CIDR) en **Firewall** > **Intervalo de direcciones**.

5. Para quitar una regla de red IP, seleccione el icono de la papelera junto al intervalo de direcciones.

6. Seleccione **Guardar** para aplicar los cambios.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

2. Enumere las reglas de red IP.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").IPRules
    ```

3. Agregue una regla de red para una dirección IP individual.

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

4. Agregue una regla de red para un intervalo de direcciones IP.

    ```powershell
    Add-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

5. Quite una regla de red para una dirección IP individual.

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"
    ```

6. Quite una regla de red para un intervalo de direcciones IP.

    ```powershell
    Remove-AzStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

1. Enumere las reglas de red IP.

    ```azurecli
    az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query ipRules
    ```

2. Agregue una regla de red para una dirección IP individual.

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

3. Agregue una regla de red para un intervalo de direcciones IP.

    ```azurecli
    az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

4. Quite una regla de red para una dirección IP individual.

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
    ```

5. Quite una regla de red para un intervalo de direcciones IP.

    ```azurecli
    az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

---

<a id="grant-access-specific-instances"></a>

## <a name="grant-access-from-azure-resource-instances-preview"></a>Concesión de acceso a instancias de recursos de Azure (versión preliminar)

En algunos casos, una aplicación puede depender de recursos de Azure que no se pueden aislar a través de una regla de red virtual o de dirección IP. Sin embargo, todavía le gustaría proteger y restringir el acceso de la cuenta de almacenamiento solo a los recursos de Azure de la aplicación. Puede configurar cuentas de almacenamiento para permitir el acceso a instancias de recursos específicas de algunos servicios de Azure mediante la creación de una regla de instancia de recurso. 

Los tipos de operaciones que puede realizar una instancia de recursos en los datos de la cuenta de almacenamiento se determinan mediante las [asignaciones de roles de Azure](storage-auth-aad.md#assign-azure-roles-for-access-rights) de la instancia de recurso. Las instancias de recursos deben estar en el mismo inquilino que la cuenta de almacenamiento, pero pueden pertenecer a cualquier suscripción del inquilino.

> [!NOTE]
> Esta característica está en versión preliminar pública y está disponible en todas las regiones de la nube pública.

> [!NOTE]
> Actualmente, las reglas de instancia de recurso solo se admiten en Azure Synapse. La compatibilidad con otros servicios de Azure que aparecen en la sección [Acceso de confianza basado en la identidad administrada asignada por el sistema](#trusted-access-system-assigned-managed-identity) de este artículo estará disponible en las próximas semanas.


### <a name="portal"></a>[Portal](#tab/azure-portal)

Puede agregar o quitar reglas de red de recursos en Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) para empezar a trabajar.

2. Busque la cuenta de almacenamiento y muestre la información general de la cuenta.

3. Seleccione **Redes** para mostrar la página de configuración de redes.

4. En la lista desplegable **Tipo de recurso**, elija el tipo de recurso de la instancia de recurso. 

5. En la lista desplegable **Nombre de instancia**, elija la instancia de recurso. También puede elegir incluir todas las instancias de recursos en el inquilino, la suscripción o el grupo de recursos activos.

6. Seleccione **Guardar** para aplicar los cambios. La instancia de recurso aparece en la sección **Instancias de recursos** de la página de configuración de red. 

Para quitar la instancia de recurso, seleccione el icono de eliminación (:::image type="icon" source="media/storage-network-security/delete-icon.png":::) junto a la instancia de recurso.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Puede usar comandos de PowerShell para agregar o quitar reglas de red de recursos.

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, las reglas de red no tendrán ningún efecto.

#### <a name="install-the-preview-module"></a>Instalación del módulo de versión preliminar

Instale la versión más reciente del módulo PowerShellGet. Luego, cierre la consola de PowerShell y vuelva a abrirla.

```powershell
install-Module PowerShellGet –Repository PSGallery –Force  
```

Instale el módulo en versión preliminar **Az. Storage**.

```powershell
Install-Module Az.Storage -Repository PsGallery -RequiredVersion 3.0.1-preview -AllowClobber -AllowPrerelease -Force 
```

Para más información sobre cómo instalar módulos de PowerShell, vea [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).

#### <a name="grant-access"></a>Conceder acceso

Agregue una regla de red que conceda acceso desde una instancia de recurso.

```powershell
$resourceId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.DataFactory/factories/myDataFactory"
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Add-AzStorageAccountNetworkRule -ResourceGroupName $resourceGroupName -Name $accountName -TenantId $tenantId -ResourceId $resourceId

```

Especifique varias instancias de recursos a la vez modificando el conjunto de reglas de red.

```powershell
$resourceId1 = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.DataFactory/factories/myDataFactory"
$resourceId2 = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/mySQLServer"
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Update-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName -ResourceAccessRule (@{ResourceId=$resourceId1;TenantId=$tenantId},@{ResourceId=$resourceId2;TenantId=$tenantId}) 
```

#### <a name="remove-access"></a>Eliminación de acceso

Quite una regla de red que conceda acceso desde una instancia de recurso.

```powershell
$resourceId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.DataFactory/factories/myDataFactory"
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Remove-AzStorageAccountNetworkRule -ResourceGroupName $resourceGroupName -Name $accountName -TenantId $tenantId -ResourceId $resourceId  
```

Quite todas las reglas de red que conceden acceso desde instancias de recursos.

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

Update-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName -ResourceAccessRule @()  
```

#### <a name="view-a-list-of-allowed-resource-instances"></a>Visualización de una lista de instancias de recursos permitidas

Vea una lista completa de las instancias de recursos a las que se ha concedido acceso para la cuenta de almacenamiento.

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mystorageaccount"

$rule = Get-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName
$rule.ResourceAccessRules 
```

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede usar comandos de la CLI de Azure para agregar o quitar reglas de red de recursos.

#### <a name="install-the-preview-extension"></a>Instale la extensión en versión preliminar

1. Abra [Azure Cloud Shell](../../cloud-shell/overview.md) o, si ha [instalado](/cli/azure/install-azure-cli) la CLI de Azure localmente, abra una aplicación de consola de comandos como Windows PowerShell.

2. A continuación, use el comando siguiente para comprobar que la versión de la CLI de Azure que ha instalado sea `2.13.0` o posterior.

   ```azurecli
   az --version
   ```

   Si la versión de la CLI de Azure es anterior a `2.13.0`, instale una versión posterior. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

3. Escriba el siguiente comando para instalar la extensión en versión preliminar.

   ```azurecli
   az extension add -n storage-preview
   ```

#### <a name="grant-access"></a>Conceder acceso

Agregue una regla de red que conceda acceso desde una instancia de recurso.

```azurecli
az storage account network-rule add \
    --resource-id /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Synapse/workspaces/testworkspace \
    --tenant-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    -g myResourceGroup \
    --account-name mystorageaccount
```

#### <a name="remove-access"></a>Eliminación de acceso

Quite una regla de red que conceda acceso desde una instancia de recurso.

```azurecli
az storage account network-rule remove \
    --resource-id /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Synapse/workspaces/testworkspace \
    --tenant-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    -g myResourceGroup \
    --account-name mystorageaccount
```

#### <a name="view-a-list-of-allowed-resource-instances"></a>Visualización de una lista de instancias de recursos permitidas

Vea una lista completa de las instancias de recursos a las que se ha concedido acceso para la cuenta de almacenamiento.

```azurecli
az storage account network-rule list \
    -g myResourceGroup \
    --account-name mystorageaccount
```

---

<a id="exceptions"></a>
<a id="trusted-microsoft-services"></a>

## <a name="grant-access-to-trusted-azure-services"></a>Concesión de acceso a servicios de Azure de confianza 

Algunos servicios de Azure funcionan desde redes que no se pueden incluir en las reglas de red. Puede conceder a un subconjunto de estos servicios de Azure de confianza el acceso a la cuenta de almacenamiento, a la vez que mantiene las reglas de red para otras aplicaciones. Estos servicios de confianza usarán una autenticación sólida para conectarse a su cuenta de almacenamiento de forma segura.

Puede conceder acceso a servicios de Azure de confianza mediante la creación de una excepción de regla de red. Para obtener instrucciones paso a paso, consulte la sección [Administración de excepciones](#manage-exceptions) de este artículo.

Cuando se concede acceso a los servicios de Azure de confianza, se conceden los siguientes tipos de acceso:

- Acceso de confianza para las operaciones seleccionadas en los recursos registrados en la suscripción.
- Acceso de confianza a los recursos basándose en la identidad administrada asignada por el sistema.

<a id="trusted-access-resources-in-subscription"></a>

### <a name="trusted-access-for-resources-registered-in-your-subscription"></a>Acceso de confianza para los recursos registrados en su suscripción

Los recursos de algunos servicios, **cuando están registrados en su suscripción**, pueden acceder a su cuenta de almacenamiento **de la misma suscripción** para ciertas operaciones, como la escritura de registros o la realización de copias de seguridad.  En la tabla siguiente se describe cada servicio y las operaciones permitidas. 

| Servicio                  | Nombre del proveedor de recursos     | Operaciones permitidas                 |
|:------------------------ |:-------------------------- |:---------------------------------- |
| Azure Backup             | Microsoft.RecoveryServices | Ejecute copias de seguridad y restauraciones de discos no administrados en máquinas virtuales de IAAS. (no se necesita para discos administrados). [Más información](../../backup/backup-overview.md). |
| Azure Data Box           | Microsoft.DataBox          | Permite la importación de datos en Azure mediante Data Box. [Más información](../../databox/data-box-overview.md). |
| Azure DevTest Labs       | Microsoft.DevTestLab       | Creación de imagen personalizada e instalación de artefactos. [Más información](../../devtest-labs/devtest-lab-overview.md). |
| Azure Event Grid         | Microsoft.EventGrid        | Habilite la publicación de eventos de Blob Storage y permita que Event Grid publique en las colas de almacenamiento. Obtenga información sobre los [eventos de Blob Storage](../../event-grid/overview.md#event-sources) y la [publicación en las colas](../../event-grid/event-handlers.md). |
| Azure Event Hubs         | Microsoft.EventHub         | Archivo de datos con Event Hubs Capture. [Más información](../../event-hubs/event-hubs-capture-overview.md). |
| Azure File Sync          | Microsoft.StorageSync      | Permite transformar el servidor de archivos local en una memoria caché para recursos compartidos de archivos de Azure. Permite la sincronización de varios sitios, la recuperación rápida ante desastres y la copia de seguridad en la nube. [Más información](../file-sync/file-sync-planning.md) |
| HDInsight de Azure          | Microsoft.HDInsight        | Aprovisione el contenido inicial del sistema de archivos predeterminado para un nuevo clúster de HDInsight. [Más información](../../hdinsight/hdinsight-hadoop-use-blob-storage.md). |
| Azure Import/Export      | Microsoft.ImportExport     | Habilita la importación de datos a Azure Storage o la exportación de datos desde Azure Storage mediante el servicio de importación y exportación de Azure Storage. [Más información](../../import-export/storage-import-export-service.md).  |
| Azure Monitor            | Microsoft.Insights         | Permite escribir datos de supervisión en una cuenta de almacenamiento protegida, incluidos los registros de recursos, los registros de inicio de sesión y de auditoría de Azure Active Directory y los registros de Microsoft Intune. [Más información](../../azure-monitor/roles-permissions-security.md). |
| Conexión a Azure         | Microsoft.Network          | Almacene y analice los registros de tráfico de red, incluidos los servicios de Network Watcher y Análisis de tráfico. [Más información](../../network-watcher/network-watcher-nsg-flow-logging-overview.md). |
| Azure Site Recovery      | Microsoft.SiteRecovery     | Habilite la replicación para la recuperación ante desastres de máquinas virtuales de IaaS de Azure al usar la caché habilitada para firewall, el origen o las cuentas de almacenamiento de destino.  [Más información](../../site-recovery/azure-to-azure-tutorial-enable-replication.md). |

<a id="trusted-access-system-assigned-managed-identity"></a>

### <a name="trusted-access-based-on-system-assigned-managed-identity"></a>Acceso de confianza basado en la identidad administrada asignada por el sistema

En la tabla siguiente se enumeran los servicios que pueden tener acceso a los datos de la cuenta de almacenamiento si las instancias de recursos de esos servicios reciben el permiso adecuado. Para conceder permiso, debe [asignar explícitamente un rol de Azure](storage-auth-aad.md#assign-azure-roles-for-access-rights) a la [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/overview.md) para cada instancia de recurso. En ese caso, el ámbito de acceso de la instancia corresponde al rol de Azure que se asigna a la identidad administrada. 

> [!TIP]
> La manera recomendada de conceder acceso a recursos específicos es usar reglas de instancia de recurso. Para conceder acceso a instancias de recursos específicas, consulte la sección [Concesión de acceso a instancias de recursos de Azure (versión preliminar)](#grant-access-specific-instances) de este artículo.


| Servicio                        | Nombre del proveedor de recursos                 | Propósito            |
| :----------------------------- | :------------------------------------- | :----------------- |
| Azure API Management           | Microsoft.ApiManagement/service        | Habilita el acceso del servicio API Management a las cuentas de almacenamiento detrás del firewall mediante directivas. [Más información](../../api-management/api-management-authentication-policies.md#use-managed-identity-in-send-request-policy). |
| Azure Cognitive Search         | Microsoft.Search/searchServices        | Habilita los servicios de Cognitive Search para acceder a las cuentas de almacenamiento con fines de indexación, proceso y consulta. |
| Azure Cognitive Services       | Microsoft.CognitiveService/accounts    | Habilita Cognitive Services para acceder a las cuentas de almacenamiento. |
| Tareas de Azure Container Registry | Microsoft.ContainerRegistry/registries | ACR Tasks puede acceder a las cuentas de almacenamiento al compilar imágenes de contenedor. |
| Azure Data Factory             | Microsoft.DataFactory/factories        | Permite el acceso a las cuentas de almacenamiento a través del tiempo de ejecución de ADF. |
| Azure Data Share               | Microsoft.DataShare/accounts           | Permite el acceso a las cuentas de almacenamiento a través de Data Share. |
| Azure DevTest Labs             | Microsoft.DevTestLab/labs              | Permite el acceso a las cuentas de almacenamiento a través de DevTest Labs. |
| Azure IoT Hub                  | Microsoft.Devices/IotHubs              | Permite que los datos de un centro de IoT se escriban en almacenamiento de blobs. [Más información](../../iot-hub/virtual-network-support.md#egress-connectivity-from-iot-hub-to-other-azure-resources) |
| Azure Logic Apps               | Microsoft.Logic/workflows              | Permite a las aplicaciones lógicas acceder a las cuentas de almacenamiento. [Más información](../../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity). |
| Servicio Azure Machine Learning | Microsoft.MachineLearningServices      | Las áreas de trabajo autorizadas de Azure Machine Learning escriben los resultados del experimento, los modelos y los registros en Blob Storage y leen los datos. [Más información](../../machine-learning/how-to-network-security-overview.md#secure-the-workspace-and-associated-resources). |
| Azure Media Services           | Microsoft.Media/mediaservices          | Permite el acceso a las cuentas de almacenamiento a través de Media Services. |
| Azure Migrate                  | Microsoft.Migrate/migrateprojects      | Permite el acceso a las cuentas de almacenamiento a través de Azure Migrate. |
| Azure Purview                  | Microsoft.Purview/accounts             | Permite que Purview acceda a las cuentas de almacenamiento. |
| Azure Remote Rendering         | Microsoft.MixedReality/remoteRenderingAccounts | Permite el acceso a las cuentas de almacenamiento a través de Remote Rendering. |
| Azure Site Recovery            | Microsoft.RecoveryServices/vaults      | Permite el acceso a las cuentas de almacenamiento a través de Site Recovery. |
| Azure SQL Database             | Microsoft.Sql                          | Permite [escribir](../../azure-sql/database/audit-write-storage-account-behind-vnet-firewall.md) datos de auditoría en cuentas de almacenamiento detrás del firewall. |
| Azure Synapse Analytics        | Microsoft.Sql                          | Permite importar y exportar datos de bases de datos SQL específicas mediante la instrucción COPY o PolyBase (en un grupo dedicado) o la función `openrowset` y las tablas externas del grupo sin servidor. [Más información](../../azure-sql/database/vnet-service-endpoint-rule-overview.md). |
| Azure Stream Analytics         | Microsoft.StreamAnalytics              | Permite que los datos de un trabajo de streaming se escriban en Blob Storage. [Más información](../../stream-analytics/blob-output-managed-identity.md). |
| Azure Synapse Analytics        | Microsoft.Synapse/workspaces           | Permite el acceso a los datos de Azure Storage desde Azure Synapse Analytics. |

## <a name="grant-access-to-storage-analytics"></a>Concesión de acceso a Storage Analytics

En algunos casos, se requiere acceso para leer registros recursos y métricas desde fuera del límite de red. Al configurar el acceso de los servicios de confianza a la cuenta de almacenamiento, puede permitir el acceso de lectura a los archivos de registro, las tablas de métricas o ambos mediante la creación de una excepción de regla de red. Para obtener instrucciones paso a paso, consulte la sección **Administración de excepciones** más adelante. Para más información sobre cómo trabajar con Storage Analytics, consulte [Uso de Azure Storage Analytics para recopilar datos de métricas y registros](./storage-analytics.md). 

<a id="manage-exceptions"></a>

## <a name="manage-exceptions"></a>Administración de excepciones

Puede administrar las excepciones de reglas de red a través de Azure Portal, PowerShell o la CLI de Azure v2.

#### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a la cuenta de almacenamiento que quiere proteger.

2. Realice la selección en el menú de configuración denominado **Redes**.

3. Compruebe haber elegido permitir el acceso desde **Redes seleccionadas**.

4. En **Excepciones**, seleccione las excepciones que quiere conceder.

5. Seleccione **Guardar** para aplicar los cambios.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Instale [Azure PowerShell](/powershell/azure/install-Az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

2. Vea las excepciones para las reglas de red de la cuenta de almacenamiento.

    ```powershell
    (Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount").Bypass
    ```

3. Configure las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass AzureServices,Metrics,Logging
    ```

4. Quite las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```powershell
    Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -Bypass None
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, la eliminación de las excepciones no tendrá ningún efecto.

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

2. Vea las excepciones para las reglas de red de la cuenta de almacenamiento.

    ```azurecli
    az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.bypass
    ```

3. Configure las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass Logging Metrics AzureServices
    ```

4. Quite las excepciones a las reglas de red de la cuenta de almacenamiento.

    ```azurecli
    az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass None
    ```

> [!IMPORTANT]
> Asegúrese de [establecer la regla predeterminada](#change-the-default-network-access-rule) en **denegar** o, de lo contrario, la eliminación de las excepciones no tendrá ningún efecto.

---

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de los puntos de conexión de servicio de red de Azure en [Puntos de conexión de servicio](../../virtual-network/virtual-network-service-endpoints-overview.md).

Profundice en la seguridad de Azure Storage en la [Guía de seguridad de Azure Storage](../blobs/security-recommendations.md).