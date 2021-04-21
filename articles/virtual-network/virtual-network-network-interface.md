---
title: Creación, modificación o eliminación de una interfaz de red de Azure
titlesuffix: Azure Virtual Network
description: Obtenga más información sobre qué es una interfaz de red y cómo crearla, cambiar la configuración y eliminarla.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 1/22/2020
ms.author: kumud
ms.openlocfilehash: 8003bf14bcade08f36a7877fdb3a53998aff9e63
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773077"
---
# <a name="create-change-or-delete-a-network-interface"></a>Creación, cambio o eliminación de una interfaz de red

Aprenda a crear, cambiar la configuración y eliminar una interfaz de red. Una interfaz de red permite que una máquina virtual de Azure se comunique con los recursos de Internet, Azure y locales. Al crear una máquina virtual desde Azure Portal, este crea una interfaz de red con la configuración predeterminada. En su lugar, puede crear interfaces de red con una configuración personalizada y agregar una o varias interfaces de red a una máquina virtual al crearla. También puede cambiar la configuración predeterminada de la interfaz de red en una interfaz de red existente. En este artículo se explica cómo crear una interfaz de red con opciones personalizadas, cambiar la configuración existente, como la asignación de filtros de red (grupo de seguridad de red), la asignación de subredes, la configuración del servidor DNS y el reenvío IP, y, finalmente, cómo eliminar una interfaz de red.

Si tiene que agregar, cambiar o quitar direcciones IP de una interfaz de red, consulte [Administración de direcciones IP](virtual-network-network-interface-addresses.md). Si necesita agregar o quitar interfaces de red en máquinas virtuales, consulte [Adición o eliminación de interfaces de red](virtual-network-network-interface-vm.md).

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Complete las tareas siguientes antes de seguir los pasos de las secciones de este artículo:

