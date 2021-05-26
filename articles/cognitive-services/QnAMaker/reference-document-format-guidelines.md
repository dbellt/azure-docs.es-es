---
title: 'Instrucciones de formato de documentos importados: QnA Maker'
description: Siga estas instrucciones para importar documentos a fin de obtener los mejores resultados de su contenido.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 04/06/2020
ms.openlocfilehash: 40b9d9e665aacf3024e409ef6bf1b5596ad28ca9
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372538"
---
# <a name="format-guidelines-for-imported-documents-and-urls"></a>Instrucciones de formato para las direcciones URL y los documentos importados

Revise estas instrucciones de formato para obtener los mejores resultados de su contenido.

## <a name="formatting-considerations"></a>Consideraciones de formato

Después de importar un archivo o una dirección URL, QnA Maker convierte y almacena el contenido en [formato de Markdown](https://en.wikipedia.org/wiki/Markdown). El proceso de conversión agrega nuevas líneas al texto, como `\n\n`. Conocer el formato de Markdown le ayuda a comprender el contenido convertido y a administrar el contenido de la base de conocimiento.

Si agrega o edita el contenido directamente en la base de conocimiento, use el **formato de Markdown** para crear contenido de texto enriquecido o cambiar el contenido de dicho formato que ya está en la respuesta. QnA Maker admite gran parte del formato de Markdown para aportar funcionalidades de texto enriquecido al contenido. Sin embargo, es posible que la aplicación cliente, como un bot de chat, no admita el mismo conjunto de formatos de Markdown. Es importante probar la presentación de las respuestas de la aplicación cliente.

Vea una lista completa de [tipos de contenido y ejemplos](./concepts/data-sources-and-content.md#content-types-of-documents-you-can-add-to-a-knowledge-base).

## <a name="basic-document-formatting"></a>Formato básico de documentos

QnA Maker identifica las secciones y subsecciones, así como las relaciones, en el archivo según las indicaciones visuales, como:

* tamaño de fuente
* estilo de fuente
* numeración
* colores

> [!NOTE]
> Actualmente no se admite la extracción de imágenes de los documentos cargados.

### <a name="product-manuals"></a>Manuales de productos

Un manual suele ser material de orientación que acompaña a un producto. Ayuda al usuario a configurar, usar y mantener el producto y a solucionar problemas con el mismo. Cuando QnA Maker procesa un manual, extrae los títulos y subtítulos como preguntas y el contenido ulterior como respuestas. Consulte un ejemplo [aquí](https://download.microsoft.com/download/2/9/B/29B20383-302C-4517-A006-B0186F04BE28/surface-pro-4-user-guide-EN.pdf).

A continuación se muestra un ejemplo de un manual con una página de índice y contenido jerárquico

> [!div class="mx-imgBorder"]
> ![Ejemplo de manual del producto para una base de conocimiento](./media/qnamaker-concepts-datasources/product-manual.png)

> [!NOTE]
> La extracción funciona mejor en manuales que tienen una tabla de contenido o una página de índice y una estructura clara con títulos jerárquicos.

### <a name="brochures-guidelines-papers-and-other-files"></a>Folletos, instrucciones, documentos y otros archivos

También se pueden procesar muchos otros tipos de documentos para generar pares de preguntas y respuestas, siempre que tengan una estructura y un diseño claros. Entre ellas se incluyen las siguientes: folletos, instrucciones, informes, documentos técnicos, documentos científicos, directivas, libros, etc. Consulte un ejemplo [aquí](https://qnamakerstore.blob.core.windows.net/qnamakerdata/docs/Manage%20Azure%20Blob%20Storage.docx).

A continuación se muestra un ejemplo de documento semiestructurado, sin un índice:

> [!div class="mx-imgBorder"]
> ![Documento semiestructurado de Azure Blob Storage](./media/qnamaker-concepts-datasources/semi-structured-doc.png)

### <a name="unstructured-document-support"></a>Compatibilidad con documentos no estructurados

La característica de respuesta a preguntas personalizada ahora admite documentos no estructurados. Se puede considerar que un documento que no tiene su contenido organizado de una manera jerárquica bien definida, al que le falta una estructura de conjunto o cuyo contenido fluye libremente es un documento no estructurado.

A continuación se muestra un ejemplo de un documento PDF no estructurado:

> [!div class="mx-imgBorder"]
> ![Ejemplo de documento no estructurado para una knowledge base](./media/qnamaker-concepts-datasources/unstructured-qna-pdf.png)

 Actualmente, esta funcionalidad solo está disponible por medio de la carga de documentos y solo para los formatos de archivo PDF y DOC.

> [!IMPORTANT]
> La compatibilidad con archivos o contenido no estructurado solo está disponible en la característica de respuesta a preguntas personalizada (versión preliminar).

### <a name="structured-qna-document"></a>Documento de preguntas y respuestas estructurado

El formato de tipo pregunta-respuestas estructurado en los archivos de documento (DOC) consiste en alternar preguntas y respuestas por línea, una pregunta por línea seguida de su respuesta en la línea siguiente, tal como se muestra a continuación:

```text
Question1

Answer1

Question2

Answer2
```

A continuación se muestra un ejemplo de un documento de Word de preguntas y respuestas estructurado:

> [!div class="mx-imgBorder"]
> ![Ejemplo de documento de QnA estructurado para una base de conocimiento](./media/qnamaker-concepts-datasources/structured-qna-doc.png)

### <a name="structured-txt-tsv-and-xls-files"></a>Archivos *TXT*, *TSV* y *XLS* estructurados

Las preguntas y respuestas en forma de archivos *.txt*, *.tsv* o *.xls* estructurados también se pueden cargar en QnA Maker para crear o ampliar una base de conocimiento.  Pueden ser texto sin formato o pueden tener contenido en formato RTF o HTML.

| Pregunta  | Respuesta  | Metadatos (1 clave: 1 valor) |
|-----------|---------|-------------------------|
| Pregunta1 | Respuesta1 | <code>Key1:Value1 &#124; Key2:Value2</code> |
| Pregunta2 | Respuesta2 |      `Key:Value`           |

Se ignoran las columnas adicionales del archivo de origen.

#### <a name="example-of-structured-excel-file"></a>Ejemplo de archivo de Excel estructurado

A continuación se muestra un ejemplo de archivo *.xls* de preguntas y respuestas estructurado, con contenido HTML:

> [!div class="mx-imgBorder"]
> ![Ejemplo de libro Excel de QnA estructurado para una base de conocimiento](./media/qnamaker-concepts-datasources/structured-qna-xls.png)

#### <a name="example-of-alternate-questions-for-single-answer-in-excel-file"></a>Ejemplo de preguntas alternativas para una sola respuesta en un archivo de Excel

A continuación se muestra un ejemplo de un archivo *.xls* de QnA estructurado, con varias preguntas alternativas para una sola respuesta:

> [!div class="mx-imgBorder"]
> ![Ejemplo de preguntas alternativas para una sola respuesta en un archivo de Excel](./media/qnamaker-concepts-datasources/xls-alternate-question-example.png)

Una vez importado el archivo, el par pregunta-respuesta se encuentra en la base de conocimiento, como se muestra a continuación:

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de preguntas alternativas para una sola respuesta importada en la base de conocimiento](./media/qnamaker-concepts-datasources/xls-alternate-question-example-after-import.png)

### <a name="structured-data-format-through-import"></a>Formato de datos estructurado mediante importación

Al importar una base de conocimiento, se reemplaza el contenido de la base de conocimiento existente. La importación requiere un archivo .tsv estructurado que contiene información del origen de datos. Esta información ayuda a QnA Maker a agrupar los pares pregunta-respuesta y a atribuirlos a un origen de datos particular.

| Pregunta  | Respuesta  | Source| Metadatos (1 clave: 1 valor) |
|-----------|---------|----|---------------------|
| Pregunta1 | Respuesta1 | URL1 | <code>Key1:Value1 &#124; Key2:Value2</code> |
| Pregunta2 | Respuesta2 | Editorial|    `Key:Value`       |

<a href="#formatting-considerations"></a>

### <a name="multi-turn-document-formatting"></a>Formato de documento de varios turnos

* Use encabezados y subtítulos para denotar la jerarquía. Por ejemplo, puede agregar h1 para indicar las preguntas y respuestas principales y h2 para las que se deben tomar como aviso. Use el tamaño de encabezado pequeño para indicar la jerarquía subsiguiente. No utilice el estilo, el color u otro mecanismo para inferir la estructura en el documento, QnA Maker no extraerá las indicaciones multiturno.
* El primer carácter del encabezado debe escribirse en mayúsculas.
* No termine un encabezado con un signo de interrogación, `?`.

**Documentos de muestra**:<br>[Surface Pro (docx)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx)<br>[Ventajas de Contoso (docx)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.docx)<br>[Ventajas de Contoso (pdf)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.pdf)

## <a name="faq-urls"></a>Preguntas más frecuentes sobre las direcciones URL

QnA Maker puede admitir páginas web de preguntas más frecuentes de 3 formas diferentes:

* Páginas de preguntas más frecuentes sin formato
* Páginas de preguntas más frecuentes con vínculos
* Páginas de preguntas más frecuentes con una página principal de temas

### <a name="plain-faq-pages"></a>Páginas de preguntas más frecuentes sin formato

Este es el tipo más común de página de preguntas más frecuentes, en la que las respuestas siguen inmediatamente las preguntas de la misma página.

Aquí tiene un ejemplo de una página de preguntas más frecuentes sin formato:

> [!div class="mx-imgBorder"]
> ![Ejemplo de página de preguntas más frecuentes sin formato para una base de conocimiento](./media/qnamaker-concepts-datasources/plain-faq.png)


### <a name="faq-pages-with-links"></a>Páginas de preguntas más frecuentes con vínculos

En este tipo de página de preguntas más frecuentes, las preguntas se agregan juntas y se vinculan a respuestas que están en secciones diferentes de la misma página o en páginas distintas.

A continuación se muestra un ejemplo de una página de preguntas más frecuentes con vínculos en las secciones que se encuentran en la misma página:

> [!div class="mx-imgBorder"]
> ![Ejemplo de página de preguntas más frecuentes con sección de vínculos para una base de conocimiento](./media/qnamaker-concepts-datasources/sectionlink-faq.png)


### <a name="parent-topics-page-links-to-child-answers-pages"></a>Vínculos de páginas de temas primarios a páginas de respuestas secundarias

Este tipo de preguntas más frecuentes tiene una página de temas en la que cada tema se vincula a un conjunto de preguntas y respuestas correspondiente en una página diferente. QnA Maker rastrea todas las páginas vinculadas para extraer las preguntas y respuestas correspondientes.

A continuación, se muestra un ejemplo de una página de temas con vínculos a secciones de preguntas más frecuentes en páginas distintas.

> [!div class="mx-imgBorder"]
> ![Ejemplo de página de preguntas más frecuentes con vínculos profundos para una base de conocimiento](./media/qnamaker-concepts-datasources/topics-faq.png)

### <a name="support-urls"></a>Direcciones URL de soporte técnico

QnA Maker puede procesar páginas web de soporte semiestructuradas, como artículos web que describan cómo realizar una tarea determinada, cómo diagnosticar y resolver un problema determinado y cuáles son las mejores prácticas para un determinado proceso. La extracción funciona mejor en contenido con una estructura clara y con encabezados jerárquicos.

> [!NOTE]
> La extracción para artículos de soporte técnico es una característica nueva que se encuentra en sus primeras etapas. Funciona mejor con páginas simples, que están bien estructuradas y no contienen encabezados ni pies de página complejos.

> [!div class="mx-imgBorder"]
> ![QnA Maker admite la extracción en páginas web semiestructuradas con una estructura clara y con encabezados jerárquicos.](./media/qnamaker-concepts-datasources/support-web-pages-with-heirarchical-structure.png)

## <a name="import-and-export-knowledge-base"></a>Importación y exportación de la base de conocimiento

**Los archivos TSV y XLS**, de las bases de conocimiento exportadas, solo se pueden usar importando los archivos de la página **Configuración** en el portal QnA Maker. No se pueden usar como orígenes de datos durante la creación de la base de conocimiento ni desde las funciones **+ Agregar archivo** o **+ Agregar URL** de la página **Configuración**. 

Al importar la knowledge base mediante estos **archivos TSV y XLS**, los pares de preguntas y respuestas se agregan al origen editorial y no a los orígenes de los que se extrajeron estos pares en la knowledge base exportada. 


## <a name="next-steps"></a>Pasos siguientes

Vea una lista completa de [tipos de contenido y ejemplos](./concepts/data-sources-and-content.md#content-types-of-documents-you-can-add-to-a-knowledge-base).
