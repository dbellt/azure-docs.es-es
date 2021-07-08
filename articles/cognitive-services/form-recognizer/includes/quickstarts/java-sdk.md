---
title: 'Inicio rápido: Biblioteca cliente de Form Recognizer para Java'
description: Use la biblioteca cliente de Form Recognizer para Java para crear una aplicación de procesamiento de formularios que extraiga pares clave-valor y datos de tabla de los documentos personalizados.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 05/25/2021
ms.custom: devx-track-java
ms.author: lajanuar
ms.openlocfilehash: de9e52dcd372740a7ba7a50e3ace3390bbf0af03
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111894336"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->

> [!IMPORTANT]
>
> * En este artículo de inicio rápido se usa la versión **3.1.0** del SDK y la API de destino, versión **2.1**.
>
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas.

[Documentación de referencia](/java/api/overview/azure/ai-formrecognizer-readme) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src) | [Paquete (Maven)](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer) | [Ejemplos](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La última versión de [Java Development Kit (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* La [herramienta de compilación de Gradle](https://gradle.org/install/) u otro administrador de dependencias.
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Train** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451) (descargue y extraiga *sample_data.zip*).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-gradle-project"></a>Creación de un proyecto de Gradle

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él.

```console
mkdir myapp && cd myapp
```

Ejecute el comando `gradle init` desde el directorio de trabajo. Este comando creará archivos de compilación esenciales para Gradle, como *build.gradle.kts*, que se usa en el runtime para crear y configurar la aplicación.

```console
gradle init --type basic
```

Cuando se le solicite que elija un **DSL**, seleccione **Kotlin**.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

En este inicio rápido se usa el administrador de dependencias Gradle. Puede encontrar la biblioteca de cliente y la información de otros administradores de dependencias en el [repositorio central de Maven](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer).

En el archivo *build.gradle.kts* del proyecto, incluya la biblioteca cliente como una instrucción `implementation`, junto con los complementos y la configuración necesarios.

```kotlin
plugins {
    java
    application
}
application {
    mainClass.set("FormRecognizer")
}
repositories {
    mavenCentral()
}
dependencies {
    implementation(group = "com.azure", name = "azure-ai-formrecognizer", version = "3.1.0")
}
```

### <a name="create-a-java-file"></a>Creación de un archivo Java

En el directorio de trabajo, ejecute el siguiente comando:

```console
mkdir -p src/main/java
```

Vaya a la nueva carpeta y cree un archivo llamado *FormRecognizer.java*. Ábralo en el editor o el IDE que prefiera y agregue las siguientes instrucciones `import`:

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_imports)]

> [!TIP]
> Si quiere ver todo el archivo con los ejemplos de código de este artículo de inicio rápido, puede encontrarlo en [**GitHub**](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/FormRecognizer/FormRecognizer.java).

En la clase **FormRecognizer** de la aplicación, cree variables para el punto de conexión y la clave del recurso.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_creds)]

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Form Recognizer que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar su clave y punto de conexión en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**.
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En producción, use métodos seguros para almacenar y acceder a sus credenciales. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

En el método **main** de la aplicación, agregue llamadas para los métodos que se usan en este inicio rápido. Se definirán más adelante. También tendrá que agregar referencias a las direcciones URL de los datos de entrenamiento y prueba.

* [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Recuperación de la dirección URL de SAS":::
* Para obtener una dirección URL de un formulario para prueba, puede usar los pasos anteriores para obtener la dirección URL de SAS de un documento del almacenamiento de blobs. También puede tomar la dirección URL de un documento situado en otro lugar.
* Use el método anterior para obtener también la dirección URL de una imagen de recibo.
<!-- markdownlint-disable MD024 -->

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_mainvars)]

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_maincalls)]

## <a name="object-model"></a>Modelo de objetos

Con Form Recognizer, puede crear dos tipos de cliente diferentes. El primero, `FormRecognizerClient`, se utiliza para consultar el servicio con los campos de formulario y el contenido reconocidos. El segundo, `FormTrainingClient`, se usa para crear y administrar modelos personalizados que se pueden emplear para mejorar el reconocimiento.

