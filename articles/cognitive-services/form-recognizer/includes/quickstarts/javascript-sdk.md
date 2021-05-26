---
title: 'Inicio rápido: Biblioteca cliente de Form Recognizer para JavaScript'
description: Use la biblioteca cliente de Form Recognizer para JavaScript con el fin de crear una aplicación de procesamiento de formularios que extraiga pares clave-valor y datos de tabla de los documentos personalizados.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 05/12/2021
ms.author: lajanuar
ms.custom: devx-track-js
ms.openlocfilehash: 84d6f181636c65aea5c247185bfd7ef083151c75
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110374208"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->

> [!IMPORTANT]
>
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. Para más información, consulte la siguiente documentación de referencia.

[Documentación de referencia](../../index.yml) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/formrecognizer/ai-form-recognizer/) | [Paquete (npm)](https://www.npmjs.com/package/@azure/ai-form-recognizer) | [Ejemplos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org/)
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Train** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451) (descargue y extraiga *sample_data.zip*).
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource </a> en Azure Portal para obtener la clave y el punto de conexión. Tras su implementación, seleccione **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
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

Instale el paquete NPM `ai-form-recognizer`:

```console
npm install @azure/ai-form-recognizer
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

Cree un archivo llamado `index.js`, ábralo e importe las bibliotecas siguientes:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_imports)]

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/FormRecognizer/FormRecognizerQuickstart.js), que contiene los ejemplos de código de este inicio rápido.

Cree variables para el punto de conexión y la clave de Azure del recurso.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_creds)]

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Form Recognizer que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar su clave y punto de conexión en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**.
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

## <a name="object-model"></a>Modelo de objetos

Con Form Recognizer, puede crear dos tipos de cliente diferentes. El primero, `FormRecognizerClient`, se utiliza para consultar el servicio con los campos de formulario y el contenido reconocidos. El segundo, `FormTrainingClient`, se usa para crear y administrar modelos personalizados que se pueden emplear para mejorar el reconocimiento.

### <a name="formrecognizerclient"></a>FormRecognizerClient

`FormRecognizerClient` proporciona operaciones para:

* Reconocimiento de los campos de formulario y del contenido mediante el uso de modelos personalizados entrenados para analizar los formularios personalizados. Estos valores se devuelven en una colección de objetos `RecognizedForm`.
* El reconocimiento del contenido de los formularios, incluidas tablas, líneas y palabras, sin necesidad de entrenar un modelo. El contenido del formulario se devuelve en una colección de objetos `FormPage`.
* Reconocimiento de los campos comunes de recibos de EE. UU., tarjetas de presentación, facturas y documentos de identificación con un modelo entrenado previamente en el servicio Form Recognizer.

### <a name="formtrainingclient"></a>FormTrainingClient

`FormTrainingClient` proporciona operaciones para:

* El entrenamiento de modelos personalizados para analizar todos los campos y los valores que se encuentren en los formularios personalizados. Se devuelve un `CustomFormModel` que indica los tipos de formulario que el modelo analizará y los campos que se extraerán para cada tipo de formulario. _Consulte_ la [documentación del servicio sobre el entrenamiento de modelos sin etiquetar](#train-a-model-without-labels) para más información.
* El entrenamiento de modelos personalizados para analizar los campos y los valores concretos que especifique mediante la etiqueta de los formularios personalizados. Se devuelve un `CustomFormModel` que indica los campos que el modelo va a extraer, así como la precisión estimada para cada campo. Consulte la [documentación del servicio sobre el entrenamiento de modelos con etiqueta](#train-a-model-with-labels) para obtener una explicación más detallada de cómo aplicar etiquetas a un conjunto de datos de entrenamiento.
* La administración de los modelos creados en una cuenta.
* La copia de un modelo personalizado entre recursos de Form Recognizer.

> [!NOTE]
> Los modelos también se pueden entrenar mediante una interfaz gráfica de usuario, como la [herramienta de etiquetado de Form Recognizer](../../label-tool.md).

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Form Recognizer para JavaScript:

* [Autenticar el cliente](#authenticate-the-client)
* [Análisis de diseño](#analyze-layout)
* [Análisis de las confirmaciones de recepción](#analyze-receipts)
* [Análisis de tarjetas de presentación](#analyze-business-cards)
* [Análisis de facturas](#analyze-invoices)
* [Análisis de documentos de identidad](#analyze-identity-documents)
* [Entrenar un modelo personalizado](#train-a-custom-model)
* [Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)
* [Administración de modelos personalizados](#manage-custom-models)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Autentique un objeto de cliente mediante las variables de suscripción que ha definido. Usará un objeto `AzureKeyCredential`, de modo que, si es necesario, pueda actualizar la clave de API sin crear otros objetos de cliente. También creará un objeto de cliente de entrenamiento.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_auth)]

## <a name="get-assets-for-testing"></a>Obtención de recursos para pruebas

También tendrá que agregar referencias a las direcciones URL de los datos de entrenamiento y prueba.

* [!INCLUDE [get SAS URL](../sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Recuperación de la dirección URL de SAS":::
* Use las imágenes de envío y de recepción que se incluyen en los ejemplos siguientes (también disponibles en [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/assets)). O bien, siga los pasos anteriores para obtener la dirección URL de SAS de un documento individual en el almacenamiento de blobs.

## <a name="analyze-layout"></a>Análisis de diseño

Puede usar Form Recognizer para analizar tablas, líneas y palabras de los documentos sin necesidad de entrenar un modelo. Para más información sobre la extracción del diseño, consulte la [guía conceptual sobre diseño](../../concept-layout.md). Para analizar el contenido de un archivo de un URI determinado, use el método `beginRecognizeContentFromUrl`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_getcontent)]

> [!TIP]
> También puede obtener contenido de un archivo local. Consulte los métodos [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), como **beginRecognizeContent**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) para escenarios relacionados con imágenes locales.

### <a name="output"></a>Resultados

```console
Page 1: width 8.5 and height 11 with unit inch
cell [0,0] has text Invoice Number
cell [0,1] has text Invoice Date
cell [0,2] has text Invoice Due Date
cell [0,3] has text Charges
cell [0,5] has text VAT ID
cell [1,0] has text 34278587
cell [1,1] has text 6/18/2017
cell [1,2] has text 6/24/2017
cell [1,3] has text $56,651.49
cell [1,5] has text PT
```

## <a name="analyze-receipts"></a>Análisis de las confirmaciones de recepción

En esta sección se muestra cómo analizar y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente. Para más información sobre el análisis de recibos, consulte la [guía conceptual sobre recibos](../../concept-receipts.md).

Para analizar los recibos de un identificador URI, use el método `beginRecognizeReceiptsFromUrl`. En el código siguiente se procesa un recibo en el URI especificado y se imprimen los campos y valores principales en la consola.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_receipts)]

> [!TIP]
> También puede analizar imágenes de recibos locales. Consulte los métodos [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), como **beginRecognizeReceipts**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) para escenarios relacionados con imágenes locales.

### <a name="output"></a>Resultados

```console
status: notStarted
status: running
status: succeeded
First receipt:
  Receipt Type: 'Itemized', with confidence of 0.659
  Merchant Name: 'Contoso Contoso', with confidence of 0.516
  Transaction Date: 'Sun Jun 09 2019 17:00:00 GMT-0700 (Pacific Daylight Time)', with confidence of 0.985
    Item Name: '8GB RAM (Black)', with confidence of 0.916
    Item Name: 'SurfacePen', with confidence of 0.858
  Total: '1203.39', with confidence of 0.774
