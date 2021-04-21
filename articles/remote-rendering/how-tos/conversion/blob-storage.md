---
title: Uso de Azure Blob Storage para la conversión de modelos
description: Describe los pasos comunes para configurar y usar Blob Storage para la conversión de modelos.
author: jakrams
ms.author: jakras
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: 629b48a378dd029d9952270e3c71762a5bb7aa82
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107306728"
---
# <a name="use-azure-blob-storage-for-model-conversion"></a>Uso de Azure Blob Storage para la conversión de modelos

El servicio de [conversión de modelos](model-conversion.md) requiere acceso a Azure Blob Storage para poder recuperar datos de entrada y almacenar datos de salida. En este artículo se describen los pasos más comunes.

## <a name="prepare-azure-storage-accounts"></a>Preparación de cuentas de Azure Storage

- Creación de una cuenta de almacenamiento (StorageV2).
- Creación de un contenedor de blobs de entrada en la cuenta de almacenamiento (por ejemplo, con el nombre "arrinput")
- Creación de un contenedor de blobs de salida en la cuenta de almacenamiento (por ejemplo, con el nombre "arroutput")

> [!TIP]
> Para obtener instrucciones detalladas sobre cómo configurar la cuenta de almacenamiento, consulte [Inicio rápido: Conversión de un modelo para su representación](../../quickstarts/convert-model.md)

La creación de la cuenta de almacenamiento y los contenedores de blobs se pueden realizar con una de las siguientes herramientas:

- [Azure Portal](https://portal.azure.com)
- [Línea de comandos de az](/cli/azure/install-azure-cli)
- [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/)
- SDK (C#, Python...)

## <a name="ensure-azure-remote-rendering-can-access-your-storage-account"></a>Comprobación de que Azure Remote Rendering puede acceder a la cuenta de almacenamiento

Azure Remote Rending necesita recuperar los datos del modelo de la cuenta de almacenamiento y volver a escribir datos en este.

Puede conceder a Azure Remote Rendering acceso a la cuenta de almacenamiento de las dos maneras siguientes:

### <a name="connect-your-azure-storage-account-with-your-azure-remote-rendering-account"></a>Conexión de la cuenta de Azure Storage con la cuenta de Azure Remote Rendering

Siga los pasos indicados en la sección [Creación de una cuenta](../create-an-account.md#link-storage-accounts).

### <a name="retrieve-sas-for-the-storage-containers"></a>Recuperación de SAS para los contenedores de almacenamiento

Se usan firmas de acceso almacenadas (SAS) para conceder acceso de lectura para la entrada y acceso de escritura para la salida. Se recomienda generar nuevos URI cada vez que se convierte un modelo. Dado que los URI expiran después de un tiempo, si se persisten durante más tiempo, la aplicación podría interrumpirse de forma inesperada.

Encontrará más información sobre SAS en la [documentación sobre SAS](../../../storage/common/storage-sas-overview.md).

Un URI de SAS se puede generar con una de las siguientes opciones:

- Módulo az de PowerShell
  - Consulte los [scripts de PowerShell de ejemplo](../../samples/powershell-example-scripts.md)
- [Línea de comandos de az](/cli/azure/install-azure-cli)
- [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/)
  - Haga clic con el botón derecho en el contenedor "Obtener firma de acceso compartido" (lectura, acceso de lista al contenedor de entrada, acceso de escritura al contenedor de salida)
- SDK (C#, Python...)

Un ejemplo del uso de firmas de acceso compartido en la conversión de recursos se muestra en Conversion.ps1, en los [scripts de ejemplo de PowerShell](../../samples/powershell-example-scripts.md#script-conversionps1).

## <a name="upload-an-input-model"></a>Cargar un modelo de entrada

Para empezar a convertir un modelo, debe cargarlo mediante una de las siguientes opciones:

- [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/): una cómoda interfaz de usuario para cargar, descargar y administrar archivos en Azure Blob Storage
- [Línea de comandos de Azure](../../../storage/blobs/storage-quickstart-blobs-cli.md)
- [Módulo de Azure PowerShell](/powershell/azure/install-az-ps)
  - Consulte los [scripts de PowerShell de ejemplo](../../samples/powershell-example-scripts.md)
- [Uso de un SDK de Storage (Python, C#...)](../../../storage/index.yml)
- [Uso de API REST de Azure Storage](/rest/api/storageservices/blob-service-rest-api)

Para obtener un ejemplo de cómo cargar los datos para la conversión, consulte Conversion.ps1 en los [scripts de ejemplo de PowerShell](../../samples/powershell-example-scripts.md#script-conversionps1).

> [!Note]
> Al cargar un modelo de entrada, no use nombres de archivo o estructuras de carpetas largos para evitar problemas con el [límite de longitud de ruta de acceso de Windows](https://docs.mxicrosoft.com/windows/win32/fileio/maximum-file-path-limitation) en el servicio. 

## <a name="get-a-sas-uri-for-the-converted-model"></a>Obtener un URI de SAS para el modelo convertido

Este paso es similar a [recuperar SAS para los contenedores de almacenamiento](#retrieve-sas-for-the-storage-containers). Sin embargo, esta vez necesita recuperar un URI de SAS para el archivo de modelo que se escribió en el contenedor de salida.

Por ejemplo, para recuperar un URI de SAS desde el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/), haga clic con el botón derecho en el archivo del modelo y seleccione "Obtener firma de acceso compartido".

Se necesita una Firma de acceso compartido (SAS) para cargar modelos si no ha conectado la cuenta de almacenamiento con la de Azure Remote Rendering. Puede obtener información sobre cómo conectar la cuenta en [Crear una cuenta](../create-an-account.md#link-storage-accounts).

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la conversión de modelos](configure-model-conversion.md)
- [API de REST de conversión de modelos](conversion-rest-api.md)
