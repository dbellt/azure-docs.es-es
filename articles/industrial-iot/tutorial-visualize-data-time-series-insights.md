---
title: Visualización de datos de OPC UA en Azure Time Series Insights
description: En este tutorial, aprenderá a visualizar datos con Time Series Insights.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: tutorial
ms.date: 3/22/2021
ms.openlocfilehash: 9a07f2b1d4b63005bff4a8963bd1258ad72e87ee
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108137281"
---
# <a name="tutorial-visualize-data-with-time-series-insights-tsi"></a>Tutorial: Visualización de datos con Time Series Insights (TSI)

El módulo OPC Publisher se conecta a los servidores de OPC UA y publica datos de estos servidores en IoT Hub. El procesador de telemetría de la plataforma IoT industrial procesa estos ejemplos y reenvía ejemplos contextualizados tanto a TSI como a otros consumidores.  

En esta guía paso a paso se muestra cómo visualizar y analizar los datos de telemetría de OPC UA mediante este entorno de Time Series Insights.

En este tutorial, aprenderá que:

> [!div class="checklist"]
> * Todos los tutoriales incluyen una lista que resume los pasos necesarios para completarlos
> * Cada uno de estos puntos se alinea con una clave H2
> * Usar estas casillas verdes en un tutorial

## <a name="prerequisite"></a>Requisito previo

* Implementar la plataforma de IIoT para que se cree automáticamente un entorno de Time Series Insights
* Los datos se publican en IoT Hub

## <a name="time-series-insights-explorer"></a>Explorador de Time Series Insights

El explorador de Time Series Insights es una aplicación web que puede usar para visualizar los datos de telemetría. Para recuperar la dirección URL de la aplicación, abra el archivo `.env` guardado como resultado de la implementación.  Abra un explorador en la dirección URL de la variable `PCS_TSI_URL`.  

Antes de usar el explorador de Time Series Insights, debe conceder acceso a los datos de Time Series Insights a los usuarios autorizados para visualizarlos. Tenga en cuenta que en una implementación nueva no se establecen directivas de acceso a datos de forma predeterminada, por lo que nadie puede ver los datos. Las directivas de acceso a datos deben establecerse en Azure Portal, en el entorno de Time Series Insights implementado en el grupo de recursos implementado de la plataforma de IIoT, como se indica a continuación:

   ![Explorador de Time Series Insights 1](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-data-access-1.png)

Seleccione Directivas de acceso a datos:

   ![Explorador de Time Series Insights 2](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-data-access-2.png)

Asigne los usuarios necesarios:

   ![Explorador de Time Series Insights 3](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-data-access-3.png)


En el explorador de Time Series Insights, tenga en cuenta las instancias de Time Series sin asignar. Una instancia de Time Series Insights corresponde a la serie de tiempo y valor para un punto de datos específico cuyo origen es un nodo publicado en un servidor OPC. La instancia de Time Series Insights, respectivamente el punto de datos de OPC UA, se identifica de forma única mediante EndpointId, SubscriptionId y NodeId. Los modelos de instancias de Time Series Insights se detectan y muestran automáticamente en el explorador en función de los datos de telemetría ingeridos en el centro de eventos del procesador de telemetría de la plataforma de IIoT.

   ![Explorador de Time Series Insights 4](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-step-0.png)

Para visualizar los datos de telemetría en el gráfico, haga clic con el botón derecho en la instancia de Time Series Insights y seleccione el valor. El periodo de tiempo que se va a usar en el gráfico se puede ajustar desde la esquina superior derecha. El valor de varias instancias se puede visualizar en la misma selección por tiempo.

Para más información, consulte [Inicio rápido: Exploración del entorno de demo de Azure Time Series Insights Gen2](../time-series-insights/quickstart-explore-tsi.md)

## <a name="define-and-apply-a-new-model"></a>Definición y aplicación de un modelo nuevo

Dado que las instancias de telemetría se encuentran ahora en formato sin procesar, deben configurarse con el formato correspondiente 

Para obtener información detallada sobre los modelos de Time Series Insights, consulte [Modelo de serie temporal en la versión preliminar de Azure Time Series Insights](../time-series-insights/concepts-model-overview.md)

1. Paso 1: En la pestaña Modelo del explorador, defina una nueva jerarquía para los datos de telemetría ingeridos. Una jerarquía es la estructura de árbol lógico diseñada para permitir al usuario insertar la metainformación necesaria para una navegación más intuitiva a través de las instancias del Time Series Insights. Un usuario puede crear, eliminar o modificar plantillas de jerarquía que se pueden usar más adelante para las distintas instancias de Time Series Insights.

   ![Paso 1](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-step-1.png)

2. Paso 2: Defina un nuevo tipo para los valores. En nuestro ejemplo, solo se controlan los tipos de datos numéricos.

   ![Paso 2](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-step-2.png)

3. Paso 3: Seleccione de la nueva instancia de Time Series Insights que se debe clasificar en la jerarquía definida previamente.

   ![Paso 3](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-step-3.png)

4. Paso 4: Rellene las propiedades de las instancias: nombre, descripción, valor de los datos, así como los campos de jerarquía para que coincidan con la estructura lógica. 

   ![Paso 4](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-step-4.png)

5. Paso 5: Repita el paso 5 para todas las instancias de Time Series Insights sin categoría.

   ![Paso 5](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-step-5.png)

6. Paso 6: De vuelta en la página principal del explorador de Time Series Insights, recorra la jerarquía de instancias clasificadas y seleccione los valores de los puntos de datos que se van a analizar.

   ![Paso 6](media/tutorial-iiot-visualize-data-tsi/tutorial-time-series-insights-step-6.png)

## <a name="connect-time-series-insights-to-power-bi"></a>Conexión Time Series Insights con Power BI

También puede conectar el entorno de Time Series Insights a Power BI.  Para más información, consulte los artículos en los que se explica [cómo conectar Time Series Insights a Power BI](../time-series-insights/how-to-connect-power-bi.md) y la [visualización de datos de Time Series Insights en Power BI](../time-series-insights/concepts-power-bi.md).


## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a visualizar datos en Time Series Insights, puede consultar el repositorio de GitHub de IoT industrial:

> [!div class="nextstepaction"]
> [Repositorio de GitHub de la plataforma de IIoT](https://github.com/Azure/iot-edge-opc-publisher)