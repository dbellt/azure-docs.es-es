---
title: Opciones de desarrollo de Azure Cognitive Services
description: Obtenga información acerca de cómo usar Azure Cognitive Services con diferentes opciones de desarrollo e implementación, como bibliotecas cliente, API REST, Logic Apps, Power Automate, Azure Functions, Azure App Service, Azure Databricks y muchas más.
services: cognitive-services
manager: nitinme
author: erhopf
ms.author: erhopf
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: c6c09389628e1a9648d827eb5562a06c4f746db2
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370949"
---
# <a name="cognitive-services-development-options"></a>Opciones de desarrollo de Cognitive Services

En este documento se proporciona información general sobre las opciones de desarrollo e implementación para ayudarle a empezar a trabajar con Azure Cognitive Services.

Azure Cognitive Services son servicios de inteligencia artificial basados en la nube que permiten a los desarrolladores incluir inteligencia en sus aplicaciones y productos sin conocimientos profundos sobre aprendizaje automático. Con Cognitive Services, tiene acceso a las funcionalidades y modelos de inteligencia artificial creados, entrenados y actualizados por Microsoft, listos para su uso en las aplicaciones. En muchos casos, también tiene la opción de personalizar los modelos en función de sus necesidades empresariales. 

Los servicios de Cognitive Services se organizan en cuatro categorías: Decisión, idioma, voz y visión. Normalmente se tiene acceso a estos servicios mediante API REST, bibliotecas cliente y herramientas personalizadas (como las interfaces de la línea de comandos) proporcionadas por Microsoft. Sin embargo, esto es solo un camino hacia el éxito. Con Azure, también tiene acceso a varias opciones de desarrollo, como:

* Herramientas de automatización e integración como Logic Apps y Power Automate.
* Opciones de implementación, como Azure Functions y App Service. 
* Contenedores de Cognitive Services para el acceso seguro.
* Herramientas como Apache Spark, Azure Databricks, Azure Synapse Analytics y Azure Kubernetes Service para macrodatos. 

Antes de comenzar, es importante saber que Cognitive Services se usa principalmente para dos tareas distintas. En función de la tarea que desee realizar, tiene distintas opciones de desarrollo e implementación entre las que puede elegir. 

