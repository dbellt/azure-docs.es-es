---
title: 'Consulta de datos desde un entorno Gen1 mediante código de C#: Azure Time Series Insights Gen1 | Microsoft Docs'
description: Aprenda a consultar datos desde un entorno de Azure Time Series Insights Gen1 mediante una aplicación personalizada escrita en C#.
ms.service: time-series-insights
services: time-series-insights
author: shreyasharmamsft
ms.author: shresha
manager: cnovak
ms.reviewer: orspodek
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: ccc49221514f7846148ff2958e9fe8954fba074e
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110790892"
---
# <a name="query-data-from-the-azure-time-series-insights-gen1-environment-using-c-sharp"></a>Consulta de datos desde el entorno de Azure Time Series Insights Gen1 mediante C#

> [!CAUTION]
> Este es un artículo de Gen1.

En este ejemplo de C# se muestra cómo usar las [API de consulta de Gen1](/rest/api/time-series-insights/gen1-query) para consultar datos desde entornos de Azure Time Series Insights Gen1.

> [!TIP]
> Consulte ejemplos de código C# de Gen1 en [https://github.com/Azure-Samples/Azure-Time-Series-Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/tree/master/gen1-sample/csharp-tsi-gen1-sample).

## <a name="summary"></a>Resumen

En el código de ejemplo siguiente se muestran las siguientes características:

* Cómo adquirir un token de acceso a través de Azure Active Directory con [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).

* Cómo pasar ese token de acceso adquirido en el encabezado `Authorization` de las solicitudes de API de consulta subsiguientes.

* En el ejemplo se llama a cada una de las API de consulta de Gen1 para demostrar cómo se hacen las solicitudes HTTP a:
  * [Get Environments API](/rest/api/time-series-insights/gen1-query-api#get-environments-api) para devolver los entornos a los que tiene acceso el usuario.
  * [Get Environment Availability API](/rest/api/time-series-insights/gen1-query-api#get-environment-availability-api)
  * [Get Environment Metadata API](/rest/api/time-series-insights/gen1-query-api#get-environment-metadata-api) para recuperar los metadatos del entorno.
  * [Get Environments Events API](/rest/api/time-series-insights/gen1-query-api#get-environment-events-api)
  * [Get Environment Aggregates API](/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-api)

* Cómo interactuar con las API de consulta de Gen1 mediante WSS para enviar un mensaje a:

  * [Get Environment Events Streamed API](/rest/api/time-series-insights/gen1-query-api#get-environment-events-streamed-api)
  * [Get Environment Aggregates Streamed API](/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-streamed-api)

## <a name="prerequisites-and-setup"></a>Requisitos previos y configuración

Realice los pasos siguientes antes de compilar y ejecutar el código de ejemplo:

1. [Aprovisione un entorno de Azure Time Series Insights Gen1](./time-series-insights-get-started.md).
1. Configure el entorno de Azure Time Series Insights para Azure Active Directory, tal como se describe en [Autenticación y autorización](time-series-insights-authentication-and-authorization.md).
1. Instale las dependencias de proyecto necesarias.
1. Edite el código de ejemplo siguiente y reemplace cada **#DUMMY#** por el identificador de entorno adecuado.
1. Ejecute el código dentro de Visual Studio.

## <a name="project-dependencies"></a>Dependencias del proyecto

Se recomienda usar la versión más reciente de Visual Studio:

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/), versión 16.4.2+

El código de ejemplo tiene dos dependencias necesarias:

* [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/), paquete 3.13.9.
* [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json), paquete 9.0.1.

Para descargar los paquetes en Visual Studio 2019, seleccione la opción **Compilar** > **Compilar solución**.

También puede agregar los paquetes con [NuGet 2.12+](https://www.nuget.org/):

* `dotnet add package Newtonsoft.Json --version 9.0.1`
* `dotnet add package Microsoft.IdentityModel.Clients.ActiveDirectory --version 3.13.9`

## <a name="c-sample-code"></a>Código de ejemplo de C#

Consulte el repositorio [Azure Time Series Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen1-sample/csharp-tsi-gen1-sample/Program.cs)] para acceder al código de ejemplo de C#.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la consulta, vea la [referencia sobre las API de consulta](/rest/api/time-series-insights/gen1-query-api).

* Lea cómo [conectar una aplicación de JavaScript mediante el SDK de cliente](https://github.com/microsoft/tsiclient) a Time Series Insights.
Azure-Samples/Azure-Time-Series-Insights/gen1-sample/csharp-tsi-gen1-sample/Program.cs