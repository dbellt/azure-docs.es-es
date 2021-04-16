---
title: Inicio rápido de la biblioteca cliente para JavaScript de Face
description: La biblioteca cliente de Face para JavaScript se usa para detectar caras, buscar similares (búsqueda de cara por imagen), identificar caras (búsqueda de reconocimiento facial) y migrar los datos de la cara.
services: cognitive-services
author: v-jaswel
manager: chrhoder
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 11/05/2020
ms.author: v-jawe
ms.openlocfilehash: 8f968572a357bb3c98d9c3133a7ec0a0a94dbf93
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105958073"
---
## <a name="quickstart-face-client-library-for-javascript"></a>Inicio rápido: Biblioteca cliente de Face para JavaScript

Comience a usar el reconocimiento facial con la biblioteca cliente de Face para JavaScript. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. El servicio Face le proporciona acceso a algoritmos avanzados para detectar y reconocer rostros humanas en imágenes.

La biblioteca cliente de Face para JavaScript se usa para realizar las siguientes operaciones:

* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación de un elemento PersonGroup](#create-a-persongroup)
* [Identificación de una cara](#identify-a-face)

[Documentación de referencia](/javascript/api/@azure/cognitiveservices-face/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-face) | [Paquete (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-face) | [Ejemplos](/samples/browse/?products=azure&term=face&languages=javascript)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* La versión más reciente de [Node.js](https://nodejs.org/en/).
* Una vez que tenga la suscripción de Azure, [cree un recurso de Face](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init` para crear una aplicación de nodo con un archivo `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente 

Instale los paquetes `ms-rest-azure` y `azure-cognitiveservices-face` de NPM:

```console
npm install @azure/cognitiveservices-face @azure/ms-rest-js
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

Cree un archivo llamado `index.js` e importe las bibliotecas siguientes:

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](), que contiene los ejemplos de código de este inicio rápido.

```javascript
const msRest = require("@azure/ms-rest-js");
const Face = require("@azure/cognitiveservices-face");
const uuid = require("uuid/v4");
```

Cree variables para el punto de conexión y la clave de Azure del recurso. 

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Face que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar su clave y punto de conexión en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**. 
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

```javascript
key = "<paste-your-face-key-here>"
endpoint = "<paste-your-face-endpoint-here>"
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de .NET para Face:

|Nombre|Descripción|
|---|---|
|[FaceClient](/javascript/api/@azure/cognitiveservices-face/faceclient) | Esta clase representa la autorización para usar el servicio Face, se necesita para que Face funcione correctamente. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases. |
|[Face](/javascript/api/@azure/cognitiveservices-face/face)|Esta clase controla las tareas básicas de detección y reconocimiento que puede realizar con las caras humanas. |
|[DetectedFace](/javascript/api/@azure/cognitiveservices-face/detectedface)|Esta clase representa todos los datos que se detectaron de una única cara en una imagen. Puede usarla para recuperar información detallada sobre la cara.|
|[FaceList](/javascript/api/@azure/cognitiveservices-face/facelist)|Esta clase administra las construcciones **FaceList** almacenadas en la nube, que almacenan a su vez un conjunto ordenado de caras. |
|[PersonGroupPerson](/javascript/api/@azure/cognitiveservices-face/persongroupperson)| Esta clase administra las construcciones **Person** almacenadas en la nube, que almacenan a su vez un conjunto de caras que pertenecen a una sola persona.|
|[PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup)| Esta clase administra las construcciones **PersonGroup** almacenadas en la nube, que almacenan a su vez un conjunto de objetos **Person** ordenados. |

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Face para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación de un elemento PersonGroup](#create-a-persongroup)
* [Identificación de una cara](#identify-a-face)

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/Face/sdk_quickstart.js), que contiene los ejemplos de código de este inicio rápido.

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto **[ApiKeyCredentials](/javascript/api/@azure/ms-rest-js/apikeycredentials)** con la clave y úselo con el punto de conexión para crear un objeto **[FaceClient](/javascript/api/@azure/cognitiveservices-face/faceclient)** .

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="credentials":::

## <a name="declare-global-values-and-helper-function"></a>Declaración de valores globales y funciones del asistente

Los siguientes valores globales son necesarios para algunas de las operaciones de Face que agregará más adelante.

La dirección URL señala a una carpeta de imágenes de ejemplo. El UUID actuará como nombre y como identificador de los objetos PersonGroup que va a crear.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="globals":::

Usará la siguiente función para esperar a que se complete el entrenamiento de PersonGroup.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="helpers":::

## <a name="detect-faces-in-an-image"></a>Detectar caras en una imagen

### <a name="get-detected-face-objects"></a>Obtención de los objetos faciales detectados

Cree un método para detectar caras. El método `DetectFaceExtract` procesa tres de las imágenes en la dirección URL especificada y crea una lista de objetos **[DetectedFace](/javascript/api/@azure/cognitiveservices-face/detectedface)** en la memoria del programa. La lista de valores **[FaceAttributeType](/javascript/api/@azure/cognitiveservices-face/faceattributetype)** especifica qué rasgos se van a extraer. 

Luego, el método `DetectFaceExtract` analiza e imprime los datos de atributo de cada una de las caras detectadas. Cada atributo debe especificarse por separado en la llamada API de detección de caras original (en la lista **[FaceAttributeType](/javascript/api/@azure/cognitiveservices-face/faceattributetype)** ). En el código siguiente se procesan todos los atributos, pero probablemente solo tendrá que usar uno o algunos.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="detect":::

> [!TIP]
> También puede detectar caras en una imagen local. Consulte los métodos de [Face](/javascript/api/@azure/cognitiveservices-face/face), como [DetectWithStream](/javascript/api/@azure/cognitiveservices-face/face#detectWithStream_msRest_HttpRequestBody__FaceDetectWithStreamOptionalParams__ServiceCallback_DetectedFace____).

## <a name="find-similar-faces"></a>Búsqueda de caras similares

El siguiente código toma una sola cara detectada (origen) y busca en un conjunto de otras caras (destino) para encontrar coincidencias (búsqueda de cara por imagen). Cuando la encuentra, imprime el identificador de la cara coincidente en la consola.

### <a name="detect-faces-for-comparison"></a>Detección de caras para la comparación

En primer lugar, defina un segundo método de detección de caras. Debe detectar las caras en las imágenes para poder compararlas y este método de detección está optimizado para las operaciones de comparación. No extrae los atributos de cara detallados como en la sección anterior y usa un modelo de reconocimiento diferente.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="recognize":::

### <a name="find-matches"></a>Búsqueda de coincidencias

El método siguiente detecta caras en un conjunto de imágenes de destino y en una sola imagen de origen. Después, los compara y busca todas las imágenes de destino que son similares a la imagen de origen. Por último, imprime los detalles coincidentes en la consola.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="find_similar":::

## <a name="identify-a-face"></a>Identificar una cara

La operación [Identify](/javascript/api/@azure/cognitiveservices-face/face#identify_string____FaceIdentifyOptionalParams__ServiceCallback_IdentifyResult____) toma una imagen de una persona (o de varias) y busca la identidad de cada una de las caras de la imagen (búsqueda de reconocimiento facial). Compara cada cara detectada con un objeto [PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup), una base de datos con distintos objetos [Person](/javascript/api/@azure/cognitiveservices-face/person) cuyos rasgos faciales se conocen. Para poder realizar la operación Identify, primero es preciso crear y entrenar un objeto [PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup).

### <a name="add-faces-to-persongroup"></a>Incorporación de caras a PersonGroup

Cree la siguiente función para agregar caras a [PersonGroup](/javascript/api/@azure/cognitiveservices-face/persongroup).

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="add_faces":::

### <a name="wait-for-training-of-persongroup"></a>Espera para el entrenamiento de PersonGroup

Cree la siguiente función del asistente para esperar a que **PersonGroup** termine el entrenamiento.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="wait_for_training":::

### <a name="create-a-persongroup"></a>Creación de un elemento PersonGroup

El código siguiente:
- Crea un objeto [PersonGroup](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-face/persongroup)
- Agrega caras al objeto **PersonGroup** mediante una llamada a `AddFacesToPersonGroup` que se ha definido previamente.
- Entrena el objeto **PersonGroup**.
- Identifica las caras del objeto **PersonGroup**.

El objeto **PersonGroup** y sus objetos **Person** asociados ya están listos para usarse en las operaciones de comprobación, identificación o agrupación.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="identify":::

> [!TIP]
> También puede crear una clase **PersonGroup** a partir de imágenes locales. Consulte los métodos [IPersonGroupPerson](/javascript/api/@azure/cognitiveservices-face/persongroupperson), como [AddFaceFromStream](/javascript/api/@azure/cognitiveservices-face/persongroupperson#addFaceFromStream_string__string__msRest_HttpRequestBody__Models_PersonGroupPersonAddFaceFromStreamOptionalParams_).

## <a name="main"></a>Método Main

Por último, cree la función `main` y llámela.

:::code language="js" source="~/cognitive-services-quickstart-code/javascript/Face/sdk_quickstart.js" id="main":::

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca cliente de Face para JavaScript para realizar tareas básicas de reconocimiento facial. A continuación, consulte la documentación de referencia para más información sobre la biblioteca.

> [!div class="nextstepaction"]
> [Referencia de Face API (JavaScript)](/javascript/api/@azure/cognitiveservices-face/)

* [¿Qué es el servicio Face?](../../overview.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/Face/sdk_quickstart.js).