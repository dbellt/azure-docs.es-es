---
title: Cognitive Services en Azure Synapse Analytics
description: Enriquezca los datos con inteligencia artificial en Azure Synapse Analytics mediante modelos previamente entrenados de Azure Cognitive Services.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: overview
ms.reviewer: jrasnick, garye, negust, ruxu
ms.date: 06/30/2021
author: garyericson
ms.author: garye
ms.openlocfilehash: c9db8c1cf508d05dfafae672e5f03ff55bb046c0
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113114583"
---
# <a name="cognitive-services-in-azure-synapse-analytics"></a>Cognitive Services en Azure Synapse Analytics

Con los modelos previamente entrenados de Azure Cognitive Services, puede enriquecer los datos con inteligencia artificial en Azure Synapse Analytics.

[Azure Cognitive Services](/azure/cognitive-services/what-are-cognitive-services) son servicios basados en la nube que agregan inteligencia cognitiva a los datos, incluso si no tiene aptitudes para la inteligencia artificial o ciencia de datos. Existen varias maneras de utilizar estos servicios con los datos en Synapse Analytics:

- El asistente para Cognitive Services de Synapse Analytics genera código PySpark en un cuaderno de Synapse que se conecta a un servicio cognitivo mediante los datos de una tabla de Spark. A continuación, mediante modelos de aprendizaje automático previamente entrenados, el servicio realiza el trabajo para agregar inteligencia artificial a los datos.

- En [Tutorial: Creación de aplicaciones de aprendizaje automático con Microsoft Machine Learning para Apache Spark (versión preliminar)](tutorial-build-applications-use-mmlspark.md) se muestra cómo llamar a varios servicios cognitivos mediante Microsoft Machine Learning para Apache Spark (MMLSpark).

- A partir del código de PySpark generado por el asistente o el código MMLSpark de ejemplo proporcionado en el tutorial, puede escribir su propio código para utilizar otros servicios cognitivos con sus datos. Consulte [¿Qué es Azure Cognitive Services?](/azure/cognitive-services/what-are-cognitive-services) para más información sobre los servicios disponibles.

## <a name="get-started"></a>Introducción

[Tutorial: Requisitos previos para el uso de Cognitive Services en Azure Synapse](tutorial-configure-cognitive-services-synapse.md) le puede servir de guía siguiendo un par de pasos que debe realizar antes de utilizar Cognitive Services en Synapse Analytics.

## <a name="tutorials"></a>Tutoriales

En los tutoriales siguientes se proporcionan ejemplos completos del uso de Cognitive Services en Synapse Analytics.

- [Tutorial: Análisis de sentimiento con Cognitive Services (versión preliminar)](tutorial-cognitive-services-sentiment.md): con un conjunto de datos de ejemplo de comentarios de clientes, cree una tabla de Spark con una columna que indique el sentimiento de los comentarios de cada fila.

- [Tutorial: Detección de anomalías con Cognitive Services (versión preliminar)](tutorial-cognitive-services-anomaly.md): con un conjunto de datos de ejemplo de datos de serie temporal, cree una tabla de Spark con una columna que indique si los datos de cada fila presentan una anomalía.

- [Tutorial: Creación de aplicaciones de aprendizaje automático con Microsoft Machine Learning para Apache Spark (versión preliminar)](tutorial-build-applications-use-mmlspark.md): en este tutorial se muestra cómo se utiliza MMLSpark para acceder a varios modelos desde Cognitive Services.

## <a name="next-steps"></a>Pasos siguientes

- [Funcionalidades de aprendizaje automático en Azure Synapse Analytics](what-is-machine-learning.md)
- [¿Qué es Cognitive Services?](/azure/cognitive-services/what-are-cognitive-services)
- [Uso de un cuaderno de ejemplo de la galería de Synapse Analytics](quickstart-gallery-sample-notebook.md)