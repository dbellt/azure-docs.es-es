---
title: Configuración de contenedores - Translator
titleSuffix: Azure Cognitive Services
description: El entorno de tiempo de ejecución de contenedores de Translator se configura mediante los argumentos de comando `docker run`. Hay configuraciones obligatorias y opcionales.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 05/05/2021
ms.author: lajanuar
recommendations: false
ms.openlocfilehash: 42376cc7c1e1745e3db3ce3467ea02221fb7b834
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110478111"
---
# <a name="configure-translator-docker-containers-preview"></a>Configuración de contenedores de Translator Docker (versión preliminar)

Cognitive Services proporciona a cada contenedor un marco de configuración común.  Puede configurar fácilmente sus contenedores de Translator y crear una arquitectura de aplicación de Translator que esté optimizada para funcionalidades sólidas en la nube y localidad perimetral.

El entorno de tiempo de ejecución de contenedores de **Translator** se configura mediante los argumentos de comando `docker run`. Este contenedor tiene varias opciones de configuración necesarias, así como otras opcionales. La configuración específica del contenedor es la configuración de facturación.

## <a name="configuration-settings"></a>Parámetros de configuración

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
|Sí|[Mounts](#mount-settings)|Lee y escribe los datos desde el equipo host al contenedor y del contenedor de nuevo al equipo host.|

 > [!IMPORTANT]
> Los parámetros [**ApiKey**](#apikey-configuration-setting), [**Billing**](#billing-configuration-setting) y [**EULA**](#eula-setting) se usan conjuntamente. Además, debe proporcionar valores válidos para los tres; de lo contrario, el contenedor no se iniciará. Para obtener más información sobre cómo usar estos parámetros de configuración con el fin de crear instancias de un contenedor,

## <a name="apikey-configuration-setting"></a>Opción de configuración ApiKey

La opción de configuración `ApiKey` especifica la clave de recurso de Azure usada para realizar un seguimiento de la información de facturación del contenedor. Debe especificar un valor para el parámetro ApiKey, que debe ser una clave válida para el recurso de _Translator_ que se haya especificado para el parámetro de configuración [`Billing`](#billing-configuration-setting).

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: administración de recursos de **Translator**, en **Claves**.

## <a name="applicationinsights-setting"></a>Opción de configuración ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Opción de configuración Billing

El parámetro `Billing` especifica el URI de punto de conexión del recurso de _Translator_ de Azure usado para medir la información de facturación del contenedor. Debe especificar un valor para este parámetro de configuración, que debe ser un URI de punto de conexión válido para un recurso de _Translator_ en Azure. El contenedor informa sobre el uso cada 10 a 15 minutos.

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: página de información general de **Translator**, etiquetado como `Endpoint`.

| Obligatorio | Nombre | Tipo de datos | Descripción |
| -------- | ---- | --------- | ----------- |
| Sí | `Billing` | String | Identificador URI del punto de conexión de facturación. Para más información sobre cómo obtener el URI de facturación, consulte la [recopilación de los parámetros necesarios](translator-how-to-install-container.md#required-elements). Para más información y para obtener una lista completa de los puntos de conexión regionales, consulte [Nombres de subdominios personalizados para Cognitive Services.](../../cognitive-services-custom-subdomains.md) |

## <a name="eula-setting"></a>Opción de configuración Eula

[!INCLUDE [Container shared configuration eula settings](../../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Opción de configuración Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Configuración de las credenciales del proxy HTTP

[!INCLUDE [Container shared HTTP proxy settings](../../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Opción de configuración Logging

[!INCLUDE [Container shared configuration logging settings](../../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Configuración de montaje

Utilice montajes de enlace para leer y escribir datos hacia y desde el contenedor. Puede especificar un montaje de entrada o un montaje de salida mediante la opción `--mount` del comando [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Los contenedores de Translator no usan montajes de entrada o salida para almacenar datos de entrenamiento o servicio.

La sintaxis exacta de la ubicación de montaje del host varía según el sistema operativo del host. Además, la ubicación de montaje del [equipo host](translator-how-to-install-container.md#host-computer) puede no estar accesible debido a un conflicto entre los permisos que utiliza la cuenta de servicio de Docker y los permisos de la ubicación de montaje del host.
<!-- markdownlint-disable MD033 -->
|Opcional| Nombre | Tipo de datos | Descripción |
|-------|------|-----------|-------------|
|No permitida| `Input` | String | Los contenedores de Translator no usan este valor.|
|Opcional| `Output` | String | Destino del montaje de salida. El valor predeterminado es `/output`. Aquí es donde encontrará la ubicación de los registros, incluidos registros de contenedor. <br><br>Ejemplo:<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los contenedores de Cognitive Services](../../cognitive-services-container-support.md)
