---
title: Escalabilidad horizontal de SAP HANA con HSR y Pacemaker en SLES | Microsoft Docs
description: Escalabilidad horizontal de SAP HANA con HSR y Pacemaker en SLES.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/12/2021
ms.author: radeltch
ms.openlocfilehash: 637616c3698cc9ec0cd13a4584bad24b6ed02c34
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107315266"
---
# <a name="high-availability-for-sap-hana-scale-out-system-with-hsr-on-suse-linux-enterprise-server"></a>Alta disponibilidad para el sistema de escalabilidad horizontal de SAP HANA con HSR en SUSE Linux Enterprise Server 

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all 
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736
[1900823]:https://launchpad.support.sap.com/#/notes/1900823

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md


En este artículo se describe cómo implementar un sistema SAP HANA de alta disponibilidad en una configuración de escalabilidad horizontal con replicación del sistema HANA (HSR) y Pacemaker en máquinas virtuales (VM) de Azure SUSE Linux Enterprise Server. Los sistemas de archivos compartidos de la arquitectura presentada se proporcionan mediante [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) y se montan en NFS.  

En las configuraciones de ejemplo, como en los comandos de instalación, la instancia de HANA es **03** y el identificador del sistema de HANA es **HN1**. Los ejemplos se basan en HANA 2.0 SP4 y SUSE Linux Enterprise Server 12 SP5. 

Antes de comenzar, consulte las siguientes notas y documentos de SAP:

* [Documentación sobre Azure NetApp Files][anf-azure-doc] 
* Nota de SAP [1928533], que incluye:  
  * Una lista de los tamaños de máquina virtual de Azure que se admiten para la implementación de software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * La versión del kernel de SAP necesaria para Windows y Linux en Microsoft Azure.
