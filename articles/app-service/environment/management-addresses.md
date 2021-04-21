---
title: Direcciones de administración
description: Busque las direcciones de administración utilizadas para controlar una instancia de App Service Environment. Configúrelas en una tabla de rutas para evitar problemas de enrutamiento asimétrico.
author: ccompy
ms.assetid: a7738a24-89ef-43d3-bff1-77f43d5a3952
ms.topic: article
ms.date: 03/22/2021
ms.author: ccompy
ms.custom: seodec18, references_regions, devx-track-azurecli
ms.openlocfilehash: aaaa190935da8c016c43832712f553a116332974
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107482575"
---
# <a name="app-service-environment-management-addresses"></a>Direcciones de administración de App Service Environment

App Service Environment (ASE) es una implementación de inquilino único de Azure App Service que se ejecuta en su instancia de Azure Virtual Network (VNet).  Si bien el ASE se ejecuta en su red virtual, debe seguir siendo accesible desde varias direcciones IP dedicadas que utiliza Azure App Service para administrar el servicio.  En el caso de un ASE, el tráfico de administración atraviesa la red controlada por el usuario. Si este tráfico está bloqueado o no está correctamente dirigido, se suspenderá la instancia de ASE. Para más información sobre las dependencias de redes de ASE, consulte [Consideraciones de redes y App Service Environment][networking]. Para información general sobre ASE, puede comenzar con la [Introducción a App Service Environment][intro].

Todas las instancias de ASE tienen una IP virtual pública por la que entra el tráfico de administración. El tráfico de administración entrante desde estas direcciones entra por los puertos 454 y 455 en la IP virtual pública de su instancia de ASE. Este documento enumera la direcciones de origen de App Service para el tráfico de administración de la instancia de ASE. Estas direcciones se encuentran también en la etiqueta de servicio IP denominada AppServiceManagement.

Las direcciones que se indican a continuación pueden configurarse en una tabla de rutas para evitar problemas de enrutamientos asimétricos con el tráfico de administración. Las rutas actúan sobre el tráfico en el nivel de IP y no tienen conocimiento de la dirección del tráfico o de que el tráfico forme parte de un mensaje de respuesta de TCP. Si la dirección de respuesta para una solicitud TCP es diferente de la dirección a la que se envió, tiene un problema de enrutamiento asimétrico. Para evitar problemas de enrutamiento asimétrico con el tráfico de administración de ASE, debe asegurarse de que las respuestas se envíen desde la misma dirección a la que se enviaron. Para más información sobre cómo configurar su ASE para que funcione en un entorno donde el tráfico saliente se envía de forma local, lea [Configuración de App Service Environment con tunelización forzada][forcedtunnel].

## <a name="list-of-management-addresses"></a>Lista de direcciones de administración ##

| Region | Direcciones |
|--------|-----------|
| Todas las regiones públicas | 13.66.140.0, 13.67.8.128, 13.69.64.128, 13.69.227.128, 13.70.73.128, 13.71.170.64, 13.71.194.129, 13.75.34.192, 13.75.127.117, 13.77.50.128, 13.78.109.0, 13.89.171.0, 13.94.141.115, 13.94.143.126, 13.94.149.179, 20.36.106.128, 20.36.114.64, 20.37.74.128, 23.96.195.3, 23.102.135.246, 23.102.188.65, 40.69.106.128, 40.70.146.128, 40.71.13.64, 40.74.100.64, 40.78.194.128, 40.79.130.64, 40.79.178.128, 40.83.120.64, 40.83.121.56, 40.83.125.161, 40.112.242.192, 51.107.58.192, 51.107.154.192, 51.116.58.192, 51.116.155.0, 51.120.99.0, 51.120.219.0, 51.140.146.64, 51.140.210.128, 52.151.25.45, 52.162.106.192, 52.165.152.214, 52.165.153.122, 52.165.154.193, 52.165.158.140, 52.174.22.21, 52.178.177.147, 52.178.184.149, 52.178.190.65, 52.178.195.197, 52.187.56.50, 52.187.59.251, 52.187.63.19, 52.187.63.37, 52.224.105.172, 52.225.177.153, 52.231.18.64, 52.231.146.128, 65.52.172.237, 65.52.250.128, 70.37.57.58, 104.44.129.141, 104.44.129.243, 104.44.129.255, 104.44.134.255, 104.208.54.11, 104.211.81.64, 104.211.146.128, 157.55.208.185, 191.233.50.128, 191.233.203.64, 191.236.154.88 |
| Microsoft Azure Government | 23.97.29.209, 13.72.53.37, 13.72.180.105, 52.181.183.11, 52.227.80.100, 52.182.93.40, 52.244.79.34, 52.238.74.16 |
| Azure China | 42.159.4.236, 42.159.80.125 |

