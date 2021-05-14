---
title: 'Inicio rápido: Biblioteca cliente de Form Recognizer para Python'
description: Use la biblioteca cliente de Form Recognizer para Python con el fin de crear una aplicación de procesamiento de formularios que extraiga pares clave-valor y datos de tabla de los documentos personalizados.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 04/09/2021
ms.author: lajanuar
ms.openlocfilehash: 606755333856f6dd97ab6c5158ac67f122a1237d
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107516466"
---
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->
> [!IMPORTANT]
>
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. Para más información, consulte la siguiente documentación de referencia.

[Documentación de referencia](/python/api/azure-ai-formrecognizer) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/azure/ai/formrecognizer) | [Paquete (PyPi)](https://pypi.org/project/azure-ai-formrecognizer/) | [Ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* [Python 3.x](https://www.python.org/)
  * La instalación de Python debe incluir [pip](https://pip.pypa.io/en/stable/). Puede comprobar si tiene pip instalado mediante la ejecución de `pip --version` en la línea de comandos. Para obtener pip, instale la versión más reciente de Python.
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Train** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451) (descargue y extraiga *sample_data.zip*).
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Después de instalar Python, puede instalar la versión más reciente de la biblioteca de cliente de Form Recognizer con:

#### <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/preview)

```console
pip install azure-ai-formrecognizer --pre
```

> [!NOTE]
> Form Recognizer 3.1.0b4 es la versión preliminar del SDK más reciente y refleja la _versión preliminar 3 de la versión 2.1 de la API._

#### <a name="v20"></a>[v2.0](#tab/ga)

```console
pip install azure-ai-formrecognizer
```

> [!NOTE]
> El SDK de Form Recognizer 3.0.0 refleja la versión 2.0 de la API.

---

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree una aplicación de Python en el IDE o editor que prefiera. A continuación, importe las bibliotecas siguientes.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_imports)]

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/FormRecognizerQuickstart.py), que contiene los ejemplos de código de este inicio rápido.

Cree variables para el punto de conexión y la clave de Azure del recurso.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_creds)]

## <a name="object-model"></a>Modelo de objetos

Con Form Recognizer, puede crear dos tipos de cliente diferentes. El primero, `form_recognizer_client`, se usa para realizar consultas en el servicio para reconocer contenido y campos del formulario. El segundo, `form_training_client`, se usa para crear y administrar modelos personalizados que se pueden usar para mejorar el reconocimiento. 

### <a name="formrecognizerclient"></a>FormRecognizerClient

`form_recognizer_client` proporciona operaciones para:

* Reconocimiento de los campos de formulario y del contenido mediante el uso de modelos personalizados entrenados para analizar los formularios personalizados.
* El reconocimiento del contenido de los formularios, incluidas tablas, líneas y palabras, sin necesidad de entrenar un modelo.
* El reconocimiento de los campos comunes de las recepciones mediante un modelo de recepción entrenado previamente en el servicio Form Recognizer.

### <a name="formtrainingclient"></a>FormTrainingClient

`form_training_client` proporciona operaciones para:

