---
title: 'Facturas: Form Recognizer'
titleSuffix: Azure Applied AI Services
description: 'Conozca los conceptos relacionados con el análisis de facturas mediante Form Recognizer API: uso y límites.'
services: cognitive-services
author: laujan
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 04/30/2021
ms.author: lajanuar
ms.openlocfilehash: fc00e651cf8ec61a884864c57c0cafd2551f1a38
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111890697"
---
# <a name="form-recognizer-prebuilt-invoice-model"></a>Modelo de factura precompilado de Form Recognizer

Azure Form Recognizer puede analizar y extraer información de facturas de compra mediante sus modelos de factura precompilados. Invoice API permite a los clientes tomar facturas en distintos formatos y devolver datos estructurados para automatizar el procesamiento de facturas. Combina las eficaces capacidades de [reconocimiento óptico de caracteres (OCR)](../computer-vision/overview-ocr.md) con modelos de aprendizaje profundo de reconocimiento de facturas para extraer información clave de facturas en inglés. Extrae el texto, las tablas y la información como el cliente, el proveedor, el identificador de la factura, la fecha de vencimiento de la factura, el total, el importe debido de la factura, el importe de los impuestos, la dirección de envío, la dirección de facturación, elementos de línea, etc. La instancia precompilada de Invoice API está disponible públicamente en Form Recognizer v2.1.

## <a name="what-does-the-invoice-service-do"></a>¿Qué hace el servicio Invoice?

La API Invoice extrae campos clave y elementos de línea de las facturas y los devuelve en una respuesta JSON estructurada organizada. Las facturas pueden ser de distintos formatos y tener diferentes grados de calidad, lo que incluye imágenes capturadas por un teléfono, documentos digitalizados y archivos PDF digitales. Invoice API extraerá la salida estructurada de todas estas facturas.

![Ejemplo de factura de Contoso](./media/invoice-example-new.jpg)

## <a name="try-it-out"></a>Prueba

Para probar el servicio Invoice de Form Recognizer, vaya a la herramienta de interfaz de usuario de ejemplo en línea:

