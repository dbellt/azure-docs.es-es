---
title: Escalado y hospedaje de Azure Functions
description: Aprenda a elegir entre el plan de consumo de Azure Functions y el plan Prémium.
ms.assetid: 5b63649c-ec7f-4564-b168-e0a74cb7e0f3
ms.topic: conceptual
ms.date: 08/17/2020
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2cf201c17b97464386ce52f4d689c2c1006d0970
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377422"
---
# <a name="azure-functions-hosting-options"></a>Opciones de hospedaje de Azure Functions

Cuando crea una aplicación de funciones en Azure, debe elegir un plan de hospedaje para su aplicación. Hay tres planes de hospedaje básico disponibles para Azure Functions: [Plan de consumo](consumption-plan.md), [plan Premium](functions-premium-plan.md) y [plan Dedicado (App Service)](dedicated-plan.md). Todos los planes de hospedaje tienen disponibilidad general (GA) en máquinas virtuales Linux y Windows.

El plan de hospedaje que elija determina los comportamientos siguientes:

* Cómo se escala la aplicación de funciones.
* Los recursos disponibles para cada instancia de aplicación de funciones.
* Compatibilidad con funcionalidad avanzada, como la conectividad con Azure Virtual Network.

En este artículo se proporciona una comparación detallada entre los distintos planes de hospedaje, además del [hospedaje basado en Kubernetes](functions-kubernetes-keda.md).

> [!NOTE]
> Si decide hospedar las funciones en un clúster de Kubernetes, considere la posibilidad de usar un [clúster de Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/overview.md). El hospedaje en un clúster de Kubernetes habilitado para Azure Arc está actualmente en versión preliminar. Para más información, consulte [App Service, Functions y Logic Apps en Azure Arc](../app-service/overview-arc-integration.md).  

## <a name="overview-of-plans"></a>Información general sobre los planes

A continuación se ofrece un resumen de las ventajas de los tres principales planes de hospedaje de Functions:

| Planear | Ventajas |
| --- | --- |  
|**[Plan de consumo](consumption-plan.md)**| Escale de forma automática y pague los recursos de proceso solo cuando se ejecuten las funciones.<br/><br/>En el plan de consumo, las instancias del host de Functions se agregan y quitan de forma dinámica según el número de eventos de entrada.<br/><br/> ✔ Plan de hospedaje predeterminado.<br/>✔ Pague solo cuando se ejecutan las funciones.<br/>✔ Escala de forma automática, incluso durante períodos de carga elevada.|  
|**[Plan Premium](functions-premium-plan.md)**|Escala automáticamente en función de la demanda mediante trabajos preparados previamente que ejecutan aplicaciones sin ningún retraso después de estar inactivas, ejecuta en instancias más eficaces y se conecta a redes virtuales. <br/><br/>Considere la posibilidad de elegir el plan Premium de Azure Functions en las siguientes situaciones: <br/><br/>✔ La aplicación de funciones se ejecuta de forma continua, o casi continua.<br/>✔ Tiene un gran número de ejecuciones pequeñas y una factura de ejecución elevada, pero pocos GB por segundo en el plan de consumo.<br/>✔ Necesita más opciones de CPU o memoria de las que proporciona el plan de consumo.<br/>✔ Su código debe ejecutarse durante más tiempo del máximo permitido en el plan de consumo.<br/>✔ Necesita características que no están disponibles en el plan de consumo, como la conectividad con red virtual.|  
|**[Plan dedicado](dedicated-plan.md)** |Ejecute las funciones en un plan de App Service a los [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/windows/) normales.<br/><br/>Mejor para escenarios de ejecución prolongada en los que no se puede usar [Durable Functions](durable/durable-functions-overview.md). Considere el plan de App Service en las situaciones siguientes:<br/><br/>✔ Tiene máquinas virtuales infrautilizadas que ya ejecutan otras instancias de App Service.<br/>✔ Quiere proporcionar una imagen personalizada en la que ejecutar sus funciones. <br/>✔ Se requieren escalado y costos predictivos.|  

En las tablas de comparación de este artículo también se incluyen las siguientes opciones de hospedaje, que proporcionan el mayor control y aislamiento posibles en los que ejecutar las aplicaciones de funciones.  