* El entrenamiento de modelos personalizados para analizar todos los campos y los valores que se encuentren en los formularios personalizados. Consulte la [documentación del servicio sobre el entrenamiento de modelos sin etiquetas](#train-a-model-without-labels) si desea obtener una explicación más detallada de la creación de un conjunto de datos de entrenamiento.
* El entrenamiento de modelos personalizados para analizar los campos y los valores concretos que especifique mediante la etiqueta de los formularios personalizados. Consulte la [documentación del servicio sobre el entrenamiento de modelos con etiqueta](#train-a-model-with-labels) para obtener una explicación más detallada de cómo aplicar etiquetas a un conjunto de datos de entrenamiento.
* La administración de los modelos creados en una cuenta.
* La copia de un modelo personalizado entre recursos de Form Recognizer.

> [!NOTE]
> Los modelos también se pueden entrenar mediante una interfaz gráfica de usuario, como la [herramienta de etiquetado de Form Recognizer](../../quickstarts/label-tool.md).

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código, se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Form Recognizer para Python:
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
#### <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/preview)

* [Autenticar el cliente](#authenticate-the-client)
* [Análisis de diseño](#analyze-layout)
* [Análisis de las confirmaciones de recepción](#analyze-receipts)
* [Análisis de tarjetas de presentación](#analyze-business-cards)
* [Análisis de facturas](#analyze-invoices)
* [Análisis de documentos de identidad](#analyze-identity-documents)
* [Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)
* [Entrenar un modelo personalizado](#train-a-custom-model)
* [Administrar modelos personalizados](#manage-your-custom-models)

#### <a name="v20"></a>[v2.0](#tab/ga)

* [Autenticar el cliente](#authenticate-the-client)
* [Análisis de diseño](#analyze-layout)
* [Análisis de las confirmaciones de recepción](#analyze-receipts)
* [Entrenar un modelo personalizado](#train-a-custom-model)
* [Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)
* [Administrar modelos personalizados](#manage-your-custom-models)

---

## <a name="authenticate-the-client"></a>Autenticar el cliente

Aquí, autenticará dos objetos de cliente mediante las variables de suscripción que definió anteriormente. Usará un objeto **AzureKeyCredential**, de modo que, si es necesario, pueda actualizar la clave de API sin crear otros objetos de cliente.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_auth)]

## <a name="get-assets-for-testing"></a>Obtención de recursos para pruebas

Es preciso que agregue referencias a las direcciones URL de los datos de entrenamiento y de prueba.

* [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Recuperación de la dirección URL de SAS":::

* Use las imágenes de envío y de recepción que se incluyen en los ejemplos siguientes (también disponibles en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms)). O bien, siga los pasos anteriores para obtener la dirección URL de SAS de un documento individual en el almacenamiento de blobs. 

> [!NOTE]
> Los fragmentos de código de esta guía usan formularios remotos con acceso mediante direcciones URL. En cambio, si quiere procesar documentos de formulario locales, consulte los métodos relacionados en la [documentación de referencia](/python/api/azure-ai-formrecognizer) y los [ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples).

## <a name="analyze-layout"></a>Análisis de diseño

Puede usar Form Recognizer para analizar tablas, líneas y palabras de los documentos sin necesidad de entrenar un modelo. Para más información sobre la extracción del diseño, consulte la [guía conceptual sobre diseño](../../concept-layout.md).

Para analizar el contenido de un archivo en una dirección URL determinada, use el método `begin_recognize_content_from_url`. El valor devuelto es una colección de objetos `FormPage`, uno para cada página del documento enviado. El código siguiente recorre en iteración estos objetos e imprime los pares clave-valor y los datos de tabla extraídos.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_getcontent)]

> [!TIP]
> También puede obtener contenido de imágenes locales. Consulte los métodos [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), como `begin_recognize_content`. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) para escenarios relacionados con imágenes locales.

### <a name="output"></a>Resultados

```console
Table found on page 1:
Cell text: Invoice Number
Location: [Point(x=0.5075, y=2.8088), Point(x=1.9061, y=2.8088), Point(x=1.9061, y=3.3219), Point(x=0.5075, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Date
Location: [Point(x=1.9061, y=2.8088), Point(x=3.3074, y=2.8088), Point(x=3.3074, y=3.3219), Point(x=1.9061, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Due Date
Location: [Point(x=3.3074, y=2.8088), Point(x=4.7074, y=2.8088), Point(x=4.7074, y=3.3219), Point(x=3.3074, y=3.3219)]
Confidence score: 1.0

Cell text: Charges
Location: [Point(x=4.7074, y=2.8088), Point(x=5.386, y=2.8088), Point(x=5.386, y=3.3219), Point(x=4.7074, y=3.3219)]
Confidence score: 1.0
...

```

## <a name="analyze-receipts"></a>Análisis de las confirmaciones de recepción

En esta sección se muestra cómo analizar y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente. Para más información sobre el análisis de recibos, consulte la [guía conceptual sobre recibos](../../concept-receipts.md). Para analizar recibos de una dirección URL, use el método `begin_recognize_receipts_from_url`.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_receipts)]

> [!TIP]
> También puede analizar imágenes de recibos locales. Consulte los métodos [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), como `begin_recognize_receipts`. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) para escenarios relacionados con imágenes locales.

### <a name="output"></a>Resultados

```console
ReceiptType: Itemized has confidence 0.659
MerchantName: Contoso Contoso has confidence 0.516
MerchantAddress: 123 Main Street Redmond, WA 98052 has confidence 0.986
MerchantPhoneNumber: None has confidence 0.99
TransactionDate: 2019-06-10 has confidence 0.985
TransactionTime: 13:59:00 has confidence 0.968
Receipt Items:
...Item #1
......Name: 8GB RAM (Black) has confidence 0.916
......TotalPrice: 999.0 has confidence 0.559
...Item #2
......Quantity: None has confidence 0.858
......Name: SurfacePen has confidence 0.858
......TotalPrice: 99.99 has confidence 0.386
Subtotal: 1098.99 has confidence 0.964
Tax: 104.4 has confidence 0.713
Total: 1203.39 has confidence 0.774
```

## <a name="analyze-business-cards"></a>Análisis de tarjetas de presentación

#### <a name="v21-preview"></a>[versión preliminar v2.1](#tab/preview)

En esta sección se muestra cómo analizar y extraer campos comunes de tarjetas de presentación inglesas mediante un modelo entrenado previamente. Para más información acerca del análisis de tarjetas de presentación, consulte la [guía conceptual sobre tarjetas de presentación](../../concept-business-cards.md). 

Para analizar tarjetas de presentación en una dirección URL, use el método `begin_recognize_business_cards_from_url`.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart-preview.py?name=snippet_bc)]

> [!TIP]
> También puede analizar imágenes de tarjeta de presentación locales. Consulte los métodos [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), como `begin_recognize_business_cards`. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) para escenarios relacionados con imágenes locales.

#### <a name="v20"></a>[v2.0](#tab/ga)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

---

## <a name="analyze-invoices"></a>Análisis de facturas

#### <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/preview)

En esta sección se muestra cómo analizar y extraer campos comunes de facturas de compra mediante un modelo entrenado previamente. Para más información sobre el análisis de facturas, consulte la [guía conceptual sobre facturas](../../concept-invoices.md). 

Para analizar facturas de una dirección URL, use el método `begin_recognize_invoices_from_url`.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart-preview.py?name=snippet_invoice)]

> [!TIP]
> También puede analizar imágenes de facturas locales. Consulte los métodos [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), como `begin_recognize_invoices`. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) para escenarios relacionados con imágenes locales.

#### <a name="v20"></a>[v2.0](#tab/ga)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

---

## <a name="analyze-identity-documents"></a>Análisis de documentos de identidad

#### <a name="v21-preview"></a>[versión preliminar v2.1](#tab/preview)

En esta sección se muestra cómo analizar y extraer información clave de documentos de identificación emitidos por la administración pública (pasaportes de todo el mundo y permisos de conducir de EE. UU.) mediante el modelo de identificación precompilado de Form Recognizer. Para obtener más información sobre el análisis de facturas, consulte nuestra [guía conceptual del modelo de identificación precompilado](../../concept-identification-cards.md).

Para analizar documentos de identificación de una dirección URL, use el método `begin_recognize_id_documents_from_url`.

:::code language="python" source="~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart-preview.py" id="snippet_id":::

> [!TIP]
> También puede analizar imágenes de documentos de identificación. _Consulte_ los métodos [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient?view=azure-python&preserve-view=true#methods), como `begin_recognize_id_documents`. _Consulte también_ el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) para escenarios relacionados con imágenes locales.

#### <a name="v20"></a>[v2.0](#tab/ga)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

---

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

En esta sección se muestra cómo entrenar un modelo con sus propios datos. Un modelo entrenado puede generar datos estructurados que incluyan las relaciones clave-valor del documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

> [!NOTE]
> También puede entrenar modelos con una interfaz gráfica de usuario, como la [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Entrenamiento de un modelo sin etiquetas

Entrene modelos personalizados para analizar todos los campos y valores que se encuentran en los formularios personalizados sin etiquetar manualmente los documentos de entrenamiento.

El código siguiente usa el cliente de entrenamiento con la función `begin_training` para entrenar un modelo en un conjunto de documentos determinado. El objeto devuelto `CustomFormModel` contiene información sobre los tipos de formulario que el modelo puede analizar y los campos que puede extraer de cada tipo de formulario. El siguiente bloque de código imprime esta información en la consola.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_train)]

### <a name="output"></a>Resultados

Esta es la salida para un modelo entrenado con los datos de entrenamiento disponibles en el [SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: 628739de-779c-473d-8214-d35c72d3d4f7
Status: ready
Training started on: 2020-08-20 23:16:51+00:00
Training completed on: 2020-08-20 23:16:59+00:00

Recognized fields:
The submodel with form type 'form-0' has recognized the following fields: Additional Notes:, Address:, Company Name:, Company Phone:, Dated As:, Details, Email:, Hero Limited, Name:, Phone:, Purchase Order, Purchase Order #:, Quantity, SUBTOTAL, Seattle, WA 93849 Phone:, Shipped From, Shipped To, TAX, TOTAL, Total, Unit Price, Vendor Name:, Website:
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

También puede entrenar modelos personalizados mediante el etiquetado manual de los documentos de entrenamiento. En algunos escenarios, el entrenamiento con etiquetas conduce a un mejor rendimiento. El valor de `CustomFormModel` devuelto indica los campos que el modelo puede extraer, junto con su precisión estimada en cada campo. El siguiente bloque de código imprime esta información en la consola.

> [!IMPORTANT]
> Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información con etiquetas especiales (`\<filename\>.pdf.labels.json`) en el contenedor de almacenamiento de blobs junto con los documentos para el entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Cuando los tenga, puede llamar a la función `begin_training` con el parámetro *use_training_labels* establecido en `true`.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_trainlabels)]

