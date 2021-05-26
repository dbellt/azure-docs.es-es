---
title: 'Inicio rápido: Etiquetado de formularios, entrenamiento de un modelo y análisis de formularios mediante la herramienta de etiquetado de ejemplo: Form Recognizer'
titleSuffix: Azure Applied AI Services
description: En este inicio rápido, utilizará la herramienta de etiquetado de ejemplo Form Recognizer para etiquetar manualmente documentos de formularios. A continuación, entrenará un modelo personalizado de procesamiento de documentos con los documentos etiquetados y lo empleará para extraer pares clave-valor.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: quickstart
ms.date: 05/14/2021
ms.author: lajanuar
ms.custom: cog-serv-seo-may-2021
keywords: procesamiento de documentos
ms.openlocfilehash: af803b92945be059f604a3890a90d3aa36e5781e
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110475160"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->
<!-- markdownlint-disable MD029 -->
# <a name="get-started-with-form-recognizer"></a>Introducción a Form Recognizer

Comience a usar Form Recognizer con su herramienta de ejemplo correspondiente. Azure Form Recognizer es un servicio cognitivo que le permite crear software de procesamiento de datos automatizado mediante tecnología de aprendizaje automático. Identifique y extraiga texto, pares clave-valor, marcas de selección, datos de tabla y mucho más de los documentos de formulario; el servicio genera datos estructurados que incluyen las relaciones en el archivo original. Puede usar Form Recognizer mediante la herramienta de ejemplo, la API REST o el SDK. Siga estos pasos para probar Form Recognizer mediante la herramienta de ejemplo.

Use Form Recognizer para:

* Análisis de diseño
* Análisis con un modelo precompilado (facturas, recibos, documentos de identificación)
* Entrenamiento y análisis de un formulario personalizado

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, debe cumplir los siguientes requisitos:

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource </a> en Azure Portal para obtener la clave y el punto de conexión.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* (Opcional) Descargue y descomprima los documentos de ejemplo de inicio rápido siguientes

## <a name="create-a-form-recognizer-resource"></a>Creación de un recurso de Form Recognizer

[!INCLUDE [create resource](../includes/create-resource.md)]

## <a name="analyze-layout"></a>Análisis de diseño

Extraiga texto, tablas, marcas de selección e información de la estructura de un documento.

