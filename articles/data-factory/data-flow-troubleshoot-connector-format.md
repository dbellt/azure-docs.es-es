---
title: Solución de problemas de conectores y formatos en flujos de datos de asignación
description: Obtenga información sobre la solución de problemas de flujos de datos relacionados con conectores y formatos en Azure Data Factory.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 05/24/2021
ms.openlocfilehash: 1dbbbc76cb67adb678cc557c4193c0a25f280540
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952965"
---
# <a name="troubleshoot-connector-and-format-issues-in-mapping-data-flows-in-azure-data-factory"></a>Solución de problemas de conectores y formatos en flujos de datos de asignación en Azure Data Factory


En este artículo se exploran los métodos de solución de problemas relacionados con conectores y formatos para flujos de datos de asignación en Azure Data Factory (ADF).


## <a name="cosmos-db--json"></a>Cosmos DB y JSON

### <a name="support-customized-schemas-in-the-source"></a>Compatibilidad con esquemas personalizados en el origen

#### <a name="symptoms"></a>Síntomas
Si quiere usar el flujo de datos de ADF para mover o transferir datos desde Cosmos DB/JSON hacia otros almacenes de datos, es posible que falten algunas columnas de los datos de origen. 

#### <a name="cause"></a>Causa 
Para los conectores sin esquema (el número de columna, el nombre de columna y el tipo de datos de columna de cada fila pueden ser diferentes cuando se comparan con otros), de manera predeterminada, ADF usa filas de ejemplo (por ejemplo, datos de las 100 o 1000 filas principales) para inferir el esquema, y el resultado inferido se usará como esquema para leer los datos. Por lo tanto, si los almacenes de datos tienen columnas adicionales que no aparecen en las filas de ejemplo, los datos de estas columnas adicionales no se leen, mueven ni transfieren a almacenes de datos receptores.

#### <a name="recommendation"></a>Recomendación
Para sobrescribir el comportamiento predeterminado y traer campos adicionales, ADF da opciones para personalizar el esquema de origen. Puede especificar columnas adicionales o que faltan que podrían faltar en el resultado de inferencia del esquema en la proyección del origen del flujo de datos para leer los datos, y puede aplicar una de las siguientes opciones para establecer el esquema personalizado. En general, se prefiere la **opción 1**.

- **Opción 1**: En comparación con los datos de origen originales, que pueden ser un archivo, una tabla o un contenedor grandes que contengan millones de filas con esquemas complejos, puede crear una tabla o contenedor temporales con algunas filas que contengan todas las columnas que quiere leer y, a continuación, pasar a la siguiente operación: 

    1. Use la **Configuración de depuración** del origen del flujo de datos para hacer que **Import projection** (Importar proyección) con los archivos o tablas de ejemplo obtenga el esquema completo. Puede seguir los pasos de la siguiente imagen:<br/>

        ![Captura de pantalla que muestra la primera parte de la primera opción para personalizar el esquema de origen.](./media/data-flow-troubleshoot-connector-format/customize-schema-option-1-1.png)<br/>
         1. Seleccione **Configuración de depuración** en el lienzo del flujo de datos.
         1. En el panel emergente, seleccione **Sample table** (Tabla de ejemplo) en la pestaña **cosmosSource** y escriba el nombre de la tabla en el bloque **Tabla**.
         1. Haga clic en **Guardar** para guardar la configuración.
         1. Seleccione **Import projection** (Importar proyección).<br/>  
    
    1. Vuelva a cambiar la **Configuración de depuración** para usar el conjunto de datos de origen para el movimiento o la transformación de datos restantes. Puede seguir con los pasos de la imagen siguiente:<br/>

        ![Captura de pantalla que muestra la segunda parte de la primera opción para personalizar el esquema de origen.](./media/data-flow-troubleshoot-connector-format/customize-schema-option-1-2.png) <br/>   
         1. Seleccione **Configuración de depuración** en el lienzo del flujo de datos.
         1. En el panel emergente, seleccione **Source dataset** (Conjunto de datos de origen) en la pestaña **cosmosSource**.
         1. Haga clic en **Guardar** para guardar la configuración.<br/>
    
    Después, el entorno en tiempo de ejecución del flujo de datos de ADF respetará y usará el esquema personalizado para leer los datos del almacén de datos original. <br/>

- **Opción 2**: Si está familiarizado con el esquema y el lenguaje DSL de los datos de origen, puede actualizar manualmente el script de origen del flujo de datos para agregar columnas adicionales o que faltan para leer los datos. En la imagen siguiente se muestra un ejemplo: 

    ![Captura de pantalla que muestra la segunda opción para personalizar el esquema de origen.](./media/data-flow-troubleshoot-connector-format/customize-schema-option-2.png)

### <a name="consume-json-files-generated-by-copy-activities"></a>Uso de archivos JSON generados por actividades de copia

#### <a name="symptoms"></a>Síntomas

Si usa la actividad de copia para generar algunos archivos JSON y, a continuación, intenta leer estos archivos en flujos de datos, se produce un error en el que aparece el siguiente mensaje: `JSON parsing error, unsupported encoding or multiline`

#### <a name="cause"></a>Causa

Existen las siguientes limitaciones en JSON para los flujos de copia y de datos, respectivamente:

- En el caso de los archivos JSON con codificaciones Unicode (utf-8, utf-16, utf-32) las actividades de copia siempre generan los archivos JSON con BOM.
- El origen JSON del flujo de datos con "Documento único" habilitado no admite la codificación Unicode con BOM.

    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/enabled-single-document.png" alt-text="Captura de pantalla que muestra el &quot;Documento único&quot; habilitado."::: 


Por lo tanto, experimentará problemas si se cumplen los siguientes criterios:

- El conjunto de datos de receptor que la actividad de copia utiliza está establecido en la codificación Unicode (utf-8, utf-16, utf-16be, utf-32, utf-32be) o se usa el valor predeterminado.
- El receptor de copia se establece para usar el patrón de archivo "Matriz de objetos" como se muestra en la siguiente imagen, independientemente de si "Documento único" está habilitado o no en el origen JSON del flujo de datos. 

    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/array-of-objects-pattern.png" alt-text="Captura de pantalla que muestra el patrón &quot;Matriz de objetos&quot;":::. 
   
#### <a name="recommendation"></a>Recomendación

- Use siempre el patrón de archivo predeterminado o el patrón "Conjunto de objetos" explícito en el receptor de copia si los archivos generados se usan en flujos de datos.
- Deshabilite la opción "Documento único" en el origen JSON del flujo de datos.

>[!Note]
> El uso de "Conjunto de objetos" también es el procedimiento recomendado desde la perspectiva del rendimiento. Como el JSON del patrón "Documento único" del flujo de datos no puede habilitar la lectura en paralelo para archivos grandes únicos, esta recomendación no tiene ningún efecto negativo.

### <a name="the-query-with-parameters-does-not-work"></a>La consulta con parámetros no funciona

#### <a name="symptoms"></a>Síntomas

Los flujos de datos de asignación de Azure Data Factory también admiten el uso de parámetros. La canalización de llamada establece los valores de parámetros mediante la actividad de ejecución de Data Flow y el uso de los parámetros hace que los flujos de datos sean de uso general, flexibles y reutilizables. Puede parametrizar la configuración y las expresiones del flujo de datos con estos parámetros: [Parametrización de flujos de datos de asignación](./parameters-data-flow.md).

Después de configurar los parámetros y usarlos en la consulta de origen del flujo de datos, no surten efecto.

#### <a name="cause"></a>Causa

Este error se produce debido a una configuración incorrecta.

