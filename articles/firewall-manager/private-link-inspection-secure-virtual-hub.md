---
title: Protección del tráfico destinado a puntos de conexión privados en Azure Virtual WAN
description: Obtenga información sobre cómo usar reglas de red y reglas de aplicación para proteger el tráfico destinado a puntos de conexión privados en Azure Virtual WAN
services: firewall-manager
author: jocortems
ms.service: firewall-manager
ms.topic: how-to
ms.date: 04/02/2021
ms.author: jocorte
ms.openlocfilehash: 7322bab635d398fc7a5335546ba6fef327ff24b2
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259360"
---
# <a name="secure-traffic-destined-to-private-endpoints-in-azure-virtual-wan"></a>Protección del tráfico destinado a puntos de conexión privados en Azure Virtual WAN

> [!NOTE]
> Este artículo solo se aplica a los centros virtuales protegidos. Si desea inspeccionar el tráfico destinado a puntos de conexión privados mediante Azure Firewall en una red virtual de centro, consulte [Uso de Azure Firewall para inspeccionar el tráfico destinado a un punto de conexión privado](../private-link/inspect-traffic-with-azure-firewall.md).

Un [punto de conexión privado de Azure](../private-link/private-endpoint-overview.md) es el bloque de creación fundamental para [Azure Private Link](../private-link/private-link-overview.md). Los puntos de conexión privados permiten que los recursos de Azure implementados en una red virtual se comuniquen de forma privada con recursos de Private Link.

Los puntos de conexión privados permiten a los recursos el acceso al servicio Private Link implementado en una red virtual. El acceso al punto de conexión privado a través del emparejamiento de red virtual y las conexiones de red locales amplía la conectividad.

Es posible que necesite filtrar el tráfico de los clientes de forma local o en Azure destinado a los servicios expuestos mediante puntos de conexión privados en una red virtual conectada a WAN Virtual. Este artículo le guiará a través de esta tarea mediante el uso de un [centro virtual protegido](../firewall-manager/secured-virtual-hub.md) con [Azure Firewall](../firewall/overview.md) como proveedor de seguridad.

Azure Firewall filtra el tráfico con cualquiera de los métodos siguientes:

