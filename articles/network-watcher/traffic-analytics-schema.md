---
title: Esquema de Análisis de tráfico de Azure | Microsoft Docs
description: Comprender el esquema de Análisis de tráfico para analizar los registros de flujo de grupo de seguridad de red de Azure.
services: network-watcher
documentationcenter: na
author: vinynigam
manager: agummadi
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: vinigam
ms.openlocfilehash: 5bc493197b6ae4e6bd969a837bb873cae38c0790
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112032022"
---
# <a name="schema-and-data-aggregation-in-traffic-analytics"></a>Esquema y agregación de datos en Análisis de tráfico

Análisis de tráfico es una solución basada en la nube, que proporciona visibilidad de la actividad de usuarios y aplicaciones en las redes en la nube. Análisis de tráfico permite analizar los registros de flujo del grupo de seguridad de Network Watcher para proporcionar información sobre el flujo de tráfico en la nube de Azure. Con Análisis de tráfico, puede:

- Visualizar la actividad de la red en las suscripciones de Azure e identificar las zonas activas.
- Identificar las amenazas de seguridad y proteger la red, con información sobre puertos abiertos, aplicaciones que intentan acceder a Internet y máquinas virtuales (VM) que se conectan a redes no autorizadas.
- Entender los patrones de flujo de tráfico entre regiones de Azure e Internet para optimizar el rendimiento y la capacidad de su implementación de red.
- Identificar errores de configuración de red que dan lugar a errores de conexión.
- Conocer el uso de red en bytes, paquetes o flujos.

### <a name="data-aggregation"></a>Agregación de datos

1. Todos los registros de flujo en un NSG entre "FlowIntervalStartTime_t" y "FlowIntervalEndTime_t" se capturan en intervalos de un minuto en la cuenta de almacenamiento como blobs antes de que el Análisis de tráfico los procese.
2. El intervalo de procesamiento predeterminado de Análisis de tráfico es 60 minutos. Esto significa que cada 60 minutos, Análisis de tráfico elige blobs del almacenamiento para la agregación. Si el intervalo de procesamiento elegido es de 10 minutos, Análisis de tráfico seleccionará los blobs de la cuenta de almacenamiento después de cada 10 minutos.
3. El Análisis de tráfico agrupa en un único flujo los flujos que tengan la misma IP de origen, IP de destino, puerto de destino, nombre NSG, regla NSG, dirección de flujo y protocolo de capa de transporte (TCP o UDP) (Nota: el puerto de origen se excluye para la agregación).
4. El Análisis de tráfico rellena este registro único (más detalles en la sección siguiente) y lo ingesta en Log Analytics. Este proceso puede tardar hasta una hora.
5. El campo FlowStartTime_t indica la primera aparición de este tipo de flujo agregado (misma tupla de cuatro) en el intervalo de procesamiento de registros de flujos entre "FlowIntervalStartTime_t" y "FlowIntervalEndTime_t".
6. Para cualquier recurso en el Análisis de tráfico, los flujos que se indican en la interfaz de usuario son flujos totales vistos por NSG, pero en Log Anlaytics, el usuario verá solo el registro único reducido. Para ver todos los flujos, use el campo blob_id, al que se puede hacer referencia desde el almacenamiento. El número total de flujos de ese registro coincidirá con los flujos individuales que se ven en el blob.

La consulta siguiente le ayuda a ver todas las subredes que interactúan con direcciones IP públicas que no son de Azure en los últimos 30 días.
```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowStartTime_t >= ago(30d) and FlowType_s == "ExternalPublic"
| project Subnet1_s, Subnet2_s  
```
Para ver la ruta de acceso de blob para los flujos en la consulta mencionada anteriormente, use la siguiente consulta:

