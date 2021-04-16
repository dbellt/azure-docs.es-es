---
title: Solución de problemas de los flujos de datos de asignación
description: Obtenga información sobre la solución de problemas relacionados con flujos de datos en Azure Data Factory.
ms.author: makromer
author: kromerm
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 03/25/2021
ms.openlocfilehash: 72ab685b58f7d940fe4d682cacba6212fe80ced8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105933090"
---
# <a name="troubleshoot-mapping-data-flows-in-azure-data-factory"></a>Solución de problemas de los flujos de datos de asignación en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran métodos comunes de solución de problemas de flujos de datos de asignación en Azure Data Factory.

## <a name="common-error-codes-and-messages"></a>Mensajes y códigos de error comunes 

### <a name="error-code-df-executor-sourceinvalidpayload"></a>Código de error: DF-Executor-SourceInvalidPayload
- **Mensaje**: Data preview, debug, and pipeline data flow execution failed because container does not exist (Error en la ejecución del flujo de datos de la depuración, la canalización y la vista previa de datos porque el contenedor no existe)
- **Causa**: Un conjunto de datos contiene un contenedor que no existe en el almacenamiento.
- **Recomendación**: Asegúrese de que el contenedor al que se hace referencia en el conjunto de datos exista o se pueda acceder.

### <a name="error-code-df-executor-systeminvalidjson"></a>Código de error: DF-Executor-SystemInvalidJson

- **Mensaje**: JSON parsing error, unsupported encoding or multiline (Error de análisis de JSON, codificación no compatible o multilínea)
- **Causa**: Posibles problemas con el archivo JSON, como codificación no admitida, bytes dañados o uso del origen JSON como un único documento en muchas líneas anidadas.
- **Recomendación**: Verifique que se admite la codificación del archivo JSON. En la transformación de origen que usa un conjunto de datos JSON, expanda **JSON Settings** (Configuración de JSON) y active **Documento único**.
 
### <a name="error-code-df-executor-broadcasttimeout"></a>Código de error: DF-Executor-BroadcastTimeout

- **Mensaje**: Broadcast join timeout error, make sure broadcast stream produces data within 60 secs in debug runs and 300 secs in job runs (Error de tiempo de espera de combinación de difusión, asegúrese de que el flujo de difusión genera datos en un plazo de 60 segundos en ejecuciones de depuración y de 300 segundos en ejecuciones de trabajos).
- **Causa**: La difusión tiene un tiempo de espera predeterminado de 60 segundos en ejecuciones de depuración y de 300 segundos en ejecuciones de trabajos. El flujo elegido para la difusión es demasiado grande para generar datos dentro de este límite.
- **Recomendación**: Marque la pestaña **Optimizar** en las transformaciones de flujo de datos para combinación, existe y búsqueda. La opción predeterminada para la difusión es **Auto**. Si se establece en **Auto** o si configura manualmente el lado izquierdo o derecho para difundir en **Fijo**, puede establecer una configuración mayor de Azure Integration Runtime (IR) o desactivar la difusión. Para el mejor rendimiento en los flujos de datos, se recomienda permitir que Spark realice la difusión con **Auto** y use una instancia de Azure IR optimizada para memoria. 
 
  Si ejecuta el flujo de datos en una ejecución de prueba de depuración desde una ejecución de canalización de depuración, puede ejecutar en esta condición con mayor frecuencia. Esto se debe a que Azure Data Factory limita el tiempo de espera de la difusión a 60 segundos para mantener una experiencia de depuración más rápida. Puede ampliar el tiempo de espera hasta el tiempo de espera de 300 segundos de una ejecución desencadenada. Para ello, puede usar la opción **Depurar** > **Use Activity Runtime** (Usar tiempo de ejecución de actividad) para usar la instancia de Azure IR definida en la actividad de ejecutar canalización de flujo de datos.

- **Mensaje**: Broadcast join timeout error, you can choose 'Off' of broadcast option in join/exists/lookup transformation to avoid this issue. If you intend to broadcast join option to improve performance then make sure broadcast stream can produce data within 60 secs in debug runs and 300 secs in job runs (Error de tiempo de espera de combinación de difusión, puede elegir "Desactivado" en la opción de difusión en la transformación join/exists/lookup para evitar este problema. Si tiene previsto difundir la opción de combinación para mejorar el rendimiento, asegúrese de que el flujo de difusión puede generar datos en un plazo de 60 segundos en ejecuciones de depuración y de 300 segundos en ejecuciones de trabajos).
- **Causa**: La difusión tiene un tiempo de espera predeterminado de 60 segundos en ejecuciones de depuración y de 300 segundos en ejecuciones de trabajos. En la combinación de difusión, el flujo elegido para la difusión es demasiado grande para generar datos dentro de este límite. Si no se usa una combinación de difusión, la difusión predeterminada realizada por el flujo de datos puede alcanzar el mismo límite.
- **Recomendación**: Desactive la opción de difusión o evite la difusión de flujos de datos de gran tamaño para los que el procesamiento pueda tardar más de 60 segundos. Elija un flujo más pequeño para la difusión. Los archivos de código fuente y las tablas de Azure SQL Data Warehouse grandes no suelen ser opciones adecuadas. En ausencia de una combinación de difusión, use un clúster mayor si se produce el error.

