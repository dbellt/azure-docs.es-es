---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: 1b84292fb0dedb1facf30fcdf54ed2d569b3c054
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753573"
---
### <a name="azure-storage-linked-service"></a>Servicio vinculado de Azure Storage
El servicio vinculado de **Azure Storage** le permite vincular una cuenta de Azure Storage a una instancia de Azure Data Factory con una **clave de cuenta**, que proporciona a la instancia de Azure Data Factory acceso global a Azure Storage. En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio vinculado de Azure Storage.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type |La propiedad type debe establecerse en: **AzureStorage** |Sí |
| connectionString |Especifique la información necesaria para conectarse a Almacenamiento de Azure para la propiedad connectionString. |Sí |

Para más información sobre cómo recuperar las claves de acceso de las cuentas de almacenamiento, consulte [Administración de claves de acceso de cuentas de almacenamiento](../../../storage/common/storage-account-keys-manage.md).

**Ejemplo**:  

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

### <a name="azure-storage-sas-linked-service"></a>Servicio vinculado de SAS Azure Storage
Una Firma de acceso compartido (SAS) ofrece acceso delegado a los recursos en la cuenta de almacenamiento. Esto le permite conceder a un cliente permisos limitados a objetos en su cuenta de almacenamiento durante un período específico y con un conjunto determinado de permisos sin tener que compartir las claves de acceso a las cuentas. La SAS es un URI que incluye en sus parámetros de consulta toda la información necesaria para el acceso autenticado a un recurso de almacenamiento. Para obtener acceso a los recursos de almacenamiento con la SAS, el cliente solo tiene que pasar la SAS al método o constructor adecuados. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../../../storage/common/storage-sas-overview.md).

> [!IMPORTANT]
> Ahora Azure Data Factory solo admite **SAS de servicio**, pero no SAS de cuenta. Tenga en cuenta que la dirección URL de SAS generable desde Azure Portal o el Explorador de Storage es una SAS de cuenta, que no es compatible.

> [!TIP]
> Puede ejecutar a continuación los comandos de PowerShell para generar una SAS de servicio para la cuenta de almacenamiento (reemplace los marcadores de posición y conceda el permiso necesario): `$context = New-AzStorageContext -StorageAccountName <accountName> -StorageAccountKey <accountKey>`
> `New-AzStorageContainerSASToken -Name <containerName> -Context $context -Permission rwdl -StartTime <startTime> -ExpiryTime <endTime> -FullUri`

El servicio vinculado de SAS de Azure Storage le permite vincular una cuenta de Azure Storage a una factoría de datos Azure con una Firma de acceso compartido (SAS). Proporciona a la instancia de Data Factory acceso restringido o limitado por el tiempo a todos los recursos o a algunos específicos (blob o contenedor) del almacenamiento. En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio vinculado de SAS de Azure Storage. 

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type |La propiedad type debe establecerse en: **AzureStorageSas** |Sí |
| sasUri |Especifique el URI de Firma de acceso compartido a los recursos de Azure Storage como blob, contenedor o tabla.  |Sí |

**Ejemplo**:

```json
{
    "name": "StorageSasLinkedService",
    "properties": {
        "type": "AzureStorageSas",
        "typeProperties": {
            "sasUri": "<Specify SAS URI of the Azure Storage resource>"
        }
    }
}
```

Al crear un **URI de SAS**, tenga en cuenta lo siguiente:  

* Establezca los **permisos** de lectura y escritura adecuados en los objetos, en función de cómo se utilizará el servicio vinculado (lectura, escritura, lectura y escritura) en la factoría de datos.
* Establezca la **hora de expiración** adecuadamente. Asegúrese de que el acceso a los objetos de Azure Storage no expirará durante el período activo de la canalización.
* El URI debe crearse en el nivel correcto del contenedor/blob o la tabla, en función de la necesidad. Un URI de SAS de un blob de Azure permite que el servicio Data Factory tenga acceso a ese blob en particular. Un URI de SAS de un contenedor de blob de Azure permite que el servicio Data Factory procese una iteración en los blobs de ese contenedor. Si necesita proporcionar acceso a más o menos objetos más adelante, o actualizar el URI de SAS, no olvide actualizar el servicio vinculado con el nuevo URI.   

