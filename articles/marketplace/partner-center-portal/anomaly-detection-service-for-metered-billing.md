---
title: 'Servicio de detección de anomalías para la facturación de uso medido: Microsoft Azure Marketplace'
description: Describe cómo funciona la detección de anomalías, cuándo se envían notificaciones y qué hacer con ellas, y las opciones de soporte técnico.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.author: mingshen
author: mingshen-ms
ms.date: 06/10/2020
ms.openlocfilehash: 77159a099dcc28739457a10afef374502d117b5a
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971160"
---
# <a name="anomaly-detection-service-for-metered-billing"></a>Servicio de detección de anomalías para la facturación de uso medido

El [servicio de medición de Marketplace](../marketplace-metering-service-apis-faq.md), le permite crear ofertas en el programa comercial de Marketplace que se cobran conforme a unidades no estándar. Con la facturación de uso medido, puede enviar eventos de uso del cliente a Microsoft y nosotros prepararemos la facturación según ese uso.

Unos datos de uso incorrectos pueden deberse a diversas causas como, por ejemplo, errores, configuraciones incorrectas en el seguimiento del consumo o fraude. Los datos de uso incorrectos pueden producir cargos incorrectos al cliente y conflictos de facturación.

Para reducir este riesgo, nuestro servicio de detección de anomalías aplica algoritmos de aprendizaje automático para determinar el comportamiento normal de la facturación de uso medido, analizar el uso de facturación medida y detectar anomalías con una intervención mínima del usuario.

Se le notificará si se detectan anomalías en el uso de facturación medida. Esto le ofrece la oportunidad de investigar y enviarnos una notificación si se confirma que una anomalía es un problema real, momento en el cual se pueden realizar acciones para solucionar el problema de facturación del cliente de forma proactiva.

Además de picos repentinos, interrupciones y cambios de tendencia en el uso de facturación medida, nuestro modelo también tiene en cuenta los efectos estacionales. Dado que la facturación de uso medido se comunica a través de datos de uso por encima del límite, nuestro modelo también es capaz de tratar correctamente con períodos largos de datos que faltan.

A continuación se muestran ejemplos de resultados de detección de anomalías. El intervalo esperable aparece como una banda amarilla. El intervalo aceptable del uso de facturación medida se indica mediante estrellas verdes en la banda. El uso para facturación fuera de la banda se muestra como un punto rojo.  

Anomalías detectadas fuera de una tendencia predecible:

![Muestra las anomalías que se detectan fuera de una tendencia predecible.](media/anomaly-1.png)

Anomalías detectadas fuera de una tendencia cíclica periódica:

![Muestra las anomalías que se detectan fuera de una tendencia cíclica periódica.](media/anomaly-2.png)

Anomalías detectadas en una tendencia al alza:

![Muestra las anomalías detectadas en una tendencia al alza.](media/anomaly-3.png)

## <a name="how-anomaly-detection-service-works"></a>Funcionamiento del servicio de detección de anomalías

La detección de anomalías se habilita automáticamente para todos los usos de facturación medida. Cuando se envían los eventos de uso a Microsoft, el servicio de detección de anomalías crea un modelo de los valores esperables basado en los datos de uso del pasado. Este modelo se ejecuta semanalmente.

La detección de anomalías funciona en un nivel por medidor y por cliente. Esto significa que cada medidor de cada cliente tendrá un modelo entrenado basado en el patrón de uso en el pasado de este cliente para este valor de medición.

El modelo funciona mediante la generación de intervalos de confianza retrospectiva. La previsión de serie temporal es un modelo de suma generalizado que se compone de una parte de predicción de tendencias y una parte de estacionalidad. Dado que el modelo se formula como una tarea de regresión, puede controlar correctamente períodos largos de datos que faltan. Si una observación está fuera de los intervalos de confianza predichos, significa que la observación no se puede explicar según los patrones históricos de la facturación de uso medido y, por lo tanto, puede constituir una anomalía.

## <a name="anomaly-detection-notification"></a>Notificaciones de detección de anomalías

Puede evaluar, administrar y confirmar anomalías en el Centro de partners. Para saber cómo, consulte [Administración de anomalías de facturación de uso medido en el Centro de partners](../anomaly-detection.md).

Para asegurarse de que a los clientes no se les cobra de más por el uso medido, debe investigar si las anomalías detectadas son problemas reales. Si es así, puede confirmar el uso incorrecto en el Centro de partners.

Se recomienda confirmar si las anomalías detectadas son de uso normal. Al hacerlo, se mejorarán los datos de anomalías que se le proporcionan. Si una anomalía representa un riesgo financiero potencialmente alto, podremos ponernos en contacto con usted para confirmar el uso.

## <a name="when-and-how-to-get-support"></a>Obtención de soporte técnico

Si nos ha enviado un uso medido incorrecto y, como resultado, al cliente se la ha cobrado, o se le cobrará, un importe menor, no se emitirá una factura al cliente por el uso menor notificado ni se le pagará ese uso. Tendrá que asumir la pérdida de ingresos debido a la falta de información.

Si se aplica uno de los siguientes casos, puede ajustar la cantidad de uso en el Centro de partners, lo que dará como resultado un reembolso o un ajuste de la facturación para sus clientes:

- Ha confirmado que una de las anomalías encontradas es un problema real y que el uso incorrecto daría como resultado cargos extra al cliente.
- Descubre que nos ha enviado un uso incorrecto y que, como resultado, al cliente se le podría cobrar de más.

Para enviar una incidencia de soporte técnico relacionada con las anomalías de facturación de uso medido:

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) con su cuenta profesional.
1. En el menú que se encuentra en la parte superior derecha de la página, seleccione el icono **Soporte técnico**. El panel **Ayuda y soporte técnico** aparece en el lado derecho de la página.
1. Para ayuda con el marketplace comercial, seleccione **Marketplace comercial**.
   ![Se muestra el panel de soporte técnico.](../media/support/commercial-marketplace-support-pane.png)
1. En el cuadro **Resumen del problema**, escriba **marketplace comercial > facturación de uso medido**.
1. En el cuadro **Tipo de problema**, seleccione uno de los siguientes:
    - **Commercial Marketplace > Metered Billing > Wrong usage sent for Azure Applications offer** (Marketplace comercial > Facturación según uso > Uso incorrecto enviado para oferta de aplicaciones de Azure)
    - **Commercial Marketplace > Metered Billing > Wrong usage sent for Azure Applications offer** (Marketplace comercial > Facturación según uso > Uso incorrecto enviado para oferta de SaaS)
1. En **Paso siguiente**, seleccione **Review solutions** (Revisar soluciones).
1. Revise los documentos recomendados, si los hay, o seleccione **Provide issue details** (Proporcionar detalles del problema) para enviar una incidencia de soporte técnico.

Para ver más opciones de soporte técnico para editores, consulte [Soporte técnico para el programa Marketplace comercial en el Centro de partners](../support.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [API del servicio de medición de Marketplace](../marketplace-metering-service-apis.md).
- [Administración de las anomalías de facturación de uso medido en el Centro de partners](../anomaly-detection.md)