---
title: 'Orígenes de datos y tipos de contenido: QnA Maker'
description: Obtenga información sobre cómo importar pares de preguntas y respuestas desde orígenes de datos y tipos de contenido admitidos, incluidos muchos documentos estructurados estándar, como PDF, DOCX y TXT en QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 10/13/2020
ms.openlocfilehash: ff1ce49395a5a7374a0dc739eb26fb7b4a6f72af
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372814"
---
# <a name="importing-from-data-sources"></a>Importación de orígenes de datos

Una base de conocimiento consta de pares de preguntas y respuestas incorporados mediante archivos y direcciones URL públicos.

## <a name="data-source-locations"></a>Ubicaciones de orígenes de datos

El contenido se incluye en una base de conocimiento desde un origen de datos. Las ubicaciones de origen de datos son **direcciones URL o archivos públicos**, que no requieren autenticación.

[Los archivos de SharePoint](../how-to/add-sharepoint-datasources.md), que están protegidos con autenticación, son la excepción. Los recursos de SharePoint deben ser archivos, no páginas web. 

QnA Maker admite direcciones URL públicas que terminan en la extensión web .ASPX y no están protegidas con autenticación.

## <a name="chit-chat-content"></a>Contenido de charla

El conjunto de contenido de charla se ofrece como un origen de datos de contenido completo en varios idiomas y estilos de conversación. Esto puede ser un punto de partida para la personalidad del bot y permite ahorrar el tiempo y el costo que supone escribirlas desde cero. Aprenda [cómo agregar contenido de charla](../how-to/chit-chat-knowledge-base.md) a su base de conocimiento.

## <a name="structured-data-format-through-import"></a>Formato de datos estructurado mediante importación

Al importar una base de conocimiento, se reemplaza el contenido de la base de conocimiento existente. La importación requiere un archivo `.tsv` estructurado que contenga preguntas y respuestas. Esta información ayuda a QnA Maker a agrupar los pares pregunta-respuesta y a atribuirlos a un origen de datos particular.

| Pregunta  | Respuesta  | Source| Metadatos (1 clave: 1 valor) |
|-----------|---------|----|---------------------|
| Pregunta1 | Respuesta1 | URL1 | <code>Key1:Value1 &#124; Key2:Value2</code> |
| Pregunta2 | Respuesta2 | Editorial|    `Key:Value`       |

## <a name="structured-multi-turn-format-through-import"></a>Formato multiturno estructurado mediante importación

Puede crear conversaciones multiturno en un formato de archivo `.tsv`. Este formato le permite crear conversaciones multiturno mediante el análisis de los registros de chat anteriores (con otros procesos, sin usar QnA Maker). A continuación, puede crear un archivo `.tsv` mediante automatización. Importe el archivo para reemplazar la base de conocimiento existente.

> [!div class="mx-imgBorder"]
> ![Modelo conceptual de preguntas de varios turnos de tres niveles](../media/qnamaker-concepts-knowledgebase/nested-multi-turn.png)

La columna de un archivo `.tsv` de varios turnos que refiere específicamente a dicha característica es **Prompts**. El siguiente es un `.tsv` de ejemplo (mostrado en Excel) que muestra la información que se va a incluir para definir los elementos secundarios multiturno:

```JSON
[
    {"displayOrder":0,"qnaId":2,"displayText":"Level 2 Question A"},
    {"displayOrder":0,"qnaId":3,"displayText":"Level 2 - Question B"}
]
```

La propiedad **displayOrder** es numérica, y **displayText** es texto que no debe incluir Markdown.

> [!div class="mx-imgBorder"]
> ![Ejemplo de pregunta multiturno como se ve en Excel](../media/qnamaker-concepts-knowledgebase/multi-turn-tsv-columns-excel-example.png)

## <a name="export-as-example"></a>Exportación como ejemplo

Si no está seguro de cómo representar el par de QnA en el archivo `.tsv`:
* Use este [ejemplo descargable de GitHub](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Structured-multi-turn-format.xlsx?raw=true).
* O bien, cree el par en el portal de QnA Maker, guárdelo y, luego, exporte la base de conocimiento para ver un ejemplo de cómo representar dicho par.

## <a name="unstructured-data-format"></a>Formato de datos no estructurados 