### <a name="error-code-df-executor-conversion"></a>Código de error: DF-Executor-Conversion

- **Mensaje**: Converting to a date or time failed due to an invalid character (Error al realizar la conversión a una fecha o hora porque un carácter no es válido).
- **Causa**: Los datos no tienen el formato esperado.
- **Recomendación:** Use el tipo de datos correcto.

### <a name="error-code-df-executor-invalidcolumn"></a>Código de error: DF-Executor-InvalidColumn
- **Mensaje**: Column name needs to be specified in the query, set an alias if using a SQL function (Es necesario especificar el nombre de la columna en la consulta; establezca un alias si usa una función SQL).
- **Causa**: No se ha especificado ningún nombre de columna.
- **Recomendación**: Establezca un alias si usa una función SQL como min () o max().

### <a name="error-code-df-executor-drivererror"></a>Código de error: DF-Executor-DriverError
- **Mensaje**: INT96 es un tipo de marca de tiempo heredado que no es compatible con el flujo de datos de ADF. Considere la posibilidad de actualizar el tipo de columna a los tipos más recientes.
- **Causa**: Error del controlador.
- **Recomendación**: INT96 es un tipo de marca de tiempo heredado que no es compatible con el flujo de datos de Azure Data Factory. Considere la posibilidad de actualizar el tipo de columna al tipo más reciente.

### <a name="error-code-df-executor-blockcountexceedslimiterror"></a>Código de error: DF-Executor-BlockCountExceedsLimitError
- **Mensaje**: el recuento de bloques sin confirmar no puede superar el límite máximo de 100 000 bloques. Compruebe la configuración de blobs.
- **Causa**: El número máximo de bloques sin confirmar en un blob es de 100 000.
- **Recomendación**: Póngase en contacto con el equipo del producto de Microsoft para obtener más detalles sobre este problema.

### <a name="error-code-df-executor-partitiondirectoryerror"></a>Código de error: DF-Executor-PartitionDirectoryError
- **Mensaje**: la ruta de origen especificada tiene varios directorios con particiones (por ejemplo, <Source Path>/<Directorio raíz de la partición 1>/a=10/b=20, <Source Path>/<Directorio raíz de la partición 2>/c=10/d=30) o un directorio con particiones con otro archivo o directorio sin particiones (por ejemplo, <Source Path>/<Directorio raíz con particiones 1>/a=10/b=20, <Source Path>/Directorio 2/archivo1). Quite el directorio raíz con particiones de la ruta de origen y léalo a través de una transformación de origen independiente.
- **Causa**: La ruta de origen tiene varios directorios con particiones o un directorio con particiones que tiene otro archivo o directorio sin particiones.
- **Recomendación**: Quite el directorio raíz con particiones de la ruta de origen y léalo a través de una transformación de origen independiente.

### <a name="error-code-df-executor-invalidtype"></a>Código de error: DF-Executor-InvalidType
- **Mensaje**: asegúrese de que el tipo de parámetro coincide con el tipo de valor pasado. Actualmente no se admite el paso de parámetros float desde canalizaciones.
- **Causa**: El tipo de datos para el tipo declarado no es compatible con el valor de parámetro real.
- **Recomendación**: Compruebe que los valores de parámetro pasados al flujo de datos coinciden con el tipo declarado.

### <a name="error-code-df-executor-parseerror"></a>Código de error: DF-Executor-ParseError
- **Mensaje**: no puede analizarse la expresión
- **Causa**: Una expresión generó errores de análisis debido a un formato incorrecto.
- **Recomendación**: Compruebe el formato de la expresión.

### <a name="error-code-df-executor-systemimplicitcartesian"></a>Código de error: DF-Executor-SystemImplicitCartesian
- **Mensaje**: Implicit cartesian product for INNER join is not supported, use CROSS JOIN instead. Columns used in join should create a unique key for rows. (No se admite el producto cartesiano implícito para la combinación interna; utilice la combinación cruzada en su lugar. Las columnas utilizadas en la combinación deben crear una clave única para las filas).
- **Causa**: No se admiten los productos cartesianos implícitos para combinaciones internas entre planes lógicos. Si utiliza columnas en la combinación, cree una clave única.
- **Recomendación**: En el caso de las combinaciones que se basan en la no igualdad, use la combinación cruzada.

