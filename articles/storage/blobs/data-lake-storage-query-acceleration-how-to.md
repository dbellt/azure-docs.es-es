---
title: Filtro de datos mediante la aceleración de consultas de Azure Data Lake Storage | Microsoft Docs
description: Use la aceleración de consultas para recuperar un subconjunto de datos de una cuenta de almacenamiento.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 01/06/2021
ms.author: normesta
ms.reviewer: jamsbak
ms.custom: devx-track-csharp
ms.openlocfilehash: 58b8cdef604861342a6489ef4e57ff1d057cd3f4
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377741"
---
# <a name="filter-data-by-using-azure-data-lake-storage-query-acceleration"></a>Filtro de datos mediante la aceleración de consultas de Azure Data Lake Storage

En este artículo se muestra cómo usar la aceleración de consultas para recuperar un subconjunto de datos de la cuenta de almacenamiento. 

La aceleración de consultas permite tanto a las aplicaciones como a los marcos de análisis optimizar considerablemente el procesamiento de datos mediante la recuperación solo de los datos necesarios para realizar una operación determinada. Para más información, consulte [Aceleración de consultas de Azure Data Lake Storage](data-lake-storage-query-acceleration.md).

## <a name="prerequisites"></a>Requisitos previos

- Para acceder a Azure Storage, necesitará una suscripción de Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Una cuenta de almacenamiento de **uso general v2**. Vea [Creación de una cuenta de Azure Storage](../common/storage-account-create.md).

