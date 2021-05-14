---
title: Uso de widgets de la comunidad en el portal para desarrolladores
titleSuffix: Azure API Management
description: Obtenga información sobre los widgets de la comunidad para el portal para desarrolladores de API Management y cómo insertarlos y usarlos en el código.
author: dlepow
ms.author: apimpm
ms.date: 03/25/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: de6160aa2e556297287ae9e9ecd58a93e54d863f
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741312"
---
# <a name="use-community-widgets-in-the-developer-portal"></a>Uso de widgets de la comunidad en el portal para desarrolladores

Todos los desarrolladores colocan sus widgets aportados por la comunidad en la carpeta `/community/widgets/` en el [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal) del portal para desarrolladores de API Management. El equipo del portal para desarrolladores ha aceptado cada uno de ellos. Para usar los widgets, puede insertarlos en su [versión autohospedada](developer-portal-self-host.md) del portal. Actualmente, la versión administrada del portal para desarrolladores no admite widgets de la comunidad.

> [!NOTE]
> El equipo del portal para desarrolladores inspecciona exhaustivamente los widgets aportados y sus dependencias. Sin embargo, el equipo no puede garantizar que sea seguro cargar los widgets. Use su propio criterio al decidir usar un widget aportado por la comunidad. Consulte nuestra [guía de contribución de widgets](developer-portal-widget-contribution-guidelines.md#contribution-guidelines) para obtener información sobre nuestras medidas preventivas.

## <a name="inject-and-use-external-widgets"></a>Inserción y uso de widgets externos

1. Configure un [entorno local ](developer-portal-self-host.md#step-1-set-up-local-environment) para la versión más reciente del portal para desarrolladores.

1. Vaya a la carpeta del widget en el directorio `/community/widgets`. Lea la descripción del widget en el archivo `readme.md`.

1. Registre el widget en los módulos del portal:

    1. `src/apim.design.module.ts`: un módulo que registra las dependencias en tiempo de diseño.
    
        ```typescript
        import { WidgetNameDesignModule } from "../community/widgets/<widget-name>/widget.design.module";
    
        ...
    
            injector.bindModule(new WidgetNameDesignModule());
        ```
    
    1. `src/apim.publish.module.ts`: un módulo que registra las dependencias en tiempo de publicación.
    
        ```typescript
        import { WidgetNamePublishModule } from "../community/widgets/<widget-name>/widget.publish.module";
    
        ...
    
            injector.bindModule(new WidgetNamePublishModule());
        ```
    
    1. `src/apim.runtime.module.ts`: un módulo que registra las dependencias en tiempo de ejecución.
    
        ```typescript
        import { WidgetNameRuntimeModule } from "../community/widgets/<widget-name>/widget.runtime.module";
    
        ...
    
            injector.bindModule(new WidgetNameRuntimeModule());
        ```

1. Compruebe si el widget tiene un archivo `npm_dependencies`.

1. Si es así, copie los comandos del archivo y ejecútelos en el directorio superior del repositorio.

    Al hacerlo, se instalarán las dependencias del widget.

1. Ejecute `npm start`.

Puede ver el widget en la categoría **Comunidad** en el selector de widgets.

:::image type="content" source="media/developer-portal-use-community-widgets/widget-selector.png" alt-text="Captura de pantalla del selector de widgets":::


## <a name="next-steps"></a>Pasos siguientes


Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)

- [Contribución de widgets](developer-portal-widget-contribution-guidelines.md)