* [Opciones de desarrollo para la predicción y el análisis](#development-options-for-prediction-and-analysis)
* [Herramientas para personalizar y configurar modelos](#tools-to-customize-and-configure-models)

## <a name="development-options-for-prediction-and-analysis"></a>Opciones de desarrollo para la predicción y el análisis 

Las herramientas que usará para personalizar y configurar modelos son diferentes de las que usará para llamar a los servicios de Cognitive Services. De forma integrada, la mayoría de los servicios de Cognitive Services permiten enviar datos y recibir conclusiones sin ninguna personalización. Por ejemplo: 

* Puede enviar una imagen al servicio Computer Vision para detectar palabras y frases, o bien contar el número de personas que hay en el fotograma.
* Puede enviar un archivo de audio al servicio de Voz y obtener transcripciones y traducir la voz a texto al mismo tiempo.
* Puede enviar un archivo PDF al servicio Form Recognizer y detectar tablas, celdas y texto dentro de esas celdas, y obtener una salida JSON con coordenadas y detalles.

Azure ofrece una amplia gama de herramientas que están diseñadas para diferentes tipos de usuarios, muchas de las cuales se pueden usar con Cognitive Services. Las herramientas controladas por diseñadores son las más fáciles de usar y son rápidas de configurar y automatizar, pero pueden tener limitaciones en cuanto a la personalización. Nuestras API REST y las bibliotecas cliente proporcionan a los usuarios más control y flexibilidad, pero requieren más esfuerzo, tiempo y experiencia para crear una solución. Si usa las API REST y las bibliotecas cliente, se espera que esté familiarizado con el trabajo con lenguajes de programación modernos como C#, Java, Python, JavaScript u otro lenguaje de programación popular. 

Echemos un vistazo a las diferentes formas en las que puede trabajar con Cognitive Services.

### <a name="client-libraries-and-rest-apis"></a>Bibliotecas cliente y API REST

Las bibliotecas cliente y las API REST de Cognitive Services proporcionan acceso directo al servicio. Estas herramientas proporcionan acceso mediante programación a los servicios de Cognitive Services, sus modelos de línea base y, en muchos casos, permiten personalizar los modelos y las soluciones mediante programación. 

* **Usuarios de destino**: desarrolladores y científicos de datos
* **Ventajas** proporciona la máxima flexibilidad para llamar a los servicios desde cualquier lenguaje y entorno. 
* **Interfaz de usuario**: N/A: solo código
* **Suscripciones**: cuenta de Azure + recursos de Cognitive Services

Si desea más información acerca de las bibliotecas cliente y las API REST disponibles, use [Documentación de Azure Cognitive Services](index.yml) para elegir un servicio y comenzar a trabajar con una de nuestras guías de inicio rápido para visión, decisión, idioma y voz.

### <a name="cognitive-services-for-big-data"></a>Cognitive Services para macrodatos

Con Cognitive Services para macrodatos, puede integrar los modelos inteligentes de mejora continua directamente en cálculos de Apache Spark y SQL. Estas herramientas liberan a los desarrolladores de las tareas de red de bajo nivel, de modo que puedan centrarse en la creación de aplicaciones distribuidas inteligentes. Cognitive Services para macrodatos admite las siguientes plataformas y conectores: Azure Databricks, Azure Synapse, Azure Kubernetes Service y conectores de datos.

* **Usuarios de destino**: científicos de datos e ingenieros de datos
* **Ventajas** Azure Cognitive Services para macrodatos permite a los usuarios canalizar terabytes de datos mediante Cognitive Services con Apache Spark&trade;. Es sencillo crear aplicaciones inteligentes a gran escala con cualquier almacén de datos.
* **Interfaz de usuario**: N/A: solo código
* **Suscripciones**: cuenta de Azure + recursos de Cognitive Services

Si desea más información sobre macrodatos para Cognitive Services, [Azure Cognitive Services para macrodatos](./big-data/cognitive-services-for-big-data.md) es un buen punto de partida. Si está listo para empezar a crear, pruebe los ejemplos de [Python](./big-data/samples-python.md) o [Scala](./big-data/samples-scala.md).

### <a name="azure-functions-and-azure-service-web-jobs"></a>Azure Functions y trabajos web de Azure App Service

[Azure Functions](../azure-functions/index.yml) y los trabajos web de [Azure App Service](../app-service/index.yml) proporcionan servicios de integración orientados a código diseñados para desarrolladores y se basan en [Azure App Services](../app-service/index.yml). Estos productos proporcionan infraestructura sin servidor para escribir código. Dentro de ese código puede realizar llamadas a nuestros servicios con las bibliotecas cliente y las API REST. 

* **Usuarios de destino**: desarrolladores y científicos de datos
* **Ventajas** servicio de proceso sin servidor que permite ejecutar código desencadenado por eventos. 
* **Interfaz de usuario**: Sí
* **Suscripciones**: cuenta de Azure + recurso de Cognitive Services + suscripción de Azure Functions

### <a name="azure-logic-apps"></a>Azure Logic Apps 

[Azure Logic Apps](../logic-apps/index.yml) comparte el mismo diseñador de flujo de trabajo y los mismos conectores que Power Automate, pero proporciona un control más avanzado, incluidas las integraciones con Visual Studio y DevOps. Power Automate facilita la integración con los recursos de Cognitive Services mediante conectores específicos del servicio que proporcionan un servidor proxy o un contenedor en torno a las API. Estos son los mismos conectores que están disponibles en Power Automate. 

* **Usuarios de destino**: desarrolladores, integradores, profesionales de TI, DevOps
* **Ventajas** modelo de desarrollo orientado a diseñador (declarativo) que proporciona opciones avanzadas e integración en una solución con poco código
* **Interfaz de usuario**: Sí
* **Suscripciones**: cuenta de Azure + recurso de Cognitive Services + implementación de Logic Apps

### <a name="power-automate"></a>Power Automate 

Power Automate es un servicio de [Power Platform](/power-platform/) que le ayuda a crear flujos de trabajo automatizados entre aplicaciones y servicios sin escribir código. Ofrecemos varios conectores para facilitar la interacción con el recurso de Cognitive Services en una solución de Power Automate. Power Automate se compila a partir de Logic Apps. 

* **Usuarios de destino**: usuarios empresariales (analistas) y administradores de SharePoint
* **Ventajas** automatice las tareas manuales repetitivas con solo grabar clics del ratón, pulsaciones de teclas y pasos de copiar y pegar desde el escritorio.
* **Herramientas de la interfaz de usuario**: sí, solo interfaz de usuario
* **Suscripciones**: cuenta de Azure + recurso de Cognitive Services + suscripción de Power Automate + suscripción de Office 365

### <a name="ai-builder"></a>AI Builder 

[AI Builder](/ai-builder/overview) es una funcionalidad de Microsoft Power Platform que puede usar para mejorar el rendimiento empresarial mediante la automatización de los procesos y la predicción de los resultados. AI Builder aporta la tecnología de la inteligencia artificial a sus soluciones mediante una experiencia de tipo apuntar y hacer clic. Muchos servicios cognitivos, como Form Recognizer, Text Analytics y Computer Vision, se han integrado directamente aquí y no es necesario crear su propio servicio de Cognitive Services. 

* **Usuarios de destino**: usuarios empresariales (analistas) y administradores de SharePoint
* **Ventajas** una solución llave en mano que aporta la tecnología de la inteligencia artificial mediante una experiencia de tipo apuntar y hacer clic. No se requieren conocimientos de codificación ni de ciencia de datos.
* **Herramientas de la interfaz de usuario**: sí, solo interfaz de usuario
* **Suscripciones**: AI Builder

### <a name="continuous-integration-and-deployment"></a>Integración e implementación continuas

Puede usar Azure DevOps y Acciones de GitHub para administrar las implementaciones. En la [sección siguiente](#continuous-integration-and-delivery-with-devops-and-github-actions), en la que se describe esto, tenemos dos ejemplos de integraciones de CI/CD para entrenar e implementar modelos personalizados para Voz y el servicio Language Understanding (LUIS). 

* **Usuarios de destino**: desarrolladores, científicos de datos e ingenieros de datos
* **Ventajas** permite ajustar, actualizar e implementar continuamente aplicaciones y modelos mediante programación. El uso periódico de los datos para mejorar y actualizar los modelos de voz, visión, idioma y decisión es una ventaja significativa. 
* **Herramientas de la interfaz de usuario**: N/A: solo código 
* **Suscripciones**: cuenta de Azure + recursos de Cognitive Services + cuenta de GitHub

## <a name="tools-to-customize-and-configure-models"></a>Herramientas para personalizar y configurar modelos

A medida que avance en el viaje de la creación de una aplicación o un flujo de trabajo con Cognitive Services, es posible que necesite personalizar el modelo para lograr el rendimiento deseado. Muchos de nuestros servicios le permiten basarse en los modelos pregenerados para satisfacer sus necesidades empresariales específicas. Para todos nuestros servicios personalizables, proporcionamos una experiencia basada en la interfaz de usuario para recorrer el proceso, así como las API para el entrenamiento controlado por código. Por ejemplo:

* Desea entrenar un modelo de Habla personalizada para reconocer correctamente términos médicos con una tasa de errores de palabras (WER) por debajo del 3 %.
* Desea crear un clasificador de imágenes con Custom Vision que pueda indicar la diferencia entre coníferas y árboles caducifolios.
* Desea crear una voz neuronal personalizada con sus datos de voz personales para mejorar la experiencia automatizada del cliente.

Las herramientas que usará para entrenar y configurar modelos son diferentes de las que usará para llamar a los servicios de Cognitive Services. En muchos casos, los servicios de Cognitive Services que admiten personalización proporcionan portales y herramientas de la interfaz de usuario diseñadas para ayudarle a entrenar, evaluar e implementar modelos. Echemos un vistazo rápido a algunas opciones:<br><br>

| Fundamento | Servicio | IU de personalización | Inicio rápido |
|--------|---------|------------------|------------|
| Visión | Custom Vision | https://www.customvision.ai/ | [Guía de inicio rápido](./custom-vision-service/quickstarts/image-classification.md?pivots=programming-language-csharp) | 
| Visión | Form Recognizer | [Herramienta de etiquetado de ejemplo](https://aka.ms/fott-2.1-ga) | [Artículo de procedimientos](./form-recognizer/label-tool.md?tabs=v2-0) |
| Decisión | Content Moderator | https://contentmoderator.cognitive.microsoft.com/dashboard | [Guía de inicio rápido](./content-moderator/review-tool-user-guide/human-in-the-loop.md) |
| Decisión | Metrics Advisor | https://metricsadvisor.azurewebsites.net/  | [Guía de inicio rápido](./metrics-advisor/quickstarts/web-portal.md) |
| Decisión | Personalizer | La interfaz de usuario está disponible en Azure Portal en el recurso de Personalizer. | [Guía de inicio rápido](./personalizer/quickstart-personalizer-sdk.md) |
| Idioma | Language Understanding (LUIS) | https://www.luis.ai/ | |
| Idioma | QnA Maker | https://www.qnamaker.ai/ | [Guía de inicio rápido](./qnamaker/quickstarts/create-publish-knowledge-base.md) |
| Idioma | Traductor y Traductor personalizado | https://portal.customtranslator.azure.ai/ | [Guía de inicio rápido](./translator/custom-translator/quickstart-build-deploy-custom-model.md) |
| Voz | Comandos personalizados | https://speech.microsoft.com/ | [Guía de inicio rápido](./speech-service/custom-commands.md) |
| Voz | Habla personalizada | https://speech.microsoft.com/ | [Guía de inicio rápido](./speech-service/custom-speech-overview.md) |
| Voz | Voz personalizada | https://speech.microsoft.com/ | [Guía de inicio rápido](./speech-service/how-to-custom-voice.md) |  

### <a name="continuous-integration-and-delivery-with-devops-and-github-actions"></a>Integración y entrega continuas con DevOps y Acciones de GitHub

Language Understanding y el servicio de Voz ofrecen soluciones de integración continua e implementación continua que se basan en Azure DevOps y Acciones de GitHub. Estas herramientas se usan para el entrenamiento automatizado, las pruebas y la administración de versiones de los modelos personalizados. 

* [CI/CD para Habla personalizada](./speech-service/how-to-custom-speech-continuous-integration-continuous-deployment.md)
* [CI/CD para LUIS](./luis/luis-concept-devops-automation.md)

## <a name="on-prem-containers"></a>Contenedores locales 

Muchos de los servicios de Cognitive Services se pueden implementar en contenedores para el acceso y uso local. El uso de estos contenedores le brinda la flexibilidad de acercar Cognitive Services a sus datos por razones de cumplimiento, seguridad u otras razones operativas. Para obtener una lista completa de contenedores de Cognitive Services, consulte [Contenedores locales para Cognitive Services](./cognitive-services-container-support.md).

## <a name="next-steps"></a>Pasos siguientes
<!--
* Learn more about low code development options for Cognitive Services -->
* [Inicio rápido: Creación de un recurso de Cognitive Services con Azure Portal](./cognitive-services-apis-create-account.md?tabs=multiservice%252clinux)
