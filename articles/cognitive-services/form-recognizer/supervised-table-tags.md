---
title: 'Cómo usar etiquetas de tabla para entrenar el modelo de formulario personalizado: Form Recognizer'
titleSuffix: Azure Applied AI Services
description: Aprenda a usar de forma eficaz el etiquetado de etiquetas de tabla supervisado.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: how-to
ms.date: 03/15/2021
ms.author: lajanuar
ms.openlocfilehash: 6cbf96a050124fe0d72bb0fc24be4ad7ea265fed
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110374058"
---
# <a name="use-table-tags-to-train-your-custom-form-model"></a>Uso de etiquetas de tabla para entrenar el modelo de formulario personalizado

En este artículo, aprenderá a entrenar el modelo de formulario personalizado con etiquetas de tabla (etiquetas). Algunos escenarios requieren un etiquetado más complejo que simplemente alinear pares clave-valor. Estos escenarios incluyen la extracción de información de formularios con estructuras jerárquicas complejas o la detección de elementos que el servicio no detecta ni extrae automáticamente. En estos casos, puede usar etiquetas de tabla para entrenar el modelo de formulario personalizado.

## <a name="when-should-i-use-table-tags"></a>¿Cuándo debo usar etiquetas de tabla?

Estos son algunos ejemplos de cuándo sería apropiado usar etiquetas de tabla:

- Los datos que quiere extraer se muestran como tablas en los formularios y la estructura de las tablas es significativa. Por ejemplo, cada fila de la tabla representa un elemento y cada columna de la fila representa una característica específica de ese elemento. En este caso, podría usar una etiqueta de tabla donde una columna representa características y una fila representa información sobre cada característica.
- Hay datos que desea extraer y que no se presentan en campos de formulario específicos, pero semánticamente, los datos podrían caber en una cuadrícula bidimensional. Por ejemplo, el formulario contiene una lista de personas e incluye nombre, apellidos y una dirección de correo electrónico. Desea extraer esta información. En este caso, puede usar una etiqueta de tabla con el nombre, los apellidos y la dirección de correo electrónico como columnas y cada fila se rellena con información sobre una persona de la lista.

> [!NOTE]
> Form Recognizer busca y extrae automáticamente todas las tablas de los documentos, independientemente de si las tablas están etiquetadas o no. Por lo tanto, no tiene que etiquetar todas las tablas del formulario con una etiqueta de tabla y las etiquetas de tabla no tienen que replicar la estructura de la tabla que se encuentra en el formulario. Las tablas extraídas automáticamente con Form Recognizer se incluirán en la sección pageResults de la salida JSON.

## <a name="create-a-table-tag-with-form-ocr-test-tool-fott"></a>Crear una etiqueta de tabla con la herramienta de prueba de OCR de formulario (FOTT)
<!-- markdownlint-disable MD004 -->
* Determine si desea una etiqueta de tabla **dinámica** o **de tamaño fijo**. Si el número de filas varía de un documento a otro, use una etiqueta de tabla dinámica. Si el número de filas es coherente en todos los documentos, utilice una etiqueta de tabla de tamaño fijo.
* Si la etiqueta de tabla es dinámica, defina los nombres de columna y el tipo de datos y el formato para cada columna.
* Si la tabla tiene un tamaño fijo, defina el nombre de la columna, el nombre de la fila, el tipo de datos y el formato de cada etiqueta.
:::image type="content" source="./media/table-tag-configure.png" alt-text="Configuración de una etiqueta de tabla":::

## <a name="label-your-table-tag-data"></a>Etiquetar los datos de la etiqueta de tabla

* Si el proyecto tiene una etiqueta de tabla, puede abrir el panel de etiquetado y rellenar la etiqueta como si etiquetara los campos de clave-valor.
:::image type="content" source="media/table-labeling.png" alt-text="Etiquetado con etiquetas de tabla":::

## <a name="next-steps"></a>Pasos siguientes

Siga nuestra guía de inicio rápido para entrenar y usar el modelo de Form Recognizer personalizado:

> [!div class="nextstepaction"]
> [Entrenamiento con etiquetas mediante la herramienta de etiquetado de ejemplo](label-tool.md)

## <a name="see-also"></a>Consulte también

* [¿Qué es Form Recognizer?](overview.md)
>
