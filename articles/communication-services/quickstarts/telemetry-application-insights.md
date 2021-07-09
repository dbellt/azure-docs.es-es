---
title: 'Inicio rápido: Exportación de datos de telemetría del SDK a Application Insights'
titleSuffix: An Azure Communication Services quickstart
description: Obtenga información sobre cómo exportar datos de telemetría del SDK de Azure Communication Services a Application Insights.
author: peiliu
manager: vravikumar
services: azure-communication-services
ms.author: peiliu
ms.date: 06/01/2021
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: 8c9468510e68f29816e52e74cb9a311e3834f107
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593135"
---
# <a name="quickstart-using-azure-opentelemetry-exporter-to-export-sdk-telemetry-data-to-application-insights"></a>Inicio rápido: Uso de Azure OpenTelemetry Exporter para exportar datos de telemetría del SDK a Application Insights

Azure OpenTelemetry Exporter es un SDK dentro de [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/). Permite exportar datos de seguimiento mediante OpenTelemetry y enviar los datos a [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview). OpenTelemetry ofrece una manera estandarizada para que las aplicaciones y los marcos recopilen información de telemetría.

Azure Application Insights es una característica de Azure Monitor que se usa para supervisar aplicaciones en directo. Muestra los datos de la aplicación sobre la aplicación en un recurso de Microsoft Azure. El modelo de telemetría está normalizado para que sea posible crear una supervisión de independiente del lenguaje y de la plataforma.

::: zone pivot="programming-language-csharp"
[!INCLUDE [.NET](./includes/telemetry-app-insights-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/telemetry-app-insights-js.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/telemetry-app-insights-java.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/telemetry-app-insights-python.md)]
::: zone-end

La salida de la aplicación describe cada acción que se completa:
<!---cSpell:disable --->
```console
Created an identity with ID: <identity-id>
Issued an access token with 'chat' scope that expires at <expiry-data>
```
<!---cSpell:enable --->

## <a name="view-the-telemetry-data-in-application-insights"></a>Vista de los datos de telemetría en Application Insights
Para analizar los datos de telemetría del SDK, vaya a la pestaña `Performance` y, a continuación, vaya a `Dependencies`. Podrá ver el `Create User Activity` y `Get Token Activity` a los que hemos hecho un seguimiento.

:::image type="content" source="media/application-insights-dependencies.png" alt-text="Captura de pantalla que muestra las entradas de datos de telemetría en Application Insights.":::

Para ver más detalles, puede profundizar en los ejemplos:

:::image type="content" source="media/application-insights-samples-drill-down.png" alt-text="Captura de pantalla que muestra la vista de exploración en profundidad de los ejemplos":::

En la vista de exploración en profundidad, hay más información sobre la actividad, por ejemplo: desde dónde se llamó, su marca de tiempo, el nombre, el rendimiento, el tipo, etc. También puede ver el nombre del rol en la nube y el identificador de la instancia que definimos en el fragmento de código de ejemplo anterior. Observe que también aparecen aquí las propiedades personalizadas de las que se ha hecho un seguimiento:

:::image type="content" source="media/application-insights-e2e-details.png" alt-text="Vista integral de los detalles de la transacción":::

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a:

> [!div class="checklist"]
> * Configuración del exportador de telemetría
> * Incorporación de los datos de telemetría en Application Insights
> * Visualización de los datos exportados en Application Insights

Puede que también le interese:

- [Más información sobre el análisis de datos en Application Insights](https://docs.microsoft.com/powerapps/maker/canvas-apps/application-insights)
