---
title: archivo de inclusión
description: archivo de inclusión
services: azure-monitor
author: rboucher
tags: azure-service-management
ms.topic: include
ms.date: 02/07/2019
ms.author: robb
ms.custom: include file
ms.openlocfilehash: 704697a88a723182c2dbbcb2f25eb18df7be34c1
ms.sourcegitcommit: ef950cf37f65ea7a0f583e246cfbf13f1913eb12
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111429595"
---
**Volumen de colección de datos y retención** 

| Nivel | Límite por día | Retención de datos | Comentario |
|:---|:---|:---|:---|
| Plan de tarifa actual por GB<br>(Introducido en abril de 2018) | Sin límite | Entre 30 y 730 días | La retención de datos más allá de 31 días está disponible con cargos adicionales. Obtenga información sobre los precios de Azure Monitor. |
| Niveles gratis heredados<br>(Introducidos en abril de 2016) | 500 MB | 7 días | Cuando el área de trabajo alcance el límite de 500 MB por día, la ingesta de datos se detiene y se reanuda al comienzo del día siguiente. Un día se basa en UTC. Tenga en cuenta que los datos que recopila Azure Security Center no se incluyen en este límite de 500 MB por día y se seguirán recopilando más allá de este límite.  |
| Nivel heredado independiente por GB<br>(Introducidos en abril de 2016) | Sin límite | De 30 a 730 días | La retención de datos más allá de 31 días está disponible con cargos adicionales. Obtenga información sobre los precios de Azure Monitor. |
| Heredado por nodo (OMS)<br>(Introducidos en abril de 2016) | Sin límite | De 30 a 730 días | La retención de datos más allá de 31 días está disponible con cargos adicionales. Obtenga información sobre los precios de Azure Monitor. |
| Nivel estándar heredado | Sin límite | 30 días  | No se puede ajustar la retención |
| Nivel Premium heredado | Sin límite | 365 días  | No se puede ajustar la retención |

**Número de áreas de trabajo por suscripción.**

| Plan de tarifa    | Límite del área de trabajo | Comentarios
|:---|:---|:---|
| Nivel Gratis  | 10 | Este límite no se puede aumentar. |
| Todos los demás niveles | Sin límite | Está limitado por el número de recursos de un grupo de recursos y el número de grupos de recursos por suscripción. |

**Azure Portal**

| Category | Límite | Comentarios |
|:---|:---|:---|
| Número máximo de registros devueltos por una consulta de registro | 30,000 | Para reducir los resultados, use un ámbito de consulta, intervalo de tiempo y filtros en la consulta. |


**API de recopilador de datos**

| Category | Límite | Comentarios |
|:---|:---|:---|
| Tamaño máximo de una sola publicación | 30 MB | Dividir volúmenes más grandes en varias publicaciones. |
| Tamaño máximo de los valores de campo  | 32 KB | Los campos de más de 32 KB se truncan. |

**API de búsqueda**

| Category | Límite | Comentarios |
|:---|:---|:---|
| Número máximo de registros devueltos por una única consulta | 500.000 | |
| Tamaño máximo de los datos devueltos | Aprox. 104 MB (aprox. 100 MiB)| |
| Tiempo máximo de ejecución de la consulta | 10 minutos | Consulte [Tiempos de espera](https://dev.loganalytics.io/documentation/Using-the-API/Timeouts) para obtener más detalles.  |
| Velocidad máxima de solicitud | 200 solicitudes por 30 segundos por dirección IP de cliente o usuario de Azure AD | Consulte [Límites de velocidad](https://dev.loganalytics.io/documentation/Using-the-API/Limits) para obtener más detalles. |

**Búsqueda de Azure Monitor Logs**

| Category | Límite | Comentarios |
|:---|:---|:---|
| Número máximo de registros | 500.000 | |
| Tamaño máximo de los datos devueltos | Aprox. 104 MB (aprox. 100 MiB) | |
| Tiempo de espera máximo de las consultas | 110 segundos | |
| Gráficos | | La visualización en la página Registros y el conector utilizan diferentes bibliotecas de gráficos y algunas funcionalidades no están disponibles en el conector actualmente. |

**Límites generales del área de trabajo**

| Category | Límite | Comentarios |
|:---|:---|:---|
| Número máximo de columnas en una tabla         | 500 | |
| Número máximo de caracteres para el nombre de columna | 500 | |

**<a name="data-ingestion-volume-rate">Velocidad de volumen de ingesta de datos</a>**

Azure Monitor es un servicio de datos a gran escala que atiende a miles de clientes que envían terabytes de datos cada mes a un ritmo creciente. Lo que se pretende con el límite de velocidad de volumen es evitar que los clientes de Azure Monitor tengan clientes de picos de ingesta repentinos en entornos con varios inquilinos. En las áreas de trabajo se define un umbral de velocidad de volumen de ingesta de datos de 500 MB (comprimidos), lo que se traduce en, aproximadamente, **6 GB/min** sin comprimir (el tamaño real puede variar entre los tipos de datos en función de la longitud del registro y su relación de compresión). El límite de velocidad se aplica a los datos ingeridos de recursos de Azure a través de [Configuración de diagnóstico](../articles/azure-monitor/essentials/diagnostic-settings.md). Cuando se alcanza el límite de velocidad, un mecanismo de reintento intenta ingerir los datos 4 veces en un período de 30 minutos y quitarlos si se produce algún error en la operación. No se aplica a los datos ingeridos de [agentes](../articles/azure-monitor/agents/agents-overview.md) ni a [Data Collector API](../articles/azure-monitor/logs/data-collector-api.md).

Cuando la velocidad de los datos enviados a un área de trabajo es superior al 80 % del umbral configurado en dicha área, se envía un evento a la tabla *Operación* del área de trabajo cada 6 horas mientras se siga superando el umbral. Cuando la velocidad de ingesta del volumen supera el umbral, se quitan algunos datos y se envía un evento a la tabla *Operación* del área de trabajo cada 6 horas mientras se siga superando el umbral. Si la velocidad de ingesta del sigue superando el umbral o prevé que lo va a alcanzar pronto, puede abrir una solicitud de soporte técnico para solicitar su aumento. 

Para crear reglas de alerta, con el fin de recibir una notificación cada vez que alcance los límites de ingesta, consulte [Supervisión del estado del área de trabajo de Log Analytics in Azure Monitor](../articles/azure-monitor/logs/monitor-workspace.md).

>[!NOTE]
>Dependiendo del tiempo que lleve utilizando Log Analytics, es posible que tenga acceso a planes de tarifa heredados. Obtenga más información sobre los [planes de tarifa heredados de Log Analytics](../articles/azure-monitor/logs/manage-cost-storage.md#legacy-pricing-tiers).
