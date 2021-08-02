---
title: Conocimiento del negocio en el Proceso de ciencia de datos en equipos
description: Los objetivos, las tareas y los resultados de la fase de descripción del negocio de los proyectos de ciencia de datos en el Proceso de ciencia de datos en equipos.
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
ms.openlocfilehash: 2764568d117783dbc22f4c166c023bf3938987d8
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111814279"
---
# <a name="the-business-understanding-stage-of-the-team-data-science-process-lifecycle"></a>Fase de descripción de negocio del ciclo de vida del Proceso de ciencia de datos en equipo

En este artículo se describen los objetivos, las tareas y los resultados asociados a la fase de descripción del negocio del Proceso de ciencia de datos en equipo (TDSP). Este proceso proporciona un ciclo de vida recomendado que puede usar para estructurar los proyectos de ciencia de datos. El ciclo de vida describe las fases principales por las que pasan normalmente los proyectos, a menudo de forma iterativa:

   1. **Conocimiento del negocio** (este artículo)
   2. **Adquisición y comprensión de los datos**
   3. **Modelado**
   4. **Implementación**
   5. **Aceptación del cliente**

Para información general y una representación visual del ciclo de vida de TDSP, consulte [Ciclo de vida del proceso de ciencia de datos en equipo](./lifecycle.md).

## <a name="goals"></a>Objetivos
* Especifique las variables principales que sirven como objetivos del modelo y cuyas métricas asociadas se utilizan para determinar el éxito del proyecto.
* Identifique los orígenes de datos pertinentes a los que tiene acceso la empresa o que necesita obtener.

## <a name="how-to-do-it"></a>Modo de hacerlo
En esta fase se abordan dos tareas principales: 

   * **Definición de objetivos**: trabaje con el cliente y con otras partes interesadas para comprender e identificar los problemas de la empresa. Formule preguntas que definan los objetivos empresariales y a las que puedan aplicarse las técnicas de ciencia de datos.
   * **Identifique los orígenes de datos**: busque los datos pertinentes que lo ayuden a responder a las preguntas que definen los objetivos del proyecto.

### <a name="define-objectives"></a>Definición de objetivos
1. Un objetivo fundamental de este paso consiste en identificar las principales variables empresariales que el análisis deberá predecir. Estas variables se denominan *objetivos del modelo* y las métricas asociadas a ellas se utilizan para determinar el éxito del proyecto. Dos ejemplos de estos destinos son la previsión de ventas o la probabilidad de que un pedido sea fraudulento.

2. Para definir los objetivos del proyecto, plantee y ajuste preguntas "certeras" que sean pertinentes, específicas y sin ambigüedad alguna. La ciencia de datos es un proceso que utiliza nombres y números para responder a estas preguntas. La ciencia de datos o el aprendizaje automático suelen utilizarse para responder a cinco tipos de preguntas:
 
   * ¿Cuánto? o ¿cuántos? (regresión)
   * ¿Qué categoría? (clasificación)
   * ¿Qué grupo? (agrupación en clústeres)
   * ¿Es extraño? (detección de anomalías)
   * ¿Qué opción se debe elegir? (recomendación)

   Determine cuál de las siguientes es su pregunta y cómo la respuesta logra sus objetivos empresariales.

3. Defina el equipo del proyecto especificando los roles y las responsabilidades de sus miembros. Desarrolle un plan general de hitos que se pueda repetir a medida que se descubra más información. 

4. Defina las métricas del éxito. Por ejemplo, podría desear una predicción sobre la renovación de los clientes. Necesita una tasa de precisión de "x%" al final de este proyecto de tres meses. Con estos datos, puede ofrecer promociones al cliente para mejorar la fidelización. Las métricas deben cumplir los requisitos **SMART**: 

   * **S** pecific (específicas) 
   * **M** easurable (mensurables)
   * **A** chievable (alcanzables) 
   * **R** elevant (pertinentes) 
   * **T** ime-bound (con un límite de tiempo) 

### <a name="identify-data-sources"></a>Identificación de los orígenes de datos
Identifique los orígenes de datos que contienen ejemplos conocidos de respuestas a las preguntas certeras. Busque los siguientes datos:

* Datos pertinentes para la pregunta. ¿Tenemos indicadores para medir el objetivo y las características que están relacionados con él?
* Datos que representen una medida precisa de nuestro objetivo de modelo y de las características de interés.

Por ejemplo, puede descubrir que los sistemas existentes tienen que recopilar y registrar otros tipos de datos para solucionar el problema y alcanzar los objetivos del proyecto. En esta situación, puede ser conveniente buscar orígenes de datos externos o actualizar los sistemas para recopilar datos nuevos.

## <a name="artifacts"></a>Artifacts
Estos son los resultados de esta fase:

   * [Documento marco](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Project/Charter.md): se ofrece una plantilla estándar en la definición de estructura del proyecto de TDSP. El documento marco es un documento en cambio continuo. La plantilla se actualiza a lo largo del proyecto a medida que se descubren nuevos elementos y cambian las necesidades empresariales. La clave consiste en realizar iteraciones de este documento e incorporarle la información oportuna según se avance a lo largo del proceso de descubrimiento. Es importante que el cliente y las demás partes interesadas se impliquen en la realización de cambios y que se les informe claramente sobre las razones que los motivan.  
   * [Orígenes de datos](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Data_Report/Data%20Defintion.md#raw-data-sources): la sección de **orígenes de datos sin procesar** del informe de **definiciones de datos** que se encuentra en la carpeta de **informe de datos** del proyecto TDSP contiene los orígenes de datos. En esta sección se especifican las ubicaciones originales y de destino para los datos sin procesar. En las fases posteriores, deberá rellenar otros detalles, tales como los scripts para mover los datos al entorno de análisis.  
   * [Diccionarios de datos](https://github.com/Azure/Azure-TDSP-ProjectTemplate/tree/master/Docs/Data_Dictionaries): este documento proporciona descripciones de los datos facilitados por el cliente. Estas descripciones incluyen información sobre el esquema (tipos de datos e información sobre las reglas de validación, si hay) y los diagramas de relación de entidades, si están disponibles.

## <a name="next-steps"></a>Pasos siguientes

Estos son los vínculos a cada uno de los pasos del ciclo de vida del Proceso de ciencia de datos en equipo:

   1. [Conocimiento del negocio](lifecycle-business-understanding.md)
   2. [Adquisición y comprensión de los datos](lifecycle-data.md)
   3. [Modelado](lifecycle-modeling.md)
   4. [Implementación](lifecycle-deployment.md)
   5. [Aceptación del cliente](lifecycle-acceptance.md)

Proporcionamos tutoriales completos que muestran todos los pasos del proceso en escenarios concretos. El artículo con [tutoriales de ejemplo](walkthroughs.md) proporciona una lista de los escenarios con vínculos y descripciones de miniatura. En los tutoriales se muestra cómo combinar servicios y herramientas en la nube y locales en un flujo de trabajo o una canalización con el fin de crear una aplicación inteligente. 
