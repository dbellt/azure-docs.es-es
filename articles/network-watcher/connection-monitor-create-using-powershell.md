---
title: 'Creación de una instancia de Connection Monitor: PowerShell'
titleSuffix: Azure Network Watcher
description: Aprenda a crear una instancia de Connection Monitor mediante PowerShell.
services: network-watcher
documentationcenter: na
author: vinigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: vinigam
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1b9fe56663f49e7fc6a7068e57df6d20003d2330
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110662025"
---
# <a name="create-a-connection-monitor-by-using-powershell"></a>Creación de una instancia de Connection Monitor mediante PowerShell

> [!IMPORTANT]
> A partir del 1 de julio de 2021, no podrá agregar nuevas pruebas en un área de trabajo existente ni habilitar un área de trabajo nueva en Network Performance Monitor. Tampoco podrá agregar nuevos monitores de conexión en Connection Monitor (clásico). Puede seguir usando las pruebas y los monitores de conexión creados antes del 1 de julio de 2021. Para minimizar la interrupción del servicio en las cargas de trabajo actuales, [migre las pruebas desde Network Performance Monitor ](migrate-to-connection-monitor-from-network-performance-monitor.md) o [desde Connection Monitor (clásico)](migrate-to-connection-monitor-from-connection-monitor-classic.md) a la nueva instancia de Connection Monitor en Azure Network Watcher antes del 29 de febrero de 2024.


Aprenda a usar la característica Connection Monitor de Azure Network Watcher para supervisar la comunicación entre los recursos.


## <a name="before-you-begin"></a>Antes de empezar

En los monitores de conexión que se crean con Connection Monitor es posible agregar máquinas locales y máquinas virtuales de Azure como orígenes. Estos monitores de conexión también pueden supervisar la conectividad con los puntos de conexión. Los puntos de conexión pueden estar en Azure o en cualquier otra dirección URL o IP.

Una instancia de Connection Monitor incluye las siguientes entidades:

* **Recurso de Connection Monitor**: Un recurso de Azure específico de una región. Todas las entidades siguientes son propiedades del recurso de Connection Monitor.
* **Punto de conexión**: Un origen o destino que participa en las comprobaciones de conectividad. Algunos ejemplos de puntos de conexión son las máquinas virtuales de Azure, los agentes locales, las direcciones URL y las direcciones IP.
* **Configuración de prueba**: La configuración de una prueba específica para el protocolo. En función del protocolo elegido, puede definir el puerto, los umbrales, la frecuencia de las pruebas y otros parámetros.
* **Grupo de prueba**: El grupo que contiene puntos de conexión de origen, puntos de conexión de destino y configuraciones de prueba. Un monitor de conexión puede contener más de un grupo de prueba.
* **Prueba**: La combinación de un punto de conexión de origen, un punto de conexión de destino y una configuración de prueba. Una prueba es el nivel más granular en el que están disponibles los datos de supervisión. Los datos de supervisión incluyen el porcentaje de comprobaciones con errores y el tiempo de ida y vuelta (RTT).

    ![Diagrama que muestra un monitor de conexión y donde se define la relación entre los grupos de prueba y las pruebas.](./media/connection-monitor-2-preview/cm-tg-2.png)

## <a name="steps-to-create-a-connection-monitor"></a>Pasos para crear un monitor de conexión

Use los comandos siguientes para crear un monitor de conexión mediante PowerShell.

