---
title: Rotación de claves de acceso para el servicio Azure Web PubSub
description: Información general sobre por qué el cliente debe rotar periódicamente las claves de acceso y cómo hacerlo.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: how-to
ms.date: 03/23/2021
ms.openlocfilehash: 73c31051a177dc16711048bec984a90c786a28f2
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167104"
---
# <a name="how-to-rotate-access-key-for-azure-web-pubsub-service"></a>Rotación de claves de acceso para el servicio Azure Web PubSub

Cada instancia del servicio Azure Web PubSub tiene un par de claves de acceso, llamadas claves principal y secundaria, que se usan para autenticar a los clientes cuando se realizan solicitudes al servicio. Las claves están asociadas con la dirección URL del punto de conexión de instancia. Mantenga seguras las claves y realice su rotación de forma periódica. Como se le proporcionan dos claves de acceso, puede mantener las conexiones con una mientras se regenera la otra.

## <a name="why-rotate-access-keys"></a>¿Por qué se deben rotar las claves de acceso?

Por motivos de seguridad y para cumplir los requisitos de cumplimiento, cambie periódicamente las claves de acceso.

## <a name="regenerate-access-keys"></a>Regenerar las claves de acceso

1. Vaya a [Azure Portal](https://portal.azure.com/) e inicie sesión con sus credenciales.

1. Busque la sección **Claves** en la instancia del servicio Azure Web PubSub con las claves que desee regenerar.

1. Seleccione **Claves** en el menú de navegación.

1. Haga clic en **Regenerar clave principal** o **Regenerar clave secundaria**.

   Se crean y se muestran una clave nueva y su correspondiente cadena de conexión.

También puede regenerar las claves mediante la CLI de Azure, una vez que el servicio Azure Web PubSub esté disponible con carácter general.

## <a name="update-configurations-with-new-connection-strings"></a>Actualización de configuraciones con nuevas cadenas de conexión

1. Copie la cadena de conexión recién generada.

1. Actualice todas las configuraciones para usar la nueva cadena de conexión.

1. Reinicie la aplicación según sea necesario.

## <a name="forced-access-key-regeneration"></a>Regeneración forzada de claves de acceso

En determinadas situaciones, el servicio Azure Web PubSub puede exigir que se regeneren las claves de acceso. El servicio se lo notifica a los clientes mediante correo electrónico y en el portal. Si recibe este tipo de comunicación o aparecen errores del servicio debido a una clave de acceso, siga las instrucciones de esta guía para cambiar las claves.