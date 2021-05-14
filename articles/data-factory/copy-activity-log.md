---
title: Registro de sesión en la actividad de copia
description: Obtenga información sobre cómo habilitar el registro de sesión en la actividad de copia en Azure Data Factory.
author: dearandyxu
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: yexu
ms.openlocfilehash: f979ca04a276f52e5f76a8e4327f9ff8ef390fe6
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108143708"
---
#  <a name="session-log-in-copy-activity"></a>Registro de sesión en la actividad de copia

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Puede registrar los nombres de archivo copiados en la actividad de copia, lo que puede ayudarle a asegurarse de que los datos no se copian correctamente desde el almacén de origen al de destino, sino que también son coherentes entre el almacén de origen y el de destino mediante la revisión de los archivos copiados en los registros de sesión de la actividad de copia.  

Al habilitar la configuración de tolerancia a errores en la actividad de copia para omitir los datos con errores, también se pueden registrar los archivos omitidos y las filas omitidas.  Puede obtener más detalles en [Tolerancia a errores de la actividad de copia en Azure Data Factory](copy-activity-fault-tolerance.md). 

## <a name="configuration"></a>Configuración
En el ejemplo siguiente se proporciona una definición JSON para habilitar el registro de sesión en la actividad de copia: 

```json
"typeProperties": {
    "source": {
        "type": "BinarySource",
        "storeSettings": {
            "type": "AzureDataLakeStoreReadSettings",
            "recursive": true
        },
        "formatSettings": {
            "type": "BinaryReadSettings"
        }
    },
    "sink": {
        "type": "BinarySink",
        "storeSettings": {
            "type": "AzureBlobFSWriteSettings"
        }
    },                    
    "skipErrorFile": {
        "fileForbidden": true,
        "dataInconsistency": true
    },
    "validateDataConsistency": true,
    "logSettings": {
        "enableCopyActivityLog": true,
        "copyActivityLogSettings": {
            "logLevel": "Warning",
            "enableReliableLogging": false
        },
        "logLocationSettings": {
            "linkedServiceName": {
               "referenceName": "ADLSGen2",
               "type": "LinkedServiceReference"
            },
            "path": "sessionlog/"
        }
    }
}
```

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | -------- 
enableCopyActivityLog | Cuando se establece en true, tendrá la oportunidad de registrar los archivos copiados, los archivos omitidos o las filas omitidas.  | True<br/>False (valor predeterminado) | No
logLevel | El nivel "Info" registrará todos los archivos copiados, los archivos omitidos y las filas omitidas. El nivel "Advertencia" solo registrará los archivos omitidos y las filas omitidas.  | Información<br/>Advertencia (valor predeterminado) | No
enableReliableLogging | Cuando sea true, la actividad de copia en modo confiable vaciará los registros inmediatamente una vez que se copie cada archivo en el destino.  Cuando se copian enormes cantidades de archivos con el modo de registro confiable habilitado en la actividad de copia, se debe esperar que se vea afectado el rendimiento de la copia, ya que se requieren operaciones de escritura dobles para cada copia de archivos. Una solicitud es para el almacén de destino y otra solicitud es para el almacén de almacenamiento de registros.  La actividad de copia en el modo de mejor esfuerzo vaciará los registros con un lote de registros dentro de un período de tiempo, de modo que el rendimiento de la copia se verá mucho menos afectado. No se garantiza la integridad y la escala de tiempo del registro en este modo, ya que hay algunas posibilidades de que el último lote de eventos de registro no se haya vaciado en el archivo de registro cuando se produce un error en la actividad de copia. En este momento, verá que algunos archivos que se copian en el destino no se registran.  | True<br/>False (valor predeterminado) | No
logLocationSettings | Un grupo de propiedades que se pueden utilizar para especificar la ubicación donde se almacenan los registros de sesión. | | No
linkedServiceName | Servicio vinculado de [Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) o [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) para almacenar los archivos de registro de sesión. | Nombres de un servicio vinculado a los tipos `AzureBlobStorage` o `AzureBlobFS`, que hace referencia a la instancia que usa para almacenar los archivos de registro. | No
path | Ruta de acceso de los archivos de registro. | Especifique la ruta de acceso donde quiere almacenar los archivos de registro. Si no se proporciona una ruta de acceso, el servicio creará un contenedor para usted. | No


## <a name="monitoring"></a>Supervisión

### <a name="output-from-copy-activity"></a>Salida de la actividad de copia
Cuando la actividad de copia se ha ejecutado completamente, puede ver la ruta de acceso de los archivos de registro desde la salida de cada ejecución de la actividad de copia. Puede encontrar los archivos de registro en la ruta de acceso: `https://[your-blob-account].blob.core.windows.net/[logFilePath]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].txt`.  Los archivos de registro generados tienen la extensión .txt, y sus datos están en formato CSV.

```json
"output": {
            "dataRead": 695,
            "dataWritten": 186,
            "filesRead": 3,  
            "filesWritten": 1, 
            "filesSkipped": 2, 
            "throughput": 297,
            "logFilePath": "myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "dataConsistencyVerification": 
           { 
                "VerificationResult": "Verified", 
                "InconsistentData": "Skipped" 
           } 
        }

```

> [!NOTE]
> Cuando la propiedad `enableCopyActivityLog` está establecida en `Enabled`, los nombres de los archivos de registro los genera el sistema.