### <a name="formrecognizerclient"></a>FormRecognizerClient

`FormRecognizerClient` proporciona operaciones para:

* Reconocimiento de los campos de formulario y del contenido, mediante el uso de modelos personalizados entrenados para analizar los formularios personalizados.  Estos valores se devuelven en una colección de objetos `RecognizedForm`. Vea el ejemplo [Análisis de formularios personalizados](#analyze-forms-with-a-custom-model).
* El reconocimiento del contenido de los formularios, incluidas tablas, líneas y palabras, sin necesidad de entrenar un modelo.  El contenido del formulario se devuelve en una colección de objetos `FormPage`. Consulte el ejemplo [Análisis de diseño](#analyze-layout).
* Reconocimiento de los campos comunes de recibos de EE. UU., tarjetas de presentación, facturas y documentos de identificación con un modelo entrenado previamente en el servicio Form Recognizer.

### <a name="formtrainingclient"></a>FormTrainingClient

`FormTrainingClient` proporciona operaciones para:

* El entrenamiento de modelos personalizados para analizar todos los campos y los valores que se encuentren en los formularios personalizados.  Se devuelve un `CustomFormModel` que indica los tipos de formulario que el modelo analizará y los campos que se extraerán para cada tipo de formulario.
* El entrenamiento de modelos personalizados para analizar los campos y los valores concretos que especifique mediante la etiqueta de los formularios personalizados.  Se devuelve un `CustomFormModel` que indica los campos que el modelo va a extraer, así como la precisión estimada para cada campo.
* La administración de los modelos creados en una cuenta.
* La copia de un modelo personalizado entre recursos de Form Recognizer.

> [!NOTE]
> Los modelos también se pueden entrenar mediante una interfaz gráfica de usuario, como la [herramienta de etiquetado de Form Recognizer](../../label-tool.md).

## <a name="authenticate-the-client"></a>Autenticar el cliente

En la parte superior del método **main**, agregue el código siguiente. Aquí, autenticará dos objetos de cliente mediante las variables de suscripción que definió anteriormente. Usará un objeto **AzureKeyCredential**, de modo que, si es necesario, pueda actualizar la clave de API sin crear otros objetos de cliente.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_auth)]

## <a name="analyze-layout"></a>Análisis de diseño

Puede usar Form Recognizer para analizar tablas, líneas y palabras de los documentos sin necesidad de entrenar un modelo. Para más información sobre la extracción del diseño, consulte la [guía conceptual sobre diseño](../../concept-layout.md).

Para analizar el contenido de un archivo en una dirección URL determinada, use el método **beginRecognizeContentFromUrl**.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_getcontent_call)]

> [!TIP]
> También puede obtener contenido de un archivo local. Consulte los métodos [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), como **beginRecognizeContent**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) para escenarios relacionados con imágenes locales.

El valor devuelto es una colección de objetos **FormPage**, uno para cada página del documento enviado. El código siguiente recorre en iteración estos objetos e imprime los pares clave-valor y los datos de tabla extraídos.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_getcontent_print)]

### <a name="output"></a>Output

```console
Get form content...
----Recognizing content ----
Has width: 8.500000 and height: 11.000000, measured with unit: inch.
Table has 2 rows and 6 columns.
Cell has text Invoice Number.
Cell has text Invoice Date.
Cell has text Invoice Due Date.
Cell has text Charges.
Cell has text VAT ID.
Cell has text 458176.
Cell has text 3/28/2018.
Cell has text 4/16/2018.
Cell has text $89,024.34.
Cell has text ET.
```

## <a name="analyze-receipts"></a>Análisis de las confirmaciones de recepción

En esta sección se muestra cómo analizar y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente. Para más información sobre el análisis de recibos, consulte la [guía conceptual sobre recibos](../../concept-receipts.md).

Para analizar los recibos a partir de un URI, use el método **beginRecognizeReceiptsFromUrl**.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_receipts_call)]

