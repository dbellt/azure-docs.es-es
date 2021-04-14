---
title: Instalación de contenedores de Speech
titleSuffix: Azure Cognitive Services
description: Detalles de las opciones de configuración de un gráfico Helm de speech-to-text
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 05/05/2020
ms.author: trbye
ms.openlocfilehash: 85c4e0641e1989ddea6c8aa8b8a8895a966a5ddb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96002305"
---
### <a name="speech-to-text-sub-chart-chartsspeechtotext"></a>Speech-to-Text (gráfico secundario: charts/speechToText)

Para reemplazar al gráfico de nivel superior, agregue el prefijo `speechToText.` a todos los parámetros para que sean más específicos. Se reemplazará el parámetro correspondiente, por ejemplo `speechToText.numberOfConcurrentRequest` reemplaza a `numberOfConcurrentRequest`.

|Parámetro|Descripción|Valor predeterminado|
| -- | -- | -- |
| `enabled` | Si el servicio **speech-to-text** está habilitado. | `false` |
| `numberOfConcurrentRequest` | Número de solicitudes simultáneas del servicio **speech-to-text**. Este gráfico calcula automáticamente los recursos de CPU y memoria en función de este valor. | `2` |
| `optimizeForAudioFile`| Si el servicio necesita optimizar la entrada de audio a través de archivos de audio. Si `true`, este gráfico asignará más recursos de CPU al servicio. | `false` |
| `image.registry`| Registro de la imagen de Docker de **speech-to-text**. | `containerpreview.azurecr.io` |
| `image.repository` | Repositorio de la imagen de Docker de **speech-to-text**. | `microsoft/cognitive-services-speech-to-text` |
| `image.tag` | Etiqueta de la imagen de Docker de **speech-to-text**. | `latest` |
| `image.pullSecrets` | Secretos de la imagen para extraer la imagen de Docker de **speech-to-text**. | |
| `image.pullByHash`| Si la imagen de Docker se extrae mediante hash. Si `true`, `image.hash` es obligatorio. | `false` |
| `image.hash`| Hash de la imagen de Docker de **speech-to-text**. Solo se usa con `image.pullByHash: true`.  | |
| `image.args.eula` (obligatorio) | Indica que ha aceptado la licencia. El único valor válido es `accept`. | |
| `image.args.billing` (obligatorio) | El valor de URI del punto de conexión de facturación está disponible en Azure Portal, en la página de Información general de Información general del servicio de voz. | |
| `image.args.apikey` (obligatorio) | Se usa para realizar un seguimiento de la información de facturación. ||
| `service.type` | Tipo de servicio de Kubernetes del servicio **speech-to-text**. Consulte las [instrucciones de los tipos de servicio de Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) para obtener más información y comprobar la compatibilidad con los proveedores de nube. | `LoadBalancer` |
| `service.port`|  Puerto del servicio **speech-to-text**. | `80` |
| `service.annotations` | Anotaciones de **conversión de voz a texto** para los metadatos del servicio. Las anotaciones son pares de clave-valor. <br>`annotations:`<br>&nbsp;&nbsp;`some/annotation1: value1`<br>&nbsp;&nbsp;`some/annotation2: value2` | |
| `service.autoScaler.enabled` | Si [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) está habilitado. Si `true`, `speech-to-text-autoscaler` se implementará en el clúster de Kubernetes. | `true` |
| `service.podDisruption.enabled` | Si [Pod Disruption Budget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) está habilitado. Si `true`, `speech-to-text-poddisruptionbudget` se implementará en el clúster de Kubernetes. | `true` |

#### <a name="sentiment-analysis-sub-chart-chartsspeechtotext"></a>Análisis de sentimiento (sub-chart: charts/speechToText)

A partir de la versión 2.2.0 del contenedor de conversión de voz a texto y la versión 0.2.0 del gráfico de Helm, se usan los siguientes parámetros para el análisis de sentimiento mediante Text Analytics API.

|Parámetro|Descripción|Valores|Valor predeterminado|
| --- | --- | --- | --- |
|`textanalytics.enabled`| Si está habilitado el servicio **text-analytics**.| true/false| `false`|
|`textanalytics.image.registry`| El registro de la imagen de Docker **text-analytics**.| registro de imagen de Docker válido| |
|`textanalytics.image.repository`| El repositorio de la imagen de Docker **text-analytics**.| repositorio de imágenes de Docker válido| |
|`textanalytics.image.tag`| La etiqueta de la imagen de Docker **text-analytics**.| etiqueta de imagen de Docker válida| |
|`textanalytics.image.pullSecrets`| Los secretos de imagen para extraer la imagen de Docker **text-analytics**.| nombre de secreto válido| |
|`textanalytics.image.pullByHash`| Especifica si se va a extraer la imagen de Docker por hash.  Si `yes`, `image.hash` también se requiere. Si `no`, establézcalo como "false". El valor predeterminado es `false`.| true/false| `false`|
|`textanalytics.image.hash`| El hash de la imagen de Docker **text-analytics**. Úselo solo con `image.pullByHash:true`.| hash de imagen de Docker válido | |
|`textanalytics.image.args.eula`| Uno de los argumentos necesarios en el contenedor **text-analytics**, que indica que ha aceptado la licencia. El valor de esta opción debe establecerse en `accept`.| `accept`, si desea utilizar el contenedor. | |
|`textanalytics.image.args.billing`| Uno de los argumentos necesarios en el contenedor **text-analytics**, que especifica el identificador URI del punto de conexión de facturación. El valor de URI del punto de conexión de facturación está disponible en Azure Portal, en la página de Información general de Información general del servicio de voz.|identificador URI del punto de conexión de facturación||
|`textanalytics.image.args.apikey`| Uno de los argumentos necesarios en el contenedor **text-analytics**, que se usa para realizar un seguimiento de la información de facturación.| apiKey válido||
|`textanalytics.cpuRequest`| La CPU solicitada para el contenedor **text-analytics**.| int| `3000m`|
|`textanalytics.cpuLimit`| La CPU limitada del contenedor **text-analytics**.| | `8000m`|
|`textanalytics.memoryRequest`| La memoria solicitada del contenedor **text-analytics**.| | `3Gi`|
|`textanalytics.memoryLimit`| La memoria limitada del contenedor **text-analytics**.| | `8Gi`|
|`textanalytics.service.sentimentURISuffix`| El sufijo URI de análisis de sentimiento, el identificador URI completo tiene el formato "http://`<service>`:`<port>`/`<sentimentURISuffix>`". | | `text/analytics/v3.0-preview/sentiment`|
|`textanalytics.service.type`| El tipo de servicio **text-analytics** de Kubernetes. Consulte los [tipos de servicio de Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/). | tipo de servicio Kubernetes válido | `LoadBalancer` |
|`textanalytics.service.port`| El puerto del servicio **text-analytics**.| int| `50085`|
|`textanalytics.service.annotations`| Las anotaciones que los usuarios pueden agregar a los metadatos del servicio **text-analytics**. Por ejemplo:<br/> **anotaciones:**<br/>`   ` **some/annotation1: value1**<br/>`  ` **some/annotation2: value2** | anotaciones, una por cada línea| |
|`textanalytics.serivce.autoScaler.enabled`| Si [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) está habilitado. Si está habilitado, `text-analytics-autoscaler` se implementará en el clúster de Kubernetes. | true/false| `true`|
|`textanalytics.service.podDisruption.enabled`| Si [Pod Disruption Budget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) está habilitado. Si está habilitado, `text-analytics-poddisruptionbudget` se implementará en el clúster de Kubernetes.| true/false| `true`|
