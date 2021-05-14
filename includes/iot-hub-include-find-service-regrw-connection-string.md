---
title: archivo de inclusión
description: archivo de inclusión
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 04/03/2020
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: e2df24a85b39d1a66a162a826dbc3985a3908d53
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109508378"
---
<!-- This tells how to create a custom shared access policy that has service connect and registry RW permissions for your IoT hub and get the connection string for it-->

Para crear una directiva de acceso compartido que conceda los permisos de  **conexión del servicio** y **escritura en el registro**, y para obtener una cadena de conexión para esta directiva, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos**. Seleccione el grupo de recursos donde se encuentra el centro y, a continuación, seleccione el centro en la lista de recursos.

1. En el panel de la izquierda del centro, seleccione **Directivas de acceso compartido**.

1. En el menú superior situado encima de la lista de directivas, seleccione **Agregar**.

1. En **Agregar una directiva de acceso compartida**, escriba un nombre descriptivo para la directiva, por ejemplo *serviceAndRegistryReadWrite*. En **Permisos**, seleccione **Registry write** (Escritura en el Registro) y **Service connect** (Conexión del servicio) y, después, seleccione **Create** (Crear) (el permiso de **lectura del Registro** se incluye automáticamente cuando se selecciona **Registry write** [Escritura en el Registro]).

    ![Cómo agregar una nueva directiva de acceso compartido](./media/iot-hub-include-find-service-regrw-connection-string/iot-hub-add-svc-regrw-policy.png)

1. Seleccione la directiva nueva en la lista de directivas.

1. En **Claves de acceso compartido**, seleccione el icono de **Cadena de conexión: clave principal** y guarde el valor.

    ![Recuperación de la cadena de conexión](./media/iot-hub-include-find-service-regrw-connection-string/iot-hub-get-connection-string.png)

Para obtener más información sobre las directivas de acceso compartido y los permisos de IoT Hub, consulte [Permisos y control del acceso](../articles/iot-hub/iot-hub-dev-guide-sas.md#access-control-and-permissions).