> [!TIP]
> También puede analizar imágenes de recibos locales. Consulte los métodos [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), como **beginRecognizeReceipts**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) para escenarios relacionados con imágenes locales.

El valor devuelto es una colección de objetos **RecognizedReceipt**, uno para cada página del documento enviado. El siguiente bloque de código recorre en iteración los recibos e imprime los detalles en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_receipts_print)]

El siguiente bloque de código recorre en iteración los elementos detectados en el recibo e imprime los detalles en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_receipts_print_items)]

### <a name="output"></a>Resultados

```console
Analyze receipt...
----------- Recognized Receipt page 0 -----------
Merchant Name: Contoso Contoso, confidence: 0.62
Merchant Address: 123 Main Street Redmond, WA 98052, confidence: 0.99
Transaction Date: 2020-06-10, confidence: 0.90
Receipt Items:
Name: Cappuccino, confidence: 0.96s
Quantity: null, confidence: 0.957s]
Total Price: 2.200000, confidence: 0.95
Name: BACON & EGGS, confidence: 0.94s
Quantity: null, confidence: 0.927s]
Total Price: null, confidence: 0.93
```

## <a name="analyze-business-cards"></a>Análisis de tarjetas de presentación

En esta sección se muestra cómo analizar y extraer campos comunes de tarjetas de presentación inglesas mediante un modelo entrenado previamente. Para más información acerca del análisis de tarjetas de presentación, consulte la [guía conceptual sobre tarjetas de presentación](../../concept-business-cards.md).

Para analizar tarjetas de presentación en una dirección URL, use el método `beginRecognizeBusinessCardsFromUrl`.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_bc_call)]

> [!TIP]
> También puede analizar imágenes de tarjeta de presentación locales. Consulte los métodos de [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), como **beginRecognizeBusinessCards**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) para escenarios relacionados con imágenes locales.

El valor devuelto es una colección de objetos **RecognizedForm**, uno para cada tarjeta del documento. En el código siguiente se procesa la tarjeta de presentación en el identificador URI dado y se imprimen los campos y valores principales en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_bc_print)]

## <a name="analyze-invoices"></a>Análisis de facturas

En esta sección se muestra cómo analizar y extraer campos comunes de facturas de compra mediante un modelo entrenado previamente. Para más información sobre el análisis de facturas, consulte la [guía conceptual sobre facturas](../../concept-invoices.md).

Para analizar facturas de una dirección URL, use el método `beginRecognizeInvoicesFromUrl`.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_invoice_call)]

> [!TIP]
> También puede analizar facturas locales. Consulte los métodos [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), como **beginRecognizeInvoices**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) para escenarios relacionados con imágenes locales.

El valor devuelto es una colección de objetos **RecognizedForm**, uno para cada factura del documento. En el código siguiente se procesa la factura en el identificador URI especificado y se imprimen los campos y valores principales en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_invoice_print)]

## <a name="analyze-identity-documents"></a>Análisis de documentos de identidad

En esta sección se muestra cómo analizar y extraer información clave de documentos de identificación emitidos por la administración pública (pasaportes de todo el mundo y permisos de conducir de EE. UU.) mediante el modelo de identificación precompilado de Form Recognizer. Para obtener más información sobre el análisis de documentos de identificación, consulte nuestra [guía conceptual del modelo de identificación precompilado](../../concept-identification-cards.md).

Para analizar documentos de identificación de un identificador URI, use el método `beginRecognizeIdentityDocumentsFromUrl`.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_id_call)]

> [!TIP]
> También puede analizar imágenes de documentos de identificación locales. Consulte los métodos [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient), como **beginRecognizeIdentityDocuments**. Consulte también el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) para escenarios relacionados con imágenes locales.

En el código siguiente se procesa el documento de identificación en el identificador URI especificado y se imprimen los campos y valores principales en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_id_print)]

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

En esta sección se muestra cómo entrenar un modelo con sus propios datos. Un modelo entrenado puede generar datos estructurados que incluyan las relaciones clave-valor del documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