### <a name="the-schema-of-the-log-file"></a>Esquema del archivo de registro

El esquema de un archivo de registro es el siguiente.

Columna | Descripción 
-------- | -----------  
Timestamp | Marca de tiempo cuando ADF lee, escribe u omite el objeto.
Nivel | Nivel de registro de este elemento. Puede ser "Warning" o "Info".
OperationName | Comportamiento operativo de la actividad de copia de ADF en cada objeto. Puede ser "FileRead", "FileWrite", "FileSkip" o "TabularRowSkip".
OperationItem | Nombres de archivo o filas omitidas.
Message | Más información que se mostrará si el archivo se ha leído del almacén de origen o se ha escrito en el almacén de destino. También puede ser el motivo por el que se ha omitido el archivo o las filas.

El siguiente es un ejemplo de un archivo de registro. 
```
Timestamp, Level, OperationName, OperationItem, Message
2020-10-19 08:39:13.6688152,Info,FileRead,"sample1.csv","Start to read file: {""Path"":""sample1.csv"",""ItemType"":""File"",""Size"":104857620,""LastModified"":""2020-10-19T08:22:31Z"",""ETag"":""\""0x8D874081F80C01A\"""",""ContentMD5"":""dGKVP8BVIy6AoTtKnt+aYQ=="",""ObjectName"":null}"
2020-10-19 08:39:56.3190846, Warning, FileSkip, "sample1.csv", "File is skipped after read 548000000 bytes: ErrorCode=DataConsistencySourceDataChanged,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Source file 'sample1.csv' is changed by other clients during the copy activity run.,Source=,'." 
2020-10-19 08:40:13.6688152,Info,FileRead,"sample2.csv","Start to read file: {""Path"":""sample2.csv"",""ItemType"":""File"",""Size"":104857620,""LastModified"":""2020-10-19T08:22:31Z"",""ETag"":""\""0x8D874081F80C01A\"""",""ContentMD5"":""dGKVP8BVIy6AoTtKnt+aYQ=="",""ObjectName"":null}"
2020-10-19 08:40:13.9003981,Info,FileWrite,"sample2.csv","Start to write file from source file: sample2.csv."
2020-10-19 08:45:17.6508407,Info,FileRead,"sample2.csv","Complete reading file successfully. "
2020-10-19 08:45:28.7390083,Info,FileWrite,"sample2.csv","Complete writing file from source file: sample2.csv. File is successfully copied."
```
En el archivo de registro anterior, puede ver que sample1.csv se ha omitido porque no se pudo comprobar su coherencia entre el almacén de origen y el de destino. El archivo sample1.csv se vuelve incoherente porque otras aplicaciones lo han modificado mientras la actividad de copia de ADF estaba realizando la copia. También puede ver que sample2.csv se ha copiado correctamente del almacén de origen al de destino.

Puede usar varios motores de análisis para analizar aún más los archivos de registro.  A continuación, se muestran algunos ejemplos de uso de la consulta SQL para analizar el archivo de registro mediante la importación del archivo de registro en formato CSV en SQL Database, donde el nombre de la tabla puede ser SessionLogDemo.  

-   Seleccionar la lista de archivos copiados. 
```sql
select OperationItem from SessionLogDemo where Message like '%File is successfully copied%'
```

-   Seleccionar la lista de archivos copiados en un intervalo de tiempo determinado. 
```sql
select OperationItem from SessionLogDemo where TIMESTAMP >= '<start time>' and TIMESTAMP <= '<end time>' and Message like '%File is successfully copied%'
```

-   Seleccionar un archivo determinado con su hora de copia y sus metadatos. 
```sql
select * from SessionLogDemo where OperationItem='<file name>'
```

-   Seleccionar una lista de archivos con sus metadatos copiados dentro de un intervalo de tiempo. 
```sql
select * from SessionLogDemo where OperationName='FileRead' and Message like 'Start to read%' and OperationItem in (select OperationItem from SessionLogDemo where TIMESTAMP >= '<start time>' and TIMESTAMP <= '<end time>' and Message like '%File is successfully copied%')
```

-   Seleccionar la lista de archivos omitidos. 
```sql
select OperationItem from SessionLogDemo where OperationName='FileSkip'
```

-   Seleccionar el motivo por el que se ha omitido un archivo determinado. 
```sql
select TIMESTAMP, OperationItem, Message from SessionLogDemo where OperationName='FileSkip'
```

-   Seleccionar la lista de archivos omitidos por la misma razón: "el archivo de blob no existe". 
```sql
select TIMESTAMP, OperationItem, Message from SessionLogDemo where OperationName='FileSkip' and Message like '%UserErrorSourceBlobNotExist%'
```

-   Seleccionar el nombre de archivo que requiere más tiempo para la copia.
```sql
select top 1 OperationItem, CopyDuration=DATEDIFF(SECOND, min(TIMESTAMP), max(TIMESTAMP)) from SessionLogDemo group by OperationItem order by CopyDuration desc
```


## <a name="next-steps"></a>Pasos siguientes
Consulte los otros artículos de la actividad de copia:

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Tolerancia a errores de la actividad de copia](copy-activity-fault-tolerance.md)
- [Comprobación de la coherencia de los datos en la actividad de copia (versión preliminar)](copy-activity-data-consistency.md)