### <a name="error-code-getcommand-outputasync-failed"></a>Código de error: Error de GetCommand OutputAsync
- **Mensaje**: Durante la depuración de Data Flow y la vista previa de datos: Error de GetCommand OutputAsync con...
- **Causa**: Se trata de un error del servicio de back-end. 
- **Recomendación**: Vuelva a intentar la operación y reinicie la sesión de depuración. Si con ello no se resuelve el problema, póngase en contacto con el servicio de soporte al cliente. 

### <a name="error-code-df-executor-outofmemoryerror"></a>Código de error: DF-Executor-OutOfMemoryError
 
- **Mensaje**: el clúster ha tenido un problema de memoria insuficiente durante la ejecución. Vuelva a intentar usar un entorno de ejecución de integración con un mayor recuento de núcleos o un tipo de proceso optimizado para memoria
- **Causa**: El clúster se está quedando sin memoria.
- **Recomendación**: Los clústeres de depuración están diseñados para el desarrollo. Use el muestreo de datos y un tipo y tamaño de proceso adecuados para ejecutar la carga. Para obtener sugerencias de rendimiento, vea [Guía del rendimiento del flujo de datos de asignación](concepts-data-flow-performance.md).

### <a name="error-code-df-executor-illegalargument"></a>Código de error: DF-Executor-illegalArgument

- **Mensaje**: asegúrese de que la clave de acceso del servicio vinculado es correcta
- **Causa**: El nombre de cuenta o la clave de acceso son incorrectos.
- **Recomendación:** asegúrese de que el nombre de cuenta o la clave de acceso especificados en el servicio vinculado son correctos. 

### <a name="error-code-df-executor-columnunavailable"></a>Código de error: DF-Executor-ColumnUnavailable
- **Mensaje**: El nombre de columna usado en la expresión no está disponible o no es válido.
- **Causa**: Se usa un nombre de columna no válido o no disponible en una expresión.
- **Recomendación**: Compruebe los nombres de columna usados en las expresiones.

 ### <a name="error-code-df-executor-outofdiskspaceerror"></a>Código de error: DF-Executor-OutOfDiskSpaceError
- **Mensaje**: Error interno del servidor
- **Causa**: El clúster se está quedando sin espacio en disco.
- **Recomendación**: Vuelva a ejecutar la canalización. Si esto no resuelve el problema, póngase en contacto con el servicio de asistencia al cliente.


 ### <a name="error-code-df-executor-storeisnotdefined"></a>Código de error: DF-Executor-StoreIsNotDefined
- **Mensaje**: The store configuration is not defined. This error is potentially caused by invalid parameter assignment in the pipeline (La configuración del almacén no está definida. Este error se puede deber a una asignación de parámetros no válida en la canalización).
- **Causa**: Sin determinar.
- **Recomendación**: Compruebe la asignación de valores de parámetros en la canalización. Una expresión de parámetros puede contener caracteres no válidos.


### <a name="error-code-4502"></a>Código de error: 4502
- **Mensaje**: Hay importantes ejecuciones de MappingDataflow simultáneas que están causando errores debido a la limitación en el entorno de ejecución de integración.
- **Causa**: Un gran número de ejecuciones de actividad de flujo de datos se producen simultáneamente en el entorno de ejecución de integración. Para más información, consulte [Límites de Data Factory](../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits).
- **Recomendación**: Si desea ejecutar más actividades de flujo de datos en paralelo, distribúyalas en varios entornos de ejecución de integración.


### <a name="error-code-invalidtemplate"></a>Código de error: InvalidTemplate
- **Mensaje**: no se puede evaluar la expresión de canalización.
- **Causa**: La expresión de canalización pasada en la actividad de flujo de datos no se está procesando correctamente debido a un error de sintaxis.
- **Recomendación**: Compruebe la actividad en la supervisión de actividades para comprobar la expresión.

### <a name="error-code-2011"></a>Código de error: 2011
- **Mensaje**: la actividad se estaba ejecutando en Azure Integration Runtime y no pudo descifrar la credencial del almacén de datos o el proceso conectado a través de un entorno de ejecución de integración autohospedado. Compruebe la configuración de los servicios vinculados asociados con esta actividad y asegúrese de usar el tipo de entorno de ejecución de integración adecuado.
- **Causa**: El flujo de datos no es compatible con los servicios vinculados en los entornos de ejecución de integración autohospedados.
- **Recomendación**: Configure el flujo de datos para que se ejecute en el entorno de ejecución de integración de red virtual administrada.

### <a name="error-code-df-xml-invalidvalidationmode"></a>Código de error: DF-Xml-InvalidValidationMode
- **Mensaje**: Se proporciona un modo de validación XML no válido.
- **Recomendación**: Compruebe el valor del parámetro y especifique el modo de validación adecuado.