1. Ir a [Form Recognizer Sample Tool](https://fott-2-1.azurewebsites.net/) (Herramienta de ejemplo de Form Recognizer)
2. En la página principal de la herramienta de ejemplo, seleccione "use layout to get text, tables and selection marks" (usar el diseño para obtener texto, tablas y marcas de selección).

     :::image type="content" source="../media/label-tool/layout-1.jpg" alt-text="Configuración de conexión para la herramienta Form Recognizer de diseño.":::

3. Reemplace {need Endpoint} por el punto de conexión que obtuvo con la suscripción de Form Recognizer.

4. Reemplace {need apiKey} por la clave de suscripción que obtuvo del recurso de Form Recognizer.

    :::image type="content" source="../media/label-tool/layout-2.jpg" alt-text="Configuración de conexión para la herramienta Form Recognizer de diseño.":::

5. Seleccione la dirección URL de origen, pegue la dirección URL siguiente del documento https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/layout-page-001.jpg de ejemplo, haga clic en el botón Capturar.

1. Haga clic en "Run Layout" (Ejecutar diseño). La herramienta de etiquetado de ejemplo de Form Recognizer llamará a la API de análisis de diseño y analizará el documento.

1. Vea los resultados: vea el texto resaltado que se ha extraído, así como las marcas de selección y las tablas detectadas.

    :::image type="content" source="../media/label-tool/layout-3.jpg" alt-text="Configuración de conexión para la herramienta Form Recognizer.":::

1. Descargue el archivo de salida JSON para ver los resultados detallados del diseño.
     * El nodo "readResults" contiene cada línea de texto con su posición de cuadro de límite correspondiente en la página.
     * El nodo "selectionMarks" muestra todas las marcas de selección (casilla, botón de opción) y si su estado es "seleccionado" o "no seleccionado".
     * En la sección "pageResults" se incluyen las tablas extraídas. Para cada tabla, se extraen el texto, el índice de filas y columnas, la expansión de filas y columnas, el rectángulo de selección, etc.

## <a name="analyze-using-a-prebuilt-model-invoices-receipts-ids-"></a>Análisis con un modelo precompilado (facturas, recibos, identificadores...)

Extraiga texto, tablas y pares clave-valor de las facturas, recibos de ventas, documentos de identidad o tarjetas de presentación mediante un modelo precompilado de Form Recognizer.

1. Ir a [Form Recognizer Sample Tool](https://fott-2-1.azurewebsites.net/) (Herramienta de ejemplo de Form Recognizer)
2. En la página principal de la herramienta de ejemplo, seleccione “use prebuilt model to get data” (usar un modelo precompilado para obtener datos).

    :::image type="content" source="../media/label-tool/prebuilt-1.jpg" alt-text="Análisis de resultados del diseño de Form Recognizer":::

3. Selección de la dirección URL de origen

4. Elija el archivo que quiere analizar entre las opciones siguientes:

    * La dirección URL de una imagen de un recibo. Puede usar una [factura de ejemplo](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/forms/Invoice_1.pdf) para este inicio rápido.
    * Una dirección URL para la imagen de un recibo. Puede usar un [documento de identificación de ejemplo](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/id-us-driver-license-wa.jpg) para este inicio rápido.
    * Una dirección URL para la imagen de un recibo. Puede usar la [imagen de un recibo de ejemplo](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/contoso-allinone.jpg) para este inicio rápido.
    * La dirección URL de una imagen de una tarjeta de presentación. Puede usar la [imagen de una tarjeta de presentación de ejemplo](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/business_cards/business-card-english.jpg) para este inicio rápido.

5. Reemplace {need Endpoint} por el punto de conexión que obtuvo con la suscripción de Form Recognizer.

6. Reemplace {need apiKey} por la clave de suscripción que obtuvo del recurso de Form Recognizer.

    :::image type="content" source="../media/label-tool/prebuilt-3.jpg" alt-text="Configuración de conexión para la herramienta Form Recognizer precompilado.":::

7. Seleccione el tipo de formulario que quiere analizar (factura, recibo, tarjetas de presentación o identificador) en función del tipo de documento que quiera analizar y seleccionar.

8. Haga clic en "Run analysis" (Ejecutar análisis). La herramienta de etiquetado de ejemplo de Form Recognizer llamará a la API de análisis precompilado y analizará el documento.
9. Vea los resultados: vea los pares clave-valor extraídos, los elementos de línea, el texto resaltado extraído y las tablas detectadas.

    :::image type="content" source="../media/label-tool/prebuilt-2.jpg" alt-text="Resultados de análisis de la factura precompilada de Form Recognizer":::

10. Descargue el archivo de salida JSON para ver los resultados detallados.

    * El nodo "readResults" contiene cada línea de texto con su posición de cuadro de límite correspondiente en la página.
    * El nodo "selectionMarks" muestra todas las marcas de selección (casilla, botón de opción) y si su estado es "seleccionado" o "no seleccionado".
    * En la sección "pageResults" se incluyen las tablas extraídas. Para cada tabla, se extraen el texto, el índice de filas y columnas, la expansión de filas y columnas, el rectángulo de selección, etc.
    * El campo "documentResults" contiene información de pares clave-valor y de elementos de línea para las partes más importantes del documento.

## <a name="train--analyze-a-custom-form"></a>Entrenamiento y análisis de un formulario personalizado

Entrene un modelo de formulario personalizado adaptado a los documentos. Extraiga texto, tablas, marcas de selección y pares clave-valor de los documentos con Form Recognizer personalizado.

### <a name="prerequisites-for-training-a-custom-form-model"></a>Requisitos previos para entrenar un modelo de formulario personalizado

* Un contenedor de blobs de Azure Storage que contenga un conjunto de datos de entrenamiento. En primer lugar, asegúrese de que todos los documentos de entrenamiento tienen el mismo formato. Si tiene formularios en varios formatos, organícelos en subcarpetas basadas en un formato común. En este inicio rápido puede usar los archivos de la carpeta Train del [conjunto de datos de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample_data_without_labels.zip) (descargue y extraiga sample_data.zip).
* Configure el uso compartido de recursos entre dominios (CORS) en el blob de Azure Storage Habilite CORS en la cuenta de almacenamiento. Seleccione la cuenta de almacenamiento en Azure Portal y elija la pestaña **CORS** del panel izquierdo. En la línea inferior, rellene los valores siguientes. Seleccione **Guardar** en la parte superior. </br></br>

  * Orígenes permitidos = *
  * Métodos permitidos = \[seleccionar todos\]
  * Encabezados permitidos = *
  * Encabezados expuestos = *
  * Antigüedad máxima = 200

> [!div class="mx-imgBorder"]
> ![Configuración de CORS en Azure Portal](../media/label-tool/cors-setup.png)

## <a name="train-a-custom-form-model"></a>Entrenar un modelo de formulario personalizado

1. Ir a [Form Recognizer Sample Tool](https://fott-2-1.azurewebsites.net/) (Herramienta de ejemplo de Form Recognizer)

1. En la página principal de la herramienta de ejemplo, seleccione "use custom form to train a model with labels and get key value pairs" (usar un formulario personalizado para entrenar un modelo con etiquetas y obtener pares clave-valor).

    :::image type="content" source="../media/label-tool/custom-1.jpg" alt-text="Entrene un modelo personalizado.":::

2. Selección de “Nuevo proyecto”

#### <a name="create-a-new-project"></a>Creación de un proyecto

Configure los valores del proyecto y rellene los campos con los valores siguientes:

* **Nombre para mostrar**: el nombre para mostrar del proyecto
* **Token de seguridad**: algunos valores de configuración del proyecto pueden incluir valores confidenciales, como claves de API u otros secretos compartidos. Cada proyecto generará un token de seguridad que se puede usar para cifrar o descifrar los valores de configuración confidenciales del proyecto. Puede buscar los tokens de seguridad en la configuración de la aplicación. Para ello, seleccione el icono de engranaje situado en la esquina inferior de la barra de navegación izquierda.

* **Conexión de origen**: la herramienta de etiquetado de ejemplo se conecta a un origen (los formularios originales cargados) y a un destino (las etiquetas creadas y los datos de salida). Las conexiones se pueden configurar y compartir entre proyectos. Usan un modelo extensible de proveedores, por lo que puede agregar fácilmente nuevos proveedores de origen y destino. Cree una conexión y haga clic en el botón **Agregar conexión**. Rellene los campos con los siguientes valores:
   * **Nombre para mostrar**: el nombre para mostrar de la conexión.
   * **Descripción**: la descripción del proyecto.
   * **Dirección URL de SAS**: la dirección URL de la firma de acceso compartido (SAS) del contenedor de Azure Blob Storage.

   [!INCLUDE [get SAS URL](../includes/sas-instructions.md)]

   :::image type="content" source="../media/quickstarts/get-sas-url.png" alt-text="Ubicación de SAS.":::

* **Ruta de acceso de la carpeta** (opcional): si los formularios de origen se encuentran en una carpeta en el contenedor de blobs, especifique aquí el nombre de la carpeta.
* **URI de servicio del servicio Form Recognizer**: la dirección URL del punto de conexión de Form Recognizer.
* **Clave de API**: la clave de suscripción de Form Recognizer.
* **Descripción** (opcional): descripción del proyecto.

    :::image type="content" source="../media/label-tool/connections.png"  alt-text="Configuración de conexión":::

#### <a name="label-your-forms"></a>Etiquetado de formularios

  :::image type="content" source="../media/label-tool/new-project.png"  alt-text="Página del nuevo proyecto":::

Al crear o abrir un proyecto, se abrirá la ventana principal del editor de etiquetas. El editor de etiquetas consta de tres partes:

* Un panel de vista previa de tamaño variable que contiene una lista desplazable de formularios de la conexión de origen.
* El panel principal del editor que permite aplicar etiquetas.
* El panel del editor de etiquetas que permite a los usuarios modificar, bloquear, reordenar y eliminar etiquetas.

##### <a name="identify-text-and-tables"></a>Identificación de textos y tablas

Seleccione **Run OCR on all files** (Ejecutar OCR en todos los archivos) en el panel izquierdo para obtener la información de diseño del texto de cada documento. La herramienta de etiquetado dibujará los cuadros de límite alrededor de cada elemento de texto.

También mostrará las tablas que se hayan extraído automáticamente. Seleccione el icono de tabla o cuadrícula en la parte izquierda del documento para ver la tabla extraída. En esta guía de inicio rápido, dado que el contenido de la tabla se extrae automáticamente, este no se etiquetará, sino que se basará en la extracción automatizada.

  :::image type="content" source="../media/label-tool/table-extraction.png" alt-text="Visualización de tablas mediante la herramienta de etiquetado de ejemplo.":::

##### <a name="apply-labels-to-text"></a>Aplicación de etiquetas a texto

A continuación, creará etiquetas y las aplicará a los elementos de texto que quiere que analice el modelo. Tenga en cuenta que el conjunto de datos de etiquetas de ejemplo incluye campos etiquetados; agregaremos otro campo.

1. En primer lugar, use el panel del editor de etiquetas para crear una etiqueta que quiera identificar.
   1. Seleccione **+** para crear una etiqueta.
   1. Escriba el nombre de la etiqueta. Agregar una etiqueta “Total”
   1. Presione Entrar para guardar la etiqueta.
1. En el editor principal, seleccione el valor total en los elementos de texto resaltados.
1. Seleccione la etiqueta Total que quiere aplicar al valor o presione la tecla correspondiente del teclado. Las teclas numéricas se asignan como teclas de acceso rápido para las diez primeras etiquetas. Puede volver a ordenar las etiquetas con los iconos de flecha arriba y abajo del panel del editor de etiquetas.

    > [!Tip]
    > Tenga en cuenta las siguientes sugerencias cuando vaya a etiquetar los formularios:
    >
    > * Solo se puede aplicar una etiqueta a cada elemento de texto seleccionado.
    > * Cada etiqueta solo se puede aplicar una vez por página. Si un valor aparece varias veces en el mismo formulario, cree etiquetas diferentes para cada instancia. Por ejemplo, "factura n.º 1", "factura n.º 2", etc.
    > * Las etiquetas no pueden abarcar varias páginas.
    > * Etiquete los valores tal como aparecen en el formulario; no intente dividir un valor en dos partes con dos etiquetas diferentes. Por ejemplo, un campo de dirección debe etiquetarse con una sola etiqueta incluso si abarca varias líneas.
    > * No incluya claves en los campos etiquetados, solo los valores.
    > * Los datos de la tabla deben detectarse automáticamente y estarán disponibles en el archivo JSON de salida final, en la sección “pageResults”. Sin embargo, si el modelo no detecta todos los datos de la tabla, también puede etiquetar y entrenar un modelo para detectar tablas. Consulte el documento sobre cómo entrenar y etiquetar << ruta al proceso >>
    > * Use los botones situados a la derecha de **+** para buscar, reordenar y eliminar las etiquetas, así como cambiarles el nombre.
    > * Para quitar una etiqueta aplicada sin eliminar la etiqueta en sí, seleccione el rectángulo etiquetado en la vista de documento y presione la tecla Supr.
    >

Siga los pasos anteriores para etiquetar en los cinco formularios del conjunto de datos de ejemplo.

  :::image type="content" source="../media/label-tool/custom-1.jpg" alt-text="Etiquete los ejemplos.":::

#### <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

Elija el icono Train (Entrenar) en el panel izquierdo para abrir la página de entrenamiento. A continuación, seleccione el botón **Train** (Entrenar) para empezar a entrenar el modelo. Una vez completado el proceso de entrenamiento, verá la siguiente información:

* **Id. del modelo**: el identificador del modelo que se ha creado y entrenado. Cada llamada de entrenamiento crea un nuevo modelo con su propio identificador. Copie esta cadena en una ubicación segura; la necesitará si desea realizar llamadas de predicción mediante la [API REST](./client-library.md?pivots=programming-language-rest-api) o la [biblioteca cliente](./client-library.md).
* **Precisión media**: el promedio de precisión del modelo. Puede mejorar la precisión del modelo si etiqueta más formularios y vuelve a realizar el entrenamiento para crear otro modelo. Se recomienda empezar por etiquetar cinco formularios y realizar un análisis y pruebas de los resultados; después, si es necesario, agregar más formularios.
* La lista de etiquetas y la precisión estimada por etiqueta.

    :::image type="content" source="../media/label-tool/custom-2.jpg" alt-text="Vista de entrenamiento.":::

#### <a name="analyze-a-custom-form"></a>Análisis de un formulario personalizado

Seleccione el icono de análisis (bombilla) de la izquierda para probar el modelo. Seleccione el origen "Archivo local". Busque un archivo en el conjunto de datos de ejemplo que descomprimió en la carpeta de prueba y selecciónelo. A continuación, elija el botón **Ejecutar análisis** para obtener las predicciones de pares clave-valor, texto y tablas del formulario. La herramienta aplicará etiquetas en los cuadros de límite e informará de la confianza de cada etiqueta.

:::image type="content" source="../media/label-tool/custom-3.jpg" alt-text="Herramienta de vista de entrenamiento.":::

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la herramienta de ejemplo Form Recognizer para probar cómo diseñar, precompilar y entrenar un modelo personalizado, además de analizar un formulario personalizado con datos etiquetados manualmente. Ahora puede probar la API REST o el SDK de la biblioteca cliente para usar Form Recognizer.

> [!div class="nextstepaction"]
> [Explorar el inicio rápido de uso de la API REST o el SDK de la biblioteca cliente de Form Recognizer](client-library.md)
