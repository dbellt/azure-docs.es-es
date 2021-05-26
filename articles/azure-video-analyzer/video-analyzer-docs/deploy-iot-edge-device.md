---
title: 'Implementación de Azure Video Analyzer en un dispositivo IoT Edge: Azure'
description: En este artículo se enumeran los pasos que ayudarán a implementar Azure Video Analyzer en el dispositivo IoT Edge. Lo haría, por ejemplo, si tiene acceso a una máquina Linux local.
ms.topic: how-to
ms.date: 04/07/2021
ms.openlocfilehash: f81d441b0070777837cc938721705335734fa3a3
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388554"
---
# <a name="deploy-azure-video-analyzer-to-an-iot-edge-device"></a>Implementación de Azure Video Analyzer en un dispositivo IoT Edge

En este artículo se describe cómo puede implementar el módulo perimetral de Azure Video Analyzer en un dispositivo IoT Edge que no tenga otros módulos instalados anteriormente. Cuando termine los pasos de este artículo, tendrá una cuenta de Video Analyzer creada y el módulo Video Analyzer implementado en el dispositivo IoT Edge, junto con un módulo que simula una cámara IP compatible con RTSP. El proceso está pensado para su uso con los inicios rápidos y tutoriales de Video Analyzer. Debe revisar el artículo [Preparación para la producción y procedimientos recomendados](production-readiness.md) si tiene previsto implementar el módulo Video Analyzer para su uso en producción.

> [!NOTE]
> El proceso descrito en este artículo desinstalará los módulos perimetrales, si los hay, que están instalados en el dispositivo IoT Edge.


## <a name="prerequisites"></a>Requisitos previos

* Un dispositivo x86-64 o ARM64 que ejecute uno de los [sistemas operativos Linux admitidos](../../iot-edge/support.md#operating-systems).
* Una cuenta de Azure que tenga una suscripción activa
* [Creación y configuración de IoT Hub](../../iot-hub/iot-hub-create-through-portal.md).
* [Registro de un dispositivo de IoT Edge](../../iot-edge/how-to-register-device.md).
* [Instalación del entorno de ejecución de Azure IoT Edge en sistemas Linux basados en Debian](../../iot-edge/how-to-install-iot-edge.md)


## <a name="create-resources-on-iot-edge-device"></a>Creación de recursos en el dispositivo IoT Edge

El módulo Azure Video Analyzer debe configurarse para ejecutarse en el dispositivo IoT Edge con una cuenta de usuario local sin privilegios. El módulo necesita ciertas carpetas locales para almacenar los datos de configuración de la aplicación. Para esta guía paso a paso, se emplea un [simulador RTSP](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) que retransmite una fuente de vídeo en tiempo real a un módulo de AVA para su análisis. Este simulador toma como entrada archivos de vídeo grabados previamente desde un directorio de entrada. El siguiente script preparará el dispositivo para que pueda usarse con nuestros inicios rápidos y tutoriales.

https://aka.ms/ava/prepare-device

`bash -c "$(curl -sL https://aka.ms/ava-edge/prep_device)"`

El script prep-device usado anteriormente automatiza la tarea de crear carpetas de entrada y configuración, descargar archivos de entrada de vídeo y crear cuentas de usuario con los privilegios correctos. Una vez que el comando finalice correctamente, debería ver las siguientes carpetas creadas en el dispositivo perimetral. 

* `/home/localedgeuser/samples`
* `/home/localedgeuser/samples/input`
* `/var/lib/videoanalyzer`
* `/var/media`

    Observe los archivos de vídeo ("*.mkv") de la carpeta /home/localedgeuser/samples/input, que se usan para simular vídeo en directo. 

## <a name="creating-azure-resources-and-deploying-edge-modules"></a>Creación de recursos de Azure e implementación de módulos perimetrales
El siguiente paso consiste en crear los recursos de Azure necesarios (cuenta de Video Analyzer, cuenta de almacenamiento, identidad administrada asignada por el usuario), registrando un módulo perimetral de Video Analyzer con la cuenta de Video Analyzer e implementando el módulo perimetral de Video Analyzer y el módulo del simulador RTSP en el dispositivo IoT Edge.

Haga clic en el botón **Implementar en Azure**

> [!WARNING]
> No lo use con dispositivos IoT Edge que ya tengan instalados módulos perimetrales, como Percept DK. Tampoco se admite con Azure Stack Edge.

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava/click-to-deploy/form)

