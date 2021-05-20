---
title: Adopción de ontologías estándar del sector
titleSuffix: Azure Digital Twins
description: Obtenga información sobre las ontologías del sector existentes que se pueden adoptar para Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 2/26/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 589e7ea53f6f247d18d4a2b532af3c87a8b0230c
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109787894"
---
# <a name="adopting-an-industry-ontology"></a>Adopción de una ontología del sector

Como es más fácil comenzar con una ontología de DTDL de código abierto que empezar de cero, Microsoft se está asociando con expertos de dominio para publicar ontologías que representan convenciones del sector ampliamente aceptadas y admiten diversos casos de uso de clientes. 

El resultado es un conjunto de ontologías de código abierto basadas en DTDL que aprenden de las normas estándar del sector, están basadas en ellas o las utilizan directamente. Las ontologías están diseñadas para satisfacer las necesidades de los desarrolladores que se encuentran en un nivel inferior, con la posibilidad de que el sector las adopte o extienda posteriormente según sus necesidades.

En este momento, Microsoft está trabajando con asociados para desarrollar una ontología para [edificios inteligentes](#realestatecore-smart-building-ontology) y otra para [ciudades inteligentes](#smart-cities-ontology) que proporciona una base común para el modelado basado en los estándares de estos sectores para evitar la reinvención. 

## <a name="realestatecore-smart-building-ontology"></a>Ontología RealEstateCore para edificios inteligentes

*Obtenga la ontología en el repositorio siguiente:* [Ontología RealEstateCore basada en el lenguaje de definición de Digital Twins para edificios inteligentes](https://github.com/Azure/opendigitaltwins-building).

Microsoft se ha asociado con [RealEstateCore](https://www.realestatecore.io/), un consorcio sueco de propietarios de bienes inmuebles, proveedores de software e instituciones de investigación, para ofrecer una ontología basada en el lenguaje de definición de Digital Twins y de código abierto para el sector inmobiliario.

Esta ontología para edificios inteligentes proporciona una base común para el modelado de edificios inteligentes mediante estándares del sector (como el  [esquema de BRICK](https://brickschema.org/ontology/) o la  [ontología de la topología para edificios W3C](https://w3c-lbd-cg.github.io/bot/index.html)) para evitar la reinvención. La ontología también incluye procedimientos recomendados para usarla y extenderla correctamente. 

Para más información sobre las convenciones de modelado y la estructura de la ontología, cómo usarla, extenderla y colaborar, visite el repositorio de la ontología en GitHub: [Azure/opendigitaltwins-building](https://github.com/Azure/opendigitaltwins-building). 

También puede leer más sobre la asociación con RealEstateCore y los objetivos de esta iniciativa en la siguiente entrada de blog y en el vídeo adjunto: [Ya está disponible RealEstateCore, una ontología para edificios inteligentes para gemelos digitales](https://techcommunity.microsoft.com/t5/internet-of-things/realestatecore-a-smart-building-ontology-for-digital-twins-is/ba-p/1914794).

## <a name="smart-cities-ontology"></a>Ontology para las ciudades inteligentes

*Obtenga la ontología en el repositorio siguiente:* [Ontología del lenguaje de definición de Digital Twins (DTDL) para ciudades inteligentes](https://github.com/Azure/opendigitaltwins-smartcities).

Microsoft ha colaborado con [Open Agile Smart Cities (OASC)](https://oascities.org/) y [Sirus](https://sirus.be/) para proporcionar una ontología basada en el lenguaje de definición de Digital Twins para ciudades inteligentes que empieza por [ETSI CIM NGSI-LD](https://www.etsi.org/committee/cim). Además de ETSI NGSI-LD, también hemos evaluado Saref4City, CityGML, ISO y otras.

La versión actual de la ontología se centra en un conjunto inicial de modelos. Los autores de la ontología le agradecen que contribuyan a ampliar el conjunto inicial de casos de uso y a mejorar los modelos existentes. 

Para más información sobre la ontología, cómo usarla y colaborar, visite el repositorio de la ontología en GitHub: [Azure/opendigitaltwins-smartcities](https://github.com/Azure/opendigitaltwins-smartcities). 

Para más información sobre las asociaciones y el enfoque para ciudades inteligentes, consulte esta entrada de blog y el vídeo que la acompaña: [Ontología de ciudades inteligentes para Digital Twins](https://techcommunity.microsoft.com/t5/internet-of-things/smart-cities-ontology-for-digital-twins/ba-p/2166585).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre cómo extender las ontologías estándar del sector para que satisfagan sus especificaciones: [Conceptos: Extensión de las ontologías del sector](concepts-ontologies-extend.md).

* O bien, continúe el proceso de aprendizaje para desarrollar modelos basados en ontologías: [Uso de estrategias de ontología en un procedimiento de desarrollo de modelos](concepts-ontologies.md#using-ontology-strategies-in-a-model-development-path).