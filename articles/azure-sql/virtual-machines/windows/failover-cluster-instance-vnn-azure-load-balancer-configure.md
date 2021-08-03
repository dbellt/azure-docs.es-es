---
title: Configuración de Azure Load Balancer para un VNN de instancia de clúster de conmutación por error
description: Aprenda a configurar una instancia de Azure Load Balancer para enrutar el tráfico al nombre de red virtual (VNN) para la instancia de clúster de conmutación por error (FCI) con SQL Server en máquinas virtuales de Azure para lograr una alta disponibilidad y recuperación ante desastres (HADR).
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: jroth
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 5cecf43f3795e38daa75f9463cadec94114046de
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111568907"
---
# <a name="configure-azure-load-balancer-for-an-fci-vnn"></a>Configuración de Azure Load Balancer para un VNN de instancia de clúster de conmutación por error
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En máquinas virtuales de Azure, los clústeres usan un equilibrador de carga para mantener una dirección IP que es preciso que esté en un único nodo de clúster a la vez. En esta solución, el equilibrador de carga contiene la dirección IP del nombre de red virtual (VNN) que utiliza el recurso en clúster en Azure. 

En este artículo aprende a configurar un equilibrador de carga mediante el servicio Azure Load Balancer. El equilibrador de carga enrutará el tráfico a la [instancia de clúster de conmutación por error (FCI)](failover-cluster-instance-overview.md) con SQL Server en máquinas virtuales de Azure para lograr una alta disponibilidad y recuperación ante desastres (HADR). 

Para obtener una opción de conectividad alternativa para SQL Server 2019 CU2 y versiones posteriores, considere la posibilidad de usar un [nombre de red distribuida](failover-cluster-instance-distributed-network-name-dnn-configure.md) en su lugar para lograr una configuración simplificada y una mejor conmutación por error. 


## <a name="prerequisites"></a>Requisitos previos

Antes de completar los pasos de este artículo, ya debe tener:

