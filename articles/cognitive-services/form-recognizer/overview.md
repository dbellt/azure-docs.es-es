---
title: ¿Qué es Azure Form Recognizer?
titleSuffix: Azure Applied AI Services
description: El servicio Azure Form Recognizer le permite identificar y extraer pares clave-valor y datos de tabla de los documentos de formulario, así como información importante de los recibos de ventas y las tarjetas de presentación.
author: laujan
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: overview
ms.date: 03/15/2021
ms.author: lajanuar
ms.custom: cog-serv-seo-aug-2020
keywords: automated data processing, document processing, automated data entry, forms processing
ms.openlocfilehash: 1688cc686b11fd6a6edb129a7ac01a048ae88f4a
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111957504"
---
# <a name="what-is-azure-form-recognizer"></a>¿Qué es Azure Form Recognizer?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Azure Form Recognizer forma parte de [Azure Applied AI Services](../../applied-ai-services/index.yml), que le permite crear software de procesamiento de datos automatizado mediante tecnología de aprendizaje automático. Identifique y extraiga texto, pares clave-valor, marcas de selección, tablas y una estructura de sus documentos (el servicio genera datos estructurados que incluyen las relaciones del archivo original, rectángulos delimitadores, confianza, etc). Obtendrá rápidamente resultados precisos a la medida de su contenido específico sin necesidad de una intervención manual pesada o una amplia experiencia en ciencia de datos. Use Form Recognizer para automatizar la introducción de datos en las aplicaciones y mejorar las funcionalidades de búsqueda de documentos.

Form Recognizer consta de modelos de procesamiento de documentos personalizados, modelos pregenerados de facturas, recibos, tarjetas de identidad y de presentación, y el modelo de diseño. Puede llamar a los modelos de Form Recognizer mediante una API REST o los SDK de la biblioteca cliente, e integrarlos en su flujo de trabajo o aplicación.

Esta documentación contiene los siguientes tipos de artículos:

* Los [**conceptos**](concept-layout.md) proporcionan explicaciones detalladas sobre la funcionalidad y las características del servicio.
* Los [**inicios rápidos**](quickstarts/client-library.md) son instrucciones de inicio que le guiarán a la hora de hacer solicitudes al servicio.
* Las [**guías de procedimientos**](build-training-data-set.md) contienen instrucciones para usar el servicio de una manera más específica o personalizada.
* Los [**tutoriales**](tutorial-ai-builder.md) son guías más largas que muestran cómo usar el servicio como un componente en soluciones empresariales más amplias.

## <a name="form-recognizer-features"></a>Características de Form Recognizer

Mediante Form Recognizer, puede extraer y analizar fácilmente datos de documentos con las siguientes características:

### <a name="layout"></a>[Diseño](concept-layout.md)

Extraiga de los documentos texto, marcas de selección y estructuras de tablas, junto con las coordenadas de sus rectángulos de selección.

Form Recognizer puede extraer de documentos texto, marcas de selección y estructuras de tablas (los números de fila y columna asociados al texto) mediante el reconocimiento óptico de caracteres (OCR) de alta definición y un modelo de aprendizaje profundo mejorado.

:::image type="content" source="./media/tables-example.jpg" alt-text="Tablas de ejemplo" lightbox="./media/tables-example.jpg":::

### <a name="custom-models"></a>[Modelos personalizados](concept-custom.md)

Extraiga texto, pares clave-valor, marcas de selección y datos de tabla de los formularios. Puede entrenar estos modelos con sus propios datos para que se adapten a sus formularios.

Puede entrenar los modelos personalizados de Form Recognizer con sus propios datos y solo necesita cinco formularios de entrada de ejemplo para empezar. Un modelo de procesamiento de documentos entrenado puede generar datos estructurados que incluyen las relaciones en el documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

Tiene las siguientes opciones al entrenar modelos personalizados: entrenamiento con datos etiquetados y sin ellos.

#### <a name="train-without-labels"></a>Entrenamiento sin etiquetas

Form Recognizer usa el aprendizaje sin supervisión para comprender el diseño y las relaciones entre los campos y las entradas de los formularios. Cuando se envían los formularios de entrada, el algoritmo agrupa los formularios por tipos, descubre qué claves y tablas están presentes y asocia los valores a las claves y entradas a las tablas. El entrenamiento sin etiquetas no requiere el etiquetado de datos manual, codificación intensiva ni mantenimiento; se recomienda probar este método primero.

Consulte [Creación de un conjunto de datos de entrenamiento](./build-training-data-set.md) para ver sugerencias sobre cómo recopilar los documentos de entrenamiento.

#### <a name="train-with-labels"></a>Entrenamiento con etiquetas

Al entrenar con datos etiquetados, el modelo realiza un aprendizaje supervisado para extraer los valores de interés mediante los formularios etiquetados que se proporcionan. Los datos etiquetados derivan en modelos con mejor rendimiento y pueden generar modelos que funcionen con formularios complejos o formularios que contengan valores sin claves.

Form Recognizer usa la API [Diseño](#layout) para aprender los tamaños y las posiciones esperados de los elementos de texto impresos y manuscritos, y para extraer tablas. A continuación, usa etiquetas especificadas por el usuario para aprender las asociaciones clave-valor y las tablas de los documentos. Se recomienda usar cinco formularios etiquetados manualmente del mismo tipo (la misma estructura) para empezar a entrenar un nuevo modelo y agregar más datos etiquetados según sea necesario para mejorar la precisión del modelo. Form Recognizer permite entrenar un modelo para extraer pares de valores clave y tablas mediante funcionalidades de aprendizaje supervisado.

[Introducción al entrenamiento con etiquetas](label-tool.md)

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Azure-Form-Recognizer/player]


