---
title: ¿Qué es Azure Static Web Apps?
description: Características clave y funcionalidad de Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: overview
ms.date: 04/01/2021
ms.author: cshoe
ms.openlocfilehash: c0a4156fb0015c6d319700c82f6285dbf0ed1443
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107947886"
---
# <a name="what-is-azure-static-web-apps-preview"></a>¿Qué es la versión preliminar de Azure Static Web Apps?

Azure Static Web Apps es un servicio que compila e implementa automáticamente aplicaciones web de pila completa en Azure desde un repositorio de código.

:::image type="content" source="media/overview/azure-static-web-apps-overview.png" alt-text="Diagrama de información general de Azure Static Web Apps":::

El flujo de trabajo de Azure Static Web Apps se adapta al flujo de trabajo diario de un desarrollador. Las aplicaciones se compilan e implementan en función de los cambios en el código.

Cuando se crea un recurso de Azure Static Web Apps, Azure interactúa directamente con GitHub o Azure DevOps para supervisar una rama de su elección. Cada vez que inserte confirmaciones o acepte solicitudes de incorporación de cambios en la rama inspeccionada, se ejecuta automáticamente una compilación y la aplicación y la API se implementan en Azure.

Las aplicaciones web estáticas se suelen compilar con bibliotecas y marcos como Angular, React, Svelte, Vue o Blazor, en los que no se requiere representación en el servidor. Estas aplicaciones incluyen recursos HTML, CSS, JavaScript y de imagen que componen la aplicación. Con un servidor web tradicional, estos recursos se ofrecen desde un único servidor junto con los puntos de conexión de la API necesarios.

Con Static Web Apps, los recursos estáticos se separan de un servidor web tradicional y, en su lugar, se ofrecen puntos distribuidos geográficamente en todo el mundo. Esta distribución permite ofrecer archivos de forma más rápida, dado que se encuentran físicamente más cerca de los usuarios finales. Además, los puntos de conexión de la API se hospedan mediante una [arquitectura sin servidor](../azure-functions/functions-overview.md), lo que evita la necesidad de usar un servidor back-end completo.

## <a name="key-features"></a>Principales características

- **Hospedaje web** para contenido estático como HTML, CSS, JavaScript e imágenes.
- Compatibilidad con la **API integrada** proporcionada por Azure Functions.
- **Integración de primera clase con GitHub y Azure DevOps**, en las que los cambios en el repositorio desencadenan compilaciones e implementaciones.
- Contenido estático **distribuido globalmente**, lo que permite que el contenido esté más cerca de los usuarios.
- **Certificados SSL gratuitos**, que se renuevan automáticamente.
- **Dominios personalizados** para proporcionar personalizaciones de marca a la aplicación.
- **Modelo de seguridad ágil** con un proxy inverso al llamar a las API, lo que no requiere ninguna configuración de CORS.
- **Integraciones del proveedor de autenticación** con Azure Active Directory, Facebook, Google, GitHub y Twitter.
- **Definición de roles de autorización personalizables** y asignaciones.
- **Reglas de enrutamiento de back-end** que permiten tener control total sobre el contenido y las rutas que atiende.
- **Versiones de almacenamiento provisional generadas** que se basan en las solicitudes de incorporación de cambios y permiten obtener versiones preliminares del sitio antes de su publicación.

## <a name="what-you-can-do-with-static-web-apps"></a>Qué puede hacer con Static Web Apps

- **Crear aplicaciones web modernas** con los marcos y bibliotecas de JavaScript, como [Angular](getting-started.md?tabs=angular), [React](getting-started.md?tabs=react), [Svelte](/learn/modules/publish-app-service-static-web-app-api/), [Vue](getting-started.md?tabs=vue) o con [Blazor](./deploy-blazor.md) para crear aplicaciones WebAssembly con un servidor back-end de [Azure Functions](apis.md).
- **Publicar sitios estáticos** con marcos como [Gatsby](publish-gatsby.md), [Hugo](publish-hugo.md) y [VuePress](publish-vuepress.md).
- **Implementar aplicaciones web** con marcos como [Next.js](deploy-nextjs.md) y [Nuxt.js](deploy-nuxtjs.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de la primera aplicación estática](getting-started.md)