### <a name="error-code-df-xml-invaliddatafield"></a>Código de error: DF-Xml-InvalidDataField
- **Mensaje**: El campo de los registros dañados debe ser de tipo cadena y admitir valores NULL.
- **Recomendación**: Asegúrese de que la columna `\"_corrupt_record\"` del proyecto de origen tiene un tipo de datos de cadena.

### <a name="error-code-df-xml-malformedfile"></a>Código de error: DF-Xml-MalformedFile
- **Mensaje**: XML con formato incorrecto en "FailFastMode".
- **Recomendación**: Actualice el contenido del archivo XML con el formato correcto.

### <a name="error-code-df-xml-invaliddatatype"></a>Código de error: DF-Xml-InvalidDataType
- **Message**: El elemento XML tiene elementos secundarios o atributos y no se puede convertir.

### <a name="error-code-df-xml-invalidreferenceresource"></a>Código de error: DF-XML-InvalidReferenceResource
- **Mensaje**: No se puede resolver el recurso de referencia en el archivo de datos XML.
- **Recomendación**: Debe comprobar el recurso de referencia en el archivo de datos XML.

### <a name="error-code-df-xml-invalidschema"></a>Código de error: DF-Xml-InvalidSchema
- **Mensaje**: Error en la validación del esquema.

### <a name="error-code-df-xml-unsupportedexternalreferenceresource"></a>Código de error: DF-XML-UnsupportedExternalReferenceResource
- **Mensaje**: No se admite el recurso de referencia externa en el archivo de datos XML.
- **Recomendación**: Actualice el contenido del archivo XML cuando el recurso de referencia externa no se admita ahora.

### <a name="error-code-df-gen2-invalidaccountconfiguration"></a>Código de error: DF-GEN2-InvalidAccountConfiguration
- **Message**: Se debe especificar la clave de cuenta o bien tenant/spnId/spnCredential/spnCredentialType o miServiceUri/miServiceToken.
- **Recomendación**: Configure la cuenta correcta en el servicio vinculado GEN2.

### <a name="error-code-df-gen2-invalidauthconfiguration"></a>Código de error: DF-GEN2-InvalidAuthConfiguration
- **Mensaje**: Solo se puede especificar uno de los tres métodos de autenticación (Key, ServicePrincipal y MI). 
- **Recomendación**: elija el tipo de autenticación adecuado en el servicio vinculado de GEN2.

### <a name="error-code-df-gen2-invalidserviceprincipalcredentialtype"></a>Código de error: DF-GEN2-InvalidServicePrincipalCredentialType
- **Mensaje**: ServicePrincipalCredentialType no es válido.

### <a name="error-code-df-gen2-invaliddatatype"></a>Código de error: DF-GEN2-InvalidDataType
- **Mensaje**: el tipo de nube no es válido.

### <a name="error-code-df-blob-invalidaccountconfiguration"></a>Código de error: DF-Blob-InvalidAccountConfiguration
- **Message**: Se debe especificar la clave de cuenta o sas_token.

### <a name="error-code-df-blob-invalidauthconfiguration"></a>Código de error: DF-BLOB-InvalidAuthConfiguration
- **Mensaje**: Solo se puede especificar uno de los dos métodos de autenticación (Key, SAS).

### <a name="error-code-df-blob-invaliddatatype"></a>Código de error: DF-Blob-InvalidDataType
- **Mensaje**: El tipo de nube no es válido.

### <a name="error-code-df-cosmos-partitionkeymissed"></a>Código de error: DF-Cosmos-PartitionKeyMissed
- **Mensaje**: Se debe especificar la ruta de acceso de la clave de partición para las operaciones de actualización y eliminación.
- **Recomendación**: Use la clave de partición de que se proporciona en la configuración del receptor de Cosmos.

### <a name="error-code-df-cosmos-invalidpartitionkey"></a>Código de error: DF-Cosmos-InvalidPartitionKey
- **Mensaje**: La ruta de acceso de la clave de partición no puede estar vacía para las operaciones de actualización y eliminación.
- **Recomendación**: Use la clave de partición de que se proporciona en la configuración del receptor de Cosmos.

### <a name="error-code-df-cosmos-idpropertymissed"></a>Código de error: DF-Cosmos-IdPropertyMissed
- **Mensaje**: la propiedad "id" debe estar asignada para operaciones de eliminación y actualización.
- **Recomendación**: Asegúrese de que los datos de entrada tienen una columna `id` en la configuración del receptor de Cosmos. Si no es así, use **la transformación de selección o derivación** para generar esta columna antes del receptor.

