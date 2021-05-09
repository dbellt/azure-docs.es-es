---
title: Fórmula de instancia de Azure Container
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo implementar contenedores de Cognitive Services en una instancia de Azure Container
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, devx-track-azurecli
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 12/18/2020
ms.author: aahi
ms.openlocfilehash: d37b01750c1d5cd900378fcb5ea7f4d1ffb0d597
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287255"
---
# <a name="deploy-and-run-container-on-azure-container-instance"></a>Implementación y ejecución de un contenedor en Azure Container Instance

Con los pasos siguientes, escale aplicaciones de Azure Cognitive Services en la nube fácilmente con [Azure Container Instances](../../container-instances/index.yml). La creación de contenedores permite centrarse en la compilación de aplicaciones en lugar de en la administración de la infraestructura. Para más información sobre el uso de contenedores, consulte [Características y ventajas](../cognitive-services-container-support.md#features-and-benefits).

## <a name="prerequisites"></a>Requisitos previos

La fórmula funciona con cualquier contenedor de Cognitive Services. El recurso de Cognitive Services debe crearse antes de usar la fórmula. Todos los servicios de Cognitive Services que admiten contenedores tienen un artículo con el "Procedimiento de instalación" específico para instalar y configurar el servicio de un contenedor. Algunos servicios requieren un archivo o un conjunto de archivos como entrada para el contenedor, así que es importante que entienda y haya usado el contenedor correctamente antes de usar esta solución.

* Un recurso de Azure de la instancia de Azure Cognitive Services que está usando.
* **Dirección URL del punto de conexión** de Cognitive Services: revise el "Procedimiento de instalación" de su servicio específico para el contenedor. Así, encontrará el origen de la dirección URL del punto de conexión dentro de Azure Portal y el aspecto de un ejemplo correcto de la dirección URL. El formato exacto puede cambiar para cada servicio.
* **Clave** de Cognitive Services: las claves se encuentran en la página **Claves** del recurso de Azure. Solo necesita una de las dos claves. La clave es una cadena de 32 caracteres alfanuméricos.

* Un solo contenedor de Cognitive Services en su host local (su equipo). Asegúrese de que puede hacer lo siguiente:
  * Descargar la imagen con un comando `docker pull`.
  * Ejecutar el contenedor local correctamente con todos los valores de configuración necesarios mediante un comando `docker run`.
  * Llamar al punto de conexión del contenedor para obtener una respuesta de HTTP 2xx y una respuesta JSON.

Todas las variables en corchetes angulares, `<>`, deben reemplazarse por sus propios valores. Este reemplazo incluye los corchetes angulares.

> [!IMPORTANT]
> El contenedor de LUIS requiere un archivo de modelo `.gz` que se extrae en tiempo de ejecución. El contenedor debe poder acceder a este archivo de modelo a través del montaje de un volumen desde la instancia de contenedor. Para cargar un archivo de modelo, siga estos pasos:
> 1. [Cree un recurso compartido de archivos de Azure](../../storage/files/storage-how-to-create-file-share.md). Tome nota tanto del nombre de la cuenta de Azure Storage, como de su clave y del nombre del recurso compartido de archivos, ya que los necesitará más adelante.
> 2. [Exporte el modelo de LUIS (aplicación empaquetada) desde el portal de LUIS](../LUIS/luis-container-howto.md#export-packaged-app-from-luis). 
> 3. En Azure Portal, vaya a la página de **información general** del recurso de la cuenta de almacenamiento y seleccione **Recursos compartidos de archivos**. 
> 4. Seleccione el nombre del recurso compartido de archivos que ha creado recientemente y, después, seleccione **Cargar**. Después, cargue la aplicación empaquetada. 

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

[!INCLUDE [Portal instructions for creating an ACI instance](includes/create-container-instances-resource.md)]

# <a name="cli"></a>[CLI](#tab/cli)

[!INCLUDE [CLI instructions for creating an ACI instance](../containers/includes/create-container-instances-resource-from-azure-cli.md)]

---


## <a name="use-the-container-instance"></a>Uso de la instancia de contenedor

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. Seleccione la **Información general** y copie la dirección IP. Será una dirección IP numérica como `55.55.55.55`.
1. Abra una nueva pestaña del explorador y use la dirección IP; por ejemplo, `http://<IP-address>:5000 (http://55.55.55.55:5000`). Verá la página de inicio del contenedor, que le permite saber que el contenedor se está ejecutando.

    ![Página principal del contenedor](../../../includes/media/cognitive-services-containers-api-documentation/container-webpage.png)

1. Seleccione **Service API Description** (descripción de la API de servicio) para ver la página Swagger del contenedor.

1. Seleccione cualquiera de las API **POST** y seleccione **Probarlo**.  Se muestran los parámetros junto con la entrada. Rellene los parámetros.

1. Seleccione **Ejecutar** para enviar la solicitud a la instancia de contenedor.

    Ha creado y usado correctamente contenedores de Cognitive Services en Azure Container Instance.

# <a name="cli"></a>[CLI](#tab/cli)

[!INCLUDE [API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

> [!NOTE]
> Si va a ejecutar el contenedor de Text Analytics for Health, use la siguiente dirección URL para enviar consultas: `http://localhost:5000/text/analytics/v3.2-preview.1/entities/health`

---
