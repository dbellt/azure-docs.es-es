---
title: Mover datos de Azure API for FHIR a Azure Synapse Analytics
description: En este artículo se describe cómo mover datos de FHIR a Synapse
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 05/13/2021
ms.author: ginle
ms.openlocfilehash: e548361f6cf8d638609dd9daa83a175fa49f3679
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486188"
---
# <a name="moving-data-from-azure-api-for-fhir-to-azure-synapse-analytics"></a>Mover datos de Azure API for FHIR a Azure Synapse Analytics

En este artículo aprenderá un par de maneras de mover datos de Azure API for FHIR [a Azure Synapse Analytics,](https://azure.microsoft.com/services/synapse-analytics/)que es un servicio de análisis ilimitado que reúne la integración de datos, el almacenamiento de datos empresariales y el análisis de big data. 

Mover datos del servidor de FHIR a Synapse implica exportar los datos mediante la operación de FHIR seguido de una serie de pasos para transformar y cargar los datos `$export` en Synapse. Este artículo le ayudará a través de dos de los distintos enfoques, en los que se mostrará cómo convertir recursos de FHIR en formatos tabulares mientras se mueven a Synapse.

* **Carga de datos exportados en Synapse mediante T-SQL:** Use la operación para mover recursos de FHIR a un almacenamiento de blobs de `$export` Azure Data Lake Gen **2 (ADL Gen 2)** en `NDJSON` formato . Cargue los datos del almacenamiento en grupos de **SQL** dedicados o sin servidor en Synapse mediante T-SQL. Convierta estos pasos en una canalización de movimiento de datos sólida mediante [canalizaciones de Synapse](../../synapse-analytics/get-started-pipelines.md).
* **Use las herramientas del repositorio de OSS de canalizaciones de análisis de FHIR:** El repositorio de canalización de Análisis de [FHIR](https://github.com/microsoft/FHIR-Analytics-Pipelines) contiene herramientas que pueden crear una canalización **de Azure Data Factory (ADF)** para mover los datos de FHIR a una carpeta **Common Data Model (CDM)** y desde la carpeta CDM a Synapse.

## <a name="load-exported-data-to-synapse-using-t-sql"></a>Carga de datos exportados en Synapse mediante T-SQL

### <a name="export-for-moving-fhir-data-into-azure-data-lake-gen-2-storage"></a>`$export` para mover datos de FHIR al almacenamiento de Azure Data Lake Gen 2

:::image type="content" source="media/export-data/export-azure-storage-option.png" alt-text="Almacenamiento de Azure en Synapse mediante $export":::

#### <a name="configure-your-fhir-server-to-support-export"></a>Configuración del servidor de FHIR para admitir `$export`

Azure API for FHIR implementa la operación definida por la especificación de FHIR para exportar todos o un subconjunto filtrado de datos `$export` de FHIR en `NDJSON` formato. Además, admite la [exportación sin identificar para](./de-identified-export.md) anonimizar los datos de FHIR durante la exportación. Si usa `$export` , obtiene la característica de des identificación de forma predeterminada; su funcionalidad ya está integrada en `$export` .

Para exportar datos de FHIR a Azure Blob Storage, primero debe configurar el servidor FHIR para exportar datos a la cuenta de almacenamiento. Tendrá que (1) habilitar la identidad administrada, (2) ir a Access Control en la cuenta de almacenamiento y agregar la asignación de roles, (3) seleccionar la cuenta de almacenamiento para `$export` . Puede encontrar más instrucciones paso a paso [aquí.](./configure-export-data.md)

Puede configurar el servidor para exportar los datos a cualquier tipo de cuenta de almacenamiento de Azure, pero se recomienda exportar a ADL Gen 2 para obtener la mejor alineación con Synapse.

#### <a name="using-export-command"></a>Uso del `$export` comando

Después de configurar el servidor de [](./export-data.md#using-export-command) FHIR, puede seguir la documentación para exportar los recursos de FHIR en el nivel de sistema, paciente o grupo. Por ejemplo, puede exportar todos los datos de FHIR relacionados con los pacientes de un con el siguiente comando, en el que especifica el nombre del almacenamiento de blobs de `Group` `$export` ADL Gen 2 en el campo `{{BlobContainer}}` :

```rest
https://{{FHIR service base URL}}/Group/{{GroupId}}/$export?_container={{BlobContainer}}  
```

También puede usar el `_type` parámetro en la llamada anterior para restringir los recursos que desea `$export` exportar. Por ejemplo, la siguiente llamada solo exportará `Patient` los recursos , y `MedicationRequest` `Observation` :

```rest
https://{{FHIR service base URL}}/Group/{{GroupId}}/$export?_container={{BlobContainer}}&
_type=Patient,MedicationRequest,Condition
```  

Para obtener más información sobre los distintos parámetros admitidos, consulte nuestra `$export` sección de página sobre los parámetros de [consulta](./export-data.md#settings-and-parameters).

### <a name="create-a-synapse-workspace"></a>Creación de un área de trabajo de Synapse

Antes de usar Synapse, necesitará un área de trabajo de Synapse. Creará un servicio de Azure Synapse Analytics en Azure Portal. Puede encontrar más instrucciones paso a paso [aquí.](../../synapse-analytics/get-started-create-workspace.md) Necesita una cuenta `ADLSGEN2` para crear un área de trabajo. El Azure Synapse de trabajo usará esta cuenta de almacenamiento para almacenar los datos del área de trabajo de Synapse.

Después de crear un área de trabajo, puede ver el área de trabajo en Synapse Studio iniciando sesión en el área de trabajo en o iniciando Synapse Studio en https://web.azuresynapse.net el Azure Portal.

#### <a name="creating-a-linked-service-between-azure-storage-and-synapse"></a>Creación de un servicio vinculado entre Azure Storage y Synapse

Para mover los datos a Synapse, debe crear un servicio vinculado que conecte su cuenta Azure Storage con Synapse. Puede encontrar más instrucciones paso a paso [aquí.](../../synapse-analytics/data-integration/data-integration-sql-pool.md#create-linked-services)

1. En Synapse Studio, vaya a la **pestaña Administrar** y, en **Conexiones externas,** seleccione **Servicios vinculados.**
2. Seleccione **Nuevo** para agregar un nuevo servicio vinculado.
3. Seleccione **Azure Data Lake Storage Gen2** en la lista y seleccione **Continuar.**
4. Especifique las credenciales de autenticación. Cuando haya terminado, seleccione **Crear**.

Ahora que tiene un servicio vinculado entre el almacenamiento de ADL Gen 2 y Synapse, está listo para usar grupos de Synapse SQL para cargar y analizar los datos de FHIR.

### <a name="decide-between-serverless-and-dedicated-sql-pool"></a>Decidir entre un grupo de SQL dedicado y sin servidor

Azure Synapse Analytics ofrece dos grupos de SQL diferentes, un grupo de SQL sin servidor y un grupo de SQL dedicado. El grupo de SQL sin servidor ofrece la flexibilidad de consultar datos directamente en el almacenamiento de blobs mediante el punto de conexión SQL sin servidor sin aprovisionamiento de recursos. El grupo de SQL dedicado tiene la capacidad de procesamiento para un alto rendimiento y simultaneidad, y se recomienda para las funcionalidades de almacenamiento de datos a escala empresarial. Para más información sobre los dos grupos de SQL, consulte la página de [documentación de Synapse](../../synapse-analytics/sql/overview-architecture.md) sobre la arquitectura de SQL.

#### <a name="using-serverless-sql-pool"></a>Uso de un grupo de SQL sin servidor

Puesto que no tiene servidor, no hay ninguna infraestructura para configurar ni clústeres que mantener. Puede empezar a consultar datos desde Synapse Studio en cuanto se crea el área de trabajo.

Por ejemplo, la siguiente consulta se puede usar para transformar campos seleccionados de `Patient.ndjson` en una estructura tabular:

```sql
SELECT * FROM  
OPENROWSET(bulk 'https://{{youraccount}}.blob.core.windows.net/{{yourcontainer}}/Patient.ndjson', 
FORMAT = 'csv', 
FIELDTERMINATOR ='0x0b', 
FIELDQUOTE = '0x0b')  
WITH (doc NVARCHAR(MAX)) AS rows     
CROSS APPLY OPENJSON(doc)     
WITH ( 
    ResourceId VARCHAR(64) '$.id', 
    Active VARCHAR(10) '$.active', 
    FullName VARCHAR(100) '$.name[0].text', 
    Gender VARCHAR(20) '$.gender', 
       ...
) 
```

En la consulta anterior, la función accede a los archivos de Azure Storage, analiza el texto JSON y devuelve las propiedades de entrada JSON como `OPENROWSET` `OPENJSON` filas y columnas. Cada vez que se ejecuta esta consulta, el grupo de SQL sin servidor lee el archivo del almacenamiento de blobs, analiza el json y extrae los campos.

También puede materializar los resultados en formato Parquet en [una tabla externa](../../synapse-analytics/sql/develop-tables-external-tables.md) para obtener un mejor rendimiento de las consultas, como se muestra a continuación:

```sql
-- Create External data source where the parquet file will be written 
CREATE EXTERNAL DATA SOURCE [MyDataSource] WITH ( 
    LOCATION = 'https://{{youraccount}}.blob.core.windows.net/{{exttblcontainer}}' 
); 
GO 

-- Create External File Format 
CREATE EXTERNAL FILE FORMAT [ParquetFF] WITH ( 
    FORMAT_TYPE = PARQUET, 
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec' 
); 
GO 

CREATE EXTERNAL TABLE [dbo].[Patient] WITH ( 
        LOCATION = 'PatientParquet/', 
        DATA_SOURCE = [MyDataSource], 
        FILE_FORMAT = [ParquetFF] 
) AS 
SELECT * FROM  
OPENROWSET(bulk 'https://{{youraccount}}.blob.core.windows.net/{{yourcontainer}}/Patient.ndjson' 
-- Use rest of the SQL statement from the previous example --
```

#### <a name="using-dedicated-sql-pool"></a>Uso de un grupo de SQL dedicado

El grupo de SQL dedicado admite tablas administradas y una caché jerárquica para el rendimiento en memoria. Puede importar big data con consultas T-SQL sencillas y, a continuación, usar la potencia del motor de consultas distribuidas para ejecutar análisis de alto rendimiento.

La manera más sencilla y rápida de cargar datos desde el almacenamiento en un grupo de SQL dedicado es usar el comando en T-SQL, que puede leer archivos CSV, Parquet y **`COPY`** ORC. Como en la consulta de ejemplo siguiente, use el `COPY` comando para cargar las filas en una estructura `NDJSON` tabular. 

```sql
-- Create table with HEAP, which is not indexed and does not have a column width limitation of NVARCHAR(4000) 
CREATE TABLE StagingPatient ( 
Resource NVARCHAR(MAX) 
) WITH (HEAP) 
COPY INTO StagingPatient 
FROM 'https://{{yourblobaccount}}.blob.core.windows.net/{{yourcontainer}}/Patient.ndjson' 
WITH ( 
FILE_TYPE = 'CSV', 
ROWTERMINATOR='0x0a', 
FIELDQUOTE = '', 
FIELDTERMINATOR = '0x00' 
) 
GO
```

Una vez que tenga las filas JSON en la tabla anterior, puede crear diferentes formatos tabulares de los datos mediante la función y almacenar los `StagingPatient` `OPENJSON` resultados en tablas. Esta es una consulta SQL de ejemplo para crear `Patient` una tabla mediante la extracción de algunos campos del `Patient` recurso:

```sql
SELECT RES.* 
INTO Patient 
FROM StagingPatient
CROSS APPLY OPENJSON(Resource)   
WITH (
    ResourceId VARCHAR(64) '$.id',
    FullName VARCHAR(100) '$.name[0].text',
    FamilyName VARCHAR(50) '$.name[0].family',
    GivenName VARCHAR(50) '$.name[0].given[0]',
    Gender VARCHAR(20) '$.gender',
    DOB DATETIME2 '$.birthDate',
    MaritalStatus VARCHAR(20) '$.maritalStatus.coding[0].display',
    LanguageOfCommunication VARCHAR(20) '$.communication[0].language.text'
) AS RES 
GO

```

## <a name="use-fhir-analytics-pipelines-oss-tools"></a>Uso de herramientas de OSS de canalizaciones de análisis de FHIR

:::image type="content" source="media/export-data/analytics-pipeline-option.png" alt-text="Uso de la canalización de ADF para mover datos a la carpeta de CDM y, a continuación, a Synapse":::

> [!Note]
> [La canalización de Análisis de FHIR](https://github.com/microsoft/FHIR-Analytics-Pipelines) es una herramienta de código abierto publicada con licencia MIT y no está cubierta por el Acuerdo de Nivel de Servicio de Microsoft para los servicios de Azure.

### <a name="adf-pipeline-for-moving-fhir-data-into-cdm-folder"></a>Canalización de ADF para mover datos de FHIR a la carpeta de CDM

Common Data Model carpeta (CDM) es una carpeta de un lago de datos que se ajusta a estructuras de metadatos bien definidas y estandarizadas y datos autodescriptos. Estas carpetas facilitan la interoperabilidad de metadatos entre los productores de datos y los consumidores de datos. Antes de mover los datos de FHIR a la carpeta de CDM, puede transformar los datos en una configuración de tabla.

### <a name="generating-table-configuration"></a>Generación de la configuración de tabla

Clone el repositorio para obtener todos los scripts y el código fuente. Use `npm install` para instalar las dependencias. Ejecute el siguiente comando desde la `Configuration-Generator` carpeta para generar una carpeta de configuración de tabla con instrucciones de formato YAML:

```bash
Configuration-Generator> node .\generate_from_yaml.js -r {resource configuration file} -p {properties group file} -o {output folder}
```

Puede usar los archivos de `YAML` ejemplo y se proporcionan en el `resourcesConfig.yml` `propertiesGroupConfig.yml` repositorio.

### <a name="generating-adf-pipeline"></a>Generación de una canalización de ADF

Ahora puede usar el contenido de la configuración de tabla generada y algunas otras configuraciones para generar una canalización de ADF. Esta canalización de ADF, cuando se desencadena, exporta los datos desde el servidor de FHIR mediante la API y escribe en una carpeta de CDM junto con los `$export` metadatos de CDM asociados.

1. Cree una aplicación Azure Active Directory (AD) y una entidad de servicio. La canalización de ADF usa un servicio de Azure Batch para realizar la transformación y necesita una Azure AD para el servicio batch. Siga [Azure AD documentación](../../active-directory/develop/howto-create-service-principal-portal.md).
2. Conceda acceso para exportar la ubicación de almacenamiento a la entidad de servicio. En del `Access Control` almacenamiento de exportación, conceda `Storage Blob Data Contributor` el rol a la Azure AD aplicación.
3. Implemente la canalización de salida. Use la plantilla `fhirServiceToCdm.json` para una implementación personalizada en Azure. Este paso creará los siguientes recursos de Azure:
    - Una canalización de ADF con el nombre `{pipelinename}-df` .
    - Un almacén de claves con el nombre `{pipelinename}-kv` para almacenar el secreto de cliente.
    - Una cuenta de batch con el nombre `{pipelinename}batch` para ejecutar la transformación.
    - Una cuenta de almacenamiento con el nombre `{pipelinename}storage` .
4. Conceda acceso al Azure Data Factory. En el panel de control de acceso del servicio FHIR, conceda `FHIR data exporter` los roles y a la `FHIR data reader` factoría de datos, `{pipelinename}-df` .
5. Cargue el contenido de la carpeta de configuración de tabla en el contenedor de configuración.
6. Vaya a `{pipelinename}-df` y desencadene la canalización. Debería ver los datos exportados en la carpeta CDM de la cuenta de almacenamiento `{pipelinename}storage` . Debería ver una carpeta para cada tabla que tenga un archivo CSV.

### <a name="from-cdm-folder-to-synapse"></a>De la carpeta CDM a Synapse

Una vez que tenga los datos exportados en formato CDM y almacenados en el almacenamiento de ADL Gen 2, ahora puede mover los datos de la carpeta CDM a Synapse.

Puede crear una canalización de CDM a Synapse mediante un archivo de configuración, que tendría un aspecto parecido al siguiente:

```json
{
  "ResourceGroup": "",
  "TemplateFilePath": "../Templates/cdmToSynapse.json",
  "TemplateParameters": {
    "DataFactoryName": "",
    "SynapseWorkspace": "",
    "DedicatedSqlPool": "",
    "AdlsAccountForCdm": "",
    "CdmRootLocation": "cdm",
    "StagingContainer": "adfstaging",
    "Entities": ["LocalPatient", "LocalPatientAddress"]
  }
}
```

Ejecute este script con el archivo de configuración anterior:

```bash
.\DeployCdmToSynapsePipeline.ps1 -Config: config.json
```

Agregue la identidad administrada de ADF como usuario de SQL a SQL Database. Este es un script SQL de ejemplo para crear un usuario y un rol de asignación:

```sql
CREATE USER [datafactory-name] FROM EXTERNAL PROVIDER
GO
EXEC sp_addrolemember db_owner, [datafactory-name]
GO
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido dos maneras diferentes de mover los datos de FHIR a Synapse: (1) mediante el uso de para mover datos al almacenamiento de blobs de ADL Gen 2 y, a continuación, cargar los datos en grupos de Synapse SQL y (2) mediante la canalización de ADF para mover datos de FHIR a la carpeta `$export` CDM y, a continuación, a Synapse.

A continuación, puede obtener información sobre la anonimización de los datos de FHIR al mover datos a Synapse para asegurarse de que la información de atención sanitaria está protegida:
 
>[!div class="nextstepaction"]
>[Exportación de datos no identificados](./de-identified-export.md)








