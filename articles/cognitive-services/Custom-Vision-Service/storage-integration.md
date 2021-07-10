---
title: Integración de Azure Storage con las notificaciones y la copia de seguridad del modelo
titleSuffix: Azure Cognitive Services
description: Aprenda a integrar Azure Storage para recibir notificaciones de envío al entrenar o exportar modelos de Custom Vision. También puede guardar una copia de seguridad de los modelos exportados.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: how-to
ms.date: 09/11/2020
ms.author: pafarley
ms.openlocfilehash: 92a9b79e4de60e7308224b435953ac7b3eefc8a1
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111951074"
---
# <a name="integrate-azure-storage-for-notifications-and-backup"></a>Integración de Azure Storage con las notificaciones y la copia de seguridad

Puede integrar el proyecto de Custom Vision con una cola de almacenamiento de blobs de Azure para obtener notificaciones de envío de actividad de entrenamiento/exportación del proyecto y copias de seguridad de los modelos publicados. Esta característica resulta útil para evitar el sondeo continuo del servicio en busca de resultados cuando se ejecutan operaciones largas. En su lugar, puede integrar las notificaciones de la cola de almacenamiento en el flujo de trabajo.

En esta guía se muestra cómo usar las API de REST con cURL. También puede usar un servicio de solicitud HTTP como Postman para emitir las solicitudes.

> [!NOTE]
> Las notificaciones de inserción dependen del parámetro opcional _notificationQueueUri_ de la API **CreateProject** y las copias de seguridad del modelo requieren que también se use el parámetro opcional _exportModelContainerUri_. En esta guía se utilizarán ambos para el conjunto completo de características.

## <a name="prerequisites"></a>Requisitos previos

- Un recurso de Custom Vision en Azure. Si no tiene uno, vaya a Azure Portal y [cree un recurso de Custom Vision](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision?azure-portal=true). Esta característica no admite actualmente el recurso de Cognitive Services (clave todo en uno).
- Una cuenta de Azure Storage con un contenedor de blobs. Si necesita ayuda con este paso, consulte el [inicio rápido de Storage](../../storage/blobs/storage-quickstart-blobs-portal.md).
- [PowerShell versión 6.0+](/powershell/scripting/install/installing-powershell-core-on-windows) o una aplicación de la línea de comandos similar.

## <a name="set-up-azure-storage-integration"></a>Configuración de la integración de Azure Storage

Vaya al recurso de entrenamiento de Custom Vision en Azure Portal, seleccione la página **Identidad** y habilite la identidad administrada asignada por el sistema.

A continuación, vaya al recurso de almacenamiento en Azure Portal. Vaya a la página **Control de acceso (IAM)** y agregue una asignación de roles para cada característica de integración:
* Seleccione el recurso de entrenamiento de Custom Vision y asigne el rol **Colaborador de datos de Storage Blob** si tiene previsto usar la característica de copia de seguridad del modelo. 
* Después, seleccione el recurso de entrenamiento de Custom Vision y asigne el rol **Colaborador de datos de la cola de Storage Blob** si tiene previsto usar la característica de cola de notificaciones.

> [!div class="mx-imgBorder"]
> ![Página Agregar asignación de roles de la cuenta de Storage](./media/storage-integration/storage-access.png)

### <a name="get-integration-urls"></a>Obtener direcciones URL de integración

A continuación, obtendrá las direcciones URL que permiten que el recurso de Custom Vision tenga acceso a estos puntos de conexión.

Para la dirección URL de integración de la cola de notificaciones, vaya a la página **Colas** de la cuenta de almacenamiento, agregue una nueva cola y guarde su dirección URL en una ubicación temporal.

> [!div class="mx-imgBorder"]
> ![Página Cola de Azure Storage](./media/storage-integration/queue-url.png) 

Para la dirección URL de integración de copia de seguridad del modelo, vaya a la página **Contenedores** de la cuenta de almacenamiento y cree un contenedor. A continuación, selecciónelo y vaya a la página **Propiedades**. Copie la dirección URL en una ubicación temporal.
 
> [!div class="mx-imgBorder"]
> ![Página Propiedades del contenedor de Azure Storage](./media/storage-integration/container-url.png) 


## <a name="integrate-custom-vision-project"></a>Integración del proyecto de Custom Vision

Ahora que tiene las direcciones URL de integración, puede crear un proyecto de Custom Vision que integre las características de Azure Storage. También puede actualizar un proyecto existente para agregar las características.

### <a name="create-new-project"></a>Creación de un proyecto

