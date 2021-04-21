---
title: Docker pull para el contenedor de análisis de sentimiento
titleSuffix: Azure Cognitive Services
description: Comando pull de Docker para el contenedor de análisis de sentimiento
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 32a550e120331a8255281d51725d2d5fc8ca1e05
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107564653"
---
#### <a name="docker-pull-for-the-sentiment-analysis-v3-container"></a>Docker pull para el contenedor de Análisis de sentimiento v3

El contenedor de Análisis de sentimiento v3 está disponible en varios idiomas. Para descargar el contenedor en inglés, use el siguiente comando. 

```
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-en
```

Para descargar el contenedor para otro idioma, reemplace `en` por uno de los siguientes códigos de idioma. 

| Contenedor de Text Analytics | Código de lenguaje |
|--|--|
| Chino simplificado    |   `zh-hans`   |
| Chino (tradicional)   |   `zh-hant`   |
| Neerlandés                 |     `nl`      |
| Inglés               |     `en`      |
| Francés                |     `fr`      |
| Alemán                |     `de`      |
| Hindi                 |    `hi`       |
| Italiano               |     `it`      |
| Japonés              |     `ja`      |
| Coreano                |     `ko`      |
| Noruego (Bokmål)   |     `no`      |
| Portugués (Brasil)   |    `pt-BR`    |
| Portugués (Portugal) |    `pt-PT`    |
| Español               |     `es`      |
| Turco               |     `tr`      |

Para obtener una descripción completa de las etiquetas disponibles para los contenedores de Text Analytics, consulte [Docker Hub](https://go.microsoft.com/fwlink/?linkid=2018654).
