---
title: Copia de datos hacia o desde Azure Blob Storage
description: 'Aprenda a copiar datos de blob en Azure Data Factory. Utilice nuestro ejemplo: Cómo copiar datos entre Azure Blob Storage y Azure SQL Database.'
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/05/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 3d42c7cc6498adad251174db7caea11feec82784
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753574"
---
# <a name="copy-data-to-or-from-azure-blob-storage-using-azure-data-factory"></a>Copia de datos hacia Azure Blob Storage o desde él con Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-azure-blob-connector.md)
> * [Versión 2 (versión actual)](../connector-azure-blob-storage.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Conector de Azure Blob Storage en V2](../connector-azure-blob-storage.md).


En este artículo se explica el uso de la actividad de copia en Azure Data Factory para copiar datos hacia Azure Blob Storage y desde este servicio. Se basa en la información general que ofrece el artículo [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md).

## <a name="overview"></a>Información general
Puede copiar datos de cualquier almacén de datos de origen compatible a Azure Blob Storage o de Azure Blob Storage a cualquier almacén de datos del receptor compatible. En la tabla siguiente se proporciona una lista de almacenes de datos que se admiten como orígenes o receptores de la actividad de copia. Por ejemplo, puede trasladar datos **de** una base de datos de SQL Server o una base de datos de Azure SQL Database **a** un almacenamiento de blobs de Azure. Y, puede copiar datos **desde** Azure Blob Storage **hacia** Azure Synapse Analytics o una colección de Azure Cosmos DB.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="supported-scenarios"></a>Escenarios admitidos
Puede copiar datos **desde Azure Blob Storage** hacia los siguientes almacenes de datos:

[!INCLUDE [data-factory-supported-sink](includes/data-factory-supported-sinks.md)]

Puede copiar datos desde los siguientes almacenes de datos **hacia Azure Blob Storage**:

[!INCLUDE [data-factory-supported-sources](includes/data-factory-supported-sources.md)]

> [!IMPORTANT]
> La actividad de copia es compatible con la copia de datos desde y hacia cuentas de Azure Storage de propósito general y el almacenamiento de blobs en frío y en caliente. La actividad admite **leer desde blobs en bloques, en anexos o en páginas**, pero solo permite **escribir en blobs en bloques**. Azure Premium Storage no es compatible como receptor porque está respaldado por blobs en páginas.
>
> La actividad de copia no elimina datos del origen una vez copiados los datos correctamente en el destino. Si necesita eliminar datos de origen tras una copia correcta, cree una [actividad personalizada](data-factory-use-custom-activities.md) para tal fin y úsela en la canalización. Para más información, consulte el [ejemplo de eliminación de un blob o una carpeta en GitHub](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/DeleteBlobFileFolderCustomActivity).

## <a name="get-started"></a>Introducción
Puede crear una canalización con actividad de copia que mueva los datos desde Azure Blob Storage o hacia él mediante el uso de diferentes herramientas o API.

