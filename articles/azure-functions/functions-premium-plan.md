---
title: Plan prémium de Azure Functions
description: Detalles y opciones de configuración (red virtual, arranques no en frío, duración de ejecución ilimitada, etc) del plan Premium de Azure Functions.
author: cachai2
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: cachai
ms.custom:
- references_regions
- fasttrack-edit
- devx-track-azurecli
ms.openlocfilehash: e53089c2287d255fd3dbe6ca7961646e0e3de05a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110457430"
---
# <a name="azure-functions-premium-plan"></a>Plan prémium de Azure Functions

El plan Prémium de Azure Functions (a veces conocido como plan Elástico Prémium) es una opción de hospedaje de las aplicaciones de funciones. Para ver otras opciones de plan de hospedaje, consulte [este artículo](functions-scale.md).

El hospedaje del plan Prémium ofrece las siguientes ventajas para sus funciones:

* Evita los arranques en frío con instancias activas continuamente.
* Conectividad de red virtual.
* Duración de ejecución ilimitada, con 60 minutos garantizados.
* Tamaños de instancia prémium (instancias de un núcleo, dos núcleos y cuatro núcleos).
* Precios más predecibles, en comparación con el plan de consumo.
* Asignación de aplicaciones de alta densidad para planes con varias aplicaciones de funciones.

Cuando se usa el plan Prémium, las instancias del host de Azure Functions se agregan y quitan según el número de eventos entrantes, al igual que con el [plan de consumo](consumption-plan.md). Pueden implementarse varias aplicaciones de funciones en el mismo plan Premium. Este plan permite configurar el tamaño de la instancia de proceso, el tamaño del plan base y el tamaño del plan máximo. 

## <a name="billing"></a>Facturación

