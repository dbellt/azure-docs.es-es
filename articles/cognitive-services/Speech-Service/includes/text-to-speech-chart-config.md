---
title: Instalación de contenedores de Speech
titleSuffix: Azure Cognitive Services
description: Detalles de las opciones de configuración de un gráfico Helm de text-to-speech
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 22168974ab8b285413b4fa6e947c05f65a73ae12
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96002306"
---
### <a name="text-to-speech-sub-chart-chartstexttospeech"></a>Text-to-Speech (gráfico secundario: charts/textToSpeech)

Para reemplazar al gráfico de nivel superior, agregue el prefijo `textToSpeech.` a todos los parámetros para que sean más específicos. Se reemplazará el parámetro correspondiente, por ejemplo `textToSpeech.numberOfConcurrentRequest` reemplaza a `numberOfConcurrentRequest`.

|Parámetro|Descripción|Valor predeterminado|
| -- | -- | -- |
| `enabled` | Si el servicio **text-to-speech** está habilitado. | `false` |
| `numberOfConcurrentRequest` | Número de solicitudes simultáneas del servicio **text-to-speech**. Este gráfico calcula automáticamente los recursos de CPU y memoria en función de este valor. | `2` |
| `optimizeForTurboMode`| Si el servicio necesita optimizar la entrada de texto a través de archivos de texto. Si `true`, este gráfico asignará más recursos de CPU al servicio. | `false` |
| `image.registry`| Registro de la imagen de Docker de **text-to-speech**. | `containerpreview.azurecr.io` |
| `image.repository` | Repositorio de la imagen de Docker de **text-to-speech**. | `microsoft/cognitive-services-text-to-speech` |
| `image.tag` | Etiqueta de la imagen de Docker de **text-to-speech**. | `latest` |
| `image.pullSecrets` | Secretos de la imagen para extraer la imagen de Docker de **text-to-speech**. | |
| `image.pullByHash`| Si la imagen de Docker se extrae mediante hash. Si `true`, `image.hash` es obligatorio. | `false` |
| `image.hash`| Hash de la imagen de Docker de **text-to-speech**. Solo se usa con `image.pullByHash: true`.  | |
| `image.args.eula` (obligatorio) | Indica que ha aceptado la licencia. El único valor válido es `accept`. | |
| `image.args.billing` (obligatorio) | El valor de URI del punto de conexión de facturación está disponible en Azure Portal, en la página de Información general de Información general del servicio de voz. | |
| `image.args.apikey` (obligatorio) | Se usa para realizar un seguimiento de la información de facturación. ||
| `service.type` | Tipo de servicio de Kubernetes del servicio **text-to-speech**. Consulte las [instrucciones de los tipos de servicio de Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) para obtener más información y comprobar la compatibilidad con los proveedores de nube. | `LoadBalancer` |
| `service.port`|  El puerto del servicio **text-to-speech**. | `80` |
| `service.annotations` | Las anotaciones de **text-to-speech** para los metadatos del servicio. Las anotaciones son pares de clave-valor. <br>`annotations:`<br>&nbsp;&nbsp;`some/annotation1: value1`<br>&nbsp;&nbsp;`some/annotation2: value2` | |
| `service.autoScaler.enabled` | Si [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) está habilitado. Si `true`, `text-to-speech-autoscaler` se implementará en el clúster de Kubernetes. | `true` |
| `service.podDisruption.enabled` | Si [Pod Disruption Budget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) está habilitado. Si `true`, `text-to-speech-poddisruptionbudget` se implementará en el clúster de Kubernetes. | `true` |