### <a name="error-code-df-cosmos-invalidpartitionkeycontent"></a>Código de error: DF-Cosmos-InvalidPartitionKeyContent
- **Mensaje**: la clave de partición debe comenzar con /.
- **Recomendación**: haga que la clave de partición empiece por `/` en la configuración del receptor de Cosmos, por ejemplo: `/movieId`.

### <a name="error-code-df-cosmos-invalidpartitionkey"></a>Código de error: DF-Cosmos-InvalidPartitionKey
- **Mensaje**: partitionKey no asignado en el receptor para las operaciones de eliminación y actualización.
- **Recomendación**: en la configuración del receptor de Cosmos, use la clave de partición que es la misma que la clave de partición del contenedor.

### <a name="error-code-df-cosmos-invalidconnectionmode"></a>Código de error: DF-Cosmos-InvalidConnectionMode
- **Mensaje**: connectionMode no válido.
- **Recomendación**: confirme que el modo compatible es **Gateway** y **DirectHttps** en configuración de Cosmos.

### <a name="error-code-df-cosmos-invalidaccountconfiguration"></a>Código de error: DF-Cosmos-InvalidAccountConfiguration
- **Mensaje**: Se debe especificar AccountName o accountEndpoint.

### <a name="error-code-df-github-writenotsupported"></a>Código de error: DF-Github-WriteNotSupported
- **Mensaje**: El almacén de Github no permite operaciones de escritura.

### <a name="error-code-df-pgsql-invalidcredential"></a>Código de error: DF-PGSQL-InvalidCredential
- **Mensaje**: Se debe especificar el usuario o la contraseña.
- **Recomendación**: Asegúrese de que tiene la configuración de credenciales correcta en el servicio vinculado de postgresql relacionado.

### <a name="error-code-df-snowflake-invalidstageconfiguration"></a>Código de error: DF-Snowflake-InvalidStageConfiguration
- **Mensaje**: Solo se puede usar el tipo de almacenamiento de blobs como ubicación temporal en la operación de lectura y escritura de Snowflake.

### <a name="error-code-df-snowflake-invalidstageconfiguration"></a>Código de error: DF-Snowflake-InvalidStageConfiguration
- **Mensaje**: Las propiedades del almacenamiento temporal de Snowflake deben especificarse con la autenticación azure blob + sas.

### <a name="error-code-df-snowflake-invaliddatatype"></a>Código de error: DF-copo de nieve-InvalidDataType
- **Mensaje**: El tipo de Spark no se admite en Snowflake.
- **Recomendación**: Use la **transformación de derivación** para cambiar la columna relacionada de datos de entrada en el tipo de cadena antes del receptor de Snowflake. 

### <a name="error-code-df-hive-invalidblobstagingconfiguration"></a>Código de error: DF-Hive-InvalidBlobStagingConfiguration
- **Mensaje**: Se deben especificar las propiedades de almacenamiento provisional de Blob Storage.

### <a name="error-code-df-hive-invalidgen2stagingconfiguration"></a>Código de error: DF-Hive-InvalidGen2StagingConfiguration
- **Mensaje**: El almacenamiento provisional del almacenamiento de ADLS Gen2 solo admite la credenciales de clave de la entidad de servicio.
- **Recomendación**: Confirme que aplica la credencial de clave de entidad de servicio en el servicio vinculado ADLS Gen2 que se usa como almacenamiento provisional.

### <a name="error-code-df-hive-invalidgen2stagingconfiguration"></a>Código de error: DF-Hive-InvalidGen2StagingConfiguration
- **Mensaje**: Se deben especificar las propiedades de almacenamiento provisional de ADLS Gen2. Se requiere la clave o tenant/spnId/spnKey o miServiceUri/miServiceToken.
- **Recomendación**: Aplique la credencial adecuada que se usa como almacenamiento provisional en el subárbol del servicio vinculado de ADLS Gen2 relacionado. 

### <a name="error-code-df-hive-invaliddatatype"></a>Código de error: DF-Hive-InvalidDataType
- **Mensaje**: Columnas no admitidas.
- **Recomendación**: Actualice la columna de datos de entrada para que coincida con el tipo de datos admitido por el subárbol.

### <a name="error-code-df-hive-invalidstoragetype"></a>Código de error: DF-Hive-InvalidStorageType
- **Mensaje**: El tipo de almacenamiento puede ser blob o gen2.

### <a name="error-code-df-delimited-invalidconfiguration"></a>Código de error: DF-Delimited-InvalidConfiguration
- **Message**: Se debe especificar una de las líneas vacías o el encabezado personalizado.
- **Recomendación**: Especifique líneas vacías o encabezados personalizados en la configuración de CSV.

