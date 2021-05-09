---
title: archivo de inclusión
description: archivo de inclusión
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: ae9b749f62dc74a6cb0dfea3701fd4ebac11c188
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107931580"
---
## <a name="enable-logging"></a>Habilitar registro

Para ayudar en los escenarios de solución de problemas de depuración y de errores de autenticación, la Biblioteca de autenticación de Microsoft proporciona compatibilidad con el registro integrado. El registro en cada biblioteca se trata en los siguientes artículos:

:::row:::
    :::column:::
        - [Registro en MSAL.NET](../articles/active-directory/develop/msal-logging-dotnet.md)
        - [Inicio de sesión en MSAL para Android](../articles/active-directory/develop/msal-logging-android.md)
        - [Registro en MSAL.js](../articles/active-directory/develop/msal-logging-js.md)
    :::column-end:::
    :::column:::
        - [Inicio de sesión en MSAL para iOS/macOS](../articles/active-directory/develop/msal-logging-ios.md)
        - [Registro en MSAL para Java](../articles/active-directory/develop/msal-logging-java.md)
        - [Inicio de sesión en MSAL para Python](../articles/active-directory/develop/msal-logging-python.md)
    :::column-end:::
:::row-end:::

Estas son algunas sugerencias para la recopilación de datos:

- Los usuarios pueden pedir ayuda cuando tengan problemas. Un procedimiento recomendado es capturar y almacenar temporalmente los registros. Proporcione una ubicación donde los usuarios puedan cargar los registros. MSAL proporciona extensiones de registro para capturar información detallada acerca de la autenticación.

- Si hay telemetría disponible, habilítela mediante MSAL para recopilar datos acerca de cómo inician sesión en la aplicación los usuarios.


## <a name="validate-your-integration"></a>Validación de la integración

Pruebe la integración siguiendo la [lista de comprobación de integración de la Plataforma de identidad de Microsoft](../articles/active-directory/develop/identity-platform-integration-checklist.md).

## <a name="build-for-resilience"></a>Compilación para resistencia

Obtenga información sobre cómo aumentar la resistencia en la aplicación. Para obtener más información, consulte [Aumento de la resistencia de las aplicaciones de autenticación y autorización desarrolladas](../articles/active-directory/fundamentals/resilience-app-development-overview.md).