### <a name="output"></a>Output

Esta es la salida para un modelo entrenado con los datos de entrenamiento disponibles en el [SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91

Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00

Recognized fields:
The submodel with form type 'form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91' has recognized the following fields: CompanyAddress, CompanyName, CompanyPhoneNumber, DatedAs, Email, Merchant, PhoneNumber, PurchaseOrderNumber, Quantity, Signature, Subtotal, Tax, Total, VendorName, Website
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

En esta sección, se muestra cómo extraer información de clave-valor y otro contenido de los tipos de formulario personalizados, con los modelos que ha entrenado con sus propios formularios.

> [!IMPORTANT]
> Para implementar este escenario, debe haber entrenado ya un modelo para que pueda pasar su identificador al método siguiente. Consulte la sección [Entrenamiento de un modelo](#train-a-model-without-labels).

Usará el método `begin_recognize_custom_forms_from_url`. El valor devuelto es una colección de objetos `RecognizedForm`, uno para cada página del documento enviado. El código siguiente imprime los resultados del análisis en la consola. Imprime cada campo reconocido y el valor correspondiente, junto con una puntuación de confianza.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_analyze)]

> [!TIP]
> También puede analizar imágenes locales. Consulte los métodos [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), como `begin_recognize_custom_forms`. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) para escenarios relacionados con imágenes locales.