### <a name="error-code-df-delimited-columndelimitermissed"></a>Código de error: DF-Delimited-ColumnDelimiterMissed
- **Mensaje**: Se requiere un delimitador de columna para el análisis.
- **Recomendación**: Confirme que tiene el delimitador de columna en la configuración de CSV.

### <a name="error-code-df-mssql-invalidcredential"></a>Código de error: DF-MSSQL-InvalidCredential
- **Message**: se debe especificar user/pwd o bien tenant/spnId/spnKey o miServiceUri/miServiceToken.
- **Recomendación**: Aplique las credenciales correctas en el servicio vinculado MSSQL relacionado.

### <a name="error-code-df-mssql-invaliddatatype"></a>Código de error: DF-MSSQL-InvalidDataType
- **Mensaje**: Campos no admitidos.
- **Recomendación**: Modifique la columna de datos de entrada para que coincida con el tipo de datos admitido por MSSQL.

### <a name="error-code-df-mssql-invalidauthconfiguration"></a>Código de error: DF-MSSQL-InvalidAuthConfiguration
- **Mensaje**: Solo se puede especificar uno de los tres métodos de autenticación (Key, ServicePrincipal y MI).
- **Recomendación**: Solo puede especificar uno de los tres métodos de autenticación (Key, ServicePrincipal and MI) en el servicio vinculado MSSQL relacionado.

### <a name="error-code-df-mssql-invalidcloudtype"></a>Código de error: DF-MSSQL-InvalidCloudType
- **Mensaje**: El tipo de nube no es válido.
- **Recomendación**: Compruebe el tipo de nube en el servicio vinculado MSSQL relacionado.

### <a name="error-code-df-sqldw-invalidblobstagingconfiguration"></a>Código de error: DF-SQLDW-InvalidBlobStagingConfiguration
- **Mensaje**: Se deben especificar las propiedades de almacenamiento provisional de Blob Storage.

### <a name="error-code-df-sqldw-invalidstoragetype"></a>Código de error: DF-SQLDW-InvalidStorageType
- **Mensaje**: El tipo de almacenamiento puede ser blob o gen2.

### <a name="error-code-df-sqldw-invalidgen2stagingconfiguration"></a>Código de error: DF-SQLDW-InvalidGen2StagingConfiguration
- **Mensaje**: El almacenamiento provisional del almacenamiento de ADLS Gen2 solo admite la credenciales de clave de la entidad de servicio.

### <a name="error-code-df-sqldw-invalidconfiguration"></a>Código de error: DF-SQLDW-InvalidConfiguration
- **Mensaje**: Se deben especificar las propiedades de almacenamiento provisional de ADLS Gen2. Se requiere la clave o bien enant/spnId/spnCredential/spnCredentialType o miServiceUri/miServiceToken.

### <a name="error-code-df-delta-invalidconfiguration"></a>Código de error: DF-DELTA-InvalidConfiguration
- **Mensaje**: No se puede establecer la marca de tiempo y la versión al mismo tiempo.

### <a name="error-code-df-delta-keycolumnmissed"></a>Código de error: DF-DELTA-KeyColumnMissed
- **Mensaje**: Se deben especificar columnas de clave para las operaciones no insertables.

### <a name="error-code-df-delta-invalidtableoperationsettings"></a>Código de error: DF-DELTA-InvalidTableOperationSettings
- **Mensaje**: No se pueden especificar las opciones de volver a crear y truncar.

### <a name="error-code-df-excel-worksheetconfigmissed"></a>Código de error: DF-Excel-WorksheetConfigMissed
- **Mensaje**: Excel sheet name or index is required (El nombre o el índice de la hoja de Excel es obligatorio).
- **Recomendación**: Compruebe el valor del parámetro y especifique el nombre de la hoja o el índice para leer los datos de Excel.

### <a name="error-code-df-excel-invalidworksheetconfiguration"></a>Código de error: DF-Excel-InvalidWorksheetConfiguration
- **Mensaje**: Excel sheet name and index cannot exist at the same time (El nombre y el índice de la hoja de Excel no pueden existir al mismo tiempo).
- **Recomendación**: Compruebe el valor del parámetro y especifique el nombre de la hoja o el índice para leer los datos de Excel.