* Nota de SAP [2015553]: enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* Nota de SAP [2205917]: configuraciones recomendadas del sistema operativo para SUSE Linux Enterprise Server para SAP Applications.
* Nota de SAP [1944799]: contiene guías de SAP para SUSE Linux Enterprise Server para SAP Applications.
* Nota de SAP [2178632]: contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* Nota de SAP [2191498]: incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* Nota de SAP [2243692]: contiene información sobre las licencias de SAP en Linux en Azure.
* Nota de SAP [1984787]: contiene información general sobre SUSE Linux Enterprise Server 12.
* Nota de SAP [1999351]: contiene información adicional sobre la solución de problemas de la extensión de supervisión mejorada de Azure para SAP.
* Nota de SAP [1900823]: incluye información sobre los requisitos de almacenamiento de SAP HANA.
* [SAP Community Wiki](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) (wiki de la comunidad de SAP): Contiene todas las notas de SAP que se requieren para Linux.
* [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
* [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* [Guías de procedimientos recomendados para la alta disponibilidad de SUSE SAP][suse-ha-guide]: contiene toda la información necesaria para configurar la alta disponibilidad de NetWeaver y la replicación del sistema local de SAP HANA (para usarse como base de referencia; proporcionan información mucho más detallada).
* [Notas de la versión de la Extensión 12 SP5 de alta disponibilidad para SUSE](https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP5/)
* [Control de recursos compartidos de NFS con errores en el clúster de alta disponibilidad de SUSE para la replicación del sistema HANA](https://www.suse.com/support/kb/doc/?id=000019904)
* [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]
* [Volúmenes NFS v4.1 en Azure NetApp Files para SAP HANA](./hana-vm-operations-netapp.md)

## <a name="overview"></a>Información general

Un método para lograr la alta disponibilidad de HANA para las instalaciones de escalabilidad horizontal de HANA es configurar la replicación del sistema HANA y proteger la solución con el clúster de Pacemaker para permitir la conmutación por error automática. Cuando se produce un error en un nodo activo, el clúster conmuta por error los recursos de HANA al otro sitio.  
La configuración presentada muestra tres nodos de HANA en cada sitio, más el nodo para lograr la mayoría que sirve para evitar el escenario de split-brain. Las instrucciones se pueden adaptar para incluir más máquinas virtuales como nodos de base de datos HANA.  

[Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) proporciona el sistema de archivos compartidos de HANA `/hana/shared` en la arquitectura presentada. Se monta mediante NFSv4.1 en cada nodo de HANA, en el mismo sitio de replicación del sistema HANA. Los sistemas de archivos `/hana/data` y `/hana/log` son sistemas de archivos locales y no se comparten entre los nodos de la base de datos HANA. SAP HANA se instalará en modo no compartido. 

> [!TIP]
> Para las configuraciones de almacenamiento de SAP HANA recomendadas, consulte [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md).   

[![Escalabilidad horizontal de SAP HANA con HSR y el clúster de Pacemaker en SLES](./media/sap-hana-high-availability/sap-hana-high-availability-scale-out-hsr-suse.png)](./media/sap-hana-high-availability/sap-hana-high-availability-scale-out-hsr-suse-detail.png#lightbox)

En el diagrama anterior, se representan tres subredes dentro de una red virtual de Azure, siguiendo las recomendaciones de la red de SAP HANA: 
* Para la comunicación del cliente: `client` 10.23.0.0/24  
* Para la comunicación interna entre nodos de HANA: `inter` 10.23.1.128/26  
* Para la replicación del sistema HANA: `hsr` 10.23.1.192/26  

Como `/hana/data` y `/hana/log` se han implementado en discos locales, no es necesario implementar una subred independiente ni tarjetas de red virtual independientes para la comunicación con el almacenamiento.  

Los volúmenes de Azure NetApp están en una subred independiente y [delegada a Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-delegate-subnet): `anf` 10.23.1.0/26.   

> [!IMPORTANT]
> No se admite la replicación del sistema en un tercer sitio. Para obtener más información, consulte la sección "Requisitos previos importantes" del [Escenario optimizado para el rendimiento de escalabilidad horizontal de la replicación del sistema SLES-SAP HANA](https://documentation.suse.com/sbp/all/html/SLES4SAP-hana-scaleOut-PerfOpt-12/index.html#_important_prerequisites).     

## <a name="set-up-the-infrastructure"></a>Configuración de la infraestructura

En las instrucciones siguientes, se supone que ya ha creado el grupo de recursos, la red virtual de Azure con tres subredes de la red de Azure: `client`, `inter` y `hsr`.

### <a name="deploy-linux-virtual-machines-via-the-azure-portal"></a>Implementación de máquinas virtuales Linux en Azure Portal
1. Implemente las máquinas virtuales de Azure.  
Para la configuración presentada en este documento, implemente siete máquinas virtuales: 
   - tres máquinas virtuales que actúan como nodos de la base de datos HANA para la replicación del sitio 1 de HANA: **hana-s1-db1**, **hana-s1-db2** y **hana-s1-db3**  
   - tres máquinas virtuales que actúan como nodos de la base de datos HANA para la replicación del sitio 2 de HANA: **hana-s2-db1**, **hana-s2-db2** y **hana-s2-db3**  
   - una máquina virtual pequeña que sirve para *obtener la mayoría*: **hana-s-mm**

   Las máquinas virtuales, implementadas como nodos de base de datos de SAP HANA, deben tener la certificación de SAP para HANA, tal como se publica en el [directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Al implementar los nodos de base de datos HANA, asegúrese de que se ha seleccionado la [red acelerada](../../../virtual-network/create-vm-accelerated-networking-cli.md).  
  
   Para el nodo que sirve para lograr la mayoría, puede implementar una máquina virtual pequeña, ya que esta máquina virtual no ejecuta ninguno de los recursos de SAP HANA. La máquina virtual para lograr la mayoría se usa en la configuración del clúster para obtener un número impar de nodos de clúster en un escenario de split-brain. La máquina virtual para lograr la mayoría solo necesita una interfaz de red virtual en la subred `client` en este ejemplo.        

   Implemente discos administrados locales para `/hana/data` y `/hana/log`. La configuración de almacenamiento mínima recomendada para `/hana/data` y `/hana/log` se describe en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md).

   Implemente la interfaz de red principal para cada máquina virtual en la subred de la red virtual `client`.  
   Cuando la máquina virtual se implementa mediante Azure Portal, el nombre de la interfaz de red se genera automáticamente. Para simplificar, en estas instrucciones haremos referencia a las interfaces de red principales, generadas automáticamente, que están conectadas a la subred de la red virtual de Azure `client` como **hana-s1-db1-client**, **hana-s1-db2-client**, **hana-s1-db3-client**, etcétera.  


   > [!IMPORTANT]
   > Asegúrese de que el sistema operativo seleccionado esté certificado por SAP para SAP HANA en los tipos específicos de VM que use. Para obtener una lista de los tipos de VM certificados para SAP HANA y las versiones de sistema operativo correspondientes, consulte el sitio [Plataformas de IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Haga clic en los detalles del tipo de VM de la lista para obtener la lista completa de las versiones de sistema operativo compatibles con SAP HANA para ese tipo.  
  

2. Cree seis interfaces de red, una para cada máquina virtual de base de datos HANA, en la subred de la red virtual `inter` (en este ejemplo, **hana-s1-db1-inter**, **hana-s1-db2-inter**, **hana-s1-db3-inter**, **hana-s2-db1-inter**, **hana-s2-db2-inter** y **hana-s2-db3-inter**).  

3. Cree seis interfaces de red, una para cada máquina virtual de base de datos HANA, en la subred de la red virtual `hsr` (en este ejemplo, **hana-s1-db1-hsr**, **hana-s1-db2-hsr**, **hana-s1-db3-hsr**, **hana-s2-db1-hsr**, **hana-s2-db2-hsr** y **hana-s2-db3-hsr**).  

4. Conexión de las interfaces de red virtual recién creadas a las máquinas virtuales correspondientes:  

    a. Vaya a la máquina virtual en [Azure Portal](https://portal.azure.com/#home).  

    b. En el menú de la izquierda, seleccione **Máquinas virtuales**. Filtre por el nombre de máquina virtual (por ejemplo, **hana-s1-db1**) y luego seleccione la máquina virtual.  

    c. En el panel **Información general**, seleccione **Detener** para desasignar la máquina virtual.  

    d. Seleccione **Redes** y adjunte la interfaz de red. En la lista desplegable **Adjuntar interfaz de red**, seleccione las interfaces de red ya creadas para las subredes `inter` y `hsr`.  
    
    e. Seleccione **Guardar**. 
 
    f. Repita los pasos del b al e para las demás máquinas virtuales (en nuestro ejemplo, **hana-s1-db2**, **hana-s1-db3**, **hana-s2-db1**, **hana-s2-db2** y **hana-s2-db3**).
 
    g. Deje las máquinas virtuales en estado detenido por ahora. A continuación, habilitaremos la opción [Redes aceleradas](../../../virtual-network/create-vm-accelerated-networking-cli.md) para todas las interfaces de red que se acaban de conectar.  

5. Habilite redes aceleradas para las interfaces de red adicionales de las subredes `inter` y `hsr`. Para ello, siga estos pasos:  

    a. Abra [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) en [Azure Portal](https://portal.azure.com/#home).  

    b. Ejecute los siguientes comandos para habilitar las redes aceleradas para las interfaces de red adicionales, conectadas a las subredes `inter` y `hsr`.  

    ```azurecli
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db1-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db2-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db3-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db1-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db2-inter --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db3-inter --accelerated-networking true
    
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db1-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db2-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s1-db3-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db1-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db2-hsr --accelerated-networking true
    az network nic update --id /subscriptions/your subscription/resourceGroups/your resource group/providers/Microsoft.Network/networkInterfaces/hana-s2-db3-hsr --accelerated-networking true
    ```

7. Inicie las máquinas virtuales de base de datos HANA.

### <a name="deploy-azure-load-balancer"></a>Implementación de Azure Load Balancer

1. Se recomienda usar Standard Load Balancer. Siga estos pasos de configuración para implementar Standard Load Balancer:
   1. Primero, cree un grupo de direcciones IP de front-end:

      1. Abra el equilibrador de carga, seleccione **frontend IP pool** (Grupo de direcciones IP de front-end) y haga clic en **Agregar**.
      1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **hana-front-end**).
      1. Establezca **Asignación** en **Estática** y escriba la dirección IP (por ejemplo, **10.23.0.27**).
      1. Seleccione **Aceptar**.
      1. Una vez creado el nuevo grupo de direcciones IP de front-end, anote la dirección IP del grupo.

   1. A continuación, cree un grupo de back-end y agregue todas las máquinas virtuales de clúster al grupo de back-end:

      1. Abra el equilibrador de carga, seleccione **Grupos de back-end** y haga clic en **Agregar**.
      1. Escriba el nombre del nuevo grupo de back-end (por ejemplo, **hana-backend**).
      1. Seleccione **Agregar una máquina virtual**.
      1. Seleccione **Máquina virtual**.
      1. Seleccione las máquinas virtuales del clúster de SAP HANA y sus direcciones IP para la subred `client`.
      1. Seleccione **Agregar**.

   1. A continuación, cree un sondeo de estado:

      1. Abra el equilibrador de carga, seleccione **Sondeos de estado** y haga clic en **Agregar**.
      1. Escriba el nombre del sondeo de estado nuevo (por ejemplo **hana-hp**).
      1. Seleccione **TCP** como protocolo y el puerto 625 **03**. Mantenga el valor de **Intervalo** en 5 y el valor de **Umbral incorrecto** en 2.
      1. Seleccione **Aceptar**.

   1. Luego cree las reglas de equilibrio de carga:
   
      1. Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
      1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **hana-lb**).
      1. Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que ha creado anteriormente (por ejemplo, **hana-frontend**, **hana-backend** y **hana-hp**).
      1. Seleccione **Puertos HA**.
      1. Asegúrese de **habilitar la dirección IP flotante**.
      1. Seleccione **Aceptar**.

   > [!IMPORTANT]
   > La dirección IP flotante no se admite en una configuración de IP secundaria de NIC en escenarios de equilibrio de carga. Para ver detalles, consulte [Limitaciones de Azure Load Balancer](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si necesita una dirección IP adicional para la VM, implemente una segunda NIC.    
   
   > [!Note]
   > Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  


   > [!IMPORTANT]
   > No habilite las marcas de tiempo TCP en VM de Azure que se encuentren detrás de Azure Load Balancer. Si habilita las marcas de tiempo TCP provocará un error en los sondeos de estado. Establezca el parámetro **net.ipv4.tcp_timestamps** a **0**. Consulte [Sondeos de estado de Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md) para obtener más información.
   > Consulte también la nota de SAP [2382421](https://launchpad.support.sap.com/#/notes/2382421).  

### <a name="deploy-the-azure-netapp-files-infrastructure"></a>Implementación de la infraestructura de Azure NetApp Files 

Implemente los volúmenes de ANF para el sistema de archivos `/hana/shared`. Necesitará un volumen `/hana/shared` independiente para cada sitio de replicación del sistema HANA. Para más información, consulte [Configuración de la infraestructura de Azure NetApp Files](./sap-hana-scale-out-standby-netapp-files-suse.md#set-up-the-azure-netapp-files-infrastructure).

En este ejemplo, se usan los siguientes volúmenes de Azure NetApp Files: 

* volumen **HN1**-shared-s1 (nfs://10.23.1.7/**HN1**-shared-s1)
* volumen **HN1**-shared-s2 (nfs://10.23.1.7/**HN1**-shared-s2)


## <a name="operating-system-configuration-and-preparation"></a>Configuración y preparación del sistema operativo

Las instrucciones de las secciones siguientes tienen como prefijo una de las siguientes abreviaturas:
* **[A]** :      Aplicable a todos los nodos
* **[AH]** :     aplicable a todos los nodos de base de datos HANA
* **[M]** :      aplicable al nodo que sirve para lograr la mayoría
* **[AH1]** :    aplicable a todos los nodos de base de datos HANA en el sitio 1
* **[AH2]** :    aplicable a todos los nodos de base de datos HANA en el sitio 2
* **[1]** :      aplicable solo al nodo 1 de base de datos HANA, sitio 1
* **[2]** :      aplicable solo al nodo 1 de base de datos HANA, sitio 2

Configure y prepare el sistema operativo. Para ello, siga estos pasos:

1. **[A]** Mantenga los archivos de host en las máquinas virtuales. Incluya las entradas para todas las subredes. En este ejemplo se agregaron las siguientes entradas a `/etc/hosts`.  

    ```bash
     # Client subnet
     10.23.0.19      hana-s1-db1
     10.23.0.20      hana-s1-db2
     10.23.0.21      hana-s1-db3
     10.23.0.22      hana-s2-db1
     10.23.0.23      hana-s2-db2
     10.23.0.24      hana-s2-db3
     10.23.0.25      hana-s-mm    
     # Internode subnet
     10.23.1.132     hana-s1-db1-inter
     10.23.1.133     hana-s1-db2-inter
     10.23.1.134     hana-s1-db3-inter
     10.23.1.135     hana-s2-db1-inter
     10.23.1.136     hana-s2-db2-inter
     10.23.1.137     hana-s2-db3-inter
     # HSR subnet
     10.23.1.196     hana-s1-db1-hsr
     10.23.1.197     hana-s1-db2-hsr
     10.23.1.198     hana-s1-db3-hsr
     10.23.1.199     hana-s2-db1-hsr
     10.23.1.200     hana-s2-db2-hsr
     10.23.1.201     hana-s2-db3-hsr
    ```

2. **[A]** SUSE ofrece agentes de recursos especiales para SAP HANA y, de forma predeterminada, los agentes para la escalabilidad vertical de SAP HANA están instalados. Desinstale los paquetes de escalabilidad vertical, si están instalados, e instale los paquetes para el escenario SAP HANAScaleOut. Este paso debe realizarse en todas las VM del clúster, incluido el creador mayoritario.   

    ```bash
     # Uninstall ScaleUp packages and patterns
     zypper remove patterns-sap-hana
     zypper remove SAPHanaSR 
     zypper remove SAPHanaSR-doc
     zypper remove yast2-sap-ha
     # Install the ScaleOut packages and patterns
     zypper in SAPHanaSR-ScaleOut  SAPHanaSR-ScaleOut-doc 
     zypper in -t pattern ha_sles
    ```

3. **[AH]** Preparación de las VM: aplique la configuración recomendada según la nota de SAP [2205917] para las aplicaciones de SAP de SUSE Linux Enterprise Server.  

## <a name="prepare-the-file-systems"></a>Preparación de los sistemas de archivos
### <a name="mount-the-shared-file-systems"></a>Montaje de los sistema de archivos compartidos

En este ejemplo, los sistemas de archivos de HANA compartidos se implementan en Azure NetApp Files y se montan en NFSv4.  

1. **[AH]** Cree puntos de montaje para los volúmenes de bases de datos HANA.  

    ```bash
    mkdir -p /hana/shared
    ```

2. **[AH]** Compruebe la configuración del dominio NFS. Asegúrese de que el dominio esté configurado como dominio predeterminado de Azure NetApp Files, es decir, **`defaultv4iddomain.com`** , y que la asignación se haya establecido en **nobody**.  
   Este paso solo es necesario si usa Azure NetAppFiles NFSv4.1.  

    > [!IMPORTANT]
    > Asegúrese de establecer el dominio NFS de `/etc/idmapd.conf` en la máquina virtual para que coincida con la configuración de dominio predeterminada en Azure NetApp Files: **`defaultv4iddomain.com`** . Si hay alguna discrepancia entre la configuración de dominio del cliente NFS (es decir, la máquina virtual) y el servidor NFS (es decir la configuración de Azure NetApp), los permisos de archivos en volúmenes de Azure NetApp que estén montados en las máquinas virtuales se mostrarán como `nobody`.  

    ```bash
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = defaultv4iddomain.com
    [Mapping]
    Nobody-User = nobody
    Nobody-Group = nobody
    ```

3. **[AH]** Compruebe `nfs4_disable_idmapping`. Debe establecerse en **S**. Para crear la estructura de directorio en la que se encuentra `nfs4_disable_idmapping`, ejecute el comando mount. No podrá crear manualmente el directorio en /sys/modules, ya que el acceso está reservado para el kernel o los controladores.  
   Este paso solo es necesario si usa Azure NetAppFiles NFSv4.1.  

    ```bash
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 10.23.1.7:/HN1-share-s1 /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    ```

4. **[AH1]** Monte los volúmenes de Azure NetApp Files compartidos en las máquinas virtuales de base de datos HANA del sitio 1.  

    ```bash
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.7:/HN1-shared-s1 /hana/shared nfs rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount all volumes
    sudo mount -a 
    ```

5. **[AH2]** Monte los volúmenes de Azure NetApp Files compartidos en las máquinas virtuales de base de datos HANA del sitio 2.  

    ```bash
    sudo vi /etc/fstab
    # Add the following entries
    10.23.1.7:/HN1-shared-s2 /hana/shared nfs rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    ```


10. **[AH]** Compruebe que los sistemas de archivos `/hana/shared/` correspondientes estén montados en todas las máquinas virtuales de base de datos HANA con la versión del protocolo NFS **NFSv4**.  

    ```bash
    sudo nfsstat -m
    # Verify that flag vers is set to 4.1 
    # Example from SITE 1, hana-s1-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s1
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.11,local_lock=none,addr=10.23.1.7
    # Example from SITE 2, hana-s2-db1
    /hana/shared from 10.23.1.7:/HN1-shared-s2
     Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.23.0.14,local_lock=none,addr=10.23.1.7
    ```

### <a name="prepare-the-data-and-log-local-file-systems"></a>Preparación de los sistemas de archivos locales de registro y datos
En la configuración presentada, los sistemas de archivos `/hana/data` y `/hana/log` están implementados en el disco administrado y conectados localmente a cada máquina virtual de base de datos HANA. Tendrá que ejecutar los pasos necesarios para crear los volúmenes locales de datos y de registro en cada máquina virtual de base de datos HANA. 

Configure el diseño de disco con el **Administrador de volúmenes lógicos (LVM)** . En el ejemplo siguiente se supone que cada máquina virtual HANA tiene tres discos de datos conectados que se usan para crear dos volúmenes.

1. **[AH]** Enumere todos los discos disponibles:
    ```bash
    ls /dev/disk/azure/scsi1/lun*
    ```

   Salida de ejemplo:

    ```bash
    /dev/disk/azure/scsi1/lun0  /dev/disk/azure/scsi1/lun1  /dev/disk/azure/scsi1/lun2 
    ```

2. **[AH]** Cree volúmenes físicos para todos los discos que quiera usar:
    ```bash
    sudo pvcreate /dev/disk/azure/scsi1/lun0
    sudo pvcreate /dev/disk/azure/scsi1/lun1
    sudo pvcreate /dev/disk/azure/scsi1/lun2
    ```

3. **[AH]** Cree un grupo de volúmenes para los archivos de datos. Use un grupo de volúmenes para los archivos de registro y otro para el directorio compartido de SAP HANA:
    ```bash
    sudo vgcreate vg_hana_data_HN1 /dev/disk/azure/scsi1/lun0 /dev/disk/azure/scsi1/lun1
    sudo vgcreate vg_hana_log_HN1 /dev/disk/azure/scsi1/lun2
    ```

4. **[AH]** Cree los volúmenes lógicos. 
   Cuando se usa `lvcreate` sin el modificador `-i`, se crea un volumen lineal. Se recomienda crear un volumen seccionado para mejorar el rendimiento de E/S y, después, alinear los tamaños de sección con los valores documentados en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md). El argumento `-i` debe ser el número de volúmenes físicos subyacentes y `-I`, el tamaño de la sección. En este documento, se usan dos volúmenes físicos para el volumen de datos, por lo que el modificador `-i` se establece en **2**. El tamaño de sección para el volumen de datos es **256 KiB**. Se usa un volumen físico para el volumen de registro, así que no se usa explícitamente ningún modificador `-i` o `-I` para los comandos del volumen de registro.  

   > [!IMPORTANT]
   > Use el modificador `-i` y establézcalo en el número del volumen físico subyacente cuando se usa más de un volumen físico para cada volumen de datos o de registro. Use el modificador `-I` para especificar el tamaño de las secciones al crear un volumen seccionado.  
   > Consulte [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md) para conocer las configuraciones de almacenamiento recomendadas, incluidos los tamaños de sección y el número de discos.  

    ```bash
    sudo lvcreate -i 2 -I 256 -l 100%FREE -n hana_data vg_hana_data_HN1
    sudo lvcreate -l 100%FREE -n hana_log vg_hana_log_HN1
    sudo mkfs.xfs /dev/vg_hana_data_HN1/hana_data
    sudo mkfs.xfs /dev/vg_hana_log_HN1/hana_log
    ```

5. **[AH]** Cree los directorios de montaje y copie el UUID de todos los volúmenes lógicos:
    ```bash
    sudo mkdir -p /hana/data/HN1
    sudo mkdir -p /hana/log/HN1
    # Write down the ID of /dev/vg_hana_data_HN1/hana_data and /dev/vg_hana_log_HN1/hana_log
    sudo blkid
    ```

6. **[AH]** Cree entradas `fstab` para los volúmenes lógicos y móntelos:
    ```bash
    sudo vi /etc/fstab
    ```

   Inserte la siguiente línea en el archivo `/etc/fstab`:

    ```bash
    /dev/disk/by-uuid/UUID of /dev/mapper/vg_hana_data_HN1-hana_data /hana/data/HN1 xfs  defaults,nofail  0  2
    /dev/disk/by-uuid/UUID of /dev/mapper/vg_hana_log_HN1-hana_log /hana/log/HN1 xfs  defaults,nofail  0  2
    ```

   Monte los nuevos volúmenes:

    ```bash
    sudo mount -a
    ```

## <a name="create-a-pacemaker-cluster"></a>Creación de un clúster de Pacemaker

Siga los pasos de [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](high-availability-guide-suse-pacemaker.md) para crear un clúster de Pacemaker básico para este servidor HANA.
Incluya todas las máquinas virtuales, inclusive la que sirve para obtener la mayoría en el clúster.  

> [!IMPORTANT]
> No establezca `quorum expected-votes` en 2, ya que no es un clúster de dos nodos.  
> Asegúrese de que la propiedad de clúster `concurrent-fencing` esté habilitada, de modo que se deserialicen las barreras de nodo.   

## <a name="installation"></a>Instalación  

En este ejemplo para implementar SAP HANA en la configuración de escalabilidad horizontal con HSR en máquinas virtuales de Azure, usamos HANA 2.0 SP4.  

### <a name="prepare-for-hana-installation"></a>Preparación de la instalación de HANA

1. **[AH]** Antes de realizar la instalación de HANA, establezca la contraseña raíz. Puede deshabilitar la contraseña raíz una vez completada la instalación. Ejecute como `passwd` del comando `root`.  

2. **[1,2]** Cambie los permisos para `/hana/shared`. 
    ```bash
    chmod 775 /hana/shared
    ```

3. **[1]** Compruebe que puede iniciar sesión a través de SSH en las máquinas virtuales de base de datos HANA en **hana-s1-db2** y **hana-s1-db3** del sitio, sin que se le pida una contraseña. Si ese no es el caso, intercambie las claves SSH tal como se describe en [Habilitación del acceso SSH a través de la clave pública](https://documentation.suse.com/sbp/all/html/SLES4SAP-hana-scaleOut-PerfOpt-12/index.html#_enable_ssh_access_via_public_key_optional).  
    ```bash
    ssh root@hana-s1-db2
    ssh root@hana-s1-db3
    ```

4. **[2]** Compruebe que puede iniciar sesión a través de SSH en las máquinas virtuales de base de datos HANA en **hana-s2-db2** y **hana-s2-db3** del sitio, sin que se le pida una contraseña.  
   Si ese no es el caso, intercambie las claves ssh.    
    ```bash
    ssh root@hana-s2-db2
    ssh root@hana-s2-db3
    ```

5. **[AH]** Instale paquetes adicionales, necesarios para HANA 2.0 SP4. Para obtener más información, consulte las notas de SAP [2593824](https://launchpad.support.sap.com/#/notes/2593824) para la versión de SLES. 

    ```bash
    # In this example, using SLES12 SP5
    sudo zypper install libgcc_s1 libstdc++6 libatomic1
    ```
### <a name="hana-installation-on-the-first-node-on-each-site"></a>Instalación de HANA en el primer nodo de cada sitio

1. **[1]** Instale SAP HANA. Para ello, siga las instrucciones de la [guía de instalación y actualización de SAP HANA 2.0](https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.04/en-US/7eb0167eb35e4e2885415205b8383584.html). En las instrucciones que se indican a continuación, se muestra la instalación de SAP HANA en el primer nodo del sitio 1.   

   a. Inicie el programa **hdblcm** como `root` desde el directorio de software de instalación de HANA. Use el parámetro `internal_network` y pase el espacio de direcciones a la subred que se usa para la comunicación interna entre nodos de HANA.  

    ```bash
    ./hdblcm --internal_network=10.23.1.128/26
    ```

   b. Escriba los siguientes valores en el símbolo del sistema:

     * En **Elegir una acción**: escriba **1** (para la instalación)
     * En **Additional components for installation** (Componentes adicionales para la instalación): especifique **2, 3**
     * Para la ruta de instalación: presione Entrar (el valor predeterminado es/Hana/Shared).
     * En **Local Host Name** (Nombre de host local): presione Entrar para aceptar el valor predeterminado.
     * En **Do you want to add hosts to the system?** (¿Desea agregar hosts al sistema): escriba **y**.
     * En **SAP HANA ID** (Id. de SAP HANA) del sistema: escriba **HN1**.
     * En **Instance number** (Número de instancia) [00]: escriba **03**.
     * En **Local Host Worker Group** (Grupo de trabajo de host local) [valor predeterminado]: presione Entrar para aceptar el valor predeterminado.
     * En **Select System Usage / Enter index [4]** (Seleccionar uso del sistema o especificar el índice [4]): escriba **4** (para personalizar).
     * En **Location of Data Volumes** (Ubicación de los volúmenes de datos) [/hana/data/HN1]: presione Entrar para aceptar el valor predeterminado.
     * En **Location of Log Volumes** (Ubicación de los volúmenes de registro) [/hana/log/HN1]: presione Entrar para aceptar el valor predeterminado.
     * En **Restrict maximum memory allocation?** (¿Restringir la asignación de memoria máxima?) [n]: especifique **n**.
     * En **Certificate Host Name For Host hana-s1-db1** (Nombre de host del certificado para el host hana-s1-db1) [hana-s1-db1]: presione Entrar para aceptar el valor predeterminado.
     * En **SAP Host Agent User (sapadm) Password** [Contraseña del usuario del agente de host de SAP (sapadm)]: escriba la contraseña.
     * En **Confirm SAP Host Agent User (sapadm) Password** [Confirmar contraseña de usuario del agente de host de SAP (sapadm)]: escriba la contraseña.
     * En **System Administrator (hn1adm) Password** (Contraseña del administrador del sistema (hn1adm)): escriba la contraseña.
     * En **System Administrator Home Directory** (Directorio principal de administrador del sistema) [/usr/sap/HN1/home]: presione Entrar para aceptar el valor predeterminado.
     * En **System Administrator Login Shell** (Shell de inicio de sesión de administrador del sistema) [/bin/sh]: presione Entrar para aceptar el valor predeterminado.
     * En **System Administrator User ID** (Id. de usuario de administrador del sistema) [1001]: presione Entrar para aceptar el valor predeterminado.
     * En **Enter ID of User Group (sapsys)** [Escriba el id. del grupo de usuarios (sapsys)] [79]: presione Entrar para aceptar el valor predeterminado.
     * En **System Database User (system) Password** (Contraseña de usuario de base de datos del sistema (sistema)): escriba la contraseña del sistema.
     * En **Confirm System Database User (system) Password** (Confirmar la contraseña del usuario (sistema) de la base de datos del sistema): escriba la contraseña del sistema.
     * En **Restart system after machine reboot?** (¿Reiniciar el sistema tras el reinicio de la máquina?) [n]: especifique **n**. 
     * En **Do you want to continue (y/n)** (¿Desea continuar (sí/no)?): valide el resumen y, si todo es correcto, especifique **s**.

2. **[2]** Repita el paso anterior para instalar SAP HANA en el primer nodo del sitio 2.   

3. **[1,2]** Compruebe global.ini.  

   Muestre global.ini y asegúrese de que la configuración de la comunicación interna entre nodos de SAP HANA esté presente. Compruebe la sección de **communication**. Debe tener el espacio de direcciones para la subred `inter`, y `listeninterface` debe estar establecido en `.internal`. Compruebe la sección **internal_hostname_resolution**. Debe tener las direcciones IP de las máquinas virtuales de HANA que pertenezcan a la subred `inter`.  

   ```bash
     sudo cat /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
     # Example from SITE1 
     [communication]
     internal_network = 10.23.1.128/26
     listeninterface = .internal
     [internal_hostname_resolution]
     10.23.1.132 = hana-s1-db1
     10.23.1.133 = hana-s1-db2
     10.23.1.134 = hana-s1-db3
   ```

4. **[1,2]** Prepare `global.ini` para la instalación en un entorno no compartido, como se describe en la nota de SAP [2080991](https://launchpad.support.sap.com/#/notes/0002080991).  

   ```bash
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    [persistence]
    basepath_shared = no
   ```

4. **[1,2]** Reinicie SAP HANA para activar los cambios.  

   ```bash
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StopSystem
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StartSystem
   ```

6. **[1,2]** Compruebe que la interfaz de cliente utilizará las direcciones IP de la subred `client` para la comunicación.  

    ```bash
    # Execute as hn1adm
    /usr/sap/HN1/HDB03/exe/hdbsql -u SYSTEM -p "password" -i 03 -d SYSTEMDB 'select * from SYS.M_HOST_INFORMATION'|grep net_publicname
    # Expected result - example from SITE 2
    "hana-s2-db1","net_publicname","10.23.0.22"
   ```

   Para más información sobre cómo comprobar la configuración, consulte la nota de SAP [2183363 - Configuration of SAP HANA internal network](https://launchpad.support.sap.com/#/notes/2183363) (Configuración de la red interna de SAP HANA).  

7. **[AH]** Cambie los permisos en los directorios de datos y de registro para evitar errores de instalación de HANA.  

   ```bash
    sudo chmod o+w -R /hana/data /hana/log
   ```

8. **[1]** Instale los nodos secundarios de HANA. Las instrucciones de ejemplo de este paso son para el sitio 1.  
   a. Inicie el programa residente **hdblcm** como `root`.    
    ```bash
     cd /hana/shared/HN1/hdblcm
     ./hdblcm 
    ```

   b. Escriba los siguientes valores en el símbolo del sistema:

     * En **Elegir una acción**: escriba **2** (para agregar hosts).
     * En **Enter comma-separated host names to add** (Escriba los nombres de host separados por comas que se deben agregar): hana-s1-db2, hana-s1-db3.
     * En **Additional components for installation** (Componentes adicionales para la instalación): especifique **2, 3**
     * En **Enter Root User Name [root]** (Escriba el nombre de usuario raíz [raíz]): presione Entrar para aceptar el valor predeterminado.
     * En **Select roles for host 'hana-s1-db2' [1]** (Seleccione los roles para el host "hana-s1-db2" [1]): 1 (para trabajo)
     * En **Enter Host Failover Group for host 'hana-s1-db2' [default]** (Escriba el grupo de conmutación por error de host para el host "hana-s1-db2" [predeterminado]): presione Entrar para aceptar el valor predeterminado.
     * En **Enter Storage Partition Number for host 'hana-s1-db2' [<<assign automatically>>]** (Escriba el número de partición de almacenamiento para el host "hana-s1-db2" [<>]): presione Entrar para aceptar el valor predeterminado.
     * En **Enter Worker Group for host 'hana-s1-db2' [default]** (Escriba el grupo de trabajo para el host "hana-s1-db2" [predeterminado]): presione Entrar para aceptar el valor predeterminado.
     * En **Select roles for host 'hana-s1-db3' [1]** (Seleccione los roles para el host "hana-s1-db3" [1]): 1 (para trabajo)
     * En **Enter Host Failover Group for host 'hana-s1-db3' [default]** (Escriba el grupo de conmutación por error de host para el host "hana-s1-db3" [predeterminado]): presione Entrar para aceptar el valor predeterminado.
     * En **Enter Storage Partition Number for host 'hana-s1-db3' [<<assign automatically>>]** (Escriba el número de partición de almacenamiento para el host "hana-s1-db3" [<>]): presione Entrar para aceptar el valor predeterminado.
     * En **Enter Worker Group for host 'hana-s1-db3' [default]** (Escriba el grupo de trabajo para el host "hana-s1-db3" [predeterminado]): presione Entrar para aceptar el valor predeterminado.
     * En **System Administrator (hn1adm) Password** (Contraseña del administrador del sistema (hn1adm)): escriba la contraseña.
     * En **Enter SAP Host Agent User (sapadm) Password** [Escriba la contraseña del usuario del agente de host de SAP (sapadm)]: escriba la contraseña.
     * En **Confirm SAP Host Agent User (sapadm) Password** [Confirmar contraseña de usuario del agente de host de SAP (sapadm)]: escriba la contraseña.
     * En **Certificate Host Name For Host hana-s1-db2** (Nombre de host del certificado para el host hana-s1-db2) [hana-s1-db2]: presione Entrar para aceptar el valor predeterminado.
     * En **Certificate Host Name For Host hana-s1-db3** (Nombre de host del certificado para el host hana-s1-db3) [hana-s1-db3]: presione Entrar para aceptar el valor predeterminado.
     * En **Do you want to continue (y/n)** (¿Desea continuar (sí/no)?): valide el resumen y, si todo es correcto, especifique **s**.

9. **[2]** Repita el paso anterior para instalar los nodos de SAP HANA secundarios en el sitio 2.   

## <a name="configure-sap-hana-20-system-replication"></a>Configuración de la Replicación del sistema de SAP HANA 2.0

1. **[1]** Configure la replicación del sistema en el sitio 1:

   Haga una copia de seguridad de las bases de datos como **hn1** adm:

    ```bash
    hdbsql -d SYSTEMDB -u SYSTEM -p "passwd" -i 03 "BACKUP DATA USING FILE ('initialbackupSYS')"
    hdbsql -d HN1 -u SYSTEM -p "passwd" -i 03 "BACKUP DATA USING FILE ('initialbackupHN1')"
    ```

   Copie los archivos PKI de sistema en el sitio secundario:

    ```bash
    scp /usr/sap/HN1/SYS/global/security/rsecssfs/data/SSFS_HN1.DAT hana-s2-db1:/usr/sap/HN1/SYS/global/security/rsecssfs/data/
    scp /usr/sap/HN1/SYS/global/security/rsecssfs/key/SSFS_HN1.KEY  hana-s2-db1:/usr/sap/HN1/SYS/global/security/rsecssfs/key/
    ```

   Cree el sitio principal:

    ```bash
    hdbnsutil -sr_enable --name=HANA_S1
    ```

2. **[2]** Configure la replicación del sistema en el sitio 2:
    
   Registre el segundo sitio para iniciar la replicación del sistema. Ejecute el siguiente comando como <hanasid\>adm:

    ```bash
    sapcontrol -nr 03 -function StopWait 600 10
    hdbnsutil -sr_register --remoteHost=hana-s1-db1 --remoteInstance=03 --replicationMode=sync --name=HANA_S2
    sapcontrol -nr 03 -function StartSystem
    ```

3. **[1]** Comprobación del estado de replicación

   Compruebe el estado de replicación y espere hasta que todas las bases de datos estén sincronizadas.

    ```bash
    sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
    # | Database | Host          | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary     | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
    # |          |               |       |              |           |         |           | Host          | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
    # | -------- | ------------- | ----- | ------------ | --------- | ------- | --------- | ------------- | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
    # | HN1      | hana-s1-db3   | 30303 | indexserver  |         5 |       1 | HANA_S1   | hana-s2-db3   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | SYSTEMDB | hana-s1-db1   | 30301 | nameserver   |         1 |       1 | HANA_S1   | hana-s2-db1   |     30301 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db1   | 30307 | xsengine     |         2 |       1 | HANA_S1   | hana-s2-db1   |     30307 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db1   | 30303 | indexserver  |         3 |       1 | HANA_S1   | hana-s2-db1   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    # | HN1      | hana-s1-db2   | 30303 | indexserver  |         4 |       1 | HANA_S1   | hana-s2-db2   |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
    #
    # status system replication site "2": ACTIVE
    # overall system replication status: ACTIVE
    #
    # Local System Replication State
    #   
    # mode: PRIMARY
    # site id: 1
    # site name: HANA_S1
    ```

4. **[1,2]** Cambie la configuración de HANA para que la comunicación para la replicación del sistema HANA se direccione a través de las interfaces de red virtual de replicación del sistema HANA.   
   - Detenga HANA en ambos sitios
    ```bash
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StopSystem HDB
    ```

   - Edite global. ini para agregar la asignación de host para la replicación del sistema HANA: use las direcciones IP de la subred `hsr`.  
    ```bash
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    #Add the section
    [system_replication_hostname_resolution]
    10.23.1.196 = hana-s1-db1
    10.23.1.197 = hana-s1-db2
    10.23.1.198 = hana-s1-db3
    10.23.1.199 = hana-s2-db1
    10.23.1.200 = hana-s2-db2
    10.23.1.201 = hana-s2-db3
    ```

   - Inicie HANA en ambos sitios
   ```bash
    sudo -u hn1adm /usr/sap/hostctrl/exe/sapcontrol -nr 03 -function StartSystem HDB
   ```

   Para más información, vea [Resolución del nombre de host para la replicación del sistema](https://help.sap.com/viewer/eb3777d5495d46c5b2fa773206bbfb46/1.0.12/en-US/c0cba1cb2ba34ec89f45b48b2157ec7b.html).  

## <a name="create-file-system-resources"></a>Creación de recursos del sistema de archivos

Cree un recurso de clúster de un sistema de archivos ficticio, que supervisará y notificará los errores, en caso de que haya un problema al acceder al sistema de archivos `/hana/shared` montado en NFS. Esto permite que el clúster desencadene la conmutación por error, en caso de que se produzca un problema al acceder a `/hana/shared`. Para obtener más detalles, consulte [Control de recursos compartidos de NFS con errores en el clúster de alta disponibilidad de SUSE para la replicación del sistema HANA](https://www.suse.com/support/kb/doc/?id=000019904). 

1. **[1]** Coloque Pacemaker en modo de mantenimiento, como preparación para la creación de los recursos de clúster de HANA.  
    ```bash
    crm configure property maintenance-mode=true
    ```

2. **[1,2]** Cree el directorio en el volumen ANF/hana/sahred, que se usará en el recurso de supervisión del sistema de archivos especial. Los directorios deben crearse en ambos sitios.  
    ```bash
    mkdir -p /hana/shared/HN1/check
    ```

2. **[AH]** Cree el directorio que se usará para montar el recurso de supervisión del sistema de archivos especial. El directorio debe crearse en todos los nodos del clúster de HANA.  
    ```bash
    mkdir -p /hana/check
    ```

3. **[1]** Cree los recursos de clúster del sistema de archivos.     

    ```bash
    crm configure primitive fs_HN1_HDB03_fscheck Filesystem \
      params device="/hana/shared/HN1/check" \
      directory="/hana/check" fstype=nfs4 \
      options="bind,defaults,rw,hard,proto=tcp,intr,noatime,vers=4.1,lock" \
      op monitor interval=120 timeout=120 on-fail=fence \
      op_params OCF_CHECK_LEVEL=20 \
      op start interval=0 timeout=120 op stop interval=0 timeout=120

    crm configure clone cln_fs_HN1_HDB03_fscheck fs_HN1_HDB03_fscheck \
      meta clone-node-max=1 interleave=true

    crm configure location loc_cln_fs_HN1_HDB03_fscheck_not_on_mm \
      cln_fs_HN1_HDB03_fscheck -inf: hana-s-mm    
    ```
 
   El atributo `OCF_CHECK_LEVEL=20` se agrega a la operación de supervisión, de modo que las operaciones de supervisión realicen una prueba de lectura/escritura en el sistema de archivos. Sin este atributo, la operación de supervisión solo comprueba que el sistema de archivos esté montado. Esto puede ser un problema porque, cuando se pierde la conectividad, el sistema de archivos puede permanecer montado, a pesar de que no se pueda acceder.  

   El atributo `on-fail=fence` también se agrega a la operación de supervisión. Con esta opción, si se produce un error en la operación de supervisión en un nodo, ese nodo se delimita inmediatamente.   

## <a name="create-sap-hana-cluster-resources"></a>Creación de recursos de clúster de SAP HANA

1. **[1,2]** Instale el "enlace de replicación del sistema" de HANA. El enlace debe instalarse en un nodo de la base de datos HANA en cada sitio de replicación del sistema.         

   1. Prepare el enlace como `root`. 
    ```bash
     mkdir -p /hana/shared/myHooks
     cp /usr/share/SAPHanaSR-ScaleOut/SAPHanaSR.py /hana/shared/myHooks
     chown -R hn1adm:sapsys /hana/shared/myHooks
    ```

   2. Detenga HANA en ambos sitios de replicación. Ejecútelo como <sid\>adm:
    ```bash
    sapcontrol -nr 03 -function StopSystem
    ```

   3. Ajuste `global.ini`.
    ```bash
    # add to global.ini
    [ha_dr_provider_SAPHanaSR]
    provider = SAPHanaSR
    path = /hana/shared/myHooks
    execution_order = 1
    
    [trace]
    ha_dr_saphanasr = info
    ```

2. **[AH]** El clúster requiere la configuración de sudoers en el nodo de clúster para <sid\>adm. En este ejemplo, esto se consigue mediante la creación de un archivo nuevo. Ejecute los comandos como `root`.    
    ```bash
    cat << EOF > /etc/sudoers.d/20-saphana
    # SAPHanaSR-ScaleOut needs for srHook
    Cmnd_Alias SOK = /usr/sbin/crm_attribute -n hana_hn1_glob_srHook -v SOK -t crm_config -s SAPHanaSR
    Cmnd_Alias SFAIL = /usr/sbin/crm_attribute -n hana_hn1_glob_srHook -v SFAIL -t crm_config -s SAPHanaSR
    hn1adm ALL=(ALL) NOPASSWD: SOK, SFAIL
    EOF
    ```

3. **[1,2]** Inicie SAP HANA en ambos sitios de replicación. Ejecute como <sid\>adm.  

    ```bash
    sapcontrol -nr 03 -function StartSystem 
    ```

4. **[1]** Compruebe la instalación del enlace. Ejecute as <sid\>adm en el sitio activo de replicación del sistema HANA.   

    ```bash
    cdtrace
     awk '/ha_dr_SAPHanaSR.*crm_attribute/ \
     { printf "%s %s %s %s\n",$2,$3,$5,$16 }' nameserver_*

     # 2021-03-31 01:02:42.695244 ha_dr_SAPHanaSR SFAIL
     # 2021-03-31 01:02:58.966856 ha_dr_SAPHanaSR SFAIL
     # 2021-03-31 01:03:04.453100 ha_dr_SAPHanaSR SFAIL
     # 2021-03-31 01:03:04.619768 ha_dr_SAPHanaSR SFAIL
     # 2021-03-31 01:03:04.743444 ha_dr_SAPHanaSR SFAIL
     # 2021-03-31 01:04:15.062181 ha_dr_SAPHanaSR SOK

    ```

5. **[1]** Cree recursos de clúster de HANA. Ejecute los siguientes comandos como `root`.    
   1. Asegúrese de que el clúster ya esté en modo de mantenimiento.  
    
   2. A continuación, cree el recurso de topología de HANA.  
      ```bash
      sudo crm configure primitive rsc_SAPHanaTopology_HN1_HDB03 ocf:suse:SAPHanaTopology \
        op monitor interval="10" timeout="600" \
        op start interval="0" timeout="600" \
        op stop interval="0" timeout="300" \
        params SID="HN1" InstanceNumber="03"

      sudo crm configure clone cln_SAPHanaTopology_HN1_HDB03 rsc_SAPHanaTopology_HN1_HDB03 \
       meta clone-node-max="1" target-role="Started" interleave="true"
      ```

   3. A continuación, cree el recurso de instancia de HANA.  
      > [!NOTE] 
      > Este artículo contiene referencias a los términos *maestro* y *esclavo*, unos términos que Microsoft ya no usa. Cuando se eliminen estos términos del software, se eliminarán también de este artículo.
 
      ```bash
      sudo crm configure primitive rsc_SAPHana_HN1_HDB03 ocf:suse:SAPHanaController \
        op start interval="0" timeout="3600" \
        op stop interval="0" timeout="3600" \
        op promote interval="0" timeout="3600" \
        op monitor interval="60" role="Master" timeout="700" \
        op monitor interval="61" role="Slave" timeout="700" \
        params SID="HN1" InstanceNumber="03" PREFER_SITE_TAKEOVER="true" \
        DUPLICATE_PRIMARY_TIMEOUT="7200" AUTOMATED_REGISTER="false"

      sudo crm configure ms msl_SAPHana_HN1_HDB03 rsc_SAPHana_HN1_HDB03 \
        meta clone-node-max="1" master-max="1" interleave="true"
      ```
      > [!IMPORTANT]
      > Se recomienda como procedimiento recomendado que solo establezca AUTOMATED_REGISTER en **no** mientras realice pruebas de conmutación por error exhaustivas, para evitar que la instancia principal con errores se registre automáticamente como secundaria. Una vez que las pruebas de conmutación por error se hayan completado correctamente, establezca AUTOMATED_REGISTER en **yes**, de modo que, después de la toma de control, la replicación del sistema se pueda reanudar automáticamente. 

   4. Cree una dirección IP virtual y los recursos asociados.  
      ```bash
      sudo crm configure primitive rsc_ip_HN1_HDB03 ocf:heartbeat:IPaddr2 \
        op monitor interval="10s" timeout="20s" \
        params ip="10.23.0.27"

      sudo crm configure primitive rsc_nc_HN1_HDB03 azure-lb port=62503 \
        meta resource-stickiness=0

      sudo crm configure group g_ip_HN1_HDB03 rsc_ip_HN1_HDB03 rsc_nc_HN1_HDB03
      ```

   5. Cree las restricciones de clúster.  
      ```bash
      # Colocate the IP with HANA master
      sudo crm configure colocation col_saphana_ip_HN1_HDB03 4000: g_ip_HN1_HDB03:Started \
        msl_SAPHana_HN1_HDB03:Master  

      # Start HANA Topology before HANA  instance
      sudo crm configure order ord_SAPHana_HN1_HDB03 Optional: cln_SAPHanaTopology_HN1_HDB03 \
        msl_SAPHana_HN1_HDB03

      # HANA resources don't run on the majority maker node
      sudo crm configure location loc_SAPHanaCon_not_on_majority_maker msl_SAPHana_HN1_HDB03 -inf: hana-s-mm
      sudo crm configure location loc_SAPHanaTop_not_on_majority_maker cln_SAPHanaTopology_HN1_HDB03 -inf: hana-s-mm
      ```

6. **[1]** Configure las propiedades de clúster adicionales.   
    ```bash
    sudo crm configure rsc_defaults resource-stickiness=1000
    sudo crm configure rsc_defaults migration-threshold=50
    ```
7. **[1]** Compruebe la comunicación entre el enlace y el clúster.
    ```bash
    crm_attribute -G -n hana_hn1_glob_srHook
    # Expected result
    # crm_attribute -G -n hana_hn1_glob_srHook
    # scope=crm_config  name=hana_hn1_glob_srHook value=SOK
    ```

8. **[1]** Coloque el clúster fuera del modo de mantenimiento. Asegúrese de que el estado del clúster sea el correcto y de que se iniciaron todos los recursos.  
    ```bash
    # Cleanup any failed resources - the following command is example 
    crm resource cleanup rsc_SAPHana_HN1_HDB03

    # Place the cluster out of maintenance mode
    sudo crm configure property maintenance-mode=false
    ```
  
   > [!NOTE]
   > Los tiempos de espera de la configuración anterior son solo ejemplos y puede ser necesario adaptarlos a la configuración específica de HANA. Por ejemplo, puede que necesite aumentar el tiempo de espera de inicio si la base de datos de SAP HANA tarda más en iniciarse.
  

## <a name="test-sap-hana-failover"></a>Prueba de la conmutación por error de SAP HANA 

> [!NOTE]
> Este artículo contiene referencias a los términos *maestro* y *esclavo*, unos términos que Microsoft ya no usa. Cuando se eliminen estos términos del software, se eliminarán también de este artículo.

1. Antes de iniciar una prueba, compruebe el estado del clúster y de la replicación del sistema SAP HANA.    

   a. Compruebe que no haya acciones de clúster con errores.  
     ```bash
     #Verify that there are no failed cluster actions
     crm status
     # Example 
     #7 nodes configured
     #24 resource instances configured
     #
     #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #
     #Full list of resources:
     #
     # stonith-sbd    (stonith:external/sbd): Started hana-s-mm
     # Clone Set: cln_fs_HN1_HDB03_fscheck [fs_HN1_HDB03_fscheck]
     #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #     Stopped: [ hana-s-mm ]
     # Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #     Stopped: [ hana-s-mm ]
     # Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     #     Masters: [ hana-s1-db1 ]
     #     Slaves: [ hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #     Stopped: [ hana-s-mm ]
     # Resource Group: g_ip_HN1_HDB03
     #     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hana-s1-db1
     #     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hana-s1-db1
     ```

   b. Compruebe que la replicación del sistema SAP HANA esté sincronizada.

     ```bash
     # Verify HANA HSR is in sync
     sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
     #| Database | Host         | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary    | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
     #|          |              |       |              |           |         |           | Host         | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
     #| -------- | ------------ | ----- | ------------ | --------- | ------- | --------- | ------------ | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
     #| SYSTEMDB | hana-s1-db1  | 30301 | nameserver   |         1 |       1 | HANA_S1   | hana-s2-db1  |     30301 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
     #| HN1      | hana-s1-db1  | 30307 | xsengine     |         2 |       1 | HANA_S1   | hana-s2-db1  |     30307 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
     #| HN1      | hana-s1-db1  | 30303 | indexserver  |         3 |       1 | HANA_S1   | hana-s2-db1  |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
     #| HN1      | hana-s1-db3  | 30303 | indexserver  |         4 |       1 | HANA_S1   | hana-s2-db3  |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
     #| HN1      | hana-s1-db2  | 30303 | indexserver  |         5 |       1 | HANA_S1   | hana-s2-db2  |     30303 |         2 | HANA_S2   | YES           | SYNC        | ACTIVE      |                |
     #
     #status system replication site "1": ACTIVE
     #overall system replication status: ACTIVE
     #
     #Local System Replication State
     #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     #
     #mode: PRIMARY
     #site id: 1
     #site name: HANA_S1
     ```

2. Le recomendamos validar exhaustivamente la configuración del clúster de SAP HANA; para ello, realice pruebas documentadas en [alta disponibilidad para SAP HANA en las VM de Azure en SLES](./sap-hana-high-availability.md#test-the-cluster-setup) y en el [escenario de rendimiento optimizado para la escalabilidad horizontal de replicación de SLES](https://documentation.suse.com/sbp/all/html/SLES4SAP-hana-scaleOut-PerfOpt-12/index.html#_testing_the_cluster).

3. Compruebe la configuración del clúster para un escenario de error, cuando un nodo pierde el acceso al recurso compartido de NFS (`/hana/shared`).  

   Los agentes de recursos de SAP HANA dependen de los archivos binarios, almacenados en `/hana/shared` para realizar operaciones durante la conmutación por error. El sistema de archivos `/hana/shared` se monta en NFS en la configuración presentada. Una prueba que puede realizar, es crear una regla de firewall temporal para bloquear el acceso al volumen de ANF `/hana/shared` en una de las VM del sitio principal. Este enfoque valida que el clúster va a poder realizar la conmutación por error, si se pierde el acceso a `/hana/shared` en el sitio de replicación del sistema activo.  

   **Resultado esperado**: al bloquear el acceso al volumen de ANF `/hana/shared` en una de las VM del sitio principal, se producirá un error en la operación de supervisión que realiza la operación de lectura y escritura en el sistema de archivos, ya que no puede acceder al sistema de archivos y desencadenará la conmutación por error de recursos de HANA. Se espera el mismo resultado cuando el nodo de HANA pierde el acceso al recurso compartido de NFS.  
     
   Puede comprobar el estado de los recursos del clúster ejecutando `crm_mon` o `crm status`. Estado del recurso antes de iniciar la prueba:
     ```bash
     # Output of crm_mon
     #7 nodes configured
     #24 resource instances configured
     #
     #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #
     #Active resources:
     #
     #stonith-sbd     (stonith:external/sbd): Started hana-s-mm
     # Clone Set: cln_fs_HN1_HDB03_fscheck [fs_HN1_HDB03_fscheck]
     #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     # Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     # Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     #     Masters: [ hana-s1-db1 ]
     #     Slaves: [ hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     # Resource Group: g_ip_HN1_HDB03
     #     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hana-s2-db1
     #     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hana-s2-db1     
      ```

   Para simular un error para `/hana/shared`, confirme primero la dirección IP del volumen de ANF `/hana/shared` en el sitio principal. Para hacerlo, ejecute `df -kh|grep /hana/shared`. 
   A continuación, configure una regla de firewall temporal para bloquear el acceso a la dirección IP del volumen de ANF `/hana/shared` mediante la ejecución del siguiente comando en una de las VM del sitio de replicación del sistema principal de HANA.
   En este ejemplo, el comando se ejecutó en hana-s1-db1.

     ```bash
     iptables -A INPUT -s 10.23.1.7 -j DROP; iptables -A OUTPUT -d 10.23.1.7 -j DROP
     ```

   Los recursos del clúster se migran a otro sitio de replicación del sistema HANA.    
              
   Si establece AUTOMATED_REGISTER = "false", tendrá que configurar la replicación del sistema SAP HANA en el sitio secundario. En este caso, puede ejecutar estos comandos para volver a configurar SAP HANA como secundario.   

     ```bash
     # Execute on the secondary 
     su - hn1adm
     # Make sure HANA is not running on the secondary site. If it is started, stop HANA
     sapcontrol -nr 03 -function StopWait 600 10
     # Register the HANA secondary site
     hdbnsutil -sr_register --name=HANA_S1 --remoteHost=hana-s2-db1 --remoteInstance=03 --replicationMode=sync
     # Switch back to root and cleanup failed resources
     crm resource cleanup SAPHana_HN1_HDB03
     ```

   El estado de los recursos, después de la prueba: 

     ```bash
     # Output of crm_mon
     #7 nodes configured
     #24 resource instances configured
     #
     #Online: [ hana-s-mm hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     #
     #Active resources:
     #
     #stonith-sbd     (stonith:external/sbd): Started hana-s-mm
     # Clone Set: cln_fs_HN1_HDB03_fscheck [fs_HN1_HDB03_fscheck]
     #     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     # Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     Started: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db1 hana-s2-db2 hana-s2-db3 ]
     # Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     #     Masters: [ hana-s2-db1 ]
     #     Slaves: [ hana-s1-db1 hana-s1-db2 hana-s1-db3 hana-s2-db2 hana-s2-db3 ]
     # Resource Group: g_ip_HN1_HDB03
     #     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hana-s2-db1
     #     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hana-s2-db1
     ```


## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* [Volúmenes NFS v4.1 en Azure NetApp Files para SAP HANA](./hana-vm-operations-netapp.md)
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en VM de Azure, consulte [Alta disponibilidad de SAP HANA en Azure Virtual Machines (VM)][sap-hana-ha].