Cuando llame a la API [CreateProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeae), agregue los parámetros opcionales _exportModelContainerUri_ y _notificationQueueUri_. Asigne los valores de dirección URL que obtuvo en la sección anterior. 

```curl
curl -v -X POST "{endpoint}/customvision/v3.3/Training/projects?exportModelContainerUri={inputUri}&notificationQueueUri={inputUri}&name={inputName}"
-H "Training-key: {subscription key}"
```

Si recibe una respuesta `200/OK`, significa que las direcciones URL se han configurado correctamente. También debería ver los valores de la dirección URL en la respuesta JSON:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "useNegativeSet": true,
    "detectionParameters": "string",
    "imageProcessingSettings": {
      "augmentationMethods": {}
},
"exportModelContainerUri": {url}
"notificationQueueUri": {url}
  },
  "created": "string",
  "lastModified": "string",
  "thumbnailUri": "string",
  "drModeEnabled": true,
  "status": "Succeeded"
}
```

### <a name="update-existing-project"></a>Actualización de un proyecto existente

Para actualizar un proyecto existente con la integración de características de Azure Storage, llame a la API [UpdateProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb1) con el identificador del proyecto que quiere actualizar. 

```curl
curl -v -X PATCH "{endpoint}/customvision/v3.3/Training/projects/{projectId}"
-H "Content-Type: application/json"
-H "Training-key: {subscription key}"

--data-ascii "{body}" 
```

Establezca el cuerpo de la solicitud (`body`) con el siguiente formato JSON y rellene los valores adecuados para _exportModelContainerUri_ y _notificationQueueUri_:

```json
{
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "imageProcessingSettings": {
      "augmentationMethods": {}
},
"exportModelContainerUri": {url}
"notificationQueueUri": {url}

  },
  "status": "Succeeded"
}
```

Si recibe una respuesta `200/OK`, significa que las direcciones URL se han configurado correctamente.

## <a name="verify-the-connection"></a>Comprobación de la conexión 

La llamada de API de la sección anterior debe haber desencadenado ya nueva información en su cuenta de Azure Storage. 

En el contenedor designado, debe haber un blob de prueba dentro de una carpeta **CustomVision-TestPermission**. Este blob solo existirá temporalmente.

En la cola de notificaciones, debería ver una notificación de prueba en el siguiente formato:
 
```json
{
"version": "1.0" ,
"type": "ConnectionTest",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000"
    }
}
```

## <a name="get-event-notifications"></a>Obtención de notificaciones de eventos

Cuando esté listo, llame a la API [TrainProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee1) en el proyecto para realizar una operación de entrenamiento normal.

En la cola de notificaciones de Storage, recibirá una notificación una vez que finalice el entrenamiento:

```json
{
"version": "1.0" ,
"type": "Training",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "iterationId": "00000000-0000-0000-0000-000000000000",
    "trainingStatus": "TrainingCompleted"
    }
}
```

El campo `"trainingStatus"` puede ser `"TrainingCompleted"` o `"TrainingFailed"`. El campo `"iterationId"` es el identificador del modelo entrenado.

## <a name="get-model-export-backups"></a>Obtención de copias de seguridad de exportación de modelos

Cuando esté listo, llame a la API [ExportIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddece) para exportar un modelo entrenado a una plataforma específica.

En el contenedor de almacenamiento designado, aparecerá una copia de seguridad del modelo exportado. El nombre del blob tendrá el formato:

```
{projectId} - {iterationId}.{platformType}
```

Además, recibirá una notificación en la cola cuando finalice la exportación. 

```json
{
"version": "1.0" ,
"type": "Export",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "iterationId": "00000000-0000-0000-0000-000000000000",
    "exportStatus": "ExportCompleted",
    "modelUri": {url}
    }
}
```

El campo `"exportStatus"` puede ser `"ExportCompleted"` o `"ExportFailed"`. El campo `"modelUri"` contendrá la dirección URL del modelo de copia de seguridad almacenado en el contenedor, en el supuesto de que haya integrado las notificaciones de cola al principio. Si no lo ha hecho, el campo `"modelUri"` mostrará la dirección URL de SAS para el blob del modelo de Custom Vision.

## <a name="next-steps"></a>Pasos siguientes

En esta guía, aprendió a copiar y trasladar un proyecto entre recursos de Custom Vision. A continuación, consulte los documentos de referencia de la API para ver qué más puede hacer con Custom Vision.
* [Documentación de referencia de la API de REST (entrenamiento)](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)
* [Documentación de referencia de la API de REST (predicción)](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.1/operations/5eb37d24548b571998fde5f3)