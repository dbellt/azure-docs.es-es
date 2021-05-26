---
title: Uso de contenedores de Azure Cognitive Services a nivel local
titleSuffix: Azure Cognitive Services
description: Aprenda a usar contenedores de Docker para usar Cognitive Services de forma local.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, cog-serv-seo-aug-2020
ms.service: cognitive-services
ms.topic: article
ms.date: 05/13/2021
ms.author: aahi
keywords: local, Docker, contenedor, Kubernetes
ms.openlocfilehash: e047a96dc154ecc57226b7ecf4f673e86547fbd3
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110465063"
---
# <a name="azure-cognitive-services-containers"></a>Contenedores de Azure Cognitive Services

Azure Cognitive Services proporciona varios [contenedores de Docker](https://www.docker.com/what-container) que permiten usar las mismas API disponibles en Azure de forma local. El uso de estos contenedores le brinda la flexibilidad de acercar Cognitive Services a sus datos por razones de cumplimiento, seguridad u otras razones operativas. La compatibilidad con los contenedores está disponible actualmente para un subconjunto de servicios de Azure Cognitive Services.

> [!VIDEO https://www.youtube.com/embed/hdfbn4Q8jbo]

La creación de contenedores es un enfoque de distribución de software en el que una aplicación o servicio, incluidas sus dependencias y la configuración, se empaqueta como una imagen de contenedor. La imagen de contenedor puede implementarse en un host de contenedor con pocas o ningunas modificaciones. Los contenedores están aislados entre sí y del sistema operativo subyacente, con una superficie menor que una máquina virtual. Se pueden crear instancias de contenedores a partir de las imágenes de contenedor para las tareas a corto plazo y quitarlas cuando ya no se necesiten.

## <a name="features-and-benefits"></a>Características y ventajas

- **Infraestructura inmutable**: permita que los equipos de DevOps aprovechen un conjunto coherente y confiable de parámetros del sistema conocidos, al tiempo que pueden adaptarse a los cambios. Los contenedores proporcionan la flexibilidad para dinamizar dentro de un ecosistema predecible y evitar el desfase de la configuración.
- **Control sobre datos**: Elija dónde se procesan los datos mediante Cognitive Services. Esto puede ser esencial si no se pueden enviar datos a la nube pero es necesario tener acceso a Cognitive Services APIs. Admite la coherencia en entornos híbridos: a través de datos, administración, identidad y seguridad.
- **Control sobre las actualizaciones del modelo**: flexibilidad para el control de versiones y la actualización de los modelos implementados en sus soluciones.
- **Arquitectura portátil**: habilita la creación de una arquitectura de aplicación portátil que se pueda implementar en Azure, en el entorno local y en la red perimetral. Los contenedores se pueden implementar directamente en [Azure Kubernetes Service](../aks/index.yml), [Azure Container Instances](../container-instances/index.yml) o en un clúster de [Kubernetes](https://kubernetes.io/) implementado en [Azure Stack](/azure-stack/operator). Para obtener más información, consulte [Implementación de Kubernetes en Azure Stack](/azure-stack/user/azure-stack-solution-template-kubernetes-deploy).
- **Alto rendimiento y baja latencia**: proporcione a los clientes la capacidad de escalar para los requisitos de alto rendimiento y baja latencia permitiendo que Cognitive Services se ejecute físicamente cerca de sus datos y lógica de aplicación. Los contenedores no limitan las transacciones por segundo (TPS) y se pueden escalar tanto vertical como horizontalmente para controlar la demanda si se proporcionan los recursos de hardware necesarios.
- **Escalabilidad**: Con la popularidad cada vez mayor del software de creación y orquestación de contenedores, como Kubernetes; la escalabilidad está a la vanguardia de los avances tecnológicos. Basado en un clúster escalable, el desarrollo de aplicaciones se encarga de la alta disponibilidad.

## <a name="containers-in-azure-cognitive-services"></a>Contenedores en Azure Cognitive Services

Los contenedores de Azure Cognitive Services proporcionan el siguiente conjunto de contenedores de Docker, cada uno de los cuales incluye un subconjunto de funcionalidades de servicios de Azure Cognitive Services. Encontrará instrucciones y las ubicaciones de las imágenes en las tablas siguientes. También está disponible una lista de [imágenes de los contenedores](containers/container-image-tags.md).

### <a name="decision-containers"></a>Contenedores de decisiones

| Servicio |  Contenedor | Descripción | Disponibilidad |
|--|--|--|--|
| [Anomaly Detector][ad-containers] | **Anomaly Detector** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-decision-anomaly-detector))  | Anomaly Detector API permite supervisar y detectar anomalías en datos de serie temporal con aprendizaje automático. | Disponibilidad general |

### <a name="language-containers"></a>Contenedores de idioma

| Servicio |  Contenedor | Descripción | Disponibilidad |
|--|--|--|--|
| [LUIS][lu-containers] |  **LUIS** ([imagen](https://go.microsoft.com/fwlink/?linkid=2043204&clcid=0x409)) | Carga un modelo de Language Understanding entrenado o publicado, lo que también se conoce como aplicación de LUIS, en un contenedor de Docker y proporciona acceso a las predicciones de consulta de los puntos de conexión de la API del contenedor. Puede recopilar registros de consultas en el contenedor y cargarlos de nuevo en el [portal de LUIS](https://www.luis.ai) para mejorar la precisión de predicción de la aplicación. | Disponibilidad general |
| [Text Analytics][ta-containers-keyphrase] | **Extracción de frases clave** ([imagen](https://go.microsoft.com/fwlink/?linkid=2018757&clcid=0x409)) | Extrae las frases clave para identificar los puntos principales. Por ejemplo, si el texto de entrada es "La comida estaba deliciosa y el personal era maravilloso", la API devuelve los principales puntos de conversación: "comida" y "personal maravilloso". | Vista previa |
| [Text Analytics][ta-containers-language] |  **Detección de idioma del texto** ([imagen](https://go.microsoft.com/fwlink/?linkid=2018759&clcid=0x409)) | Se detecta el idioma (120 como máximo) en que está escrito el texto de entrada y se usa un código de idioma único para informar acerca de cada documento enviado en la solicitud. El código de idioma se empareja con una puntuación que indica la intensidad de esta. | Disponibilidad general |
| [Text Analytics][ta-containers-sentiment] | **Análisis de sentimiento, versión 3** ([imagen](https://go.microsoft.com/fwlink/?linkid=2018654&clcid=0x409)) | Analiza el texto sin formato para obtener pistas sobre opiniones positivas o negativas. Esta versión del análisis de sentimiento devuelve etiquetas de sentimiento (por ejemplo, *positivo* o *negativo*) para cada documento y oración que contiene. |  Disponibilidad general |
| [Text Analytics][ta-containers-health] |  **Text Analytics para el estado** | Extraiga y etiquete la información médica del texto clínico no estructurado. | Versión preliminar "validada". [Solicite acceso][request-access]. |
| [Traductor][tr-containers] | **Traductor** | Traduzca texto a varios idiomas y dialectos. | Versión preliminar "validada". [Solicite acceso][request-access]. | 

### <a name="speech-containers"></a>Contenedores de Speech

> [!NOTE]
> Para usar los contenedores de Voz, debe completar un [formulario de solicitud en línea](https://aka.ms/csgate).

| Servicio |  Contenedor | Descripción | Disponibilidad |
|--|--|--|
| [Speech Service API][sp-containers-stt] |  **Conversión de voz en texto** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-custom-speech-to-text)) | Permite transcribir en tiempo real voz en texto. | Disponibilidad general |
| [Speech Service API][sp-containers-cstt] | **Conversión de voz en texto personalizada** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-custom-speech-to-text)) | Permite transcribir en tiempo real voz en texto mediante un modelo personalizado. | Disponibilidad general |
| [Speech Service API][sp-containers-tts] | **Texto a voz** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-text-to-speech)) | Convierte el texto a una voz que parece natural. | Disponibilidad general |
| [Speech Service API][sp-containers-ctts] | **Conversión de texto a voz personalizada** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-custom-text-to-speech)) | Convierte el texto en una voz que parece natural mediante un modelo personalizado. | Versión preliminar "validada" |
| [Speech Service API][sp-containers-ntts] | **Texto a voz neuronal** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-neural-text-to-speech)) | Convierte texto en voz con un sonido natural utilizando una tecnología de red neuronal profunda, lo que permite obtener una voz sintetizada más natural. | Disponibilidad general |
| [Speech Service API][sp-containers-lid] | **Detección de idioma de voz** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-speechservices-language-detection)) | Determina el idioma del audio hablado. | Versión preliminar "validada" |

