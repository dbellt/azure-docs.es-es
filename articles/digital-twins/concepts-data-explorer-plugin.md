---
title: Consultas con Azure Data Explorer
titleSuffix: Azure Digital Twins
description: Descripción del complemento de consulta de Azure Digital Twins para Azure Data Explorer
author: baanders
ms.author: baanders
ms.date: 5/19/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: cb5ef8e90f018cb501b44383fa51bdab58a04b13
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112079092"
---
# <a name="azure-digital-twins-query-plugin-for-azure-data-explorer"></a>Complemento de consulta de Azure Digital Twins para Azure Data Explorer

El complemento de Azure Digital Twins para [Azure Data Explorer (ADX)](/azure/data-explorer/data-explorer-overview) permite ejecutar consultas de ADX que acceden a datos en grafos de Azure Digital Twins y bases de datos de series temporales de ADX y los combinan. Use el complemento para contextualizar datos de series temporales dispares mediante el razonamiento entre los gemelos digitales y sus relaciones para obtener información sobre el comportamiento de los entornos modelados.

Por ejemplo, con este complemento, puede escribir una consulta KQL que...
1. seleccione gemelos digitales de interés mediante el complemento de consulta de Azure Digital Twins,
2. combine esos gemelos con la serie temporal correspondiente de ADX y, a continuación, 
3. realice análisis avanzados de series temporales en esos gemelos.  

La combinación de los datos de un grafo de un gemelo de Azure Digital Twins con datos de series temporales de ADX puede ayudarle a comprender el comportamiento operativo de varios componentes de la solución. 

## <a name="using-the-plugin"></a>Uso del complemento

Para que el complemento se ejecute en su propio clúster de ADX que contiene los datos de la serie temporal, empiece por ejecutar el siguiente comando en ADX para habilitar el complemento:

```kusto
.enable plugin azure_digital_twins_query_request
```

Este comando requiere el permiso de **Administrador de todas las bases de datos**. Para más información sobre el comando, consulte la [documentación del complemento .enable](/azure/data-explorer/kusto/management/enable-plugin). 

Una vez habilitado el complemento, puede invocarlo dentro de una consulta de Kusto en ADX con el siguiente comando. Hay dos marcadores de posición, `<Azure-Digital-Twins-endpoint>` y `<Azure-Digital-Twins-query>`, que son cadenas que representan el punto de conexión de la instancia de Azure Digital Twins y la consulta de Azure Digital Twins, respectivamente. 

```kusto
evaluate azure_digital_twins_query_request(<Azure-Digital-Twins-endpoint>, <Azure-Digital-Twins-query>) 
```

El complemento funciona mediante una llamada a la [API de consulta de Azure Digital Twins](/rest/api/digital-twins/dataplane/query) y la [estructura de lenguaje de la consulta](concepts-query-language.md) es la misma que cuando se usa la API. 

