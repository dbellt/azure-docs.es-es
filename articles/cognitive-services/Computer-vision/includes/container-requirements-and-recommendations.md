---
title: Recomendaciones y requisitos del contenedor
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/23/2020
ms.author: aahi
ms.openlocfilehash: 2a399b683dc9596d3514ce7d3be1fa2444449e2a
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284954"
---
> [!NOTE]
> Los requisitos y las recomendaciones se basan en pruebas comparativas con una única solicitud por segundo, con una imagen de 8 MB de una carta comercial digitalizada que contiene 29 líneas y un total de 803 caracteres.

En la tabla siguiente se describe la asignación mínima y recomendada de recursos para cada contenedor OCR de Read.

| Contenedor | Mínima | Recomendado |
|-----------|---------|-------------|
| Versión preliminar de Read 2.0 | 1 núcleo, 8 GB de memoria |    8 núcleos, 16 GB de memoria |
| Read 3.2 versión preliminar | 8 núcleos, 16 GB de memoria | 8 núcleos, 24 GB de memoria |

* Cada núcleo debe ser de 2,6 gigahercios (GHz) como mínimo.

El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.
