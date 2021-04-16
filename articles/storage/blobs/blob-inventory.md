---
title: Uso del inventario de Azure Storage para administrar datos de blobs (versión preliminar)
description: El inventario de Azure Storage es una herramienta que ayuda a obtener información general de todos los datos de blobs dentro de una cuenta de almacenamiento.
services: storage
author: twooley
ms.service: storage
ms.date: 04/01/2021
ms.topic: conceptual
ms.author: twooley
ms.reviewer: klaasl
ms.subservice: blobs
ms.custom: references_regions
ms.openlocfilehash: 33d50d1a6b5e84d178b522851795bcc42f5fc169
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106277273"
---
# <a name="use-azure-storage-blob-inventory-to-manage-blob-data-preview"></a>Uso del inventario de blobs de Azure Storage para administrar datos de blobs (versión preliminar)

La característica de inventario de blobs de Azure Storage proporciona información general de todos los datos de blobs dentro de una cuenta de almacenamiento. Use el informe de inventario para comprender el tamaño total de los datos, la antigüedad, el estado de cifrado, etc. El informe proporciona información general de los datos para los requisitos empresariales y de cumplimiento. Una vez habilitado, se crea automáticamente un informe de inventario a diario.

## <a name="availability"></a>Disponibilidad

El inventario de blobs es compatible con las cuentas de almacenamiento de blob en bloques prémium y de uso general de la versión 2 (GPv2). Esta característica es compatible con la característica [espacio de nombres jerárquico](data-lake-storage-namespace.md) habilitada o sin esta.

> [!IMPORTANT]
> Inventario de blobs está actualmente en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

### <a name="preview-regions"></a>Regiones en versión preliminar

La versión preliminar del inventario de blobs está disponible en las cuentas de almacenamiento de las siguientes regiones:

- Centro de Francia
- Centro de Canadá
- Este de Canadá
- Este de EE. UU.
- Este de EE. UU. 2
- Oeste de Europa

### <a name="pricing-and-billing"></a>Precios y facturación

La tarifa de los informes de inventario no se cobra durante el período de versión preliminar. El precio se determinará cuando esta característica esté disponible con carácter general.

## <a name="enable-inventory-reports"></a>Habilitación de informes de inventario

