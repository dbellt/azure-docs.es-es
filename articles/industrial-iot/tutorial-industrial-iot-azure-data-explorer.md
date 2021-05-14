---
title: Extracción de datos de Azure Industrial IoT en ADX
description: En este tutorial, aprenderá a extraer datos de IIoT en ADX.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: tutorial
ms.date: 3/22/2021
ms.openlocfilehash: 9a18609ab0dc41031a22db6d8b21c1fff83920f2
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108137317"
---
# <a name="tutorial-pull-azure-industrial-iot-data-into-adx"></a>Tutorial: Extracción de datos de Azure Industrial IoT en ADX

La plataforma de Azure Industrial IoT (IIoT) combina módulos perimetrales y microservicios en la nube con varios servicios de PaaS de Azure para proporcionar funcionalidades para la detección de recursos industriales y la recopilación de datos de estos recursos mediante OPC UA. [Azure Data Explorer (ADX)](/azure/data-explorer) es un destino natural para los datos de IIoT con características de análisis de datos que permiten ejecutar consultas flexibles en los datos ingeridos de los servidores de OPC UA conectados a IoT Hub mediante OPC Publisher. Aunque un clúster de Azure Data Explorer puede ingerir datos directamente de IoT Hub, la plataforma de IIoT realiza un procesamiento adicional de los datos para que sean más útiles antes de colocarlos en el centro de eventos que se proporciona cuando se usa una implementación completa de los microservicios (consulte la arquitectura de la plataforma de IIoT).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una tabla en Azure Data Explorer
> * Conectar el centro de eventos al clúster de Azure Data Explorer
> * Analizar los datos en Azure Data Explorer

## <a name="how-to-make-the-data-available-in-the-adx-cluster-to-query-it-effectively"></a>Cómo hacer que los datos estén disponibles en el clúster de Azure Data Explorer para consultarlos de forma eficaz 

Si observamos el formato del mensaje del centro de eventos (tal y como se define en la clase Microsoft.Azure.IIoT.OpcUa.Subscriber.Models.MonitoredItemMessageModel), podemos ver una sugerencia sobre la estructura que necesitamos para el esquema de tabla de Azure Data Explorer.

![Estructura](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-1.png)

