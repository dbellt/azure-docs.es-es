---
title: Requisitos de consulta
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: applied-ai-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 09/30/2020
ms.author: mbullwin
ms.openlocfilehash: c7d038153385c2d36d48f660a9a4e2bab29a3c45
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111894027"
---
Dentro de la consulta, use el parámetro `@StartTime` para obtener datos de métricas para una sola marca de tiempo. El asesor de métricas reemplazará el parámetro por una cadena de formato `yyyy-MM-ddTHH:mm:ss` cuando ejecute la consulta.

> [!IMPORTANT]
> La consulta debe devolver como máximo un registro para cada combinación de dimensión, en cada marca de tiempo. Y todos los registros devueltos por la consulta deben tener las mismas marcas de tiempo. El asesor de métricas ejecutará esta consulta para cada marca de tiempo que deba ingerir los datos. Consulte [Preguntas frecuentes sobre las consultas](../faq.md#how-do-i-write-a-valid-query-for-ingesting-my-data) para más información y ejemplos. 