Habilite los informes de inventario de blobs agregando una directiva a su cuenta de almacenamiento. Agregue, edite o quite una directiva mediante [Azure Portal](https://portal.azure.com/).

1. Vaya a [Azure Portal](https://portal.azure.com/).
1. Seleccione una de las cuentas de almacenamiento.
1. En **Blob service**, seleccione **Blobs**.
1. Asegúrese de que la opción **Blob inventory enabled** (Inventario de blobs habilitado) está seleccionada.
1. Seleccione **Agregar una regla**.
1. Asigne un nombre a la nueva regla.
1. Seleccione una opción en **Blob types** (Tipos de blobs) para el informe de inventario.
1. Agregue una coincidencia de prefijo para filtrar el informe de inventario.
1. Seleccione **Include blob versions** (Incluir versiones de blob) y **Include snapshots** (Incluir instantáneas) en función de lo que desee incluir en el informe de inventario. Las versiones y las instantáneas deben estar habilitadas en la cuenta para guardar una nueva regla con la opción correspondiente habilitada.
1. Seleccione **Guardar**.

:::image type="content" source="./media/blob-inventory/portal-blob-inventory.png" alt-text="Captura de pantalla que muestra cómo agregar una regla de inventario de blobs mediante Azure Portal":::

Las directivas de inventario se leen o escriben en su totalidad. No se admiten las actualizaciones parciales.

> [!IMPORTANT]
> Si habilita reglas de firewall para la cuenta de almacenamiento, puede que se bloqueen las solicitudes de inventario. Puede desbloquear estas solicitudes proporcionando excepciones para los servicios de confianza de Microsoft. Para más información, consulte la sección Excepciones en [Configuración de firewalls y redes virtuales](../common/storage-network-security.md#exceptions).

Cada día se programa automáticamente una ejecución de inventario de blobs. Puede tardar hasta 24 horas en completarse una ejecución de inventario. Un informe de inventario se configura mediante la adición de una directiva de inventario con una o más reglas.

## <a name="inventory-policy"></a>Directiva de inventario

Una directiva de inventario es una colección de reglas en un documento JSON.

```json
{
    "destination": "destinationContainer",
    "enabled": true,
    "rules": [
    {
        "enabled": true,
        "name": "inventoryrule1",
        "definition": {. . .}
    },
    {
        "enabled": true,
        "name": "inventoryrule2",
        "definition": {. . .}
    }]
}
```

Para ver el documento JSON para una directiva de inventario, seleccione la pestaña **Vista Código** en la sección **Blob inventory** (Inventario de blobs) de Azure Portal.

| Nombre de parámetro | Tipo de parámetro        | Notas | ¿Necesario? |
|----------------|-----------------------|-------|-----------|
| destination    | String                | Contenedor de destino en el que se generarán todos los archivos de inventario. El contenedor de destino ya debe existir. | Sí |
| enabled        | Boolean               | Se usa para deshabilitar toda la directiva. Cuando se establece en **true**, el campo habilitado de nivel de regla invalida este parámetro. Cuando se deshabilita, se deshabilitará el inventario para todas las reglas. | Sí |
| reglas          | Matriz de objetos de regla | Se requiere al menos una regla en una directiva. Se admiten hasta 10 reglas. | Sí |

## <a name="inventory-rules"></a>Reglas de inventario

Una regla captura las condiciones de filtrado y los parámetros de salida para generar un informe de inventario. Cada regla crea un informe de inventario. Las reglas pueden tener prefijos superpuestos. Un blob puede aparecer en más de un inventario en función de las definiciones de la regla.

Cada regla de la directiva tiene varios parámetros:

| Nombre de parámetro | Tipo de parámetro                 | Notas | ¿Necesario? |
|----------------|--------------------------------|-------|-----------|
| name           | String                         | Un nombre de regla puede incluir hasta 256 caracteres alfanuméricos que distinguen mayúsculas de minúsculas. El nombre debe ser único dentro de un proyecto. | Sí |
| enabled        | Boolean                        | Marca que permite habilitar o deshabilitar una regla. El valor predeterminado es **true**. | Sí |
| definición     | Definición de regla de inventario JSON | Cada definición se compone de un conjunto de filtros de regla. | Sí |

La marca global **Blob inventory enabled** (Inventario de blobs habilitado) tiene prioridad sobre el parámetro *enabled* en una regla.

### <a name="rule-filters"></a>Filtros de reglas

Hay varios filtros disponibles para personalizar un informe de inventario de blobs:

| Nombre de filtro         | Tipo de filtro                     | Notas | ¿Necesario? |
|---------------------|---------------------------------|-------|-----------|
| blobTypes           | Matriz de valores de enumeración predefinidos | Los valores válidos son `blockBlob` y `appendBlob` para cuentas habilitadas para el espacio de nombres jerárquico, y `blockBlob`, `appendBlob` y `pageBlob` para otras cuentas. | Sí |
| prefixMatch         | Matriz de hasta 10 cadenas de prefijos con los que debe hacer coincidencias. Un prefijo debe comenzar con un nombre de contenedor, por ejemplo, "container1/foo" | Si no define *prefixMatch* o proporciona un prefijo vacío, la regla se aplica a todos los blobs de la cuenta de almacenamiento. | No |
| includeSnapshots    | Boolean                         | Especifica si el inventario debe incluir instantáneas. El valor predeterminado es **false**. | No |
| includeBlobVersions | Boolean                         | Especifica si el inventario debe incluir versiones de blob. El valor predeterminado es **false**. | No |

Para ver el documento JSON para reglas de inventario, seleccione la pestaña **Vista Código** en la sección **Blob inventory** (Inventario de blobs) de Azure Portal. Los filtros se especifican dentro de una definición de regla.

```json
{
    "destination": "destinationContainer",
    "enabled": true,
    "rules": [
    {
        "enabled": true,
        "name": "inventoryrule1",
        "definition":
        {
            "filters":
            {
                "blobTypes": ["blockBlob", "appendBlob", "pageBlob"],
                "prefixMatch": ["inventorycontainer1", "inventorycontainer2/abcd", "etc"]
            }
        }
    },
    {
        "enabled": true,
        "name": "inventoryrule2",
        "definition":
        {
            "filters":
            {
                "blobTypes": ["pageBlob"],
                "prefixMatch": ["inventorycontainer-disks-", "inventorycontainer4/"],
                "includeSnapshots": true,
                "includeBlobVersions": true
            }
        }
    }]
}
```

## <a name="inventory-output"></a>Salida de inventario

Cada ejecución de inventario genera un conjunto de archivos con formato CSV en el contenedor de destino de inventario especificado. La salida de inventario se genera en la siguiente ruta de acceso: `https://<accountName>.blob.core.windows.net/<inventory-destination-container>/YYYY/MM/DD/HH-MM-SS/` donde:

- *accountName* es el nombre de la cuenta de Azure Blob Storage
- *inventory-destination-container* es el contenedor de destino que especificó en la directiva de inventario
- *YYYY/MM/DD/HH-MM-SS* es el momento en el que el inventario comenzó a ejecutarse

### <a name="inventory-files"></a>Archivos de inventario

Cada ejecución de inventario genera los siguientes archivos:

- **Archivo CSV de inventario**: un archivo de valores separados por comas (CSV) para cada regla de inventario. Cada archivo contiene objetos coincidentes y sus metadatos. La primera fila de cada archivo con formato CSV siempre es la fila del esquema. En la siguiente imagen se muestra un archivo CSV de inventario abierto en Microsoft Excel.

   :::image type="content" source="./media/blob-inventory/csv-file-excel.png" alt-text="Captura de pantalla de un archivo CSV de inventario abierto en Microsoft Excel":::

- **Archivo de manifiesto**: un archivo manifest.json que contiene los detalles de los archivos de inventario generados para cada regla en esa ejecución. El archivo de manifiesto también captura la definición de la regla proporcionada por el usuario y la ruta de acceso al inventario para esa regla.

- **Archivo de suma de comprobación**: archivo manifest.checksum que contiene la suma de comprobación MD5 del contenido del archivo manifest.json. La generación del archivo manifest.checksum marca la finalización de una ejecución de inventario.

## <a name="inventory-completed-event"></a>Evento de inventario completado

Suscríbase al evento de inventario completado para recibir una notificación cuando la ejecución de inventario se complete. Este evento se genera cuando se crea el archivo de suma de comprobación del manifiesto. El evento de inventario completado también tiene lugar si se produce un error de usuario en la ejecución de inventario antes de que empiece a ejecutarse. Por ejemplo, un error de directiva no válida o contenedor de destino no presente desencadenará el evento. El evento se publica en el tema de inventario de blobs.

Evento de ejemplo:

```json
{
  "topic": "/subscriptions/3000151d-7a84-4120-b71c-336feab0b0f0/resourceGroups/BlobInventory/providers/Microsoft.EventGrid/topics/BlobInventoryTopic",
  "subject": "BlobDataManagement/BlobInventory",
  "eventType": "Microsoft.Storage.BlobInventoryPolicyCompleted",
  "id": "c99f7962-ef9d-403e-9522-dbe7443667fe",
  "data": {
    "scheduleDateTime": "2020-10-13T15:37:33Z",
    "accountName": "inventoryaccountname",
    "policyRunStatus": "Succeeded",
    "policyRunStatusMessage": "Inventory run succeeded, refer manifest file for inventory details.",
    "policyRunId": "b5e1d4cc-ee23-4ed5-b039-897376a84f79",
    "manifestBlobUrl": "https://inventoryaccountname.blob.core.windows.net/inventory-destination-container/2020/10/13/15-37-33/manifest.json"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-10-13T15:47:54Z"
}
```

## <a name="known-issues"></a>Problemas conocidos

En esta sección se describen las limitaciones y los problemas conocidos de la característica de inventario de blobs de Azure Storage.

### <a name="inventory-job-fails-to-complete"></a>No se puede completar el trabajo de inventario

Es posible que el trabajo de inventario no pueda completarse en 24 horas para una cuenta con millones de blobs y espacios de nombres jerárquicos habilitados. Si esto ocurre, no se creará ningún archivo de inventario.

## <a name="next-steps"></a>Pasos siguientes

- [Cálculo del recuento y el tamaño total de los blobs por contenedor](calculate-blob-count-size.md)
- [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md)
