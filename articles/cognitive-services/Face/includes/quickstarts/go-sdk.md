---
title: Inicio rápido de la biblioteca cliente de Go para Face
description: Use la biblioteca cliente de Face para Go a fin de detectar caras, buscar similares (búsqueda de cara por imagen), identificar caras (búsqueda de reconocimiento facial) y migrar los datos de cara.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 10/26/2020
ms.author: pafarley
ms.openlocfilehash: 57c152546bfdbcdfbeba45536990c6c204106e7a
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109857998"
---
Comience a usar el reconocimiento facial con la biblioteca cliente de Face para Go. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. El servicio Face le proporciona acceso a algoritmos avanzados para detectar y reconocer rostros humanas en imágenes.

Use la biblioteca cliente del servicio Face para Go para la:

* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación y entrenamiento de un objeto PersonGroup](#create-and-train-a-persongroup)
* [Identificación de una cara](#identify-a-face)

[Documentación de referencia](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v1.0/face) | [Descarga de SDK](https://github.com/Azure/azure-sdk-for-go)

## <a name="prerequisites"></a>Requisitos previos

* La versión más reciente de [Go](https://golang.org/dl/).
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Creación de un recurso de Face"  target="_blank">cree un recurso de Face </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Después de obtener una clave y un punto de conexión, [crear variables de entorno](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) para la clave y el punto de conexión, denominados `FACE_SUBSCRIPTION_KEY` y `FACE_ENDPOINT`, respectivamente.

## <a name="setting-up"></a>Instalación

### <a name="create-a-go-project-directory"></a>Creación de un directorio del proyecto de Go

En una ventana de consola (cmd, PowerShell, Terminal, Bash), cree una nueva área de trabajo para el proyecto de Go llamado `my-app` y vaya hasta ella.

```
mkdir -p my-app/{src, bin, pkg}  
cd my-app
```

El área de trabajo contendrá tres carpetas:

* **src**: este directorio contendrá código fuente y paquetes. Todos los paquetes instalados con el comando `go get` se encontrarán en esta carpeta.
* **pkg**: este directorio contendrá objetos de paquete de Go compilados. Todos estos archivos tienen la extensión `.a`.
* **bin**: este directorio contendrá los archivos ejecutables binarios que se crean al ejecutar `go install`.

> [!TIP]
> Para más información sobre la estructura de un área de trabajo de Go, consulte la [documentación del lenguaje Go](https://golang.org/doc/code.html#Workspaces). En esta guía se incluye información para establecer `$GOPATH` y `$GOROOT`.

### <a name="install-the-client-library-for-go"></a>Instalación de la biblioteca cliente para Go

Instale a continuación la biblioteca cliente para Go:

```bash
go get -u github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v1.0/face
```

O, si usa dep, dentro de su repositorio ejecute:

```bash
dep ensure -add https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v1.0/face
```

### <a name="create-a-go-application"></a>Creación de una aplicación de Go

A continuación, cree un archivo en el directorio **src** denominado `sample-app.go`:

```bash
cd src
touch sample-app.go
```

Abra `sample-app.go` en el entorno de desarrollo integrado o editor de texto que prefiera. A continuación, agregue el nombre del paquete e importe las siguientes bibliotecas:

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_imports)]

A continuación, empezará a agregar código para llevar a cabo distintas operaciones del servicio Face.

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de Go para Face.

|Nombre|Descripción|
|---|---|
|[BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#BaseClient) | Esta clase representa la autorización para usar el servicio Face, se necesita para que Face funcione correctamente. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases. |
|[Cliente](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client)|Esta clase controla las tareas básicas de detección y reconocimiento que puede realizar con las caras humanas. |
|[DetectedFace](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#DetectedFace)|Esta clase representa todos los datos que se detectaron de una única cara en una imagen. Puede usarla para recuperar información detallada sobre la cara.|
|[ListClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#ListClient)|Esta clase administra las construcciones **FaceList** almacenadas en la nube, que almacenan a su vez un conjunto ordenado de caras. |
|[PersonGroupPersonClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupPersonClient)| Esta clase administra las construcciones **Person** almacenadas en la nube, que almacenan a su vez un conjunto de caras que pertenecen a una sola persona.|
|[PersonGroupClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupClient)| Esta clase administra las construcciones **PersonGroup** almacenadas en la nube, que almacenan a su vez un conjunto de objetos **Person** ordenados. |
|[SnapshotClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#SnapshotClient)|Esta clase administra la funcionalidad de instantáneas. Puede usarla para guardar temporalmente todos los datos de caras en la nube y migrar los datos a una nueva suscripción de Azure. |

## <a name="code-examples"></a>Ejemplos de código

En estos ejemplos de código se muestra cómo realizar tareas básicas con la biblioteca cliente del servicio Face para Go:

* [Autenticar el cliente](#authenticate-the-client)
* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación y entrenamiento de un objeto PersonGroup](#create-and-train-a-persongroup)
* [Identificación de una cara](#identify-a-face)

## <a name="authenticate-the-client"></a>Autenticar el cliente

> [!NOTE] 
> En este inicio rápido se da por supuesto que ha [creado variables de entorno](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) para la clave y el punto de conexión de Face, denominadas `FACE_SUBSCRIPTION_KEY` y `FACE_ENDPOINT` respectivamente.

Cree una función **main** y agréguele el código siguiente para crear una instancia de un cliente con su punto de conexión y clave. Puede crear un objeto **[CognitiveServicesAuthorizer](https://godoc.org/github.com/Azure/go-autorest/autorest#CognitiveServicesAuthorizer)** con la clave y usarlo con el punto de conexión para crear un objeto **[Client](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client)** . Este código también crea instancias de un objeto de contexto, lo cual es necesario para la creación de objetos de cliente. También define una ubicación remota en la que se encuentran algunas de las imágenes de ejemplo de este inicio rápido.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_main_client)]


## <a name="detect-faces-in-an-image"></a>Detectar caras en una imagen

Agregue el siguiente código al método **main**. Este código define una imagen de ejemplo remota y especifica qué características faciales se van a extraer de la imagen. También especifica el modelo de inteligencia artificial que se va a usar para extraer datos de las caras detectadas. Consulte [Especificación de un modelo de reconocimiento](../../Face-API-How-to-Topics/specify-recognition-model.md) para información sobre estas opciones. Por último, el método **[DetectWithURL](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client.DetectWithURL)** realiza la operación de detección de caras en la imagen y guarda los resultados en la memoria del programa.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_detect)]

> [!TIP]
> También puede detectar caras en una imagen local. Consulte los métodos [Client](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client), como **DetectWithStream**.

### <a name="display-detected-face-data"></a>Visualización de los datos faciales detectados

El siguiente bloque de código toma el primer elemento de la matriz de objetos **[DetectedFace](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#DetectedFace)** e imprime sus atributos en la consola. Si usó una imagen con varias caras, debe iterar la matriz en su lugar.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_detect_display)]

## <a name="find-similar-faces"></a>Búsqueda de caras similares

El siguiente código toma una sola cara detectada (origen) y busca en un conjunto de otras caras (destino) para encontrar coincidencias (búsqueda de cara por imagen). Cuando la encuentra, imprime el identificador de la cara coincidente en la consola.

### <a name="detect-faces-for-comparison"></a>Detección de caras para la comparación

En primer lugar, guarde una referencia a la cara que detectó en la sección [Detectar caras en una imagen](#detect-faces-in-an-image). Esta cara será el origen.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar_single_ref)]

A continuación, escriba el código siguiente para detectar un conjunto de caras en una imagen diferente. Estas caras serán el destino.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar_multiple_ref)]

### <a name="find-matches"></a>Búsqueda de coincidencias

En el código siguiente se usa el método **[FindSimilar](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client.FindSimilar)** para buscar todas las caras de destino que coinciden con la cara de origen.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar)]

### <a name="print-matches"></a>Impresión de las coincidencias

El siguiente código imprime los detalles coincidentes en la consola.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar_print)]


## <a name="create-and-train-a-persongroup"></a>Creación y entrenamiento de un objeto PersonGroup

Para seguir en este escenario, debe guardar las siguientes imágenes en el directorio raíz del proyecto: https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

Este grupo de imágenes contiene tres conjuntos de imágenes de caras simples correspondientes a tres personas distintas. El código definirá tres objetos **PersonGroup Person** y los asociará con archivos de imagen que comienzan por `woman`, `man` y `child`.

### <a name="create-persongroup"></a>Creación de un objeto PersonGroup

Una vez que haya descargado las imágenes, agregue el código siguiente en la parte inferior del método **main**. Este código autentica un objeto **[PersonGroupClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupClient)** y, a continuación, lo usa para definir un nuevo objeto **PersonGroup**.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pg_setup)]

### <a name="create-persongroup-persons"></a>Creación de un objeto PersonGroup Persons

El siguiente bloque de código autentica un **[PersonGroupPersonClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupPersonClient)** y lo usa para definir tres nuevos objetos **PersonGroup Person**. Cada uno de estos objetos representa una sola persona en el conjunto de imágenes.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pgp_setup)]