```
let TableWithBlobId =
(AzureNetworkAnalytics_CL
   | where SubType_s == "Topology" and ResourceType == "NetworkSecurityGroup" and DiscoveryRegion_s == Region_s and IsFlowEnabled_b
   | extend binTime = bin(TimeProcessed_t, 6h),
            nsgId = strcat(Subscription_g, "/", Name_s),
            saNameSplit = split(FlowLogStorageAccount_s, "/")
   | extend saName = iif(arraylength(saNameSplit) == 3, saNameSplit[2], '')
   | distinct nsgId, saName, binTime)
| join kind = rightouter (
   AzureNetworkAnalytics_CL
   | where SubType_s == "FlowLog"  
   | extend binTime = bin(FlowEndTime_t, 6h)
) on binTime, $left.nsgId == $right.NSGList_s  
| extend blobTime = format_datetime(todatetime(FlowIntervalStartTime_t), "yyyy MM dd hh")
| extend nsgComponents = split(toupper(NSGList_s), "/"), dateTimeComponents = split(blobTime, " ")
| extend BlobPath = strcat("https://", saName,
                        "@insights-logs-networksecuritygroupflowevent/resoureId=/SUBSCRIPTIONS/", nsgComponents[0],
                        "/RESOURCEGROUPS/", nsgComponents[1],
                        "/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/", nsgComponents[2],
                        "/y=", dateTimeComponents[0], "/m=", dateTimeComponents[1], "/d=", dateTimeComponents[2], "/h=", dateTimeComponents[3],
                        "/m=00/macAddress=", replace(@"-", "", MACAddress_s),
                        "/PT1H.json")
| project-away nsgId, saName, binTime, blobTime, nsgComponents, dateTimeComponents;

TableWithBlobId
| where SubType_s == "FlowLog" and FlowStartTime_t >= ago(30d) and FlowType_s == "ExternalPublic"
| project Subnet_s , BlobPath
```

La consulta anterior construye una dirección URL para acceder al blob directamente. A continuación figura la dirección URL con los marcadores de posición:

```
https://{saName}@insights-logs-networksecuritygroupflowevent/resoureId=/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroup}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json

```

### <a name="fields-used-in-traffic-analytics-schema"></a>Campos que se usan en el esquema de Análisis de tráfico
  > [!IMPORTANT]
  > El esquema de Análisis de tráfico se ha actualizado el 22 de agosto de 2019. El nuevo esquema proporciona direcciones IP de origen y de destino por separado, lo que elimina la necesidad de analizar el campo FlowDirection y simplifica las consultas. </br>
  > FASchemaVersion_s actualizado de 1 a 2. </br>
  > Campos en desuso: VMIP_s, Subscription_s, Region_s, NSGRules_s, Subnet_s, VM_s, NIC_s, PublicIPs_s, FlowCount_d </br>
  > Nuevos campos: SrcPublicIPs_s, DestPublicIPs_s, NSGRule_s </br>
  > Los campos en desuso estarán disponibles hasta el 22 de noviembre de 2019.

El Análisis de tráfico se basa en Log Analytics, por lo que puede ejecutar consultas personalizadas en los datos que Análisis de tráfico rellena y establecer alertas en ellas.

A continuación se proporcionan los campos en el esquema y su significado