### <a name="output"></a>Resultados

Con el modelo del ejemplo anterior, se proporciona la siguiente salida.

```console
Form type: form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Field 'Merchant' has label 'Merchant' with value 'Invoice For:' and a confidence score of 0.116
Field 'CompanyAddress' has label 'CompanyAddress' with value '1 Redmond way Suite 6000 Redmond, WA' and a confidence score of 0.258
Field 'Website' has label 'Website' with value '99243' and a confidence score of 0.114
Field 'VendorName' has label 'VendorName' with value 'Charges' and a confidence score of 0.145
Field 'CompanyPhoneNumber' has label 'CompanyPhoneNumber' with value '$56,651.49' and a confidence score of 0.249
Field 'CompanyName' has label 'CompanyName' with value 'PT' and a confidence score of 0.245
Field 'DatedAs' has label 'DatedAs' with value 'None' and a confidence score of None
Field 'Email' has label 'Email' with value 'None' and a confidence score of None
Field 'PhoneNumber' has label 'PhoneNumber' with value 'None' and a confidence score of None
Field 'PurchaseOrderNumber' has label 'PurchaseOrderNumber' with value 'None' and a confidence score of None
Field 'Quantity' has label 'Quantity' with value 'None' and a confidence score of None
Field 'Signature' has label 'Signature' with value 'None' and a confidence score of None
Field 'Subtotal' has label 'Subtotal' with value 'None' and a confidence score of None
Field 'Tax' has label 'Tax' with value 'None' and a confidence score of None
Field 'Total' has label 'Total' with value 'None' and a confidence score of None
```