- Elija una pestaña para ver todos los requisitos previos específicos del SDK.

  ### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

  No es aplicable

  ### <a name="net"></a>[.NET](#tab/dotnet)

  El [SDK de .NET](https://dotnet.microsoft.com/download) 

  ### <a name="java"></a>[Java](#tab/java)

  - [Kit de desarrollo de Java (JDK)](/java/azure/jdk/), versión 8 o posterior

  - [Apache Maven](https://maven.apache.org/download.cgi) 

    > [!NOTE] 
    > En este artículo se considera que ha creado un proyecto de Java mediante Apache Maven. Para obtener un ejemplo de cómo crear un proyecto con Apache Maven, consulte [Instalación](storage-quickstart-blobs-java.md#setting-up).
  
  ### <a name="python"></a>[Python](#tab/python)

  [Python](https://www.python.org/downloads/)3.8 o superior.

  ### <a name="nodejs"></a>[Node.js](#tab/nodejs)

  No hay requisitos previos adicionales necesarios para usar el SDK de Node.js.

---

## <a name="enable-query-acceleration"></a>Habilitación de la aceleración de consultas

Para usar la aceleración de consultas, debe registrar la característica de aceleración de consultas en su suscripción. Una vez que haya comprobado que la característica está registrada, debe registrar el proveedor de recursos de Azure Storage. 

### <a name="step-1-register-the-query-acceleration-feature"></a>Paso 1: Registrar la característica de aceleración de consultas

Para usar la aceleración de consultas, antes debe registrar la característica de aceleración de consultas en su suscripción. 

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Abra una ventana de comando de Windows PowerShell.

1. Inicie sesión en la suscripción a Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

   ```powershell
   Connect-AzAccount
   ```

2. Si su identidad está asociada a más de una suscripción, establezca la suscripción activa.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

3. Utilice el comando [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature) para registrar la característica de aceleración de consultas.

   ```powershell
   Register-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName BlobQuery
   ```

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Abra [Azure Cloud Shell](../../cloud-shell/overview.md) o, si ha [instalado](/cli/azure/install-azure-cli) la CLI de Azure localmente, abra una aplicación de consola de comandos como Windows PowerShell.

2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

3. Utilice el comando [az feature register](/cli/azure/feature#az-feature-register) para registrar la característica de aceleración de consultas.

   ```azurecli
   az feature register --namespace Microsoft.Storage --name BlobQuery
   ```

---

### <a name="step-2-verify-that-the-feature-is-registered"></a>Paso 2: Comprobar si la característica está registrada

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar si el registro se ha completado, utilice el comando [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName BlobQuery
```

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para comprobar si el registro se ha completado, utilice el comando [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage --name BlobQuery
```

---

### <a name="step-3-register-the-azure-storage-resource-provider"></a>Paso 3: Registro del proveedor de recursos de Azure Storage

Una vez aprobado el registro, debe volver a registrar el proveedor de recursos de Azure Storage. 

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Para registrar el proveedor de recursos, llame al comando [Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider).

```powershell
Register-AzResourceProvider -ProviderNamespace 'Microsoft.Storage'
```

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para registrar el proveedor de recursos, llame al comando [az provider register](/cli/azure/provider#az-provider-register).

```azurecli
az provider register --namespace 'Microsoft.Storage'
```

---

## <a name="set-up-your-environment"></a>Configurar el entorno

### <a name="step-1-install-packages"></a>Paso 1: Instalar paquetes 

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Instale la versión 4.6.0 o superior del módulo Az.

```powershell
Install-Module -Name Az -Repository PSGallery -Force
```

Para actualizar desde una versión anterior de Az, ejecute el siguiente comando:

```powershell
Update-Module -Name Az
```

#### <a name="net"></a>[.NET](#tab/dotnet)

1. Abra un símbolo del sistema y cambie el directorio (`cd`) a la carpeta del proyecto, por ejemplo:

   ```console
   cd myProject
   ```

2. Instale la versión `12.5.0-preview.6` o posterior de biblioteca cliente de Azure Blob Storage para el paquete .NET mediante el comando `dotnet add package`. 

   ```console
   dotnet add package Azure.Storage.Blobs -v 12.8.0
   ```

3. En los ejemplos que aparecen en este artículo se analiza un archivo CSV mediante la biblioteca de [CsvHelper](https://www.nuget.org/packages/CsvHelper/). Para usar esa biblioteca, use el siguiente comando.

   ```console
   dotnet add package CsvHelper
   ```

#### <a name="java"></a>[Java](#tab/java)

1. Abra el archivo *pom.xml* del proyecto en un editor de texto. Agregue los siguientes elementos de dependencia al grupo de dependencias. 

   ```xml
   <!-- Request static dependencies from Maven -->
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-core</artifactId>
       <version>1.6.0</version>
   </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-csv</artifactId>
        <version>1.8</version>
    </dependency>    
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-storage-blob</artifactId>
      <version>12.8.0-beta.1</version>
    </dependency>
   ```

#### <a name="python"></a>[Python](#tab/python)

Instale la biblioteca de cliente de Azure Data Lake Storage para Python con [PIP](https://pypi.org/project/pip/).

```
pip install azure-storage-blob==12.4.0
```

#### <a name="nodejs"></a>[Node.js](#tab/nodejs)

Instale la biblioteca cliente de Data Lake para JavaScript; para ello, abra una ventana de terminal y, a continuación, escriba el siguiente comando.

```javascript
    npm install @azure/storage-blob
    npm install @fast-csv/parse
```

---

### <a name="step-2-add-statements"></a>Paso 2: Adición de instrucciones

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

No es aplicable

#### <a name="net"></a>[.NET](#tab/dotnet)

Agregue estas instrucciones `using` al inicio del archivo de código.

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs.Specialized;
```

La aceleración de consultas recupera datos con formato CSV y JSON. Por lo tanto, asegúrese de agregar instrucciones USING para las bibliotecas de análisis de CSV o JSON que decida usar. En los ejemplos que aparecen en este artículo se analiza un archivo CSV mediante la biblioteca de [CsvHelper](https://www.nuget.org/packages/CsvHelper/) que está disponible en NuGet. Por lo tanto, se agregan estas instrucciones `using` al inicio del archivo de código.

```csharp
using CsvHelper;
using CsvHelper.Configuration;
```

Para compilar los ejemplos presentados en este artículo, también deberá agregar estas instrucciones `using`.

```csharp
using System.Threading.Tasks;
using System.IO;
using System.Globalization;
```

#### <a name="java"></a>[Java](#tab/java)

Agregue estas instrucciones `import` al inicio del archivo de código.

```java
import com.azure.storage.blob.*;
import com.azure.storage.blob.options.*;
import com.azure.storage.blob.models.*;
import com.azure.storage.common.*;
import java.io.*;
import java.util.function.Consumer;
import org.apache.commons.csv.*;
```

#### <a name="python"></a>[Python](#tab/python)

Agregue estas instrucciones import al inicio del archivo de código.

```python
import sys, csv
from azure.storage.blob import BlobServiceClient, ContainerClient, BlobClient, DelimitedTextDialect, BlobQueryError
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

Incluya el módulo `storage-blob` colocando esta instrucción al principio del archivo del código. 

```javascript
const { BlobServiceClient } = require("@azure/storage-blob");
```

La aceleración de consultas recupera datos con formato CSV y JSON. Por consiguiente, asegúrese de agregar instrucciones para todos los módulo de análisis de CSV o JSON que decida usar. En los ejemplos que aparecen en este artículo se analiza un archivo CSV mediante el módulo [fast-csv](https://www.npmjs.com/package/fast-csv). Por consiguiente, agregaremos esta instrucción al principio del archivo de código.

```javascript
const csv = require('@fast-csv/parse');
```

---

## <a name="retrieve-data-by-using-a-filter"></a>Recuperación de datos mediante un filtro

Puede usar SQL para especificar los predicados de filtro de fila y las proyecciones de columna en una solicitud de aceleración de consultas. En el código siguiente se consulta un archivo CSV en el almacenamiento y se devuelven todas las filas de datos en las que la tercera columna coincide con el valor `Hemingway, Ernest`. 

- En la consulta SQL, se usa la palabra clave `BlobStorage` para indicar el archivo que se consulta.

- Las referencias de columna se especifican como `_N`, donde la primera columna es `_1`. Si el archivo de código fuente contiene una fila de encabezado, puede hacer referencia a las columnas por el nombre especificado en la fila de encabezado. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$blob = "csv/csv-general/seattle-library.csv"
Get-QueryCsv $ctx $container $blob "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'" $false

```

### <a name="net"></a>[.NET](#tab/dotnet)

El método asincrónico `BlobQuickQueryClient.QueryAsync` envía la consulta a la API de aceleración de consultas y, a continuación, vuelve a transmitir los resultados a la aplicación como un objeto [Stream](/dotnet/api/system.io.stream).

```cs
static async Task QueryHemingway(BlockBlobClient blob)
{
    string query = @"SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    await DumpQueryCsv(blob, query, false);
}

private static async Task DumpQueryCsv(BlockBlobClient blob, string query, bool headers)
{
    try
    {
        var options = new BlobQueryOptions() {
            InputTextConfiguration = new BlobQueryCsvTextOptions() { HasHeaders = headers },
            OutputTextConfiguration = new BlobQueryCsvTextOptions() { HasHeaders = true },
            ProgressHandler = new Progress<long>((finishedBytes) => Console.Error.WriteLine($"Data read: {finishedBytes}"))
        };
        options.ErrorHandler += (BlobQueryError err) => {
            Console.ForegroundColor = ConsoleColor.Red;
            Console.Error.WriteLine($"Error: {err.Position}:{err.Name}:{err.Description}");
            Console.ResetColor();
        };
        // BlobDownloadInfo exposes a Stream that will make results available when received rather than blocking for the entire response.
        using (var reader = new StreamReader((await blob.QueryAsync(
                query,
                options)).Value.Content))
        {
            using (var parser = new CsvReader(reader, new CsvConfiguration(CultureInfo.CurrentCulture, hasHeaderRecord: true) { HasHeaderRecord = true }))
            {
                while (await parser.ReadAsync())
                {
                    Console.Out.WriteLine(String.Join(" ", parser.Parser.Record));
                }
            }
        }
    }
    catch (Exception ex)
    {
        Console.Error.WriteLine("Exception: " + ex.ToString());
    }
}

```

### <a name="java"></a>[Java](#tab/java)

El método `BlobQuickQueryClient.openInputStream()` envía la consulta a la API de aceleración de consultas y, a continuación, vuelve a transmitir los resultados a la aplicación como un objeto `InputStream` que se puede leer como cualquier otro objeto InputStream.

```java
static void QueryHemingway(BlobClient blobClient) {
    String expression = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    DumpQueryCsv(blobClient, expression, true);
}

static void DumpQueryCsv(BlobClient blobClient, String query, Boolean headers) {
    try {
        BlobQuerySerialization input = new BlobQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(headers)
            .setFieldQuote('\0')
            .setEscapeChar('\\');
        BlobQuerySerialization output = new BlobQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(true)
            .setFieldQuote('\0')
            .setEscapeChar('\n');
        Consumer<BlobQueryError> errorConsumer = System.out::println;
        Consumer<BlobQueryProgress> progressConsumer = progress -> System.out.println("total bytes read: " + progress.getBytesScanned());
        BlobQueryOptions queryOptions = new BlobQueryOptions(query)
            .setInputSerialization(input)
            .setOutputSerialization(output)
            .setErrorConsumer(errorConsumer)
            .setProgressConsumer(progressConsumer);            

        /* Open the query input stream. */
        InputStream stream = blobClient.openQueryInputStream(queryOptions).getValue();
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(stream))) {
            /* Read from stream like you normally would. */
            for (CSVRecord record : CSVParser.parse(reader, CSVFormat.EXCEL.withHeader())) {
                System.out.println(record.toString());
            }
        }
    } catch (Exception e) {
        System.err.println("Exception: " + e.toString());
        e.printStackTrace(System.err);
    }
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_hemingway(blob: BlobClient):
    query = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'"
    dump_query_csv(blob, query, False)

def dump_query_csv(blob: BlobClient, query: str, headers: bool):
    qa_reader = blob.query_blob(query, blob_format=DelimitedTextDialect(has_header=headers), on_error=report_error, encoding='utf-8')
    # records() returns a generator that will stream results as received. It will not block pending all results.
    csv_reader = csv.reader(qa_reader.records())
    for row in csv_reader:
        print("*".join(row))
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

En este ejemplo se envía la consulta a la API de aceleración de consultas y luego se hace streaming de los resultados. El objeto `blob` que se pasa a la función auxiliar `queryHemingway` es del tipo [BlockBlobClient](/javascript/api/@azure/storage-blob/blockblobclient). Para obtener más información acerca de cómo obtener un objeto [BlockBlobClient](/javascript/api/@azure/storage-blob/blockblobclient), consulte [Inicio rápido: Administración de blobs con el SDK de JavaScript v12 en Node.js](storage-quickstart-blobs-nodejs.md).

```javascript
async function queryHemingway(blob)
{
    const query = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    await dumpQueryCsv(blob, query, false);
}

async function dumpQueryCsv(blob, query, headers)
{
    var response = await blob.query(query, {
        inputTextConfiguration: {
            kind: "csv",
            recordSeparator: '\n',
            hasHeaders: headers
        },
        outputTextConfiguration: {
            kind: "csv",
            recordSeparator: '\n',
            hasHeaders: true
        },
        onProgress: (progress) => console.log(`Data read: ${progress.loadedBytes}`),
        onError: (err) => console.error(`Error: ${err.position}:${err.name}:${err.description}`)});
    return new Promise(
        function (resolve, reject) {
            csv.parseStream(response.readableStreamBody)
                .on('data', row => console.log(row))
                .on('error', error => {
                    console.error(error);
                    reject(error);
                })
                .on('end', rowCount => resolve());
    });
}
```

---

## <a name="retrieve-specific-columns"></a>Recuperación de columnas específicas

Puede limitar los resultados a un subconjunto de columnas. De este modo, solo recuperará las columnas necesarias para realizar un cálculo determinado. Esto mejora el rendimiento de la aplicación y reduce los costos porque se transfieren menos datos a través de la red. 

Este código solo recupera la columna `BibNum` para todos los libros del conjunto de datos. También utiliza la información de la fila de encabezado del archivo de origen para hacer referencia a las columnas de la consulta.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$blob = "csv/csv-general/seattle-library-with-headers.csv"
Get-QueryCsv $ctx $container $blob "SELECT BibNum FROM BlobStorage" $true

```

### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryBibNum(BlockBlobClient blob)
{
    string query = @"SELECT BibNum FROM BlobStorage";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryBibNum(BlobClient blobClient)
{
    String expression = "SELECT BibNum FROM BlobStorage";
    DumpQueryCsv(blobClient, expression, true);
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_bibnum(blob: BlobClient):
    query = "SELECT BibNum FROM BlobStorage"
    dump_query_csv(blob, query, True)
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
async function queryBibNum(blob)
{
    const query = "SELECT BibNum FROM BlobStorage";
    await dumpQueryCsv(blob, query, true);
}
```

---

El código siguiente combina el filtrado de filas y las proyecciones de columnas en la misma consulta. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Get-QueryCsv $ctx $container $blob $query $true

Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType 
            FROM BlobStorage 
            WHERE ItemType IN 
                ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')"

```

### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryDvds(BlockBlobClient blob)
{
    string query = @"SELECT BibNum, Title, Author, ISBN, Publisher, ItemType 
        FROM BlobStorage 
        WHERE ItemType IN 
            ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryDvds(BlobClient blobClient)
{
    String expression = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType " +
                        "FROM BlobStorage " +
                        "WHERE ItemType IN " +
                        "   ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    DumpQueryCsv(blobClient, expression, true);
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_dvds(blob: BlobClient):
    query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType "\
        "FROM BlobStorage "\
        "WHERE ItemType IN "\
        "   ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')"
    dump_query_csv(blob, query, True)
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
async function queryDvds(blob)
{
    const query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType " +
                  "FROM BlobStorage " +
                  "WHERE ItemType IN " + 
                  " ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    await dumpQueryCsv(blob, query, true);
}
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Aceleración de consultas de Azure Data Lake Storage](data-lake-storage-query-acceleration.md)
- [Referencia del lenguaje SQL de aceleración de consultas](query-acceleration-sql-reference.md)
