---
title: 'Tutorial: Configuración de la nube del servicio Azure IoT Hub Device Provisioning en el portal'
description: Este tutorial le indica la configuración de los recursos en la nube para el aprovisionamiento de dispositivos en [Azure Portal](https://portal.azure.com) con IoT Hub Device Provisioning Service (DPS).
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: 5707d873bbf67add78f9a8e29fcb9135898486d6
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107226844"
---
# <a name="tutorial-configure-cloud-resources-for-device-provisioning-with-the-iot-hub-device-provisioning-service"></a>Tutorial: Configurar los recursos de la nube para el aprovisionamiento de dispositivos con el servicio IoT Hub Device Provisioning

Este tutorial muestra cómo configurar la nube para el aprovisionamiento automático de dispositivos mediante el servicio IoT Hub Device Provisioning. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar Azure Portal para crear un servicio IoT Hub Device Provisioning y obtener el ámbito del identificador
> * Crear un centro de IoT
> * Vincular la instancia de IoT Hub al servicio Device Provisioning
> * Establecer la directiva de asignación del servicio Device Provisioning

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-device-provisioning-service-instance-and-get-the-id-scope"></a>Creación de una instancia del servicio Device Provisioning y obtención del ámbito del identificador

Siga estos pasos para crear una nueva instancia del servicio Device Provisioning.

1. En la esquina superior izquierda de Azure Portal, haga clic en **Crear un recurso**.

2. En el cuadro de búsqueda, escriba **aprovisionamiento de dispositivos**. 

3. Haga clic en el **servicio Azure IoT Hub Device Provisioning**.

4. Rellene el formulario del **servicio IoT Hub Device Provisioning** con la siguiente información:
    
   | Valor       | Valor sugerido | Descripción | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | **Nombre** | Cualquier nombre único | -- | 
   | **Suscripción** | Su suscripción  | Para más información acerca de sus suscripciones, consulte [Suscripciones](https://account.windowsazure.com/Subscriptions). |
   | **Grupos de recursos** | myResourceGroup | Para conocer cuáles son los nombres de grupo de recursos válidos, consulte el artículo [Convenciones de nomenclatura](/azure/architecture/best-practices/resource-naming). |
   | **Ubicación** | Cualquier ubicación válida | Para obtener información acerca de las regiones, consulte [Regiones de Azure](https://azure.microsoft.com/regions/). |   

   ![Especificación de la información básica del servicio de aprovisionamiento de dispositivos en el portal](./media/tutorial-set-up-cloud/create-iot-dps-portal.png)

5. Haga clic en **Crear**. Transcurridos unos instantes, se crea la instancia de Device Provisioning Service y aparece la página **Información general**.

6. En la página **Información general** de la nueva instancia del servicio, copie el valor de **Ámbito de id.** para su uso posterior. Este valor se usa para diferenciar los identificadores de registro y, además, garantiza que el identificador de registro es único.

7. Copie también el valor del **punto de conexión de servicio** para su uso posterior. 

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

### <a name="retrieve-connection-string-for-iot-hub"></a>Recuperación de la cadena de conexión del centro de IoT

[!INCLUDE [iot-hub-include-find-connection-string](../../includes/iot-hub-include-find-connection-string.md)]

Ya se creó IoT Hub y ya tiene el nombre de host y la cadena de conexión de IoT Hub que necesita para completar el resto del tutorial.

## <a name="link-the-device-provisioning-service-to-an-iot-hub"></a>Vinculación del servicio Device Provisioning a una instancia de IoT Hub

El paso siguiente es vincular el servicio Device Provisioning a una instancia de IoT Hub para que este servicio pueda registrar dispositivos en ese concentrador. Este servicio solo puede aprovisionar los dispositivos para las instancias de IoT Hub que se hayan vinculado a él. Siga estos pasos.

1. En la página **Todos los recursos**, haga clic en la instancia del servicio Device Provisioning que creó anteriormente.

2. En la página del servicio Device Provisioning, haga clic en **Linked IoT hubs** (Centros de IoT vinculados).

3. Haga clic en **Agregar**.

4. En la página **Agregar un vínculo a IoT Hub**, proporcione la información siguiente y haga clic en **Guardar**:

    * **Suscripción:** asegúrese de que está seleccionada la suscripción que contiene el centro de IoT. Puede vincular a un centro de IoT que resida en una suscripción diferente.

    * **Centro de IoT:** elija el nombre del centro de IoT que desea vincular con esta instancia de Device Provisioning Service.

    * **Directiva de acceso:** seleccione **iothubowner** como las credenciales para establecer el vínculo con el centro de IoT.

   ![Vínculo del nombre del centro al servicio de aprovisionamiento de dispositivos en el portal](./media/tutorial-set-up-cloud/link-iot-hub-to-dps-portal.png)

## <a name="set-the-allocation-policy-on-the-device-provisioning-service"></a>Establecer la directiva de asignación del servicio Device Provisioning

La directiva de asignación es una configuración del servicio IoT Hub Device Provisioning que determina cómo se asignan los dispositivos a un centro de IoT. Hay tres directivas de asignación admitidas: 

1. **Latencia más baja**: los dispositivos se aprovisionan en una instancia de IoT Hub en función del centro con la latencia más baja en el dispositivo.

2. **Distribución uniformemente ponderada** (predeterminada): las instancias de IoT Hub vinculadas tienen la misma probabilidad de tener dispositivos aprovisionados para ellos. Esta configuración es la predeterminada. Si va a aprovisionar dispositivos para un único centro de IoT Hub, puede mantener esta configuración. 

3. **Configuración estática a través de la lista de inscripción**: la especificación de la instancia de IoT deseada en la lista de inscripción tiene prioridad sobre la directiva de asignación del nivel del servicio Device Provisioning.

Para establecer la directiva de asignación, en la página del servicio Device Provisioning haga clic en **Manage allocation policy** (Administrar directiva de asignación). Asegúrese de que la directiva de asignación está establecida en **Distribución uniformemente ponderada** (la opción predeterminada). Si realiza cambios, haga clic en **Guardar** cuando haya terminado.

![Administrar directiva de asignación](./media/tutorial-set-up-cloud/iot-dps-manage-allocation.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Otros tutoriales de esta colección se basan en los valores de este. Si tiene previsto seguir trabajando con las siguientes guías de inicio rápido o tutoriales, no elimine los recursos creados en este tutorial. Si no tiene previsto continuar, siga estos pasos para eliminar todos los recursos creados en este tutorial en Azure Portal.

1. Desde el menú de la izquierda en Azure Portal, haga clic en **Todos los recursos** y seleccione la instancia del servicio IoT Hub Device Provisioning. En la parte superior de la página **Todos los recursos**, haga clic en **Eliminar**.  

2. Desde el menú de la izquierda en Azure Portal, haga clic en **Todos los recursos** y seleccione su centro de IoT. En la parte superior de la página **Todos los recursos**, haga clic en **Eliminar**.
 
## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Usar Azure Portal para crear un servicio IoT Hub Device Provisioning y obtener el ámbito del identificador
> * Crear un centro de IoT
> * Vincular la instancia de IoT Hub al servicio Device Provisioning
> * Establecer la directiva de asignación del servicio Device Provisioning

Pase al siguiente tutorial para aprender a configurar el dispositivo para el aprovisionamiento.

> [!div class="nextstepaction"]
> [Configurar el dispositivo para el aprovisionamiento](tutorial-set-up-device.md)
