---
title: 'Inicio rápido: Biblioteca cliente de Administración de Azure para .NET'
description: En este inicio rápido, comience con la biblioteca cliente de Administración de Azure para .NET.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 06/04/2021
ms.author: pafarley
ms.openlocfilehash: ce14ff28c69b514e0d10ef5eabff83828026bb19
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593787"
---
[Documentación de referencia](/dotnet/api/overview/azure/cognitiveservices/management) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Microsoft.Azure.Management.CognitiveServices) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.Management.CognitiveServices/) | [Ejemplos](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Microsoft.Azure.Management.CognitiveServices/tests)

## <a name="c-prerequisites"></a>Requisitos previos de C#

* Una suscripción a Azure válida: [cree una de manera gratuita](https://azure.microsoft.com/free/).
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* [!INCLUDE [terms-azure-portal](./terms-azure-portal.md)]

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

Cree una aplicación de .NET Core. En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `azure-management-quickstart`. Este comando crea un sencillo proyecto de C#, "Hola mundo", con un solo archivo de origen: *program.cs*. 

```console
dotnet new console -n azure-management-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

En el directorio de aplicaciones, instale la biblioteca cliente de Administración de Azure para .NET con el siguiente comando:

```console
dotnet add package Microsoft.Azure.Management.CognitiveServices
dotnet add package Microsoft.Azure.Management.Fluent
dotnet add package Microsoft.Azure.Management.ResourceManager.Fluent
```

Si usa el IDE de Visual Studio, la biblioteca cliente estará disponible como un paquete de NuGet descargable.

### <a name="import-libraries"></a>Importación de bibliotecas

Abra *program.cs* y agregue las siguientes instrucciones`using` a la parte superior del archivo:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_using)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Agregue los campos siguientes a la raíz de *program.cs* y rellene sus valores con la entidad de servicio que creó y la información de su cuenta de Azure.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_constants)]

Luego, en el método **Main**, use estos valores para construir un objeto **CognitiveServicesManagementClient**. Este objeto es necesario para todas las operaciones de Administración de Azure.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_assigns)]

## <a name="call-management-methods"></a>Métodos de administración de llamada

Agregue el código siguiente al método **Main** para enumerar los recursos disponibles, crear un recurso de ejemplo, enumerar los recursos que posee y, después, eliminar el recurso de ejemplo. Estos métodos los definirá en los siguientes pasos.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_calls)]

## <a name="create-a-cognitive-services-resource-c"></a>Creación de un recurso de Cognitive Services (C#)

Para crear un recurso de Cognitive Services y suscribirse a él, use el método **Create**. Este método agrega un nuevo recurso facturable al grupo de recursos que se pasa. Cuando cree el recurso, deberá conocer el "tipo" de servicio que desea usar, junto con su plan de tarifa (o SKU) y una ubicación de Azure. El siguiente método usa todos estos valores como argumentos y crea un recurso.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_create)]

### <a name="choose-a-service-and-pricing-tier"></a>Elección de un servicio y un plan de tarifa

Al crear un recurso, necesitará conocer el "tipo" de servicio que desea usar y el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/) (o la SKU) que desee. Usará esta y otra información como parámetros al crear el recurso. Puede encontrar una lista de los "tipos" de Cognitive Service disponibles llamando al siguiente método en el script:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="view-your-resources"></a>Visualización de los recursos

Para ver todos los recursos de su cuenta de Azure (de todos los grupos de recursos), use el siguiente método:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_list)]

## <a name="delete-a-resource"></a>Eliminación de un recurso

El método siguiente elimina el recurso especificado del grupo de recursos dado.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/azure_management_service/create_delete_resource.cs?name=snippet_delete)]

Si necesita recuperar un recurso eliminado, consulte el artículo sobre la [recuperación de recursos de Cognitive Services eliminados](../../manage-resources.md).

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```dotnet
dotnet run
```

## <a name="see-also"></a>Consulte también

* Consulte **[Autenticación de solicitudes en Azure Cognitive Services](../../authentication.md)** para aprender a trabajar de forma segura con Cognitive Services.
* Consulte **[¿Qué es Azure Cognitive Services?](../../what-are-cognitive-services.md)** para obtener una lista de las distintas categorías de Cognitive Services.
* Consulte **[Compatibilidad con idiomas naturales](../../language-support.md)** para ver la lista de los idiomas naturales admitidos por Cognitive Services.
* Consulte **[Contenedores de Azure Cognitive Services](../../cognitive-services-container-support.md)** para aprender a usar Cognitive Services en un entorno local.
* Consulte **[Planeamiento y administración de los costos de Azure Cognitive Services](../../plan-manage-costs.md)** para estimar el costo de usar Cognitive Services.
* Consulte la **[documentación de referencia del SDK de administración de Cognitive Services](/dotnet/api/overview/azure/cognitiveservices/management)** para más detalles sobre el SDK de administración.
