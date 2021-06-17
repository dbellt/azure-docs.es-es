---
author: erhopf
manager: nitinme
ms.service: applied-ai-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 03/04/2021
ms.author: erhopf
ms.openlocfilehash: c2cadc349c24bb59e6ab97a89a8f5182a9b96632
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111893376"
---
## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Un recurso del Lector inmersivo configurado para la autenticación de Azure Active Directory. Siga [estas instrucciones](../../how-to-create-immersive-reader.md) para realizar la configuración.  Necesitará algunos de los valores que se crean aquí cuando configure las propiedades del entorno. Guarde la salida de la sesión en un archivo de texto para futuras referencias.
* [Git](https://git-scm.com/).
* [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk).
* [Android Studio](https://developer.android.com/studio).

## <a name="configure-authentication-credentials"></a>Configuración de las credenciales de autenticación

1. Inicie Android Studio y abra el proyecto desde el directorio **immersive-reader-sdk/js/samples/quickstart-java-android** (Java) o **immersive-reader-sdk/js/samples/quickstart-kotlin** (Kotlin).

1. Cree un archivo denominado **env** dentro de la carpeta **/assets**. Agregue los siguientes nombres y valores y proporcione los valores según corresponda. No confirme este archivo en el control de código fuente, ya que contiene secretos que no deben hacerse públicos.
    
    ```text
    TENANT_ID=<YOUR_TENANT_ID>
    CLIENT_ID=<YOUR_CLIENT_ID>
    CLIENT_SECRET=<YOUR_CLIENT_SECRET>
    SUBDOMAIN=<YOUR_SUBDOMAIN>
    ```

## <a name="start-the-immersive-reader-with-sample-content"></a>Inicio del Lector inmersivo con contenido de ejemplo

Elija un emulador de dispositivos de AVD Manager y ejecute el proyecto.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y [Referencia del SDK del Lector inmersivo](../../reference.md).
* Puede ver ejemplos de código en [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/).