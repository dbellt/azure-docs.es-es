---
title: Instalación de Azure Percept DK
description: Conexión del kit de desarrollo a Azure e implementación del primer modelo de inteligencia artificial
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: quickstart
ms.date: 03/17/2021
ms.custom: template-quickstart
ms.openlocfilehash: 1b844eb66a287e103e453e242bec407b2f3a38e6
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109788416"
---
# <a name="set-up-your-azure-percept-dk-and-deploy-your-first-ai-model"></a>Instalación de Azure Percept DK e implementación del primer modelo de inteligencia artificial

Complete la experiencia de instalación de Azure Percept DK para configurar el kit de desarrollo e implementar su primer modelo de IA. Después de comprobar que la cuenta de Azure es compatible con Azure Percept, siga estos pasos:

- Conectar su kit de desarrollo a una red Wi-Fi
- Configurar un inicio de sesión de SSH con fines de acceso remoto al kit de desarrollo
- Crear una nueva instancia de IoT Hub para usarla con Azure Percept
- Conectar el kit de desarrollo a su instancia de IoT Hub y cuenta de Azure

Si tiene algún problema durante el proceso, consulte las posibles soluciones en la [guía de solución de problemas](./how-to-troubleshoot-setup.md).

> [!TIP]
> Puede volver a la configuración en cualquier momento para reinicializar el kit de desarrollo para conectarse a una nueva red Wi-Fi, crear un usuario SSH y volver a conectarse a IoT Hub, por ejemplo.

## <a name="prerequisites"></a>Prerequisites

