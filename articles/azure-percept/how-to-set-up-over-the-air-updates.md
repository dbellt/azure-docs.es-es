---
title: Configuración de Azure IoT Hub para la implementación de actualizaciones de forma inalámbrica
description: Aprenda a configurar Azure IoT Hub para implementar actualizaciones de forma inalámbrica en Azure Percept DK
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 03/30/2021
ms.custom: template-how-to
ms.openlocfilehash: 81697efc7832adf62d26c751d1e6e93070c2579b
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109684080"
---
# <a name="how-to-set-up-azure-iot-hub-to-deploy-over-the-air-updates-to-your-azure-percept-dk"></a>Configuración de Azure IoT Hub para la implementación de actualizaciones de forma inalámbrica en Azure Percept DK

Mantenga Azure Percept DK protegido y actualizado con actualizaciones de forma inalámbrica. En unos pocos pasos sencillos, podrá configurar el entorno de Azure con Device Update para IoT Hub e implementar las actualizaciones más recientes en Azure Percept DK.

## <a name="prerequisites"></a>Requisitos previos

- Kit de desarrollo Azure Percept DK
- [Suscripción de Azure](https://azure.microsoft.com/free/)
- [Experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md): ya ha conectado el dispositivo DevKit a una red Wi-Fi, creado una instancia de IoT Hub y conectado el dispositivo DevKit a esa instancia

## <a name="create-a-device-update-account"></a>Creación de una cuenta de Device Update

1. Vaya a [Azure Portal](https://portal.azure.com) e inicie sesión mediante la cuenta de Azure que utiliza con Azure Percept.

1. En la barra de búsqueda de la parte superior de la página, escriba **Device Update for IoT Hubs**.

1. Seleccione **Device Update for IoT Hubs** cuando aparezca en la barra de búsqueda.

1. Seleccione el botón **+Agregar** en la parte superior izquierda de la página.

1. Seleccione la **suscripción de Azure** y el **grupo de recursos** asociados al dispositivo de Azure Percept y al IoT Hub.

1. Especifique un **nombre** y una **ubicación** para la cuenta de Device Update.

1. Active la casilla con el texto **Asigne el rol de administrador de Device Update**. 

1. Revise los detalles y, luego, seleccione **Revisión y creación**.

1. Seleccione el botón **Crear**.

1. Una vez finalizada la implementación, haga clic en **Ir al recurso**.

## <a name="create-a-device-update-instance"></a>Creación de una instancia de Device Update

1. En el recurso de Device Update para IoT Hub, haga clic en **Instancias** en **Instance Management** (Administración de instancias).

1. Haga clic en **+ Crear**, especifique un nombre de instancia y seleccione la instancia de IoT Hub asociada al dispositivo de Azure Percept. Esta operación puede tardar algunos minutos en completarse.

1. Haga clic en **Crear**.

## <a name="configure-iot-hub"></a>Configuración de IoT Hub

1. En la página **Instancias** de la administración de instancias, espere a que la instancia de Device Update pase al estado **correcto**. Haga clic en el icono **Actualizar** para actualizar el estado.

1. Seleccione la instancia que se ha creado automáticamente y haga clic en **Configurar IoT Hub**. En el panel izquierdo, seleccione **I agree to make these changes** (Acepto realizar estos cambios) y haga clic en **Actualizar**.

1. Espere hasta que se complete el proceso correctamente.

## <a name="configure-access-control-roles"></a>Configuración de roles de control de acceso

El paso final le permitirá conceder permisos a los usuarios para publicar e implementar actualizaciones.

1. En el recurso de Device Update para IoT Hub, haga clic en **Control de acceso (IAM)** .

1. Haga clic en **+Agregar** y, después, seleccione **Agregar asignación de roles**.

1. En **Rol**, seleccione **Device Update Administrator** (Administrador de Device Update). En **Asignar acceso a**, seleccione **User, group, or service principle** (Usuario, grupo o entidad de servicio). En **Seleccionar**, elija su cuenta o la cuenta de la persona que va a implementar las actualizaciones. Haga clic en **Guardar**.

> [!TIP]
> Si desea dar acceso a más personas de su organización, puede repetir este paso y convertir a cada uno de estos usuarios en **administrador de Device Update**.

## <a name="next-steps"></a>Pasos siguientes

Ahora ya está listo para [actualizar el kit de desarrollo de Azure Percept de forma inalámbrica](./how-to-update-over-the-air.md) mediante Device Update para IoT Hub.