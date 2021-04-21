---
title: Arquitectura de mantenimiento predictivo para usar la API Anomaly Detector Multivariate
titleSuffix: Azure Cognitive Services
description: Arquitectura de referencia para usar las API Anomaly Detector Multivariate para aplicar la detección de anomalías a los datos de serie temporal a fin de realizar el mantenimiento predictivo.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 04/01/2021
ms.author: mbullwin
keywords: anomaly detection, machine learning, algorithms
ms.openlocfilehash: 4d379cfe01dcbd88531b98a32b45e0b30f6adeef
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107318861"
---
# <a name="predictive-maintenance-solution-with-anomaly-detector-multivariate"></a>Solución de mantenimiento predictivo con Anomaly Detector Multivariate

Muchos sectores diferentes necesitan soluciones de mantenimiento predictivo para reducir los riesgos y obtener información sobre la que actuar mediante el procesamiento de datos de sus equipos. El mantenimiento predictivo evalúa la condición del equipo mediante la supervisión en línea. El objetivo es realizar el mantenimiento antes de que el equipo se degrade o averíe.

Supervisar el estado de mantenimiento del equipo puede ser complicado, ya que cada componente dentro del equipo puede generar docenas de señales, por ejemplo, vibración, orientación y rotación.  Esta tarea puede ser incluso más compleja cuando esas señales tienen una relación implícita y deben supervisarse y analizarse juntas. Definir diferentes reglas para esas señales y correlacionarlas manualmente puede ser costoso. La característica multivariante de Anomaly Detector permite que:

* Varias señales correlacionadas se supervisen juntas y las correlaciones entre ellas se tengan en cuenta en el modelo.
* En cada anomalía capturada, el rango de contribución de las distintas señales pueda ayudar a explicar la anomalía y analizar la causa principal del incidente.
* El modelo de detección de anomalías multivariante se cree sin supervisión. Los modelos se puedan entrenar específicamente para distintos tipos de equipo.

Aquí se proporciona una arquitectura de referencia para una solución de mantenimiento predictivo basada en Anomaly Detector Multivariate.

## <a name="reference-architecture"></a>Arquitectura de referencia

[ ![Diagrama de arquitectura que comienza con la recopilación de datos del sensor en el perímetro con una pieza de equipo industrial y que realiza un seguimiento de la canalización de procesamiento y análisis hasta una salida final de una alerta de incidente que se genera después de la ejecución de Anomaly Detector.](../media/multivariate-architecture/multivariate-architecture.png) ](../media/multivariate-architecture/multivariate-architecture.png#lightbox)

En la arquitectura anterior, los eventos de streaming procedentes de datos del sensor se almacenarán en Azure Data Lake y, luego, se procesarán mediante un módulo de transformación de datos para convertirlos en un formato de serie temporal. Mientras tanto, el evento de streaming desencadenará la detección en tiempo real con el modelo entrenado. En general, habrá un módulo para administrar el ciclo de vida del modelo multivariante, como *Bridge Service* (Servicio puente) en esta arquitectura.

**Entrenamiento del modelo**: antes de usar Anomaly Detector Multivariante para detectar anomalías en los componentes o el equipo, es necesario entrenar un modelo en señales específicas (serie temporal) generadas por esta entidad. El módulo *Bridge Service* (Servicio puente) capturará los datos históricos, enviará un trabajo de entrenamiento a Anomaly Detector y, luego, conservará el identificador de modelo en el almacenamiento de *metadatos del modelo*.

**Validación del modelo**: el tiempo de entrenamiento de un modelo determinado puede variar en función del volumen de datos de entrenamiento. El módulo *Bridge Service* (Servicio puente) podría consultar la información de estado y diagnóstico del modelo de forma periódica. Es posible que sea necesario validar la calidad del modelo antes de ponerlo en línea. Si hay etiquetas en el escenario, esas etiquetas se pueden usar para comprobar la calidad del modelo. De lo contrario, se puede usar la información de diagnóstico para evaluar la calidad del modelo, y también se puede realizar la detección sobre datos históricos con el modelo entrenado y evaluar el resultado para confirmar su validez.

**Inferencia de modelos**: la detección en línea se realizará con el modelo válido y el identificador del resultado se puede almacenar en la *tabla de inferencia*. Tanto el proceso de entrenamiento como el proceso de inferencia se realizan de forma asincrónica. En general, una tarea de detección se puede completar en cuestión de segundos. Las señales usadas para la detección deben ser las mismas que las usadas para el entrenamiento. Por ejemplo, si usamos vibración, orientación y rotación para el entrenamiento, en la detección las tres señales deben incluirse como entrada.

**Alertas de incidentes**: los resultados de la detección se pueden consultar con los identificadores de resultado. Cada resultado contiene la gravedad de cada anomalía y el rango de contribución. El rango de contribución se puede usar para comprender por qué se produjo esta anomalía y qué señal provocó este incidente. Se pueden establecer diferentes umbrales de gravedad para generar las alertas y notificaciones que se enviarán a los ingenieros de campo para realizar el trabajo de mantenimiento.

## <a name="next-steps"></a>Pasos siguientes

- [Inicios rápidos](../quickstarts/client-libraries-multivariate.md).
- [Procedimientos recomendados](../concepts/best-practices-multivariate.md): este artículo trata sobre los patrones recomendados para usar con las API multivariantes.