```

## <a name="analyze-business-cards"></a>Análisis de tarjetas de presentación

En esta sección se muestra cómo analizar y extraer campos comunes de tarjetas de presentación en inglés mediante un modelo entrenado previamente. Para más información acerca del análisis de tarjetas de presentación, consulte la [guía conceptual sobre tarjetas de presentación](../../concept-business-cards.md).

Para analizar tarjetas de presentación en una dirección URL, use el método `beginRecognizeBusinessCardsFromURL`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js" id="snippet_bc":::

> [!TIP]
> También puede analizar imágenes de tarjeta de presentación locales. Consulte los métodos de [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), como **beginRecognizeBusinessCards**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) para escenarios relacionados con imágenes locales.

## <a name="analyze-invoices"></a>Análisis de facturas

En esta sección se muestra cómo analizar y extraer campos comunes de facturas de compra mediante un modelo entrenado previamente. Para más información sobre el análisis de facturas, consulte la [guía conceptual sobre facturas](../../concept-invoices.md).

Para analizar facturas de una dirección URL, use el método `beginRecognizeInvoicesFromUrl`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js" id="snippet_invoice":::

> [!TIP]
> También puede analizar imágenes de tarjeta de presentación locales. Consulte los métodos [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), como **beginRecognizeInvoices**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) para escenarios relacionados con imágenes locales.

## <a name="analyze-identity-documents"></a>Análisis de documentos de identidad

En esta sección se muestra cómo analizar y extraer información clave de documentos de identificación emitidos por la administración pública (pasaportes de todo el mundo y permisos de conducir de EE. UU.) mediante el modelo de identificación precompilado de Form Recognizer. Para obtener más información sobre el análisis de facturas, consulte nuestra [guía conceptual del modelo de identificación precompilado](../../concept-identification-cards.md).

Para analizar documentos de identificación de una dirección URL, use el método `beginRecognizeIdDocumentsFromUrl`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js" id="snippet_id":::

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

En esta sección se muestra cómo entrenar un modelo con sus propios datos. Un modelo entrenado puede generar datos estructurados que incluyan las relaciones clave-valor del documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

> [!NOTE]
> También puede entrenar modelos con una interfaz gráfica de usuario, como la [herramienta de etiquetado de ejemplo de Form Recognizer](../../label-tool.md).

### <a name="train-a-model-without-labels"></a>Entrenamiento de un modelo sin etiquetas

Entrene modelos personalizados para analizar todos los campos y valores que se encuentran en los formularios personalizados sin etiquetar manualmente los documentos de entrenamiento.

La función siguiente entrena un modelo en un conjunto de documentos determinado e imprime el estado del modelo en la consola.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_train)]

### <a name="output"></a>Resultados

Esta es la salida para un modelo entrenado con los datos de entrenamiento disponibles en el [JavaScript de Python](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer). Esta salida de ejemplo se ha truncado para mejorar la legibilidad.

```console
training status: creating
training status: ready
Model ID: 9d893595-1690-4cf2-a4b1-fbac0fb11909
Status: ready
Training started on: Thu Aug 20 2020 20:27:26 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:27:37 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'field-0'
The model found field 'field-1'
The model found field 'field-2'
The model found field 'field-3'
The model found field 'field-4'
The model found field 'field-5'
The model found field 'field-6'
The model found field 'field-7'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors:
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors:
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors:
...
```

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

También puede entrenar modelos personalizados mediante el etiquetado manual de los documentos de entrenamiento. En algunos escenarios, el entrenamiento con etiquetas conduce a un mejor rendimiento. Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información con etiquetas especiales (`\<filename\>.pdf.labels.json`) en el contenedor de almacenamiento de blobs junto con los documentos para el entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Cuando los tenga, puede llamar al método `beginTraining` con el parámetro `uselabels` establecido en `true`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_trainlabels)]

### <a name="output"></a>Output

Esta es la salida para un modelo entrenado con los datos de entrenamiento disponibles en el [JavaScript de Python](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples). Esta salida de ejemplo se ha truncado para mejorar la legibilidad.

```console
training status: creating
training status: ready
Model ID: 789b1b37-4cc3-4e36-8665-9dde68618072
Status: ready
Training started on: Thu Aug 20 2020 20:30:37 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:30:43 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'CompanyAddress'
The model found field 'CompanyName'
The model found field 'CompanyPhoneNumber'
The model found field 'DatedAs'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
...
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