* [FQDN en las reglas de red](../firewall/fqdn-filtering-network-rules.md) para los protocolos TCP y UDP
* [FQDN en las reglas de aplicación](../firewall/features.md#application-fqdn-filtering-rules) para HTTP, HTTPS y MSSQL.
* Direcciones IP de origen y destino, puerto y protocolo mediante [reglas de red](../firewall/features.md#network-traffic-filtering-rules)

Use reglas de aplicación sobre reglas de red para inspeccionar el tráfico destinado a puntos de conexión privados.
Microsoft administra el centro virtual protegido y no se puede vincular a una [zona DNS privada](../dns/private-dns-privatednszone.md). Es necesario para resolver un FQDN de [recurso de vínculo privado](../private-link/private-endpoint-overview.md#private-link-resource) en su dirección IP de punto de conexión privado correspondiente.

El filtrado por nombre de dominio completo de SQL se admite solo en [modo de proxy](../azure-sql/database/connectivity-architecture.md#connection-policy) (puerto 1433). El modo de *proxy* puede traducirse en una mayor latencia en comparación con el modo de *redirección*. Si quiere seguir usando el modo de redirección, que es el valor predeterminado para los clientes que se conectan desde dentro de Azure, puede filtrar el acceso mediante FQDN en reglas de red del firewall.

## <a name="filter-traffic-using-fqdn-in-network-and-application-rules"></a>Filtrado del tráfico mediante FQDN en reglas de red y de aplicación

En los pasos siguientes se habilita Azure Firewall para filtrar el tráfico mediante FQDN en reglas de red y de aplicación:

1. Implemente una máquina virtual de [reenviador DNS](../private-link/private-endpoint-dns.md#virtual-network-and-on-premises-workloads-using-a-dns-forwarder) en una red virtual conectada al centro virtual protegido y vinculada a las zonas DNS privadas que hospedan los tipos de registro A para los puntos de conexión privados.

2. Configure los [valores de DNS personalizados](../firewall/dns-settings.md#configure-custom-dns-servers---azure-portal) para que apunten a la dirección IP de la máquina virtual del reenviador de DNS y habilite el proxy DNS en la directiva de firewall asociada a la instancia de Azure Firewall implementada en el centro virtual protegido.

3. Configure los [servidores DNS personalizados](../virtual-network/manage-virtual-network.md#change-dns-servers) para las redes virtuales conectadas al centro virtual protegido para que apunten a la dirección IP privada asociada con la instancia de Azure Firewall implementada en el centro virtual protegido.

4. Configure los servidores DNS locales para que reenvíen las consultas DNS para las zonas DNS públicas de los puntos de conexión privados a la dirección IP privada asociada a la instancia de Azure Firewall implementada en el centro virtual protegido.

5. Configure una [regla de aplicación](../firewall/tutorial-firewall-deploy-portal.md#configure-an-application-rule) o una [regla de red](../firewall/tutorial-firewall-deploy-portal.md#configure-a-network-rule) según sea necesario en la directiva de firewall asociada a la instancia de Azure Firewall implementada en el centro virtual protegido con el FQDN de *Tipo de destino* y el FQDN público de recurso de vínculo privado como *Destino*.

6. Navegue a los *centros virtuales protegidos* en la directiva de firewall asociada a la instancia de Azure Firewall implementada en el centro virtual protegido y seleccione el centro virtual protegido en el que se configurará el filtrado de tráfico destinado a los puntos de conexión privados.

7. Vaya a **Configuración de seguridad** y seleccione **Send via Azure Firewall** (Enviar a través de Azure Firewall) en **Tráfico privado**.

8. Seleccione **Private traffic prefixes** (Prefijos de tráfico privado) para editar los prefijos CIDR que se inspeccionarán a través de Azure Firewall en el centro virtual protegido y agregue el prefijo /32 para cada punto de conexión privado, como se indica a continuación:

   > [!IMPORTANT]
   > Si no se configuran los prefijos /32, el tráfico destinado a los puntos de conexión privados omitirá Azure Firewall.

   :::image type="content" source="./media/private-link-inspection-secure-virtual-hub/firewall-manager-security-configuration.png" alt-text="Configuración de seguridad de Firewall Manager" border="true":::

Estos pasos solo funcionan cuando los clientes y los puntos de conexión privados se implementan en diferentes redes virtuales conectadas al mismo centro virtual protegido y para clientes locales. Si los clientes y los puntos de conexión privados se implementan en la misma red virtual, se debe crear un UDR con rutas /32 para los puntos de conexión privados. Configure estas rutas con la opción **Tipo del próximo salto** establecida en **Dispositivo virtual** y la opción **Dirección del próximo salto** establecida en la dirección IP privada de la instancia de Azure Firewall implementada en el centro virtual protegido. **Propagate gateway routes** (Propagar rutas de puerta de enlace) debe establecerse en **Sí**.

En el siguiente diagrama se ilustran los flujos de tráfico de datos y DNS para que los distintos clientes se conecten a un punto de conexión privado implementado en Azure Virtual WAN:

:::image type="content" source="./media/private-link-inspection-secure-virtual-hub/private-link-inspection-virtual-wan-architecture.png" alt-text="Flujos de tráfico" border="true":::

## <a name="troubleshooting"></a>Solución de problemas

Los principales problemas que podría tener al intentar filtrar el tráfico destinado a puntos de conexión privados a través de un centro virtual protegido son:

- Los clientes no pueden conectarse a los puntos de conexión privados.

- Se omite Azure Firewall. Este síntoma puede validarse por la ausencia de entradas de registro de reglas de red o de aplicación en Azure Firewall.

En la mayoría de los casos, estos problemas se deben a una de las siguientes incidencias:

- Resolución de nombres DNS incorrecta

- Configuración de enrutamiento incorrecta

### <a name="incorrect-dns-name-resolution"></a>Resolución de nombres DNS incorrecta

1. Compruebe que los servidores DNS de la red virtual están establecidos en *Personalizado* y la dirección IP es la dirección IP privada de Azure Firewall en el centro virtual protegido.

   CLI de Azure:

   ```azurecli-interactive
   az network vnet show --name <VNET Name> --resource-group <Resource Group Name> --query "dhcpOptions.dnsServers"
   ```
2. Compruebe que los clientes de la misma red virtual que la máquina virtual del reenviador DNS puedan resolver el FQDN público del punto de conexión privado en su dirección IP privada correspondiente; para ello, consulte directamente la máquina virtual configurada como reenviador de DNS.

   Linux:

   ```bash
   dig @<DNS forwarder VM IP address> <Private endpoint public FQDN>
   ```
3. Inspeccione las entradas del registro *AzureFirewallDNSProxy* de Azure Firewall y valide que puede recibir y resolver consultas DNS de los clientes.

   ```kusto
   AzureDiagnostics
   | where Category contains "DNS"
   | where msg_s contains "database.windows.net"
   ```
4. Compruebe que se ha habilitado *Proxy DNS* y que se ha configurado un servidor DNS *Personalizado* que apunta a la dirección IP de la máquina virtual del reenviador DNS en la directiva de firewall asociada a la instancia de Azure Firewall en el centro virtual protegido.

   CLI de Azure:

   ```azurecli-interactive
   az network firewall policy show --name <Firewall Policy> --resource-group <Resource Group Name> --query dnsSettings
   ```

### <a name="incorrect-routing-configuration"></a>Configuración de enrutamiento incorrecta

1. Compruebe la *Configuración de seguridad* en la directiva de firewall asociada a la instancia de Azure Firewall implementada en el centro virtual protegido. Asegúrese de que en la columna **PRIVATE TRAFFIC** (TRÁFICO PRIVADO) aparezca como **Secured by Azure Firewall** (Protegida por Azure Firewall) para todas las conexiones de red virtual y ramas para las que quiere filtrar el tráfico.

   :::image type="content" source="./media/private-link-inspection-secure-virtual-hub/firewall-policy-private-traffic-configuration.png" alt-text="Tráfico privado protegido por Azure Firewall" border="true":::

2. Compruebe la **Configuración de seguridad** en la directiva de firewall asociada a la instancia de Azure Firewall implementada en el centro virtual protegido. Asegúrese de que hay una entrada /32 para cada dirección IP privada de punto de conexión privado para la que quiere filtrar el tráfico en **Private traffic prefixes** (Prefijos de tráfico privado).

   :::image type="content" source="./media/private-link-inspection-secure-virtual-hub/firewall-manager-security-configuration.png" alt-text="Configuración de seguridad de Firewall Manager: prefijos de tráfico privado" border="true":::

3. En el centro virtual protegido en Virtual WAN, inspeccione las rutas efectivas de las tablas de rutas asociadas a las redes virtuales y las conexiones de ramas para las que desea filtrar el tráfico. Asegúrese de que hay entradas /32 para cada dirección IP privada de punto de conexión privado para la que quiere filtrar el tráfico.

   :::image type="content" source="./media/private-link-inspection-secure-virtual-hub/secured-virtual-hub-effective-routes.png" alt-text="Rutas efectivas del centro virtual protegido" border="true":::

4. Inspeccione las rutas efectivas en las NIC conectadas a las máquinas virtuales implementadas en las redes virtuales en las que quiere filtrar el tráfico. Asegúrese de que hay entradas /32 para cada dirección IP privada de punto de conexión privado para la que quiere filtrar el tráfico.
 
   CLI de Azure:

   ```azurecli-interactive
   az network nic show-effective-route-table --name <Network Interface Name> --resource-group <Resource Group Name> -o table
   ```
5. Inspeccione las tablas de enrutamiento de los dispositivos de enrutamiento locales. Asegúrese de que está obteniendo los espacios de direcciones de las redes virtuales donde están implementados los puntos de conexión privados.

   Azure Virtual WAN no anuncia los prefijos configurados en **Private traffic prefixes** (Prefijos de tráfico privado) en la **Configuración de seguridad** de la directiva de firewall en local. Se espera que las entradas /32 no se muestren en las tablas de enrutamiento de los dispositivos de enrutamiento locales.

6. Inspeccione los registros **AzureFirewallApplicationRule** y **AzureFirewallNetworkRule** de Azure Firewall. Asegúrese de que se está registrando el tráfico destinado a los puntos de conexión privados.

   Las entradas del registro **AzureFirewallNetworkRule** no incluyen información de FQDN. Filtre por dirección IP y puerto al inspeccionar las reglas de red.

   Al filtrar el tráfico destinado a puntos de conexión privados de [Azure Files](../storage/files/storage-files-introduction.md), solo se generarán entradas del registro **AzureFirewallNetworkRule** cuando un cliente monte o se conecte al recurso compartido de archivos por primera vez. Azure Firewall no generará registros para las operaciones [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) para archivos en el recurso compartido de archivos. Esto se debe a que las operaciones CRUD se transfieren por el canal TCP persistente abierto cuando el cliente se conecta o monta el recurso compartido de archivos por primera vez.

   Ejemplo de consulta de registro de reglas de aplicación:

   ```kusto
   AzureDiagnostics
   | where msg_s contains "database.windows.net"
   | where Category contains "ApplicationRule"
   ```
## <a name="next-steps"></a>Pasos siguientes

- [Uso de Azure Firewall para inspeccionar el tráfico destinado a un punto de conexión privado](../private-link/inspect-traffic-with-azure-firewall.md)