#### <a name="recommendation"></a>Recomendación

Use las siguientes reglas para establecer los parámetros de la consulta y, para obtener información más detallada, consulte [Generación de expresiones del flujo de datos de asignación](./concepts-data-flow-expression-builder.md).

1. Utilice comillas dobles al principio de la instrucción SQL.
2. Use comillas simples alrededor del parámetro.
3. Emplee letras en minúsculas para todas las instrucciones CLAUSE.

Por ejemplo:

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/set-parameter-in-query.png" alt-text="Captura de pantalla que muestra el parámetro establecido en la consulta."::: 

## <a name="cdm"></a>CDM

### <a name="modeljson-files-with-special-characters"></a>Archivos Model.Json con caracteres especiales

#### <a name="symptoms"></a>Síntomas 
Puede encontrarse con un problema por el que el nombre final del archivo model.json contiene caracteres especiales.  

#### <a name="error-message"></a>Mensaje de error  
`at Source 'source1': java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: PPDFTable1.csv@snapshot=2020-10-21T18:00:36.9469086Z. ` 

#### <a name="recommendation"></a>Recomendación  
Reemplace los caracteres especiales del nombre de archivo, lo cual funcionará en Synapse, pero no en ADF.  

### <a name="no-data-output-in-the-data-preview-or-after-running-pipelines"></a>No hay salida de datos en la vista previa de los datos ni después de ejecutar canalizaciones

#### <a name="symptoms"></a>Síntomas
Cuando se usa el archivo manifest.json para CDM no se muestra ningún dato en la vista previa de los datos ni después de ejecutar una canalización. Solo se muestran los encabezados. Puede ver este problema en la imagen siguiente.<br/>

![Captura de pantalla que muestra el síntoma de salida sin datos.](./media/data-flow-troubleshoot-connector-format/no-data-output.png)

#### <a name="cause"></a>Causa
En el documento de manifiesto se describe la carpeta CDM, por ejemplo, qué entidades tiene en la carpeta, las referencias de esas entidades y los datos que corresponden a esta instancia. Al documento de manifiesto le falta la información sobre `dataPartitions` que indica a ADF dónde leer los datos y, como está vacía, devuelve cero datos. 

