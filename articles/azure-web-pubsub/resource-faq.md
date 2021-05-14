---
title: Preguntas frecuentes sobre el servicio Azure Web PubSub
description: Obtenga respuestas a las preguntas más frecuentes sobre el servicio Azure Web PubSub.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: overview
ms.date: 04/26/2021
ms.openlocfilehash: e53823539194fe1082621f458e8b5b3d493c191d
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167075"
---
# <a name="azure-web-pubsub-service-faq"></a>Preguntas frecuentes sobre el servicio Azure Web PubSub

Estas son las preguntas más frecuentes sobre el servicio Azure Web PubSub. 

## <a name="is-azure-web-pubsub-service-ready-for-production-use"></a>¿Está listo el servicio Azure Web PubSub para usarlo en el entorno de producción?

El servicio Azure Web PubSub está en estado de versión preliminar pública y no tiene un Acuerdo de Nivel de Servicio vigente. 

##  <a name="where-does-my-data-reside"></a>¿Dónde residen mis datos?

El servicio Azure Web PubSub funciona como un servicio de procesador de datos. No almacenará ningún contenido del cliente y la residencia de datos se incluye intencionadamente. Si usa el servicio Azure Web PubSub junto con otros servicios de Azure, como Azure Storage para diagnósticos, consulte [estas notas del producto](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure/) para obtener instrucciones sobre cómo mantener la residencia de los datos en las regiones de Azure.