La facturación del plan Prémium se basa en el número de núcleos por segundo y en la memoria asignada entre instancias. Esta facturación difiere de la del plan de consumo, que se factura por ejecución y consumo de memoria. No hay ningún cargo de ejecución con el plan Premium. Debe haber al menos una instancia asignada en todo momento en cada plan. Esta facturación da como resultado un costo mensual mínimo por plan activo, independientemente de si la función está activa o inactiva. Tenga en cuenta que todas las aplicaciones de función de un plan Premium comparten instancias asignadas. Para más información, consulte la [página de precios de Azure Functions](https://azure.microsoft.com/pricing/details/functions/).

## <a name="create-a-premium-plan"></a>Creación de un plan Premium

Al crear una aplicación de funciones en Azure Portal, el plan de consumo es el predeterminado. Para crear una aplicación de función que se ejecute en un plan Prémium, debe crear explícitamente un plan de App Service con una de las SKU de _Elástico Prémium_. La aplicación de función que cree se hospeda en este plan. Azure Portal permite crear fácilmente el plan Prémium y la aplicación de funciones al mismo tiempo. Puede ejecutar más de una aplicación de funciones en el mismo plan Premium, pero deben ejecutarse en el mismo sistema operativo (Windows o Linux). 

En los artículos siguientes se muestra cómo crear una aplicación de funciones con un plan Prémium, ya sea mediante programación o en Azure Portal:

+ [Azure Portal](create-premium-plan-function-app-portal.md)
+ [CLI de Azure](scripts/functions-cli-create-premium-plan.md)
+ [Plantilla de Azure Resource Manager](functions-infrastructure-as-code.md#deploy-on-premium-plan)

## <a name="eliminate-cold-starts"></a>Eliminación de los arranques en frío

Cuando no se producen eventos ni ejecuciones en el plan de consumo, la aplicación puede escalarse a cero instancias. Cuando se produzcan nuevos eventos, será necesario especializar una nueva instancia con la aplicación que se ejecute en ella. La especialización de nuevas instancias puede tardar algún tiempo dependiendo de la aplicación. Esta latencia adicional de la primera llamada también suele denominarse _arranque en frío_ de la aplicación.

El plan Prémium proporciona dos características que funcionan juntas para eliminar de forma eficaz los arranques en frío en las funciones: _instancias siempre preparadas_ e _instancias activadas previamente_. 

### <a name="always-ready-instances"></a>Instancias siempre preparadas

En el plan Premium, puede hacer que la aplicación previamente activada esté siempre preparada en un número concreto de instancias. El número máximo de instancias siempre preparadas es 20. Cuando los eventos empiezan a desencadenar la aplicación, siempre se enrutan primero a las instancias siempre preparadas. A medida que la función se activa, las instancias adicionales se activarán como búferes. Este búfer evita que las nuevas instancias necesarias durante el escalado se arranquen en frío. Estas instancias almacenadas en búfer se denominan [instancias activadas previamente](#pre-warmed-instances). Con la combinación de las instancias siempre preparadas y un búfer activado previamente, la aplicación puede eliminar eficazmente los arranques en frío.

> [!NOTE]
> Todos los planes Prémium tendrán al menos una instancia activa (facturada) en todo momento.

# <a name="portal"></a>[Portal](#tab/portal)

Puede configurar el número de instancias siempre preparadas en Azure Portal. Para ello, seleccione una aplicación de funciones en **Function App**, vaya a la pestaña **Características de la plataforma** y seleccione las opciones para **Escalar horizontalmente**. En la ventana de edición de la aplicación de funciones, las instancias siempre preparadas son específicas para esa aplicación.

![Configuración del escalado elástico](./media/functions-premium-plan/scale-out.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

Las instancias siempre preparadas para una aplicación también se pueden configurar con la CLI de Azure.

```azurecli-interactive
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.minimumElasticInstanceCount=<desired_always_ready_count> --resource-type Microsoft.Web/sites
```
---

### <a name="pre-warmed-instances"></a>Instancias activadas previamente

Las instancias activadas previamente son las instancias que se han activado como búferes durante los eventos de escalado y activación. Las instancias activadas previamente siguen almacenándose en el búfer hasta que se alcanza el límite máximo de escalabilidad horizontal. El número predeterminado de instancias activadas previamente es 1 y, con la mayoría de los escenarios, este valor debería dejarse en 1.

Si una aplicación tiene un período de activación largo (como una imagen de contenedor personalizada), puede que deba aumentar este búfer. Una instancia activada previamente solo se activa después de que todas las instancias activas se hayan usado lo suficiente.

Tenga en cuenta este ejemplo que muestra cómo funcionan juntas las instancias siempre preparadas y las instancias activadas previamente. Una aplicación de funciones Premium tiene configuradas cinco instancias siempre preparadas, y una instancia activada previamente como predeterminada. Cuando la aplicación está inactiva y no se desencadena ningún evento, se aprovisiona y se ejecuta con cinco instancias. En este momento, no se le facturan las instancias activadas previamente, porque las instancias siempre preparadas no se usan y no hay ninguna instancia activada previamente asignada.

En cuanto se inicia el primer desencadenador, las cinco instancias siempre preparadas se activan y se asigna una instancia activada previamente. La aplicación se está ejecutando ahora con seis instancias aprovisionadas: las cinco instancias siempre preparadas y la sexta instancia de búfer inactiva y activada previamente. Si la tasa de ejecuciones sigue aumentando, al final se usarán las cinco instancias activas. Cuando la plataforma decide escalarse más allá de cinco instancias, lo hace a la instancia activada previamente. Cuando esto sucede, seis instancias están ahora activas y una séptima se aprovisiona al instante, con lo que se rellena el búfer de instancias activadas previamente. Esta secuencia de escalado y activación previa continúa hasta que se alcanza el recuento de instancias máximo de la aplicación. Ninguna instancia se activará previamente ni se activará por encima del número máximo.

Puede modificar el número de instancias activadas previamente para una aplicación mediante la CLI de Azure.

```azurecli-interactive
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.preWarmedInstanceCount=<desired_prewarmed_count> --resource-type Microsoft.Web/sites
```

### <a name="maximum-function-app-instances"></a>Número máximo de instancias de aplicación de funciones

Además del [recuento de instancias máximo de un plan](#plan-and-sku-settings), puede configurar un máximo por aplicación. El máximo de la aplicación se puede configurar mediante el [límite de escalabilidad de aplicaciones](./event-driven-scaling.md#limit-scale-out).

## <a name="private-network-connectivity"></a>Conectividad de red privada

Las aplicaciones de funciones implementadas en un plan Prémium pueden aprovechar las ventajas de la [integración de red virtual para aplicaciones web](../app-service/web-sites-integrate-with-vnet.md). Cuando se configura, la aplicación puede comunicarse con los recursos de la red virtual o protegerse mediante puntos de conexión de servicio. Las restricciones de IP también están disponibles en la aplicación para restringir el tráfico entrante.

Cuando se asigne una subred a la aplicación de funciones en un plan Premium, necesitará una subred con suficientes direcciones IP para cada posible instancia. Se requiere un bloque de direcciones IP con al menos 100 direcciones disponibles.

Para más información, consulte [Integración de una aplicación de funciones con una red virtual](functions-create-vnet.md).

## <a name="rapid-elastic-scale"></a>Escalado elástico rápido

Otras instancias de proceso se agregan automáticamente a la aplicación utilizando la misma lógica de escalado rápido que el plan de consumo. Las aplicaciones dentro del mismo plan de App Service se escalan de forma independiente entre sí y en función de las necesidades de una aplicación individual, pero las aplicaciones de Functions dentro del mismo plan de App Service sí comparten recursos de máquina virtual para contribuir a reducir los costes, siempre que sea posible. El número de aplicaciones asociadas a una máquina virtual depende de la superficie de cada aplicación y del tamaño de la máquina virtual en cuestión.

Para más información sobre cómo funciona el escalado, consulte [Escalado basado en eventos en Azure Functions](event-driven-scaling.md).

## <a name="longer-run-duration"></a>Duración de la ejecución más larga

Azure Functions en un plan de consumo que impone un límite de 10 minutos en cada ejecución. En el plan Premium, la duración de ejecución predeterminada es de 30 minutos para evitar ejecuciones descontroladas. Sin embargo, puede [modificar la configuración de host.json](./functions-host-json.md#functiontimeout) para que la duración sea ilimitada en las aplicaciones del plan Prémium. Cuando se establece en una duración ilimitada, se garantiza que la aplicación de funciones se ejecuta durante al menos 60 minutos. 

## <a name="plan-and-sku-settings"></a>Configuración del plan y la SKU

Cuando se crea un plan, hay dos configuraciones de tamaño de plan: el número mínimo de instancias (o tamaño de plan) y el límite máximo de ráfaga.

Si la aplicación necesita un número de instancias mayor al de las instancias siempre preparadas, se puede seguir escalando horizontalmente hasta que el número de instancias alcance el límite máximo de ráfaga. Las instancias que superen el tamaño del plan solo se le facturan (por segundo) cuando estén en ejecución y las tenga asignadas. La plataforma hace todo lo posible por escalar la aplicación hasta el límite máximo definido.

Puede configurar el tamaño del plan y establecer valores máximos en Azure Portal seleccionando las opciones **Escalar horizontalmente** en el plan o una aplicación de funciones implementada en el plan (en **Características de la plataforma**).

También puede aumentar el límite máximo de ráfaga mediante la CLI de Azure:

```azurecli-interactive
az functionapp plan update -g <resource_group> -n <premium_plan_name> --max-burst <desired_max_burst>
```

El mínimo de cada plan será al menos una instancia. El número mínimo real de instancias se configurará automáticamente en función de las instancias siempre preparadas que hayan solicitado las aplicaciones del plan. Por ejemplo, si la aplicación A solicita cinco instancias siempre preparadas y la aplicación B solicita dos instancias de este tipo en el mismo plan, el tamaño mínimo del plan se calculará como cinco. La aplicación A se ejecutará en las 5 y la aplicación B solo se ejecutará en 2.

> [!IMPORTANT]
> Se le cobrará por cada instancia especificada en el número mínimo de instancias independientemente de si las funciones se ejecutan o no.

En la mayoría de los casos, este mínimo que se calculó automáticamente es suficiente. Sin embargo, el escalado que supere el mínimo se realiza de la mejor manera posible. Es posible, aunque poco probable, que se retrase el escalado horizontal en un momento específico si no existen otras instancias. Al establecer un valor mínimo superior al mínimo calculado automáticamente, se reservan instancias antes del escalado horizontal.

El aumento del mínimo calculado para un plan se puede realizar mediante la CLI de Azure.

```azurecli-interactive
az functionapp plan update -g <resource_group> -n <premium_plan_name> --min-instances <desired_min_instances>
```

### <a name="available-instance-skus"></a>SKU de instancias disponibles

Cuando cree o escale un plan, podrá elegir entre tres tamaños de instancia. Se le facturará la cantidad total de núcleos y memoria aprovisionada, por los segundos que tenga asignada cada instancia. La aplicación puede escalar horizontalmente de forma automática en varias instancias cuando sea necesario.

|SKU|Núcleos|Memoria|Storage|
|--|--|--|--|
|EP1|1|3,5 GB|250 GB|
|EP2|2|7 GB|250 GB|
|EP3|4|14 GB|250 GB|

### <a name="memory-usage-considerations"></a>Consideraciones de uso de memoria

La ejecución en una máquina con más memoria no siempre significa que la aplicación de funciones vaya a usar toda la memoria disponible.

Por ejemplo, una aplicación de funciones de JavaScript está restringida por el límite de memoria predeterminado en Node.js. Para aumentar este límite de memoria fijo, agregue la configuración de la aplicación `languageWorkers:node:arguments` con un valor de `--max-old-space-size=<max memory in MB>`.

Además, en el caso de los planes con más de 4 GB de memoria, asegúrese de que la configuración de la plataforma de valor de bits esté establecida en `64 Bit` en [Configuración general](../app-service/configure-common.md#configure-general-settings).

## <a name="region-max-scale-out"></a>Escalabilidad horizontal máxima en regiones

A continuación se muestran los valores máximos de escalabilidad horizontal actualmente admitidos para un solo plan en cada región y configuración del sistema operativo. Para solicitar un aumento, abra una incidencia de soporte técnico.

Consulte la disponibilidad regional completa de Functions en el [sitio web de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=functions).

|Region| Windows | Linux |
|--| -- | -- |
|Centro de Australia| 100 | No disponible |
|Centro de Australia 2| 100 | No disponible |
|Este de Australia| 100 | 20 |
|Sudeste de Australia | 100 | 20 |
|Sur de Brasil| 100 | 20 |
|Centro de Canadá| 100 | 20 |
|Centro de EE. UU.| 100 | 20 |
|Este de China 2| 100 | 20 |
|Norte de China 2| 100 | 20 |
|Este de Asia| 100 | 20 |
|Este de EE. UU. | 100 | 20 |
|Este de EE. UU. 2| 100 | 20 |
|Centro de Francia| 100 | 20 |
|Centro-oeste de Alemania| 100 | No disponible |
|Japón Oriental| 100 | 20 |
|Japón Occidental| 100 | 20 |
|Centro de Corea del Sur| 100 | 20 |
|Corea del Sur| No disponible | 20 |
|Centro-Norte de EE. UU| 100 | 20 |
|Norte de Europa| 100 | 20 |
|Este de Noruega| 100 | 20 |
|Centro-sur de EE. UU.| 100 | 20 |
|Sur de la India | 100 | No disponible |
|Sudeste de Asia| 100 | 20 |
|Norte de Suiza| 100 | No disponible |
|Oeste de Suiza| 100 | No disponible |
|Sur de Reino Unido| 100 | 20 |
|Oeste de Reino Unido| 100 | 20 |
|USGov: Arizona| 100 | 20 |
|USGov Virginia| 100 | 20 |
|USNat East| 100 | No disponible |
|USNat West| 100 | No disponible |
|Oeste de Europa| 100 | 20 |
|Oeste de la India| 100 | 20 |
|Centro-Oeste de EE. UU.| 100 | 20 |
|Oeste de EE. UU.| 100 | 20 |
|Oeste de EE. UU. 2| 100 | 20 |

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Descripción de las opciones de hospedaje de Azure Functions](functions-scale.md)