A continuación se indican los pasos necesarios para hacer que los datos estén disponibles en el clúster de Azure Data Explorer y poder consultar los datos de forma eficaz.  
1. Cree un clúster de Azure Data Explorer. Si no tiene aún un clúster de Azure Data Explorer aprovisionado con la plataforma de IIoT, o si desea usar otro clúster, siga los pasos que se indican [aquí](/azure/data-explorer/create-cluster-database-portal#create-a-cluster). 
2. Habilite la ingesta de streaming en el clúster de Azure Data Explorer como se indica [aquí](/azure/data-explorer/ingest-data-streaming#enable-streaming-ingestion-on-your-cluster). 
3. Cree una base de datos de Azure Data Explorer siguiendo los pasos que se indican [aquí](/azure/data-explorer/create-cluster-database-portal#create-a-database).

En el paso siguiente, se usará la [interfaz web de Azure Data Explorer](/azure/data-explorer/web-query-data) para ejecutar las consultas necesarias. Asegúrese de agregar el clúster a la interfaz web tal como se explica en el vínculo.  
 
4. Cree una tabla de Azure Data Explorer en la que colocar los datos ingeridos.  Aunque la clase MonitoredItemMessageModel se puede usar para definir el esquema de la tabla de Azure Data Explorer, se recomienda ingerir los datos primero en una tabla de almacenamiento provisional con una columna de tipo [Dinámico](/azure/data-explorer/kusto/query/scalar-data-types/dynamic). Esto nos proporciona más flexibilidad a la hora de administrar los datos y de procesarlos en otras tablas (posiblemente combinándolos con otros orígenes de datos) que satisfacen las necesidades de varios casos de uso. La siguiente consulta de Azure Data Explorer crea la tabla de almacenamiento provisional "iiot_stage" con una columna "payload".

```
.create table ['iiot_stage']  (['payload']:dynamic)
```

También es necesario agregar una asignación de ingesta de JSON para indicar al clúster que coloque todo el mensaje JSON del centro de eventos en la tabla de almacenamiento provisional.

```
.create table ['iiot_stage'] ingestion json mapping 'iiot_stage_mapping' '[{"column":"payload","path":"$","datatype":"dynamic"}]'
```

5. Nuestra tabla ya está lista para recibir datos del centro de eventos. 
6. Siga las instrucciones que se indican [aquí](/azure/data-explorer/ingest-data-event-hub#connect-to-the-event-hub) para conectar el centro de eventos al clúster de Azure Data Explorer y comenzar a ingerir los datos en nuestra tabla de almacenamiento provisional. Solo es necesario crear la conexión, ya que ya tenemos un centro de eventos aprovisionado por la plataforma de IIoT.  
7. Una vez que se comprueba la conexión, los datos comenzarán a fluir hasta la tabla y, después de un breve intervalo, se podrán empezar a examinar los datos de la tabla. Use la siguiente consulta de la interfaz web de Azure Data Explorer para ver una muestra de datos de 10 filas. Aquí se puede ver cómo los datos de la carga se asemejan a la clase MonitoredItemMessageModel mencionada anteriormente.

![Consultar](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-2.png)

8. Ahora vamos a ejecutar algunos análisis sobre estos datos mediante el análisis directo de los datos dinámicos de la columna "carga". En este ejemplo, se calcula el promedio de los datos de telemetría identificados mediante "DisplayName": "PositiveTrendData", a lo largo de períodos de tiempo de 10 minutos, en todos los registros ingeridos desde un punto temporal específico (definido por la variable min_t) let min_t = datetime(2020-10-23); iiot_stage | donde todatetime(payload.Timestamp) > min_t | donde tostring(payload.DisplayName)== 'PositiveTrendData' | summarize event_avg = avg(todouble(payload.Value)) por bin(todatetime(payload.Timestamp), 10 m)
 
Dado que nuestra columna "carga" contiene un tipo de datos dinámico, es necesario realizar la conversión de los datos en el momento de la consulta para que los cálculos se realicen en los tipos de datos correctos.

![Marca de tiempo de carga](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-3.png)

Como se mencionó anteriormente, la ingesta de datos de OPC UA en una tabla de almacenamiento provisional con una columna "dinámica" nos proporciona flexibilidad. Sin embargo, tener que ejecutar las conversiones de tipos de datos en el momento de la consulta puede producir retrasos en la ejecución de las consultas especialmente si el volumen de datos es grande y si hay muchas consultas simultáneas. En esta fase, se puede crear otra tabla con los tipos de datos ya determinados, de modo que se eviten las conversiones de tipos de datos en el momento de la consulta.
 
9. Cree una nueva tabla para los datos analizados que se componga de una selección limitada del contenido de la "carga" dinámica en la tabla de almacenamiento provisional. Tenga en cuenta que hemos creado una columna de valor para cada uno de los tipos de datos previsibles que se pueden esperar en la telemetría.

```
.create table ['iiot_parsed']  
    (['Tag_Timestamp']: datetime ,  
    ['Tag_PublisherId']:string ,  
    ['Tag']:string ,
    ['Tag_Datatype']:string ,  
    ['Tag_NodeId']:string,  
    ['Tag_value_long']:long ,  
    ['Tag_value_double']:double,  
    ['Tag_value_boolean']:bool)
```

10. Cree una función (en el nivel de base de datos) para proyectar los datos necesarios de la tabla de almacenamiento provisional. Aquí se seleccionan los elementos "Timestamp", "PublisherId", "DisplayName", "Datatype" y "NodeId" de la columna "carga" y se proyectan como "Tag_Timestamp", "Tag_PublisherId", "Tag", "Tag_Datatype", "Tag_NodeId". El elemento "Valor" se proyecta como tres partes diferentes basadas en "DataType".

```
.create-or-alter function fn_InflightParseIIoTEvent()
{
iiot_stage
| extend Tag_Timestamp =  todatetime(payload.Timestamp)
| extend Tag_PublisherId = tostring(payload.PublisherId)
| extend Tag = tostring(payload.DisplayName)
| extend Tag_Datatype = tostring(payload.DataType)
| extend Tag_NodeId = tostring(payload.NodeId)
| extend Tag_value_long = case(Tag_Datatype == "Int64", tolong(payload.Value), long(null))
| extend Tag_value_double = case(Tag_Datatype == "Double", todouble(payload.Value), double(null))
| extend Tag_value_boolean = case(Tag_Datatype == "Boolean", tobool(payload.Value), bool(null))
| project Tag_Timestamp, Tag_PublisherId, Tag, Tag_Datatype, Tag_NodeId, Tag_value_long, Tag_value_double, Tag_value_boolean
}
```

Para más información sobre la asignación de tipos de datos en Azure Data Explorer, consulte [esta información](/azure/data-explorer/kusto/query/scalar-data-types/dynamic) y para ver las funciones de Azure Data Explorer puede empezar por [aquí](/azure/data-explorer/kusto/query/schema-entities/stored-functions).
 
11. Aplique la función del paso anterior a la tabla analizada mediante una directiva de actualización. La [directiva](/azure/data-explorer/kusto/management/updatepolicy) de actualización indica a Azure Data Explorer que anexe datos automáticamente a una tabla de destino cada vez que se inserten nuevos datos en la tabla de origen, en función de una consulta de transformación que se ejecuta en los datos insertados en la tabla de origen. Se puede usar la siguiente consulta para asignar la tabla analizada como destino y la tabla de almacenamiento provisional como origen de la directiva de actualización definida por la función que se creó en el paso anterior.

```
.alter table iiot_parsed policy update
@'[{"IsEnabled": true, "Source": "iiot_stage", "Query": "fn_InflightParseIIoTEvent()", "IsTransactional": true, "PropagateIngestionProperties": true}]'
```

En cuanto se ejecute la consulta anterior, los datos comenzarán a fluir y rellenarán la tabla de destino "iiot_parsed". Podemos ver los datos de "iiot_parsed" como se indica a continuación.

![Tabla analizada](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-4.png)

12. Echemos un vistazo a cómo podemos repetir el análisis que hicimos en un paso anterior. Calcule el promedio de los datos telemetría identificados mediante "DisplayName": "PositiveTrendData", a lo largo de períodos de tiempo de 10 minutos, en todos los registros ingeridos desde un punto temporal específico (definido por la variable min_t). Como ahora tenemos los valores de la etiqueta "PositiveTrendData" almacenados en una columna con el tipo de datos Double, es previsible una mejora en el rendimiento de las consultas.

![Repetición de análisis](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-5.png)

13. Por último, vamos a comparar el rendimiento de las consultas en ambos casos. Podemos ver el tiempo que se ha tardado en ejecutar la consulta mediante "Stats" en la interfaz de usuario de Azure Data Explorer (que se encuentra encima de los resultados de la consulta).  

![Rendimiento de consultas 1](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-6.png)

![Rendimiento de consultas 2](media/tutorial-iiot-data-adx/industrial-iot-in-azure-data-explorer-pic-7.png)

Podemos ver que la consulta que utiliza la tabla analizada es aproximadamente dos veces más rápida que la de la tabla de almacenamiento provisional. En este ejemplo, tenemos un conjunto de datos pequeño y no hay ninguna consulta simultánea en ejecución, por lo que el efecto en el tiempo de ejecución de la consulta no es muy grande, pero para una carga de trabajo real supondría un gran impacto en el rendimiento. Este es el motivo por el que es importante considerar la posibilidad de separar los distintos tipos de datos en diferentes columnas.

> [!NOTE] 
> La directiva de actualización solo funciona en los datos que se ingieren en la tabla de almacenamiento provisional después de que la directiva se haya configurado y no se aplica a los datos ya existentes. Esto se debe tener en cuenta, por ejemplo, a la hora de cambiar la directiva de actualización. Se puede encontrar más información en la documentación de Azure Data Explorer.

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a cambiar los valores predeterminados de la configuración, puede 

> [!div class="nextstepaction"]
> [Configurar componentes de Industrial IoT](tutorial-configure-industrial-iot-components.md)

> [!div class="nextstepaction"]
> [Visualizar y analizar los datos mediante Time Series Insights](tutorial-visualize-data-time-series-insights.md)