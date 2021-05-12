---
title: Conservación de los metadatos y las ACL mediante la actividad de copia en Azure Data Factory
description: Obtenga información sobre cómo conservar los metadatos y las ACL durante la copia mediante la actividad de copia en Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: jianleishen
ms.openlocfilehash: 15221c5884ca52d5560fe46dc36ba1647bdea394
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488598"
---
#  <a name="preserve-metadata-and-acls-using-copy-activity-in-azure-data-factory"></a>Conservación de los metadatos y las ACL mediante la actividad de copia en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cuando se usa la actividad de copia de Azure Data Factory para copiar datos desde el origen al receptor en los escenarios siguientes, también se pueden conservar los metadatos y las ACL.

## <a name="preserve-metadata-for-lake-migration"></a><a name="preserve-metadata"></a> Conservación de los metadatos para la migración de lagos

Al migrar datos de un lago de datos a otro, incluido [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) y [Azure File Storage](connector-azure-file-storage.md), puede optar por conservar los metadatos de archivo junto con los datos.

La actividad de copia admite la conservación de los siguientes atributos durante la copia de datos:

- **Todos los metadatos especificados por el cliente** 
- Y las siguientes **cinco propiedades del sistema integradas del almacén de datos**: `contentType`, `contentLanguage` (excepto Amazon S3), `contentEncoding``contentDisposition``cacheControl`.

**Control de las diferencias en los metadatos:** Amazon S3 y Azure Storage permiten diferentes conjuntos de caracteres en las claves de metadatos especificadas por el cliente. Cuando elige conservar los metadatos mediante una actividad de copia, ADF reemplaza automáticamente los caracteres no válidos por "_".

Al copiar los archivos tal cual desde Amazon S3/Azure Data Lake Storage Gen2/Azure Blob/Azure File Storage a Azure Data Lake Storage Gen2/Azure Blob/Azure File Storage con el formato binario, puede encontrar la opción **Conservar** en la pestaña **Actividad de copia** > **Configuración** para la creación de la actividad o en la página **Configuración** en la herramienta Copiar datos.

![Metadatos de conservación de la actividad de copia](./media/copy-activity-preserve-metadata/copy-activity-preserve-metadata.png)

Este es un ejemplo de configuración de JSON de la actividad de copia (consulte `preserve`): 

```json
"activities":[
    {
        "name": "CopyAndPreserveMetadata",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AmazonS3ReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "Attributes"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset Amazon S3/Azure Blob/ADLS Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset for Azure Blob/ADLS Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="preserve-acls-from-data-lake-storage-gen1gen2-to-gen2"></a><a name="preserve-acls"></a> Conservación de las ACL desde Data Lake Storage Gen1/Gen2 a Gen2

Al actualizar de Azure Data Lake Storage Gen1 a Gen2 o copiar datos entre ADLS Gen2, se puede optar por conservar las listas de control de acceso (ACL) de POSIX junto con los archivos de datos. Para más información sobre el control de acceso, consulte [Control de acceso en Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md) y [Control de acceso en Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-access-control.md).

La actividad de copia admite la conservación de los siguientes tipos de ACL durante la copia de datos. Puede seleccionar uno o varios tipos:

- **ACL**: Copie y conserve las listas de control de acceso de POSIX en archivos y directorios. Esto copia las ACL existentes completas del origen al destino. 
- **Propietario**: Copie y conserve el usuario propietario de archivos y directorios. Se requiere acceso de superusuario en Data Lake Storage Gen2 de destino.
- **Grupo**: Copie y conserve el grupo propietario de archivos y directorios. Se requiere acceso de superusuario para Data Lake Storage Gen2 de destino o el usuario propietario (si el usuario propietario también es miembro del grupo de destino).

Si especifica que desea copiar desde una carpeta, Data Factory replicará las ACL de esa carpeta concreta y los archivos y directorios incluidos en ella si `recursive` se establece en true. Si especifica que desea copiar desde un único archivo, se copiarán las ACL de ese archivo.

>[!NOTE]
>Cuando use ADF para conservar las ACL de Data Lake Storage Gen1/Gen2 a Gen2, se sobrescribirán las ACL existentes en la carpeta o los archivos correspondientes de Gen2 del receptor.

>[!IMPORTANT]
>Si decide conservar las ACL, asegúrese de que concede permisos lo suficientemente altos para que Data Factory funcione en la cuenta de destino de Data Lake Storage Gen2. Por ejemplo, utilice la autenticación de clave de cuenta o asigne el rol Propietario de datos de Blob Storage a la entidad de servicio o identidad administrada.

Si configura el origen como Data Lake Storage Gen1/Gen2 con la opción de formato binario o de copia binaria, y el receptor como Data Lake Storage Gen2 con las mismas opciones, puede encontrar la opción **Conservar** en la página **Configuración** de la herramienta Copiar datos o en la pestaña **Copiar actividad** > **Configuración** para la creación de actividades.

![Opción Conservar ACL de Data Lake Storage Gen1/Gen2 a Gen2](./media/connector-azure-data-lake-storage/adls-gen2-preserve-acl.png)

Este es un ejemplo de configuración de JSON de la actividad de copia (consulte `preserve`): 

```json
"activities":[
    {
        "name": "CopyAndPreserveACLs",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AzureDataLakeStoreReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "ACL",
                "Owner",
                "Group"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen1/Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los otros artículos de la actividad de copia:

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Rendimiento de la actividad de copia](copy-activity-performance.md)
