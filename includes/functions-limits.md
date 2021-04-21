---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/04/2020
ms.author: glenga
ms.openlocfilehash: 91f72117fdbcdbeda1d906a9760243e66404920c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105105798"
---
| Recurso |[Plan de consumo](../articles/azure-functions/consumption-plan.md)|[Plan Premium](../articles/azure-functions/functions-premium-plan.md)|[Plan dedicado](../articles/azure-functions/dedicated-plan.md)|[Azure App Service Environment](../articles/app-service/environment/intro.md)| [Kubernetes](../articles/aks/quotas-skus-regions.md) |
| --- | --- | --- | --- | --- | --- |
|[Duración de tiempo de espera](../articles/azure-functions/functions-scale.md#timeout) predeterminada (min) |5 | 30 |30<sup>1</sup> | 30 | 30 |
|[Duración máxima de tiempo de espera](../articles/azure-functions/functions-scale.md#timeout) (min) |10 | sin enlazar<sup>7</sup> | sin enlazar<sup>2</sup> | sin enlazar | sin enlazar |
| Número máximo de conexiones salientes (por instancia) | 600 activas (1200 en total) | sin enlazar | sin enlazar | sin enlazar | sin enlazar |
| Tamaño máximo de la solicitud (MB)<sup>3</sup> | 100 | 100 | 100 | 100 | Depende del clúster |
| Longitud máxima de la cadena de consulta<sup>3</sup> | 4096 | 4096 | 4096 | 4096 | Depende del clúster |
| Longitud máxima de URL de solicitud<sup>3</sup> | 8192 | 8192 | 8192 | 8192 | Depende del clúster |
|[ACU](../articles/virtual-machines/acu.md) por instancia | 100 | 210-840 | 100-840 | 210-250<sup>8</sup> | [Precios de AKS](https://azure.microsoft.com/pricing/details/container-service/) |
| Memoria máxima (GB por instancia) | 1.5 | 3,5-14 | 1,75-14 | 3,5-14 | Se admite cualquier nodo |
| Número máximo de instancias | 200 | 100<sup>9</sup> | varía según la SKU<sup>10</sup> | 100<sup>10</sup> | Depende del clúster |   
| Aplicaciones de funciones por plan |100 |100 |sin enlazar<sup>4</sup> | sin enlazar | sin enlazar |
| [Planes de App Service](../articles/app-service/overview-hosting-plans.md) | 100 por [región](https://azure.microsoft.com/global-infrastructure/regions/) |100 por grupo de recursos |100 por grupo de recursos | - | - |
| Almacenamiento<sup>5</sup> |5 TB |250 GB |50-1000 GB | 1 TB | N/D |
| Dominios personalizados por aplicación</a> |500<sup>6</sup> |500 |500 | 500 | N/D |
| Compatibilidad con dominio [Compatibilidad con SSL](../articles/app-service/configure-ssl-bindings.md) |conexión SNI SSL sin enlazar incluida | conexiones SNI SSL ilimitadas y 1 conexión SSL de IP incluidas |conexiones SNI SSL ilimitadas y 1 conexión SSL de IP incluidas | conexiones SNI SSL ilimitadas y 1 conexión SSL de IP incluidas | N/D |

<sup>1</sup> De manera predeterminada, el tiempo de expiración del runtime de Functions 1.x en un plan de App Service no está enlazado.  
<sup>2</sup> Requiere que el plan de App Service se establezca en [Always On](../articles/azure-functions/dedicated-plan.md#always-on). Abonar según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/) estándar.  
<sup>3</sup> Estos límites se [establecen en el host](https://github.com/Azure/azure-functions-host/blob/dev/src/WebJobs.Script.WebHost/web.config).  
<sup>4</sup> El número real de aplicaciones de funciones que puede hospedar depende de la actividad de las aplicaciones, el tamaño de las instancias de máquina y la utilización de recursos correspondiente.  
<sup>5</sup> El límite de almacenamiento es el tamaño total del almacenamiento temporal entre todas las aplicaciones en el mismo plan de App Service. El plan de consumo usa Azure Files para el almacenamiento temporal.  
<sup>6</sup> Cuando la aplicación de funciones se hospeda en un [plan de consumo](../articles/azure-functions/consumption-plan.md), solo se admite solo la opción CNAME. Para aplicaciones de funciones se hospedan en un [plan Premium](../articles/azure-functions/functions-premium-plan.md) o en un [plan de App Service](../articles/azure-functions/dedicated-plan.md), puede asignar un dominio personalizado mediante un CNAME o un registro A.  
<sup>7</sup> Garantizado para un máximo de 60 minutos.  
<sup>8</sup> Los trabajos son roles que hospedan las aplicaciones del cliente. Los trabajos están disponibles en tres tamaños fijos: Una CPU virtual/3,5 GB RAM; dos CPU virtuales/7 GB RAM; cuatro CPU virtuales/14 GB RAM.   
<sup>9</sup> Cuando se trabaja en Linux en un plan Premium, hay un límite de 20 instancias.  
<sup>10</sup> Consulte [Límites de App Service](../articles/azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits) para obtener más información.