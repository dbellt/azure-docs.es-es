---
title: Cómo contribuir con widgets para el portal para desarrolladores
titleSuffix: Azure API Management
description: Obtenga información sobre las directrices recomendadas que se deben seguir al contribuir con un widget al repositorio del portal para desarrolladores de API Management.
author: dlepow
ms.author: apimpm
ms.date: 03/25/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: c4d3ed2aeaac57f721d23d7c7aa1c70ef14e4294
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741370"
---
# <a name="how-to-contribute-widgets-to-the-api-management-developer-portal"></a>Contribución de widgets al portal para desarrolladores de API Management

Si quiere contribuir con un widget al [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal) del portal para desarrolladores de API Management, siga este proceso de tres pasos:

1. Bifurque el repositorio.

1. Implemente el widget.

1. Abra una solicitud de incorporación de cambios para incluir el widget en el repositorio oficial.

El widget heredará la licencia del repositorio. Estará disponible para la [instalación de participación](developer-portal-use-community-widgets.md) en la versión autohospedada del portal. El equipo del portal para desarrolladores puede decidir incluirlo también en la versión administrada del portal.

Consulte el tutorial de [implementación de widgets](developer-portal-implement-widgets.md) para obtener un ejemplo de cómo desarrollar su propio widget.

## <a name="contribution-guidelines"></a>Guías para la contribución

Esta guía está pensada para garantizar la seguridad y la privacidad de nuestros clientes y de los visitantes de sus portales. Siga esta guía para asegurarse de que se acepte su contribución:

1. Coloque el widget en la carpeta `community/widgets/<your-widget-name>`.

1. El nombre del widget debe estar en minúsculas y ser alfanumérico, con guiones que separen las palabras. Por ejemplo, `my-new-widget`.

1. La carpeta debe contener una captura de pantalla del widget en un portal publicado.

1. La carpeta debe contener un archivo `readme.md`, que siga la plantilla del archivo `/scaffolds/widget/readme.md`.

1. La carpeta puede contener un archivo `npm_dependencies` con comandos npm para instalar o administrar las dependencias del widget.

    Especifique explícitamente la versión de cada dependencia. Por ejemplo:  

    ```console
    npm install azure-storage@2.10.3 axios@0.19.1
    ```

    El widget debe requerir un mínimo de dependencias. Los revisores inspeccionarán cuidadosamente todas las dependencias. En concreto, la lógica principal del widget debe ser de código abierto en la carpeta del widget. No lo ajuste en un paquete de npm.

1. No se permiten cambios en los archivos fuera de la carpeta del widget como parte de la contribución de un widget. Esto incluye, entre otras cosas, al archivo `/package.json`.

1. No se permite insertar scripts de seguimiento ni enviar datos creados por el cliente a servicios personalizados.

    > [!NOTE]
    > Solo puede recopilar datos creados por el cliente a través de la interfaz `Logger`.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre las contribuciones, consulte el [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal/) del portal para desarrolladores de API Management.

- Consulte [Implementación de widgets](developer-portal-implement-widgets.md) para aprender a desarrollar su propio widget, paso a paso.

- Consulte [Uso de widgets de la comunidad](developer-portal-use-community-widgets.md) para aprender a usar widgets aportados por la comunidad.