### <a name="assign-faces-to-persons"></a>Asignación de caras a los objetos Person

El siguiente código ordena las imágenes por su prefijo, detecta caras y las asigna a cada objeto **PersonGroup Person** respectivo en función del nombre de archivo de la imagen.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pgp_assign)]

> [!TIP]
> También puede crear una clase **PersonGroup** a partir de imágenes remotas referenciadas por una dirección URL. Consulte los métodos [PersonGroupPersonClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupPersonClient), como **AddFaceFromURL**.

### <a name="train-persongroup"></a>Entrenamiento del objeto PersonGroup

Una vez asignadas las caras, entrene el objeto **PersonGroup** para que identifique las características visuales asociadas con cada uno de sus objetos **Person**. El siguiente código llama al método **train** asincrónico, sondea el resultado e imprime el estado en la consola.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pg_train)]

> [!TIP]
> API Face se ejecuta en un conjunto de modelos precompilados que son estáticos por naturaleza (el rendimiento del modelo no empeorará ni mejorará si se ejecuta el servicio). Los resultados que genera el modelo pueden cambiar si Microsoft actualiza su back-end sin migrar a una versión de modelo completamente nueva. Para aprovechar las ventajas de una versión más reciente de un modelo, puede volver a entrenar **PersonGroup**, pero especifique el modelo más reciente como un parámetro con las mismas imágenes de inscripción.

