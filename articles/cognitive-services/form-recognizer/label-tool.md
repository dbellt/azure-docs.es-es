---
title: 'Procedimientos para: analizar documentos, etiquetar formularios, entrenar un modelo y analizar formularios con Form Recognizer'
titleSuffix: Azure Applied AI Services
description: En este proceso, usará la herramienta de ejemplo Form Recognizer para analizar documentos, facturas, recibos, etc. Etiquete y cree un modelo personalizado para extraer texto, tablas, marcas de selección, estructura y pares de clave/valor de documentos.
author: laujan
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: how-to
ms.date: 05/11/2021
ms.author: lajanuar
ms.custom: cog-serv-seo-aug-2020
keywords: procesamiento de documentos
ms.openlocfilehash: 8e5c297ced429137389d1c6314b47ef06f784b11
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112081936"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->
# <a name="train-a-custom-model-using-the-sample-labeling-tool"></a>Entrenamiento de un modelo personalizado mediante la herramienta de etiquetado de ejemplo

En este artículo, usará la API REST de Form Recognizer con la herramienta de etiquetado de ejemplo para entrenar un modelo personalizado de procesamiento de documentos con datos etiquetados manualmente. Consulte la sección [Entrenamiento con etiquetas](overview.md#train-with-labels) de la introducción para más información sobre el entrenamiento con Form Recognizer.

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Azure-Form-Recognizer/player]

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, debe cumplir los siguientes requisitos:

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource </a> en Azure Portal para obtener la clave y el punto de conexión. Tras su implementación, seleccione **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Un conjunto de al menos seis formularios del mismo tipo. Usará estos datos para entrenar el modelo y probar un formulario. En este inicio rápido, puede usar un [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451) (descargue y extraiga *sample_data.zip*). Cargue los archivos de entrenamiento en la raíz de un contenedor de almacenamiento de blobs de una cuenta de Azure Storage de nivel de rendimiento estándar.

## <a name="create-a-form-recognizer-resource"></a>Creación de un recurso de Form Recognizer

[!INCLUDE [create resource](includes/create-resource.md)]

## <a name="try-it-out"></a>Prueba