> [!div class="nextstepaction"]
> [Uso de modelos precompilados](https://aka.ms/fott-2.1-ga)

Necesitará una suscripción a Azure ([cree una gratis](https://azure.microsoft.com/free/cognitive-services)) y un punto de conexión de [recursos de Form Recognizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer), así como la clave para probar el servicio Invoice de Form Recognizer.

:::image type="content" source="media/analyze-invoice-new.png" alt-text="Ejemplo de factura analizada" lightbox="media/analyze-invoice-new.png":::

### <a name="input-requirements"></a>Requisitos de entrada

[!INCLUDE [input requirements](./includes/input-requirements-receipts.md)]

## <a name="supported-locales"></a>Configuraciones regionales admitidas

**La instancia precompilada de Invoice API v2.1** admite facturas en la configuración regional **en-us**.

## <a name="the-analyze-invoice-operation"></a>La operación Analyze Invoice

La operación [Analyze Invoice](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/5ed8c9843c2794cbb1a96291) toma una imagen o un archivo PDF de una factura como entrada y extrae los valores de interés. La llamada devuelve un campo de encabezado de respuesta denominado `Operation-Location`. El valor `Operation-Location` es una dirección URL que contiene el id. de resultado que se va a usar en el paso siguiente.

|Encabezado de respuesta| Dirección URL del resultado |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.1/prebuilt/invoice/analyzeResults/49a36324-fc4b-4387-aa06-090cfbf0064f` |

## <a name="the-get-analyze-invoice-result-operation"></a>La operación Get Analyze Invoice Result

El segundo paso consiste en llamar a la operación [Get Analyze Invoice Result](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/5ed8c9acb78c40a2533aee83). Esta operación toma como entrada el id. de resultado que la operación Analyze Invoice ha creado. Devuelve una respuesta JSON que contiene un campo de **estado** con los siguientes valores posibles. Llamará a esta operación de forma iterativa hasta que se devuelva con el valor **correcto**. Use un intervalo de 3 a 5 segundos para evitar superar la tasa de solicitudes por segundo (RPS).

|Campo| Tipo | Valores posibles |
|:-----|:----:|:----|
|status | string | notStarted: la operación de análisis no se ha iniciado.<br /><br />running: la operación de análisis está en curso.<br /><br />failed: error en la operación de análisis.<br /><br />succeeded: la operación de análisis se realizó correctamente.|

Cuando el campo de **estado** tiene el valor **succeeded**, la respuesta de JSON incluirá los resultados del reconocimiento de la factura, las tablas extraídas y los resultados del reconocimiento de texto opcional, en caso de que se solicite. El resultado del reconocimiento de la factura se organiza como un diccionario de valores de campo con nombre, donde cada valor contiene el texto extraído, el valor normalizado, el rectángulo delimitador, la confianza y los elementos de la palabra correspondiente. También incluye los elementos de línea extraídos, donde cada elemento de línea contiene el importe, la descripción, el precio unitario, la cantidad, etc. El resultado del reconocimiento de texto se organiza como una jerarquía de líneas y palabras, con texto, rectángulo delimitador e información de confianza.

### <a name="sample-json-output"></a>Salida de JSON de ejemplo

La respuesta a la operación Get Analyze Invoice Result será la representación estructurada de la factura con toda la información extraída.
Aquí encontrará un [archivo con una factura de ejemplo](media/sample-invoice.jpg) y su salida estructurada, una [salida de una factura de ejemplo](media/invoice-example-new.jpg).

La salida JSON tiene tres partes:
* El nodo `"readResults"` contiene todo el texto reconocido y las marcas de selección. El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales.
* El nodo `"pageResults"` contiene las tablas y celdas extraídas con sus rectángulos delimitadores, su confianza y una referencia a las líneas y palabras de "readResults".
* El nodo `"documentResults"` contiene los valores específicos y los elementos de línea de la factura que el modelo ha detectado. Aquí es donde se encuentran todos los campos de la factura, como el identificador, la dirección de envío, la de facturación, el cliente, el total, los elementos de línea, etc.

## <a name="example-output"></a>Salida de ejemplo

El servicio Invoice extrae el texto, las tablas y 26 campos de la factura. A continuación se muestran los campos extraídos de una factura en la respuesta de salida JSON (la siguiente salida usa esta [factura de ejemplo](media/sample-invoice.jpg)).

### <a name="key-value-pairs"></a>Pares clave-valor 

|Nombre| Tipo | Descripción | Texto | Valor (salida estándar) |
|:-----|:----|:----|:----| :----|
| CustomerName | string | Cliente al que se va a facturar | Microsoft Corp |  |
| CustomerId | string | Identificador de referencia del cliente | CID-12345 |  |
| PurchaseOrder | string | Número de referencia del pedido | PO-3333 | |
| InvoiceId | string | Identificador de esta factura concreta (a menudo "Número de factura") | INV-100 | |
| FechaFactura | date | Fecha en que se generó la factura | 15/11/2019 | 15-11-2019 |
| DueDate | date | Fecha de vencimiento de esta factura | 15/12/2019 | 15-12-2019 |
| VendorName | string | Proveedor que ha creado esta factura | CONTOSO LTD. | |
| VendorAddress | string | Dirección de correo del proveedor | 123 456th St New York, NY, 10001 | |
| VendorAddressRecipient | string | Nombre asociado a VendorAddress | Oficina central de Contoso | |
| CustomerAddress | string | Dirección de correo del cliente | 123 Other St, Redmond WA, 98052 | |
| CustomerAddressRecipient | string | Nombre asociado a CustomerAddress | Microsoft Corp | |
| BillingAddress | string | Dirección facturación del cliente | 123 Bill St, Redmond WA, 98052 | |
| BillingAddressRecipient | string | Nombre asociado a BillingAddress | Servicios Microsoft | |
| ShippingAddress | string | Dirección de envío explícita del cliente | 123 Ship St, Redmond WA, 98052 | |
| ShippingAddressRecipient | string | Nombre asociado a ShippingAddress | Microsoft Delivery | |
| SubTotal | number | Campo de subtotal identificado en esta factura | 100,00 USD | 100 |
| TotalTax | number | Campo de total de impuestos identificado en esta factura | 10,00 USD | 10 |
| InvoiceTotal | number | Total de cargos nuevos asociados a esta factura | 110,00 USD | 110 |
| AmountDue |  number | Importe total debido al proveedor | 610,00 USD | 610 |
| ServiceAddress | string | Dirección de servicio o dirección de propiedad explícitas para el cliente | 123 Service St, Redmond WA, 98052 | |
| ServiceAddressRecipient | string | Nombre asociado a ServiceAddress | Servicios Microsoft | |
| RemittanceAddress | string | Dirección de remite o de pago explícitas del cliente | 123 Remit St New York, NY, 10001 |  |
| RemittanceAddressRecipient | string | Nombre asociado a RemittanceAddress | Facturación de Contoso |  |
| ServiceStartDate | date | Primera fecha del período de servicio (por ejemplo, un período de servicio de factura de la utilidad) | 14/10/2019 | 2019-10-14 |
| ServiceEndDate | date | Fecha de finalización del período de servicio (por ejemplo, un período de servicio de factura de la utilidad) | 14/11/2019 | 14-11-2019 |
| PreviousUnpaidBalance | number | Saldo explícito no pagado previamente | 500,00 USD | 500 |

### <a name="line-items"></a>Elementos de línea

A continuación se muestran los elementos de línea extraídos de una factura en la respuesta de salida JSON (la siguiente salida usa esta [factura de ejemplo](./media/sample-invoice.jpg))

|Nombre| Tipo | Descripción | Texto (elemento de línea n.º 1) | Valor (salida estándar) |
|:-----|:----|:----|:----| :----|
| Elementos | string | Línea de texto de cadena completa del elemento de línea | 3/4/2021 A123 Consulting Services 2 hours $30.00 10% $60.00 | |
| Amount | number | La cantidad del elemento de línea | $60.00 | 100 |
| Descripción | string | La descripción de texto para el elemento de la línea de factura | Servicios de consultoría | Servicios de consultoría |
| Cantidad | number | La cantidad para este elemento de línea de factura | 2 | 2 |
| UnitPrice | number | El precio neto o bruto (según la configuración de factura bruta de la factura) de una unidad de este elemento | $30.00 | 30 |
| ProductCode | string| Código de producto, número de producto o SKU asociado al elemento de línea específico | A123 | |
| Unidad | string| Unidad del elemento de línea, por ejemplo, kg, lb, etc. | horas | |
| Fecha | fecha| Fecha correspondiente a cada elemento de línea. Suele ser la fecha en la que se ha enviado el elemento de línea | 3/4/2021| 2021-03-04 |
| Impuesto | number | Impuestos asociados a cada elemento de línea. Los valores posibles incluyen importe de impuestos, porcentaje de impuestos e impuesto S/N | 10 % | |

Los pares de clave/valor y los elementos de línea de la factura extraídos se encuentran en la sección documentResults de la salida JSON. 

## <a name="next-steps"></a>Pasos siguientes

- Pruebe sus propias facturas y ejemplos en la interfaz de usuario de ejemplo de [Form Recognizer](https://aka.ms/fott-2.1-ga).
- Realice un [inicio rápido de Form Recognizer](quickstarts/client-library.md) para empezar a escribir una aplicación de procesamiento de facturas con Form Recognizer en el lenguaje de desarrollo que prefiera.

## <a name="see-also"></a>Consulte también

* [¿Qué es Form Recognizer?](./overview.md)
* [Documentos de referencia de la API de REST](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/5ed8c9843c2794cbb1a96291)
