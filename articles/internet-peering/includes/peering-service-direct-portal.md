---
title: archivo de inclusión
titleSuffix: Azure
description: archivo de inclusión
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: include
ms.date: 3/18/2020
ms.author: derekol
ms.openlocfilehash: e5804aa1b005e670d8b430b1c0a3bd62efd0bb06
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "81687047"
---
1. Seleccione una conexión de emparejamiento que quiera habilitar para Azure Peering Service. Luego, seleccione **...**  > **Edit connection** (Editar conexión).
    > [!div class="mx-imgBorder"]
    > ![Conexión de emparejamiento: Edit connection (Editar conexión)](../media/setup-direct-modify-editconnection.png)
1. En **Use for Peering Service** (Uso para Peering Service), seleccione **Enabled** (Habilitado) y, luego, elija **Save** (Guardar).
    > [!div class="mx-imgBorder"]
    > ![Conexión de emparejamiento: Habilitar Peering Service](../media/setup-direct-modify-editconnectionsettings-peering-service.png)
1. En la pantalla **Overview** (Información general), verá los detalles de la implementación. Cuando la implementación finalice, seleccione **Ir al recurso**.
    > [!div class="mx-imgBorder"]
    > ![Se completó la implementación](../media/setup-direct-modify-overview-deployment-complete.png)

1. En el panel **Registered Prefixes** (Prefijos registrados), seleccione **Add registered prefix** (Agregar prefijo registrado).
    > [!div class="mx-imgBorder"]
    > ![Adición del prefijo registrado](../media/setup-direct-modify-add-registered-prefix.png)
1. Para registrar un prefijo, seleccione un valor en **Nombre** y **Prefijo** y elija **Guardar**.
    > [!div class="mx-imgBorder"]
    >  ![Registro de un prefijo](../media/setup-direct-modify-register-a-prefix.png) 

1. Una vez creado el prefijo, lo verá en la lista **Registered Prefixes** (Prefijos registrados). En **Nombre**, seleccione el nombre del prefijo para ver más detalles.
    > [!div class="mx-imgBorder"]
    > ![Conexiones y prefijos registrados](../media/setup-direct-modify-registered-prefixes.png)
1. En la página de prefijos registrados, verá los detalles completos, junto con la **clave de prefijo** de cada prefijo. Esta clave se debe proporcionar al cliente al que el proveedor ISP asignó este prefijo. Luego, el cliente puede registrar su prefijo dentro de su suscripción con esta clave.
    > [!div class="mx-imgBorder"]
    > ![Prefijo con clave de prefijo](../media/setup-direct-modify-registered-prefix-detail.png)