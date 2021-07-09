---
title: 'Inicio rápido: Biblioteca cliente de Administración de Azure para Python'
description: En este inicio rápido, comience con la biblioteca cliente de Administración de Azure para Python.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 06/04/2021
ms.author: pafarley
ms.openlocfilehash: d98b71680e8be744b52f5060cc9be38366963fd6
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111594368"
---
[Documentación de referencia](/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices) | [Paquete (PyPi)](https://pypi.org/project/azure-mgmt-cognitiveservices/) | [Ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices/tests)

## <a name="python-prerequisites"></a>Requisitos previos de Python

* Una suscripción a Azure válida: [cree una de manera gratuita](https://azure.microsoft.com/free/).
* [Python 3.x](https://www.python.org/)
* [!INCLUDE [terms-azure-portal](./terms-azure-portal.md)]

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree una aplicación de Python en el editor o IDE que prefiera y vaya al proyecto en una ventana de consola.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Puede instalar la biblioteca cliente con lo siguiente:

```console
pip install azure-mgmt-cognitiveservices
```

Si usa el IDE de Visual Studio, la biblioteca cliente estará disponible como un paquete de NuGet descargable.

### <a name="import-libraries"></a>Importación de bibliotecas

Abra el script de Python e importe las siguientes bibliotecas.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Agregue los campos siguientes a la raíz del script y rellene sus valores, para lo que debe usar la entidad de servicio que creó y la información de su cuenta de Azure.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_constants)]

Luego, agregue el siguiente código para construir un objeto **CognitiveServicesManagementClient**. Este objeto es necesario para todas las operaciones de Administración de Azure.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_auth)]

## <a name="create-a-cognitive-services-resource-python"></a>Creación de un recurso de Cognitive Services (Python)

Para crear un recurso de Cognitive Services y suscribirse a él, use la función **Create**. Esta función agrega un nuevo recurso facturable al grupo de recursos que se pasa. Cuando cree el recurso, deberá conocer el "tipo" de servicio que desea usar, el plan de tarifa (o SKU) y la ubicación de Azure. La siguiente función usa todos estos valores como argumentos y crea un recurso.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_create)]

### <a name="choose-a-service-and-pricing-tier"></a>Elección de un servicio y un plan de tarifa

Al crear un recurso, necesitará conocer el "tipo" de servicio que desea usar y el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/) (o la SKU) que desee. Usará esta y otra información como parámetros al crear el recurso. La siguiente función enumera los "tipos" de Cognitive Services disponibles.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="view-your-resources"></a>Visualización de los recursos

Para ver todos los recursos de su cuenta de Azure (de todos los grupos de recursos), use la siguiente función:

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_list)]

## <a name="delete-a-resource"></a>Eliminar un recurso

La función siguiente elimina el recurso especificado del grupo de recursos dado.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_delete)]

Si necesita recuperar un recurso eliminado, consulte el artículo sobre la [recuperación de recursos de Cognitive Services eliminados](../../manage-resources.md).

## <a name="call-management-functions"></a>Funciones de administración de llamadas

Agregue el siguiente código al final del script para llamar a las funciones anteriores. En este código se muestran los recursos disponibles, se crea un recurso de ejemplo, se enumeran los recursos que posee y, después, se elimina el recurso de ejemplo.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_calls)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde la línea de comandos con el comando `python`.

```console
python <your-script-name>.py
```

## <a name="see-also"></a>Consulte también

* Consulte **[Autenticación de solicitudes en Azure Cognitive Services](../../authentication.md)** para aprender a trabajar de forma segura con Cognitive Services.
* Consulte **[¿Qué es Azure Cognitive Services?](../../what-are-cognitive-services.md)** para obtener una lista de las distintas categorías de Cognitive Services.
* Consulte **[Compatibilidad con idiomas naturales](../../language-support.md)** para ver la lista de los idiomas naturales admitidos por Cognitive Services.
* Consulte **[Contenedores de Azure Cognitive Services](../../cognitive-services-container-support.md)** para aprender a usar Cognitive Services en un entorno local.
* Consulte **[Planeamiento y administración de los costos de Azure Cognitive Services](../../plan-manage-costs.md)** para estimar el costo de usar Cognitive Services.
* Consulte la **[documentación de referencia del SDK de administración de Azure](/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices)** para más detalles sobre el SDK de administración.