## <a name="identify-a-face"></a>Identificar una cara

La operación de identificación toma una imagen de una persona (o de varias) y busca la identidad de cada una de las caras de la imagen (búsqueda de reconocimiento facial). Compara cada cara detectada con un objeto **PersonGroup**, una base de datos con distintos objetos **Person** cuyos rasgos faciales se conocen.

> [!IMPORTANT]
> Para ejecutar este ejemplo, primero debe ejecutar el código que se indica en [Creación y entrenamiento de un objeto PersonGroup](#create-and-train-a-persongroup).

### <a name="get-a-test-image"></a>Obtención de una imagen de prueba

El siguiente código busca en la raíz del proyecto una imagen _test-image-person-group.jpg_ y la carga en la memoria del programa. Puede encontrar esta imagen en el mismo repositorio que las imágenes que se usan en [Creación y entrenamiento de un objeto PersonGroup](#create-and-train-a-persongroup): https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id_source_get)]

### <a name="detect-source-faces-in-test-image"></a>Detección de caras de origen en la imagen de prueba

El siguiente bloque de código realiza la detección de caras normal en la imagen de prueba para recuperar todas las caras y guardarlas en una matriz.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id_source_detect)]

### <a name="identify-faces"></a>Identificación de caras

El método **[Identify](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client.Identify)** toma la matriz de caras detectadas y las compara con el grupo **PersonGroup** proporcionado (que se definió y entrenó en la sección anterior). Si puede hacer coincidir una cara detectada con un objeto **Person** del grupo, guarda el resultado.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id)]

Este código imprime los resultados detallados de las coincidencias en la consola.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id_print)]


## <a name="verify-faces"></a>Comprobar caras

La operación Verificar toma dos identificadores de caras o un objeto **Person**, y determina si pertenecen a la misma persona.

En el código siguiente se detectan caras en dos imágenes de origen y, a continuación, se compara cada una con una cara detectada a partir de una imagen de destino.

### <a name="get-test-images"></a>Obtención de imágenes de prueba

Los siguientes bloques de código declaran variables que apuntarán a las imágenes de origen y de destino para la operación de comprobación.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver_images)]

### <a name="detect-faces-for-verification"></a>Detectar caras para la comprobación

El código siguiente detecta caras en las imágenes de origen y de destino y las guarda en variables.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver_detect_source)]

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver_detect_target)]

### <a name="get-verification-results"></a>Obtención de los resultados de la comprobación

El código siguiente compara cada una de las imágenes de origen con la imagen de destino e imprime un mensaje que indica si pertenecen a la misma persona.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación de reconocimiento facial desde el directorio de la aplicación con el comando `go run <app-name>`.

```bash
go run sample-app.go
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

Si ha creado un objeto **PersonGroup** en este inicio rápido y desea eliminarlo, llame al método **[Delete](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupClient.Delete)** .

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca cliente de Face para Go para realizar tareas básicas de reconocimiento facial. A continuación, obtenga información sobre los diferentes modelos de detección de caras y cómo especificar el modelo adecuado para su caso de uso.

> [!div class="nextstepaction"]
> [Especificación de una versión del modelo de detección de caras](../../Face-API-How-to-Topics/specify-detection-model.md)

* [¿Qué es el servicio Face?](../../overview.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/Face/FaceQuickstart.go).