>[!IMPORTANT]
>Al usuario del complemento se le debe conceder el rol **Lector de datos de Azure Digital Twins** o **Propietario de datos de Azure Digital Twins**, ya que el token de Azure AD del usuario se usa para autenticarse. Puede encontrar información sobre cómo asignar este rol en [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md#authorization-azure-roles-for-azure-digital-twins).

Para más información sobre el uso del complemento, consulte la [documentación de Kusto para el complemento azure_digital_twins_query_request](/azure/data-explorer/kusto/query/azure-digital-twins-query-request-plugin).

Para ver consultas de ejemplo y completar un tutorial con datos de ejemplo, consulte [Complemento de consulta de Azure Digital Twins para ADX: Consultas y tutorial de ejemplo](https://github.com/Azure-Samples/azure-digital-twins-getting-started/tree/main/adt-adx-queries) en GitHub.

## <a name="using-adx-iot-data-with-azure-digital-twins"></a>Uso de datos de IoT de ADX con Azure Digital Twins

Hay varias maneras de ingerir datos de IoT en ADX. Estas son dos que puede utilizar al usar ADX con Azure Digital Twins:
* Historiar los valores de propiedad del gemelo digital en ADX con una función de Azure que administre los eventos de cambio y escriba los datos del gemelo digital en ADX, de forma parecida al proceso que se utilizó en [Procedimiento: Integración con Azure Time Series Insights](how-to-integrate-time-series-insights.md). Esta manera es adecuada para los clientes que usan datos de telemetría para que sus gemelos digitales cobren vida.
* [Ingerir los datos de IoT directamente en el clúster de ADX desde IoT Hub](/azure/data-explorer/ingest-data-iot-hub) o desde otros orígenes. Posteriormente, se usará el grafo de Azure Digital Twins para contextualizar los datos de la serie temporal mediante consultas conjuntas de Azure Digital Twins y ADX. Esta manera puede ser adecuada para cargas de trabajo de ingesta directa. 

### <a name="mapping-data-across-adx-and-azure-digital-twins"></a>Asignación de datos en ADX y Azure Digital Twins

Si va a ingerir datos de serie temporal directamente en ADX, es probable que tenga que convertir estos datos de serie temporal sin procesar en un esquema adecuado para consultas conjuntas de Azure Digital Twins y ADX.

Una [directiva de actualización](/azure/data-explorer/kusto/management/updatepolicy) en ADX permite transformar y anexar datos automáticamente a una tabla de destino cada vez que se inserten nuevos datos en una tabla de origen. 

Puede usar una directiva de actualización para enriquecer los datos de serie temporal sin procesar con el **identificador del gemelo** correspondiente de Azure Digital Twins y conservarlo en una tabla de destino. Con el identificador del gemelo, la tabla de destino se puede combinar con los gemelos digitales seleccionados por el complemento de Azure Digital Twins. 

Por ejemplo, supongamos que ha creado la tabla siguiente para que contenga los datos de serie temporal sin procesar que se transfieren a la instancia de ADX. 

```kusto
.createmerge table rawData (Timestamp:datetime, someId:string, Value:string, ValueType:string)  
```

Puede crear una tabla de asignación para relacionar los identificadores de series temporales con los identificadores de los gemelos y otros campos opcionales. 

```kusto
.createmerge table mappingTable (someId:string, twinId:string, otherMetadata:string) 
```

A continuación, cree una tabla de destino que contenga los datos de series temporales enriquecidos. 

```kusto
.createmerge table timeseriesSilver (twinId:string, Timestamp:datetime, someId:string, otherMetadata:string, ValueNumeric:real, ValueString:string)  
```

A continuación, cree una función `Update_rawData` para enriquecer los datos sin procesar combinándolos con la tabla de asignación. Esto agregará el identificador del gemelo a la tabla de destino resultante. 

```kusto
.createoralter function with (folder = "Update", skipvalidation = "true") Update_rawData() { 
rawData 
| join kind=leftouter mappingTable on someId 
| project 
    Timestamp, ValueNumeric = toreal(Value), ValueString = Value, ... 
} 
```

Por último, cree una directiva de actualización para llamar a la función y actualizar la tabla de destino. 

```kusto
.alter table timeseriesSilver policy update 
@'[{"IsEnabled": true, "Source": "rawData", "Query": "Update_rawData()", "IsTransactional": false, "PropagateIngestionProperties": false}]' 
```

Una vez creada la tabla de destino, puede usar el complemento de Azure Digital Twins para seleccionar los gemelos de interés y, a continuación, combinarlos con los datos de series temporales de la tabla de destino. 

### <a name="example-schema"></a>Esquema de ejemplo

Este es un ejemplo de un esquema que podría usarse para representar datos compartidos.

| timestamp | twinId | modelId | name | value | relationshipTarget | relationshipID |
| --- | --- | --- | --- | --- | --- | --- |
| 01-02-2021 17:24 | ConfRoomTempSensor | dtmi:com:example:TemperatureSensor;1 | temperatura | 301,0 |  |  |

Las propiedades de los gemelos digitales se almacenan como pares clave-valor (`name, value`). `name` y `value` se almacenan como tipos de datos dinámicos. 

El esquema también admite el almacenamiento de propiedades de las relaciones, según los campos `relationshipTarget` y `relationshipID`. El esquema clave-valor evita la necesidad de crear una columna para cada propiedad del gemelo.

### <a name="representing-properties-with-multiple-fields"></a>Representación de propiedades con varios campos 

Es posible que quiera almacenar una propiedad en el esquema con varios campos. Estas propiedades se representan mediante el almacenamiento de un objeto JSON como `value` en el esquema.

Por ejemplo, si desea representar una propiedad con tres campos para roll, pitch y yaw, el objeto de valor tendría este aspecto: `{"roll": 20, "pitch": 15, "yaw": 45}`.

## <a name="next-steps"></a>Pasos siguientes

* Vea la documentación del complemento para el lenguaje Kusto en ADX: [complemento azure_digital_twins_query_request](/azure/data-explorer/kusto/query/azure-digital-twins-query-request-plugin)

* Vea consultas de ejemplo mediante el complemento, incluido un tutorial que ejecuta las consultas en un escenario de ejemplo: [Complemento de consulta de Azure Digital Twins para ADX: Consultas y tutorial de ejemplo](https://github.com/Azure-Samples/azure-digital-twins-getting-started/tree/main/adt-adx-queries). 

* Obtenga información sobre otra estrategia para analizar datos históricos en Azure Digital Twins: [Procedimiento: Integración con Azure Time Series Insights](how-to-integrate-time-series-insights.md)