| Opción de hospedaje | Detalles |
| --- | --- |  
|**[ASE](dedicated-plan.md)** | App Service Environment (ASE) es una característica de App Service que proporciona un entorno completamente aislado y dedicado para ejecutar de forma segura las aplicaciones de App Service a gran escala.<br/><br/>Las instancias de ASE son adecuadas para cargas de trabajo de aplicaciones que necesitan: <br/><br/>✔ Una gran escala.<br/>✔ Aislamiento de proceso completo y acceso a redes seguro.<br/>✔ Elevado uso de memoria.|  
| **Kubernetes**<br/>([Directo](functions-kubernetes-keda.md) o<br/>[Azure Arc](../app-service/overview-arc-integration.md)) | Kubernetes proporciona un entorno completamente aislado y dedicado que se ejecuta sobre la plataforma de Kubernetes.<br/><br/> Kubernetes resulta adecuado para cargas de trabajo de aplicaciones que necesitan: <br/>✔ Requisitos de hardware personalizados.<br/>✔ Aislamiento y acceso a redes seguro.<br/>✔ Capacidad de ejecutarse en entornos híbridos o de varias nubes.<br/>✔ Ejecutarse junto con aplicaciones y servicios de Kubernetes existentes.|  

En las tablas restantes de este artículo se comparan los planes con respecto a diversas características y comportamientos. Para obtener una comparación de costos entre los planes de hospedaje dinámicos (consumo y Premium), vea la página [Precios de Azure Functions](https://azure.microsoft.com/pricing/details/functions/). Para obtener los precios de las diversas opciones del plan dedicado, vea la página [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/windows/). 

## <a name="operating-systemruntime"></a>Sistema operativo o entorno de ejecución

En la tabla siguiente se muestran los sistemas operativos admitidos y la compatibilidad del runtime de lenguaje con los planes de hospedaje.

| | Linux<sup>1</sup><br/>Solo código | Windows<sup>2</sup><br/>Solo código | Linux<sup>1, 3</sup><br/>Contenedor de Docker |
| --- | --- | --- | --- |
| **[Plan de consumo](consumption-plan.md)** | .NET Core<br/>Node.js<br/>Java<br/>Python | .NET Core<br/>Node.js<br/>Java<br/>PowerShell Core | No se admite  |
| **[Plan Premium](functions-premium-plan.md)** | .NET Core<br/>Node.js<br/>Java<br/>Python|.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python  | 
| **[Plan dedicado](dedicated-plan.md)** | .NET Core<br/>Node.js<br/>Java<br/>Python|.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python |
| **[ASE](dedicated-plan.md)** | .NET Core<br/>Node.js<br/>Java<br/>Python |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core  |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python | 
| **[Kubernetes (directo)](functions-kubernetes-keda.md)** | N/D | N/D |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python |
| **[Azure Arc (versión preliminar)](../app-service/overview-arc-integration.md)** | .NET Core<br/>Node.js<br/>Java<br/>Python | N/D |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python |

<sup>1</sup> Linux es el único sistema operativo compatible con la pila del runtime de Python. <br/>
<sup>2</sup> Windows es el único sistema operativo compatible con la pila del runtime de PowerShell.<br/>
<sup>3</sup> Linux es el único sistema operativo compatible con los contenedores de Docker.<br/>

[!INCLUDE [Timeout Duration section](../../includes/functions-timeout-duration.md)]

## <a name="scale"></a>Escala

En la siguiente tabla se comparan los comportamientos de escalado de los distintos planes de hospedaje.

| Planear | Escalado horizontal | N.º máximo de instancias |
| --- | --- | --- |
| **[Plan de consumo](consumption-plan.md)** | [Controlado por eventos](event-driven-scaling.md). Escale horizontalmente de forma automática, incluso durante períodos de gran carga. La infraestructura de Azure Functions escala los recursos de CPU y memoria mediante la incorporación de instancias adicionales del host de Functions, según el número de eventos de desencadenador entrantes. | 200 |
| **[Plan Premium](functions-premium-plan.md)** | [Controlado por eventos](event-driven-scaling.md). Escale horizontalmente de forma automática, incluso durante períodos de gran carga. La infraestructura de Azure Functions escala automáticamente los recursos de CPU y la memoria mediante la incorporación de instancias del host de Functions, según el número de eventos desencadenados por las funciones. |100|
| **[Plan dedicado](dedicated-plan.md)** <sup>1</sup> | Escalabilidad automática o manual |10-20|
| **[ASE](dedicated-plan.md)** <sup>1</sup> | Escalabilidad automática o manual |100 |
| **[Kubernetes](functions-kubernetes-keda.md)**  | Escalado automático basado en eventos para los clústeres de Kubernetes mediante [KEDA](https://keda.sh). | Varía &nbsp;por&nbsp; clúster&nbsp;&nbsp;|

<sup>1</sup> Para conocer los límites específicos de las distintas opciones de plan de App Service, consulte [Límites del plan de App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).

## <a name="cold-start-behavior"></a>Comportamiento del arranque en frío

| Planear | Detalles | 
| -- | -- |
| **[Plan de&nbsp;consumo](consumption-plan.md)** | Las aplicaciones pueden escalar a cero si están inactivas, lo que significa que algunas solicitudes pueden tener una latencia adicional durante el arranque.  El plan de consumo incluye algunas optimizaciones para reducir el tiempo de arranque en frío, incluida la extracción desde funciones de marcador de posición previamente preparadas que ya tienen en ejecución un host de función y procesos de lenguaje. |
| **[Plan Premium](functions-premium-plan.md)** | Instancias permanentemente preparadas para evitar cualquier arranque en frío. |
| **[Plan dedicado](dedicated-plan.md)** | Cuando se ejecuta en un plan dedicado, el host de Functions se puede ejecutar de forma continua, lo que significa que el arranque en frío no es realmente un problema. |
| **[ASE](dedicated-plan.md)** | Cuando se ejecuta en un plan dedicado, el host de Functions se puede ejecutar de forma continua, lo que significa que el arranque en frío no es realmente un problema. |
| **[Kubernetes](functions-kubernetes-keda.md)**  | En función de la configuración de KEDA, las aplicaciones se pueden configurar para evitar un arranque en frío. Si están configuradas para escalar a cero, se produce un arranque en frío de los nuevos eventos. 

## <a name="service-limits"></a>Límites de servicio

[!INCLUDE [functions-limits](../../includes/functions-limits.md)]

## <a name="networking-features"></a>Características de red

[!INCLUDE [functions-networking-features](../../includes/functions-networking-features.md)]

## <a name="billing"></a>Facturación

| Planear | Detalles |
| --- | --- |
| **[Plan de consumo](consumption-plan.md)** | Solo paga por el tiempo durante el que se ejecutan las funciones. La facturación se basa en el número de ejecuciones, el tiempo de ejecución y el uso de la memoria. |
| **[Plan Premium](functions-premium-plan.md)** | El plan Premium se basa en la cantidad de núcleos por segundo y en la memoria usada en las instancias necesarias y preparadas previamente. Al menos una instancia por plan se debe mantener preparada en todo momento. Este plan ofrece los precios más predecibles. |
| * *[Plan dedicado](dedicated-plan.md)* | Paga lo mismo por las aplicaciones de funciones en un plan de App Service que por otros recursos de App Service, como las aplicaciones web.|
| **[App Service Environment (ASE)](dedicated-plan.md)** | Hay una tarifa plana mensual para las instancias de ASE en la que se paga por la infraestructura y que no varía según el tamaño de la instancia de ASE. Además, existe un costo por cada vCPU del plan de App Service. Todas las aplicaciones hospedadas en una instancia de ASE están en el SKU de precios Aislado. |
| **[Kubernetes](functions-kubernetes-keda.md)**| Solo paga los costos del clúster de Kubernetes; no existe facturación adicional para Functions. La aplicación de funciones se ejecuta como una carga de trabajo de aplicación en el clúster, al igual que una aplicación normal. |

## <a name="next-steps"></a>Pasos siguientes

+ [Tecnologías de implementación en Azure Functions](functions-deployment-technologies.md) 
+ [Guía para desarrolladores de Azure Functions](functions-reference.md)