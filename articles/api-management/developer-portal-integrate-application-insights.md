---
title: Integración de Application Insights en el portal para desarrolladores
titleSuffix: Azure API Management
description: Obtenga información sobre cómo integrar Application Insights en el portal para desarrolladores administrado o autohospedado.
author: dlepow
ms.author: apimpm
ms.date: 03/25/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: 5e1c9caa55d0b3b7820f766a30c878fdc01f5137
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741399"
---
# <a name="integrate-application-insights-to-developer-portal"></a>Integración de Application Insights en el portal para desarrolladores

Una característica conocida de Azure Monitor es Application Insights. Se trata de un servicio de Application Performance Management (APM) extensible para desarrolladores y profesionales de DevOps. Úselo para supervisar el portal para desarrolladores y detectar anomalías en el rendimiento. Application Insights incluye eficaces herramientas de análisis para ayudarle a aprender qué hacen realmente los usuarios al visitar su portal para desarrolladores.

## <a name="add-application-insights-to-your-portal"></a>Adición de Application Insights a un portal

Siga estos pasos para conectar Application Insights con el portal para desarrolladores administrado o autohospedado.

> [!IMPORTANT]
> Los pasos 1 y 2 no son necesarios para los portales administrados. Si tiene un portal administrado, vaya al paso 4.

1. Configure un [entorno local ](developer-portal-self-host.md#step-1-set-up-local-environment) para la versión más reciente del portal para desarrolladores.

1. Instale el paquete **npm** para agregar [Paperbits para Azure](https://github.com/paperbits/paperbits-azure):

    ```console
    npm install @paperbits/azure --save
    ```

1. En el archivo `startup.publish.ts` de la carpeta `src`, importe el módulo de Application Insights y regístrelo:

    ```typescript
    import { AppInsightsPublishModule } from "@paperbits/azure";
    ...
    injector.bindModule(new AppInsightsPublishModule());
    ```

1. Recupere la configuración del portal:

    ```http
    GET /contentTypes/document/contentItems/configuration
    ```

    ```json
    {
        "nodes": [
            {
                "site": {
                    "title": "Microsoft Azure API Management - developer portal",
                    "description": "Discover APIs, learn how to use them, try them out interactively, and sign up to acquire keys.",
                    "keywords": "Azure, API Management, API, developer",
                    "faviconSourceId": null,
                    "author": "Microsoft Azure API Management"
                }
            }
        ]
    }
    ```

1. Extienda la configuración del sitio del paso anterior con la configuración de Application Insights:

    ```http
    PUT /contentTypes/document/contentItems/configuration
    ```

    ```json
    {
        "nodes": [
            {
                "site": { ... },
                "integration": {
                    "appInsights": {
                        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxxxxxxxxxxxxxxx"
                    }
                }
            }
        ]
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)
- [Automatización de implementaciones del portal](automate-portal-deployments.md)
- [Autohospedaje del portal para desarrolladores](developer-portal-self-host.md)
