---
author: baanders
description: archivo de inclusión para publicar y configurar la aplicación de función
ms.service: digital-twins
ms.topic: include
ms.date: 03/15/2021
ms.author: baanders
ms.openlocfilehash: 463661af3f576b6382aacee9abefaa6f3caa7957
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110615936"
---
Para instrucciones sobre cómo hacerlo, consulte la sección [Publicación de la aplicación de funciones en Azure](../articles/digital-twins/how-to-create-azure-function.md#publish-the-function-app-to-azure) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

> [!IMPORTANT]
> La primera vez que se crea la aplicación de funciones en la [sección de requisitos previos](#prerequisites), es posible que ya se haya asignado un rol de acceso para la función y se haya configurado la aplicación para que acceda a la instancia de Azure Digital Twins. Estos deben realizarse una vez para toda la aplicación de función, por lo que debe comprobar que se han completado en la aplicación antes de continuar. Puede encontrar instrucciones en la sección [Configuración del acceso de seguridad para la aplicación de funciones](../articles/digital-twins/how-to-create-azure-function.md#set-up-security-access-for-the-function-app) del artículo *Ingesta de telemetría de IoT Hub*.