- Decidido que Azure Load Balancer es la [opción de conectividad adecuada para su instancia de clúster de conmutación por error](hadr-windows-server-failover-cluster-overview.md#virtual-network-name-vnn).
- Configuró las [instancias de clúster de conmutación por error](failover-cluster-instance-overview.md). 
- Instaló la versión más reciente de [Azure PowerShell](/powershell/scripting/install/installing-powershell-core-on-windows). 

## <a name="create-load-balancer"></a>Creación de un equilibrador de carga

Utilice [Azure Portal](https://portal.azure.com) para crear del equilibrador de carga:

1. En Azure Portal, vaya al grupo de recursos que contiene las máquinas virtuales.

1. Seleccione **Agregar**. Busque **Equilibrador de carga** en Azure Marketplace. Seleccione **Equilibrador de carga**.

1. Seleccione **Crear**.

1. Configure el equilibrador de carga con los siguientes valores:

   - **Suscripción**: Su suscripción de Azure.
   - **Grupo de recursos**: El grupo de recursos que contiene las máquinas virtuales.
   - **Name**: un nombre que identifica el equilibrador de carga.
   - **Región**: La ubicación de Azure que contiene las máquinas virtuales.
   - **Tipo**: Pública o privada. A los equilibradores de carga privados se puede acceder desde la red virtual. La mayoría de las aplicaciones de Azure pueden usar un equilibrador de carga privado. Si la aplicación necesita acceder a SQL Server directamente a través de Internet, utilice un equilibrador de carga público.
   - **SKU**: Estándar.
   - **Red virtual**: la misma red que la de las máquinas virtuales.
   - **Asignación de dirección IP**: Estática. 
   - **Dirección IP privada**: la dirección IP que asignó al recurso de red en clúster.

   En la imagen siguiente se muestra la interfaz de usuario **Crear un equilibrador de carga**:

   ![Configuración del equilibrador de carga](./media/failover-cluster-instance-premium-file-share-manually-configure/30-load-balancer-create.png)
   

## <a name="configure-backend-pool"></a>Configuración del grupo back-end

1. Vuelva al grupo de recursos de Azure que contiene las máquinas virtuales y busque el equilibrador de carga nuevo. Es posible que tenga que actualizar la vista en el grupo de recursos. Seleccione el equilibrador de carga.

1. Seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.

1. Asocie el grupo de back-end con el conjunto de disponibilidad que contiene las máquinas virtuales.

1. En **Configuraciones IP de red de destino**, active **MÁQUINA VIRTUAL** y seleccione las máquinas virtuales que participarán como nodos de clúster. No olvide incluir todas las máquinas virtuales que hospedarán el FCI.

1. Seleccione **Aceptar** para crear el grupo de back-end.

## <a name="configure-health-probe"></a>Configuración de sondeo de estado

1. En el panel del equilibrador de carga, seleccione **Sondeos de estado**.

1. Seleccione **Agregar**.

1. En el panel **Agregar sondeo de estado**, <span id="probe"></span>establezca los parámetros del sondeo de estado siguientes:

   - **Name**: nombre del sondeo de estado.
   - **Protocolo**: TCP.
   - **Puerto**: el puerto que creó en el firewall para el sondeo de estado al [preparar la máquina virtual](failover-cluster-instance-prepare-vm.md#uninstall-sql-server-1). En este artículo, el ejemplo usa el puerto TCP `59999`.
   - **Intervalo**: 5 segundos.
   - **Umbral incorrecto**: 2 errores consecutivos.

1. Seleccione **Aceptar**.

## <a name="set-load-balancing-rules"></a>Establecimiento de reglas de equilibrio de carga

1. En el panel del equilibrador de carga, seleccione **Reglas de equilibrio de carga**.

1. Seleccione **Agregar**.

1. Establezca los parámetros de la regla de equilibrio de carga:

   - **Name**: nombre de las reglas de equilibrio de carga.
   - **Dirección IP de front-end**: La dirección IP del recurso de red en clúster del cliente de escucha del grupo de disponibilidad o de la FCI de SQL Server.
   - **Puerto**: el puerto TCP de SQL Server. El puerto de la instancia predeterminado es 1433.
   - **Puerto back-end**: el mismo puerto que el valor **Puerto** cuando se habilita **IP flotante (Direct Server Return)** .
   - **Grupo de back-end**: el nombre del grupo de back-end que configuró anteriormente.
   - **Sondeo de mantenimiento**: el sondeo de estado que configuró anteriormente.
   - **Persistencia de la sesión**: Ninguno.
   - **Tiempo de espera de inactividad (minutos)** : 4.
   - **IP flotante (Direct Server Return)** : Habilitado.

1. Seleccione **Aceptar**.

## <a name="configure-cluster-probe"></a>Configuración del clúster de sondeo

Establezca el parámetro del puerto de sondeo de clúster en PowerShell.

Para establecer el parámetro del puerto de sondeo de clúster, actualice las variables del siguiente script con los valores del entorno. Quite los corchetes angulares (`<` y `>`) del script.

```powershell
$ClusterNetworkName = "<Cluster Network Name>"
$IPResourceName = "<SQL Server FCI IP Address Resource Name>" 
$ILBIP = "<n.n.n.n>" 
[int]$ProbePort = <nnnnn>

Import-Module FailoverClusters

Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
```

En la tabla siguiente se describen los valores que debe actualizar:


|**Valor**|**Descripción**|
|---------|---------|
|`Cluster Network Name`| el nombre del clúster de conmutación por error de Windows Server para la red. En **Administrador de clústeres de conmutación por error** > **Redes**, haga clic con el botón derecho en la red y después seleccione **Propiedades**. El valor correcto está debajo del campo **Nombre** en la pestaña **General**.|
|`SQL Server FCI/AG listener IP Address Resource Name`|El nombre del recurso para la dirección IP del cliente de escucha del grupo de disponibilidad o de la FCI de SQL Server. En **Administrador de clústeres de conmutación por error** > **Roles**, en el rol FCI de SQL Server, en **Nombre del servidor**, haga clic con el botón derecho en el recurso de la dirección IP y después seleccione **Propiedades**. El valor correcto está debajo del campo **Nombre** en la pestaña **General**.|
|`ILBIP`|La dirección IP del equilibrador de carga interno (ILB). Esta dirección se configura en Azure Portal como la dirección front-end de ILB. También es la dirección IP de FCI de SQL Server. Puede encontrarla en **Administrador de clústeres de conmutación por error** en la misma página de propiedades donde encuentra `<SQL Server FCI/AG listener IP Address Resource Name>`.|
|`nnnnn`|El puerto de sondeo configurado en el sondeo de estado del equilibrador de carga. Cualquier puerto TCP no utilizado es válido.|
|"SubnetMask"| Máscara de subred para el parámetro de clúster. Debe ser la dirección de difusión TCP IP: `255.255.255.255`.| 


Después de establecer el sondeo de clúster puede ver todos los parámetros del clúster en PowerShell. Ejecute este script:

```powershell
Get-ClusterResource $IPResourceName | Get-ClusterParameter
```

## <a name="modify-connection-string"></a>Modificación de la cadena de conexión 

Para los clientes que lo admitan, agregue `MultiSubnetFailover=True` a la cadena de conexión.  Cuando la opción de conexión MultiSubnetFailover no es necesaria, ofrece la ventaja de una conmutación por error más rápida de la subred. Esto se debe a que el controlador de cliente intentará abrir un socket de TCP para cada dirección IP en paralelo. El controlador cliente esperará que la primera dirección IP responda y, una vez que lo haga, la utilizará para la conexión.

Si el cliente no admite el parámetro MultiSubnetFailover, puede modificar la configuración de RegisterAllProvidersIP y HostRecordTTL para evitar retrasos en la conectividad tras la conmutación por error. 

Use PowerShell para modificar la configuración de RegisterAllProvidersIp y HostRecordTTL: 

```powershell
Get-ClusterResource yourFCIname | Set-ClusterParameter RegisterAllProvidersIP 0  
Get-ClusterResource yourFCIname | Set-ClusterParameter HostRecordTTL 300 
```

Para obtener más información, consulte la documentación sobre el [tiempo de espera de la conexión de escucha](/troubleshoot/sql/availability-groups/listener-connection-times-out) de SQL Server. 

> [!TIP]
> - Establezca el parámetro MultiSubnetFailover en true en la cadena de conexión, incluso para las soluciones HADR que abarquen una única subred, para admitir la expansión futura de las subredes sin necesidad de actualizar las cadenas de conexión.  
> - De forma predeterminada, los clientes almacenan en memoria caché los registros DNS de clúster durante 20 minutos. Al reducir el valor HostRecordTTL, se reduce el período de vida (TTL) del registro almacenado en memoria caché y los clientes heredados pueden volver a conectarse más rápido. Por lo tanto, la reducción del valor de HostRecordTTL también puede producir un aumento de tráfico en los servidores DNS.


## <a name="test-failover"></a>Conmutación por error de prueba


Pruebe la conmutación por error del recurso en clúster para validar la funcionalidad del clúster. 

Siga estos pasos.

1. Conéctese a uno de los nodos de clúster de SQL Server con RDP.
1. Abra el **Administrador de clústeres de conmutación por error**. Seleccione **Roles**. Observe qué nodo posee el rol de FCI de SQL Server.
1. Haga clic con el botón derecho en el rol de FCI de SQL Server. 
1. Seleccione **Mover** y, después, seleccione **Mejor nodo posible**.

El **Administrador de clústeres de conmutación por error** muestra el rol y sus recursos pierden su conexión. Después, los recursos se mueven y vuelven a conectarse en el otro nodo.


## <a name="test-connectivity"></a>Comprobación de la conectividad

Para probar la conectividad, inicie sesión en otra máquina virtual de la misma red virtual. Abra **SQL Server Management Studio** y conéctese al nombre de la FCI de SQL Server. 

>[!NOTE]
>Si es necesario, puede [descargar SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).





## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte:

- [Clúster de conmutación por error de Windows Server con SQL Server en máquinas virtuales de Azure](hadr-windows-server-failover-cluster-overview.md)
- [Instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines](failover-cluster-instance-overview.md)
- [Información general de las instancias de clúster de conmutación por error](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
- [Configuración de alta disponibilidad y recuperación ante desastres para SQL Server en máquinas virtuales de Azure](hadr-cluster-best-practices.md)




