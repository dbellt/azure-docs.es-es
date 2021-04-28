---
title: Solución de problemas de conectores en Azure Data Factory
description: Obtenga información acerca de la solución de problemas relacionados con conectores en Azure Data Factory.
author: linda33wj
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 04/13/2021
ms.author: jingwang
ms.custom: has-adal-ref
ms.openlocfilehash: 21b5522f07519e9a0c3353cb2463e0ec49063f34
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107713433"
---
# <a name="troubleshoot-azure-data-factory-connectors"></a>Solución de problemas de conectores en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran las formas más comunes de solucionar problemas con los conectores de Azure Data Factory.

## <a name="azure-blob-storage"></a>Azure Blob Storage

### <a name="error-code-azurebloboperationfailed"></a>Código de error: AzureBlobOperationFailed

- **Mensaje**: "Error en la operación de blob. ContainerName: %containerName;, ruta de acceso: %path;."

- **Causa**: Problema con la operación de Blob Storage.

- **Recomendación:**  Para consultar los detalles del error, vaya a [Códigos de error de Blob Storage](/rest/api/storageservices/blob-service-error-codes). Si necesita más ayuda, póngase en contacto con el equipo de Blob Storage.


### <a name="invalid-property-during-copy-activity"></a>Propiedad no válida durante la actividad de copia

- **Mensaje**: `Copy activity \<Activity Name> has an invalid "source" property. The source type is not compatible with the dataset \<Dataset Name> and its linked service \<Linked Service Name>. Please verify your input against.`

- **Causa**: El tipo definido en el conjunto de datos no es coherente con el tipo de origen o de receptor definido en la actividad de copia.

- **Solución:** Edite la definición JSON del conjunto de datos o la canalización para que los tipos sean coherentes y vuelva a ejecutar la implementación.


## <a name="azure-cosmos-db"></a>Azure Cosmos DB

### <a name="error-message-request-size-is-too-large"></a>Mensaje de error: Request size is too large (El tamaño de la solicitud es demasiado grande)

- **Síntomas**: Cuando se copian datos en Azure Cosmos DB con un tamaño de lote de escritura predeterminado, se recibe el siguiente error: `Request size is too large.`

- **Causa**: Azure Cosmos DB limita el tamaño de una única solicitud a 2 MB. La fórmula es *tamaño de la solicitud = tamaño de documento único \* tamaño de lote de escritura*. Si el tamaño del documento es grande, el comportamiento predeterminado generará un tamaño de solicitud demasiado grande. Puede ajustar el tamaño del lote de escritura.

- **Solución:** En el receptor de la actividad de copia, reduzca el valor del *tamaño de lote de escritura* (el valor predeterminado es 10000).

### <a name="error-message-unique-index-constraint-violation"></a>Mensaje de error: Unique index constraint violation (Infracción de restricción de índice único)

- **Síntomas**: Al copiar datos en Azure Cosmos DB, recibe el siguiente error:

    `Message=Partition range id 0 | Failed to import mini-batch. 
    Exception was Message: {"Errors":["Encountered exception while executing function. Exception = Error: {\"Errors\":[\"Unique index constraint violation.\"]}...`

- **Causa**: Hay dos causas posibles:

    - Causa 1: Si utiliza **Insert** como comportamiento de escritura, este error indica que los datos de origen tienen filas u objetos con el mismo identificador.
    - Causa 2: Si usa **Upsert** como comportamiento de escritura y establece otra clave única en el contenedor, este error significa que los datos de origen tienen filas u objetos con identificadores diferentes, pero con el mismo valor para la clave única definida.

- **Solución:** 

    - Para la primera causa, establezca **Upsert** como comportamiento de escritura.
    - Para la segunda causa, asegúrese de que cada documento tenga un valor diferente para la clave única definida.

### <a name="error-message-request-rate-is-large"></a>Mensaje de error: La tasa de solicitudes es grande

- **Síntomas**: Al copiar datos en Azure Cosmos DB, recibe el siguiente error:

    `Type=Microsoft.Azure.Documents.DocumentClientException,
    Message=Message: {"Errors":["Request rate is large"]}`

