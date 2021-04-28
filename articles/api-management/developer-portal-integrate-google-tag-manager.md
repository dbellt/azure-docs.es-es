---
title: Integración de Google Tag Manager en el portal para desarrolladores
titleSuffix: Azure API Management
description: Aprenda a conectar Google Tag Manager en el portal para desarrolladores administrado o autohospedado en Azure API Management.
author: dlepow
ms.author: apimpm
ms.date: 03/25/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: c209eb782787146d947b4684d41c5d1e9bb6364e
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741400"
---
# <a name="integrate-google-tag-manager-to-api-management-developer-portal"></a>Integración de Google Tag Manager en el portal para desarrolladores de API Management

[Google Tag Manager](https://developers.google.com/tag-manager) es un sistema de administración de etiquetas creado por Google. Puede usarlo para administrar etiquetas JavaScript y HTML que se usan para el seguimiento y el análisis en sitios web. Por ejemplo, puede usar Google Tag Manager para integrar Google Analytics, mapas térmicos o bots de chat, como LiveChat.

Siga los pasos de este artículo para conectar Google Tag Manager en el portal para desarrolladores administrado o autohospedado en Azure API Management.

## <a name="add-google-tag-manager-to-your-portal"></a>Adición de Google Tag Manager al portal

Siga estos pasos para conectar Google Tag Manager en el portal para desarrolladores administrado o autohospedado.

> [!IMPORTANT]
> Los pasos 1 y 2 no son necesarios para los portales administrados. Si tiene un portal administrado, vaya al paso 4.

1. Configure un [entorno local ](developer-portal-self-host.md#step-1-set-up-local-environment) para la versión más reciente del portal para desarrolladores.

1. Instale el paquete **npm** para agregar [Paperbits para Google Tag Manager](https://github.com/paperbits/paperbits-gtm):

    ```sh
    npm install @paperbits/gtm --save
    ```

1. En el archivo `startup.publish.ts` de la carpeta `src`, importe el módulo GTM y regístrelo:

    ```typescript
    import { GoogleTagManagerPublishModule } from "@paperbits/gtm/gtm.publish.module";
    ...
    injector.bindModule(new GoogleTagManagerPublishModule());
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

1. Extienda la configuración del sitio del paso anterior con la configuración de Google Tag Manager:

    ```http
    PUT /contentTypes/document/contentItems/configuration
    ```

    ```json
    {
        "nodes": [
            {
                "site": { ... },
                "integration": {
                    "googleTagManager": {
                        "containerId": "GTM-..."
                    }
                }
            }
        ]
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Automatización de implementaciones del portal](automate-portal-deployments.md)
- [Autohospedaje del portal para desarrolladores](developer-portal-self-host.md)