## <a name="manage-your-custom-models"></a>Administración de modelos personalizados

En esta sección se muestra cómo administrar los modelos personalizados almacenados en su cuenta.

### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Comprobación del número de modelos de la cuenta de recursos de FormRecognizer

El siguiente bloque de código comprueba cuántos modelos se han guardado en la cuenta de Form Recognizer y compara esta cifra con el límite de la cuenta.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_count)]


### <a name="output"></a>Resultados

```console
Our account has 5 custom models, and we can have at most 5000 custom models
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Enumeración de los modelos almacenados actualmente en la cuenta de recursos

El siguiente bloque de código enumera los modelos actuales de la cuenta e imprime los detalles en la consola. También se guarda una referencia al primer modelo.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_list)]


### <a name="output"></a>Resultados

Esta es una salida de ejemplo para la cuenta de prueba.

```console
We have models with the following ids:
453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
628739de-779c-473d-8214-d35c72d3d4f7
ae636292-0b14-4e26-81a7-a0bfcbaf7c91
b4b5df77-8538-4ffb-a996-f67158ecd305
c6309148-6b64-4fef-aea0-d39521452699
```

### <a name="get-a-specific-model-using-the-models-id"></a>Obtención de un modelo específico mediante el identificador del modelo

El siguiente bloque de código usa el identificador de modelo guardado en la sección anterior y lo emplea para recuperar detalles sobre el modelo.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_getmodel)]


### <a name="output"></a>Resultados

Esta es la salida de ejemplo para el modelo personalizado creado en el ejemplo anterior.

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00
```

### <a name="delete-a-model-from-the-resource-account"></a>Eliminación de un modelo de la cuenta de recursos

También puede eliminar un modelo de su cuenta haciendo referencia a su identificador. Este código elimina el modelo usado en la sección anterior.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_delete)]


## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `python` en el archivo de inicio rápido.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="general"></a>General

La biblioteca cliente de Form Recognizer producirá excepciones definidas en [Azure Core](https://aka.ms/azsdk-python-azure-core).

### <a name="logging"></a>Registro

Esta biblioteca usa la [biblioteca de registro estándar](https://docs.python.org/3/library/logging.html) para el registro. La información básica sobre las sesiones HTTP (direcciones URL, encabezados, etc.) se registra en el nivel de información.

El registro detallado del nivel de depuración, incluidos los cuerpos de solicitud y respuesta, y los encabezados no redactados, se puede habilitar en un cliente con el argumento de palabra clave `logging_enable`:

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerLogging.py?name=snippet_logging)]


Igualmente, `logging_enable` puede habilitar el registro detallado de una sola operación, aunque no esté habilitado para el cliente:

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerLogging.py?name=snippet_example)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se ha usado la biblioteca cliente de Python para Form Recognizer para entrenar modelos y analizar formularios de maneras diferentes. A continuación, obtenga sugerencias para crear un mejor conjunto de datos de entrenamiento y generar modelos más precisos.

> [!div class="nextstepaction"]
> [Creación de un conjunto de datos de aprendizaje](../../build-training-data-set.md)

* [¿Qué es Form Recognizer?](../../overview.md)
* El código de ejemplo de esta guía está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/FormRecognizerQuickstart.py).
