---
author: IEvangelist
ms.author: dapine
ms.date: 10/02/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: a6f399e19cadf3d6ce9edfaecb3d904e62c498aa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96006887"
---
Este contenedor tiene las siguientes opciones de configuración:

|Obligatorio|Configuración|Propósito|
|--|--|--|
|Sí|[ApiKey](#apikey-configuration-setting)|Realiza el seguimiento de la información de facturación.|
|No|[ApplicationInsights](#applicationinsights-setting)|Permite agregar compatibilidad con los datos de telemetría de [Azure Application Insights](/azure/application-insights) al contenedor.|
|Sí|[Facturación](#billing-configuration-setting)|Especifica el URI del punto de conexión del recurso de servicio en Azure.|
|Sí|[Eula](#eula-setting)| Indica que ha aceptado la licencia del contenedor.|
|No|[Fluentd](#fluentd-settings)|Escribe el registro y, opcionalmente, los datos de métricas en un servidor de Fluentd.|
|No|Proxy HTTP|Configura un proxy HTTP para realizar solicitudes de salida.|
|No|[Logging](#logging-settings)|Proporciona compatibilidad con el registro de ASP.NET Core al contenedor. |
|No|[Mounts](#mount-settings)|Lee y escribe los datos desde el equipo host al contenedor y del contenedor de nuevo al equipo host.|