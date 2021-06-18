---
author: erhopf
manager: nitinme
ms.service: applied-ai-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 03/04/2021
ms.author: erhopf
ms.openlocfilehash: 90344270938c778d2a264292049d723810481c26
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111893361"
---
## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Un recurso del Lector inmersivo configurado para la autenticación de Azure Active Directory. Siga [estas instrucciones](../../how-to-create-immersive-reader.md) para realizar la configuración.  Necesitará algunos de los valores que se crean aquí cuando configure las propiedades del entorno. Guarde la salida de la sesión en un archivo de texto para futuras referencias.
* [macOS](https://www.apple.com/macos).
* [Git](https://git-scm.com/).
* [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk).
* [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12).

## <a name="configure-authentication-credentials"></a>Configuración de las credenciales de autenticación

1. Abra Xcode y abra **immersive-reader-sdk/js/samples/ios/quickstart-swift/quickstart-swift.xcodeproj**.
1. En el menú superior, seleccione **Product** > **Scheme** > **Edit Scheme** (Producto > Esquema > Editar esquema).
1. En la vista **Run** (Ejecutar), seleccione la pestaña **Arguments** (Argumentos).
1. En la sección **Environment Variables** (Variables de entorno), agregue los siguientes nombres y valores. Proporcione los valores especificados al crear el recurso del Lector inmersivo.

    ```text
    TENANT_ID=<YOUR_TENANT_ID>
    CLIENT_ID=<YOUR_CLIENT_ID>
    CLIENT_SECRET<YOUR_CLIENT_SECRET>
    SUBDOMAIN=<YOUR_SUBDOMAIN>
    ```

No confirme este cambio en el control de código fuente, ya que contiene secretos que no deben hacerse públicos.

## <a name="start-the-immersive-reader-with-sample-content"></a>Inicio del Lector inmersivo con contenido de ejemplo

En Xcode, seleccione **Ctrl+R** para ejecutar el proyecto.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y [Referencia del SDK del Lector inmersivo](../../reference.md).
* Puede ver ejemplos de código en [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/).