> [!NOTE]
> También puede entrenar modelos con una interfaz gráfica de usuario, como la [herramienta de etiquetado de ejemplo de Form Recognizer](../../label-tool.md).

### <a name="train-a-model-without-labels"></a>Entrenamiento de un modelo sin etiquetas

Entrene modelos personalizados para analizar todos los campos y valores que se encuentran en los formularios personalizados sin etiquetar manualmente los documentos de entrenamiento.

El método siguiente entrena un modelo en un conjunto de documentos dado e imprime el estado del modelo en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_train_call)]

El objeto devuelto **CustomFormModel** contiene información sobre los tipos de formulario que el modelo puede analizar y los campos que puede extraer de cada tipo de formulario. El siguiente bloque de código imprime esta información en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_train_print)]

Por último, este método devuelve el identificador único del modelo.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_train_return)]

### <a name="output"></a>Output

```console
Train Model with training data...
Model Id: 20c3544d-97b4-49d9-b39b-dc32d85f1358
Model Status: ready
Training started on: 2020-08-31T16:52:09Z
Training completed on: 2020-08-31T16:52:23Z

Recognized Fields:
The subModel has form type form-0
The model found field 'field-0' with label: Address:
The model found field 'field-1' with label: Charges
The model found field 'field-2' with label: Invoice Date
The model found field 'field-3' with label: Invoice Due Date
The model found field 'field-4' with label: Invoice For:
The model found field 'field-5' with label: Invoice Number
The model found field 'field-6' with label: VAT ID
```

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

También puede entrenar modelos personalizados mediante el etiquetado manual de los documentos de entrenamiento. En algunos escenarios, el entrenamiento con etiquetas conduce a un mejor rendimiento. Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información de etiquetas especiales ( *\<filename\>.pdf.labels.json*) en el contenedor de almacenamiento de blobs junto con los documentos de entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Cuando los tenga, puede llamar al método **beginTraining** con el parámetro *useTrainingLabels* establecido en `true`.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_trainlabels_call)]

El valor de **CustomFormModel** devuelto indica los campos que el modelo puede extraer, junto con su precisión estimada en cada campo. El siguiente bloque de código imprime esta información en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_trainlabels_print)]

### <a name="output"></a>Resultados

```console
Train Model with training data...
Model Id: 20c3544d-97b4-49d9-b39b-dc32d85f1358
Model Status: ready
Training started on: 2020-08-31T16:52:09Z
Training completed on: 2020-08-31T16:52:23Z

Recognized Fields:
The subModel has form type form-0
The model found field 'field-0' with label: Address:
The model found field 'field-1' with label: Charges
The model found field 'field-2' with label: Invoice Date
The model found field 'field-3' with label: Invoice Due Date
The model found field 'field-4' with label: Invoice For:
The model found field 'field-5' with label: Invoice Number
The model found field 'field-6' with label: VAT ID
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

En esta sección, se muestra cómo extraer información de clave-valor y otro contenido de los tipos de formulario personalizados, con los modelos que ha entrenado con sus propios formularios.

> [!IMPORTANT]
> Para implementar este escenario, debe haber entrenado ya un modelo para que pueda pasar su identificador al método siguiente. Consulte la sección [Entrenamiento de un modelo](#train-a-model-without-labels).

Usará el método **beginRecognizeCustomFormsFromUrl**.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_analyze_call)]

> [!TIP]
> También puede analizar un archivo local. Consulte los métodos [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), como **beginRecognizeCustomForms**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) para escenarios relacionados con imágenes locales.

El valor devuelto es una colección de objetos **RecognizedForm**, uno para cada página del documento enviado. El código siguiente imprime los resultados del análisis en la consola. Imprime cada campo reconocido y el valor correspondiente, junto con una puntuación de confianza.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_analyze_print)]

### <a name="output"></a>Resultados

```console
Analyze PDF form...
----------- Recognized custom form info for page 0 -----------
Form type: form-0
Field 'field-0' has label 'Address:' with a confidence score of 0.91.
Field 'field-1' has label 'Invoice For:' with a confidence score of 1.00.
Field 'field-2' has label 'Invoice Number' with a confidence score of 1.00.
Field 'field-3' has label 'Invoice Date' with a confidence score of 1.00.
Field 'field-4' has label 'Invoice Due Date' with a confidence score of 1.00.
Field 'field-5' has label 'Charges' with a confidence score of 1.00.
Field 'field-6' has label 'VAT ID' with a confidence score of 1.00.
```

## <a name="manage-custom-models"></a>Administración de modelos personalizados

En esta sección se muestra cómo administrar los modelos personalizados almacenados en su cuenta. Como ejemplo, en el código siguiente se realizan todas las tareas de administración del modelo en un único método. Comience por copiar la firma del método siguiente:

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage)]

### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Comprobación del número de modelos de la cuenta de recursos de FormRecognizer

El siguiente bloque de código comprueba cuántos modelos se han guardado en la cuenta de Form Recognizer y compara esta cifra con el límite de la cuenta.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage_count)]

#### <a name="output"></a>Resultados

```console
The account has 12 custom models, and we can have at most 250 custom models
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Enumeración de los modelos almacenados actualmente en la cuenta de recursos