### <a name="vision-containers"></a>Contenedores de Vision

> [!WARNING]
> El 11 de junio de 2020 Microsoft anunció que no venderá tecnología de reconocimiento facial a los departamentos de policía de Estados Unidos hasta que se promulgue un reglamento estricto cimentado en los derechos humanos. Por lo tanto, es posible que los clientes no usen las características o la funcionalidad del reconocimiento facial incluidas en los servicios de Azure, como Face o Video Indexer, si un es un departamento de policía de Estados Unidos o permite el uso de dichos servicios por parte de cualquiera de ellos.

| Servicio |  Contenedor | Descripción | Disponibilidad |
|--|--|--|--|
| [Computer Vision][cv-containers] | **Lectura de OCR** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-vision-read)) | El contenedor Read OCR permite extraer texto impreso y manuscrito de imágenes y documentos con compatibilidad con los formatos de archivo JPEG, PNG, BMP, PDF y TIFF. Para obtener más información, consulte la [documentación de la API Read](./computer-vision/overview-ocr.md). | Versión preliminar "validada". [Solicite acceso][request-access]. |
| [Análisis espacial][spa-containers] | **Análisis espacial** ([imagen](https://hub.docker.com/_/microsoft-azure-cognitive-services-vision-spatial-analysis)) | Analiza vídeo en streaming en tiempo real para comprender las relaciones espaciales entre los usuarios, su movimiento y las interacciones con objetos de los entornos físicos. | Versión preliminar "validada". [Solicite acceso][request-access]. |
| [Face][fa-containers] | **Face** | Detecta caras humanas en imágenes e identifica atributos, incluidos faciales (como narices y ojos), sexo, edad y otras características faciales previstas por la máquina. Además de la detección, Face puede comprobar si dos caras en la misma o en diferentes imágenes son iguales mediante una puntuación de confianza, o bien comparar caras en una base de datos para ver si ya existe un aspecto similar o una cara idéntica. También puede organizar caras similares en grupos mediante rasgos visuales compartidos. | No disponible |
| [Form Recognizer][fr-containers] | **Form Recognizer** | Form Understanding aplica tecnología de aprendizaje automático para identificar y extraer pares clave-valor de los formularios. | No disponible | 


<!--
|[Personalizer](./personalizer/what-is-personalizer.md) |F0, S0|**Personalizer** ([image](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409))|Azure Personalizer is a cloud-based API service that allows you to choose the best experience to show to your users, learning from their real-time behavior.|
-->

Además, se admiten algunos contenedores en la oferta de [recurso de varios servicios](cognitive-services-apis-create-account.md) de Cognitive Services. Puede crear un único recurso todo en uno de Cognitive Services y usar la misma clave de facturación en los servicios compatibles para los siguientes servicios:

* Computer Vision
* Caras
* LUIS
* Text Analytics

## <a name="prerequisites"></a>Requisitos previos

Debe cumplir los siguientes requisitos previos para poder utilizar contenedores de Azure Cognitive Services:

**Motor de Docker**: debe tener el motor de Docker instalado localmente. Docker proporciona paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Linux](https://docs.docker.com/engine/installation/#supported-platforms) y [Windows](https://docs.docker.com/docker-for-windows/). En Windows, Docker debe configurarse para admitir los contenedores de Linux. Los contenedores de Docker también se pueden implementar directamente en [Azure Kubernetes Service](../aks/index.yml) o [Azure Container Instances](../container-instances/index.yml).

Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio.

**Familiaridad con Microsoft Container Registry y Docker**: debe tener un conocimiento básico de los conceptos de Microsoft Container Registry y Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como de los comandos `docker` básicos.

Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).

Los contenedores individuales también pueden tener sus propios requisitos, incluidos los requisitos de asignación de memoria y servidor.

[!INCLUDE [Cognitive Services container security](containers/includes/cognitive-services-container-security.md)]

## <a name="developer-samples"></a>Ejemplos para desarrolladores

Hay ejemplos para desarrolladores disponibles en nuestro [repositorio de GitHub](https://github.com/Azure-Samples/cognitive-services-containers-samples).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre las [recetas del contenedor](containers/container-reuse-recipe.md) que puede usar con Cognitive Services.

Instale y explore la funcionalidad proporcionada por los contenedores en Azure Cognitive Services:

* [Contenedores de Anomaly Detector][ad-containers]
* [Contenedores de Computer Vision][cv-containers]
* [Contenedores de Face][fa-containers]
* [Contenedores de Form Recognizer][fr-containers]
* [Contenedores de Language Understanding (LUIS)][lu-containers]
* [Contenedores de Speech Service API][sp-containers]
* [Contenedores de Text Analytics][ta-containers]
* [Contenedores de Translator][tr-containers]

<!--* [Personalizer containers](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409)
-->

[ad-containers]: anomaly-Detector/anomaly-detector-container-howto.md
[cv-containers]: computer-vision/computer-vision-how-to-install-containers.md
[fa-containers]: face/face-how-to-install-containers.md
[fr-containers]: form-recognizer/form-recognizer-container-howto.md
[lu-containers]: luis/luis-container-howto.md
[sp-containers]: speech-service/speech-container-howto.md
[spa-containers]: ./computer-vision/spatial-analysis-container.md
[sp-containers-lid]: speech-service/speech-container-howto.md?tabs=lid
[sp-containers-stt]: speech-service/speech-container-howto.md?tabs=stt
[sp-containers-cstt]: speech-service/speech-container-howto.md?tabs=cstt
[sp-containers-tts]: speech-service/speech-container-howto.md?tabs=tts
[sp-containers-ctts]: speech-service/speech-container-howto.md?tabs=ctts
[sp-containers-ntts]: speech-service/speech-container-howto.md?tabs=ntts
[ta-containers]: text-analytics/how-tos/text-analytics-how-to-install-containers.md
[ta-containers-keyphrase]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=keyphrase
[ta-containers-language]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=language
[ta-containers-sentiment]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=sentiment
[ta-containers-health]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=health
[tr-containers]: translator/containers/translator-how-to-install-container.md
[request-access]: https://aka.ms/csgate