| Campo | Formato | Comentarios |
|:---   |:---    |:---  |
| TableName | AzureNetworkAnalytics_CL | Tabla para los datos de Análisis de tráfico.
| SubType_s | FlowLog | Subtipo para los registros de flujo. Usar solo "FlowLog"; otros valores de SubType_s son para el funcionamiento interno del producto. |
| FASchemaVersion_s |   2   | Versión de esquema. No refleja la versión del registro de flujo de NSG. |
| TimeProcessed_t   | Fecha y hora en UTC  | Hora a la que el Análisis de tráfico procesó los registros de flujo sin formato desde la cuenta de almacenamiento. |
| FlowIntervalStartTime_t | Fecha y hora en UTC |  Hora de inicio del intervalo de procesamiento de registros de flujo. Se trata de la hora desde la que se mide el intervalo de flujos. |
| FlowIntervalEndTime_t | Fecha y hora en UTC | Hora de finalización del intervalo de procesamiento de registros de flujo. |
| FlowStartTime_t | Fecha y hora en UTC |  Primera aparición del flujo (que se agregará) en el intervalo de procesamiento de registros de flujos entre "FlowIntervalStartTime_t" y "FlowIntervalEndTime_t". Este flujo se agrega en función de la lógica de agregación. |
| FlowEndTime_t | Fecha y hora en UTC | Última aparición del flujo (que se agregará) en el intervalo de procesamiento de registros de flujos entre "FlowIntervalStartTime_t" y "FlowIntervalEndTime_t". Desde el punto de vista del registro de flujos v2, este campo contiene la hora de inicio del último flujo con la misma tupla de cuatro (marcado como "B" en el registro de flujos sin formato). |
| FlowType_s |  * IntraVNet <br> * InterVNet <br> * S2S <br> * P2S <br> * AzurePublic <br> * ExternalPublic <br> * MaliciousFlow <br> * UnknownPrivate <br> * Unknown | Definición en las notas después de la tabla. |
| SrcIP_s | Dirección IP de origen | Quedará en blanco en el caso de los flujos AzurePublic y ExternalPublic. |
| DestIP_s | Dirección IP de destino | Quedará en blanco en el caso de los flujos AzurePublic y ExternalPublic. |
| VMIP_s | Dirección IP de la VM. | Se usa para los flujos AzurePublic y ExternalPublic. |
| PublicIP_s | Direcciones IP públicas | Se usa para los flujos AzurePublic y ExternalPublic. |
| DestPort_d | Puerto de destino | Puerto en el que el tráfico es entrante. |
| L4Protocol_s  | * T <br> * U  | Protocolo de transporte. T = TCP <br> U = UDP |
| L7Protocol_s  | Nombre del protocolo | Derivado del puerto de destino. |
| FlowDirection_s | * I = Entrante<br> * O = Saliente | Dirección del flujo entrante o saliente de NSG según el registro de flujos |
| FlowStatus_s  | * A = Permitido por la regla de NSG <br> * D = Denegado por la regla de NSG  | Estado del flujo permitido o bloqueado por NSG según el registro de flujos. |
| NSGList_s | \<SUBSCRIPTIONID>\/<RESOURCEGROUP_NAME>\/<NSG_NAME> | Grupo de seguridad de red (NSG) asociado con el flujo. |
| NSGRules_s | \<Index value 0)>\|\<NSG_RULENAME>\|\<Flow Direction>\|\<Flow Status>\|\<FlowCount ProcessedByRule> |  Regla de NSG que permitió o denegó este flujo. |
| NSGRule_s | NSG_RULENAME |  Regla de NSG que permitió o denegó este flujo. |
| NSGRuleType_s | * Definido por el usuario * predeterminado |   El tipo de regla de NSG que el flujo usa. |
| MACAddress_s | Dirección MAC | Dirección MAC de la NIC en la que se capturó el flujo. |
| Subscription_s | En este campo se indica la suscripción de la red virtual, interfaz de red o máquina virtual de Azure. | Solo se aplica para los tipos de flujo FlowType = S2S, P2S, AzurePublic, ExternalPublic, MaliciousFlow y UnknownPrivate (tipos de flujo donde solo un lado es Azure). |
| Subscription1_s | Id. de suscripción | Id. de suscripción de la red virtual, interfaz de red o máquina virtual a la que pertenece la dirección IP de origen del flujo. |
| Subscription2_s | Id. de suscripción | Id. de suscripción de la red virtual, interfaz de red o máquina virtual a la que pertenece la dirección IP de origen en el flujo. |
| Region_s | Región de Azure de la red virtual, interfaz de red o máquina virtual a la que pertenece la dirección IP en el flujo. | Solo se aplica para los tipos de flujo FlowType = S2S, P2S, AzurePublic, ExternalPublic, MaliciousFlow y UnknownPrivate (tipos de flujo donde solo un lado es Azure). |
| Region1_s | Región de Azure | Región de Azure de la red virtual, interfaz de red o máquina virtual a la que pertenece la dirección IP de origen en el flujo. |
| Region2_s | Región de Azure | Región de Azure de la red virtual a la que pertenece la dirección IP de destino en el flujo. |
| NIC_s | \<resourcegroup_Name>\/\<NetworkInterfaceName> |  NIC asociada con la VM que envía o recibe el tráfico. |
| NIC1_s | <resourcegroup_Name>/\<NetworkInterfaceName> | NIC asociada con la dirección IP de origen en el flujo. |
| NIC2_s | <resourcegroup_Name>/\<NetworkInterfaceName> | NIC asociada con la dirección IP de destino en el flujo. |
| VM_s | <resourcegroup_Name>\/\<NetworkInterfaceName> | Máquina virtual asociada con la NIC_s de interfaz de red. |
| VM1_s | <resourcegroup_Name>/\<VirtualMachineName> | Máquina virtual asociada con la dirección IP de origen en el flujo. |
| VM2_s | <resourcegroup_Name>/\<VirtualMachineName> | Máquina virtual asociada con la dirección IP de destino en el flujo. |
| Subnet_s | <ResourceGroup_Name>/<VNET_Name>/\<SubnetName> | Subred asociada con la NIC_s. |
| Subnet1_s | <ResourceGroup_Name>/<VNET_Name>/\<SubnetName> | Subred asociada con la dirección IP de origen en el flujo. |
| Subnet2_s | <ResourceGroup_Name>/<VNET_Name>/\<SubnetName>    | Subred asociada con la dirección IP de destino en el flujo. |
| ApplicationGateway1_s | \<SubscriptionID>/\<ResourceGroupName>/\<ApplicationGatewayName> | Puerta de enlace de aplicación asociada con la dirección IP de origen en el flujo. |
| ApplicationGateway2_s | \<SubscriptionID>/\<ResourceGroupName>/\<ApplicationGatewayName> | Puerta de enlace de aplicaciones asociado con la dirección IP de destino en el flujo. |
| LoadBalancer1_s | \<SubscriptionID>/\<ResourceGroupName>/\<LoadBalancerName> | Equilibrador de carga asociado con la dirección IP de origen en el flujo. |
| LoadBalancer2_s | \<SubscriptionID>/\<ResourceGroupName>/\<LoadBalancerName> | Equilibrador de carga asociado con la dirección IP de destino en el flujo. |
| LocalNetworkGateway1_s | \<SubscriptionID>/\<ResourceGroupName>/\<LocalNetworkGatewayName> | Puerta de enlace de red local asociada con la dirección IP de origen en el flujo. |
| LocalNetworkGateway2_s | \<SubscriptionID>/\<ResourceGroupName>/\<LocalNetworkGatewayName> | Puerta de enlace de red local asociada con la dirección IP de destino en el flujo. |
| ConnectionType_s | Los valores posibles son VNetPeering, VpnGateway y ExpressRoute. |    Tipo de conexión. |
| ConnectionName_s | \<SubscriptionID>/\<ResourceGroupName>/\<ConnectionName> | Nombre de la conexión. En el caso del tipo de flujo de la conexión de punto a sitio, el formato será <gateway name>_<VPN Client IP> |
| ConnectingVNets_s | Lista separada por espacios de nombres de red virtual. | En el caso de una topología de concentrador y radio, las redes virtuales de centro se rellenarán aquí. |
| Country_s | Código de país o región de dos letras (ISO 3166-1 alfa-2). | Se rellena para el tipo de flujo ExternalPublic. Todas las direcciones IP en el campo PublicIPs_s compartirán el mismo código de país o región. |
| AzureRegion_s | Ubicaciones de la región de Azure | Se rellena para el tipo de flujo AzurePublic. Todas las direcciones IP en el campo PublicIPs_s compartirán la región de Azure. |
| AllowedInFlows_d | | Número de flujos entrantes que se permitieron. Esto representa el número de flujos que compartieron la misma tupla de cuatro a la interfaz de red en la que se capturó el flujo. |
| DeniedInFlows_d |  | Número de flujos entrantes que se denegaron. (Entrante en la interfaz de red en la que se capturó el flujo). |
| AllowedOutFlows_d | | Número de flujos salientes que se permitieron. (Saliente a la interfaz de red en la que se capturó el flujo). |
| DeniedOutFlows_d  | | Número de flujos salientes que se denegaron. (Saliente a la interfaz de red en la que se capturó el flujo). |
| FlowCount_d | En desuso. Total de flujos que coincidieron con la misma tupla de cuatro. En el caso de tipos de flujo ExternalPublic y AzurePublic, el número también incluirá los flujos de distintas direcciones de PublicIP.
| InboundPackets_d | Paquetes recibidos según se capturaron en la interfaz de red donde se aplicó la regla de NSG. | Esto se rellena solo para la versión 2 del esquema de registro de flujos de NSG. |
| OutboundPackets_d  | Paquetes enviados según se capturaron en la interfaz de red donde se aplicó la regla de NSG. | Esto se rellena solo para la versión 2 del esquema de registro de flujos de NSG. |
| InboundBytes_d |  Bytes recibidos según se capturaron en la interfaz de red donde se aplicó la regla de NSG. | Esto se rellena solo para la versión 2 del esquema de registro de flujos de NSG. |
| OutboundBytes_d | Bytes enviados según se capturaron en la interfaz de red donde se aplicó la regla de NSG. | Esto se rellena solo para la versión 2 del esquema de registro de flujos de NSG. |
| CompletedFlows_d  |  | Esto se rellenará con un valor distinto de cero solo para la versión 2 del esquema de registro de flujos de NSG. |
| PublicIPs_s | <PUBLIC_IP>\|\<FLOW_STARTED_COUNT>\|\<FLOW_ENDED_COUNT>\|\<OUTBOUND_PACKETS>\|\<INBOUND_PACKETS>\|\<OUTBOUND_BYTES>\|\<INBOUND_BYTES> | Entradas separadas por barras |
| SrcPublicIPs_s | <SOURCE_PUBLIC_IP>\|\<FLOW_STARTED_COUNT>\|\<FLOW_ENDED_COUNT>\|\<OUTBOUND_PACKETS>\|\<INBOUND_PACKETS>\|\<OUTBOUND_BYTES>\|\<INBOUND_BYTES> | Entradas separadas por barras |
| DestPublicIPs_s | <DESTINATION_PUBLIC_IP>\|\<FLOW_STARTED_COUNT>\|\<FLOW_ENDED_COUNT>\|\<OUTBOUND_PACKETS>\|\<INBOUND_PACKETS>\|\<OUTBOUND_BYTES>\|\<INBOUND_BYTES> | Entradas separadas por barras |