También puede crear una knowledge base en función del contenido no estructurado que importa desde un archivo. Actualmente, esta funcionalidad solo está disponible desde la carga de documentos que utilizan cualquiera de los formatos de archivo admitidos.

> [!IMPORTANT]
> La compatibilidad con contenido no estructurado por medio de la carga de archivos solo está disponible en la característica de respuesta a preguntas personalizada (versión preliminar).

## <a name="content-types-of-documents-you-can-add-to-a-knowledge-base"></a>Tipos de contenido de documento que se pueden agregar a una base de conocimiento
Entre los tipos de contenido se incluyen muchos documentos estructurados estándar, como los archivos PDF, DOC y TXT.

### <a name="file-and-url-data-types"></a>Tipos de datos de archivo y URL

En la tabla siguiente se resumen los tipos de contenido y los formatos de archivo que son compatibles con QnA Maker.

|Tipo de origen|Tipo de contenido| Ejemplos|
|--|--|--|
|URL|Preguntas más frecuentes<br> (sin formato, con secciones o con una página principal de temas)<br>Páginas de soporte técnico <br> (artículos paso a paso de una sola página, artículos de solución de problemas, etc.)|[Preguntas más frecuentes sin formato](../troubleshooting.md), <br>[preguntas más frecuentes con vínculos](https://www.microsoft.com/software-download/faq),<br> [página principal de preguntas más frecuentes por temas](https://www.microsoft.com/Licensing/servicecenter/Help/Faq.aspx)<br>[artículo de soporte técnico](./best-practices.md)|
|PDF/DOC|Preguntas más frecuentes,<br> manual de producto,<br> folletos,<br> documento,<br> directiva de folleto,<br> guía de soporte técnico,<br> archivo de preguntas y respuestas estructurado,<br> etc.|**Sin varios turnos**<br>[Archivo de preguntas y respuestas estructurado.docx](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/structured.docx),<br> [Manual de producto de ejemplo.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/product-manual.pdf),<br> [Ejemplo semiestructurado.docx](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/semi-structured.docx),<br> [Notas del producto de ejemplo.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/white-paper.pdf),<br> [Blog no estructurado.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Introducing-surface-laptop-4-and-new-access.pdf),<br> [Notas del producto no estructuradas.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/sample-unstructured-paper.pdf)<br><br>**Varios turnos**:<br>[Surface Pro (docx)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx)<br>[Ventajas de Contoso (docx)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.docx)<br>[Ventajas de Contoso (pdf)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.pdf)|
|*Excel|Archivo de preguntas y respuestas estructurado<br> (incluye compatibilidad con RTF y HTML)|**Sin varios turnos**:<br>[Preguntas más frecuentes de preguntas y respuestas de ejemplo.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/QnA%20Maker%20Sample%20FAQ.xlsx)<br><br>**Varios turnos**:<br>[Preguntas más frecuentes de archivo simple estructurado.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Structured-multi-turn-format.xlsx)<br>[Preguntas más frecuentes sobre dispositivos Surface Laptop.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-Surface-Pro.xlsx)|
|*TXT/TSV|Archivo de preguntas y respuestas estructurado|[Charla de ejemplo.tsv](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Scenario_Responses_Friendly.tsv)|

Si necesita autenticación para el origen de datos, tenga en cuenta los siguientes métodos para introducir ese contenido en QnA Maker:

* Descargar manual el archivo e importarlo en QnA Maker
* Agregar el archivo desde una [ubicación de SharePoint](../how-to/add-sharepoint-datasources.md) autenticada

### <a name="url-content"></a>Contenido de URL

Se pueden importar dos tipos de documentos a través de una **dirección URL** en QnA Maker:

* Preguntas más frecuentes sobre las direcciones URL
* Direcciones URL de soporte técnico

Cada tipo indica un formato esperado.

### <a name="file-based-content"></a>Contenido basado en archivos

Puede agregar archivos a una base de conocimiento desde un origen de datos público o desde el sistema de archivos local en el portal de [QnA Maker](https://www.qnamaker.ai).

### <a name="content-format-guidelines"></a>Instrucciones de formato de contenido

Obtenga más información sobre las [directrices de formato](../reference-document-format-guidelines.md) para los distintos archivos.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [editar preguntas y respuestas](../how-to/edit-knowledge-base.md).