Para probar la herramienta de etiquetado de ejemplo de Form Recognizer en línea, vaya al [sitio web de FOTT](https://fott-2-1.azurewebsites.net/).

### <a name="v21"></a>[v2.1](#tab/v2-1)

> [!div class="nextstepaction"]
> [Uso de modelos precompilados](https://fott-2.1.azurewebsites.net/)

### <a name="v20"></a>[v2.0](#tab/v2-0)

> [!div class="nextstepaction"]
> [Uso de modelos precompilados](https://fott.azurewebsites.net/)

---

Necesitará una suscripción de Azure ([cree una gratis](https://azure.microsoft.com/free/cognitive-services)) y un punto de conexión de [recursos de Form Recognizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer), así como la clave para probar el servicio Form Recognizer.

## <a name="set-up-the-sample-labeling-tool"></a>Configuración de la herramienta de etiquetado de ejemplo

Usará el motor de Docker para ejecutar la herramienta de etiquetado de ejemplo. Siga estos pasos para configurar el contenedor de Docker. Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).

> [!TIP]
> OCR Form Labeling Tool también está disponible como proyecto de código abierto en GitHub. La herramienta es una aplicación web en TypeScript compilada mediante React + Redux. Para más información o para realizar una contribución, consulte el repositorio [OCR Form Labeling Tool](https://github.com/microsoft/OCR-Form-Tools/blob/master/README.md#run-as-web-application). Para probar la herramienta en línea, vaya al [sitio web de FOTT](https://fott.azurewebsites.net/).

1. En primer lugar, instale Docker en un equipo host. En esta guía se muestra cómo usar el equipo local como un host. Si desea usar un servicio de hospedaje de Docker en Azure, consulte la guía paso a paso [Implementación de la herramienta de etiquetado de ejemplo](deploy-label-tool.md).

   El equipo host debe cumplir los siguientes requisitos de hardware:

    | Contenedor | Mínima | Recomendado|
    |:--|:--|:--|
    |Herramienta de etiquetado de ejemplo|2 núcleos, 4 GB de memoria|4 núcleos, 8 GB de memoria|

   Instale Docker en la máquina siguiendo las instrucciones adecuadas para su sistema operativo:

   * [Windows](https://docs.docker.com/docker-for-windows/)
   * [macOS](https://docs.docker.com/docker-for-mac/)
   * [Linux](https://docs.docker.com/install/)

1. Obtenga el contenedor de la herramienta de etiquetado de ejemplo con el comando `docker pull`.

### <a name="v21"></a>[v2.1](#tab/v2-1)

```console
 docker pull mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:latest-2.1
```

### <a name="v20"></a>[v2.0](#tab/v2-0)

```console
docker pull mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool
```

---
</br>
  3. Ahora ya está listo para ejecutar el contenedor con `docker run`.

### <a name="v21"></a>[v2.1](#tab/v2-1)

```console
 docker run -it -p 3000:80 mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:latest-2.1 eula=accept
```

### <a name="v20"></a>[v2.0](#tab/v2-0)

```console
docker run -it -p 3000:80 mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool eula=accept
```

---

   Este comando hará que la herramienta de etiquetado de ejemplo esté disponible mediante un explorador web. Ir a `http://localhost:3000`.

> [!NOTE]
> También puede etiquetar documentos y entrenar modelos mediante la API REST de Form Recognizer. Para entrenar y analizar con la API REST, consulte [Entrenamiento con etiquetas mediante la API REST y Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-labeled-data.md).

## <a name="set-up-input-data"></a>Configuración de datos de entrada

En primer lugar, asegúrese de que todos los documentos de entrenamiento tienen el mismo formato. Si tiene formularios en varios formatos, organícelos en subcarpetas basadas en un formato común. Al entrenar, deberá dirigir la API a una subcarpeta.

### <a name="configure-cross-domain-resource-sharing-cors"></a>Configuración del uso compartido de recursos entre dominios (CORS)

Habilite CORS en la cuenta de almacenamiento. Seleccione la cuenta de almacenamiento en Azure Portal y elija la pestaña **CORS** del panel izquierdo. En la línea inferior, rellene los valores siguientes. Seleccione **Guardar** en la parte superior.

* Orígenes permitidos = *
* Métodos permitidos = \[seleccionar todos\]
* Encabezados permitidos = *
* Encabezados expuestos = *
* Antigüedad máxima = 200

> [!div class="mx-imgBorder"]
> ![Configuración de CORS en Azure Portal](media/label-tool/cors-setup.png)

## <a name="connect-to-the-sample-labeling-tool"></a>Conexión a la herramienta de etiquetado de ejemplo

 La herramienta de etiquetado de ejemplo se conecta a un origen (los formularios originales que cargó) y a un destino (las etiquetas creadas y los datos de salida).

Las conexiones se pueden configurar y compartir entre proyectos. Usan un modelo extensible de proveedores, por lo que puede agregar fácilmente nuevos proveedores de origen y destino.

Para crear una conexión, seleccione el icono **New Connections** (Nuevas conexiones) (un enchufe) en la barra de navegación izquierda.

Rellene los campos con los siguientes valores:

* **Nombre para mostrar**: el nombre para mostrar de la conexión.
* **Descripción**: la descripción del proyecto.
* **Dirección URL de SAS**: la dirección URL de la firma de acceso compartido (SAS) del contenedor de Azure Blob Storage. [!INCLUDE [get SAS URL](includes/sas-instructions.md)]

   :::image type="content" source="media/quickstarts/get-sas-url.png" alt-text="Recuperación de la dirección URL de SAS":::

:::image type="content" source="media/label-tool/connections.png" alt-text="Configuración de la conexión de la herramienta de etiquetado de ejemplo.":::

## <a name="create-a-new-project"></a>Creación de un nuevo proyecto

En la herramienta de etiquetado de ejemplo, los proyectos almacenan las configuraciones y los valores. Cree un nuevo proyecto y rellene los campos con los siguientes valores:

* **Nombre para mostrar**: el nombre para mostrar del proyecto
* **Token de seguridad**: algunos valores de configuración del proyecto pueden incluir valores confidenciales, como claves de API u otros secretos compartidos. Cada proyecto generará un token de seguridad que se puede usar para cifrar o descifrar los valores de configuración confidenciales del proyecto. Puede buscar los tokens de seguridad en la configuración de la aplicación. Para ello, seleccione el icono de engranaje situado en la esquina inferior de la barra de navegación izquierda.
* **Conexión de origen**: la conexión de Azure Blob Storage que creó en el paso anterior que le gustaría usar para este proyecto.
* **Ruta de acceso a la carpeta** (opcional): si los formularios de origen se encuentran en una carpeta en el contenedor de blobs, especifique aquí el nombre de la carpeta.
* **URI de servicio del servicio Form Recognizer**: la dirección URL del punto de conexión de Form Recognizer.
* **Clave de API**: la clave de suscripción de Form Recognizer.
* **Descripción** (opcional): descripción del proyecto.

:::image type="content" source="media/label-tool/new-project.png" alt-text="Página Nuevo proyecto en la herramienta de etiquetado de ejemplo.":::

## <a name="label-your-forms"></a>Etiquetado de formularios

Al crear o abrir un proyecto, se abrirá la ventana principal del editor de etiquetas. El editor de etiquetas consta de tres partes:

* Un panel de vista previa de tamaño variable que contiene una lista desplazable de formularios de la conexión de origen.
* El panel principal del editor que permite aplicar etiquetas.
* El panel del editor de etiquetas que permite a los usuarios modificar, bloquear, reordenar y eliminar etiquetas.

### <a name="identify-text-and-tables"></a>Identificación de textos y tablas 

Seleccione **Run OCR on all files** (Ejecutar OCR en todos los archivos) en el panel izquierdo para obtener la información de diseño del texto de cada documento. La herramienta de etiquetado dibujará los cuadros de límite alrededor de cada elemento de texto.

También mostrará las tablas que se hayan extraído automáticamente. Seleccione el icono de tabla o cuadrícula en la parte izquierda del documento para ver la tabla extraída. En esta guía de inicio rápido, dado que el contenido de la tabla se extrae automáticamente, este no se etiquetará, sino que se basará en la extracción automatizada.

:::image type="content" source="media/label-tool/table-extraction.png" alt-text="Visualización de tablas mediante la herramienta de etiquetado de ejemplo.":::

En la versión 2.1, si el documento de entrenamiento no tiene un valor rellenado, puede dibujar un cuadro para este. Use **Draw region** (Dibujar región) en la esquina superior izquierda de la ventana para hacer que la región se pueda etiquetar.

### <a name="apply-labels-to-text"></a>Aplicación de etiquetas a texto

A continuación, creará etiquetas y las aplicará a los elementos de texto que desea que analice el modelo.

### <a name="v21"></a>[v2.1](#tab/v2-1)  

1. Primero, use el panel del editor de etiquetas para crear las etiquetas que le gustaría identificar.
   1. Seleccione **+** para crear una etiqueta.
   1. Escriba el nombre de la etiqueta.
   1. Presione Entrar para guardar la etiqueta.
1. En el editor principal, seleccione las palabras de los elementos de texto resaltados o una región en la que dibujara.
1. Seleccione la etiqueta que desea aplicar o presione la tecla correspondiente del teclado. Las teclas numéricas se asignan como teclas de acceso rápido para las diez primeras etiquetas. Puede volver a ordenar las etiquetas con los iconos de flecha arriba y abajo del panel del editor de etiquetas.
    > [!Tip]
    > Tenga en cuenta las siguientes sugerencias cuando vaya a etiquetar los formularios:
    >
    > * Solo se puede aplicar una etiqueta a cada elemento de texto seleccionado.
    > * Cada etiqueta solo se puede aplicar una vez por página. Si un valor aparece varias veces en el mismo formulario, cree etiquetas diferentes para cada instancia. Por ejemplo, "factura n.º 1", "factura n.º 2", etc.
    > * Las etiquetas no pueden abarcar varias páginas.
    > * Etiquete los valores tal como aparecen en el formulario; no intente dividir un valor en dos partes con dos etiquetas diferentes. Por ejemplo, un campo de dirección debe etiquetarse con una sola etiqueta incluso si abarca varias líneas.
    > * No incluya claves en los campos etiquetados, solo los valores.
    > * Los datos de la tabla se deben detectar automáticamente y estarán disponibles en el archivo JSON de salida final. Sin embargo, si el modelo no detecta todos los datos de la tabla, también puede etiquetar manualmente estos campos. Etiquete cada celda de la tabla con una etiqueta diferente. Si los formularios tienen tablas con un número variable de filas, asegúrese de etiquetar al menos un formulario con la tabla más grande posible.
    > * Use los botones situados a la derecha de **+** para buscar, reordenar y eliminar las etiquetas, así como cambiarles el nombre.
    > * Para quitar una etiqueta aplicada sin eliminar la etiqueta en sí, seleccione el rectángulo etiquetado en la vista de documento y presione la tecla Supr.
    >

### <a name="v20"></a>[v2.0](#tab/v2-0)

1. Primero, use el panel del editor de etiquetas para crear las etiquetas que le gustaría identificar.
   1. Seleccione **+** para crear una etiqueta.
   1. Escriba el nombre de la etiqueta.
   1. Presione Entrar para guardar la etiqueta.
1. En el editor principal, seleccione las palabras en los elementos de texto resaltados.
1. Seleccione la etiqueta que desea aplicar o presione la tecla correspondiente del teclado. Las teclas numéricas se asignan como teclas de acceso rápido para las diez primeras etiquetas. Puede volver a ordenar las etiquetas con los iconos de flecha arriba y abajo del panel del editor de etiquetas.
    > [!Tip]
    > Tenga en cuenta las siguientes sugerencias cuando vaya a etiquetar los formularios:
    >
    > * Solo se puede aplicar una etiqueta a cada elemento de texto seleccionado.
    > * Cada etiqueta solo se puede aplicar una vez por página. Si un valor aparece varias veces en el mismo formulario, cree etiquetas diferentes para cada instancia. Por ejemplo, "factura n.º 1", "factura n.º 2", etc.
    > * Las etiquetas no pueden abarcar varias páginas.
    > * Etiquete los valores tal como aparecen en el formulario; no intente dividir un valor en dos partes con dos etiquetas diferentes. Por ejemplo, un campo de dirección debe etiquetarse con una sola etiqueta incluso si abarca varias líneas.
    > * No incluya claves en los campos etiquetados, solo los valores.
    > * Los datos de la tabla se deben detectar automáticamente y estarán disponibles en el archivo JSON de salida final. Sin embargo, si el modelo no detecta todos los datos de la tabla, también puede etiquetar manualmente estos campos. Etiquete cada celda de la tabla con una etiqueta diferente. Si los formularios tienen tablas con un número variable de filas, asegúrese de etiquetar al menos un formulario con la tabla más grande posible.
    > * Use los botones situados a la derecha de **+** para buscar, reordenar y eliminar las etiquetas, así como cambiarles el nombre.
    > * Para quitar una etiqueta aplicada sin eliminar la etiqueta en sí, seleccione el rectángulo etiquetado en la vista de documento y presione la tecla Supr.
>

---
---

:::image type="content" source="media/label-tool/main-editor-2-1.png" alt-text="Ventana principal del editor de la herramienta de etiquetado de ejemplo.":::

Siga los pasos anteriores para etiquetar al menos cinco de sus formularios.

### <a name="specify-tag-value-types"></a>Especificación de los tipos de valores de etiqueta

Puede establecer el tipo de datos esperado para cada etiqueta. Abra el menú contextual a la derecha de una etiqueta y seleccione un tipo del menú. Esta característica permite que el algoritmo de detección realice suposiciones para mejorar la precisión de la detección de texto. También garantiza que los valores detectados se devuelvan en un formato estandarizado en la salida JSON final. La información del tipo de valor se guarda en el archivo **fields.json** en la misma ruta de acceso que los archivos de etiqueta.

> [!div class="mx-imgBorder"]
> ![Selección del tipo de valor con la herramienta de etiquetado de ejemplo](media/whats-new/value-type.png)

Actualmente se admiten los siguientes tipos de valor y variaciones:

* `string`
  * predeterminado, `no-whitespaces`, `alphanumeric`

* `number`
  * predeterminado, `currency`

* `date`
  * predeterminado, `dmy`, `mdy`, `ymd`

* `time`
* `integer`
* `selectionMark`

> [!NOTE]
> Consulte estas reglas para el formato de fecha:
>
> Para que el formato de fecha funcione, debe especificar un formato (`dmy`, `mdy`, `ymd`).
>
> Los siguientes caracteres se pueden usar como delimitadores de fecha: `, - / . \`. No se puede usar un espacio en blanco como delimitador. Por ejemplo:
>
> * 01,01,2020
> * 01-01-2020
> * 01/01/2020
>
> El día y el mes se pueden escribir con uno o dos dígitos y el año puede tener dos o cuatro dígitos:
>
> * 1-1-2020
> * 1-01-20
>
> Si una cadena de fecha tiene ocho dígitos, el delimitador es opcional:
>
> * 01012020
> * 01 01 2020
>
> El mes también se puede escribir con su nombre completo o abreviado. Si se usa el nombre, los caracteres delimitadores son opcionales. Sin embargo, es posible que este formato se reconozca con menos precisión que otros.
>
> * 01/Ene/2020
> * 01Ene2020
> * 01 de enero de 2020

### <a name="label-tables-v21-only"></a>Tablas de etiquetas (solo v. 2.1)

A veces los datos se prestan mejor a ser etiquetados como tabla en lugar de como pares clave-valor. En este caso puede crear una etiqueta de tabla; para ello, haga clic en "Add a new table tag" (Agregar nueva etiqueta de tabla), especifique si la tabla tendrá un número fijo o variable de filas según el documento, y defina el esquema.

:::image type="content" source="media/label-tool/table-tag.png" alt-text="Configuración de una etiqueta de tabla.":::

Una vez definida la etiqueta de tabla, etiquete los valores de celda.

:::image type="content" source="media/table-labeling.png" alt-text="Etiquetado de una tabla.":::

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

Elija el icono Train (Entrenar) en el panel izquierdo para abrir la página de entrenamiento. A continuación, seleccione el botón **Train** (Entrenar) para empezar a entrenar el modelo. Una vez completado el proceso de entrenamiento, verá la siguiente información:

* **Id. del modelo**: el identificador del modelo que se ha creado y entrenado. Cada llamada de entrenamiento crea un nuevo modelo con su propio identificador. Copie esta cadena en una ubicación segura; la necesitará si desea realizar llamadas de predicción mediante la [API REST](quickstarts/client-library.md?pivots=programming-language-rest-api&tabs=preview%2Cv2-1) o la [guía de la biblioteca cliente](quickstarts/client-library.md).
* **Precisión media**: el promedio de precisión del modelo. Puede mejorar la precisión del modelo si etiqueta más formularios y vuelve a realizar el entrenamiento para crear otro modelo. Se recomienda empezar por etiquetar cinco formularios y agregar más formularios según sea necesario.
* La lista de etiquetas y la precisión estimada por etiqueta.


:::image type="content" source="media/label-tool/train-screen.png" alt-text="Vista de entrenamiento.":::

Una vez finalizado el entrenamiento, examine el valor de **Precisión media**. Si es bajo, debe agregar más documentos de entrada y repetir los pasos anteriores. Los documentos que ya ha etiquetado permanecerán en el índice del proyecto.

> [!TIP]
> También puede ejecutar el proceso de entrenamiento con una llamada a la API REST. Para aprender a hacerlo, consulte [Entrenamiento con etiquetas mediante Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-labeled-data.md).

## <a name="compose-trained-models"></a>Creación de modelos entrenados

### <a name="v21"></a>[v2.1](#tab/v2-1)

Con el modelo Compose, puede crear hasta 100 modelos en un único identificador de modelo. Cuando llame a Analyze con el `modelID` compuesto, Form Recognizer clasificará primero el formulario que envió, elegirá el mejor modelo coincidente y después devolverá los resultados de ese modelo. Esta operación resulta útil cuando los formularios de entrada pueden pertenecer a una de varias plantillas.

Para componer modelos en la herramienta de etiquetado de ejemplo, seleccione el icono de composición de modelos (flecha combinada) de la izquierda. A la izquierda, seleccione los modelos que desee unir. Los modelos con el icono de flechas ya son modelos compuestos.
Elija el **botón Compose** (Componer). En el elemento emergente, asígnele un nombre al nuevo modelo compuesto y seleccione **Compose** (Componer). Una vez finalizada la operación, el modelo recién compuesto debe aparecer en la lista.

:::image type="content" source="media/label-tool/model-compose.png" alt-text="Vista de experiencia de usuario para la creación de modelos.":::

### <a name="v20"></a>[v2.0](#tab/v2-0)

Esta característica solo está disponible en la versión 2.1.

---

## <a name="analyze-a-form"></a>Análisis de un formulario

Seleccione el icono Predict (Predecir) (bombilla) de la izquierda para probar el modelo. Cargue un documento de formulario que no haya utilizado en el proceso de entrenamiento. A continuación, elija el botón **Predict** (Predecir) de la derecha para obtener las predicciones de clave y valor del formulario. La herramienta aplicará etiquetas en los cuadros de límite e informará de la confianza de cada etiqueta.

> [!TIP]
> También puede ejecutar la API de análisis con una llamada a REST. Para aprender a hacerlo, consulte [Entrenamiento con etiquetas mediante Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-labeled-data.md).

## <a name="improve-results"></a>Mejora de los resultados

Según la precisión notificada, puede que desee realizar un entrenamiento adicional para mejorar el modelo. Después de haber realizado una predicción, examine los valores de confianza de cada una de las etiquetas aplicadas. Si el valor de precisión media del entrenamiento es alto, pero las puntuaciones de confianza son bajas (o los resultados son inexactos), debe agregar el archivo de la predicción al conjunto de entrenamiento, etiquetarlo y realizar el entrenamiento de nuevo.

La precisión media notificada, las puntuaciones de confianza y la precisión real pueden ser incoherentes cuando los documentos que se analizan son diferentes de los usados en el entrenamiento. Tenga en cuenta que algunos documentos tienen un aspecto similar cuando los ven los usuarios, pero pueden parecer distintos para el modelo de IA. Por ejemplo, puede realizar el entrenamiento con un tipo de formulario que tiene dos variantes, en el que el conjunto de entrenamiento consta de un 20 % perteneciente a la variante A y un 80 % a la variante B. Durante la predicción, es probable que las puntuaciones de confianza de los documentos de la variante A sean menores.

## <a name="save-a-project-and-resume-later"></a>Guardar un proyecto y reanudarlo más tarde

Para reanudar el proyecto en otro momento o en otro explorador, debe guardar el token de seguridad del proyecto y volver a escribirlo más tarde.

### <a name="get-project-credentials"></a>Obtención de las credenciales del proyecto

Vaya a la página de configuración del proyecto (icono de control deslizante) y anote el nombre del token de seguridad. A continuación, vaya a la configuración de la aplicación (icono de engranaje), que muestra todos los tokens de seguridad de la instancia actual del explorador. Busque el token de seguridad del proyecto y copie su nombre y valor de clave en una ubicación segura.

### <a name="restore-project-credentials"></a>Restauración de las credenciales del proyecto

Si desea reanudar el proyecto, primero debe crear una conexión al mismo contenedor de almacenamiento de blobs. Para ello, repita los pasos anteriores. A continuación, vaya a la página de configuración de la aplicación (icono de engranaje) y compruebe si el token de seguridad del proyecto está allí. Si no es así, agregue un nuevo token de seguridad y copie el nombre y la clave del token del paso anterior. Seleccione **Guardar** para guardar la configuración.

### <a name="resume-a-project"></a>Reanudación de un proyecto

Por último, vaya a la página principal (icono de la casa) y seleccione **Open Cloud Project** (Abrir proyecto en la nube). Después, seleccione la conexión al almacenamiento de blobs y seleccione el archivo **.fott** del proyecto. La aplicación cargará todos los valores del proyecto ya que tiene el token de seguridad.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a utilizar la herramienta de etiquetado de ejemplo de Form Recognizer para entrenar un modelo con datos etiquetados manualmente. Si quiere crear su propia utilidad para etiquetar los datos de entrenamiento, use las API de REST que se ocupan del entrenamiento con datos etiquetados.

> [!div class="nextstepaction"]
> [Entrenamiento con etiquetas mediante Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-labeled-data.md)

* [¿Qué es Form Recognizer?](overview.md)
* [Inicio rápido: Uso de la biblioteca cliente de Form Recognizer](quickstarts/client-library.md)