El siguiente bloque de código enumera los modelos actuales de la cuenta e imprime los detalles en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage_list)]

#### <a name="output"></a>Resultados

Esta respuesta se ha truncado para facilitar la lectura.

```console
We have following models in the account:
Model Id: 0b048b60-86cc-47ec-9782-ad0ffaf7a5ce
Model Id: 0b048b60-86cc-47ec-9782-ad0ffaf7a5ce
Model Status: ready
Training started on: 2020-06-04T18:33:08Z
Training completed on: 2020-06-04T18:33:10Z
Custom Model Form type: form-0b048b60-86cc-47ec-9782-ad0ffaf7a5ce
Custom Model Accuracy: 1.00
Field Text: invoice date
Field Accuracy: 1.00
Field Text: invoice number
Field Accuracy: 1.00
...
```

### <a name="delete-a-model-from-the-resource-account"></a>Eliminación de un modelo de la cuenta de recursos

También puede eliminar un modelo de su cuenta haciendo referencia a su identificador.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage_delete)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Vuelva al directorio principal del proyecto. Luego, compile la aplicación con el siguiente comando:

```console
gradle build
```

Ejecute la aplicación con el objetivo `run`:

```console
gradle run
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Solución de problemas

Los clientes de From Recognizer generan excepciones `ErrorResponseException`. Por ejemplo, si intenta proporcionar una dirección URL de origen de archivo no válida, se producirá una excepción `ErrorResponseException` con un error que indica la causa. En el siguiente fragmento de código, el error se controla correctamente mediante la captura y presentación de la información adicional sobre el error.

```java Snippet:FormRecognizerBadRequest
try {
    formRecognizerClient.beginRecognizeContentFromUrl("invalidSourceUrl");
} catch (ErrorResponseException e) {
    System.out.println(e.getMessage());
}
```

### <a name="enable-client-logging"></a>Habilitación del registro de cliente

Los SDK de Azure para Java ofrecen un escenario de registro coherente que ayuda a solucionar los errores de la aplicación y a acelerar su resolución. Los registros generados capturarán el flujo de una aplicación antes de alcanzar el estado terminal para ayudar a encontrar el problema raíz. Consulte la [wiki de registro](https://github.com/Azure/azure-sdk-for-java/wiki/Logging-with-Azure-SDK) para obtener instrucciones sobre cómo habilitar el registro.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se ha usado la biblioteca cliente de Java para Form Recognizer para entrenar modelos y analizar formularios de maneras diferentes. A continuación, obtenga sugerencias para crear un mejor conjunto de datos de entrenamiento y generar modelos más precisos.

> [!div class="nextstepaction"]
> [Creación de un conjunto de datos de aprendizaje](../../build-training-data-set.md)

* [¿Qué es Form Recognizer?](../../overview.md)
* El código de ejemplo de esta guía (y más) se puede encontrar en [GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples).