- Si todavía no tiene una cuenta de Azure, regístrese para obtener una [cuenta de evaluación gratuita](https://azure.microsoft.com/free).
- Si usa el portal, abra https://portal.azure.com e inicie sesión con la cuenta de Azure.
- Si usa comandos de PowerShell para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/powershell) o ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para realizar este tutorial, se necesita la versión 1.0.0 del módulo de Azure PowerShell u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.
- Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. Para realizar este tutorial es necesaria la versión 2.0.28 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta de forma local la CLI de Azure, también debe ejecutar `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

## <a name="create-a-network-interface"></a>Crear una interfaz de red

Al crear una máquina virtual desde Azure Portal, este crea una interfaz de red con la configuración predeterminada. Si prefiere especificar toda la configuración de la interfaz de red, puede crear una interfaz de red con una configuración personalizada y asociar la interfaz de red a una máquina virtual al crear esta última (con PowerShell o la CLI de Azure). También puede crear una interfaz de red y agregarla a una máquina virtual existente (con PowerShell o la CLI de Azure). Para información sobre cómo crear una máquina virtual con una interfaz de red existente, o cómo agregar o quitar interfaces de red en máquinas virtuales existentes, consulte el artículo [Adición o eliminación de interfaces de red](virtual-network-network-interface-vm.md). En la misma ubicación y suscripción donde vaya a crear la interfaz de red, debe tener previamente una [red virtual](manage-virtual-network.md).

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *interfaces de red*. Cuando aparezcan las **interfaces de red** en los resultados de búsqueda, selecciónelas.
2. Seleccione **+ Agregar** en **Interfaces de red**.
3. Escriba o seleccione valores para las siguientes opciones y seleccione **Crear**:

    |Configuración|¿Necesario?|Detalles|
    |---|---|---|
    |Nombre|Sí|El nombre debe ser único dentro del grupo de recursos que seleccione. Con el tiempo, probablemente tendrá varias interfaces de red en la suscripción de Azure. Para recibir sugerencias sobre la creación de convenciones de nomenclatura para facilitar la administración de varias interfaces de red, consulte [Convenciones de nomenclatura](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#resource-naming). El nombre no se puede cambiar una vez creada la interfaz de red.|
    |Virtual network|Sí|Seleccione la red virtual para la interfaz de red. Solo se puede asignar una interfaz de red a una red virtual que exista en la misma suscripción y ubicación que la interfaz de red. Una vez creada la interfaz de red, no se puede cambiar la red virtual a la que está asignada. La máquina virtual que se agrega a la interfaz de red también debe existir en la misma ubicación y suscripción que la interfaz de red.|
    |Subnet|Sí|Seleccione una subred dentro de la red virtual que seleccionó. Puede cambiar la subred a la que está asignada la interfaz de red después de crearla.|
    |Asignación de la dirección IP privada|Sí| En esta configuración, va a elegir el método de asignación para la dirección IPv4. Elija entre los siguientes métodos de asignación: **Dinámica:** al seleccionar esta opción, Azure asigna automáticamente la siguiente dirección disponible del espacio de direcciones de la subred seleccionada. **Estática:** al seleccionar esta opción, debe asignar manualmente una dirección IP disponible del espacio de direcciones de la subred seleccionada. Las direcciones estática y dinámica no cambian hasta que usted realice algún cambio elimine la interfaz de red. Una vez creada la interfaz de red, puede cambiar el método de asignación. El servidor DHCP de Azure asigna esta dirección a la interfaz de red en el sistema operativo de la máquina virtual.|
    |Grupo de seguridad de red|No| Déjelo establecido en **Ninguno**, seleccione un [grupo de seguridad de red](./network-security-groups-overview.md) existente o [cree uno](tutorial-filter-network-traffic.md). Los grupos de seguridad de red permiten filtrar el tráfico de red hacia y desde una interfaz de red. Puede aplicar un grupo de seguridad de red, o ninguno, a una interfaz de red. También puede aplicar un grupo de seguridad de red, o ninguno, a la subred a la que está asignada la interfaz de red. Cuando un grupo de seguridad de red se aplica a una interfaz de red y a la subred a la que está asignada la interfaz de red, en ocasiones se producen resultados inesperados. Para solucionar problemas de los grupos de seguridad de red aplicados a interfaces de red y subredes, consulte [Solución de problemas de grupos de seguridad de red](diagnose-network-traffic-filter-problem.md).|
    |Subscription|Sí|Seleccione una de las [suscripciones](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) de Azure. La máquina virtual asociada a una interfaz de red y la red virtual a la que está conectada deben existir en la misma suscripción.|
    |Dirección IP privada (IPv6)|No| Si activa esta casilla, se asigna una dirección IPv6 a la interfaz de red, además de la dirección IPv4 ya asignada. Consulte la sección IPv6 de este artículo para obtener información importante sobre el uso de IPv6 con interfaces de red. No se puede seleccionar un método de asignación a la dirección IPv6. Si desea asignar una dirección IPv6, hay que asignarla con el método dinámico.
    |Nombre de IPv6 (solo aparece cuando la casilla **Dirección IP privada (IPv6)** está activada) |Sí, si la casilla **Dirección IP privada (IPv6)** está activada.| Este nombre se asigna a una configuración IP secundaria de la interfaz de red. Para más información sobre las configuraciones de IP, consulte [Visualización de la configuración de la interfaz de red](#view-network-interface-settings).|
    |Resource group|Sí|Seleccione un [grupo de recursos](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group) existente o cree uno. La interfaz de red puede existir en el mismo grupo de recursos que la máquina virtual a la que está asociada o conectada, o en otro.|
    |Location|Sí|La máquina virtual asociada a una interfaz de red y la red virtual a la que está conectada deben existir en la misma [ubicación](https://azure.microsoft.com/regions), que también se conoce como región.|

El portal no proporciona la opción de asignar una dirección IP pública a la interfaz de red al crearla, aunque el portal crea una dirección IP pública y la asigna a una interfaz de red cuando se crea una máquina virtual mediante el portal. Para aprender a agregar una dirección IP pública a la interfaz de red después de crearla, consulte [Administración de direcciones IP](virtual-network-network-interface-addresses.md). Si desea crear una interfaz de red con una dirección IP pública, debe utilizar la CLI o PowerShell para crearla.

El portal no proporciona la opción de asignar una interfaz de red a los grupos de seguridad de aplicaciones al crear una interfaz de red, pero la CLI de Azure y PowerShell sí. Sin embargo, puede asignar una interfaz de red existente a un grupo de seguridad de aplicaciones mediante el portal, siempre que la interfaz de red esté asociada a una máquina virtual. Para aprender a asignar una interfaz de red a un grupo de seguridad de aplicaciones, consulte [Adición o eliminación de grupos de seguridad de aplicaciones](#add-to-or-remove-from-application-security-groups).

>[!Note]
> Azure asigna una dirección MAC a la interfaz de red solo después de asociar la interfaz de red a una máquina virtual y que esta se inicie la primera vez. No se puede especificar la dirección MAC que Azure asigna a la interfaz de red. La dirección MAC permanece asignada a la interfaz de red hasta que esta se elimina o se cambia la dirección IP privada asignada a la configuración de IP principal de la interfaz de red principal. Para más información sobre las direcciones IP y las configuraciones IP, consulte [Administración de direcciones IP](virtual-network-network-interface-addresses.md).

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

**Comandos**

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic create](/cli/azure/network/nic)|
|PowerShell|[New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface)|

## <a name="view-network-interface-settings"></a>Visualización de la configuración de la interfaz de red

Puede ver y cambiar la mayoría de las opciones de una interfaz de red después de crearla. El portal no muestra el sufijo DNS ni la pertenencia a grupos de seguridad de aplicaciones para la interfaz de red. Puede usar los [comandos](#view-settings-commands) de PowerShell o de la CLI de Azure para ver el sufijo DNS y la pertenencia a grupos de seguridad de aplicaciones.

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *interfaces de red*. Cuando aparezcan las **interfaces de red** en los resultados de búsqueda, selecciónelas.
2. En la lista, seleccione la interfaz de red para la que quiere ver o cambiar la configuración.
3. Se muestran los elementos siguientes para la interfaz de red seleccionada:
   - **Información general:** proporciona información acerca de la interfaz de red, como las direcciones IP asignadas a ella, la red o subred virtual a la que está asignada la interfaz de red y la máquina virtual a la que está asociada la interfaz de red (si está asociada a una). La siguiente imagen muestra la configuración de información general de una interfaz de red llamada **mywebserver256**: ![Información general acerca de interfaz de red](./media/virtual-network-network-interface/nic-overview.png)

     Una interfaz de red se puede mover a un grupo de recursos o una suscripción diferentes; para ello, es preciso seleccionar (**cambiar**) junto a **Grupo de recursos** o **Nombre de la suscripción**. Si traslada la interfaz de red a una nueva suscripción, debe trasladar con ella todos sus recursos relacionados. Por ejemplo, si la interfaz de red está asociada a una máquina virtual, también debe mover la máquina virtual y otros recursos relacionados con ella. Para trasladar una interfaz de red, consulte [cómo mover recursos a un nuevo grupo de recursos o suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=%2fazure%2fvirtual-network%2ftoc.json#use-the-portal). En el artículo se enumeran los requisitos previos y se indica cómo trasladar recursos mediante Azure Portal, PowerShell y la CLI de Azure.
   - **Configuraciones IP:** las direcciones IPv4 e IPv6 públicas y privadas asignadas a las configuraciones IP se enumeran aquí. Si se asigna una dirección IPv6 a una configuración de IP, la dirección no se muestra. Para más información sobre las configuraciones de IP y cómo agregar y quitar direcciones IP, consulte [cómo configurar direcciones IP para una interfaz de red de Azure](virtual-network-network-interface-addresses.md). El reenvío de IP y la asignación de subred también se configuran en esta sección. Para más información sobre estos valores de configuración, consulte [Habilitación o deshabilitación del reenvío IP](#enable-or-disable-ip-forwarding) y [Cambio de la asignación de subred](#change-subnet-assignment).
   - **Servidores DNS:** puede especificar a qué servidor DNS asignarán una interfaz de red los servidores DHCP de Azure. La configuración de la red virtual puede heredar la configuración de la red virtual a la que está asignada la interfaz de red, o tener una configuración personalizada que reemplaza la configuración de la red virtual a la que está asignada. Para modificar lo que se muestra, consulte [Cambio de los servidores DNS](#change-dns-servers).
   - **Grupo de seguridad de red (NSG):** muestra el grupo de seguridad de red asociado a la interfaz de red (en caso de haberlos). Un grupo de seguridad de red contiene reglas de entrada y salida para filtrar el tráfico de red de la interfaz de red. Si hay un grupo de seguridad de red asociado a la interfaz de red, se muestra el nombre del grupo de seguridad de red asociado. Para modificar lo que se muestra, consulte [Associate or dissociate a network security group](#associate-or-dissociate-a-network-security-group) (Asociar o desasociar un grupo de seguridad de red).
   - **Propiedades:** muestra las opciones de configuración clave de la interfaz de red, incluida la dirección MAC (en blanco si la interfaz de red no está asociada a una máquina virtual) y la suscripción donde se encuentra.
   - **Reglas de seguridad vigentes:**  las reglas de seguridad se muestran si la interfaz de red está asociada a una máquina virtual en ejecución y hay un grupo de seguridad de red asociado a la interfaz de red, a la subred a la que está asignada o ambas. Para obtener más información sobre lo que se muestra, consulte [Visualización de las reglas de seguridad vigentes](#view-effective-security-rules). Para más información sobre los grupos de seguridad de red, consulte [Grupos de seguridad de red](./network-security-groups-overview.md).
   - **Rutas vigentes:** las rutas se muestran si la interfaz de red está asociada a una máquina virtual en ejecución. Las rutas son una combinación de las rutas predeterminadas de Azure, las rutas definidas por el usuario y las rutas BGP que pueda haber en la subred a la cual está asignada la interfaz de red. Para obtener más información sobre lo que se muestra, consulte [View effective routes](#view-effective-routes) (Visualización de rutas vigentes). Para obtener más información sobre las rutas predeterminadas de Azure y las rutas definidas por el usuario, consulte [Routing overview](virtual-networks-udr-overview.md) (Información general sobre enrutamiento).
Opciones comunes de Azure Resource Manager: para más información acerca de las opciones de configuración comunes de Azure Resource Manager, consulte [Registro de actividad](../azure-monitor/essentials/platform-logs-overview.md), [Control de acceso (IAM)](../role-based-access-control/overview.md), [Etiquetas](../azure-resource-manager/management/tag-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [Bloqueos](../azure-resource-manager/management/lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Script de Automation](../azure-resource-manager/templates/export-template-portal.md).

<a name="view-settings-commands"></a>**Comandos**

Si se asigna una dirección IPv6 a una interfaz de red, la salida de PowerShell devuelve el hecho de que la dirección está asignada, pero no devuelve la dirección asignada. De forma similar, la CLI devuelve el hecho de que la dirección está asignada, pero devuelve *null* en su salida para la dirección.

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic list](/cli/azure/network/nic) para ver las interfaces de red en la suscripción; [az network nic show](/cli/azure/network/nic) para ver la configuración de una interfaz de red|
|PowerShell|[Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) para ver las interfaces de red de la suscripción o ver la configuración de una interfaz de red|

## <a name="change-dns-servers"></a>Cambio de los servidores DNS

El servidor DHCP de Azure asigna el servidor DNS a la interfaz de red en el sistema operativo de la máquina virtual. El servidor DNS asignado tiene la configuración de servidor DNS de la interfaz de red. Para más información sobre las opciones de resolución de nombres de una interfaz de red, consulte el artículo [Resolución de nombres de máquinas virtuales](virtual-networks-name-resolution-for-vms-and-role-instances.md). La interfaz de red puede heredar la configuración de la red virtual o utilizar sus propias opciones que reemplacen las de la red virtual.

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *interfaces de red*. Cuando aparezcan las **interfaces de red** en los resultados de búsqueda, selecciónelas.
2. En la lista, seleccione la interfaz de red cuyo servidor DNS quiere cambiar.
3. Seleccione **Servidores DNS**, en **CONFIGURACIÓN**.
4. Seleccione:
   - **Heredar de la red virtual**: elija esta opción para heredar la configuración del servidor DNS definida para la red virtual a la que está asignada la interfaz de red. En el nivel de red virtual, se define un servidor DNS personalizado o el servidor DNS proporcionado por Azure. El servidor DNS proporcionado por Azure puede resolver los nombres de host de los recursos asignados a la misma red virtual. Para resolver los recursos asignados a otras redes virtuales, debe utilizarse FQDN.
   - **Personalizado**: puede configurar su propio servidor DNS para resolver nombres en diferentes redes virtuales. Escriba la dirección IP del servidor que desea usar como servidor DNS. La dirección del servidor DNS especificado se asigna solo a esta interfaz de red y reemplaza cualquier configuración DNS de la red virtual a la que está asignada la interfaz de red.
     >[!Note]
     >Si la máquina virtual usa una tarjeta de interfaz de red que forma parte de un conjunto de disponibilidad, se heredarán todos los servidores DNS que se especifican para cada una de las máquinas virtuales de todas las tarjetas de interfaz de red que forman parte del conjunto de disponibilidad.
5. Seleccione **Guardar**.

**Comandos**

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic update](/cli/azure/network/nic)|
|PowerShell|[Set-AzNetworkInterface](/powershell/module/az.network/set-aznetworkinterface)|

## <a name="enable-or-disable-ip-forwarding"></a>Habilitación o deshabilitación del reenvío IP

El reenvío IP permite a la máquina virtual a la que está asociada la interfaz de red:
- Recibir tráfico de red no destinado a una de las direcciones IP asignadas a cualquiera de las configuraciones de IP asignadas a la interfaz de red.
- Enviar tráfico de red con una dirección IP de origen diferente de la asignada a una de las configuraciones de IP de la interfaz de red.

La configuración debe estar habilitada para cada interfaz de red que esté asociada a la máquina virtual que recibe el tráfico y que esta debe reenviar. Una máquina virtual puede reenviar el tráfico si tiene varias interfaces de red o una sola interfaz de red asociada a él. Aunque el reenvío IP es una configuración de Azure, la máquina virtual también debe ejecutar una aplicación que pueda reenviar el tráfico, como aplicaciones de firewall, de optimización de WAN o de equilibrio de carga. Cuando una máquina virtual está ejecutando aplicaciones de red, la máquina virtual suele denominarse aplicación virtual de red. Puede ver una lista de las aplicaciones virtuales de red listas para su implementación en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances). El reenvío IP normalmente se usa con rutas definidas por el usuario. Para más información sobre las rutas definidas por el usuario, consulte [Rutas definidas por el usuario](virtual-networks-udr-overview.md).

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *interfaces de red*. Cuando aparezcan las **interfaces de red** en los resultados de búsqueda, selecciónelas.
2. Seleccione la interfaz de red para la que quiere habilitar o deshabilitar el reenvío de IP.
3. Seleccione **Configuraciones IP** en la sección **CONFIGURACIÓN**.
4. Seleccione **Habilitado** o **Deshabilitado** (valor predeterminado) para cambiar la configuración.
5. Seleccione **Guardar**.

**Comandos**

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic update](/cli/azure/network/nic)|
|PowerShell|[Set-AzNetworkInterface](/powershell/module/az.network/set-aznetworkinterface)|

## <a name="change-subnet-assignment"></a>Cambio de la asignación de subred

Puede cambiar la subred, pero no la red virtual, a la que está asignada una interfaz de red.

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *interfaces de red*. Cuando aparezcan las **interfaces de red** en los resultados de búsqueda, selecciónelas.
2. Seleccione la interfaz de red para la que quiere cambiar la asignación de subred.
3. Seleccione **Configuraciones IP** en **CONFIGURACIÓN**. Si junto a las direcciones IP privadas de las configuraciones de IP que se enumeran aquí aparece **(Estático)** , debe seguir los pasos a continuación y así cambiar al método dinámico de asignación de direcciones IP. Para cambiar la asignación de subred para la interfaz de red de todas las direcciones IP privadas, es obligatorio el método dinámico de asignación. Si las direcciones se han asignado con el método dinámico, vaya al paso cinco. Si se asignó alguna dirección IPv4 con el método estático, complete los pasos siguientes para cambiar al método de asignación dinámico:
   - En la lista de configuraciones de IP, seleccione la configuración de IP cuyo método de asignación de direcciones IPv4 desea cambiar.
   - Seleccione **Dinámico** como el método de **asignación** de direcciones IP privadas. No se puede asignar una dirección IPv6 con el método de asignación estático.
   - Seleccione **Guardar**.
4. En la lista desplegable **Subred**, seleccione la subred a la que quiere mover la interfaz de red.
5. Seleccione **Guardar**. Las nuevas direcciones dinámicas se asignan a partir del intervalo de direcciones de la subred nueva. Una vez asignada la interfaz de red a una nueva subred, si lo prefiere, podrá asignar direcciones IPv4 estáticas del nuevo intervalo de direcciones de subred. Para más información sobre cómo agregar, cambiar y quitar direcciones IP en una interfaz de red, consulte [Administración de direcciones IP](virtual-network-network-interface-addresses.md).

**Comandos**

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic ip-config update](/cli/azure/network/nic/ip-config)|
|PowerShell|[Set-AzNetworkInterfaceIpConfig](/powershell/module/az.network/set-aznetworkinterfaceipconfig)|

## <a name="add-to-or-remove-from-application-security-groups"></a>Adición o eliminación de grupos de seguridad de aplicaciones

Solo puede agregar o quitar una interfaz de red de un grupo de seguridad de aplicaciones mediante el portal si la interfaz de red está asociada a una máquina virtual. Puede usar PowerShell o la CLI de Azure para agregar o quitar una interfaz de red de un grupo de seguridad de aplicaciones si la interfaz de red está asociada o no a una máquina virtual. Para más información sobre los [grupos de seguridad de aplicaciones](./network-security-groups-overview.md#application-security-groups), consulte el artículo sobre cómo [crear un grupo de seguridad de aplicaciones](manage-network-security-group.md).

1. En el cuadro *Buscar recursos, servicios y documentos* en la parte superior del portal, comience a escribir el nombre de una máquina virtual que tenga una interfaz de red que desee agregar o quitar de un grupo de seguridad de aplicaciones. Cuando el nombre de la máquina virtual aparezca en los resultados de búsqueda, selecciónelo.
2. En **CONFIGURACIÓN**, seleccione **Redes**.  Seleccione **Grupos de seguridad de aplicación**, después **Configurar grupos de seguridad de aplicación**, seleccione los grupos de seguridad de la aplicación a los que desee agregar la interfaz de red o anule la selección de los grupos de seguridad de la aplicación de los que desee quitar la interfaz de red y, a continuación, seleccione **Guardar**. Solo las interfaces de red que existen en la misma red virtual se pueden agregar al mismo grupo de seguridad de aplicaciones. El grupo de seguridad de aplicaciones debe existir en la misma ubicación que la interfaz de red.

**Comandos**

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic update](/cli/azure/network/nic)|
|PowerShell|[Set-AzNetworkInterface](/powershell/module/az.network/set-aznetworkinterface)|

## <a name="associate-or-dissociate-a-network-security-group"></a>Asociación o desasociación de un grupo de seguridad de red

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *network interfaces*. Cuando aparezcan las **interfaces de red** en los resultados de búsqueda, selecciónelas.
2. Seleccione la interfaz de red de la lista a la que quiera asociar o de la que quiera desasociar un grupo de seguridad.
3. Seleccione **Grupo de seguridad de red** en **CONFIGURACIÓN**.
4. Seleccione **Editar**.
5. Seleccione **Grupo de seguridad de red** y, a continuación, seleccione el grupo de seguridad de red que quiere asociar a la interfaz de red, o bien seleccione **Ninguno** para desasociar un grupo de seguridad de red.
6. Seleccione **Guardar**.

**Comandos**

- CLI de Azure: [az network nic update](/cli/azure/network/nic#az_network_nic_update)
- PowerShell: [Set-AzNetworkInterface](/powershell/module/az.network/set-aznetworkinterface)

## <a name="delete-a-network-interface"></a>Eliminar una interfaz de red

Puede eliminar una interfaz de red siempre y cuando no esté asociada a una máquina virtual. Si una interfaz de red está asociada a una máquina virtual, primero debe cambiar el estado de la máquina virtual a detenido (desasignado) y, después, desasociar la interfaz de red de la máquina virtual. Para desasociar una interfaz de red de una máquina virtual, complete los pasos que se indican en [Desasociar una interfaz de red de una máquina virtual](virtual-network-network-interface-vm.md#remove-a-network-interface-from-a-vm). Sin embargo, no se puede desasociar una interfaz de red de una máquina virtual si es la única interfaz de red asociada a la máquina virtual. Una máquina virtual debe tener siempre una interfaz de red asociada como mínimo. Al eliminar una máquina virtual, se desasocian todas las interfaces de red asociadas a ella, pero no se eliminan las interfaces de red.

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *interfaces de red*. Cuando aparezcan las **interfaces de red** en los resultados de búsqueda, selecciónelas.
2. Seleccione la interfaz de red en la lista que quiere eliminar.
3. En **Información general**, seleccione **Eliminar**.
4. Seleccione **Sí** para confirmar la eliminación de la interfaz de red.

Cuando se elimina una interfaz de red, se liberan las direcciones MAC o IP asignadas a ella.

**Comandos**

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic delete](/cli/azure/network/nic)|
|PowerShell|[Remove-AzNetworkInterface](/powershell/module/az.network/remove-aznetworkinterface)|

## <a name="resolve-connectivity-issues"></a>Resolución de problemas de conectividad

Si no puede comunicarse con o desde una máquina virtual, es posible que las reglas de seguridad del grupo de seguridad de red o las rutas vigentes de una interfaz de red provoquen el problema. Las siguientes opciones pueden ayudarle a resolver el problema:

### <a name="view-effective-security-rules"></a>Visualización de las reglas de seguridad vigentes

Las reglas de seguridad vigentes de cada interfaz de red asociada a una máquina virtual son una combinación de las reglas que ha creado en un grupo de seguridad de red y las [reglas de seguridad predeterminadas](./network-security-groups-overview.md#default-security-rules). Comprender las reglas de seguridad vigentes de una interfaz de red puede ayudarle a determinar por qué no se puede comunicar con o desde una máquina virtual. Puede ver las reglas vigentes de cualquier interfaz de red asociada a una máquina virtual en ejecución.

1. En el cuadro de búsqueda que se encuentra en la parte superior del portal, escriba el nombre de una máquina virtual cuyas reglas de seguridad vigentes quiera ver. Si no conoce el nombre de una máquina virtual, escriba *máquinas virtuales* en el cuadro de búsqueda. Cuando la opción **Máquinas virtuales** aparezca en los resultados de la búsqueda, selecciónela y, después, seleccione una máquina virtual en la lista.
2. Seleccione **Redes** en **CONFIGURACIÓN**.
3. Seleccione el nombre de una interfaz de red.
4. Seleccione **Reglas de seguridad vigentes** en **SOPORTE TÉCNICO Y SOLUCIÓN DE PROBLEMAS**.
5. Revise la lista de reglas de seguridad vigentes para determinar si existen las reglas correctas para la comunicación necesaria entrante y saliente. Obtenga más información sobre lo que se ve en la lista en [Información general sobre el grupo de seguridad de red](./network-security-groups-overview.md).

La característica de comprobación del flujo de IP de Azure Network Watcher también puede ayudarle a determinar si las reglas de seguridad impiden la comunicación entre una máquina virtual y un punto de conexión. Para obtener más información, consulte [IP flow verify](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Comprobación del flujo de IP).

**Comandos**

- CLI de Azure: [az network nic list-effective-nsg](/cli/azure/network/nic#az_network_nic_list_effective_nsg)
- PowerShell: [Get-AzEffectiveNetworkSecurityGroup](/powershell/module/az.network/get-azeffectivenetworksecuritygroup)

### <a name="view-effective-routes"></a>Vista de las rutas eficaces

Las rutas vigentes de las interfaces de red asociadas a una máquina virtual son una combinación de las rutas predeterminadas, las rutas que creó y cualquier ruta propagada desde las redes locales vía BGP a través de una puerta de enlace de red virtual de Azure. Comprender las rutas vigentes de una interfaz de red puede ayudarle a determinar por qué no se puede comunicar con o desde una máquina virtual. Puede ver las rutas eficaces de cualquier interfaz de red conectada a una máquina virtual en ejecución.

1. En el cuadro de búsqueda que se encuentra en la parte superior del portal, escriba el nombre de una máquina virtual cuyas reglas de seguridad vigentes quiera ver. Si no conoce el nombre de una máquina virtual, escriba *máquinas virtuales* en el cuadro de búsqueda. Cuando la opción **Máquinas virtuales** aparezca en los resultados de la búsqueda, selecciónela y, después, seleccione una máquina virtual en la lista.
2. Seleccione **Redes** en **CONFIGURACIÓN**.
3. Seleccione el nombre de una interfaz de red.
4. Seleccione **Rutas eficaces** en **SOPORTE Y SOLUCIÓN DE PROBLEMAS**.
5. Revise la lista de rutas vigentes para determinar si existen las rutas correctas para la comunicación necesaria entrante y saliente. Obtenga más información sobre lo que ve en la lista en [Routing overview](virtual-networks-udr-overview.md) (Información general sobre enrutamiento).

La característica de próximo salto de Azure Network Watcher también puede ayudarle a determinar si las rutas impiden la comunicación entre una máquina virtual y un punto de conexión. Para obtener más información, consulte [Próximo salto](../network-watcher/diagnose-vm-network-routing-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

**Comandos**

- CLI de Azure: [az network nic show-effective-route-table](/cli/azure/network/nic#az_network_nic_show_effective_route_table)
- PowerShell: [Get-AzEffectiveRouteTable](/powershell/module/az.network/get-azeffectiveroutetable)

## <a name="permissions"></a>Permisos

Para realizar tareas en interfaces virtuales, su cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un rol [personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignados los permisos adecuados que se muestran en la tabla siguiente:

| Acción                                                                     | Nombre                                                      |
| ---------                                                                  | -------------                                             |
| Microsoft.Network/networkInterfaces/read                                   | Obtener interfaz de red                                     |
| Microsoft.Network/networkInterfaces/write                                  | Crear o actualizar una interfaz de red                        |
| Microsoft.Network/networkInterfaces/join/action                            | Adjuntar una interfaz de red a una máquina virtual           |
| Microsoft.Network/networkInterfaces/delete                                 | Eliminar la interfaz de red                                  |
| Microsoft.Network/networkInterfaces/joinViaPrivateIp/action                | Unir un recurso a una interfaz de red a través de una asociación de servicio     |
| Microsoft.Network/networkInterfaces/effectiveRouteTable/action             | Obtener una tabla de rutas eficaces de interfaz de red               |
| Microsoft.Network/networkInterfaces/effectiveNetworkSecurityGroups/action  | Obtener grupos de seguridad eficaces de interfaz de red           |
| Microsoft.Network/networkInterfaces/loadBalancers/read                     | Obtener equilibradores de carga de interfaz de red                      |
| Microsoft.Network/networkInterfaces/serviceAssociations/read               | Obtener una asociación de servicio                                   |
| Microsoft.Network/networkInterfaces/serviceAssociations/write              | Crear o actualizar una asociación de servicio                    |
| Microsoft.Network/networkInterfaces/serviceAssociations/delete             | Eliminar una asociación de servicio                                |
| Microsoft.Network/networkInterfaces/serviceAssociations/validate/action    | Validar una asociación de servicio                              |
| Microsoft.Network/networkInterfaces/ipconfigurations/read                  | Obtener una configuración de dirección IP de interfaz de red                    |

## <a name="next-steps"></a>Pasos siguientes

- Crear una máquina virtual con varias NIC mediante la [CLI de Azure](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [PowerShell](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- Crear una sola máquina virtual NIC con varias direcciones IPv4 mediante la [CLI de Azure](virtual-network-multiple-ip-addresses-cli.md) o [PowerShell](virtual-network-multiple-ip-addresses-powershell.md)
- Crear una sola máquina virtual NIC con una dirección IPv6 privada (detrás de una instancia de Azure Load Balancer) mediante la [CLI de Azure](../load-balancer/load-balancer-ipv6-internet-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [PowerShell](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o una [plantilla de Azure Resource Manager](../load-balancer/load-balancer-ipv6-internet-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- Crear una interfaz de red con scripts de ejemplo de [PowerShell](powershell-samples.md) o de la [CLI de Azure](cli-samples.md) o con una [plantilla de Azure Resource Manager](template-samples.md)
- Crear y asignar [definiciones de Azure Policy](./policy-reference.md) para redes virtuales