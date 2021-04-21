---
title: Rotación de claves de acceso para Azure SignalR Service
description: Información general acerca de los motivos por los qué el cliente debe cambiar periódicamente las claves de acceso y cómo hacerlo con la GUI de Azure Portal y la CLI de Azure.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 03/01/2019
ms.author: zhshang
ms.openlocfilehash: cd03720b65883c5f43b11837d4a88a705209fb10
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764106"
---
# <a name="how-to-rotate-access-key-for-azure-signalr-service"></a>Rotación de claves de acceso para Azure SignalR Service

Cada instancia de Azure SignalR Service tiene un par de claves de acceso, llamadas claves principal y secundaria, que se usan para autenticar a los clientes de SignalR cuando se realizan solicitudes al servicio. Las claves están asociadas con la dirección URL del punto de conexión de instancia. Mantenga seguras las claves y realice su rotación de forma periódica. Como se le proporcionan dos claves de acceso, puede mantener las conexiones con una mientras se regenera la otra.

## <a name="why-rotate-access-keys"></a>¿Por qué se deben rotar las claves de acceso?

Por motivos de seguridad y para cumplir los requisitos de cumplimiento, cambie periódicamente las claves de acceso.

## <a name="regenerate-access-keys"></a>Regenerar las claves de acceso

1. Vaya a [Azure Portal](https://portal.azure.com/) e inicie sesión con sus credenciales.

1. Busque la sección **Claves** en la instancia de Azure SignalR Service con las claves que desee regenerar.

1. Seleccione **Claves** en el menú de navegación.

1. Haga clic en **Regenerar clave principal** o **Regenerar clave secundaria**.

   Se crean y se muestran una clave nueva y su correspondiente cadena de conexión.

   ![Regeneración de claves](media/signalr-howto-key-rotation/regenerate-keys.png)

Las claves también se pueden regenerar mediante la [CLI de Azure](/cli/azure/signalr/key#az_signalr_key_renew).

## <a name="update-configurations-with-new-connection-strings"></a>Actualización de configuraciones con nuevas cadenas de conexión

1. Copie la cadena de conexión recién generada.

1. Actualice todas las configuraciones para usar la nueva cadena de conexión.

1. Reinicie la aplicación según sea necesario.

## <a name="forced-access-key-regeneration"></a>Regeneración forzada de claves de acceso

En determinadas situaciones, Azure SignalR Service puede exigir que se vuelvan una generar las claves de acceso. El servicio se lo notifica a los clientes mediante correo electrónico y en el portal. Si recibe este tipo de comunicación o aparecen errores del servicio debido a una clave de acceso, siga las instrucciones de esta guía para cambiar las claves.

## <a name="next-steps"></a>Pasos siguientes

Como buena práctica de seguridad, cambie las claves de acceso con frecuencia.

En esta guía ha aprendido a volver a generar las claves de acceso. Continúe con los siguientes tutoriales sobre la autenticación con OAuth o Azure Functions.

> [!div class="nextstepaction"]
> [Integración con la identidad de ASP.NET Core](./signalr-concept-authenticate-oauth.md)

> [!div class="nextstepaction"]
> [Compilación de una aplicación en tiempo real sin servidor con autenticación](./signalr-tutorial-authenticate-azure-functions.md)