---
title: 'Replicación de datos a través de ExpressRoute con Azure Migrate: Server Migration'
description: 'Use Azure ExpressRoute para la replicación con Azure Migrate: Server Migration.'
author: DeSeelam
ms.author: deseelam
ms.manager: bsiva
ms.topic: how-to
ms.date: 02/22/2021
ms.openlocfilehash: 3a6afa0fadf5a84ad938b0b0cec321c0e17adeff
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108317530"
---
# <a name="replicate-data-over-expressroute-with-azure-migrate-server-migration"></a>Replicación de datos a través de ExpressRoute con Azure Migrate: Server Migration

En este artículo aprenderá a configurar la herramienta [Azure Migrate: Server Migration](./migrate-services-overview.md#azure-migrate-server-migration-tool) para replicar datos a través de un circuito ExpressRoute al migrar servidores a Azure.

## <a name="understand-azure-expressroute-circuits"></a>Descripción de los circuitos ExpressRoute

Un circuito ExpressRoute conecta la infraestructura local a Microsoft a través de un proveedor de conectividad. Los circuitos ExpressRoute se pueden configurar para usar el emparejamiento privado, el emparejamiento de Microsoft o ambos. Para más información sobre las opciones de emparejamiento con ExpressRoute, consulte [Circuitos ExpressRoute y emparejamiento](../expressroute/expressroute-circuit-peerings.md#peeringcompare).

La herramienta Azure Migrate: Server Migration le ayuda a migrar servidores locales y servidores de otras nubes a Azure Virtual Machines. La herramienta configura un flujo de replicación continuo para replicar los datos de los servidores que se van a migrar a discos administrados en la suscripción a Azure. Cuando esté listo para migrar los servidores, se usarán los datos replicados en Azure para ello.

Los datos replicados desde los servidores locales se pueden configurar para enviarlos a su suscripción a Azure a través de Internet o una conexión ExpressRoute. Los datos enviados a través de Internet usan una conexión cifrada segura. Si tiene un gran número de servidores para migrar, el uso de ExpressRoute para la replicación puede ayudarle a completar el proceso de forma más eficaz al aprovechar el ancho de banda aprovisionado disponible con el circuito ExpressRoute.

En este artículo, aprenderá a replicar datos mediante:
> [!div class="checklist"]
>
> * Un circuito ExpressRoute con emparejamiento privado.
> * Un circuito ExpressRoute con emparejamiento de Microsoft.

## <a name="replicate-data-by-using-an-expressroute-circuit-with-private-peering"></a>Replicación de datos mediante un circuito ExpressRoute con emparejamiento privado

> [!Note]
> En este artículo se muestra cómo replicar a través de un circuito de emparejamiento privado para la [migración sin agente de máquinas virtuales de VMware a Azure](./tutorial-migrate-vmware.md). Para usar la compatibilidad de puntos de conexión privados con [otros métodos de replicación](./migrate-services-overview.md#azure-migrate-server-migration-tool), consulte [Uso de Azure Migrate con puntos de conexión privados](./how-to-use-azure-migrate-with-private-endpoints.md).
 
En el método de migración sin agente de máquinas virtuales de VMware a Azure, el dispositivo de Azure Migrate primero carga los datos de replicación en una cuenta de almacenamiento (cuenta de almacenamiento en caché) de la suscripción. Después, el servicio Azure Migrate mueve los datos replicados de la cuenta de almacenamiento en caché a los discos administrados por la réplica de la suscripción.

Si quiere usar un circuito de emparejamiento privado para la replicación, deberá crear y asociar un punto de conexión privado a la cuenta de almacenamiento en caché. Los puntos de conexión privados usan una o varias direcciones IP privadas de la red virtual, lo cual incorpora la cuenta de almacenamiento de manera eficaz a la red virtual de Azure. El punto de conexión privado permite que el dispositivo de Azure Migrate se conecte con la cuenta de almacenamiento en caché mediante el emparejamiento privado de ExpressRoute. A continuación, los datos se pueden transferir directamente en la dirección IP privada. <br/>

![Captura de pantalla que muestra el proceso de replicación.](./media/replicate-using-expressroute/replication-process.png)

> [!Important]
> - Además de los datos de replicación, el dispositivo de Azure Migrate se comunica con el servicio de Azure Migrate para obtener las actividades de plano de control. Estas actividades incluyen la orquestación de la replicación. La comunicación del plano de control entre el dispositivo de Azure Migrate y el servicio de Azure Migrate sigue realizándose a través de Internet en el punto de conexión público del servicio de Azure Migrate.
> - El punto de conexión privado de la cuenta de almacenamiento debe ser accesible desde la red en la que está implementado el dispositivo de Azure Migrate.
> - El DNS debe configurarse para resolver las consultas de DNS que realiza el dispositivo de Azure Migrate para el punto de conexión de servicio del blob de la cuenta de almacenamiento en caché a la dirección IP privada del punto de conexión privado que está asociado a la cuenta de almacenamiento en caché.
> - La cuenta de almacenamiento en caché debe ser accesible en el punto de conexión público. Azure Migrate usa el punto de conexión público de la cuenta de almacenamiento en caché para trasladar los datos de la cuenta de almacenamiento a los discos administrados de réplica.

### <a name="prerequisites"></a>Requisitos previos

Necesita los permisos siguientes en el grupo de recursos y la red virtual donde se creará el punto de conexión privado.

Caso de uso | Permisos
--- | --- 
 Creación y administración de puntos de conexión privados. | Microsoft.Network/privateEndpoint/write/action<br/>Microsoft.Network/privateEndpoint/read/action 
|Adjunte un punto de conexión privado a una red virtual o subred.<br/>Este permiso es necesario en la red virtual en la que se creará el punto de conexión privado.| Microsoft.Network/virtualNetworks/subnet/join/action <br/> Microsoft.Network/virtualNetworks/join/action
|Vinculación del punto de conexión privado a una cuenta de almacenamiento. <br/>| Microsoft.Microsoft.Storage/storageAccounts/privateEndpointConnectionApproval/action <br/> Microsoft.Microsoft.Storage/storageAccounts/privateEndpointConnections/read
|Creación de una interfaz de red y unión a un grupo de seguridad de red. | Microsoft.Network/networkInterfaces/read <br/> Microsoft.Network/networkInterfaces/subnets/write <br/> Microsoft.Network/networkInterfaces/subnets/read<br/> Microsoft.Network/networkSecurityGroups/join/action (opcional)
Creación y administración de zonas DNS privadas.| Rol de colaborador en la zona DNS privada <br/> _O_ <br/> Microsoft.Network/privateDnsZones/A/* <br/> Microsoft.Network/privateDnsZones/write Microsoft.Network/privateDnsZones/read <br/> Microsoft.Network/privateEndpoints/privateDnsZoneGroups/write <br/> Microsoft.Network/privateEndpoints/privateDnsZoneGroups/read <br/> Microsoft.Network/privateDnsZones/virtualNetworkLinks/write <br/> Microsoft.Network/privateDnsZones/virtualNetworkLinks/read <br/> Microsoft.Network/virtualNetworks/join/action 

### <a name="identify-the-cache-storage-account"></a>Identificación de la cuenta de almacenamiento en caché

 Azure Migrate crea automáticamente una cuenta de almacenamiento en caché cuando la replicación se configura (con la experiencia de Azure Portal) por primera vez para una máquina virtual en un proyecto de Azure Migrate. La cuenta de almacenamiento se crea en la misma suscripción y grupo de recursos en el que creó el proyecto de Azure Migrate.

Para crear y ubicar la cuenta de almacenamiento, haga lo siguiente:

1. Use Azure Portal con el fin de replicar una o varias máquinas virtuales en el proyecto de Azure Migrate.
1. Vaya hasta el grupo de recursos del proyecto de Azure Migrate.
1. Ubique la cuenta de almacenamiento en caché; para ello, identifique el prefijo **lsa** en el nombre de la cuenta de almacenamiento.

   ![Captura de pantalla que muestra una vista del grupo de recursos.](./media/replicate-using-expressroute/storage-account-name.png)

> [!Tip]
> Si tiene más de una cuenta de almacenamiento con el prefijo **lsa** en el grupo de recursos, puede comprobar la cuenta de almacenamiento. Para ello, vaya al menú Configuración de la replicación y Configuración de destino de cualquiera de las máquinas virtuales de replicación del proyecto.
>
> ![Captura de pantalla que muestra información general sobre la configuración de replicación.](./media/replicate-using-expressroute/storage-account.png)

### <a name="upgrade-the-cache-storage-account-to-general-purpose-v2"></a>Actualización de la cuenta de almacenamiento de caché a la versión 2 de uso general

Solo se pueden crear puntos de conexión privados en una cuenta de almacenamiento de uso general v2. Si la cuenta de almacenamiento en caché no es una cuenta de almacenamiento de uso general v2, actualícela.

1. Vaya a la cuenta de almacenamiento.
1. Seleccione **Configuración**.
1. En **Tipo de cuenta**, seleccione **Actualizar**.
1. En **Confirmar actualización**, escriba el nombre de la cuenta.
1. Seleccione **Actualizar** en la parte inferior de la página.

   ![Captura de pantalla que muestra cómo se actualiza una cuenta de almacenamiento.](./media/replicate-using-expressroute/upgrade-storage-account.png)

### <a name="create-a-private-endpoint-for-the-storage-account"></a>Creación de un punto de conexión privado para la cuenta de almacenamiento

1. Vaya a la cuenta de almacenamiento, seleccione **Redes** en el menú de la izquierda y seleccione la pestaña **Conexiones de punto de conexión privado**.
1. Seleccione **+ Punto de conexión privado**.

    1. En la ventana **Crear un punto de conexión privado**, seleccione la **suscripción** y el **grupo de recursos**. Especifique un nombre para el punto de conexión privado y seleccione la región de la cuenta de almacenamiento.
     
       ![Captura de pantalla que muestra la ventana de configuración de un punto de conexión privado.](./media/replicate-using-expressroute/storage-account-private-endpoint-creation.png)

    1. En la pestaña **Recurso**, especifique el **Nombre de la suscripción** en la que se encuentra la cuenta de almacenamiento. Elija **Microsoft.Storage/storageAccounts** como **Tipo de recurso**. En **Recurso**, escriba el nombre de la cuenta de almacenamiento de replicación de uso general v2. Seleccione **Blob** como **Subrecurso de destino**.
     
       ![Captura de pantalla que muestra la configuración del punto de conexión privado de la cuenta de almacenamiento.](./media/replicate-using-expressroute/storage-account-private-endpoint-settings.png)

    1. En la pestaña **Configuración**, seleccione la **Red virtual** y la **Subred** para el punto de conexión privado de la cuenta de almacenamiento.

       > [!Note]
       > La red virtual debe contener el punto de conexión de la puerta de enlace de ExpressRoute, o debe estar conectado a la red virtual con la puerta de enlace de ExpressRoute.

       En la sección **Integración de DNS privado**, seleccione **Sí** e intégrese con una zona DNS privada. Si selecciona **Sí**, la zona DNS se vincula automáticamente a la red virtual seleccionada. También agrega los registros DNS necesarios para la resolución DNS de nuevas direcciones IP y nombres de dominio completos creados para el punto de conexión privado. Más información sobre las [zonas DNS privadas.](../dns/private-dns-overview.md)

       ![Captura de pantalla que muestra zonas DNS privadas.](./media/replicate-using-expressroute/private-dns-zone.png)

    1. También puede agregar **Etiquetas** para el punto de conexión privado.

    1. Cuando haya terminado de especificar los detalles, seleccione la pestaña **Revisar y crear**. Una vez completada la validación, seleccione **Crear** para crear el punto de conexión privado.

> [!Note]
> Si el usuario que creó el punto de conexión privado también es el propietario de la cuenta de almacenamiento, el punto de conexión privado se aprobará automáticamente. De lo contrario, el propietario deberá aprobar el punto de conexión privado para su uso.

#### <a name="create-private-dns-zones-and-add-dns-records-manually-optional"></a>Creación de zonas DNS privadas y adición de registros DNS manualmente (opcional)

Si no seleccionó la opción de integración con una zona DNS privada en el momento en que creó el punto de conexión privado, debe crear manualmente una zona DNS privada.

> [!Note]
> Si seleccionó **Sí** para la integración con una zona DNS privada, puede omitir esta sección.

Para crear manualmente una zona DNS privada:

1. Seleccione **Zonas DNS privadas**.

    ![Captura de pantalla que muestra la creación de una zona DNS privada.](./media/replicate-using-expressroute/create-private-dns.png)

    1. En la página **Zonas DNS privadas**, seleccione **+ Agregar** para agregar una zona nueva.
    1. En la página **Crear zona DNS privada**, rellene los datos necesarios. Escriba el nombre de la zona DNS privada como **_privatelink_.blob.core.windows.net**.
    1. En la pestaña **Revisar y crear**, revise y cree la zona DNS.

1. Vincule la zona DNS privada a su red virtual.

    La zona DNS privada que creó anteriormente debe estar vinculada a la red virtual a la que está asociado el punto de conexión privado.

    1. Vaya a la zona DNS privada que creó en el paso anterior y diríjase a los vínculos de red virtual en la parte de la izquierda de la página. Seleccione **+Agregar**.
    1. Ingrese todos los detalles obligatorios. Los campos **Suscripción** y **Red virtual** deben rellenarse con los datos correspondientes de la red virtual en la que se asoció el punto de conexión privado. Los demás campos pueden dejarse tal cual.

1. El siguiente paso consiste en agregar registros DNS a la zona DNS. Agregue una entrada para el nombre de dominio completo de la cuenta de almacenamiento en la zona DNS privada.

    1. Vaya a la zona DNS privada y navegue hasta la sección **Información general** a la izquierda de la página. Seleccione **+ Registro** para empezar a agregar registros.
    1. En la página **Agregar conjunto de registros**, agregue una entrada para el nombre de dominio completo y la dirección IP privada como un registro de tipo D.

> [!Important]
> Puede requerir una configuración adicional de DNS para resolver la dirección IP privada del punto de conexión privado de la cuenta de almacenamiento desde el entorno de origen. Para comprender la configuración de DNS necesaria, consulte [Configuración de DNS para puntos de conexión privados de Azure](../private-link/private-endpoint-dns.md#on-premises-workloads-using-a-dns-forwarder).

## <a name="replicate-data-by-using-an-expressroute-circuit-with-microsoft-peering"></a>Replicación de datos mediante un circuito ExpressRoute con emparejamiento de Microsoft

Puede usar el emparejamiento de Microsoft o un dominio de emparejamiento público existente (en desuso para las nuevas conexiones de ExpressRoute) para enrutar el tráfico de replicación a través de un circuito ExpressRoute.

![Diagrama que muestra la replicación con el emparejamiento de Microsoft.](./media/replicate-using-expressroute/replication-with-microsoft-peering.png)

Incluso con los datos de replicación que cruzan el circuito emparejado de Microsoft, la conectividad a Internet desde el sitio local sigue siendo necesaria para llevar a cabo otras comunicaciones (plano de control) con Azure Migrate. Hay algunas direcciones URL a las que no se puede acceder a través de ExpressRoute. El dispositivo de replicación o el host de Hyper-V necesitan acceso a las direcciones URL para orquestar el proceso de replicación. Revise los requisitos de direcciones URL en función del escenario de migración, ya sean [migraciones sin agente de VMware](./migrate-appliance.md#public-cloud-urls) o [migraciones basadas en agentes](./migrate-replication-appliance.md).

Si usa un proxy en el sitio local y quiere usar ExpressRoute para el tráfico de replicación, configure una omisión de proxy para las direcciones URL correspondientes en el dispositivo local.

### <a name="configure-proxy-bypass-rules-on-the-azure-migrate-appliance-for-vmware-agentless-migrations"></a>Configuración de reglas de omisión de proxy en el dispositivo de Azure Migrate (para migraciones sin agente de VMware)

1. Inicie sesión mediante Escritorio remoto en el dispositivo Azure Migrate.
1. Abra el archivo *C:/ProgramData/MicrosoftAzure/Config/appliance.json* con el bloc de notas.
1. En el archivo, cambie la línea que dice `"EnableProxyBypassList": "false"` para que diga `"EnableProxyBypassList": "true"`. Guarde los cambios y reinicie el dispositivo.
1. Después de reiniciar, cuando abra el administrador de configuración del dispositivo,verá la opción de omisión de proxy en la interfaz de usuario de la aplicación web. Agregue las siguientes direcciones URL a la lista de omisión de proxy:

    - .*.vault.azure.net
    - .*.servicebus.windows.net
    - .*.discoverysrv.windowsazure.com
    - .*.migration.windowsazure.com
    - .*.hypervrecoverymanager.windowsazure.com
    - .*.blob.core.windows.net

### <a name="configure-proxy-bypass-rules-on-the-replication-appliance-for-agent-based-migrations"></a>Configuración de reglas de omisión de proxy en el dispositivo de replicación (para migraciones basadas en agentes)

Para configurar la lista de omisión de proxy en el servidor de configuración y los servidores de procesos:

1. Descargue la [herramienta PsExec](/sysinternals/downloads/psexec) para acceder al contexto de usuario del sistema.
1. Abra Internet Explorer en el contexto de usuario del sistema ejecutando la línea de comandos siguiente: `psexec -s -i "%programfiles%\Internet Explorer\iexplore.exe"`.
1. Agregue la configuración de proxy en Internet Explorer.
1. En la lista de omisión, agregue la dirección URL de Azure Storage: *.blob.core.windows.net.

Las reglas de omisión anteriores garantizarán que el tráfico de replicación puede fluir a través de ExpressRoute, mientras que la comunicación de administración puede cruzar el proxy de Internet.

Además, debe anunciar las rutas en el filtro de ruta para las siguientes comunidades de BGP para que el tráfico de replicación de Azure Migrate pase por un circuito ExpressRoute en lugar de Internet:

- Comunidad de BGP regional para la región de Azure de origen (región del proyecto de Azure Migrate)
- Comunidad de BGP regional para la región de Azure de destino (región para la migración)
- Comunidad de BGP para Azure Active Directory (12076:5060)

Obtenga más información sobre los [filtros de ruta](../expressroute/how-to-routefilter-portal.md) y la lista de [comunidades de BGP para ExpressRoute](../expressroute/expressroute-routing.md#bgp).

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener más información sobre:

- [Circuitos ExpressRoute](../expressroute/expressroute-circuit-peerings.md)
- [Dominios de enrutamiento de ExpressRoute](../expressroute/expressroute-circuit-peerings.md#peeringcompare)
- [Puntos de conexión privados](../private-link/private-endpoint-overview.md)