### <a name="prebuilt-models"></a>Modelos creados previamente

 Form Recognizer también incluye modelos creados previamente para el procesamiento automatizado de datos de recibos, tarjetas de presentación, facturas y documentos de identidad.

### <a name="receipts"></a>[Receipts](concept-receipts.md)

El modelo de recibo pregenerado se usa para leer los recibos de ventas en Inglés de Australia, Canadá, Gran Bretaña, India y Estados Unidos (el tipo que usan los restaurantes, las estaciones de gas, minoristas, etc). Este modelo extrae información clave como la hora y la fecha de la transacción, información del comerciante, importe de los impuestos, totales y mucho más. Además, el modelo de recibo pregenerado se entrena para analizar y devolver todo el texto en un recibo.

:::image type="content" source="./media/overview-receipt.jpg" alt-text="recibo de ejemplo" lightbox="./media/overview-receipt.jpg":::

### <a name="business-cards"></a>[Tarjetas de presentación](concept-business-cards.md)

El modelo de tarjetas de presentación le permite extraer información como el nombre de la persona, el puesto de trabajo, la dirección, el correo electrónico, la empresa y los números de teléfono de las tarjetas de presentación en inglés.

:::image type="content" source="./media/overview-business-card.jpg" alt-text="tarjeta de presentación de ejemplo" lightbox="./media/overview-business-card.jpg":::

### <a name="invoices"></a>[Facturas](concept-invoices.md)

El modelo de factura pregenerado extrae datos de las facturas en varios formatos y devuelve datos estructurados. Este modelo extrae información clave como el identificador de factura, los detalles del cliente, los detalles del proveedor, la dirección de envío, la dirección de facturación, el total, los impuestos, el subtotal, los elementos de la línea, etc. Además, este modelo se entrena para analizar y devolver todo el texto y las tablas de la factura.

:::image type="content" source="./media/overview-invoices.jpg" alt-text="Factura de ejemplo" lightbox="./media/overview-invoices.jpg":::

### <a name="identity-documents"></a>[Documentos de identidad](concept-identification-cards.md)

El modelo de documentos de identidad (ID) permite extraer información clave de los pasaportes internacionales y los carnés de conducir estadounidenses. Extrae datos como el identificador de documento, la fecha de nacimiento, la fecha de expiración, el nombre, el país, la región, la zona de lectura automática y mucho más.

:::image type="content" source="./media/id-example-drivers-license.jpg" alt-text="tarjeta de identidad de ejemplo" lightbox="./media/overview-id.jpg":::

## <a name="get-started"></a>Primeros pasos

Use la herramienta Form Recognizer de ejemplo para probar el diseño y los modelos pregenerados, además de entrenar un modelo personalizado para los documentos. Necesitará una suscripción de Azure ([**cree una gratis**](https://azure.microsoft.com/free/cognitive-services)) y un punto de conexión y la clave del [**recurso de Form Recognizer**](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer) para probar el servicio Form Recognizer.

### <a name="v21"></a>[v2.1](#tab/v2-1)

>
> [!div class="nextstepaction"]
> [Probar Form Recognizer](https://aka.ms/fott-2.1-ga/)
>

### <a name="v20"></a>[v2.0](#tab/v2-0)

>
> [!div class="nextstepaction"]
> [Probar Form Recognizer](https://fott.azurewebsites.net/)

---

Siga el artículo [Inicio rápido de la API REST y la biblioteca de cliente](./quickstarts/client-library.md) para empezar a extraer datos de los documentos. Se recomienda usar el servicio gratuito cuando se está aprendiendo la tecnología. Recuerde que el número de páginas gratuitas se limita a 500 al mes.

### <a name="v21"></a>[v2.1](#tab/v2-1)

Explore la [documentación de referencia de API REST](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeWithCustomForm) para más información. Si está familiarizado con una versión anterior de la API, consulte el artículo sobre [novedades](./whats-new.md) para más información sobre los cambios recientes.

### <a name="v20"></a>[v2.0](#tab/v2-0)

Explore la [documentación de referencia de API REST](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeLayoutAsync) para más información. Si está familiarizado con una versión anterior de la API, consulte el artículo sobre [novedades](./whats-new.md) para más información sobre los cambios recientes.

---

## <a name="input-requirements"></a>Requisitos de entrada

[!INCLUDE [input requirements](./includes/input-requirements.md)]

## <a name="service-availability-and-redundancy"></a>Disponibilidad y redundancia del servicio

### <a name="is-form-recognizer-service-zone-resilient"></a>¿Tiene el servicio Form Recognizer resistencia de zona?

Sí. El servicio Form Recognizer tiene resistencia de zona de manera predeterminada.

### <a name="how-do-i-configure-the-form-recognizer-service-to-be-zone-resilient"></a>¿Cómo se configura el servicio Form Recognizer para que tenga resistencia de zona?

No es necesaria ninguna configuración de parte del cliente para habilitar la resistencia de zonas. La resistencia de zonas para los recursos de Form Recognizer está disponible de manera predeterminada y la administra el propio servicio.

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

* Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan el servicio Form Recognizer deben estar al tanto de las directivas de Microsoft sobre los datos de los clientes. Para más información, consulte la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

* Form Recognizer no almacena ni procesa los datos del cliente fuera de la región donde el cliente implementa la instancia de servicio.

## <a name="next-steps"></a>Pasos siguientes

Pruebe nuestra herramienta en línea y el inicio rápido para más información sobre el servicio Form Recognizer.

* [**Herramienta Form Recognizer**](https://aka.ms/fott-2.1-ga)
* [**Inicio rápido de la API REST y la biblioteca de cliente**](quickstarts/client-library.md)