- **Causa**: El número de unidades de solicitud (RU) usadas es mayor que el valor de RU disponible configurado en Azure Cosmos DB. Para saber cómo Azure Cosmos DB calcula las RU, consulte [Unidades de solicitud en Azure Cosmos DB](../cosmos-db/request-units.md#request-unit-considerations).

- **Solución:** Pruebe una de las dos soluciones siguientes:

    - Aumente el número de *RU de contenedor* a un valor mayor en Azure Cosmos DB. Esta solución mejorará el rendimiento de la actividad de copia, pero supondrá más costo en Azure Cosmos DB. 
    - Disminuya *writeBatchSize* a un valor inferior, como 1000, y *parallelCopies* a un valor inferior, como 1. Esta solución reducirá el rendimiento de la ejecución de copias, pero no supondrá un mayor costo en Azure Cosmos DB.

### <a name="columns-missing-in-column-mapping"></a>Falta una columna en la asignación de columnas

- **Síntomas**: Cuando se importa un esquema en Azure Cosmos DB para la asignación de columnas, faltan algunas columnas. 

- **Causa**: Data Factory infiere el esquema de los 10 primeros documentos de Azure Cosmos DB. Si algunas columnas o propiedades del documento no contienen valores, Data Factory no detecta el esquema y, por tanto, no se muestra.

- **Solución:** Puede optimizar la consulta como se muestra en el código siguiente para forzar que los valores de columna se muestren en el conjunto de resultados con valores vacíos. Supongamos que falta la columna *impossible* en los 10 primeros documentos. Como alternativa, puede agregar manualmente la columna para la asignación.

    ```sql
    select c.company, c.category, c.comments, (c.impossible??'') as impossible from c
    ```

### <a name="error-message-the-guidrepresentation-for-the-reader-is-csharplegacy"></a>Mensaje de error: The GuidRepresentation for the reader is CSharpLegacy (El valor de GuidRepresentation del lector es CSharpLegacy)

- **Síntomas**: Al copiar datos desde MongoAPI o MongoDB de Azure Cosmos DB con el campo de identificador único universal (UUID), recibe el siguiente error:

    `Failed to read data via MongoDB client., 
    Source=Microsoft.DataTransfer.Runtime.MongoDbV2Connector,Type=System.FormatException, 
    Message=The GuidRepresentation for the reader is CSharpLegacy which requires the binary sub type to be UuidLegacy not UuidStandard.,Source=MongoDB.Bson,’“,`

- **Causa**: Hay dos maneras de representar el UUID en JSON binario (BSON): UuidStardard y UuidLegacy. De forma predeterminada, UuidLegacy se usa para leer datos. Si los datos de UUID en MongoDB son UuidStandard, recibirá un error.

- **Solución:** En la cadena de conexión de MongoDB, agregue la opción *uuidRepresentation=standard*. Para más información, consulte [Cadena de conexión de MongoDB](connector-mongodb.md#linked-service-properties).
            
## <a name="azure-cosmos-db-sql-api"></a>Azure Cosmos DB (API de SQL)

### <a name="error-code-cosmosdbsqlapioperationfailed"></a>Código de error: CosmosDbSqlApiOperationFailed

- **Mensaje**: `CosmosDbSqlApi operation Failed. ErrorMessage: %msg;.`

- **Causa**: Problema con la operación CosmosDbSqlApi.

- **Recomendación:**  Para consultar los detalles del error, vea el [documento de ayuda de Azure Cosmos DB](../cosmos-db/troubleshoot-dot-net-sdk.md). Para recibir ayuda adicional, póngase en contacto con el equipo de Azure Cosmos DB.

## <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

### <a name="error-message-the-underlying-connection-was-closed-could-not-establish-trust-relationship-for-the-ssltls-secure-channel"></a>Mensaje de error: Se ha cerrado la conexión subyacente: No se ha podido establecer la relación de confianza para el canal seguro SSL/TLS.

- **Síntomas**: Se produce el siguiente error en la actividad de copia: 

    `Message: ErrorCode = UserErrorFailedFileOperation, Error Message = The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.`

- **Causa**: Error de validación de certificado durante el protocolo de enlace TLS.

- **Solución:** Como solución alternativa, use la copia almacenada provisionalmente para omitir la validación de seguridad de la capa de transporte (TLS) en Azure Data Lake Storage Gen1. Debe reproducir este problema y recopilar el seguimiento de supervisión de la red (netmon) y, luego, que el equipo de red participe en la comprobación de la configuración de la red local.

    ![Diagrama de conexiones de Azure Data Lake Storage Gen1 para solucionar problemas.](./media/connector-troubleshoot-guide/adls-troubleshoot.png)


### <a name="error-message-the-remote-server-returned-an-error-403-forbidden"></a>Mensaje de error: Error en el servidor remoto: (403) Prohibido

- **Síntomas**: Se produce el siguiente error en la actividad de copia: 

   `Message: The remote server returned an error: (403) Forbidden.   
    Response details: {"RemoteException":{"exception":"AccessControlException""message":"CREATE failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.)....`

- **Causa**: Una posible causa es que la entidad de servicio o la identidad administrada que usa no tengan permiso para acceder a determinadas carpetas o archivos.

- **Solución:** Conceda los permisos correspondientes a todas las carpetas y subcarpetas que necesite copiar. Para más información, consulte [Copia de datos con Azure Data Lake Storage Gen1 como origen o destino mediante Azure Data Factory](connector-azure-data-lake-store.md#linked-service-properties).

### <a name="error-message-failed-to-get-access-token-by-using-service-principal-adal-error-service_unavailable"></a>Mensaje de error: No se pudo obtener el token de acceso mediante la entidad de servicio. Error de ADAL: service_unavailable

- **Síntomas**: Se produce el siguiente error en la actividad de copia:

    `Failed to get access token by using service principal.  
    ADAL Error: service_unavailable, The remote server returned an error: (503) Server Unavailable.`

- **Causa**: Cuando el servicio de token de seguridad (STS) propiedad de Azure Active Directory no está disponible, significa que está demasiado ocupado para administrar las solicitudes y devuelve un error HTTP 503. 

- **Solución:** Vuelva a ejecutar la actividad de copia después de unos minutos.


## <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

### <a name="error-code-adlsgen2operationfailed"></a>Código de error: ADLSGen2OperationFailed

- **Mensaje**: `ADLS Gen2 operation failed for: %adlsGen2Message;.%exceptionData;.`

- **Causas y recomendaciones**: diversas causas pueden provocar este error. Busque en la lista siguiente el análisis de las posibles causas y la recomendación relacionada.

  | Análisis de las causas                                               | Recomendación                                               |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | Si Azure Data Lake Storage Gen2 genera un error que indica que alguna operación no funcionó.| Compruebe el mensaje de error detallado que ha generado Azure Data Lake Storage Gen2. Si el error es transitorio, vuelva a intentar la operación. Si necesita más ayuda, póngase en contacto con el soporte técnico de Azure Storage y proporcione el identificador de la solicitud del mensaje de error. |
  | Si el mensaje de error contiene la cadena "Prohibido", es posible que la entidad de servicio o la identidad administrada que está usando no tengan permisos suficientes para acceder a Azure Data Lake Storage Gen2. | Para solucionar este error, consulte [Copia y transformación de los datos de Azure Data Lake Storage Gen2 mediante Azure Data Factory](./connector-azure-data-lake-storage.md#service-principal-authentication). |
  | Si el mensaje de error contiene la cadena "InternalServerError", Azure Data Lake Storage Gen2 devuelve el error. | El error podría deberse a un problema temporal. Si es así, vuelva a intentar la operación. Si el problema persiste, póngase en contacto con el soporte técnico de Azure Storage y proporcione el identificador de solicitud del mensaje de error. |

### <a name="request-to-azure-data-lake-storage-gen2-account-caused-a-timeout-error"></a>Al realizar una solicitud a la cuenta de Azure Data Lake Storage Gen2, se produce un error de tiempo de espera

- **Message**: 
  * Código de error = `UserErrorFailedBlobFSOperation`
  * Mensaje de error = `BlobFS operation failed for: A task was canceled.`

- **Causa**: El problema se debe al error de tiempo de espera del receptor de Azure Data Lake Storage Gen2, que suele producirse en la máquina del entorno de ejecución de integración autohospedado.

- **Recomendación:** 

    - Si es posible, coloque la máquina del entorno de ejecución de integración autohospedado y la cuenta de Azure Data Lake Storage Gen2 de destino en la misma región. Esta solución puede ayudar a evitar errores de tiempo de espera aleatorios y produce un mejor rendimiento.

    - Compruebe si hay una configuración de red especial, como ExpressRoute, y asegúrese de que la red tenga suficiente ancho de banda. Cuando el ancho de banda total sea bajo, se recomienda reducir la configuración de trabajos simultáneos del entorno de ejecución de integración autohospedado. Esto puede ayudar a evitar la competencia por los recursos de red en varios trabajos simultáneos.

    - Si el tamaño de archivo es moderado o pequeño, use un tamaño de bloque menor para la copia no binaria a fin de mitigar este error de tiempo de espera. Para más información, consulte [Operación Put Block de Blob Storage](/rest/api/storageservices/put-block).

       Para especificar el tamaño de bloque personalizado, edite la propiedad en el editor de archivos JSON como se muestra aquí:
    
        ```
        "sink": {
            "type": "DelimitedTextSink",
            "storeSettings": {
                "type": "AzureBlobFSWriteSettings",
                "blockSizeInMB": 8
            }
        }
        ```

                  
## <a name="azure-files-storage"></a>Almacenamiento de Azure Files

### <a name="error-code-azurefileoperationfailed"></a>Código de error: AzureFileOperationFailed

- **Mensaje**: `Azure File operation Failed. Path: %path;. ErrorMessage: %msg;.`

- **Causa**: Problema con la operación de almacenamiento de Azure Files.

- **Recomendación:**  Para consultar los detalles del error, vea [Ayuda de Azure Files](/rest/api/storageservices/file-service-error-codes). Para recibir ayuda adicional, póngase en contacto con el equipo de Azure Files.


## <a name="azure-synapse-analytics-azure-sql-database-and-sql-server"></a>Azure Synapse Analytics, Azure SQL Database, y SQL Server

### <a name="error-code-sqlfailedtoconnect"></a>Código de error: SqlFailedToConnect

- **Mensaje**: `Cannot connect to SQL Database: '%server;', Database: '%database;', User: '%user;'. Check the linked service configuration is correct, and make sure the SQL Database firewall allows the integration runtime to access.`
- **Causas y recomendaciones**: diversas causas pueden provocar este error. Busque en la lista siguiente el análisis de las posibles causas y la recomendación relacionada.

    | Análisis de las causas                                               | Recomendación                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | En Azure SQL, si el mensaje de error contiene la cadena "SqlErrorNumber=47073", significa que se deniega el acceso a la red pública en la configuración de conectividad. | En el firewall de Azure SQL, establezca la opción **Deny public network access** (Denegar acceso a red pública) en *No*. Para más información, consulte [Configuración de conectividad de Azure SQL](../azure-sql/database/connectivity-settings.md#deny-public-network-access). |
    | En Azure SQL, si el mensaje de error contiene un código de error SQL como "SqlErrorNumber=[errorcode]", consulte la guía de solución de problemas de Azure SQL. | Puede encontrar algunas recomendaciones en [Solución de problemas de conectividad y otros errores con Azure SQL Database y Azure SQL Managed Instance](../azure-sql/database/troubleshoot-common-errors-issues.md). |
    | Compruebe si el puerto 1433 está en la lista de permitidos del firewall. | Para más información, consulte [Puertos que usa SQL Server](/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access#ports-used-by-). |
    | Si el mensaje de error contiene la cadena "SqlException", SQL Database genera el error que indica que se produjo un problema en una operación específica. | Para más información, busque por el código de error de SQL en [Errores del motor de base de datos](/sql/relational-databases/errors-events/database-engine-events-and-errors). Si necesita más ayuda, póngase en contacto con el soporte técnico de Azure SQL. |
    | Si se trata de un problema transitorio (por ejemplo, una conexión de red inestable), agregue reintentos en la directiva de actividad para mitigarlo. | Para más información, consulte [Canalizaciones y actividades en Azure Data Factory](./concepts-pipelines-activities.md#activity-policy). |
    | Si el mensaje de error contiene la cadena "El cliente con la dirección IP "..." no tiene permitido acceder" y está intentando conectarse a Azure SQL Database, normalmente se debe a un problema con el firewall de Azure SQL Database. | En la configuración del firewall de Azure SQL Server, habilite la opción **Permitir que los servicios y recursos de Azure accedan a este servidor**. Para más información, consulte [Reglas de firewall de IP de Azure SQL Database y Azure Synapse](../azure-sql/database/firewall-configure.md). |
    
### <a name="error-code-sqloperationfailed"></a>Código de error: SqlOperationFailed

- **Mensaje**: `A database operation failed. Please search error to get more details.`

- **Causas y recomendaciones**: diversas causas pueden provocar este error. Busque en la lista siguiente el análisis de las posibles causas y la recomendación relacionada.

    | Análisis de las causas                                               | Recomendación                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | Si el mensaje de error contiene la cadena "SqlException", SQL Database genera un error que indica que se produjo un problema en una operación concreta. | Si el error de SQL no está claro, intente modificar la base de datos al nivel de compatibilidad más reciente de "150". Esta acción puede producir los errores de la versión de SQL más reciente. Para más información, consulte la [documentación](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level#backwardCompat). <br/> Para más información sobre la solución de problemas de SQL, busque por el código de error de SQL en [Errores del motor de base de datos](/sql/relational-databases/errors-events/database-engine-events-and-errors). Si necesita más ayuda, póngase en contacto con el soporte técnico de Azure SQL. |
    | Si el mensaje de error contiene la cadena "PdwManagedToNativeInteropException", normalmente se debe a una falta de coincidencia entre los tamaños de columna de origen y receptor. | Compruebe el tamaño de las columnas de origen y receptor. Si necesita más ayuda, póngase en contacto con el soporte técnico de Azure SQL. |
    | Si el mensaje de error contiene la cadena "InvalidOperationException", normalmente se debe a que los datos de entrada no son válidos. | Para saber en qué fila se encuentra el problema, habilite la característica de tolerancia a errores en la actividad de copia, que puede redirigir las filas problemáticas al almacenamiento para investigarlas más a fondo. Para más información, consulte [Tolerancia a errores de la actividad de copia en Azure Data Factory](./copy-activity-fault-tolerance.md). |


### <a name="error-code-sqlunauthorizedaccess"></a>Código de error: SqlUnauthorizedAccess

- **Mensaje**: `Cannot connect to '%connectorName;'. Detail Message: '%message;'`

- **Causa**: Las credenciales son incorrectas o la cuenta de inicio de sesión no puede acceder a SQL Database.

- **Recomendación:**  Compruebe que la cuenta de inicio de sesión tenga permisos suficientes para acceder a SQL Database.


### <a name="error-code-sqlopenconnectiontimeout"></a>Código de error: SqlOpenConnectionTimeout

- **Mensaje**: `Open connection to database timeout after '%timeoutValue;' seconds.`

- **Causa**: El problema podría ser un error transitorio de SQL Database.

- **Recomendación:**  Vuelva a intentar la operación para actualizar la cadena de conexión del servicio vinculado con un valor de tiempo de espera de conexión mayor.


### <a name="error-code-sqlautocreatetabletypemapfailed"></a>Código de error: SqlAutoCreateTableTypeMapFailed

- **Mensaje**: `Type '%dataType;' in source side cannot be mapped to a type that supported by sink side(column name:'%columnName;') in autocreate table.`

- **Causa**: La tabla de creación automática no puede cumplir el requisito de origen.

- **Recomendación:**  Actualice el tipo de columna en *mappings* o cree manualmente la tabla del receptor en el servidor de destino.


### <a name="error-code-sqldatatypenotsupported"></a>Código de error: SqlDataTypeNotSupported

- **Mensaje**: `A database operation failed. Check the SQL errors.`

- **Causa**: Si el problema se produce en el origen de SQL y el error está relacionado con el desbordamiento de SqlDateTime, el valor de datos está por encima del intervalo de tipos de lógica (1/1/1753 12:00:00 AM - 12/31/9999 11:59:59 PM).

- **Recomendación:**  Convierta el tipo en una cadena en la consulta SQL de origen o, en la asignación de columnas de la actividad de copia, cambie el tipo de columna a *String*.

- **Causa**: Si el problema se produce en el receptor de SQL y el error está relacionado con el desbordamiento de SqlDateTime, el valor de los datos estará por encima del intervalo permitido en la tabla del receptor.

- **Recomendación:**  Actualice el tipo de columna correspondiente al tipo *datetime2* en la tabla del receptor.


### <a name="error-code-sqlinvaliddbstoredprocedure"></a>Código de error: SqlInvalidDbStoredProcedure

- **Mensaje**: `The specified Stored Procedure is not valid. It could be caused by that the stored procedure doesn't return any data. Invalid Stored Procedure script: '%scriptName;'.`

- **Causa**: El procedimiento almacenado especificado no es válido. La causa podría ser que el procedimiento almacenado no devuelva ningún dato.

- **Recomendación:**  Valide el procedimiento almacenado con las herramientas de SQL. Asegúrese de que el procedimiento almacenado pueda devolver datos.


### <a name="error-code-sqlinvaliddbquerystring"></a>Código de error: SqlInvalidDbQueryString

- **Mensaje**: `The specified SQL Query is not valid. It could be caused by that the query doesn't return any data. Invalid query: '%query;'`

- **Causa**: La consulta SQL especificada no es válida. La causa podría ser que la consulta no devuelva ningún dato.

- **Recomendación:**  Valide la consulta SQL con las herramientas de SQL. Asegúrese de que la consulta pueda devolver datos.


### <a name="error-code-sqlinvalidcolumnname"></a>Código de error: SqlInvalidColumnName

- **Mensaje**: `Column '%column;' does not exist in the table '%tableName;', ServerName: '%serverName;', DatabaseName: '%dbName;'.`

- **Causa**: No se puede encontrar la columna porque es posible que la configuración no sea correcta.

- **Recomendación:**  Compruebe la columna de la consulta, el valor *structure* en el conjunto de datos y el valor *mappings* en la actividad.


### <a name="error-code-sqlbatchwritetimeout"></a>Código de error: SqlBatchWriteTimeout

- **Mensaje**: `Timeouts in SQL write operation.`

- **Causa**: El problema podría ser un error transitorio de SQL Database.

- **Recomendación:**  Vuelva a intentar la operación. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Azure SQL.


### <a name="error-code-sqlbatchwritetransactionfailed"></a>Código de error: SqlBatchWriteTransactionFailed

- **Mensaje**: `SQL transaction commits failed.`

- **Causa**: Si los detalles de la excepción indican constantemente el agotamiento del tiempo de espera de la transacción, significa que la latencia de red entre el entorno de ejecución de integración y la base de datos supera el umbral predeterminado de 30 segundos.

- **Recomendación:**  Actualice la cadena de conexión del servicio vinculado de SQL con un valor de *tiempo de espera de la conexión* igual a 120 o superior y vuelva a ejecutar la actividad.

- **Causa**: Si los detalles de la excepción indican de forma intermitente que la conexión SQL se ha interrumpido, podría tratarse de un error de red transitorio o de un problema de SQL Database.

- **Recomendación:**  Vuelva a intentar la actividad y revise las métricas de SQL Database.


### <a name="error-code-sqlbulkcopyinvalidcolumnlength"></a>Código de error: SqlBulkCopyInvalidColumnLength

- **Mensaje**: `SQL Bulk Copy failed due to receive an invalid column length from the bcp client.`

- **Causa**: Error en la copia masiva de SQL, ya que se ha recibido una longitud de columna no válida del cliente de la utilidad del programa de copia masiva (bcp).

- **Recomendación:**  Para identificar en qué fila se ha encontrado el problema, habilite la característica de tolerancia a errores en la actividad de copia. Esta acción puede redirigir las filas problemáticas al almacenamiento para investigarlas más a fondo. Para más información, consulte [Tolerancia a errores de la actividad de copia en Azure Data Factory](./copy-activity-fault-tolerance.md).


### <a name="error-code-sqlconnectionisclosed"></a>Código de error: SqlConnectionIsClosed

- **Mensaje**: `The connection is closed by SQL Database.`

- **Causa**: SQL Database cierra la conexión SQL cuando la ejecución simultánea excesiva y el servidor cierran la conexión.

- **Recomendación:**  Vuelva a intentar la conexión. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Azure SQL.


### <a name="error-message-conversion-failed-when-converting-from-a-character-string-to-uniqueidentifier"></a>Mensaje de error: Error de conversión al convertir de una cadena de caracteres a uniqueidentifier

- **Síntomas**: Al copiar datos desde un origen de datos tabulares (como SQL Server) en Azure Synapse Analytics mediante copias almacenadas provisionalmente y PolyBase, recibe el siguiente error:

   `ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException, 
    Message=Error happened when loading data into Azure Synapse Analytics., 
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException, 
    Message=Conversion failed when converting from a character string to uniqueidentifier...`

- **Causa**: PolyBase de Azure Synapse Analytics no puede convertir una cadena vacía en un GUID.

- **Solución:** En el receptor de la actividad de copia, en la configuración de PolyBase, establezca la opción **Use Type default** (Usar tipo predeterminado) en *false*.


### <a name="error-message-expected-data-type-decimalxx-offending-value"></a>Mensaje de error: Expected data type: DECIMAL(x,x), Offending value (Tipo de datos esperado: DECIMAL(x,x), valor incorrecto)

- **Síntomas**: Al copiar datos desde un origen de datos tabulares (como SQL Server) en Azure Synapse Analytics mediante copias almacenadas provisionalmente y PolyBase, recibe el siguiente error:

    `ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException, 
    Message=Error happened when loading data into Azure Synapse Analytics., 
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException, 
    Message=Query aborted-- the maximum reject threshold (0 rows) was reached while reading from an external source: 1 rows rejected out of total 415 rows processed. (/file_name.txt)  
    Column ordinal: 18, Expected data type: DECIMAL(x,x), Offending value:..`

- **Causa**: PolyBase de Azure Synapse Analytics no puede insertar una cadena vacía (valor null) en una columna decimal.

- **Solución:** En el receptor de la actividad de copia, en la configuración de PolyBase, establezca la opción **Use Type default** (Usar tipo predeterminado) en false.


### <a name="error-message-java-exception-message-hdfsbridgecreaterecordreader"></a>Mensaje de error: Mensaje de excepción de Java: HdfsBridge::CreateRecordReader

- **Síntomas**: Al copiar datos en Azure Synapse Analytics mediante PolyBase, recibe el siguiente error:

    `Message=110802;An internal DMS error occurred that caused this operation to fail.  
    Details: Exception: Microsoft.SqlServer.DataWarehouse.DataMovement.Common.ExternalAccess.HdfsAccessException, 
    Message: Java exception raised on call to HdfsBridge_CreateRecordReader.  
    Java exception message:HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.: Error [HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.] occurred while accessing external file.....`

- **Causa**: La causa podría ser que el esquema (ancho total de columna) sea demasiado grande (más de 1 MB). Compruebe el esquema de la tabla de Azure Synapse Analytics de destino sumando el tamaño de todas las columnas:

    - Int = 4 bytes
    - Bigint = 8 bytes
    - Varchar(n), char(n), binary(n), varbinary(n) = n bytes
    - Nvarchar(n), nchar(n) = n*2 bytes
    - Date = 6 bytes
    - Datetime/(2), smalldatetime = 16 bytes
    - Datetimeoffset = 20 bytes
    - Decimal = 19 bytes
    - Float = 8 bytes
    - Money = 8 bytes
    - Smallmoney = 4 bytes
    - Real = 4 bytes
    - Smallint = 2 bytes
    - Time = 12 bytes
    - Tinyint = 1 byte

- **Solución:** 
    - Reduzca el ancho de columna a menos de 1 MB.
    - También puede deshabilitar PolyBase para usar un enfoque de inserción masiva.


### <a name="error-message-the-condition-specified-using-http-conditional-headers-is-not-met"></a>Mensaje de error: The condition specified using HTTP conditional header(s) is not met (No se cumple la condición especificada mediante encabezados condicionales HTTP)

- **Síntomas**: Al utilizar una consulta SQL para extraer datos de Azure Synapse Analytics, recibe el siguiente error:

    `...StorageException: The condition specified using HTTP conditional header(s) is not met...`

- **Causa**: Se ha encontrado un problema en Azure Synapse Analytics al consultar la tabla externa en Azure Storage.

- **Solución:** Ejecute la misma consulta en SQL Server Management Studio (SSMS) y compruebe si obtiene el mismo resultado. En caso afirmativo, abra una incidencia de soporte técnico en Azure Synapse Analytics y proporcione el nombre del servidor y de la base de datos de Azure Synapse Analytics.


### <a name="performance-tier-is-low-and-leads-to-copy-failure"></a>El nivel de rendimiento es bajo y genera un error de copia

- **Síntomas**: Al copiar datos en Azure Cosmos DB, recibe el siguiente error: `Database operation failed. Error message from database execution : ExecuteNonQuery requires an open and available Connection. The connection's current state is closed.`.

- **Causa**: Azure SQL Database s1 ha alcanzado los límites de entrada/salida (E/S).

- **Solución:** actualice el nivel de rendimiento de Azure SQL Database para corregir el problema. 


### <a name="sql-table-cant-be-found"></a>No se encuentra la tabla SQL 

- **Síntomas**: Al copiar datos de un entorno híbrido en una tabla de SQL Server local, recibe el siguiente error:`Cannot find the object "dbo.Contoso" because it does not exist or you do not have permissions.`

- **Causa**: La cuenta de SQL actual no tiene permisos suficientes para ejecutar las solicitudes emitidas por SqlBulkCopy.WriteToServer de .NET.

- **Solución:** Cambie a una cuenta de SQL con más privilegios.


### <a name="error-message-string-or-binary-data-is-truncated"></a>Mensaje de error: Los datos binarios o de tipo cadena se truncan

- **Síntomas**: Se produce un error al copiar datos en una tabla de Azure SQL Server local. 

- **Causa**: La definición del esquema de la tabla SQL Cx tiene una o más columnas con menos longitud que la esperada.

- **Solución:** Para resolver la incidencia, intente lo siguiente:

    1. Para detectar las filas que tienen el problema, aplique [tolerancia a errores](./copy-activity-fault-tolerance.md) al receptor SQL, especialmente "redirectIncompatibleRowSettings".

        > [!NOTE]
        > La tolerancia a errores podría requerir tiempo de ejecución adicional, lo que podría conllevar costos mayores.

    2. Vuelva a comprobar los datos redirigidos con la longitud de la columna del esquema de la tabla SQL para ver qué columnas deben actualizarse.

    3. Actualice el esquema de tabla como corresponda.


## <a name="azure-table-storage"></a>Azure Table Storage

### <a name="error-code-azuretableduplicatecolumnsfromsource"></a>Código de error: AzureTableDuplicateColumnsFromSource

- **Mensaje**: `Duplicate columns with same name '%name;' are detected from source. This is NOT supported by Azure Table Storage sink.`

- **Causa**: Se pueden producir columnas de origen duplicadas por uno de los siguientes motivos:
   * Usa la base de datos como una combinación de tablas de origen y tablas aplicadas.
   * Tiene archivos CSV no estructurados con nombres de columna duplicados en la fila de encabezado.

- **Recomendación:**  Vuelva a comprobar y corrija las columnas de origen, según sea necesario.


## <a name="db2"></a>DB2

### <a name="error-code-db2driverrunfailed"></a>Código de error: DB2DriverRunFailed

- **Mensaje**: `Error thrown from driver. Sql code: '%code;'`

- **Causa**: Si el mensaje de error contiene la cadena "SQLSTATE=51002 SQLCODE=-805", siga la sugerencia que se incluye en [Copia de datos de DB2 con Azure Data Factory](./connector-db2.md#linked-service-properties).

- **Recomendación:**  Intente establecer "NULLID" en la propiedad `packageCollection`.


## <a name="delimited-text-format"></a>Formato de texto delimitado

### <a name="error-code-delimitedtextcolumnnamenotallownull"></a>Código de error: DelimitedTextColumnNameNotAllowNull

- **Mensaje**: `The name of column index %index; is empty. Make sure column name is properly specified in the header row.`

- **Causa**: Cuando se establece "firstRowAsHeader" en la actividad, se usa la primera fila como nombre de columna. Este error significa que la primera fila contiene un valor vacío (por ejemplo, "ColumnA, ColumnB").

- **Recomendación:**  Compruebe la primera fila y corrija el valor si está vacío.


### <a name="error-code-delimitedtextmorecolumnsthandefined"></a>Código de error: DelimitedTextMoreColumnsThanDefined

- **Mensaje**: `Error found when processing '%function;' source '%name;' with row number %rowCount;: found more columns than expected column count: %expectedColumnCount;.`

- **Causas y recomendaciones**: diversas causas pueden provocar este error. Busque en la lista siguiente el análisis de las posibles causas y la recomendación relacionada.

  | Análisis de las causas                                               | Recomendación                                               |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | el número de columnas de la fila problemática es mayor que el recuento de columnas de la primera fila. La causa podría ser un problema con los datos o con una configuración incorrecta del delimitador de columna o del carácter de comillas. | Obtenga el número de filas del mensaje de error, compruebe la columna de la fila y corrija los datos. |
  | Si el número de columnas esperado es "1" en el mensaje de error, es posible que haya especificado una configuración errónea de compresión o formato, lo que ha causado que Data Factory analice los archivos de manera incorrecta. | Compruebe la configuración de formato para asegurarse de que coincide con la de los archivos de origen. |
  | Si el origen es una carpeta, es posible que los archivos de la carpeta especificada tengan otro esquema. | Asegúrese de que los archivos de la carpeta especificada tengan un esquema idéntico. |


## <a name="dynamics-365-common-data-service-and-dynamics-crm"></a>Dynamics 365, Common Data Service y Dynamics CRM

### <a name="error-code-dynamicscreateserviceclienterror"></a>Código de error: DynamicsCreateServiceClientError

- **Mensaje**: `This is a transient issue on Dynamics server side. Try to rerun the pipeline.`

- **Causa**: Se trata de un problema transitorio en el servidor de Dynamics.

- **Recomendación:**  vuelva a ejecutar la canalización. Si sigue sin funcionar, intente reducir el paralelismo. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Dynamics.


### <a name="missing-columns-when-you-import-a-schema-or-preview-data"></a>Faltan columnas al importar un esquema u obtener una vista previa de los datos

- **Síntomas**: Faltan algunas columnas al importar un esquema u obtener una vista previa de los datos. Mensaje de error: `The valid structure information (column name and type) are required for Dynamics source.`

- **Causa**: Este problema es así por naturaleza, ya que Data Factory no puede mostrar las columnas que no contienen valores en los 10 primeros registros. Asegúrese de que las columnas que ha agregado tengan el formato correcto. 

- **Recomendación:** Agregue manualmente las columnas en la pestaña Asignación.


### <a name="error-code-dynamicsmissingtargetformultitargetlookupfield"></a>Código de error: DynamicsMissingTargetForMultiTargetLookupField

- **Mensaje**: `Cannot find the target column for multi-target lookup field: '%fieldName;'.`

- **Causa**: La columna de destino no existe en el origen o en la asignación de columnas.

- **Recomendación:**  
  1. Asegúrese de que el origen contenga la columna de destino. 
  2. Agregue la columna de destino en la asignación de columnas. Asegúrese de que la columna de receptor tenga el formato *{fieldName}@EntityReference* .


### <a name="error-code-dynamicsinvalidtargetformultitargetlookupfield"></a>Código de error: DynamicsInvalidTargetForMultiTargetLookupField

- **Mensaje**: `The provided target: '%targetName;' is not a valid target of field: '%fieldName;'. Valid targets are: '%validTargetNames;'`

- **Causa**: Se ha proporcionado un nombre de entidad incorrecto como entidad de destino de un campo de búsqueda de varios destinos.

- **Recomendación:**  Proporcione un nombre de entidad válido para el campo de búsqueda de varios destinos.


### <a name="error-code-dynamicsinvalidtypeformultitargetlookupfield"></a>Código de error: DynamicsInvalidTypeForMultiTargetLookupField

- **Mensaje**: `The provided target type is not a valid string. Field: '%fieldName;'.`

- **Causa**: El valor de la columna de destino no es una cadena.

- **Recomendación:**  Proporcione una cadena válida en la columna de destino de búsqueda de varios destinos.


### <a name="error-code-dynamicsfailedtorequetserver"></a>Código de error: DynamicsFailedToRequetServer

- **Mensaje**: `The Dynamics server or the network is experiencing issues. Check network connectivity or check Dynamics server log for more details.`

- **Causa**: El servidor de Dynamics es inestable o no es accesible, o bien en la red se experimentan problemas.

- **Recomendación:**  Para conocer los detalles, compruebe la conectividad de red o el registro del servidor de Dynamics. Si necesita ayuda adicional, póngase en contacto con el soporte técnico de Dynamics.


### <a name="error-code--dynamicsfailedtoconnect"></a>Código de error: DynamicsFailedToConnect 
 
 - **Mensaje**: `Failed to connect to Dynamics: %message;` 
 

 - **Causa**: Si ve `Office 365 auth with OAuth failed` en el mensaje de error, es posible que el servidor tenga algunas configuraciones no compatibles con OAuth. 
 
 - **Recomendación:** 
    1. Para obtener ayuda, póngase en contacto con el equipo de soporte técnico de Dynamics con el mensaje de error detallado.  
    1. Use la autenticación de la entidad de servicio, puede consultar este artículo: [Ejemplo: Dynamics en línea con la entidad de servicio de Azure AD y la autenticación de certificados](https://docs.microsoft.com/azure/data-factory/connector-dynamics-crm-office-365#example-dynamics-online-using-azure-ad-service-principal-and-certificate-authentication). 
 

 - **Causa**: Si ve `Unable to retrieve authentication parameters from the serviceUri` en el mensaje de error, puede que haya escrito una dirección URL de servicio de Dynamics incorrecta o aplicado un proxy o firewall para interceptar el tráfico. 
 
 - **Recomendación:**
    1. Asegúrese de haber indicado el URI de servicio correcto en el servicio vinculado. 
    1. Si usa el IR autohospedado, asegúrese de que el firewall o proxy no intercepten las solicitudes al servidor de Dynamics. 
   
 
 - **Causa**: Si ve `An unsecured or incorrectly secured fault was received from the other party` en el mensaje de error, se han obtenido respuestas inesperadas del lado servidor. 
 
 - **Recomendación:** 
    1. Asegúrese de que el nombre de usuario y la contraseña sean correctos si usa la autenticación de Office 365. 
    1. Asegúrese de haber escrito el URI de servicio correcto. 
    1. Si usa la dirección URL de CRM regional (la dirección URL tiene un número después de "crm"), asegúrese de usar el identificador regional correcto.
    1. Para obtener ayuda, póngase en contacto con el equipo de soporte técnico de Dynamics. 
 

 - **Causa**: Si ve `No Organizations Found` en el mensaje de error, el nombre de la organización es incorrecto o ha usado un identificador de región de CRM incorrecto en la dirección URL del servicio. 
 
 - **Recomendación:** 
    1. Asegúrese de haber escrito el URI de servicio correcto.
    1. Si usa la dirección URL de CRM regional (la dirección URL tiene un número después de "crm"), asegúrese de usar el identificador regional correcto. 
    1. Para obtener ayuda, póngase en contacto con el equipo de soporte técnico de Dynamics. 

 
 - **Causa**: Si ve `401 Unauthorized` y un mensaje de error relacionado con AAD, hay un problema con la entidad de servicio. 

 - **Recomendación**: Siga la guía del mensaje de error para corregir el problema de la entidad de servicio.  
 
 
 - **Causa**: Para otros errores, el problema suele encontrarse en el lado servidor. 

 - **Recomendación**: Use [XrmToolBox](https://www.xrmtoolbox.com/) para establecer la conexión. Si el error persiste, póngase en contacto con el equipo de soporte técnico de Dynamics para obtener ayuda. 
 
 
### <a name="error-code--dynamicsoperationfailed"></a>Código de error: DynamicsOperationFailed 
 
- **Mensaje**: `Dynamics operation failed with error code: %code;, error message: %message;.` 

- **Causa**: Error en la operación en el lado servidor. 

- **Recomendación**: Extraiga el código de error de la operación de Dynamics que aparece en el mensaje de error: `Dynamics operation failed with error code: {code}`, y consulte el artículo [Códigos de error de servicio web](https://docs.microsoft.com/powerapps/developer/data-platform/org-service/web-service-error-codes) para obtener información más detallada. Si es necesario, puede ponerse en contacto con el equipo de soporte técnico de Dynamics. 
 
 
### <a name="error-code--dynamicsinvalidfetchxml"></a>Código de error: DynamicsInvalidFetchXml 
  
- **Mensaje**: `The Fetch Xml query specified is invalid.` 

- **Causa**: Existe un error en el XML de captura.  

- **Recomendación**: Corrija el error en el XML de captura. 
 
 
### <a name="error-code--dynamicsmissingkeycolumns"></a>Código de error: DynamicsMissingKeyColumns 
 
- **Mensaje**: `Input DataSet must contain keycolumn(s) in Upsert/Update scenario. Missing key column(s): %column;`
 
- **Causa**: Los datos de origen no contienen la columna de clave para la entidad receptora. 

- **Recomendación**: Confirme que las columnas de clave estén en los datos de origen o asigne una columna de origen a la columna de clave de la entidad receptora. 
 
 
### <a name="error-code--dynamicsprimarykeymustbeguid"></a>Código de error: DynamicsPrimaryKeyMustBeGuid 
 
- **Mensaje**: `The primary key attribute '%attribute;' must be of type guid.` 
 
- **Causa**: El tipo de la columna de clave principal no es "Guid". 
 
- **Recomendación**: Asegúrese de que la columna de clave principal de los datos de origen sea de tipo "Guid". 
 

### <a name="error-code--dynamicsalternatekeynotfound"></a>Código de error: DynamicsAlternateKeyNotFound 
 
- **Mensaje**: `Cannot retrieve key information of alternate key '%key;' for entity '%entity;'.` 
 
- **Causa**: La clave alternativa proporcionada no existe, lo que puede deberse a nombres de clave incorrectos o permisos insuficientes. 
 
- **Recomendación:** <br/> 
    1. Corrija los errores tipográficos en el nombre de clave.<br/> 
    1. Asegúrese de tener permisos suficientes en la entidad. 
 
 
### <a name="error-code--dynamicsinvalidschemadefinition"></a>Código de error: DynamicsInvalidSchemaDefinition 
 
- **Mensaje**: `The valid structure information (column name and type) are required for Dynamics source.` 
 
- **Causa**: Falta la propiedad "type" en las columnas receptoras de la asignación de columnas. 
 
- **Recomendación**: Puede agregar la propiedad "type" a esas columnas de la asignación de columnas mediante el editor JSON en el portal. 


## <a name="ftp"></a>FTP

### <a name="error-code-ftpfailedtoconnecttoftpserver"></a>Código de error: FtpFailedToConnectToFtpServer

- **Mensaje**: `Failed to connect to FTP server. Please make sure the provided server information is correct, and try again.`

- **Causa**: Podría haberse usado un tipo de servicio vinculado incorrecto para el servidor FTP, como, por ejemplo, cuando se utiliza el servicio vinculado FTP (SFTP) para conectarse a un servidor FTP.

- **Recomendación:**  Compruebe el puerto del servidor de destino. FTP utiliza el puerto 21.


## <a name="http"></a>HTTP

### <a name="error-code-httpfilefailedtoread"></a>Código de error: HttpFileFailedToRead

- **Mensaje**: `Failed to read data from http server. Check the error from http server：%message;`

- **Causa**: Este error se produce cuando Azure Data Factory se comunica con el servidor HTTP, pero se produce un error en la operación de solicitud HTTP.

- **Recomendación:**  Compruebe el código de estado HTTP del mensaje error y solucione el problema del servidor remoto.


## <a name="oracle"></a>Oracle

### <a name="error-code-argumentoutofrangeexception"></a>Código de error: ArgumentOutOfRangeException

- **Mensaje**: `Hour, Minute, and Second parameters describe an un-representable DateTime.`

- **Causa**: En Data Factory, los valores datetime admitidos se encuentran en el intervalo de 0001-01-01 00:00:00 a 9999-12-31 23:59:59. Sin embargo, Oracle admite un intervalo más amplio de valores datetime (como siglos AC o min/seg > 59), lo que da lugar a un error en Data Factory.

- **Recomendación:** 

    Para ver si el valor de Oracle está en el intervalo de Data Factory, ejecute `select dump(<column name>)`. 

    Para conocer la secuencia de bytes en el resultado, consulte [¿Cómo se almacenan las fechas en Oracle?](https://stackoverflow.com/questions/13568193/how-are-dates-stored-in-oracle)


## <a name="orc-format"></a>Formato ORC

### <a name="error-code-orcjavainvocationexception"></a>Código de error: OrcJavaInvocationException

- **Mensaje**: `An error occurred when invoking Java, message: %javaException;.`
- **Causas y recomendaciones**: diversas causas pueden provocar este error. Busque en la lista siguiente el análisis de las posibles causas y la recomendación relacionada.

    | Análisis de las causas                                               | Recomendación                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | Si el mensaje de error contiene las cadenas "java.lang.OutOfMemory", "Java heap space" y "doubleCapacity", suele tratarse de un problema de administración de memoria en la versión anterior del entorno de ejecución de integración. | Si usa el entorno de ejecución de integración autohospedado, se recomienda actualizar a la versión más reciente. |
    | Si el mensaje de error contiene la cadena "java.lang.OutOfMemory", significa que el entorno de ejecución de integración no tiene suficientes recursos para procesar los archivos. | limite las ejecuciones simultáneas en Integration Runtime. En el caso del entorno de ejecución de integración autohospedado, escale verticalmente a una máquina eficaz con una memoria igual o superior a 8 GB. |
    |Cuando el mensaje de error contiene la cadena "NullPointerReference", la causa podría ser un error transitorio. | Vuelva a intentar la operación. Si el problema persiste, póngase en contacto con el soporte técnico. |
    | Cuando el mensaje de error contiene la cadena "BufferOverflowException", la causa podría ser un error transitorio. | Vuelva a intentar la operación. Si el problema persiste, póngase en contacto con el soporte técnico. |
    | Cuando el mensaje de error contiene la cadena "java.lang.ClassCastException:org.apache.hadoop.hive.serde2.io.HiveCharWritable no se puede transmitir a org.apache.hadoop.io.Text", la causa podría ser un problema de conversión de tipos dentro de Java Runtime. Normalmente, significa que los datos de origen no se pueden controlar correctamente en Java Runtime. | Se trata de un problema de datos. Trate de usar una cadena en lugar de los tipos char o varchar en los datos de formato ORC. |

### <a name="error-code-orcdatetimeexceedlimit"></a>Código de error: OrcDateTimeExceedLimit

- **Mensaje**: `The Ticks value '%ticks;' for the datetime column must be between valid datetime ticks range -621355968000000000 and 2534022144000000000.`

- **Causa**: Si el valor datetime es "0001-01-01 00:00:00", podría deberse a la diferencia entre el [calendario Juliano y el calendario Gregoriano](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar#Difference_between_Julian_and_proleptic_Gregorian_calendar_dates).

- **Recomendación:**  Compruebe el valor de tics y evite usar el valor datetime "0001-01-01 00:00:00".


## <a name="parquet-format"></a>Formato Parquet

### <a name="error-code-parquetjavainvocationexception"></a>Código de error: ParquetJavaInvocationException

- **Mensaje**: `An error occurred when invoking java, message: %javaException;.`

- **Causas y recomendaciones**: diversas causas pueden provocar este error. Busque en la lista siguiente el análisis de las posibles causas y la recomendación relacionada.

    | Análisis de las causas                                               | Recomendación                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | Si el mensaje de error contiene las cadenas "java.lang.OutOfMemory", "Java heap space" y "doubleCapacity", suele tratarse de un problema de administración de memoria en la versión anterior de Integration Runtime. | Si usa el entorno de ejecución de integración autohospedado y la versión es anterior a 3.20.7159.1, se recomienda actualizar a la versión más reciente. |
    | Si el mensaje de error contiene la cadena "java.lang.OutOfMemory", significa que el entorno de ejecución de integración no tiene suficientes recursos para procesar los archivos. | limite las ejecuciones simultáneas en Integration Runtime. En el caso del entorno de ejecución de integración autohospedado, escale verticalmente a una máquina eficaz con una memoria igual o superior a 8 GB. |
    | Cuando el mensaje de error contiene la cadena "NullPointerReference", podría tratarse de un error transitorio. | Vuelva a intentar la operación. Si el problema persiste, póngase en contacto con el soporte técnico. |

### <a name="error-code-parquetinvalidfile"></a>Código de error: ParquetInvalidFile

- **Mensaje**: `File is not a valid Parquet file.`

- **Causa**: Se trata de un problema con el archivo Parquet.

- **Recomendación:**  Compruebe que la entrada sea un archivo Parquet válido.


### <a name="error-code-parquetnotsupportedtype"></a>Código de error: ParquetNotSupportedType

- **Mensaje**: `Unsupported Parquet type. PrimitiveType: %primitiveType; OriginalType: %originalType;.`

- **Causa**: el formato Parquet no se admite en Azure Data Factory.

- **Recomendación:**  Vaya a [Formatos de archivo y códecs de compresión que admite la actividad de copia en Azure Data Factory](./supported-file-formats-and-compression-codecs.md) para volver a comprobar los datos de origen.


### <a name="error-code-parquetmisseddecimalprecisionscale"></a>Código de error: ParquetMissedDecimalPrecisionScale

- **Mensaje**: `Decimal Precision or Scale information is not found in schema for column: %column;.`

- **Causa**: La precisión y la escala del número se analizaron, pero no se proporcionó dicha información.

- **Recomendación:**  El origen no devuelve la información de precisión y escala correcta. Consulte la información en la columna con el problema.


### <a name="error-code-parquetinvaliddecimalprecisionscale"></a>Código de error: ParquetInvalidDecimalPrecisionScale

- **Mensaje**: `Invalid Decimal Precision or Scale. Precision: %precision; Scale: %scale;.`

- **Causa**: el esquema no es válido.

- **Recomendación:**  Compruebe la precisión y la escala de la columna con el problema.


### <a name="error-code-parquetcolumnnotfound"></a>Código de error: ParquetColumnNotFound

- **Mensaje**: `Column %column; does not exist in Parquet file.`

- **Causa**: El esquema de origen no coincide con el esquema de receptor.

- **Recomendación:**  Compruebe las asignaciones de la actividad. Asegúrese de que la columna de origen se pueda asignar a la columna de receptor correcta.


### <a name="error-code-parquetinvaliddataformat"></a>Código de error: ParquetInvalidDataFormat

- **Mensaje**: `Incorrect format of %srcValue; for converting to %dstType;.`

- **Causa**: Los datos no se pueden convertir al tipo especificado en mappings.source.

- **Recomendación:**  Vuelva a comprobar los datos de origen o especifique el tipo de datos correcto para esta columna en la asignación de columnas de la actividad de copia. Para más información, consulte [Formatos de archivo y códecs de compresión que admite la actividad de copia en Azure Data Factory](./supported-file-formats-and-compression-codecs.md).


### <a name="error-code-parquetdatacountnotmatchcolumncount"></a>Código de error: ParquetDataCountNotMatchColumnCount

- **Mensaje**: `The data count in a row '%sourceColumnCount;' does not match the column count '%sinkColumnCount;' in given schema.`

- **Causa**:  Una desigualdad entre el número de columnas de origen y el número de columnas de receptor.

- **Recomendación:**  Vuelva a comprobar que el número de columnas de origen sea igual al número de columnas de receptor en "mapping".


### <a name="error-code-parquetdatatypenotmatchcolumntype"></a>Código de error: ParquetDataTypeNotMatchColumnType

- **Mensaje**: `The data type %srcType; is not match given column type %dstType; at column '%columnIndex;'.`

- **Causa**: Los datos del origen no se pueden convertir al tipo que se define en el receptor.

- **Recomendación:**  especifique un tipo correcto en mapping.sink.


### <a name="error-code-parquetbridgeinvaliddata"></a>Código de error: ParquetBridgeInvalidData

- **Mensaje**: `%message;`

- **Causa**: El valor de los datos ha superado el límite.

- **Recomendación:**  Vuelva a intentar la operación. Si el problema persiste, póngase en contacto con nosotros.


### <a name="error-code-parquetunsupportedinterpretation"></a>Código de error: ParquetUnsupportedInterpretation

- **Mensaje**: `The given interpretation '%interpretation;' of Parquet format is not supported.`

- **Causa**: Este escenario no se admite.

- **Recomendación:**  el valor de "ParquetInterpretFor" no debe ser "sparkSql".


### <a name="error-code-parquetunsupportfilelevelcompressionoption"></a>Código de error: ParquetUnsupportFileLevelCompressionOption

- **Mensaje**: `File level compression is not supported for Parquet.`

- **Causa**: Este escenario no se admite.

- **Recomendación:**  Quite "CompressionType" en la carga.


### <a name="error-code-usererrorjniexception"></a>Código de error: UserErrorJniException

- **Mensaje**: `Cannot create JVM: JNI return code [-6][JNI call failed: Invalid arguments.]`

- **Causa**: No se puede crear una Máquina virtual Java (JVM) porque se han establecido algunos argumentos no válidos (globales).

- **Recomendación:**  Inicie sesión en la máquina que hospeda *cada nodo* del entorno de ejecución de integración autohospedado. Asegúrese de que la variable del sistema esté configurada correctamente, como se indica a continuación: `_JAVA_OPTIONS "-Xms256m -Xmx16g" with memory bigger than 8 G`. Reinicie todos los nodos del entorno de ejecución de integración y vuelva a ejecutar la canalización.


### <a name="arithmetic-overflow"></a>Desbordamiento aritmético

- **Síntomas**: Ha aparecido un mensaje de error al copiar los archivos de Parquet: `Message = Arithmetic Overflow., Source = Microsoft.DataTransfer.Common`

- **Causa**: Actualmente solo se admite el decimal de precisión <= 38 y una longitud de la parte entera <= 20 al copiar archivos de Oracle en Parquet. 

- **Solución:** Como solución alternativa, puede convertir en VARCHAR2 cualquier columna que tenga este problema.


### <a name="no-enum-constant"></a>Ninguna constante de enumeración

- **Síntomas**: Ha aparecido un mensaje de error al copiar datos en formato Parquet: `java.lang.IllegalArgumentException:field ended by &apos;;&apos;` o: `java.lang.IllegalArgumentException:No enum constant org.apache.parquet.schema.OriginalType.test`.

- **Causa**: 

    El problema puede deberse a espacios en blanco o caracteres especiales no admitidos (como ,;{}()\n\t=) en el nombre de la columna, ya que Parquet no admite este formato. 

    Por ejemplo, un nombre de columna como *contoso(test)* analizará el tipo entre corchetes a partir del [código](https://github.com/apache/parquet-mr/blob/master/parquet-column/src/main/java/org/apache/parquet/schema/MessageTypeParser.java) `Tokenizer st = new Tokenizer(schemaString, " ;{}()\n\t");`. El error se produce porque no hay ningún tipo "test".

    Para comprobar los tipos admitidos, vaya al [sitio apache/parquet-mr](https://github.com/apache/parquet-mr/blob/master/parquet-column/src/main/java/org/apache/parquet/schema/OriginalType.java) de GitHub.

- **Solución:** 

    Vuelva a comprobar que:
    - Haya espacios en blanco en el nombre de la columna de receptor.
    - La primera fila con espacios en blanco se use como nombre de columna.
    - Se admita el tipo OriginalType. Intente evitar el uso de estos caracteres especiales: `,;{}()\n\t=`. 


## <a name="rest"></a>REST

### <a name="error-code-restsinkcallfailed"></a>Código de error: RestSinkCallFailed

- **Mensaje**: `Rest Endpoint responded with Failure from server. Check the error from server:%message;`

- **Causa**: Este error se produce cuando Azure Data Factory se comunica con el punto de conexión REST sobre el protocolo HTTP, y se produce un error en la operación de solicitud.

- **Recomendación:**  Compruebe el código de estado HTTP o el mensaje de error y solucione el problema del servidor remoto.

### <a name="unexpected-network-response-from-the-rest-connector"></a>Respuesta de red inesperada del conector REST

- **Síntomas**: A veces, el punto de conexión recibe una respuesta inesperada (400/401/403/500) del conector REST.

- **Causa**: El conector de origen REST utiliza la dirección URL y el método HTTP/encabezado/cuerpo del servicio vinculado/conjunto de datos/origen de copia como parámetros al crear una solicitud HTTP. Lo más probable es que el problema se deba a errores en uno o varios parámetros especificados.

- **Solución:** 
    - Utilice "curl" en la ventana del símbolo del sistema para ver si el parámetro es la causa (los encabezados **Accept** y **User-Agent** se deben incluir siempre):
    
      `curl -i -X <HTTP method> -H <HTTP header1> -H <HTTP header2> -H "Accept: application/json" -H "User-Agent: azure-data-factory/2.0" -d '<HTTP body>' <URL>`
      
      Si el comando devuelve la misma respuesta inesperada, corrija los parámetros anteriores con "curl" hasta que devuelva la respuesta esperada. 

      También puede usar "curl--help" para realizar un uso más avanzado del comando.

    - Si solo el conector REST de Data Factory devuelve una respuesta inesperada, póngase en contacto con el servicio de soporte técnico de Microsoft para solucionar el problema.
    
    - Tenga en cuenta que "curl" puede no ser adecuado para reproducir el problema de validación de certificados SSL. En algunos escenarios, el comando "curl" se ha ejecutado correctamente sin generar ningún problema de validación de certificados SSL. Sin embargo, cuando se ejecuta la misma dirección URL en un explorador, en realidad no se devuelve ningún certificado SSL para que el cliente establezca una relación de confianza con el servidor.

      En el caso anterior, se recomiendan herramientas como **Postman** y **Fiddler**.


## <a name="sftp"></a>SFTP

#### <a name="error-code-sftpoperationfail"></a>Código de error: SftpOperationFail

- **Mensaje**: `Failed to '%operation;'. Check detailed error from SFTP.`

- **Causa**: Problema con la operación SFTP.

- **Recomendación:**  Compruebe los detalles del error desde SFTP.


### <a name="error-code-sftprenameoperationfail"></a>Código de error: SftpRenameOperationFail

- **Mensaje**: `Failed to rename the temp file. Your SFTP server doesn't support renaming temp file, set "useTempFileRename" as false in copy sink to disable uploading to temp file.`

- **Causa**: El servidor SFTP no admite el cambio de nombre del archivo temporal.

- **Recomendación:**  Establezca "useTempFileRename" como false en el receptor de copia para deshabilitar la carga en el archivo temporal.


### <a name="error-code-sftpinvalidsftpcredential"></a>Código de error: SftpInvalidSftpCredential

- **Mensaje**: `Invalid SFTP credential provided for '%type;' authentication type.`

- **Causa**: El contenido de la clave privada se ha obtenido de Azure Key Vault o del SDK, pero no se ha codificado correctamente.

- **Recomendación:**  

    Si el contenido de la clave privada procede del almacén de claves, el archivo de clave original puede funcionar si lo carga directamente en el servicio vinculado de SFTP.

    Para más información, consulte [Copia de datos hacia y desde un servidor SFTP mediante Azure Data Factory](./connector-sftp.md#use-ssh-public-key-authentication). El contenido de la clave privada es contenido de clave privada SSH codificado en Base64.

    Codifique *todo* el archivo de clave privada original con codificación Base64 y almacene la cadena codificada en el almacén de claves. El archivo de clave privada original es el que puede funcionar en el servicio vinculado de SFTP si selecciona **Cargar** en el archivo.

    Estos son algunos ejemplos que puede usar para generar la cadena:

    - Mediante código en C#:

        ```
        byte[] keyContentBytes = File.ReadAllBytes(Private Key Path);
        string keyContent = Convert.ToBase64String(keyContentBytes, Base64FormattingOptions.None);
        ```

    - Mediante código de Python:

        ```
        import base64
        rfd = open(r'{Private Key Path}', 'rb')
        keyContent = rfd.read()
        rfd.close()
        print base64.b64encode(Key Content)
        ```

    - Use una herramienta de conversión de Base64 de terceros. Se recomienda la herramienta [Encode to Base64 format](https://www.base64encode.org/).

- **Causa**: Se ha elegido un formato de contenido de clave incorrecto.

- **Recomendación:**  

    La clave privada de SSH con el formato PKCS#8 (que empieza por "-----BEGIN ENCRYPTED PRIVATE KEY-----") no se admite actualmente para acceder al servidor SFTP en Data Factory. 

    Para convertir la clave al formato de clave SSH tradicional, que empieza por "-----BEGIN RSA PRIVATE KEY-----", ejecute los siguientes comandos:

    ```
    openssl pkcs8 -in pkcs8_format_key_file -out traditional_format_key_file
    chmod 600 traditional_format_key_file
    ssh-keygen -f traditional_format_key_file -p
    ```

- **Causa**: Contenido de clave privada o credenciales no válido.

- **Recomendación:**  Para comprobar si el archivo de clave o la contraseña son correctos, use herramientas como WinSCP.

### <a name="sftp-copy-activity-failed"></a>Error en la actividad de copia de SFTP

- **Síntomas**: 
  * Código de error: UserErrorInvalidColumnMappingColumnNotFound 
  * Mensaje de error: `Column 'AccMngr' specified in column mapping cannot be found in source data.`

- **Causa**: El origen no incluye una columna denominada "AccMngr".

- **Solución:** Para determinar si existe la columna "AccMngr", vuelva a comprobar la configuración del conjunto de datos asignando la columna del conjunto de datos de destino.


### <a name="error-code-sftpfailedtoconnecttosftpserver"></a>Código de error: SftpFailedToConnectToSftpServer

- **Mensaje**: `Failed to connect to SFTP server '%server;'.`

- **Causa**: Si el mensaje de error contiene la cadena "La operación de lectura del socket ha agotado el tiempo de espera después de 30 000 milisegundos", una posible causa es que se use un tipo de servicio vinculado incorrecto para el servidor SFTP. Por ejemplo, podría estar usando el servicio vinculado FTP para conectarse al servidor SFTP.

- **Recomendación:**  Compruebe el puerto del servidor de destino. De forma predeterminada, SFTP usa el puerto 22.

- **Causa**: Si el mensaje de error contiene la cadena "La respuesta del servidor no contiene la identificación del protocolo SSH", una posible causa es que el servidor SFTP haya limitado la conexión. Data Factory crea varias conexiones para realizar descargas del servidor SFTP en paralelo y, a veces, se alcanza la limitación del servidor SFTP. Normalmente, los distintos servidores devuelven errores diferentes cuando encuentran limitaciones.

- **Recomendación:**  

    Especifique el número máximo de conexiones simultáneas del conjunto de datos SFTP como 1 y vuelva a ejecutar la actividad de copia. Si la actividad se realiza correctamente, puede estar seguro de que la limitación era la causa.

    Si quiere promocionar el rendimiento bajo, póngase en contacto con el administrador de SFTP para aumentar el límite de conexiones simultáneas, o también puede hacer lo siguiente:

    * Si usa el entorno de ejecución de integración autohospedado, agregue la dirección IP de la máquina de dicho entorno a la lista de permitidos.
    * Si usa Azure IR, agregue [direcciones IP de Azure Integration Runtime](./azure-integration-runtime-ip-addresses.md). Si no quiere agregar un intervalo de direcciones IP a la lista de permitidos del servidor SFTP, use en su lugar el entorno de ejecución de integración autohospedado.

## <a name="sharepoint-online-list"></a>Lista de SharePoint Online

### <a name="error-code-sharepointonlineauthfailed"></a>Código de error: SharePointOnlineAuthFailed

- **Mensaje**: `The access token generated failed, status code: %code;, error message: %message;.`

- **Causa**: Es posible que el identificador y la clave de la entidad de servicio no se hayan establecido correctamente.

- **Recomendación:**  Compruebe que la aplicación registrada (el identificador de la entidad de servicio) y la clave se hayan establecido correctamente.


## <a name="xml-format"></a>Formato XML

### <a name="error-code-xmlsinknotsupported"></a>Código de error: XmlSinkNotSupported

- **Mensaje**: `Write data in XML format is not supported yet, choose a different format!`

- **Causa**: Se ha usado un conjunto de datos XML como conjunto de datos de receptor en la actividad de copia.

- **Recomendación:**  Use un conjunto de datos con un formato diferente al del conjunto de datos de receptor.


### <a name="error-code-xmlattributecolumnnameconflict"></a>Código de error: XmlAttributeColumnNameConflict

- **Mensaje**: `Column names %attrNames;' for attributes of element '%element;' conflict with that for corresponding child elements, and the attribute prefix used is '%prefix;'.`

- **Causa**: Se ha usado un prefijo de atributo, que ha provocado el conflicto.

- **Recomendación:**  Establezca otro valor para la propiedad "attributePrefix".


### <a name="error-code-xmlvaluecolumnnameconflict"></a>Código de error: XmlValueColumnNameConflict

- **Mensaje**: `Column name for the value of element '%element;' is '%columnName;' and it conflicts with the child element having the same name.`

- **Causa**: Se ha usado uno de los nombres de elemento secundario como nombre de columna para el valor del elemento.

- **Recomendación:**  Establezca otro valor para la propiedad "valueColumn".


### <a name="error-code-xmlinvalid"></a>Código de error: XmlInvalid

- **Mensaje**: `Input XML file '%file;' is invalid with parsing error '%error;'.`

- **Causa**: El archivo XML de entrada no tiene el formato correcto.

- **Recomendación:**  Corrija el archivo XML para que tenga el formato correcto.


## <a name="general-copy-activity-error"></a>Error general de la actividad de copia

### <a name="error-code-jrenotfound"></a>Código de error: JreNotFound

- **Mensaje**: `Java Runtime Environment cannot be found on the Self-hosted Integration Runtime machine. It is required for parsing or writing to Parquet/ORC files. Make sure Java Runtime Environment has been installed on the Self-hosted Integration Runtime machine.`

- **Causa**: El entorno de ejecución de integración autohospedado no encuentra Java Runtime, y es necesario para leer orígenes específicos.

- **Recomendación:**  Compruebe el entorno de ejecución de integración en [Uso del entorno de ejecución de integración autohospedado](./format-parquet.md#using-self-hosted-integration-runtime).


### <a name="error-code-wildcardpathsinknotsupported"></a>Código de error: WildcardPathSinkNotSupported

- **Mensaje**: `Wildcard in path is not supported in sink dataset. Fix the path: '%setting;'.`

- **Causa**: El conjunto de datos de receptor no admite caracteres comodín.

- **Recomendación:**  Compruebe el conjunto de datos de receptor y escriba de nuevo la ruta de acceso sin caracteres comodín.


### <a name="fips-issue"></a>Problema de PFIPS

- **Síntomas**: Se produce un error en la actividad de copia en la máquina del entorno de ejecución de integración autohospedado habilitado para FIPS con el mensaje de error siguiente: `This implementation is not part of the Windows Platform FIPS validated cryptographic algorithms.` 

- **Causa**: Este error puede producirse al copiar datos con conectores como Azure Blob, SFTP, etc. El Estándar federal de procesamiento de información (FIPS) define un determinado conjunto de algoritmos criptográficos que se pueden usar. Cuando se habilita el modo FIPS en el equipo, algunas clases criptográficas de las que depende la actividad se bloquean en algunos escenarios.

- **Solución:** Sepa [por qué ya no se recomienda el "modo FIPS"](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/why-we-8217-re-not-recommending-8220-fips-mode-8221-anymore/ba-p/701037), y valore si puede deshabilitar FIPS en la máquina del entorno de ejecución de integración autohospedado.

    De manera alternativa, si solo quiere permitir que Azure Data Factory omita FIPS y que las ejecuciones de actividad se realicen correctamente, haga lo siguiente:

    1. Abra la carpeta donde está instalado el entorno de ejecución de integración autohospedado. La ruta de acceso suele ser *C:\Program Files\Microsoft Integration Runtime \<IR version>\Shared*.

    2. Abra el archivo *diawp.exe.config* y, luego, al final de la sección `<runtime>`, agregue `<enforceFIPSPolicy enabled="false"/>`, como se muestra aquí:

        ![Captura de pantalla de una sección del archivo diawp.exe.config que muestra FIPS deshabilitado.](./media/connector-troubleshoot-guide/disable-fips-policy.png)

    3. Guarde el archivo y, luego, reinicie la máquina del entorno de ejecución de integración autohospedado.


## <a name="next-steps"></a>Pasos siguientes

Para obtener ayuda para solucionar problemas, pruebe estos recursos:

*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-data-factory.html)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)