En esta sección, se muestra cómo extraer información de clave-valor y otro contenido de los tipos de formulario personalizados, con los modelos que ha entrenado con sus propios formularios.

> [!IMPORTANT]
> Para implementar este escenario, debe haber entrenado ya un modelo para que pueda pasar su identificador al método siguiente. Consulte la sección [Entrenamiento de un modelo](#train-a-model-without-labels).

Usará el método `beginRecognizeCustomFormsFromUrl`. El valor devuelto es una colección de objetos `RecognizedForm`, uno para cada página del documento enviado.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_analyze)]

> [!TIP]
> También puede analizar archivos locales. Consulte los métodos [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), como **beginRecognizeCustomForms**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) para escenarios relacionados con imágenes locales.

### <a name="output"></a>Resultados

```console
status: notStarted
status: succeeded
Forms:
custom:form, page range: [object Object]
Pages:
Page number: 1
Tables
cell (0,0) Invoice Number
cell (0,1) Invoice Date
cell (0,2) Invoice Due Date
cell (0,3) Charges
cell (0,5) VAT ID
cell (1,0) 34278587
cell (1,1) 6/18/2017
cell (1,2) 6/24/2017
cell (1,3) $56,651.49
cell (1,5) PT
Fields:
Field Merchant has value 'Invoice For:' with a confidence score of 0.116
Field CompanyPhoneNumber has value '$56,651.49' with a confidence score of 0.249
Field VendorName has value 'Charges' with a confidence score of 0.145
Field CompanyAddress has value '1 Redmond way Suite 6000 Redmond, WA' with a confidence score of 0.258
Field CompanyName has value 'PT' with a confidence score of 0.245
Field Website has value '99243' with a confidence score of 0.114
Field DatedAs has value 'undefined' with a confidence score of undefined
Field Email has value 'undefined' with a confidence score of undefined
Field PhoneNumber has value 'undefined' with a confidence score of undefined
Field PurchaseOrderNumber has value 'undefined' with a confidence score of undefined
Field Quantity has value 'undefined' with a confidence score of undefined
Field Signature has value 'undefined' with a confidence score of undefined
Field Subtotal has value 'undefined' with a confidence score of undefined
Field Tax has value 'undefined' with a confidence score of undefined
Field Total has value 'undefined' with a confidence score of undefined
```