### <a name="error-code-df-excel-invalidrange"></a>Código de error: DF-Excel-InvalidRange
- **Mensaje**: Invalid range is provided (Se proporcionó un intervalo no válido).
- **Recomendación**: Compruebe el valor del parámetro y especifique el intervalo válido mediante la siguiente referencia: [Formato de Excel en Azure Data Factory - Propiedades del conjunto de datos](./format-excel.md#dataset-properties).

### <a name="error-code-df-excel-worksheetnotexist"></a>Código de error: DF-Excel-WorksheetNotExist
- **Mensaje**: Excel worksheet does not exist (La hoja de cálculo de Excel no existe).
- **Recomendación**: Compruebe el valor del parámetro y especifique el nombre de la hoja válida o el índice para leer los datos de Excel.

### <a name="error-code-df-excel-differentschemanotsupport"></a>Código de error: DF-Excel-DifferentSchemaNotSupport
- **Mensaje**: En este momento no se admite la lectura de archivos de Excel con esquemas diferentes.

### <a name="error-code-df-excel-invaliddatatype"></a>Código de error: DF-Excel-InvalidDataType
- **Mensaje**: Tipo de datos no admitido.

### <a name="error-code-df-excel-invalidfile"></a>Código de error: DF-Excel-InvalidFile
- **Mensaje**: Se proporcionó un archivo de Excel no válido mientras que solo se admiten .xlsx y .xls.


## <a name="miscellaneous-troubleshooting-tips"></a>Consejos de solución de problemas varios
- **Problema**: Se produjo una excepción inesperada y un error en la ejecución.
    - **Mensaje**: Durante la ejecución de una actividad de Data Flow: Se obtuvo una excepción inesperada y se produjo un error de ejecución.
    - **Causa**: Se trata de un error del servicio de back-end. Vuelva a intentar la operación y reinicie la sesión de depuración.
    - **Recomendación**: Si reintenta y reinicia y no se resuelve el problema, póngase en contacto con el servicio de soporte al cliente.

-  **Problema**: No hay datos de salida en la combinación durante la vista previa de los datos de depuración.
    - **Mensaje**: hay un gran número de valores NULL o valores que faltan, lo que puede deberse a que no se han muestreado suficientes filas. Pruebe a actualizar el límite de filas de depuración y los datos.
    - **Causa**: La condición de combinación no ha coincidido con ninguna fila o dio como resultado un gran número de valores NULL durante la vista previa de los datos.
    - **Recomendación**: En **Configuración de depuración**, aumente el número de filas del límite de filas de origen. Asegúrese de seleccionar una instancia de Azure IR que tenga un clúster de flujo de datos lo suficientemente grande como para controlar más datos.
    
- **Problema**: Error de validación en el origen con archivos CSV de varias líneas. 
    - **Mensaje**: Puede ver uno de estos mensajes de error:
        - Falta la última columna o es NULL.
        - Error en la validación del esquema en el origen.
        - La importación de esquema no se muestra correctamente en la experiencia de usuario y la última columna tiene un carácter de nueva línea en el nombre.
    - **Causa**: En el flujo de datos de asignación, los archivos de origen CSV de varias líneas no funcionan actualmente cuando se usa \r\n como delimitador de filas. A veces, las líneas adicionales de los retornos de carro pueden producir errores. 
    - **Recomendación** Genere el archivo en el origen usando el carácter \n como delimitador de filas, en lugar de \r\n. O bien, use la actividad de copia para convertir el archivo CSV para que use \n como delimitador de filas.

## <a name="general-troubleshooting-guidance"></a>Guía de solución de problemas generales
1. Compruebe el estado de las conexiones del conjunto de datos. En cada transformación de origen y de receptor, vaya al servicio vinculado para cada conjunto de datos que use y pruebe las conexiones.
2. Compruebe el estado de las conexiones de archivos y tablas en el diseñador de flujo de datos. En el modo de depuración, seleccione **Vista previa de datos** en las transformaciones de origen para asegurarse de que puede tener acceso a los datos.
3. Si todo parece correcto en la vista previa de los datos, vaya al diseñador de canalizaciones y coloque el flujo de datos en una actividad de canalización. Depure la canalización para realizar una prueba de un extremo a otro.

### <a name="improvement-on-csvcdm-format-in-data-flow"></a>Mejora en el formato CSV/CDM en Data Flow 

Si usa el **formato de texto delimitado o CDM para asignar el flujo de datos en Azure Data Factory V2**, puede que se enfrente a cambios de comportamiento en las canalizaciones existentes debido a la mejora del texto delimitado o CDM en el flujo de datos a partir del **1 de mayo de 2021**. 

Es posible que encuentre los siguientes problemas antes de realizar la mejora, pero después de este proceso se solucionarán los problemas. Lea el siguiente contenido para determinar si esta mejora le afecta. 

#### <a name="scenario-1-encounter-the-unexpected-row-delimiter-issue"></a>Escenario 1: encontrar el problema de delimitador de filas inesperado.

 Se verá afectado si se encuentra en las siguientes condiciones:
 - Uso del texto delimitado con la configuración de varias líneas establecida en "true" o CDM como origen.
 - La primera fila tiene más de 128 caracteres. 
 - El delimitador de fila de los archivos de datos no es `\n`.

 Antes de realizar la mejora, el delimitador de filas predeterminado `\n` se puede usar inesperadamente para analizar archivos de texto delimitados, ya que cuando la configuración de varias líneas está establecida en "true", se invalida la configuración del delimitador de filas y este se detecta automáticamente en función de los primeros 128 caracteres. Si no detecta el delimitador de filas real, se revertiría a `\n`.  

 Después de realizar la mejora, debe trabajar con cualquiera de estos tres delimitadores de fila: `\r`, `\n`, `\r\n`.
 
 En el ejemplo siguiente se muestra un cambio de comportamiento en la canalización después de realizar la mejora:

 **Ejemplo**:<br/>
   En la siguiente columna:<br/>
    `C1, C2, {long first row}, C128\r\n `<br/>
    `V1, V2, {values………………….}, V128\r\n `<br/>
 
   Antes de realizar la mejora, `\r` se mantiene en el valor de la columna. El resultado de la columna analizada es:<br/>
   `C1 C2 {long first row} C128`**`\r`**<br/>
   `V1 V2 {values………………….} V128`**`\r`**<br/> 

   Después de realizar la mejora, el resultado de la columna analizada debe ser:<br/>
   `C1 C2 {long first row} C128`<br/>
   `V1 V2 {values………………….} V128`<br/>
  
#### <a name="scenario-2-encounter-an-issue-of-incorrectly-reading-column-values-containing-rn"></a>Escenario 2: detecte un problema de lectura incorrecta de valores de columna que contengan "\r\n".

 Se verá afectado si se encuentra en las siguientes condiciones:
 - Uso del texto delimitado con la configuración de varias líneas establecida en "true" o CDM como origen. 
 - El delimitador de filas es `\r\n`.

 Antes de la mejora, al leer el valor de la columna, puede que se reemplace `\r\n` incorrectamente por `\n`. 

 Después de realizar la mejora, `\r\n` en el valor de la columna no se reemplazará por `\n`.

 En el ejemplo siguiente se muestra un cambio de comportamiento en la canalización después de realizar la mejora:
 
 **Ejemplo**:<br/>
  
 En la siguiente columna:<br/>
  **`"A\r\n"`**`, B, C\r\n`<br/>

 Antes de realizar la mejora, el resultado de la columna analizada es:<br/>
  **`A\n`**` B C`<br/>

 Después de realizar la mejora, el resultado de la columna analizada debe ser:<br/>
  **`A\r\n`**` B C`<br/>  

#### <a name="scenario-3-encounter-an-issue-of-incorrectly-writing-column-values-containing-n"></a>Escenario 3: detecte un problema de escritura incorrecta de valores de columna que contengan "\n".

 Se verá afectado si se encuentra en las siguientes condiciones:
 - Uso del texto delimitado como receptor.
 - El valor de la columna contiene `\n`.
 - El delimitador de filas está establecido en `\r\n`.
 
 Antes de realizar la mejora, al escribir el valor de la columna, puede que se reemplace `\n` incorrectamente por `\r\n`. 

 Después de realizar la mejora, `\n` en el valor de la columna no se reemplazará por `\r\n`.
 
 En el ejemplo siguiente se muestra un cambio de comportamiento en la canalización después de realizar la mejora:

 **Ejemplo**:<br/>

 En la siguiente columna:<br/>
 **`A\n`**` B C`<br/>

 Antes de realizar la mejora, el receptor de CSV es:<br/>
  **`"A\r\n"`**`, B, C\r\n` <br/>

 Después de realizar la mejora, el receptor de CSV debe ser:<br/>
  **`"A\n"`**`, B, C\r\n`<br/>

#### <a name="scenario-4-encounter-an-issue-of-incorrectly-reading-empty-string-as-null"></a>Escenario 4: se detectó un problema al leer incorrectamente una cadena vacía como NULL.
 
 Se verá afectado si se encuentra en las siguientes condiciones:
 - Uso del texto delimitado como origen. 
 - El valor NULL se establece en un valor que no está vacío. 
 - El valor de la columna es una cadena vacía y no tiene comillas. 
 
 Antes de realizar la mejora, el valor de columna de cadena vacía sin comillas se lee como NULL. 

 Después de realizar la mejora, la cadena vacía no se analizará como un valor NULL. 
 
 En el ejemplo siguiente se muestra un cambio de comportamiento en la canalización después de realizar la mejora:

 **Ejemplo**:<br/>

 En la siguiente columna:<br/>
  `A, ,B, `<br/>

 Antes de realizar la mejora, el resultado de la columna analizada es:<br/>
  `A null B null`<br/>

 Después de realizar la mejora, el resultado de la columna analizada debe ser:<br/>
  `A "" (empty string) B "" (empty string)`<br/>


## <a name="next-steps"></a>Pasos siguientes

Para obtener más ayuda para solucionar problemas, consulte estos recursos:

*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