### <a name="notes"></a>Notas

1. En el caso de los flujos AzurePublic y ExternalPublic, la IP de la VM de Azure propiedad del cliente se rellena en el campo VMIP_s, mientras que las direcciones IP públicas que se rellenan en el campo PublicIPs_s. Para dos estos tipos de flujo de dos, debemos usar VMIP_s y PublicIPs_s en lugar de campos SrcIP_s y DestIP_s. Para las direcciones de AzurePublic y ExternalPublicIP, agregamos aún más de modo que el número de registros ingeridos en el área de trabajo de análisis de registros de cliente sea mínimo. (Este campo quedará en desuso próximamente y deberíamos usar SrcIP_ y DestIP_s, en función de si la VM de Azure era el origen o el destino del flujo).
1. Detalles de los tipos de flujo: En función de las direcciones IP que se usan en el flujo, los flujos se clasifican en los siguientes tipos de flujo:
1. IntraVNet: ambas direcciones IP en el flujo residen en la misma red virtual de Azure.
1. InterVNet: las direcciones IP en el flujo residen en las dos diferentes redes virtuales de Azure.
1. S2S: (sitio a sitio) una de las direcciones IP pertenece a la red virtual de Azure mientras que la otra dirección IP pertenece a la red del cliente (sitio) conectado a la red virtual de Azure a través de la puerta de enlace de VPN o Express Route.
1. P2S: (punto a punto) una de las direcciones IP pertenece a la red virtual de Azure mientras que la otra dirección IP pertenece a la red del cliente (sitio) conectado a la red virtual de Azure a través de la puerta de enlace de VPN.
1. AzurePublic: una de las direcciones IP pertenece a la red virtual de Azure mientras que la otra dirección IP pertenece a las direcciones IP públicas internas de Azure propiedad de Microsoft. Las direcciones IP públicas propiedad del cliente no formarán parte de este tipo de flujo. Por ejemplo, una VM propiedad del cliente que envíe tráfico a un servicio de Azure (punto de conexión de almacenamiento) se clasificaría en este tipo de flujo.
1. ExternalPublic: una de las direcciones IP pertenece a la red virtual de Azure, mientras que la otra dirección IP es una dirección IP pública que no está en Azure, no se notifica como malintencionada en las fuentes ASC que Análisis de tráfico consume para el intervalo de procesamiento entre "FlowIntervalStartTime_t" y "FlowIntervalEndTime_t".
1. MaliciousFlow: una de las direcciones IP pertenece a la red virtual de Azure, mientras que la otra dirección IP es una dirección IP pública que no está en Azure, se notifica como malintencionada en las fuentes ASC que Análisis de tráfico consume para el intervalo de procesamiento entre "FlowIntervalStartTime_t" y "FlowIntervalEndTime_t".
1. UnknownPrivate: una de las direcciones IP pertenecen a la red virtual de Azure, mientras que la otra dirección IP pertenece al intervalo IP privadas según se define en RFC 1918 y Análisis de tráfico no la puede asignar a un sitio o red virtual de Azure propiedad del cliente.
1. Unknown: no se puede asignar ninguna de las direcciones IP en los flujos con la topología de cliente en Azure ni de forma local (sitio).
1. Algunos nombres de campo se anexan con \_s o \_d. NO significan el origen y el destino, sino que indican los tipos de datos de cadena y decimales respectivamente.

### <a name="next-steps"></a>Pasos siguientes
Para obtener respuestas a las preguntas más frecuentes, consulte [Preguntas más frecuentes sobre Análisis de tráfico](traffic-analytics-faq.yml). Para ver detalles sobre la funcionalidad, consulte la [documentación de Análisis de tráfico](traffic-analytics.md)