## <a name="manage-custom-models"></a>Administración de modelos personalizados

En esta sección se muestra cómo administrar los modelos personalizados almacenados en su cuenta. Como ejemplo, el código siguiente realiza todas las tareas de administración del modelo en una única función.

### <a name="get-number-of-models"></a>Obtención del número de modelos

El siguiente bloque de código obtiene el número de modelos que hay actualmente en su cuenta.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_count)]

### <a name="get-list-of-models-in-account"></a>Obtención de una lista de modelos en la cuenta

El siguiente bloque de código proporciona una lista completa de los modelos disponibles en su cuenta, incluida información sobre cuándo se creó el modelo y su estado actual.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_list)]

### <a name="output"></a>Resultados

```console
model 0:
{
  modelId: '453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e',
  status: 'invalid',
  trainingStartedOn: 2020-08-20T22:28:52.000Z,
  trainingCompletedOn: 2020-08-20T22:28:53.000Z
}
model 1:
{
  modelId: '628739de-779c-473d-8214-d35c72d3d4f7',
  status: 'ready',
  trainingStartedOn: 2020-08-20T23:16:51.000Z,
  trainingCompletedOn: 2020-08-20T23:16:59.000Z
}
model 2:
{
  modelId: '789b1b37-4cc3-4e36-8665-9dde68618072',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:30:37.000Z,
  trainingCompletedOn: 2020-08-21T03:30:43.000Z
}
model 3:
{
  modelId: '9d893595-1690-4cf2-a4b1-fbac0fb11909',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:27:26.000Z,
  trainingCompletedOn: 2020-08-21T03:27:37.000Z
}
```

### <a name="get-list-of-model-ids-by-page"></a>Obtención de una lista de identificadores de modelo por página

Este bloque de código proporciona una lista paginada de modelos e identificadores de modelo.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_listpages)]

### <a name="output"></a>Resultados

```console
model 1: 453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
model 2: 628739de-779c-473d-8214-d35c72d3d4f7
model 3: 789b1b37-4cc3-4e36-8665-9dde68618072
```

### <a name="get-model-by-id"></a>Obtención del modelo en función del identificador

La siguiente función toma un identificador de modelo y obtiene el objeto de modelo correspondiente. Esta función no recibe una llamada de forma predeterminada.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_getmodel)]

### <a name="delete-a-model-from-the-resource-account"></a>Eliminación de un modelo de la cuenta de recursos

También puede eliminar un modelo de su cuenta haciendo referencia a su identificador. Esta función elimina el modelo con el identificador especificado. Esta función no recibe una llamada de forma predeterminada.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_delete)]

### <a name="output"></a>Output

```console
Model with id 789b1b37-4cc3-4e36-8665-9dde68618072 has been deleted
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="enable-logs"></a>Habilitamiento de registros

Puede establecer la siguiente variable de entorno para ver los registros de depuración cuando se usa esta biblioteca.

```console
export DEBUG=azure*
```

Para obtener instrucciones más detalladas sobre cómo habilitar los registros, vea los documentos del paquete [@azure/logger](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/core/logger).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se ha usado la biblioteca cliente de Form Recognizer para JavaScript con el fin de entrenar modelos y analizar formularios de diversas maneras. A continuación, obtenga sugerencias para crear un mejor conjunto de datos de entrenamiento y generar modelos más precisos.

> [!div class="nextstepaction"]
> [Creación de un conjunto de datos de aprendizaje](../../build-training-data-set.md)

## <a name="see-also"></a>Consulte también

* [¿Qué es Form Recognizer?](../../overview.md)
* El código de ejemplo de esta guía está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/FormRecognizer/FormRecognizerQuickstart.js).
