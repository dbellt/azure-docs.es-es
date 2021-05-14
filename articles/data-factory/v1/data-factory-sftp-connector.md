---
title: Movimiento de datos desde un servidor SFTP mediante Azure Data Factory
description: Aprenda a mover datos desde un servidor SFTP local o de nube mediante Azure Data Factory.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/12/2018
robots: noindex
ms.openlocfilehash: 99f914ff793accbea4d348e83e2a515dfd2368e8
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108741368"
---
# <a name="move-data-from-an-sftp-server-using-azure-data-factory"></a>Movimiento de datos de un servidor FTP mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-sftp-connector.md)
> * [Versión 2 (versión actual)](../connector-sftp.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [SFTPconnector in V2](../connector-sftp.md) (Conector SFTP en V2).

En este artículo se describe el uso de la actividad de copia en Azure Data Factory para mover datos de un servidor SFTP local o de nube a un almacén de datos receptor compatible. Este artículo se basa en el artículo sobre las [actividades de movimiento de datos](data-factory-data-movement-activities.md), que presenta una introducción general del movimiento de datos con la actividad de copia la lista de almacenes de datos compatibles como orígenes/receptores.

Data Factory solo admite actualmente el movimiento de datos de un servidor SFTP a otros almacenes de datos, pero no al contrario. Admite servidores SFTP tanto locales como de nube.

> [!NOTE]
> La actividad de copia no elimina el archivo de origen una vez copiado correctamente en el destino. Si necesita eliminar el archivo de origen tras una copia correcta, cree una actividad personalizada para tal fin y úsela en la canalización.

## <a name="supported-scenarios-and-authentication-types"></a>Escenarios admitidos y tipos de autenticación
Puede usar este conector SFTP para copiar datos de **servidores SFTP locales y de nube**. Se admiten los tipos de autenticación **Básica** y **SshPublicKey** al conectar con el servidor SFTP.

Al copiar datos de un servidor SFTP local, deberá instalar una puerta de enlace de administración de datos en la máquina virtual de Azure o en el entorno local. Consulte [Puerta de enlace de administración de datos](data-factory-data-management-gateway.md) para detalles sobre la puerta de enlace. Consulte el artículo sobre cómo [mover datos entre ubicaciones locales y la nube](data-factory-move-data-between-onprem-and-cloud.md) para instrucciones paso a paso sobre cómo configurar y usar la puerta de enlace.

## <a name="getting-started"></a>Introducción
Puede crear una canalización con una actividad de copia que mueva los datos desde un origen SFTP mediante diferentes herramientas o API.

- La manera más fácil de crear una canalización es usar el **Asistente para copiar**. Consulte [Tutorial: Creación de una canalización mediante el Asistente para copia](data-factory-copy-data-wizard-tutorial.md) para ver un tutorial rápido sobre la creación de una canalización utilizando el Asistente para copia de datos.

- Puede usar las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** y **API REST**. Consulte el [tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia. Para obtener ejemplos de JSON para copiar datos desde un servidor SFTP a Azure Blob Storage, consulte la sección [Ejemplo JSON: Copia de datos de un servidor SFTP a un blob de Azure](#json-example-copy-data-from-sftp-server-to-azure-blob) de este artículo.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado
En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio vinculado de FTP.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type | La propiedad type debe establecerse en `Sftp`. |Sí |
| host | Nombre o dirección IP del servidor SFTP. |Sí |
| port |Puerto en el que escucha el servidor SFTP. El valor predeterminado es: 21 |No |
| authenticationType |Especifique el tipo de autenticación. Valores permitidos: **Basic**, **SshPublicKey**. <br><br> Consulte las secciones [Uso de autenticación básica](#using-basic-authentication) y [Uso de autenticación de clave pública SSH](#using-ssh-public-key-authentication) sobre más propiedades y ejemplos JSON respectivamente. |Sí |
| skipHostKeyValidation | Especifique si desea omitir la validación de claves de host. | No. El valor predeterminado es false. |
| hostKeyFingerprint | Especifique la huella dactilar de la clave de host. | Sí, si `skipHostKeyValidation` está establecido en false.  |
| gatewayName |Nombre de la puerta de enlace de administración de datos para conectarse a un servidor SFTP local. | Sí, si va a copiar datos desde un servidor SFTP local. |
| encryptedCredential | Credenciales cifradas para acceder al servidor SFTP. Se generan automáticamente cuando se especifica la autenticación básica (nombre de usuario + contraseña) o la autenticación SshPublicKey (nombre de usuario + ruta de acceso o contenido de la clave privada) en el Asistente para copiar o el cuadro de diálogo emergente ClickOnce. | No. Se aplica solo cuando se copian datos desde un servidor SFTP local. |

### <a name="using-basic-authentication"></a>Uso de autenticación básica

Para usar la autenticación básica, establezca `authenticationType` como `Basic`, y especifique las siguientes propiedades además de las genéricas del conector SFTP descritas en la última sección:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| username | Usuario que tiene acceso al servidor SFTP. |Sí |
| password | Contraseña para el usuario (nombre de usuario). | Sí |

#### <a name="example-basic-authentication"></a>Ejemplo: Autenticación básica
```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "password": "xxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
    }
}
```

#### <a name="example-basic-authentication-with-encrypted-credential"></a>Ejemplo: Autenticación básica con credenciales cifradas

```JSON
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
      }
}
```

### <a name="using-ssh-public-key-authentication"></a>Uso de autenticación de clave pública SSH

Para usar la autenticación de clave pública SSH, establezca `authenticationType` como `SshPublicKey`, y especifique las siguientes propiedades además de las genéricas del conector SFTP descritas en la última sección:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| username |Usuario que tiene acceso al servidor SFTP. |Sí |
| privateKeyPath | Especifique una ruta de acceso absoluta al archivo de clave privada al que pueda acceder la puerta de enlace. | Especifique `privateKeyPath` o `privateKeyContent`. <br><br> Se aplica solo cuando se copian datos desde un servidor SFTP local. |
| privateKeyContent | Una cadena serializada del contenido de la clave privada. El Asistente para copiar puede leer el archivo de clave privada y extraer automáticamente el contenido de la clave privada. Si va a usar cualquier otra herramienta o SDK, utilice en su lugar la propiedad privateKeyPath. | Especifique `privateKeyPath` o `privateKeyContent`. |
| passPhrase | Especifique la contraseña o la frase de contraseña para descifrar la clave privada si el archivo de clave está protegido por una frase de contraseña. | Sí, si el archivo de clave privada está protegido por una frase de contraseña. |

> [!NOTE]
> El conector SFTP solo admite la clave OpenSSH de RSA/DSA. Asegúrese de que el contenido del archivo de claves comienza con "-----BEGIN [RSA/DSA] PRIVATE KEY-----". Si el archivo de clave privada es un archivo con formato ppk, use la herramienta Putty para convertir del formato .ppk a OpenSSH.

#### <a name="example-sshpublickey-authentication-using-private-key-filepath"></a>Ejemplo: Autenticación de SshPublicKey mediante el valor de filePath de clave privada

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyPath",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true,
            "gatewayName": "mygateway"
        }
    }
}
```

#### <a name="example-sshpublickey-authentication-using-private-key-content"></a>Ejemplo: Autenticación de SshPublicKey mediante contenido de clave privada

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyContent",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver.westus.cloudapp.azure.com",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyContent": "<base64 string of the private key content>",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos
Para una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, vea el artículo [Creación de conjuntos de datos](data-factory-create-datasets.md). Las secciones como estructura, disponibilidad y directiva de un JSON de conjunto de datos son similares para todos los tipos de conjunto de datos.

La sección **typeProperties** es diferente para cada tipo de conjunto de datos. Proporciona información específica del tipo de conjunto de datos. La sección typeProperties del conjunto de datos de tipo **FileShare** tiene las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| folderPath |Subruta de acceso a la carpeta. Use el carácter de escape "\" en los caracteres especiales de la cadena. Consulte los casos que se exponen en Ejemplos de definiciones de servicio vinculado y conjunto de datos.<br/><br/>Puede combinar esta propiedad con **partitionBy** para que las rutas de acceso de carpeta se basen en las fechas y horas de inicio y finalización del segmento. |Sí |
| fileName |Especifique el nombre del archivo en **folderPath** si quiere que la tabla haga referencia a un archivo específico de la carpeta. Si no especifica ningún valor para esta propiedad, la tabla apunta a todos los archivos de la carpeta.<br/><br/>Si no se especifica fileName para un conjunto de datos de salida, el nombre del archivo generado estaría en el siguiente formato: <br/><br/>`Data.<Guid>.txt` (Ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |No |
| fileFilter |Especifique el filtro que se va a usar para seleccionar un subconjunto de archivos de folderPath, en lugar de todos los archivos.<br/><br/>Valores permitidos son: `*` (varios caracteres) y `?` (un único individual).<br/><br/>Ejemplos 1: `"fileFilter": "*.log"`<br/>Ejemplo 2: `"fileFilter": 2014-1-?.txt"`<br/><br/> fileFilter es aplicable a un conjunto de datos FileShare de entrada. Esta propiedad no es compatible con HDFS. |No |
| partitionedBy |partitionedBy se puede usar para especificar un valor de folderPath dinámico, un nombre de archivo para datos de series temporales. Por ejemplo, folderPath se parametriza por cada hora de datos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Estos son los niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |
| useBinaryTransfer |Especifica si se usará el modo de transferencia binario. Su valor es true para el modo binario y false para ASCII. Valor predeterminado: verdadero. Esta propiedad solo puede usarse cuando el tipo de servicio vinculado asociado es: FtpServer. |No |

> [!NOTE]
> filename y fileFilter no pueden usarse simultáneamente.

### <a name="using-partionedby-property"></a>Uso de la propiedad partitionedBy
Como ya se ha indicado en la sección anterior, con partitionedBy se puede especificar un valor de folderPath dinámico, un nombre de archivo para datos de series temporales. Esto se puede hacer con las macros de Factoría de datos y las variables de sistema SliceStart y SliceEnd, que indican el periodo de tiempo lógico de un segmento de datos especificado.

Para aprender más sobre los conjuntos de datos, la programación y los segmentos de series temporales, consulte [Creación de conjuntos de datos](data-factory-create-datasets.md), [Programación y ejecución](data-factory-scheduling-and-execution.md) y [Creación de canalizaciones](data-factory-create-pipelines.md).

#### <a name="sample-1"></a>Muestra 1:

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
En este ejemplo, {Slice} se reemplaza por el valor de la variable del sistema SliceStart de Data Factory en el formato (YYYYMMDDHH) especificado. SliceStart hace referencia a la hora de inicio del segmento. folderPath es diferente para cada segmento. Ejemplo: wikidatagateway/wikisampledataout/2014100103 o wikidatagateway/wikisampledataout/2014100104.

#### <a name="sample-2"></a>Ejemplo 2:

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
Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo [Creación de canalizaciones](data-factory-create-pipelines.md). Las propiedades (como nombre, descripción, tablas de entrada y salida, y directivas) están disponibles para todos los tipos de actividades.

Por otra parte, las propiedades disponibles en la sección typeProperties de la actividad varían con cada tipo de actividad. Para la actividad de copia, las propiedades de tipo varían en función de los tipos de orígenes y receptores.

[!INCLUDE [data-factory-file-system-source](includes/data-factory-file-system-source.md)]

## <a name="supported-file-and-compression-formats"></a>Formatos de archivo y de compresión admitidos
Consulte los detalles en [Formatos de compresión y de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md).

## <a name="json-example-copy-data-from-sftp-server-to-azure-blob"></a>Ejemplo JSON: Copia de datos del servidor SFTP a un blob de Azure
En el siguiente ejemplo, se proporcionan definiciones JSON de ejemplo que puede usar para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Muestran cómo copiar datos del origen SFTP en Azure Blob Storage. Sin embargo, los datos se pueden copiar **directamente** de cualquiera de los orígenes a cualquiera de los receptores indicados [aquí](data-factory-data-movement-activities.md#supported-data-stores-and-formats) mediante la actividad de copia en Azure Data Factory.

> [!IMPORTANT]
> Este ejemplo proporciona fragmentos JSON. No incluye instrucciones paso a paso para crear la factoría de datos. Las instrucciones paso a paso se encuentran en el artículo sobre cómo [mover datos entre ubicaciones locales y en la nube](data-factory-move-data-between-onprem-and-cloud.md) .

El ejemplo consta de las siguientes entidades de factoría de datos:

* Un servicio vinculado de tipo [sftp](#linked-service-properties).
* Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [FileShare](#dataset-properties).
* Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [FileSystemSource](#copy-activity-properties) y [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

En el ejemplo se copian datos de un servidor SFTP en un blob de Azure cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

**Servicio vinculado de SFTP**

En este ejemplo se usa la autenticación básica con el nombre de usuario y la contraseña en texto sin formato. También se puede hacer de una de las siguientes maneras:

* Autenticación básica con credenciales cifradas
* Autenticación de la clave pública SSH

Para los diferentes tipos de autenticación que se pueden usar, consulte la sección [Propiedades del servicio vinculado de FTP](#linked-service-properties).

```JSON

{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "myuser",
            "password": "mypassword",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
    }
}
```
**Servicio vinculado de Azure Storage**

```JSON
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Conjunto de datos de entrada de SFTP**

Este conjunto de datos hace referencia a la carpeta `mysharedfolder` de SFTP y al archivo `test.csv`. La canalización copia el archivo en el destino.

Si se establece "external": "true", se informa al servicio Data Factory de que el conjunto de datos es externo a la factoría de datos y que no lo genera ninguna actividad de la factoría de datos.

```JSON
{
  "name": "SFTPFileInput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": "SftpLinkedService",
    "typeProperties": {
      "folderPath": "mysharedfolder",
      "fileName": "test.csv"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Conjunto de datos de salida de blob de Azure**

Los datos se escriben en un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta para el blob se evalúa dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa las partes year, month, day y hours de la hora de inicio.

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/sftp/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "HH"
                    }
                }
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Canalización con actividad de copia**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **FileSystemSource** y el tipo **sink** se establece en **BlobSink**.

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "SFTPToBlobCopy",
            "inputs": [{
                "name": "SFTPFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
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
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2017-02-20T18:00:00Z",
        "end": "2017-02-20T19:00:00Z"
    }
}
```

## <a name="performance-and-tuning"></a>Rendimiento y optimización
Consulte [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md) para más información sobre los factores clave que afectan al rendimiento del movimiento de datos (actividad de copia) en Azure Data Factory y las diversas formas de optimizarlo.

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:

* [Tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para la creación de una canalización con una actividad de copia.