#### <a name="recommendation"></a>Recomendación
Actualice el documento de manifiesto para que incluya la información sobre `dataPartitions` y pueda hacer referencia a este documento de manifiesto de ejemplo para actualizar el documento: [Metadatos de Common Data Model: Introducción del documento de manifiesto de ejemplo](/common-data-model/cdm-manifest#example-manifest-document).

### <a name="json-array-attributes-are-inferred-as-separate-columns"></a>Los atributos de la matriz JSON se deducen como columnas independientes.

#### <a name="symptoms"></a>Síntomas 
Puede encontrarse con un problema por el que un atributo (de tipo cadena) de la entidad de CDM tiene una matriz JSON como datos. Cuando se encuentra estos datos, ADF deduce de manera incorrecta los datos como columnas independientes. Como puede ver en las imágenes siguientes, un único atributo que se presenta en el origen (msfp_otherproperties) se deduce como una columna independiente en la vista previa del conector de CDM.<br/> 

- En los datos de origen del archivo .csv (consulte la segunda columna): <br/>

    ![Captura de pantalla que muestra el atributo en los datos de origen del archivo .csv.](./media/data-flow-troubleshoot-connector-format/json-array-csv.png)

- En la vista previa de los datos de origen de CDM: <br/>

    ![Captura de pantalla que muestra la columna independiente en los datos de origen de CDM.](./media/data-flow-troubleshoot-connector-format/json-array-cdm.png)

 
También puede intentar asignar las columnas desplazadas y usar la expresión del flujo de datos para transformar este atributo como una matriz. Pero, dado que este atributo se lee como una columna independiente, la transformación en una matriz no funciona.  

#### <a name="cause"></a>Causa
Es probable que este problema se deba a las comas que hay en el valor del objeto JSON de esa columna. Como se espera que el archivo de datos sea un archivo .csv, la coma indica que es el final del valor de una columna. 

#### <a name="recommendation"></a>Recomendación
Para resolver este problema, debe entrecomillar con comillas dobles la columna JSON y reemplazar cualquiera de las comillas internas por una barra diagonal inversa (`\`). De este modo, el contenido del valor de esa columna se puede leer como una sola columna por completo.  
  
>[!Note]
>El CDM no informa de que el tipo de datos del valor de columna es JSON, pero informa de que es una cadena y se analiza como tal.

### <a name="unable-to-fetch-data-in-the-data-flow-preview"></a>No se pueden capturar datos en la vista previa del flujo de datos

#### <a name="symptoms"></a>Síntomas
Se usa CDM con el archivo model.json que Power BI genera. Al obtener una vista previa de los datos de CDM mediante la vista previa del flujo de datos, se produce un error: `No output data.`

#### <a name="cause"></a>Causa
 El código siguiente existe en las particiones del archivo model.json que el flujo de datos de Power BI genera.
```json
"partitions": [  
{  
"name": "Part001",  
"refreshTime": "2020-10-02T13:26:10.7624605+00:00",  
"location": "https://datalakegen2.dfs.core.windows.net/powerbi/salesEntities/salesPerfByYear.csv @snapshot=2020-10-02T13:26:10.6681248Z"  
}  
```
En este archivo model.json, el problema es que el esquema de nomenclatura del archivo de partición de datos contiene caracteres especiales y las rutas de acceso de archivo compatibles con "@" no existen actualmente.  

#### <a name="recommendation"></a>Recomendación
Quite la parte `@snapshot=2020-10-02T13:26:10.6681248Z` del nombre del archivo de partición de datos y del archivo model.json y vuelva a intentarlo. 

### <a name="the-corpus-path-is-null-or-empty"></a>La ruta de acceso del corpus es NULL o está vacía

#### <a name="symptoms"></a>Síntomas
Cuando se usa CDM en el flujo de datos con el formato de modelo, no se puede obtener una vista previa de los datos y se produce el error: `DF-CDM_005 The corpus path is null or empty`. En la imagen siguiente se muestra el error:  

![Captura de pantalla que muestra el error en la ruta de acceso del corpus.](./media/data-flow-troubleshoot-connector-format/corpus-path-error.png)

#### <a name="cause"></a>Causa
La ruta de acceso de la partición de datos del archivo model.json apunta a una ubicación de almacenamiento de blobs y no al lago de datos. La ubicación debe tener la dirección URL base de **.dfs.core.windows.net** para ADLS Gen2. 

#### <a name="recommendation"></a>Recomendación
Para solucionar este problema, puede consultar este artículo: [ADF incorpora la compatibilidad con los conjuntos de datos en línea y Common Data Model a los flujos de datos](https://techcommunity.microsoft.com/t5/azure-data-factory/adf-adds-support-for-inline-datasets-and-common-data-model-to/ba-p/1441798), y la siguiente imagen muestra la manera de corregir el error de la ruta de acceso del corpus de este artículo.

![Captura de pantalla que muestra cómo solucionar el error en la ruta de acceso del corpus.](./media/data-flow-troubleshoot-connector-format/fix-format-issue.png)

### <a name="unable-to-read-csv-data-files"></a>No se pueden leer archivos de datos CSV

#### <a name="symptoms"></a>Síntomas 
Usa el conjunto de datos en línea como Common Data Model con el manifiesto como origen y ha proporcionado el archivo de manifiesto de entrada, la ruta de acceso raíz, el nombre de la entidad y la ruta de acceso. En el manifiesto, tiene las particiones de datos con la ubicación del archivo .csv. Mientras tanto, el esquema de entidad y el esquema csv son idénticos y todas las validaciones se han realizado correctamente. Sin embargo, en la vista previa de los datos, solo se carga el esquema en lugar de los datos y estos no se pueden ver. En esta imagen se muestra esta situación:

![Captura de pantalla que muestra el problema de no poder leer los archivos de datos.](./media/data-flow-troubleshoot-connector-format/unable-read-data.png)

#### <a name="cause"></a>Causa
La carpeta de CDM no está separada en modelos lógicos y físicos, y solo existen modelos físicos en esta carpeta. En los dos artículos siguientes se describe la diferencia: [Definiciones lógicas](/common-data-model/sdk/logical-definitions) y [Resolución de una definición de entidad lógica](/common-data-model/sdk/convert-logical-entities-resolved-entities).<br/> 

#### <a name="recommendation"></a>Recomendación
Para el flujo de datos que usa CDM como origen, intente usar un modelo lógico como referencia de entidad y use el manifiesto que describe la ubicación de las entidades físicas resueltas y las ubicaciones de las particiones de datos. Puede ver algunos ejemplos de definiciones de entidades lógicas en el repositorio de GitHub de CDM público: [CDM-schemaDocuments](https://github.com/microsoft/CDM/tree/master/schemaDocuments)<br/>

Un buen punto de partida para formar el corpus es copiar los archivos dentro de la carpeta de documentos de esquema (solo ese nivel dentro del repositorio de github) y colocar esos archivos en una carpeta. Después, puede usar una de las entidades lógicas predefinidas dentro del repositorio (como punto de partida o de referencia) para crear el modelo lógico.<br/>

Una vez configurado el corpus, se recomienda usar CDM como receptor en los flujos de datos, para que se pueda crear correctamente una carpeta CDM bien formada. Puede usar el conjunto de datos CSV como origen y, a continuación, recibirlos en el modelo de CDM que ha creado.

## <a name="delta"></a>Delta

### <a name="the-sink-does-not-support-the-schema-drift-with-upsert-or-update"></a>El receptor no admite el desfase de esquema con las operaciones de actualización e inserción o de solo actualización.

#### <a name="symptoms"></a>Síntomas
Puede encontrarse con el problema de que el receptor delta de los flujos de datos de asignación no admite el desfase de esquema con las operaciones de actualización e inserción o de solo actualización. El problema es que el desfase de esquema no funciona si el receptor delta es el destino de un flujo de datos de asignación y el usuario configura una operación de actualización e inserción o de solo actualización. 

Si se agrega una columna al origen después de una carga "inicial" en el receptor delta, los trabajos posteriores simplemente producirán un error que le indica que no puede encontrar la nueva columna y esto sucede cuando realiza la operación de actualización e inserción con la fila de alteración. Solo parece funcionar con las operaciones de inserción.

#### <a name="error-message"></a>Mensaje de error
`DF-SYS-01 at Sink 'SnkDeltaLake': org.apache.spark.sql.AnalysisException: cannot resolve target.BICC_RV in UPDATE clause given columns target. `

#### <a name="cause"></a>Causa
Se trata de un problema con el formato delta debido a la limitación de la biblioteca io delta que se usa en el entorno de ejecución del flujo de datos. Este problema todavía está en vías de solución.

#### <a name="recommendation"></a>Recomendación
Para solucionar este problema, primero debe actualizar el esquema y, a continuación, escribir los datos. Siga estos pasos: <br/>
1. Cree un flujo de datos que incluya un receptor delta de solo inserción con la opción de combinación de esquema para actualizar el esquema. 
1. Después del paso 1, use las operaciones de eliminación, actualización e inserción, y actualización para modificar el receptor de destino sin cambiar el esquema. <br/>

## <a name="azure-postgresql"></a>Azure PostgreSQL

### <a name="encounter-an-error-failed-with-exception-handshake_failure"></a>Se produce un error: No se pudo ejecutar la operación porque se produjo la siguiente excepción: handshake_failure 

#### <a name="symptoms"></a>Síntomas
Azure PostgreSQL se usa como origen o receptor en el flujo de datos como, por ejemplo, en la obtención de una vista previa de los datos y en una depuración o desencadenamiento de la ejecución, y puede que el trabajo no funcione y que aparezca el siguiente mensaje de error: 

   `PSQLException: SSL error: Received fatal alert: handshake_failure `<br/>
   `Caused by: SSLHandshakeException: Received fatal alert: handshake_failure.`

#### <a name="cause"></a>Causa 
Si usa el servidor flexible o Hiperescala (Citus) para el servidor de Azure PostgreSQL, como el sistema se crea mediante Spark en un clúster de Azure Databricks, existe una limitación en Azure Databricks que impide al sistema conectarse al servidor flexible o de Hiperescala (Citus). Puede consultar los dos vínculos siguientes como referencias.
- [Se produce un error en el protocolo de enlace al intentar conectarse desde Azure Databricks a Azure PostgreSQL con SSL](/answers/questions/170730/handshake-fails-trying-to-connect-from-azure-datab.html)
 
- [MCW: Datos en tiempo real con Azure Database for PostgreSQL: Hiperescala](https://github.com/microsoft/MCW-Real-time-data-with-Azure-Database-for-PostgreSQL-Hyperscale/blob/master/Hands-on%20lab/HOL%20step-by%20step%20-%20Real-time%20data%20with%20Azure%20Database%20for%20PostgreSQL%20Hyperscale.md)<br/>
    Consulte el contenido de la siguiente imagen de este artículo:<br/>

    ![Capturas de pantalla que muestran el contenido de referencia del artículo anterior.](./media/data-flow-troubleshoot-connector-format/handshake-failure-cause-2.png)

#### <a name="recommendation"></a>Recomendación
Puede intentar usar actividades de copia para desbloquear este problema. 

## <a name="csv-and-excel"></a>CSV y Excel

### <a name="set-the-quote-character-to-no-quote-char-is-not-supported-in-the-csv"></a>Establecer el carácter de comillas en "ningún carácter de comillas" no se admite en el archivo CSV.
 
#### <a name="symptoms"></a>Síntomas

Se producen varios problemas en el archivo .csv si el carácter de comillas está establecido en "ningún carácter de comillas":

1. Si el carácter de comillas está establecido en "ningún carácter de comillas", el delimitador de columna de varios caracteres no puede empezar ni terminar por las mismas letras.
2. Si el carácter de comillas está establecido en "ningún carácter de comillas", el delimitador de columna de varios caracteres no puede contener este carácter de escape: `\`.
3. Si el carácter de comillas está establecido en "ningún carácter de comillas", el valor de columna no puede contener ningún delimitador de filas.
4. El carácter de comillas y el carácter de escape no pueden estar ambos vacíos (sin comillas y sin escape) si el valor de columna contiene un delimitador de columna.

#### <a name="cause"></a>Causa

A continuación se indican las causas de los síntomas con sus respectivos ejemplos:
1. Empieza y termina por las mismas letras.<br/>
`column delimiter: $*^$*`<br/>
`column value: abc$*^    def`<br/>
`csv sink: abc$*^$*^$*def ` <br/>
`will be read as "abc" and "^&*def"`<br/>

2. El delimitador de varios caracteres contiene caracteres de escape.<br/>
`column delimiter: \x`<br/>
`escape char:\`<br/>
`column value: "abc\\xdef"`<br/>
El carácter de escape se aplicará al delimitador de columna o al carácter.

3. El valor de columna contiene el delimitador de fila. <br/>
`We need quote character to tell if row delimiter is inside column value or not.`

4. El carácter de comillas y el carácter de escape están ambos vacíos y el valor de columna contiene delimitadores de columna.<br/>
`Column delimiter: \t`<br/>
`column value: 111\t222\t33\t3`<br/>
`It will be ambigious if it contains 3 columns 111,222,33\t3 or 4 columns 111,222,33,3.`<br/>

#### <a name="recommendation"></a>Recomendación
El primer y el segundo síntoma no se pueden solucionar actualmente. Para el tercer y cuarto síntoma, puede aplicar los métodos siguientes:
- En el caso del tercer síntoma, no use la opción "ningún carácter de comillas" en un archivo .csv de varias líneas.
- En el caso del cuarto síntoma, establezca el carácter de comilla o el carácter de escape como no vacío, o bien elimine todos los delimitadores de columna de los datos.

### <a name="read-files-with-different-schemas-error"></a>Error de lectura de archivos con esquemas diferentes

#### <a name="symptoms"></a>Síntomas

Cuando se usan flujos de datos para leer archivos como, por ejemplo, archivos .csv y Excel con esquemas diferentes, se producirá un error en la depuración del flujo de datos, el espacio aislado o la ejecución de la actividad.
- En el caso de los archivos .csv, la desalineación de los datos se produce cuando el esquema de los archivos es diferente. 

    ![Captura de pantalla que muestra el primer error de esquema.](./media/data-flow-troubleshoot-connector-format/schema-error-1.png)

- En el caso de Excel, se produce un error cuando el esquema del archivo es diferente.

    ![Captura de pantalla que muestra el segundo error de esquema.](./media/data-flow-troubleshoot-connector-format/schema-error-2.png)

#### <a name="cause"></a>Causa

No se admite la lectura de archivos con esquemas diferentes en el flujo de datos.

#### <a name="recommendation"></a>Recomendación

Si aún desea transferir archivos, por ejemplo, archivos CSV y Excel, con esquemas diferentes en el flujo de datos, puede usar los siguientes métodos:

- Para CSV, debe combinar manualmente el esquema de los diferentes archivos para obtener el esquema completo. Por ejemplo, file_1 tiene las columnas `c_1, c_2, c_3` mientras que file_2 tiene las columnas `c_3, c_4,... c_10`, por lo que el esquema combinado y el completo sería `c_1, c_2... c_10`. A continuación, haga que otros archivos también tengan el mismo esquema completo aunque no tengan datos, por ejemplo, file_x solo tiene las columnas `c_1, c_2, c_3, c_4`, agregue las columnas `c_5, c_6, ... c_10` adicionales al archivo y ya puede funcionar.

- En el caso de Excel, puede solucionar este problema aplicando una de las siguientes opciones:

    - **Opción 1**: debe combinar manualmente el esquema de los diferentes archivos para obtener el esquema completo. Por ejemplo, file_1 tiene las columnas `c_1, c_2, c_3` mientras que file_2 tiene las columnas `c_3, c_4,... c_10`, por lo que el esquema combinado y el completo sería `c_1, c_2... c_10`. A continuación, haga que otros archivos también tengan el mismo esquema aunque no tengan datos, por ejemplo, file_x con la hoja "SHEET_1" solo tiene las columnas `c_1, c_2, c_3, c_4`, agregue las columnas `c_5, c_6, ... c_10` adicionales también a la hoja y ya puede funcionar.
    - **Opción 2:** use un **intervalo (por ejemplo, A1:G100) + firstRowAsHeader=false** y, a continuación, puede cargar los datos de todos los archivos de Excel, aunque el nombre y el número de columnas sean diferentes.

## <a name="azure-synapse-analytics"></a>Azure Synapse Analytics

### <a name="serverless-pool-sql-on-demand-related-issues"></a>Problemas relacionados con un grupo sin servidor (SQL a petición)

#### <a name="symptoms"></a>Síntomas
Utiliza Azure Synapse Analytics y el servicio vinculado es en realidad un grupo sin servidor de Synapse. Su nombre anterior es grupo de SQL a petición y se puede distinguir porque el nombre del servidor contiene `ondemand` como, por ejemplo, `space-ondemand.sql.azuresynapse.net`. Puede encontrarse con varios errores únicos como se indica a continuación:<br/>

1. Si desea usar el grupo sin servidor de Synapse como receptor, se produce el siguiente error:<br/>
`Sink results in 0 output columns. Please ensure at least one column is mapped`
1. Al seleccionar "Enable Staging" (Habilitar almacenamiento provisional) en el origen, se produce el siguiente error: `shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: Incorrect syntax near 'IDENTITY'.`
1. Si desea capturar datos de una tabla externa, se produce el siguiente error: `shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: External table 'dbo' is not accessible because location does not exist or it is used by another process.`
1. Si desea capturar datos de Cosmos DB a través de un grupo sin servidor mediante consulta o desde la vista, se produce el siguiente error: `Job failed due to reason: Connection reset.`
1. Si desea capturar datos desde una vista, puede encontrarse con diferentes errores.

#### <a name="cause"></a>Causa
Las respectivas causas de los síntomas se indican a continuación:
1. El grupo sin servidor no se puede usar como receptor. No admite la escritura de datos en la base de datos.
1. El grupo sin servidor no admite la carga de datos almacenados provisionalmente, por lo que no se admite la opción para "habilitar el almacenamiento provisional". 
1. El método de autenticación que usa no tiene un permiso correcto para el origen de datos externo al que hace referencia la tabla externa.
1. Existe una limitación conocida en el grupo sin servidor de Synapse que le impide capturar datos de Cosmos DB a partir de flujos de datos.
1. La vista es una tabla virtual basada en una instrucción SQL. La causa principal está dentro de la instrucción de la vista.

#### <a name="recommendation"></a>Recomendación

Puede aplicar los pasos siguientes para resolver los problemas correspondientes.
1. Es mejor no usar un grupo sin servidor como receptor.
1. No use la opción para "habilitar el almacenamiento provisional" en el origen para el grupo sin servidor.
1. Solo la entidad de servicio o la identidad administrada que tenga el permiso para los datos de la tabla externa pueden consultarla. Debe conceder el permiso "Colaborador de datos de Storage Blob" al origen de datos externo del método de autenticación que use en ADF.
    >[!Note]
    > La autenticación de usuario y contraseña no permite consultar tablas externas. Para más información, puede consultar este artículo: [Modelo de seguridad](../synapse-analytics/metadata/database.md#security-model).

1. Puede usar la actividad de copia para capturar datos de Cosmos DB del grupo sin servidor.
1. Puede proporcionar la instrucción SQL que crea la vista al equipo de soporte técnico de ingeniería y ellos le pueden ayudar a analizar si la instrucción tiene un problema de autenticación o de cualquier otro tipo.


### <a name="load-small-size-data-to-data-warehouse-without-staging-is-slow"></a>La carga de datos de tamaño pequeño en Data Warehouse sin almacenamiento provisional es lenta 

#### <a name="symptoms"></a>Síntomas
Cuando se cargan datos pequeños en Data Warehouse sin almacenamiento provisional, la operación tarda mucho tiempo en finalizar. Por ejemplo, el tamaño de los datos es de 2 MB, pero se tarda más de una hora en realizar la carga. 

#### <a name="cause"></a>Causa
Este problema se debe al recuento de filas en lugar de al tamaño. El recuento de filas presenta varios miles y cada inserción se debe empaquetar en una solicitud independiente, ir al nodo de control, iniciar una nueva transacción, obtener bloqueos e ir al nodo de distribución en repetidas ocasiones. La carga masiva obtiene el bloqueo una vez y cada nodo de distribución realiza la inserción mediante el procesamiento por lotes en memoria de forma eficaz.

Si se insertaran 2 MB en tan solo unos pocos registros, sería rápido. Por ejemplo, sería rápido si cada registro es de 500 kb * 4 filas. 

#### <a name="recommendation"></a>Recomendación
Debe habilitar el almacenamiento provisional para mejorar el rendimiento.


### <a name="read-empty-string-value--as-null-with-the-enable-staging"></a>Lectura del valor de cadena vacío ("") como NULL con el almacenamiento provisional habilitado. 

#### <a name="symptoms"></a>Síntomas
Cuando se usa Synapse como origen del flujo de datos, por ejemplo en la obtención de la vista previa de datos y en la depuración y desencadenamiento de una ejecución, y se habilita el almacenamiento provisional para que use PolyBase, si el valor de columna contiene un valor de cadena vacío (`""`), este se cambiará a NULL.

#### <a name="cause"></a>Causa
El back-end del flujo de datos usa Parquet como formato de PolyBase y hay una limitación conocida en el grupo de Synapse SQL gen2, que cambia automáticamente el valor de cadena vacía a NULL.

#### <a name="recommendation"></a>Recomendación
Puede intentar solucionar este problema mediante los métodos siguientes:
1. Si el tamaño de los datos no es muy grande, puede deshabilitar la opción **Enable staging** (Habilitar el almacenamiento provisional) en el origen, pero esto afectará al rendimiento.
1. Si necesita habilitar el almacenamiento provisional, puede usar la función **iifNull()** para cambiar manualmente la columna específica de NULL a un valor de cadena vacío.

### <a name="managed-service-identity-error"></a>Error de identidad de servicio administrada

#### <a name="symptoms"></a>Síntomas
Cuando usa Synapse como origen o receptor del flujo de datos para obtener una vista previa de los datos, depurar o desencadenar una ejecución, etc., y habilita el almacenamiento provisional para usar PolyBase, y se crea el servicio vinculado del almacén de almacenamiento provisional (Blob, Gen2, etc.) para usar la autenticación con identidades administradas, el trabajo podría producir el siguiente error que se muestra en la imagen: <br/>

![Capturas de pantalla que muestran el error de identidad del servicio.](./media/data-flow-troubleshoot-connector-format/service-identity-error.png)

#### <a name="error-message"></a>Mensaje de error
`shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: Managed Service Identity has not been enabled on this server. Please enable Managed Service Identity and try again.`

#### <a name="cause"></a>Causa
1. Si el grupo de SQL se crea a partir del área de trabajo de Synapse, no se admite la autenticación con la identidad administrada en el almacenamiento provisional con PolyBase para el grupo de SQL anterior. 
1. Si el grupo de SQL es de la versión de Data Warehouse anterior, la identidad administrada del servidor de SQL Server no se asigna al almacén de almacenamiento provisional.

#### <a name="recommendation"></a>Recomendación
Debe confirmar si el grupo de SQL se ha creado a partir del área de trabajo de Synapse.

- Si el grupo de SQL se ha creado a partir del área de trabajo de Synapse, debe volver a registrar la identidad administrada del área de trabajo. Puede aplicar los pasos siguientes para solucionar este problema volviendo a registrar la identidad administrada del área de trabajo:
    1. Vaya al área de trabajo de Synapse en Azure Portal.
    1. Vaya a la hoja **Identidades administradas**.
    1. Si la opción **Allow pipelines** (Permitir canalizaciones) ya está activada, debe desactivarla y guardar.
    1. Active la opción **Allow pipelines** (Permitir canalizaciones) y guarde.

- Si el grupo de SQL es de la versión anterior de Data Warehouse, habilite la identidad administrada solo para su servidor de SQL Server y asigne el permiso del almacén de almacenamiento provisional a esta. Puede consultar los pasos de este artículo como ejemplo: [Uso de reglas y puntos de conexión de servicio de red virtual para servidores de Azure SQL Database](../azure-sql/database/vnet-service-endpoint-rule-overview.md#steps).

### <a name="failed-with-an-error-sqlserverexception-not-able-to-validate-external-location-because-the-remote-server-returned-an-error-403"></a>No se pudo ejecutar la operación debido a un error: "SQLServerException: No se puede validar la ubicación externa porque el servidor remoto devolvió un error: (403)"

#### <a name="symptoms"></a>Síntomas

Cuando se usa SQLDW como receptor para desencadenar y ejecutar actividades de flujo de datos, se puede producir un error en la actividad como el siguiente: `"SQLServerException: Not able to validate external location because the remote server returned an error: (403)"`

#### <a name="cause"></a>Causa
1. Cuando se usa la identidad administrada en el método de autenticación de la cuenta de ADLS Gen2 como almacenamiento provisional, puede que cx no establezca correctamente la configuración de autenticación.
1. Con el entorno de ejecución de integración de la red virtual, debe usar la identidad administrada en el método de autenticación de la cuenta de ADLS Gen2 como almacenamiento provisional. Si el almacenamiento provisional de Azure Storage está configurado con el punto de conexión de servicio de la red virtual, tiene que utilizar la autenticación de identidad administrada con la opción para permitir el servicio de Microsoft de confianza habilitada en la cuenta de almacenamiento.
1. Compruebe si el nombre de la carpeta contiene el carácter de espacio u otros caracteres especiales como, por ejemplo: `Space " < > # % |`.
Actualmente, los nombres de carpeta que contienen determinados caracteres especiales no se admiten en el comando copy de Data Warehouse.

#### <a name="recommendation"></a>Recomendación

Para la causa 1, puede consultar el siguiente documento para solucionar el problema: [Uso de reglas y puntos de conexión de servicio de red virtual para servidores de Azure SQL Database: Pasos](../azure-sql/database/vnet-service-endpoint-rule-overview.md#steps).

Para la causa 2, ponga solución mediante una de las siguientes opciones:

- Opción 1: si usa el entorno de ejecución de integración de la red virtual, debe usar la identidad administrada en el método de autenticación de la cuenta de ADLS Gen2 como almacenamiento provisional.

- Opción 2: si el almacenamiento provisional de Azure Storage está configurado con el punto de conexión de servicio de la red virtual, tiene que utilizar la autenticación de identidad administrada con la opción para permitir el servicio de Microsoft de confianza habilitada en la cuenta de almacenamiento. Para más información, consulte este documento: [Copia almacenada provisionalmente con PolyBase](./connector-azure-sql-data-warehouse.md#staged-copy-by-using-polybase).

Para la causa 3, trate de solucionar el problema con una de las siguientes opciones:

- Opción 1: cambie el nombre de la carpeta y evite usar caracteres especiales.
- Opción 2: elimine la propiedad `allowCopyCommand:true` del script del flujo de datos, por ejemplo:

    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/remove-allow-copy-command-true.png" alt-text="Captura de pantalla que indica cómo quitar &quot;allowcopycommand:true&quot;."::: 


### <a name="failed-with-an-error-this-operation-is-not-permitted-on-a-non-empty-directory"></a>No se pudo ejecutar la operación debido a un error: "Esta operación no se permite en un directorio no vacío".

#### <a name="symptoms"></a>Síntomas

Si usa Azure Synapse Analytics como receptor en el flujo de datos para obtener una vista previa de los datos, depurar o desencadenar una ejecución, o realizar otras actividades, y la opción para habilitar el almacenamiento provisional está establecida en true, el trabajo puede generar el siguiente mensaje de error:

`DF-SYS-01 at Sink 'sink': Unable to stage data before write. Check configuration/credentials of storage.`<br/>
`org.apache.hadoop.fs.azure.AzureException: com.microsoft.azure.storage.StorageException: This operation is not permitted on a non-empty directory.`

#### <a name="cause"></a>Causa
Usa Azure Blob Storage como el servicio vinculado de almacenamiento provisional para vincularse a una cuenta de almacenamiento que tiene habilitado el espacio de nombres jerárquico, y esa cuenta usa la autenticación de clave en el servicio vinculado.

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/storage-account-configuration.png" alt-text="Captura de pantalla que muestra la configuración de la cuenta de almacenamiento."::: 

#### <a name="recommendation"></a>Recomendación
Cree un servicio vinculado de Azure Data Lake Gen2 para el almacenamiento y seleccione el almacenamiento Gen2 como servicio vinculado de almacenamiento provisional en las actividades del flujo de datos.


## <a name="azure-blob-storage"></a>Azure Blob Storage

### <a name="account-kind-of-storage-general-purpose-v1-doesnt-support-service-principal-and-mi-authentication"></a>El tipo de cuenta de Storage (uso general v1) no admite la autenticación de entidades de servicio ni de identidades administradas.

#### <a name="symptoms"></a>Síntomas

En los flujos de datos, si usa Azure Blob Storage (uso general v1) con la autenticación de entidad de servicio o de identidad administrada, puede que aparezca el siguiente mensaje de error:

`com.microsoft.dataflow.broker.InvalidOperationException: ServicePrincipal and MI auth are not supported if blob storage kind is Storage (general purpose v1)`

#### <a name="cause"></a>Causa

Cuando se usa el servicio vinculado de Azure Blob Storage en los flujos de datos, la autenticación de identidad administrada o de entidad de servicio no se admite cuando el tipo de cuenta está definido como vacío o "almacenamiento". Esta situación se muestra en las imágenes 1 y 2 que aparecen a continuación.

Imagen 1: el tipo de cuenta del servicio vinculado Azure Blob Storage

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/storage-account-kind.png" alt-text="Captura de pantalla que muestra el tipo de cuenta de almacenamiento del servicio vinculado Azure Blob Storage."::: 

Imagen 2: página de la cuenta de almacenamiento

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/storage-account-page.png" alt-text="Captura de pantalla que muestra la página de la cuenta de almacenamiento." lightbox="./media/data-flow-troubleshoot-connector-format/storage-account-page.png"::: 


#### <a name="recommendation"></a>Recomendación

Para solucionar este problema, consulte las siguientes recomendaciones:

- Si el tipo de cuenta de almacenamiento es **Ninguno** en el servicio vinculado Azure Blob Storage, especifique el tipo de cuenta adecuado y consulte la imagen 3 que se muestra a continuación para realizarlo. Además, consulte la imagen 2 para obtener el tipo de cuenta de almacenamiento y compruebe y confirme que este no es Storage (uso general v1).

    Imagen 3: especifique el tipo de cuenta de almacenamiento del servicio vinculado Azure Blob Storage.

    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/specify-storage-account-kind.png" alt-text="Captura de pantalla que muestra cómo especificar el tipo de cuenta de almacenamiento del servicio vinculado Azure Blob Storage."::: 
    

- Si el tipo de cuenta es Storage (uso general v1), actualice la cuenta de almacenamiento a la versión de **uso general v2** o elija una autenticación diferente.

    Imagen 4: actualice la cuenta de almacenamiento a la versión de uso general v2.

    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/upgrade-storage-account.png" alt-text="Captura de pantalla que muestra cómo actualizar la cuenta de almacenamiento a la versión de uso general v2." lightbox="./media/data-flow-troubleshoot-connector-format/upgrade-storage-account.png"::: 
    

## <a name="snowflake"></a>Snowflake

### <a name="unable-to-connect-to-the-snowflake-linked-service"></a>No se puede conectar con el servicio vinculado Snowflake.

#### <a name="symptoms"></a>Síntomas

Se produce el siguiente error al crear el servicio vinculado Snowflake en la red pública y debe usar el entorno de ejecución de integración de resolución automática.

`ERROR [HY000] [Microsoft][Snowflake] (4) REST request for URL https://XXXXXXXX.east-us- 2.azure.snowflakecomputing.com.snowflakecomputing.com:443/session/v1/login-request?requestId=XXXXXXXXXXXXXXXXXXXXXXXXX&request_guid=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` 

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/connection-fail-error.png" alt-text="Captura de pantalla que muestra el error de conexión."::: 

#### <a name="cause"></a>Causa

No ha aplicado el nombre de la cuenta en el formato que se proporciona en el documento de la cuenta de Snowflake (incluidos los segmentos adicionales que identifican la región y la plataforma de nube), por ejemplo, `XXXXXXXX.east-us-2.azure`. Para más información, puede consultar este documento: [Propiedades del servicio vinculado](./connector-snowflake.md#linked-service-properties).

#### <a name="recommendation"></a>Recomendación

Para solucionar el problema, cambie el formato del nombre de la cuenta. El rol debe ser uno de los roles que se muestran en la siguiente imagen, pero el valor predeterminado es **Público**.

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/account-role.png" alt-text="Captura de pantalla que muestra los roles de cuenta."::: 

### <a name="sql-access-control-error-insufficient-privileges-to-operate-on-schema"></a>Error de control de acceso de SQL: "Privilegios insuficientes para operar en el esquema"

#### <a name="symptoms"></a>Síntomas

Cuando intenta usar "import projection" (Importar proyección), "data preview" (Vista previa de los datos), etc. en el origen de los flujos de datos de Snowflake, se producen errores como `net.snowflake.client.jdbc.SnowflakeSQLException: SQL access control error: Insufficient privileges to operate on schema`.

#### <a name="cause"></a>Causa

Este error se produce debido a una configuración incorrecta. Cuando se usa el flujo de datos para leer datos de Snowflake, la instancia de Azure Databricks del entorno de ejecución no selecciona directamente la consulta a Snowflake. En su lugar, se crea un almacenamiento provisional y Azure Databricks extrae los datos de las tablas en el almacenamiento provisional y, a continuación, los comprime y extrae. Este proceso se muestra en la imagen siguiente.

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/snowflake-data-read-model.png" alt-text=" Captura de pantalla que muestra el modelo de lectura de datos de Snowflake."::: 

Por lo tanto, el usuario o rol que se use en Azure Databricks debe tener los permisos necesarios para hacerlo en Snowflake. Pero normalmente el usuario o rol no los tienen, ya que la base de datos se crea en el recurso compartido. 

#### <a name="recommendation"></a>Recomendación
Para solucionar este problema, puede crear diferentes bases de datos y vistas sobre la base de datos compartida para acceder a ella desde Azure Databricks. Para más información, consulte [Snowflake](https://community.snowflake.com/s/question/0D50Z000095ktE4SAI/insufficient-privileges-to-operate-on-schema).

### <a name="failed-with-an-error-snowflakesqlexception-ip-xxxx-is-not-allowed-to-access-snowflake-contact-your-local-security-administrator"></a>No se pudo ejecutar la operación debido a un error: "SnowflakeSQLException: IP x.x.x.x no tiene permiso para acceder a Snowflake. Póngase en contacto con el administrador de seguridad local"

#### <a name="symptoms"></a>Síntomas

Cuando utiliza Snowflake en Azure Data Factory, puede usar correctamente la actividad de prueba de la conexión en el servicio vinculado Snowflake, la de obtener vista previa de los datos y la de importar esquema en el conjunto de datos de Snowflake, y la de ejecutar la copia, búsqueda u obtención de metadatos, u otras actividades. Pero cuando se usa Snowflake en la actividad de flujo de datos, es posible que se produzca un error como el siguiente: `SnowflakeSQLException: IP 13.66.58.164 is not allowed to access Snowflake. Contact your local security administrator.`.

#### <a name="cause"></a>Causa

El flujo de datos de Azure Data Factory no admite el uso de intervalos IP fijos y puede consultar [Direcciones IP de Azure Integration Runtime](./azure-integration-runtime-ip-addresses.md) para obtener información más detallada.

#### <a name="recommendation"></a>Recomendación

Para solucionar este problema, puede cambiar la configuración de firewall de la cuenta de Snowflake con los pasos siguientes:

1. Puede obtener la lista de intervalos IP de etiquetas de servicio en el "vínculo de descarga de intervalos IP de etiquetas de servicio": [Detección de etiquetas de servicio mediante archivos JSON descargables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/ip-range-list.png" alt-text="Captura de pantalla que muestra la lista de intervalos IP."::: 

1. Si ejecuta un flujo de datos en la región "southcentralus", debe permitir el acceso desde todas las direcciones con el nombre "AzureCloud.southcentralus", por ejemplo:

    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/allow-access-with-name.png" alt-text="Captura de pantalla que muestra cómo permitir el acceso desde todas las direcciones con un nombre determinado."::: 

### <a name="queries-in-the-source-does-not-work"></a>Las consultas en el origen no funcionan

#### <a name="symptoms"></a>Síntomas

Cuando intenta leer datos de Snowflake con una consulta, puede que se produzca este error:

1. `SQL compilation error: error line 1 at position 7 invalid identifier 'xxx'`
2. `SQL compilation error: Object 'xxx' does not exist or not authorized.`

#### <a name="cause"></a>Causa

Este error se produce debido a una configuración incorrecta.

#### <a name="recommendation"></a>Recomendación

En el caso de Snowflake, este aplica las siguientes reglas para almacenar los identificadores en el momento de la creación o definición y resolverlos en consultas y otras instrucciones SQL:

Cuando un identificador (nombre de tabla, nombre de esquema, nombre de columna, etc.) no tiene comillas, se almacena y resuelve en mayúsculas de forma predeterminada y no distingue entre mayúsculas y minúsculas. Por ejemplo:

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/unquoted-identifier.png" alt-text="Captura de pantalla que muestra el ejemplo de identificador sin comillas."lightbox="./media/data-flow-troubleshoot-connector-format/unquoted-identifier.png"::: 

Como no distingue entre mayúsculas y minúsculas, puede usar la siguiente consulta para leer datos de Snowflake aunque el resultado sea el mismo:<br/>
- `Select MovieID, title from Public.TestQuotedTable2`<br/>
- `Select movieId, title from Public.TESTQUOTEDTABLE2`<br/>
- `Select movieID, TITLE from PUBLIC.TESTQUOTEDTABLE2`<br/>

Cuando un identificador (nombre de tabla, nombre de esquema, nombre de columna, etc.) está entre comillas dobles, se almacena y resuelve exactamente como se ha especificado, incluidas las mayúsculas y minúsculas, ya que distingue entre ellas. En la siguiente imagen puede ver un ejemplo. Para más información, consulte este documento: [Requisitos de identificador](https://docs.snowflake.com/en/sql-reference/identifiers-syntax.html#identifier-requirements).

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/double-quoted-identifier.png" alt-text="Captura de pantalla que muestra el ejemplo de identificador entre comillas dobles." lightbox="./media/data-flow-troubleshoot-connector-format/double-quoted-identifier.png"::: 

Dado que el identificador que distingue mayúsculas de minúsculas (nombre de tabla, nombre de esquema, nombre de columna, etc.) incluye caracteres en minúsculas, debe entrecomillar el identificador durante la lectura de datos con la consulta, por ejemplo: <br/>

- Seleccione **"movieId",** **"title"** en Public. **"testQuotedTable2"** .

Si se produce un error con la consulta de Snowflake, compruebe si algunos identificadores (nombre de tabla, nombre de esquema, nombre de columna, etc.) distinguen mayúsculas de minúsculas con los pasos siguientes:

1. Inicie sesión en el servidor de Snowflake (`https://{accountName}.azure.snowflakecomputing.com/`, reemplace {accountName} por el nombre de su cuenta) para comprobar el identificador (nombre de tabla, nombre de esquema, nombre de columna, etc.).

1. Cree hojas de cálculo para probar y validar la consulta:
    - Ejecute `Use database {databaseName}` y reemplace {databaseName} por el nombre de su base de datos.
    - Ejecute una consulta con la tabla , por ejemplo: `select "movieId", "title" from Public."testQuotedTable2"`
    
1. Después de probar y validar la consulta SQL de Snowflake, puede usarla directamente en el origen de Snowflake del flujo de datos.

## <a name="azure-sql-database"></a>Azure SQL Database
 
### <a name="unable-to-connect-to-the-sql-database"></a>No se puede conectar con SQL Database

#### <a name="symptoms"></a>Síntomas

La instancia de Azure SQL Database funciona correctamente en la copia de datos, la obtención de la vista previa de los datos del conjunto de datos y la conexión de prueba del servicio vinculado, pero se produce un error cuando se usa la misma instancia de Azure SQL Database como origen o receptor en el flujo de datos. El error es este: `Cannot connect to SQL database: 'jdbc:sqlserver://powerbasenz.database.windows.net;..., Please check the linked service configuration is correct, and make sure the SQL database firewall allows the integration runtime to access`.

#### <a name="cause"></a>Causa

Hay una configuración de firewall incorrecta en el servidor de Azure SQL Database, por lo que el entorno de ejecución del flujo de datos no se puede conectar. En la actualidad, cuando intenta usar el flujo de datos para leer o escribir en Azure SQL Database, Azure Databricks se usa para crear el clúster de Spark para ejecutar el trabajo, pero este no admite intervalos IP fijos. Para más información, consulte [Direcciones IP de Azure Integration Runtime](./azure-integration-runtime-ip-addresses.md).

#### <a name="recommendation"></a>Recomendación

Compruebe la configuración del firewall de la instancia de Azure SQL Database y establézcala en "Permitir el acceso a servicios de Azure" en lugar de establecer el intervalo IP fijo.

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/allow-access-to-azure-service.png" alt-text="Captura de pantalla que muestra cómo permitir el acceso a servicios de Azure en la configuración del firewall."::: 

### <a name="syntax-error-when-using-queries-as-input"></a>Error de sintaxis al usar consultas como entrada

#### <a name="symptoms"></a>Síntomas

Cuando usa consultas como entrada en el origen del flujo de datos con Azure SQL, se produce el siguiente mensaje de error:

`at Source 'source1': shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: Incorrect syntax XXXXXXXX.`

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/error-detail.png" alt-text="Captura de pantalla que muestra los detalles del error."::: 

#### <a name="cause"></a>Causa

La consulta que se usa en el origen del flujo de datos se debe poder ejecutar como una subconsulta. El motivo del error es que la sintaxis de la consulta es incorrecta o no se puede ejecutar como una subconsulta. Puede ejecutar la siguiente consulta en SQL Server Management Studio para comprobarla:

`SELECT top(0) * from ($yourQuery) as T_TEMP`

#### <a name="recommendation"></a>Recomendación

Proporcione una consulta correcta y pruébela primero en SQL Server Management Studio.

### <a name="failed-with-an-error-sqlserverexception-111212-operation-cannot-be-performed-within-a-transaction"></a>No se pudo ejecutar la operación debido a un error: "SQLServerException: 111212; No se puede realizar la operación dentro de una transacción".

#### <a name="symptoms"></a>Síntomas

Cuando usa Azure SQL Database como receptor en el flujo de datos para obtener una vista previa de los datos, depurar o desencadenar una ejecución y realizar otras actividades, es posible que aparezca el siguiente mensaje de error en el trabajo:

`{"StatusCode":"DFExecutorUserError","Message":"Job failed due to reason: at Sink 'sink': shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: 111212;Operation cannot be performed within a transaction.","Details":"at Sink 'sink': shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: 111212;Operation cannot be performed within a transaction."}`

#### <a name="cause"></a>Causa
El error "`111212;Operation cannot be performed within a transaction.`" solo se produce en el grupo de SQL dedicado de Synapse. Sin embargo, se usa erróneamente la instancia de Azure SQL Database como conector en su lugar.

#### <a name="recommendation"></a>Recomendación
Confirme si la instancia de SQL Database es un grupo de SQL dedicado de Synapse. Si es así, use Azure Synapse Analytics como conector como se indica en la imagen siguiente.

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/synapse-analytics-connector.png" alt-text="Captura de pantalla que muestra el conector de Azure Synapse Analytics."::: 

### <a name="data-with-the-decimal-type-become-null"></a>Los datos con el tipo decimal se convierten en NULL

#### <a name="symptoms"></a>Síntomas

Quiere insertar datos en una tabla de la base de datos SQL. Si los datos contienen el tipo decimal y se deben insertar en una columna con este tipo de la base de datos SQL, puede que el valor de los datos cambie a NULL.

Si obtiene la vista previa, en fases anteriores, se mostrará un valor como el de la siguiente imagen:

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/value-in-previous-stage.png" alt-text="Captura de pantalla que muestra el valor en las fases anteriores."::: 

En la fase del receptor, se convertirá en NULL, como se indica en la imagen siguiente.

:::image type="content" source="./media/data-flow-troubleshoot-connector-format/value-in-sink-stage.png" alt-text="Captura de pantalla que muestra el valor en la fase del receptor."::: 

#### <a name="cause"></a>Causa
El tipo decimal tiene propiedades de escala y precisión. Si el tipo de datos no coincide con el de la tabla del receptor, el sistema validará que el decimal de destino es más amplio que el decimal original y que el valor original no se desborda en el decimal de destino. Por lo tanto, el valor se convertirá en NULL.

#### <a name="recommendation"></a>Recomendación
Compruebe y compare el tipo decimal entre los datos y la tabla de la base de datos SQL, y modifique la escala y la precisión a los mismos valores.

Puede usar toDecimal (IDecimal, escala, precisión) para averiguar si los datos originales se pueden convertir a la escala y precisión de destino. Si devuelve NULL, significa que los datos no se pueden convertir ni hacer avanzar al insertar.

## <a name="adls-gen2"></a>ADLS Gen2

### <a name="failed-with-an-error-error-while-reading-file-xxx-it-is-possible-the-underlying-files-have-been-updated"></a>No se pudo ejecutar la operación debido a un error: "Error al leer el archivo XXX. Es posible que se hayan actualizado los archivos subyacentes"

#### <a name="symptoms"></a>Síntomas

Si usa ADLS Gen2 como receptor en el flujo de datos (para obtener una vista previa de los datos, depurar o desencadenar una ejecución, etc.) y la configuración de partición de la pestaña **Optimizar** de la fase **Receptor** no es la predeterminada, es posible que se produzca un error en el trabajo con el siguiente mensaje de error:

`Job failed due to reason: Error while reading file abfss:REDACTED_LOCAL_PART@prod.dfs.core.windows.net/import/data/e3342084-930c-4f08-9975-558a3116a1a9/part-00000-tid-7848242374008877624-5df7454e-7b14-4253-a20b-d20b63fe9983-1-1-c000.csv. It is possible the underlying files have been updated. You can explicitly invalidate the cache in Spark by running 'REFRESH TABLE tableName' command in SQL or by recreating the Dataset/DataFrame involved.`

#### <a name="cause"></a>Causa

1. No se asigna un permiso adecuado a la autenticación de identidad administrada o de entidad de servicio.
1. Es posible que tenga un trabajo personalizado para controlar los archivos que no desea, lo que afectará a la salida intermedia del flujo de datos.

#### <a name="recommendation"></a>Recomendación
1. Compruebe si el servicio vinculado tiene los permisos de lectura, escritura y edición para Gen2. Si usa la autenticación de identidad administrada o de entidad de servicio, conceda al menos el rol Colaborador de datos de Storage Blob en el control de acceso (IAM).
1. Confirme si tiene trabajos específicos que eliminan o trasladan archivos a otro lugar cuyo nombre no coincide con la regla. Dado que los flujos de datos escribirán los archivos de partición en la carpeta de destino en primer lugar y, después, realizarán las operaciones de combinación y cambio de nombre, es posible que el nombre del archivo intermedio no coincida con la regla.

## <a name="adls-gen1"></a>ADLS Gen1

### <a name="fail-to-create-files-with-service-principle-authentication"></a>No se pueden crear archivos con la autenticación de entidad de servicio

#### <a name="symptoms"></a>Síntomas
Cuando intenta trasladar o transferir datos de diferentes orígenes al receptor de ADLS gen1, si el método de autenticación del servicio vinculado es la autenticación de entidad de servicio, se puede producir el siguiente error en el trabajo:

`org.apache.hadoop.security.AccessControlException: CREATE failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.). [2b5e5d92-xxxx-xxxx-xxxx-db4ce6fa0487] failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.)`

#### <a name="cause"></a>Causa

Los permisos de lectura, escritura y ejecución, o la propiedad del conjunto de datos, no están establecidos correctamente.

#### <a name="recommendation"></a>Recomendación

- Si la carpeta de destino no tiene los permisos correctos, consulte este documento para asignar el permiso correcto en Gen1: [Uso de la autenticación de entidad de servicio](./connector-azure-data-lake-store.md#use-service-principal-authentication).

- Si la carpeta de destino tiene el permiso correcto y usa la propiedad de nombre de archivo en el flujo de datos para dirigirse a la carpeta y al nombre de archivo correctos, pero la propiedad de ruta de acceso del archivo del conjunto de datos no está establecida en la ruta de acceso del archivo de destino (que normalmente está sin configurar), como se indica en el ejemplo que se muestra en las imágenes siguientes, se produce este error porque el sistema back-end intenta crear archivos basados en la ruta de acceso del conjunto de datos y esta no tiene el permiso correcto.
    
    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/file-path-property.png" alt-text="Captura de pantalla que muestra la propiedad de ruta de acceso del archivo"::: 
    
    :::image type="content" source="./media/data-flow-troubleshoot-connector-format/file-name-property.png" alt-text="Captura de pantalla que muestra la propiedad de nombre del archivo"::: 

    
    Hay dos métodos para solucionar este problema:
    1. Asigne los permisos de escritura y ejecución a la ruta de acceso del archivo del conjunto de datos.
    1. Establezca la ruta de acceso del archivo del conjunto de datos como la carpeta con permisos de escritura y ejecución, y establezca la ruta de acceso de la carpeta rest y el nombre de archivo en los flujos de datos.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más ayuda para solucionar problemas, consulte estos recursos:

*  [Solución de problemas de los flujos de datos de asignación en Azure Data Factory](data-flow-troubleshoot-guide.md)
*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)