```powershell

//Connect to your Azure account with the subscription
Connect-AzAccount
Select-AzSubscription -SubscriptionId <your-subscription>
//Select region
$nw = "NetworkWatcher_centraluseuap"
//Declare endpoints like Azure VM below. You can also give VNET,Subnet,Log Analytics workspace
$sourcevmid1 = New-AzNetworkWatcherConnectionMonitorEndpointObject -Name MyAzureVm -ResourceID /subscriptions/<your-subscription>/resourceGroups/<your resourceGroup>/providers/Microsoft.Compute/virtualMachines/<vm-name>
//Declare endpoints like URL, IPs
$bingEndpoint = New-AzNetworkWatcherConnectionMonitorEndpointObject -name Bing -Address www.bing.com # Destination URL
//Create test configuration.Choose Protocol and parametersSample configs below.

$IcmpProtocolConfiguration = New-AzNetworkWatcherConnectionMonitorProtocolConfigurationObject -IcmpProtocol
$TcpProtocolConfiguration = New-AzNetworkWatcherConnectionMonitorProtocolConfigurationObject -TcpProtocol -Port 80
$httpProtocolConfiguration = New-AzNetworkWatcherConnectionMonitorProtocolConfigurationObject -HttpProtocol -Port 443 -Method GET -RequestHeader @{Allow = "GET"} -ValidStatusCodeRange 2xx, 300-308 -PreferHTTPS
$httpTestConfiguration = New-AzNetworkWatcherConnectionMonitorTestConfigurationObject -Name http-tc -TestFrequencySec 60 -ProtocolConfiguration $httpProtocolConfiguration -SuccessThresholdChecksFailedPercent 20 -SuccessThresholdRoundTripTimeMs 30
$icmpTestConfiguration = New-AzNetworkWatcherConnectionMonitorTestConfigurationObject -Name icmp-tc -TestFrequencySec 30 -ProtocolConfiguration $icmpProtocolConfiguration -SuccessThresholdChecksFailedPercent 5 -SuccessThresholdRoundTripTimeMs 500
$tcpTestConfiguration = New-AzNetworkWatcherConnectionMonitorTestConfigurationObject -Name tcp-tc -TestFrequencySec 60 -ProtocolConfiguration $TcpProtocolConfiguration -SuccessThresholdChecksFailedPercent 20 -SuccessThresholdRoundTripTimeMs 30
//Create Test Group
$testGroup1 = New-AzNetworkWatcherConnectionMonitorTestGroupObject -Name testGroup1 -TestConfiguration $httpTestConfiguration, $tcpTestConfiguration, $icmpTestConfiguration -Source $sourcevmid1 -Destination $bingEndpoint,
$testname = "cmtest9"
//Create Connection Monitor
New-AzNetworkWatcherConnectionMonitor -NetworkWatcherName $nw -ResourceGroupName NetworkWatcherRG -Name $testname -TestGroup $testGroup1

```

## <a name="description-of-properties"></a>Descripción de las propiedades

* **ConnectionMonitorName**: nombre del recurso de Connection Monitor.

* **SUB**: identificador de la suscripción donde quiere crear un monitor de conexión.

* **NW**: identificador de recurso de Network Watcher en el que se crea un monitor de conexión.

* **Ubicación**: región en la que se crea un monitor de conexión.

* **Extremos**
    * **Name**: nombre único de cada punto de conexión.
    * **Identificador de recurso**: en el caso de los puntos de conexión de Azure, el identificador de recurso hace referencia al identificador de recurso de Azure Resource Manager de las máquinas virtuales. En el caso de los puntos de conexión que no son de Azure, el identificador de recurso hace referencia al identificador de recurso de Azure Resource Manager del área de trabajo de Log Analytics vinculada a agentes que no son de Azure.
    * **Address**: solo se aplica cuando no se especifica ningún identificador de recurso o si el identificador de recurso está en el área de trabajo de Log Analytics. Si se usa sin un identificador de recurso, puede ser la dirección URL o IP de cualquier punto de conexión público. Si se usa con un identificador de recurso de Log Analytics, se refiere al FQDN del agente de supervisión.
    * **Filtro**: en el caso de los puntos de conexión que no son de Azure, use filtros para seleccionar los agentes de supervisión del área de trabajo de Log Analytics del recurso de Connection Monitor. Si no se establecen filtros, todos los agentes que pertenecen al área de trabajo de Log Analytics se pueden usar para la supervisión.
        * **Tipo**: se establece en **Dirección de agente**.
        * **Address**: se establece en el FQDN del agente local.

