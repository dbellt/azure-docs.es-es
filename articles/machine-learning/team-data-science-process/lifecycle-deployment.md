---
title: Fase de implementación del ciclo de vida del proceso de ciencia de datos en equipos
description: Los objetivos, las tareas y los resultados de la fase de implementación de los proyectos de ciencia de datos.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: dd28d1a111c6655588b2e1254e2e503ad3f3ad28
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111813545"
---
# <a name="deployment-stage-of-the-team-data-science-process-lifecycle"></a>Fase de implementación del ciclo de vida del proceso de ciencia de datos en equipos

En este artículo se describen los objetivos, las tareas y los resultados asociados a la implementación del Proceso de ciencia de datos en equipo (TDSP). Este proceso proporciona un ciclo de vida recomendado que puede usar para estructurar los proyectos de ciencia de datos. El ciclo de vida describe las fases principales por las que pasan normalmente los proyectos, a menudo de forma iterativa:

   1. **Conocimiento del negocio**
   2. **Adquisición y comprensión de los datos**
   3. **Modelado**
   4. **Implementación** (este artículo)
   5. **Aceptación del cliente**

Para información general y una representación visual del ciclo de vida de TDSP, consulte [Ciclo de vida del proceso de ciencia de datos en equipo](./lifecycle.md).

## <a name="goal"></a>Objetivo
Implemente modelos con canalización de datos en un entorno de producción o similar para que el usuario final los acepte. 

## <a name="how-to-do-it"></a>Modo de hacerlo
La tarea principal que se aborda en esta fase es la siguiente:

**Uso del modelo**: implemente el modelo y la canalización en un entorno de producción o semejante para el consumo de aplicaciones.

### <a name="operationalize-a-model"></a>Uso de modelos
Cuando ya disponga de un conjunto de modelos que funcionan bien, los puede hacer operativos para que los consuman otras aplicaciones. Dependiendo de los requisitos empresariales, se realizan predicciones en tiempo real o por lotes. Para implementar modelos, los expone con una interfaz de API abierta. La interfaz permite que el modelo se utilice fácilmente por diferentes aplicaciones, como las siguientes:

   * Sitios web en línea
   * Hojas de cálculo 
   * Paneles
   * Aplicaciones de línea de negocio 
   * Aplicaciones de back-end 

Para obtener información sobre cómo crear e implementar un servicio web de Azure Machine Learning, consulte [Implementación de un servicio web Azure Machine Learning](../classic/deploy-a-machine-learning-web-service.md). Es un procedimiento recomendado compilar la telemetría y la supervisión en el modelo de producción y la canalización de datos que se implementan. Este procedimiento ayuda con las posteriores tareas de solución de problemas e informes de estado del sistema.  

## <a name="artifacts"></a>Artifacts

* Un panel de estado que muestra el estado del sistema y métricas clave.
* Un informe de modelado final con detalles de implementación.
* Un documento de arquitectura de la solución final.


## <a name="next-steps"></a>Pasos siguientes

Estos son los vínculos a cada uno de los pasos del ciclo de vida del Proceso de ciencia de datos en equipo:

   1. [Conocimiento del negocio](lifecycle-business-understanding.md)
   2. [Adquisición y comprensión de los datos](lifecycle-data.md)
   3. [Modelado](lifecycle-modeling.md)
   4. [Implementación](lifecycle-deployment.md)
   5. [Aceptación del cliente](lifecycle-acceptance.md)

Proporcionamos tutoriales completos que muestran todos los pasos del proceso en escenarios concretos. El artículo con [tutoriales de ejemplo](walkthroughs.md) proporciona una lista de los escenarios con vínculos y descripciones de miniatura. En los tutoriales se muestra cómo combinar servicios y herramientas en la nube y locales en un flujo de trabajo o una canalización con el fin de crear una aplicación inteligente. 

Para obtener ejemplos de cómo ejecutar pasos en TDSP que usan Microsoft Azure Machine Learning Studio, consulte [Uso del Proceso de ciencia de los datos en equipos con Azure Machine Learning](./index.yml).