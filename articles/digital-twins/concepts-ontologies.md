---
title: ¿Qué es una ontología?
titleSuffix: Azure Digital Twins
description: Más información sobre ontologías del sector basadas en DTDL para el modelado en0 un determinado dominio
author: baanders
ms.author: baanders
ms.date: 2/12/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: ba261609772e380e85de084096a3242f2f92fd49
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108734316"
---
# <a name="what-is-an-ontology"></a>¿Qué es una ontología? 

El vocabulario de una solución de Azure Digital Twins se define mediante [modelos](concepts-models.md) que describen los tipos de entidad que existen en su entorno.

A veces, cuando la solución está asociada a un sector determinado, puede ser más fácil y eficaz comenzar con un conjunto de modelos para ese sector que ya exista en lugar de crear su propio modelo desde cero. Estos conjuntos de modelos ya existentes se denominan **ontologías**. 

En general, una ontología es un conjunto de modelos para un dominio determinado, como la estructura de un edificio, un sistema de IoT, una ciudad inteligente, la red eléctrica, el contenido web, etc. Las ontologías se usan a menudo como esquemas para grafos de gemelos, ya que permiten:
* La armonización de componentes de software, documentación, bibliotecas de consulta, etc.
* Una inversión reducida en el modelado conceptual y el desarrollo del sistema
* Una interoperabilidad de datos más sencilla en un nivel semántico
* La reutilización de procedimientos recomendados, en lugar de empezar desde cero o "reinventar la rueda"

En este artículo se explica por qué y cómo usar ontologías para los modelos de Azure Digital Twins así como qué ontologías y herramientas están disponibles hoy en día.

## <a name="using-ontologies-for-azure-digital-twins"></a>Uso de ontologías para Azure Digital Twins

Las ontologías proporcionan un excelente punto de partida para las soluciones de gemelos digitales. Incluyen un conjunto de modelos específicos del dominio y relaciones entre entidades para diseñar, crear y analizar un grafo de gemelo digital. Las ontologías permiten a los desarrolladores de soluciones comenzar una solución de gemelos digitales desde un punto de partida probado y centrarse en resolver problemas empresariales. Los ontologías que proporciona Microsoft también están diseñadas para ser fácilmente extensibles, por lo que puede personalizarlas para su solución. 

Además, el uso de estas ontologías en las soluciones permite configurarlas para una integración más fluida entre distintos asociados y proveedores, ya que las ontologías pueden proporcionar un vocabulario común entre soluciones.

Dado que los modelos de Azure Digital Twins se representan en el [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md), las ontologías para su uso con Azure Digital Twins también se escriben en DTDL. 

## <a name="strategies-for-integrating-ontologies"></a>Estrategias para la integración de ontologías

Hay tres posibles estrategias para la integración de ontologías estándar del sector con DTDL. Puede elegir la que mejor se adapte a sus necesidades:

| Estrategia | Descripción | Recursos |
| --- | --- | --- |
| **Adoptar** | puede iniciar la solución con una ontología de DTDL de código abierto que se ha creado a partir de estándares del sector profusamente adoptados. Puede usar estos conjuntos de modelos directamente o extenderlos con sus propias adiciones para una solución personalizada. | [Conceptos:&nbsp;Adopción de&nbsp;ontologías&nbsp;estándar del sector](concepts-ontologies-adopt.md)<br><br>[Conceptos:&nbsp;Extensión de&nbsp;ontologías](concepts-ontologies-extend.md) |
| **Convertir** | Si ya tiene modelos existentes representados en otro formato estándar, puede convertirlos en DTDL para usarlos con Azure Digital Twins. | [Conceptos:&nbsp;Conversión de&nbsp;ontologías](concepts-ontologies-convert.md)<br><br>[Conceptos:&nbsp;Extensión de&nbsp;ontologías](concepts-ontologies-extend.md) |
| **Autor** | Siempre puede desarrollar sus propios modelos de DTDL personalizados desde cero, mediante cualquier estándar del sector aplicable como inspiración. | [Conceptos: Modelos de DTDL](concepts-models.md) |

### <a name="using-ontology-strategies-in-a-model-development-path"></a>Uso de estrategias de ontología en un procedimiento de desarrollo de modelos

Independientemente de la estrategia que elija para integrar una ontología en Azure Digital Twins, puede seguir el procedimiento completo que se muestra a continuación para que le oriente en la creación y carga de ontologías como modelos de DTDL.

1. Empiece por revisar y comprender el [modelado de DTDL en Azure Digital Twins](concepts-models.md).
1. Continúe con la estrategia de integración de ontologías elegida anteriormente: [Adopte](concepts-ontologies-adopt.md), [convierta](concepts-ontologies-convert.md) o [cree](concepts-models.md) los modelos según su ontología.
    1. Si es necesario, [extienda](concepts-ontologies-extend.md) su ontología para personalizarla según sus necesidades.
1. [Valide](how-to-parse-models.md) los modelos para comprobar que son documentos DTDL operativos.
1. Cargue los modelos terminados en Azure Digital Twins, mediante las [API](how-to-manage-model.md#upload-models) o un ejemplo como el del [cargador de modelos de Azure Digital Twins](https://github.com/Azure/opendigitaltwins-building-tools/tree/master/ModelUploader).

Después, debería poder usar los modelos en la instancia de Azure Digital Twins. 

Puede visualizarlos con ejemplos como el de [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) o el [visualizador de modelos de Azure Digital Twins](https://github.com/Azure/opendigitaltwins-building-tools/tree/master/AdtModelVisualizer), o bien pasar a usarlos para crear [gemelos digitales](concepts-twins-graph.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las estrategias de adopción, conversión y creación de ontologías:
* [Conceptos: Adopción de ontologías estándar del sector](concepts-ontologies-adopt.md)
* [Conceptos: Conversión de ontologías](concepts-ontologies-convert.md)
* [Procedimientos: Administración de modelos DTDL](how-to-manage-model.md)

O bien, obtenga información sobre cómo se usan los modelos para crear gemelos digitales: [Conceptos: Gemelos digitales y grafo de gemelos](concepts-twins-graph.md).