* **Grupos de prueba**
    * **Nombre**: asigne un nombre al grupo de prueba.
    * **Orígenes**: elija entre los puntos de conexión que ha creado anteriormente. Los puntos de conexión de origen basados en Azure deben tener instalada la extensión Azure Network Watcher; los que no se basan en Azure deben tener instalado un agente de Azure Log Analytics. Para instalar un agente para el origen, consulte [Instalación de los agentes de supervisión](./connection-monitor-overview.md#install-monitoring-agents).
    * **Destinos**:  elija entre los puntos de conexión que ha creado anteriormente. Se puede supervisar la conectividad a las máquinas virtuales de Azure o a cualquier punto de conexión (una IP pública, una dirección URL o un FQDN) si se especifican como destinos. En un solo grupo de prueba, puede agregar máquinas virtuales de Azure, direcciones URL de Office 365, direcciones URL de Dynamics 365 y puntos de conexión personalizados.
    * **Disable**: deshabilite la supervisión de todos los orígenes y destinos que especifica el grupo de prueba.

* **Configuraciones de prueba**
    * **Name**: asigne un nombre a la configuración de prueba.
    * **TestFrequencySec**: especifique la frecuencia con la que los orígenes hacen ping a los destinos en el protocolo y el puerto especificados. Puede elegir 30 segundos, 1 minuto, 5 minutos, 15 minutos o 30 minutos. Los orígenes prueban la conectividad a los destinos según el valor que se seleccione. Por ejemplo, si selecciona 30 segundos, los orígenes comprueban la conectividad al destino al menos una vez en un período de 30 segundos.
    * **Protocolo**: elija TCP, ICMP, HTTP o HTTPS. En función del protocolo, también puede seleccionar las siguientes configuraciones específicas de cada protocolo:
        * **preferHTTPS**: especifique si se usa HTTPS en lugar de HTTP.
        * **port**: especifique el puerto de destino que prefiera.
        * **disableTraceRoute**: evite que los orígenes detecten la topología y el valor de RTT de salto a salto. Esto se aplica a los grupos de prueba con TCP o ICMP.
        * **method**: seleccione el método de solicitud HTTP (GET o POST). Se aplica a las configuraciones de prueba con HTTP.
        * **path**: especifique los parámetros de ruta de acceso que se van a anexar a la dirección URL.
        * **validStatusCodes**: elija los códigos de estado aplicables. Si el código de respuesta no coincide, se muestra un mensaje de diagnóstico.
        * **requestHeaders**: especifique cadenas de encabezado de solicitud personalizadas que pasen al destino.
    * **Success threshold**: establezca umbrales en los siguientes parámetros de red:
        * **checksFailedPercent**: establezca el porcentaje de comprobaciones que pueden presentar errores cuando los orígenes comprueban la conectividad con los destinos mediante los criterios especificados. En el caso del protocolo TCP o ICMP, el porcentaje de comprobaciones erróneas puede ser igual al porcentaje de pérdida de paquetes. En el caso del protocolo HTTP, este campo representa el porcentaje de solicitudes HTTP que no recibieron respuesta.
        * **roundTripTimeMs**: establezca cuánto tiempo en milisegundos pueden tardar los orígenes en conectarse al destino a través de la configuración de prueba.

## <a name="scale-limits"></a>Límites de escalado

Los monitores de conexión tienen los límites de escala siguientes:

* Máximo de monitores de conexión por suscripción por región: 100
* Máximo de grupos de prueba por monitor de conexión: 20
* Máximo de orígenes y destinos por monitor de conexión: 100
* Máximo de configuraciones de prueba por monitor de conexión: 20

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [analizar los datos de supervisión y a definir alertas](./connection-monitor-overview.md#analyze-monitoring-data-and-set-alerts).
* Aprenda a [Diagnosticar problemas en la red](./connection-monitor-overview.md#diagnose-issues-in-your-network).