1. Seleccione la **suscripción**.
2. Seleccione su **región** preferida
3. Seleccione el **grupo de recursos** al que pertenecen los dispositivos IoT Hub e IoT Edge
4. En el menú desplegable de **¿Necesita un dispositivo perimetral?** , seleccione la opción **_Usar un dispositivo perimetral existente_**.
5. Haga clic en **Siguiente**
![Captura de pantalla del formulario de implementación inicial](./media/deploy-iot-edge-device/project-details.png)

1. Seleccione el **nombre de IoT Hub existente** al que se conecta el dispositivo IoT Edge
1. Haga clic en **Siguiente**
![Captura de pantalla del segundo formulario de implementación](./media/deploy-iot-edge-device/iot-hub-name.png)

1. En la página final, haga clic en **Crear**

Los recursos de Azure pueden tardar unos minutos en crearse. Asimismo, los módulos perimetrales pueden tardar el mismo tiempo en implementarse.


### <a name="verify-your-deployment"></a>Comprobación de la implementación

Después de crear la implementación, en Azure Portal, vaya a la página del dispositivo IoT Edge del centro de IoT.

1. Seleccione el dispositivo IoT Edge elegido como destino con la implementación para abrir sus detalles.
2. En los detalles del dispositivo, compruebe que los módulos aparecen como **Especificado en la implementación y Notificado por el dispositivo**.

Puede tardar unos minutos para que los módulos se inicien en el dispositivo y, a continuación, notifique a IoT Hub. Actualice la página para ver el estado actualizado.
Código de estado: 200: OK significa que [el entorno de ejecución de Azure IoT Edge](../../iot-edge/iot-edge-runtime.md) es correcto y funciona bien.

![Captura de pantalla que muestra un valor de estado de un entorno de ejecución de IoT Edge.](./media/deploy-iot-edge-device/status.png)

#### <a name="invoke-a-direct-method"></a>Invocación de un método directo

Después, vamos a probar el ejemplo con la invocación de un método directo. Lea [Métodos directos de Azure Video Analyzer](direct-methods.md) para conocer todos los métodos directos que proporciona nuestro módulo avaEdge.

1. Al hacer clic en el módulo Edge que se ha creado, irá a su página de configuración.  

    ![Captura de pantalla que muestra la página de configuración de un módulo perimetral.](./media/deploy-iot-edge-device/modules.png)
1. Haga clic en la opción de menú Método directo.

    > [!NOTE] 
    > Tendrá que agregar un valor en las secciones Cadena de conexión, tal como se puede ver en la página actual. No es necesario ocultar ni cambiar nada en la sección **Nombre de configuración**. Es correcto dejar que sea pública.

    ![Método directo](./media/deploy-iot-edge-device/module-details.png)
1. Después, escriba "pipelineTopologyList" en el cuadro Nombre del método.
1. Luego, copie y pegue la carga de JSON siguiente en el cuadro de carga.
    
   ```
   {
       "@apiVersion": "1.0"
   }
   ```
1. Haga clic en la opción "Invocar método" situada en la parte superior de la página.

    ![Métodos directos](./media/deploy-iot-edge-device/direct-method.png)
1. Debería ver un mensaje de estado 200 en el cuadro de resultados.

    ![Mensaje de estado 200](./media/deploy-iot-edge-device/connection-timeout.png) 

## <a name="next-steps"></a>Pasos siguientes

Pruebe [Inicio rápido: Introducción: Azure Video Analyzer](get-started-detect-motion-emit-events.md)

> [!TIP]
> Si continúa con el inicio rápido anterior, al invocar los métodos directos mediante Visual Studio Code, utilizará el dispositivo que se agregó a IoT Hub a través de este artículo, en lugar del predeterminado `avasample-iot-edge-device`.