## <a name="configuring-a-network-security-group"></a>Configuración de un grupo de seguridad de red

Con los grupos de seguridad de red, no es necesario preocuparse de las direcciones individuales ni mantener su propia configuración. Hay una etiqueta de servicio IP denominada AppServiceManagement que se mantiene al día con todas las direcciones. Para usar esta etiqueta de servicio IP en el grupo de seguridad de red, vaya al portal, abra la interfaz de usuario de grupos de seguridad de red y seleccione las reglas de seguridad de entrada. Si tiene una regla previa para el tráfico de administración de entrada, modifíquela. Si este grupo de seguridad de red no se creó con el ASE o si es totalmente nuevo, seleccione **Agregar**. En la lista desplegable Origen, seleccione **Etiqueta de servicio**.  En la etiqueta de servicio Origen, seleccione **AppServiceManagement**. Establezca los intervalos de puertos de origen para \*destino en **Cualquiera**, los intervalos de puerto de destino en **454-455**, protocolo en **TCP** y acción en **Permitir**. Si está creando la regla, debe establecer la prioridad. 

![Creación de un grupo de seguridad de red con la etiqueta de servicio][1]

## <a name="configuring-a-route-table"></a>Configuración de una tabla de rutas

Las direcciones de administración pueden colocarse en una tabla de rutas con un próximo salto de internet para asegurarse de que todo el tráfico de administración de entrada es capaz de recorrer la misma ruta de acceso. Estas rutas se necesitan al configurar la tunelización forzada. Para crear la tabla de rutas, puede usar el portal, PowerShell o la CLI de Azure.  Los comandos para crear una tabla de rutas mediante la CLI de Azure desde un símbolo del sistema de PowerShell se encuentran a continuación. 

```azurepowershell-interactive
$rg = "resource group name"
$rt = "route table name"
$location = "azure location"
$managementAddresses = "13.66.140.0", "13.67.8.128", "13.69.64.128", "13.69.227.128", "13.70.73.128", "13.71.170.64", "13.71.194.129", "13.75.34.192", "13.75.127.117", "13.77.50.128", "13.78.109.0", "13.89.171.0", "13.94.141.115", "13.94.143.126", "13.94.149.179", "20.36.106.128", "20.36.114.64", "20.37.74.128", "23.96.195.3", "23.102.135.246", "23.102.188.65", "40.69.106.128", "40.70.146.128", "40.71.13.64", "40.74.100.64", "40.78.194.128", "40.79.130.64", "40.79.178.128", "40.83.120.64", "40.83.121.56", "40.83.125.161", "40.112.242.192", "51.107.58.192", "51.107.154.192", "51.116.58.192", "51.116.155.0", "51.120.99.0", "51.120.219.0", "51.140.146.64", "51.140.210.128", "52.151.25.45", "52.162.106.192", "52.165.152.214", "52.165.153.122", "52.165.154.193", "52.165.158.140", "52.174.22.21", "52.178.177.147", "52.178.184.149", "52.178.190.65", "52.178.195.197", "52.187.56.50", "52.187.59.251", "52.187.63.19", "52.187.63.37", "52.224.105.172", "52.225.177.153", "52.231.18.64", "52.231.146.128", "65.52.172.237", "65.52.250.128", "70.37.57.58", "104.44.129.141", "104.44.129.243", "104.44.129.255", "104.44.134.255", "104.208.54.11", "104.211.81.64", "104.211.146.128", "157.55.208.185", "191.233.50.128", "191.233.203.64", "191.236.154.88"

az network route-table create --name $rt --resource-group $rg --location $location
foreach ($ip in $managementAddresses) {
    az network route-table route create -g $rg --route-table-name $rt -n $ip --next-hop-type Internet --address-prefix ($ip + "/32")
}
```

Una vez creada la tabla de rutas, deberá establecerla en la subred de ASE.  

## <a name="get-your-management-addresses-from-api"></a>Obtención de direcciones de administración de la API ##

Puede enumerar las direcciones de administración que coinciden con el ASE con la siguiente llamada API.

```http
get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01
```

La API devuelve un documento JSON que incluye todas las direcciones entrantes para su ASE. La lista de direcciones incluye las direcciones de administración, la dirección IP virtual que utiliza su ASE y el intervalo de direcciones de subred de ASE propio.  

Para llamar a la API con [armclient](https://github.com/projectkudu/ARMClient), use los siguientes comandos pero sustituya el identificador de suscripción, el grupo de recursos y el nombre de ASE.  

```azurepowershell-interactive
armclient login
armclient get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01
```

<!--IMAGES-->
[1]: ./media/management-addresses/managementaddr-nsg.png

<!-- LINKS -->
[networking]: ./network-info.md
[intro]: ./intro.md
[forcedtunnel]: ./forced-tunnel-support.md