- Un kit de desarrollo de Azure Percept DK
- Un equipo host basado en Windows, OS X o Linux con funcionalidad Wi-Fi y un explorador web.
- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita.](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- La cuenta de Azure debe tener el rol de **propietario** o **colaborador** en la suscripción. Siga los pasos que se indican a continuación para comprobar el rol de la cuenta de Azure. Para más información sobre las definiciones de roles de Azure, consulte la [documentación de control de acceso basado en roles de Azure](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles).

    > [!CAUTION]
    > Si tiene varias cuentas de Azure, el explorador puede almacenar en caché las credenciales de otra cuenta. Para evitar confusiones, se recomienda cerrar todas las ventanas del explorador sin usar e iniciar sesión en [Azure Portal](https://portal.azure.com/) antes de empezar la experiencia de instalación. Consulte la [guía de solución de problemas](./how-to-troubleshoot-setup.md) para obtener más información sobre cómo asegurarse de que ha iniciado sesión con la cuenta correcta.

### <a name="check-your-azure-account-role"></a>Comprobación del rol de la cuenta de Azure

Para comprobar si la cuenta de Azure tiene los roles de "propietario" o "colaborador" en la suscripción, siga estos pasos:

1. Vaya a [Azure Portal](https://portal.azure.com/) e inicie sesión con la misma cuenta de Azure que va a usar con Azure Percept.

1. Haga clic en el icono de las **suscripciones** (parecido a una tecla amarilla).

1. Seleccione la suscripción en la lista. Si no ve su suscripción en la lista, asegúrese de haber iniciado sesión con la cuenta de Azure correcta. Si desea crear una suscripción, siga [estos pasos](../cost-management-billing/manage/create-subscription.md).

1. En el menú Suscripción, seleccione **Control de acceso (IAM)** .
1. Haga clic en **View my access** (Ver mi acceso).
1. Compruebe el rol:
    - Si su rol aparece como **Lector** o usted recibe un mensaje indicando que no tiene permisos para ver los roles, debe seguir el proceso aplicable en su organización para elevar el rol de su cuenta.
    - Si su rol aparece como **propietario** o **colaborador**, la cuenta funcionará con Azure Percept y usted puede continuar con la experiencia de instalación.

## <a name="launch-the-azure-percept-dk-setup-experience"></a>Inicio de la experiencia de instalación de Azure Percept DK

1. Conecte el equipo host directamente al punto de acceso Wi-Fi del kit de desarrollo. Como la conexión a cualquier otra red Wi-Fi, abra la configuración de red e Internet en el equipo, haga clic en la red que se indica y escriba la contraseña de red cuando se le solicite:

    - **Nombre de red**: en función de la versión del sistema operativo del kit de desarrollo, el nombre del punto de acceso Wi-Fi es **scz-xxxx** o **apd-xxxx** (donde "xxxx" son los cuatro últimos dígitos de la dirección MAC del kit de desarrollo)
    - **Contraseña**: puede encontrarla en la Tarjeta de bienvenida que se incluye en el kit de desarrollo

    > [!WARNING]
    > Mientras está conectado al punto de acceso Wi-Fi de Azure Percept DK, el equipo host perderá temporalmente su conexión a Internet. Se interrumpirán las llamadas de videoconferencia en curso, las transmisiones web u otras experiencias que necesiten la red.

1. Una vez conectado al punto de acceso Wi-Fi del kit de desarrollo, el equipo host iniciará automáticamente la experiencia de instalación en una nueva ventana del explorador; y **your.new.device/** se mostrará en la barra de direcciones. Si la pestaña no se abre automáticamente, inicie la experiencia de instalación. Para ello, vaya a [http://10.1.1.1](http://10.1.1.1). Asegúrese de que el explorador haya iniciado sesión con las credenciales de la cuenta de Azure que va a usar con Azure Percept.

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-01-welcome.png" alt-text="Página de bienvenida.":::

    > [!CAUTION]
    > El servicio web de la experiencia de instalación se cerrará después de 30 minutos sin utilizarse. Si esto sucede, reinicie el kit de desarrollo para evitar problemas de conectividad con el punto de acceso Wi-Fi del kit de desarrollo.

## <a name="complete-the-azure-percept-dk-setup-experience"></a>Finalización de la experiencia de instalación de Azure Percept DK

1. Haga clic en **Siguiente** en la pantalla de **bienvenida**.

1. En la página **Conexión de red**, haga clic en **Connect to a new Wi-Fi network** (Conectarse a una nueva red Wi-Fi).

    Si ya ha conectado el kit de desarrollo a la red Wi-Fi, haga clic en **Omitir**.

1. Seleccione su red de Wi-Fi en la lista de redes disponibles y haga clic en **Conectar**. Escriba la contraseña de la red cuando se le solicite.

1. Una vez que el kit de desarrollo se ha conectado correctamente a la red elegida, la página mostrará la dirección IPv4 asignada al kit de desarrollo. **Anote la dirección IPv4 que se muestra en la página.** Necesitará la dirección IP al conectarse al kit de desarrollo a través de SSH para solucionar problemas y actualizar los dispositivos.

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-04-success-wi-fi.png" alt-text="Copia de la dirección IP.":::

    > [!NOTE]
    > La dirección IP puede cambiar con cada arranque del dispositivo.

1. Lea el contrato de licencia, seleccione **I have read and agree to the License Agreement** (He leído y acepto el contrato de licencia), en la parte inferior del contrato, y haga clic en **Next** (Siguiente).

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-05-eula.png" alt-text="Aceptación del CLUF.":::

1. Escriba un nombre de cuenta y una contraseña de SSH y **tome nota de la información de inicio de sesión para usarla posteriormente**.

    > [!NOTE]
    > SSH (Secure Shell) es un protocolo de red que le permite conectarse al kit de desarrollo de forma remota a través de un equipo host.

1. En la página siguiente, haga clic en **Setup as a new device** (Configurar como dispositivo nuevo) para crear un nuevo dispositivo en su cuenta de Azure.

1. Haga clic en **Copy** (Copiar) para copiar el código del dispositivo. Después haga clic en **Login to Azure** (Iniciar sesión en Azure).

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-08-copy-code.png" alt-text="Copia del código del dispositivo.":::

1. Se abrirá una nueva pestaña del explorador con una ventana que indica **Enter code** (Escribir código). Pegue el código en la ventana y haga clic en **Next** (Siguiente). NO cierre la pestaña de **bienvenida** con la experiencia de instalación.

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-09-enter-code.png" alt-text="Escritura del código del dispositivo.":::

1. Inicie sesión en Azure Percept con las credenciales de la cuenta de Azure que va a usar con el kit de desarrollo. Cuando finalice, mantenga abierta la pestaña del explorador.

    > [!CAUTION]
    > El explorador puede almacenar en caché automáticamente otras credenciales. Compruebe que está iniciando sesión con la cuenta correcta.

    Después de iniciar sesión correctamente en Azure Percent en el dispositivo, vuelva a la pestaña de **bienvenida** para continuar con la experiencia de instalación.

1. Cuando aparezca la página **Assign your device to your Azure IoT Hub** (Asignación de su dispositivo a Azure IoT Hub) en la pestaña de **bienvenida**, siga uno de estos pasos:

    - Si ya tiene una instancia de IoT Hub que desea usar con Azure Percept y aparece en esta página, selecciónela para ir directamente al paso 15.
    - Si no tiene ninguna instancia de IoT Hub o desea crear una nueva, haga clic en **Create a new Azure IOT Hub** (Crear una nueva instancia de Azure IoT Hub).

    > [!IMPORTANT]
    > Si tiene una instancia de IoT Hub, pero no aparece en la lista, es posible que haya iniciado sesión en Azure Percept con las credenciales erróneas. Consulte la [guía de solución de problemas de la instalación](./how-to-troubleshoot-setup.md) para obtener ayuda.

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-13-iot-hub-select.png" alt-text="Selección de una instancia de IoT Hub.":::

1. Para crear una nueva instancia de IoT Hub, rellene los siguientes campos:

    - Seleccione la suscripción de Azure que va a usar con Azure Percept.
    - Seleccione un grupo de recursos existente. Si no hay ninguno, haga clic en **Create new** (Crear nuevo) y siga las indicaciones.
    - Seleccione la región de Azure más próxima a su ubicación física.
    - Asigne un nombre a la nueva instancia de IoT Hub.
    - Seleccione el plan de tarifa S1 (estándar).

    > [!NOTE]
    > Si a la larga necesita un mayor [rendimiento de los mensajes](../iot-hub/iot-hub-scaling.md#message-throughput) con las aplicaciones de IA perimetrales, tiene la posibilidad de [actualizar IOT Hub a un nivel estándar superior](../iot-hub/iot-hub-upgrade.md) en Azure Portal en cualquier momento. Los niveles B y F no admiten Azure Percept.

1. La implementación de IoT Hub puede tardar algunos minutos. Cuando finalice, haga clic en **Registrar**.

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-16-iot-hub-success.png" alt-text="Instancia de IoT Hub implementada correctamente.":::

1. Asigne un nombre al kit de desarrollo y haga clic en **Next** (Siguiente).

1. Espere a que se descarguen los módulos del dispositivo. Esto tardará unos minutos.

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-18-finalize.png" alt-text="Finalización de la instalación.":::

1. Cuando vea la página **Device setup complete!** (Se completó la instalación del dispositivo), el kit de desarrollo se habrá vinculado correctamente a la instancia de IoT Hub y habrá descargado el software necesario. El kit de desarrollo se desconectará automáticamente del punto de acceso Wi-Fi que genera estas dos notificaciones:

    > [!NOTE]
    > Los contenedores de IoT Edge que se configuran como parte de este proceso usan certificados que expirarán a los 90 días. Los certificados se pueden volver a generar automáticamente. Para ello, solo hay que reiniciar IoT Edge. Para más información, consulte [Administración de certificados en un dispositivo IoT Edge](https://docs.microsoft.com/azure/iot-edge/how-to-manage-device-certificates).

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-19-0-warning.png" alt-text="Advertencia de desconexión de la experiencia de instalación.":::

1. Conecte el equipo host a la red Wi-Fi a la que conectó el kit de desarrollo en el paso 2.

1. Haga clic en **Continue to the Azure portal** (Continuar en Azure Portal).

    :::image type="content" source="./media/quickstart-percept-dk-setup/main-20-Azure-portal-continue.png" alt-text="Ir a Azure Percept Studio.":::

## <a name="view-your-dev-kit-video-stream-and-deploy-a-sample-model"></a>Visualización de la secuencia de vídeo del kit de desarrollo e implementación de un modelo de ejemplo

1. La [página de información general de Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819) es el punto de inicio para acceder a diferentes flujos de trabajo para el desarrollo de soluciones de IA perimetrales de nivel inicial y avanzado. Para empezar, haga clic en **Dispositivos** en el menú de la izquierda.

    :::image type="content" source="./media/quickstart-percept-dk-setup/portal-01-get-device-list.png" alt-text="Visualización de la lista de dispositivos.":::

1. Compruebe que el kit de desarrollo aparezca **conectado** y haga clic en él para ver la página del dispositivo.

    :::image type="content" source="./media/quickstart-percept-dk-setup/portal-02-select-device.png" alt-text="Seleccione el dispositivo.":::

1. Haga clic en **Ver el flujo del dispositivo**. Si esta es la primera vez que ve la secuencia de vídeo del dispositivo, observará una notificación de que se está implementando un nuevo modelo en la esquina superior derecha. Esta operación puede tardar unos minutos.

    :::image type="content" source="./media/quickstart-percept-dk-setup/portal-03-1-start-video-stream.png" alt-text="Ver la secuencia de vídeo.":::

    Una vez implementado el modelo, recibirá otra notificación con un vínculo **View stream** (Ver secuencia). Haga clic en el vínculo para ver la secuencia de vídeo de la cámara de Azure Percept Vision en una nueva ventana del explorador. El kit de desarrollo está precargado con un modelo de IA que detecta automáticamente muchos objetos comunes.

    :::image type="content" source="./media/quickstart-percept-dk-setup/portal-03-2-object-detection.png" alt-text="Ver la detección de objetos.":::

1. Azure Percept Studio también incluye una serie de modelos de IA de ejemplo. Para implementar un modelo de ejemplo en el kit de desarrollo, vuelva a la página del dispositivo y haga clic en **Deploy a sample model** (Implementar un modelo de ejemplo).

    :::image type="content" source="./media/quickstart-percept-dk-setup/portal-04-explore-prebuilt.png" alt-text="Exploración de modelos pregenerados.":::

1. Seleccione un modelo de ejemplo de la biblioteca y haga clic en **Deploy to Device** (Implementar en el dispositivo).

    :::image type="content" source="./media/quickstart-percept-dk-setup/portal-05-2-select-journey.png" alt-text="Ver detección de objetos en acción.":::

1. Una vez que el modelo se haya implementado correctamente, verá una notificación con un vínculo **View streamo** (Ver secuencia) en la esquina superior derecha de la pantalla. Para ver la inferencia del modelo en acción, haga clic en el vínculo de la notificación o vuelva a la página del dispositivo y haga clic en **Ver el flujo del dispositivo**. Todos los modelos que se ejecutaran previamente en el kit de desarrollo se reemplazarán por el nuevo modelo.

## <a name="video-walkthrough"></a>Tutorial en vídeo

Para ver un tutorial con imágenes de los pasos descritos anteriormente, consulte el vídeo siguiente (la experiencia de instalación comienza en 0:50).

</br>

> [!VIDEO https://www.youtube.com/embed/-dmcE2aQkDE]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una solución de visión sin código](./tutorial-nocode-vision.md)