La manera más fácil de crear una canalización es usar el **Asistente para copiar**. Este artículo tiene un [tutorial](#walkthrough-use-copy-wizard-to-copy-data-tofrom-blob-storage) para crear una canalización para copiar datos entre ubicaciones de Azure Blob Storage. Para ver un tutorial sobre la creación de una canalización para copiar datos desde una cuenta de Azure Blob Storage en una base de datos de Azure SQL, consulte [Tutorial: crear una canalización con el Asistente para copia](data-factory-copy-data-wizard-tutorial.md).

Puede usar las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** y **API REST**. Consulte el [tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia.

Tanto si usa las herramientas como las API, realice los pasos siguientes para crear una canalización que mueva datos de un almacén de datos de origen a un almacén de datos receptor:

1. Crear una **factoría de datos**. Una factoría de datos puede contener una o más canalizaciones.
2. Cree **servicios vinculados** para vincular almacenes de datos de entrada y salida a la factoría de datos. Por ejemplo, si va a copiar datos de una cuenta de Azure Blob Storage en Azure SQL Database, creará dos servicios vinculados para vincular la cuenta de Azure Storage y Azure SQL Database a su factoría de datos. Para información sobre las propiedades de los servicios vinculados que son específicas de Azure Blob Storage, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).
2. Cree **conjuntos de datos** con el fin de representar los datos de entrada y salida para la operación de copia. En el ejemplo mencionado en el último paso, se crea un conjunto de datos para especificar el contenedor de blobs y la carpeta que contiene los datos de entrada. Además, se crea otro conjunto de datos para especificar la tabla SQL en Azure SQL Database que contiene los datos copiados del almacenamiento de blobs. Para información sobre las propiedades del conjunto de datos que son específicas de Azure Blob Storage, consulte la sección [Propiedades del conjunto de datos](#dataset-properties).
3. Cree una **canalización** con una actividad de copia que tome como entrada un conjunto de datos y un conjunto de datos como salida. En el ejemplo que se ha mencionado anteriormente, se usa BlobSource como origen y SqlSink como receptor para la actividad de copia. De igual forma, si va a copiar desde Azure SQL Database hacia Azure Blob Storage, usará SqlSource y BlobSink en la actividad de copia. Para información sobre las propiedades de la actividad de copia que son específicas de Azure Blob Storage, consulte la sección [Propiedades de la actividad de copia](#copy-activity-properties). Para obtener más información sobre cómo usar un almacén de datos como origen o receptor, haga clic en el vínculo de la sección anterior para el almacén de datos.

Cuando se usa el Asistente, se crean automáticamente definiciones de JSON para estas entidades de Data Factory (servicios vinculados, conjuntos de datos y la canalización). Al usar herramientas o API (excepto la API de .NET), se definen estas entidades de Data Factory con el formato JSON.  Para obtener ejemplos con definiciones de JSON para entidades de Data Factory que se utilizan para copiar datos con Azure Blob Storage como origen y destino, consulte la sección [Ejemplos de JSON](#json-examples-for-copying-data-to-and-from-blob-storage  ) de este artículo.

En las secciones siguientes se proporcionan detalles sobre las propiedades JSON que se usan para definir entidades de Data Factory específicas de Azure Blob Storage.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado
Hay dos tipos de servicios vinculados que puede usar para vincular una instancia de Azure Storage a una instancia de Azure Data Factory. Son las siguientes: Servicio vinculado **AzureStorage** y servicio vinculado **AzureStorageSas**. El servicio vinculado de Azure Storage proporciona a la factoría de datos acceso global a Azure Storage. Mientras que el servicio vinculado de SAS (firma de acceso compartido) de Azure Storage proporciona a la factoría de datos acceso restringido/controlado por tiempo a Azure Storage. No existen otras diferencias entre estos dos servicios vinculados. Elija el servicio vinculado que se adapte a sus necesidades. En las siguientes secciones se ofrecen más detalles sobre estos dos servicios vinculados.

[!INCLUDE [data-factory-azure-storage-linked-services](includes/data-factory-azure-storage-linked-services.md)]

## <a name="dataset-properties"></a>Propiedades del conjunto de datos
Para especificar un conjunto de datos para representar datos de entrada o salida en Azure Blob Storage, establezca la propiedad de tipo del conjunto de datos en **AzureBlob**. Establezca la propiedad **linkedServiceName** del conjunto de datos en el nombre del servicio vinculado Azure Storage o SAS de Azure Storage.  Las propiedades de tipo del conjunto de datos especifican el **contenedor de blobs** y la **carpeta** en Blob Storage.

Para obtener una lista completa de las secciones y propiedades JSON disponibles para definir conjuntos de datos, consulte el artículo [Creación de conjuntos de datos](data-factory-create-datasets.md). Las secciones como structure, availability y policy del código JSON del conjunto de datos son similares para todos los tipos de conjunto de datos (SQL Azure, blob de Azure, tabla de Azure, etc.).

Data Factory admite los siguientes valores de tipo basados en .NET compatible con CLS para proporcionar información de tipo en la sección "structure" de los orígenes de datos de esquema sobre la lectura como un blob de Azure: Int16, Int32, Int64, Single, Double, Decimal, Byte[], Bool, String, Guid, Datetime, Datetimeoffset y Timespan. Data Factory realiza automáticamente las conversiones de tipo al mover datos desde un almacén de datos de origen a un almacén de datos de receptor.

La sección **typeProperties** es diferente para cada tipo de conjunto de datos y proporciona información acerca de la ubicación, el formato, etc. de los datos del almacén de datos. La sección typeProperties del conjunto de datos de tipo **AzureBlob** tiene las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| folderPath |Ruta de acceso para el contenedor y la carpeta en el almacenamiento de blobs. Ejemplo: myblobcontainer\myblobfolder\ |Sí |
| fileName |Nombre del blob. La propiedad fileName es opcional y distingue entre mayúsculas y minúsculas.<br/><br/>Si especifica fileName, la actividad (incluida la copia) funciona en el blob específico.<br/><br/>Cuando no se especifica fileName, la copia incluirá todos los blobs de folderPath para el conjunto de datos de entrada.<br/><br/>Cuando **fileName** no se especifica para un conjunto de datos de salida y **preserveHierarchy** no se especifica en el receptor de la actividad, el nombre del archivo generado tendrá este formato: `Data.<Guid>.txt` (por ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |No |
| partitionedBy |partitionedBy es una propiedad opcional. Puede usarla para especificar un folderPath dinámico y un nombre de archivo para datos de series temporales. Por ejemplo, se puede parametrizar folderPath por cada hora de datos. Consulte la sección [Uso de la propiedad partitionedBy](#using-partitionedby-property) para ver información detallada y ejemplos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Estos son los niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |

### <a name="using-partitionedby-property"></a>Uso de la propiedad partitionedBy
Como ya se ha indicado en la sección anterior, se puede especificar un valor dinámico de folderPath y filename para datos de series temporales con la propiedad **partitionedBy**, [funciones de Data Factory y las variables del sistema](data-factory-functions-variables.md).

Para más información sobre los conjuntos de datos de series temporales, la programación y los segmentos, consulte los artículos [Creación de conjuntos de datos](data-factory-create-datasets.md) y [Programación y ejecución](data-factory-scheduling-and-execution.md).

#### <a name="sample-1"></a>Ejemplo 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

En este ejemplo, {Slice} se reemplaza por el valor de la variable del sistema SliceStart de Data Factory en el formato (YYYYMMDDHH) especificado. SliceStart hace referencia a la hora de inicio del segmento. folderPath es diferente para cada segmento. Por ejemplo: wikidatagateway/wikisampledataout/2014100103 o wikidatagateway/wikisampledataout/2014100104

#### <a name="sample-2"></a>Ejemplo 2

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
[
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```

En este ejemplo, year, month, day y time de SliceStart se extraen en variables independientes que se usan en las propiedades folderPath y fileName.

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia
Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo [Creación de canalizaciones](data-factory-create-pipelines.md). Las propiedades (como nombre, descripción, conjuntos de datos de entrada y salida, y directivas) están disponibles para todos los tipos de actividades. Por otra parte, las propiedades disponibles en la sección **typeProperties** de la actividad varían con cada tipo de actividad. Para la actividad de copia, varían en función de los tipos de orígenes y receptores. Si va a mover datos desde un Azure Blob Storage, establezca el tipo de origen en la actividad de copia en **BlobSource**. De igual forma, si va a mover datos desde un Azure Blob Storage, establezca el tipo de receptor en la actividad de copia en **BlobSink**. Esta sección proporciona una lista de propiedades admitidas por BlobSource y BlobSink.

**BlobSource** admite las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True (valor predeterminado), False |No |

**BlobSink** admite las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| copyBehavior |Define el comportamiento de copia cuando el origen es BlobSource o FileSystem. |<b>PreserveHierarchy:</b> conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen que apunta a la carpeta de origen es idéntica a la ruta de acceso relativa del archivo de destino que apunta a la carpeta de destino.<br/><br/><b>FlattenHierarchy:</b> todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen un nombre generado automáticamente. <br/><br/><b>MergeFiles:</b> combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre de archivo/blob, el nombre de archivo combinado sería el nombre especificado; de lo contrario, sería el nombre de archivo generado automáticamente. |No |

**BlobSource** también admite estas dos propiedades para ofrecer compatibilidad con versiones anteriores.

* **treatEmptyAsNull**: especifica si se debe tratar una cadena nula o vacía como un valor nulo.
* **skipHeaderLineCount** : especifica cuántas líneas deben omitirse. Es aplicable únicamente cuando el conjunto de datos de entrada usa TextFormat.

De forma similar, **BlobSink** admite la siguiente propiedad para ofrecer compatibilidad con versiones anteriores.

* **blobWriterAddHeader**: especifica si se debe agregar un encabezado de definiciones de columna al escribir en un conjunto de datos de salida.

Los conjuntos de datos ahora son compatibles con las siguientes propiedades que implementan la misma funcionalidad: **treatEmptyAsNull**, **skipLineCount**, **firstRowAsHeader**.

En la tabla siguiente se proporciona orientación sobre cómo utilizar las nuevas propiedades de conjunto de datos en lugar de estas propiedades de origen/receptor de blob.

| Propiedad de la actividad de copia | Propiedad de conjunto de datos |
|:--- |:--- |
| skipHeaderLineCount en BlobSource |skipLineCount y firstRowAsHeader. Las líneas se omiten en primer lugar y, a continuación, se lee la primera fila como encabezado. |
| treatEmptyAsNull en BlobSource |treatEmptyAsNull en el conjunto de datos de entrada |
| blobWriterAddHeader en BlobSink |firstRowAsHeader en el conjunto de datos de salida |

Consulte la sección [Especificación de TextFormat](data-factory-supported-file-and-compression-formats.md#text-format) para obtener información detallada acerca de estas propiedades.

### <a name="recursive-and-copybehavior-examples"></a>Ejemplos de recursive y copyBehavior
En esta sección se describe el comportamiento resultante de la operación de copia para diferentes combinaciones de valores recursive y copyBehavior.

| recursive | copyBehavior | Comportamiento resultante |
| --- | --- | --- |
| true |preserveHierarchy |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la misma estructura que la de origen<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5. |
| true |flattenHierarchy |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File5 |
| true |mergeFiles |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 + File3 + File4 + File 5 se combina en un archivo con un nombre de archivo generado automáticamente |
| false |preserveHierarchy |Si la carpeta de origen Folder1 tiene esta estructura: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |
| false |flattenHierarchy |Si la carpeta de origen Folder1 tiene esta estructura:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Nombre de archivo generado automáticamente para File2<br/><br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |
| false |mergeFiles |Si la carpeta de origen Folder1 tiene esta estructura:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>la carpeta de destino Folder1 se crea con la estructura siguiente<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;El contenido de File1 + File2 se combina en un archivo con un nombre de archivo generado automáticamente. Nombre de archivo generado automáticamente para File1<br/><br/>No se selecciona la subcarpeta Subfolder1, que contiene los archivos File3, File4 y File5. |

## <a name="walkthrough-use-copy-wizard-to-copy-data-tofrom-blob-storage"></a>Tutorial: Uso del Asistente para copia para copiar datos a y desde Blob Storage
Vamos a ver cómo copiar rápidamente datos a y desde una instancia de Azure Blob Storage. En este tutorial, los almacenes de datos de origen y destino son de tipo: Azure Blob Storage. La canalización en este tutorial copia datos de una carpeta a otra carpeta en el mismo contenedor de blobs. Este tutorial es sencillo a propósito para mostrar sus valores de configuración o propiedades al usar Blob Storage como origen o receptor.

### <a name="prerequisites"></a>Requisitos previos
1. Si aún no tiene una, cree una **cuenta de Azure Storage** de uso general. En este tutorial usará el almacenamiento de blobs como almacén de datos de **origen** y **destino**. Si no tiene una cuenta de almacenamiento de Azure, consulte la sección [Crear una cuenta de almacenamiento](../../storage/common/storage-account-create.md) para ver los pasos para su creación.
2. Cree un contenedor de blobs denominado **adfblobconnector** en la cuenta de almacenamiento.
4. Cree una carpeta denominada **entrada** en el contenedor **adfblobconnector**.
5. Cree un archivo denominado **emp.txt** con el siguiente contenido y cárguelo en la carpeta **entrada** mediante herramientas como el [Explorador de Azure Storage](https://azurestorageexplorer.codeplex.com/).
    ```json
    John, Doe
    Jane, Doe
    ```

### <a name="create-the-data-factory"></a>Creación de la factoría de datos
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Haga clic en **Crear un recurso** en la esquina superior izquierda, después en **Inteligencia y análisis** y en **Data Factory**.
3. En el panel **Nueva factoría de datos**:  
    1. Escriba **ADFBlobConnectorDF** como **nombre**. El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe el siguiente error: `*Data factory name "ADFBlobConnectorDF" is not available`, cambie el nombre de la factoría de datos (por ejemplo, yournameADFBlobConnectorDF) e intente crearla de nuevo. Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.
    2. Selección la **suscripción** de Azure.
    3. Para Grupo de recursos, seleccione **Usar el existente** para seleccionar un grupo de recursos existente (o) seleccione **Crear nuevo** para escribir un nombre para un grupo de recursos.
    4. Seleccione una **ubicación** para la factoría de datos.
    5. Seleccione la casilla **Anclar al panel** en la parte inferior de la hoja.
    6. Haga clic en **Crear**.
3. Una vez completada la creación, puede ver la hoja **Data Factory** como se muestra en la siguiente imagen:  ![Página principal de Data Factory](./media/data-factory-azure-blob-connector/data-factory-home-page.png)

### <a name="copy-wizard"></a>Asistente para copia
1. En la página principal de Data Factory, haga clic en el icono **Copiar datos** para iniciar el **Asistente para copia de datos** en una pestaña aparte.  

    > [!NOTE]
    > Si ve que el explorador web está atascado en "Autorizando...", deshabilite o desactive la opción **Block third-party cookies and site data** (Bloquear cookies y datos de sitios de terceros) o déjela habilitada y cree una excepción para **login.microsoftonline.com** e intente volver a iniciar el asistente.
2. En la página **Propiedades** :
    1. Escriba **CopyPipeline** en **Task name** (Nombre de tarea). El nombre de la tarea es el nombre de la canalización de la factoría de datos.
    2. Escriba una **descripción** para la tarea (opcional).
    3. En **Task cadence or Task schedule** (Cadencia de tareas o programación de tareas), mantenga la opción **Run regularly on schedule** (Copiar datos periódicamente según programación). Si quiere ejecutar esta tarea solo una vez en lugar de ejecutarla varias veces según una programación, seleccione **Run once now** (Ejecutar una vez ahora). Si selecciona la opción **Run once now** (Ejecutar una vez ahora), se crea una [canalización única](data-factory-create-pipelines.md#onetime-pipeline).
    4. Conserve el valor de configuración de **Recurring pattern** (Patrón de repetición). Esta tarea se ejecuta diariamente entre las horas de inicio y finalización que especifique en el paso siguiente.
    5. Cambie la **fecha y hora de inicio** a **04/21/2017**.
    6. Cambie la **fecha y hora de finalización** a **04/25/2017**. Puede escribir la fecha en lugar de buscarla en el calendario.
    8. Haga clic en **Next**.
        ![Herramienta de copia: Página de propiedades](./media/data-factory-azure-blob-connector/copy-tool-properties-page.png)
3. En la página **Almacén de datos de origen**, haga clic en el icono **Azure Blob Storage**. Use esta página para especificar el almacén de datos de origen para la tarea de copia. Puede usar un servicio vinculado del almacén de datos existente, o bien especificar un almacén de datos nuevo. Para usar un servicio vinculado existente, seleccionaría **FROM EXISTING LINKED SERVICES** (DE SERVICIOS VINCULADOS EXISTENTES) y luego el servicio vinculado correcto.
    ![Herramienta de copia: Página de almacén de datos de origen](./media/data-factory-azure-blob-connector/copy-tool-source-data-store-page.png)
4. En la página **Especificar cuenta de Almacenamiento de blobs de Azure** :
    1. En **Nombre de la conexión**, mantenga el nombre generado automáticamente. El nombre de conexión es el nombre del servicio vinculado de tipo: Azure Storage.
    2. Confirme que la opción **De suscripciones de Azure** está seleccionada para **Método de selección de cuenta**.
    3. Seleccione la suscripción de Azure o mantenga **Seleccionar todo** para **Suscripción de Azure**.
    4. Seleccione una **cuenta de Azure Storage** en la lista de cuentas de Azure Storage disponibles en la suscripción seleccionada. También puede elegir especificar la configuración de la cuenta de almacenamiento manualmente, para lo que debe seleccionar la opción **Enter manually** (Especificar manualmente) en **Account selection method** (Método de selección de cuenta).
    5. Haga clic en **Next**.  
        ![Herramienta de copia: Especificación de la cuenta de Azure Blob Storage](./media/data-factory-azure-blob-connector/copy-tool-specify-azure-blob-storage-account.png)
5. En la página **Elegir el archivo o la carpeta de entrada** :
    1. Haga doble clic en **adfblobcontainer**.
    2. Seleccione **entrada** y haga clic en **Elegir**. En este tutorial, seleccionará la carpeta de entrada. También podría seleccionar en su lugar el archivo emp.txt de la carpeta.
        ![Herramienta de copia: selección del archivo o la carpeta de entrada 1](./media/data-factory-azure-blob-connector/copy-tool-choose-input-file-or-folder.png)
6. En la página **Choose the input file or folder** (Elegir el archivo o la carpeta de entrada):
    1. Confirme que el **archivo o la carpeta** están establecidos en **adfblobconnector/entrada**. Si los archivos se encuentran en subcarpetas, por ejemplo, 2017/04/01, 2017/04/02, etc., escriba adfblobconnector/entrada/{año}/{mes}/{día} para el archivo o la carpeta. Al presionar la tecla de tabulación fuera del cuadro de texto, verá tres listas desplegables para seleccionar el formato de año (aaaa), meso (MM) y día (dd).
    2. No marque la casilla **Copy file recursively** (Copiar archivo de forma recursiva). Seleccione esta opción para recorrer las carpetas de forma recursiva para encontrar los archivos que se van a copiar en el destino.
    3. No marque la opción **Binary copy** (Copia binaria). Seleccione esta opción para realizar una copia binaria del archivo de origen en el destino. No la seleccione para este tutorial, así podrá ver más opciones en las páginas siguientes.
    4. Confirme que **Compression type** (Tipo de compresión) está establecido en **Ninguno**. Seleccione un valor para esta opción si sus archivos de origen están comprimidos en uno de los formatos admitidos.
    5. Haga clic en **Next**.
    ![Herramienta de copia: selección del archivo o la carpeta de entrada 2](./media/data-factory-azure-blob-connector/chose-input-file-folder.png)
7. En la página **Configuración de formato de archivo**, verá los delimitadores y el esquema que el asistente detecta automáticamente al analizar el archivo.
    1. Confirme las siguientes opciones:  
        a. El **formato de archivo** está establecido en **Formato de texto**. Puede ver todos los formatos admitidos en la lista desplegable. Por ejemplo: JSON, Avro, ORC y Parquet.
       b. El **delimitador de columnas** está establecido en `Comma (,)`. Puede ver los demás delimitadores de columna compatibles Data Factory en la lista desplegable. También puede especificar un delimitador personalizado.
       c. El **delimitador de filas** está establecido en `Carriage Return + Line feed (\r\n)`. Puede ver los otros delimitadores de fila admitidos por Data Factory en la lista desplegable. También puede especificar un delimitador personalizado.
       d. El **número de líneas que se omiten** está establecido en **0**. Si quiere que se omitan algunas líneas al principio del archivo, escriba aquí el número.
       e. La casilla **first data row contains column names** (La primera fila de datos contiene nombres de columna) no esta seleccionada. Si los archivos de origen contienen nombres de columna en la primera fila, seleccione esta opción.
       f. La opción **treat empty column value as null** (Tratar el valor de columna vacío como null) está seleccionada.
    2. Expanda **Configuración avanzada** para ver las opciones avanzadas disponibles.
    3. En la parte inferior de la página, obtenga la **vista previa** de datos del archivo emp.txt.
    4. Haga clic en la pestaña **ESQUEMA** en la parte inferior para ver el esquema que dedujo el Asistente para copia examinando los datos del archivo de origen.
    5. Haga clic en **Siguiente** después de revisar los delimitadores y obtener una vista previa de los datos.
    ![Herramienta de copia: Configuración de formato de archivo](./media/data-factory-azure-blob-connector/copy-tool-file-format-settings.png)
8. En la página **Destination data store** (Almacén de datos de destino), seleccione **Azure Blob Storage** y haga clic en **Siguiente**. En este tutorial usará Azure Blob Storage como almacén de datos de origen y de destino.  
    ![Herramienta de copia: Selección del almacén de datos de destino](media/data-factory-azure-blob-connector/select-destination-data-store.png)
9. En la página **Specify the Azure Blob storage account** (Especificar cuenta de Azure Blob Storage):  
    1. Escriba **AzureStorageLinkedService** en el campo **Nombre de la conexión**.
    2. Confirme que la opción **De suscripciones de Azure** está seleccionada para **Método de selección de cuenta**.
    3. Selección la **suscripción** de Azure.
    4. Seleccione su cuenta de almacenamiento de Azure.
    5. Haga clic en **Next**.
10. En la página **Choose the output file or folder** (Elegir el archivo o la carpeta de salida):  
    1. Especifique la **ruta de acceso de carpeta** como **adfblobconnector/output/{year}/{month}/{day}** . Escriba **TAB**.
    1. Para el **año**, seleccione **aaaa**.
    1. Para el **mes**, confirme que está configurado como **MM**.
    1. Para el **día**, confirme que está configurado como **dd**.
    1. Confirme que **Compression type** (Tipo de compresión) está establecido en **Ninguno**.
    1. Confirme que el **comportamiento de copia** está establecido en **Merge files** (Combinar archivos). Si ya existe un archivo de salida con el mismo nombre, el nuevo contenido se agrega al final al mismo archivo.
    1. Haga clic en **Next**.
       ![Herramienta de copia: Selección del archivo o la carpeta de salida](media/data-factory-azure-blob-connector/choose-the-output-file-or-folder.png)
11. En la página **File format settings** (Configuración de formato de archivo), revise la configuración y haga clic en **Siguiente**. Una de las opciones adicionales aquí es agregar un encabezado al archivo de salida. Si selecciona esta opción, se agrega una fila de encabezado con nombres de las columnas del esquema de origen. Puede cambiar los nombres de columna predeterminados al visualizar el esquema para el origen. Por ejemplo, podría cambiar la primera columna a Nombre y la segunda columna a Apellido. El archivo de salida se genera entonces con un encabezado con estos nombres como nombres de columna.
    ![Herramienta de copia: Configuración del formato de archivo de destino](media/data-factory-azure-blob-connector/file-format-destination.png)
12. En la página **Performance settings** (Configuración de rendimiento), confirme que **cloud units** (Unidades de nube) y **parallel copies** (Copias paralelas) están establecidas en **Auto** y haga clic en Siguiente. Para más información sobre esta configuración, consulte [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md#parallel-copy).
    ![Herramienta de copia: Configuración de rendimiento](media/data-factory-azure-blob-connector/copy-performance-settings.png)
14. En la página **Resumen**, revise toda la configuración (propiedades de tareas, configuración de origen y destino y configuración de copia) y haga clic en **Siguiente**.
    ![Herramienta de copia: Página de resumen](media/data-factory-azure-blob-connector/copy-tool-summary-page.png)
15. Revise la información de la página **Resumen** y haga clic en **Finalizar**. El asistente crea dos servicios vinculados, dos conjuntos de datos (entrada y salida) y una canalización en la factoría de datos (desde donde se inició al Asistente para copia).
    ![Herramienta de copia: Página de implementación](media/data-factory-azure-blob-connector/copy-tool-deployment-page.png)

### <a name="monitor-the-pipeline-copy-task"></a>Supervisión de la canalización (tarea de copia)

1. Haga clic en el vínculo `Click here to monitor copy pipeline` de la página **Implementación**.
2. Verá la opción **Monitor and Manage application** (Supervisar y administrar la aplicación) en una pestaña aparte.  ![Supervisar y administrar la aplicación](media/data-factory-azure-blob-connector/monitor-manage-app.png)
3. Cambie la hora de **inicio** de la parte superior a `04/19/2017` y la hora de **finalización** a `04/27/2017`; a continuación, haga clic en **Aplicar**.
4. Verá cinco ventanas de actividad en la lista **ACTIVITY WINDOWS** (VENTANAS DE ACTIVIDAD). Las horas de **WindowStart** deben abarcar todos los días desde las horas de inicio hasta las horas de finalización de la canalización.
5. Haga clic en el botón **Actualizar** de la lista **ACTIVITY WINDOWS** (VENTANAS DE ACTIVIDAD) unas cuantas veces hasta que vea el estado de todas las ventanas de actividad establecido en Listo.
6. Ahora, compruebe que los archivos de salida se generan en la carpeta de salida del contenedor adfblobconnector. Verá la siguiente estructura de carpetas en la carpeta de salida:
    ```
    2017/04/21
    2017/04/22
    2017/04/23
    2017/04/24
    2017/04/25
    ```
   Para más información sobre la supervisión y administración de factorías de datos, consulte el artículo [Supervisión y administración de canalizaciones de Data Factory](data-factory-monitor-manage-app.md).

### <a name="data-factory-entities"></a>Entidades de Factoría de datos
Ahora, vuelva a la pestaña con la página principal de Data Factory. Observe que ahora hay en su factoría de datos dos servicios vinculados, dos conjuntos de datos y una canalización.

![Página principal de Data Factory con entidades](media/data-factory-azure-blob-connector/data-factory-home-page-with-numbers.png)

Haga clic en **Author and deploy** (Crear e implementar) para iniciar Data Factory Editor.

![Editor de la Factoría de datos](media/data-factory-azure-blob-connector/data-factory-editor.png)

Verá las siguientes entidades de Data Factory en su factoría de datos:

- Dos servicios vinculados. Uno para el origen y el otro para el destino. Ambos servicios vinculados hacen referencia a la misma cuenta de Azure Storage de este tutorial.
- Dos conjuntos de datos. Un conjunto de datos de entrada y un conjunto de datos de salida. En este tutorial, ambos usan el mismo contenedor de blobs, pero hacen referencia a carpetas diferentes (entrada y salida).
- Una canalización. La canalización contiene una actividad de copia que usa un origen de blob y un receptor de blob para copiar datos entre ubicaciones de blob de Azure.

En las secciones siguientes se proporciona más información sobre estas entidades.

#### <a name="linked-services"></a>Servicios vinculados
Verá dos servicios vinculados. Uno para el origen y el otro para el destino. En este tutorial, ambas definiciones parecen las mismas, si exceptuamos los nombres. El **tipo** del servicio vinculado se establece en **AzureStorage**. La propiedad más importante de la definición del servicio vinculado es **connectionString**, que se usa en Data Factory para la conexión a la cuenta de Azure Storage en tiempo de ejecución. Omita la propiedad hubName en la definición.

##### <a name="source-blob-storage-linked-service"></a>Servicio vinculado de almacenamiento de blobs de origen
```json
{
    "name": "Source-BlobStorage-z4y",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=**********"
        }
    }
}
```

##### <a name="destination-blob-storage-linked-service"></a>Servicio vinculado de almacenamiento de blobs de destino

```json
{
    "name": "Destination-BlobStorage-z4y",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=**********"
        }
    }
}
```

Para más información sobre el servicio vinculado Azure Storage, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties).

#### <a name="datasets"></a>Conjuntos de datos
Hay dos conjuntos de datos: un conjunto de datos de entrada y un conjunto de datos de salida. El tipo del conjunto de datos está establecido en **AzureBlob** para ambos.

El conjunto de datos de entrada apunta a la carpeta **entrada** del contenedor de blobs **adfblobconnector**. La propiedad **external** está establecida en **true** para este conjunto de datos ya que los datos no se generan mediante la canalización con la actividad de copia que toma este conjunto de datos como entrada.

El conjunto de datos de salida apunta a la carpeta **salida** del mismo contenedor de blobs. También usa el año, mes y día de la variable del sistema **SliceStart** para evaluar de forma dinámica la ruta de acceso del archivo de salida. Para ver la lista de funciones y variables del sistema que admite Data Factory, consulte [Azure Data Factory: funciones y variables del sistema](data-factory-functions-variables.md). La propiedad **external** está establecida en **false** (valor predeterminado) porque este conjunto de datos lo genera la canalización.

Para más información sobre las propiedades admitidas por el conjunto de datos de blob de Azure, consulte la sección [Propiedades del conjunto de datos](#dataset-properties).

##### <a name="input-dataset"></a>Conjunto de datos de entrada

```json
{
    "name": "InputDataset-z4y",
    "properties": {
        "structure": [
            { "name": "Prop_0", "type": "String" },
            { "name": "Prop_1", "type": "String" }
        ],
        "type": "AzureBlob",
        "linkedServiceName": "Source-BlobStorage-z4y",
        "typeProperties": {
            "folderPath": "adfblobconnector/input/",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

##### <a name="output-dataset"></a>Conjunto de datos de salida

```json
{
    "name": "OutputDataset-z4y",
    "properties": {
        "structure": [
            { "name": "Prop_0", "type": "String" },
            { "name": "Prop_1", "type": "String" }
        ],
        "type": "AzureBlob",
        "linkedServiceName": "Destination-BlobStorage-z4y",
        "typeProperties": {
            "folderPath": "adfblobconnector/output/{year}/{month}/{day}",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            },
            "partitionedBy": [
                { "name": "year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
                { "name": "month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
                { "name": "day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }
            ]
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "external": false,
        "policy": {}
    }
}
```

#### <a name="pipeline"></a>Canalización
La canalización tiene una sola actividad. El **tipo** de la actividad se establece en **Copia**. En las propiedades de tipo de la actividad, hay dos secciones, una para el origen y otra para el receptor. El tipo de origen se establece en **BlobSource** ya que la actividad es copiar datos de un almacenamiento de blobs. El tipo de receptor se establece en **BlobSink** ya que la actividad es copiar datos en un almacenamiento de blobs. La actividad de copia toma InputDataset z4y como entrada y OutputDataset-z4y como salida.

Para más información sobre las propiedades admitidas por BlobSource y BlobSink, consulte la sección [Propiedades de la actividad de copia](#copy-activity-properties).

```json
{
    "name": "CopyPipeline",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "copyBehavior": "MergeFiles",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset-z4y"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset-z4y"
                    }
                ],
                "policy": {
                    "timeout": "1.00:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 3,
                    "longRetry": 0,
                    "longRetryInterval": "00:00:00"
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "Activity-0-Blob path_ adfblobconnector_input_->OutputDataset-z4y"
            }
        ],
        "start": "2017-04-21T22:34:00Z",
        "end": "2017-04-25T05:00:00Z",
        "isPaused": false,
        "pipelineMode": "Scheduled"
    }
}
```

## <a name="json-examples-for-copying-data-to-and-from-blob-storage"></a>Ejemplos de JSON para copiar datos hacia y desde Blob Storage
En los siguientes ejemplos, se proporcionan definiciones JSON de ejemplo que puede usar para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Muestran cómo copiar datos entre Azure Blob Storage y Azure SQL Database. Sin embargo, los datos se pueden copiar **directamente** de cualquiera de los orígenes a cualquiera de los receptores indicados [aquí](data-factory-data-movement-activities.md#supported-data-stores-and-formats) mediante la actividad de copia en Azure Data Factory.

### <a name="json-example-copy-data-from-blob-storage-to-sql-database"></a>Ejemplo JSON: Copia de datos de Blob Storage en SQL Database
El ejemplo siguiente muestra:

1. Un servicio vinculado de tipo [AzureSqlDatabase](data-factory-azure-sql-connector.md#linked-service-properties).
2. Un servicio vinculado de tipo [AzureStorage](#linked-service-properties)
3. Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureBlob](#dataset-properties).
4. Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties).
5. Una [canalización](data-factory-create-pipelines.md) con una actividad de copia que usa [BlobSource](#copy-activity-properties) y [SqlSink](data-factory-azure-sql-connector.md#copy-activity-properties).

El ejemplo copia los datos de la serie temporal desde un blob de Azure a una tabla de SQL de Azure cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

**Servicio vinculado SQL de Azure:**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Servicio vinculado de Azure Storage:**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory admite dos tipos de servicios vinculados de Azure Storage: **AzureStorage** y **AzureStorageSas**. En el primer caso, especifique la cadena de conexión que incluye la clave de cuenta. En el segundo, especifique el Uri de firma de acceso compartido (SAS). Para más información, consulte la sección [Servicios vinculados](#linked-service-properties).

**Conjunto de datos de entrada de blob de Azure:**

Los datos se seleccionan de un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta y el nombre de archivo para el blob se evalúan dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa la parte year, month y day de la hora de inicio y el nombre de archivo, la parte hour. El valor "external": "true" informa a Data Factory de que la tabla es externa a la factoría de datos y no la produce ninguna actividad de dicha factoría.

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" } }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
**Conjunto de datos de salida SQL de Azure:**

En el ejemplo, los datos se copian en una tabla denominada "MyTable" de Azure SQL Database. Cree la tabla en la base de datos SQL con el mismo número de columnas que espera que contenga el archivo CSV de blob. Se agregan nuevas filas a la tabla cada hora.

```json
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**Una actividad de copia en una canalización con el origen de blob y el receptor SQL:**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **BlobSource** y el tipo **sink**, en **SqlSink**.

```json
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoSQL",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
### <a name="json-example-copy-data-from-azure-sql-to-azure-blob"></a>Ejemplo JSON: Copia de datos de Azure SQL a un blob de Azure
El ejemplo siguiente muestra:

1. Un servicio vinculado de tipo [AzureSqlDatabase](data-factory-azure-sql-connector.md#linked-service-properties).
2. Un servicio vinculado de tipo [AzureStorage](#linked-service-properties)
3. Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties).
4. Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureBlob](#dataset-properties).
5. Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [SqlSource](data-factory-azure-sql-connector.md#copy-activity-properties) y [BlobSink](#copy-activity-properties).

El ejemplo copia los datos de la serie temporal desde una tabla de SQL de Azure a un blob de Azure cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

**Servicio vinculado SQL de Azure:**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Servicio vinculado de Azure Storage:**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory admite dos tipos de servicios vinculados de Azure Storage: **AzureStorage** y **AzureStorageSas**. En el primer caso, especifique la cadena de conexión que incluye la clave de cuenta. En el segundo, especifique el Uri de firma de acceso compartido (SAS). Para más información, consulte la sección [Servicios vinculados](#linked-service-properties).

**Conjunto de datos de entrada SQL de Azure:**

El ejemplo supone que ha creado una tabla "MyTable" en Azure SQL y que contiene una columna denominada "timestampcolumn" para los datos de series temporales.

Si se establece "external": "true", se informa al servicio Data Factory de que la tabla es externa a la factoría de datos y no la produce ninguna actividad de dicha factoría.

```json
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Conjunto de datos de salida de blob de Azure:**

Los datos se escriben en un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta para el blob se evalúa dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa las partes year, month, day y hours de la hora de inicio.

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
      "partitionedBy": [
        {
          "name": "Year",
          "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" } }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Actividad de copia en una canalización con el origen SQL y el receptor de blob:**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de la canalización JSON, el tipo **source** se establece en **SqlSource** y el tipo **sink**, en **BlobSink**. La consulta SQL especificada para la propiedad **SqlReaderQuery** selecciona los datos de la última hora que se van a copiar.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[
            {
                "name": "AzureSQLtoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureSQLInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "SqlSource",
                        "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

> [!NOTE]
> Para asignar columnas del conjunto de datos de origen a columnas del conjunto de datos del receptor, consulte [Mapping dataset columns in Azure Data Factory](data-factory-map-columns.md) (Asignación de columnas de conjunto de datos en Azure Data Factory).

## <a name="performance-and-tuning"></a>Rendimiento y optimización
Consulte [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md) para más información sobre los factores clave que afectan al rendimiento del movimiento de datos (actividad de copia) en Azure Data Factory y las diversas formas de optimizarlo.
