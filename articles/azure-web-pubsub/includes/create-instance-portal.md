---
title: Creación de una instancia de Azure Web PubSub con Azure Portal
description: Archivo de inclusión sobre la creación de una instancia de Azure Web PubSub con el portal
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: include
ms.date: 03/11/2021
ms.openlocfilehash: e147d6f954009f8ed4267f8b947438e2e60da0b4
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110487219"
---
## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com/](https://portal.azure.com/) con su cuenta de Azure.

## <a name="create-an-azure-web-pubsub-service-instance"></a>Creación de una instancia de servicio de Azure Web PubSub

La aplicación se conectará a una instancia de servicio de Web PubSub en Azure.

1. Seleccione el botón Nuevo de la esquina superior izquierda en Azure Portal. En la pantalla "Nuevo", escriba *Web PubSub* en el cuadro de búsqueda y presione Entrar. (También puede buscar Azure Web PubSub en la categoría `Web`).

:::image type="content" source="../media/create-instance-portal/search-web-pubsub-in-portal.png" alt-text="Captura de pantalla de la búsqueda de Azure Web PubSub en el portal.":::

2. Seleccione **Web PubSub** en los resultados de la búsqueda y, después, haga clic en **Crear**.

3. Escriba la siguiente configuración.

    | Configuración      | Valor sugerido  | Descripción                                        |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre del recurso** | Nombre único globalmente | Nombre que identifica la nueva instancia del servicio Web PubSub. Los caracteres válidos son `a-z`, `0-9` y `-`.  | 
    | **Suscripción** | Su suscripción | Suscripción en la que se va a crear esta instancia de servicio de Web PubSub. | 
    | **[Grupo de recursos](../../azure-resource-manager/management/overview.md)** |  myResourceGroup | Nombre del grupo de recursos nuevo en el que se va a crear la instancia del servicio Web PubSub. | 
    | **Ubicación** | Oeste de EE. UU. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de usted. |
    | **Plan de tarifa** | Gratuito | Pruebe el servicio Azure Web PubSub de forma gratuita. |
    | **Recuento de unidades** |  No aplicable | El recuento de unidades especifica cuántas conexiones puede aceptar la instancia de servicio de Web PubSub. Solo es configurable en el nivel Estándar. |

4. Seleccione **Crear** para empezar a implementar la instancia de servicio de Web PubSub.
