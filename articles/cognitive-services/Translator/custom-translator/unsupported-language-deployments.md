---
title: 'Implementaciones de idiomas no compatibles: Traductor personalizado'
titleSuffix: Azure Cognitive Services
description: En este artículo se muestra cómo implementar pares de idiomas no admitidos en Traductor personalizado de Azure Cognitive Services.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 04/24/2019
ms.author: lajanuar
ms.openlocfilehash: ca2fab82dd08ea999186a430b78b6329c64ae816
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111409602"
---
# <a name="unsupported-language-deployments"></a>Implementaciones en lenguajes no compatibles

<!--Custom Translator provides the highest-quality translations possible using the latest techniques in neural machine learning. While Microsoft intends to make neural training available in all languages, there are some limitations that prevent us from being able to offer neural machine translation in all language pairs.-->  

Con la próxima retirada de Microsoft Translator Hub, Microsoft anulará la implementación de todos los modelos implementados actualmente a través de la instancia de Hub. Muchos de ustedes tienen modelos implementados en la instancia de Hub cuyos pares de idiomas no se admiten en Traductor personalizado.  No queremos que los usuarios en esta situación carezcan de recursos para traducir su contenido.

Ahora tenemos un proceso que le permite implementar sus modelos no compatibles a través de Traductor personalizado.  Este proceso le permite continuar traduciendo contenido mediante la API V3 más reciente.  Estos modelos se hospedarán hasta que decida anular su implementación o el par de idiomas esté disponible en Traductor personalizado.  En este artículo se explica el proceso de implementación de modelos con pares de idiomas no compatibles.

## <a name="prerequisites"></a>Requisitos previos

Para que sus modelos sean candidatos para la implementación, deben cumplir los siguientes criterios:
* El proyecto que incluye el modelo debe haber migrado de la instancia de Hub a Traductor personalizado mediante la herramienta de migración.  
* El modelo debe tener el estado Implementado al producirse la migración.  
* El par de idiomas del modelo debe ser un par de idiomas no compatible en Traductor personalizado.  Los pares de idiomas en los que se admite un idioma, hacia o desde el inglés, pero el par en sí no incluye el inglés, son candidatos para las implementaciones de idiomas no compatibles.  Por ejemplo, un modelo de Hub para un par de idiomas de francés a alemán se considera un par de idiomas no compatible, aunque francés a inglés e inglés a alemán son pares de idiomas compatibles.

## <a name="process"></a>Proceso
Una vez que haya migrado los modelos de la instancia de Hub que sean candidatos para la implementación, los encontrará yendo a la página **Configuración** para su área de trabajo y desplazamiento al final de la página donde verá la sección sobre **entrenamientos de Translator Hub no admitidos**.  Solo aparecerá esta sección si tiene proyectos que cumplen los requisitos previos mencionados anteriormente.

![Captura de pantalla que resalta la sección de entrenamientos no admitidos del centro de traductores.](media/unsupported-language-deployments/unsupported-translator-hub-trainings.jpg)

En la página de selección de **entrenamientos de Translator Hub no admitidos**, la pestaña **Unrequested trainings** (Entrenamientos no solicitados) incluye modelos que cumplen los requisitos para la implementación.  Seleccione los modelos que desea implementar y envíe una solicitud.   Antes del 30 de abril, fecha límite de la implementación, podrá seleccionar tantos modelos como desee para la implementación.
 
![Captura de pantalla que muestra la pestaña de entrenamientos no solicitados.](media/unsupported-language-deployments/unsupported-translator-hub-trainings-list.jpg)

Después del envío, el modelo dejará de estar disponible en la pestaña **Unrequested trainings** (Entrenamientos no solicitados) y aparecerá, en su lugar, en la pestaña **Requested trainings** (Entrenamientos solicitados).  Podrá ver sus entrenamientos solicitados en cualquier momento.

![Migración desde Microsoft Translator Hub](media/unsupported-language-deployments/request-unsupported-trainings.jpg) 

## <a name="whats-next"></a>¿Qué sigue?

Los modelos que seleccionó para la implementación se guardarán una vez que se retire la instancia de Hub y se anule la implementación de todos los modelos.  Hasta el 24 de mayo podrá enviar solicitudes de implementación de modelos no compatibles.  Implementaremos dichos modelos el 15 de junio, momento en el que se podrá tener acceso a ellos a través de la API de Translator V3.  Además, estarán disponibles a través de la API V2 hasta el 1 de julio.  

Para obtener información adicional sobre fechas importantes relativas al desuso de la instancia de Hub, consulte [aquí](https://www.microsoft.com/translator/business/hub/).
Una vez realizada la implementación, se aplicarán cargos de hospedaje normales.  Consulte [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/) para obtener detalles.  

A diferencia de los modelos de Traductor personalizado estándar, los modelos de Hub solo estarán disponibles en una sola región, por lo que no se aplicarán cargos de hospedaje en varias regiones.  Una vez realizada la implementación, podrá anular la implementación del modelo de Hub en cualquier momento a través del proyecto de Traductor personalizado migrado.

## <a name="next-steps"></a>Pasos siguientes

- [Entrenamiento de un modelo](how-to-train-model.md).
- Comience a utilizar el modelo de traducción personalizado implementado mediante [Microsoft Translator Text API V3](../reference/v3-0-translate.md?tabs=curl).
