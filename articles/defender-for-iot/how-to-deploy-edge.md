---
title: Implementación del módulo de seguridad de IoT Edge
description: Aprenda cómo implementar un agente de seguridad de Defender para IoT en IoT Edge.
ms.topic: conceptual
ms.date: 04/21/2021
ms.openlocfilehash: 71efb0bb12d1e20f918481a086fd411d3a237e33
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107813603"
---
# <a name="deploy-a-security-module-on-your-iot-edge-device"></a>Implementación de un módulo de seguridad en el dispositivo IoT Edge

El módulo **Defender para IoT** proporciona una solución de seguridad completa para los dispositivos IoT Edge.
El módulo de seguridad recopila, agrega y analiza los datos de seguridad sin procesar del sistema operativo y del sistema de contenedores para generar alertas y recomendaciones de seguridad que requieren acción.
Para aprender más, consulte [Módulo de seguridad de Azure Security Center for IoT](security-edge-architecture.md).

En este artículo, aprenderá a implementar un módulo de seguridad en el dispositivo IoT Edge.

## <a name="deploy-security-module"></a>Implementación de un módulo de seguridad

Siga estos pasos para implementar un módulo de seguridad de Defender para IoT para IoT Edge.

### <a name="prerequisites"></a>Prerrequisitos

1. En su IoT Hub, asegúrese de que el dispositivo está [registrado como un dispositivo IoT Edge](../iot-edge/how-to-register-device.md#register-a-new-device).

1. El módulo Defender para IoT Edge requiere que el [marco AuditD](https://linux.die.net/man/8/auditd) esté instalado en el dispositivo IoT Edge.

    - Instale el marco de trabajo ejecutando el comando siguiente en el dispositivo IoT Edge:

    `sudo apt-get install auditd audispd-plugins`

    - Compruebe que AuditID está activo mediante la ejecución del comando siguiente:

    `sudo systemctl status auditd`<br>
    - La respuesta esperada es: `active (running)`

### <a name="deployment-using-azure-portal"></a>Implementación mediante Azure Portal

1. Desde Azure Portal, abra **Marketplace**.

1. Seleccione **Internet de las cosas** y luego busque **Azure Security Center para IoT** y selecciónelo.

   :::image type="content" source="media/howto/edge-onboarding-8.png" alt-text="Selección de Defender para IoT":::

1. Seleccione **Crear** para configurar la implementación.

1. Elija la **suscripción** de Azure de IoT Hub y luego seleccione su **IoT Hub**.<br>Seleccione **Implementar en un dispositivo** para un único dispositivo de destino o **Implementar a escala** para varios dispositivos de destino y, a continuación, seleccione **Crear**. Para más información sobre la implementación a escala, consulte [Implementación y supervisión de módulos de IoT Edge a escala mediante Azure Portal](../iot-edge/how-to-deploy-at-scale.md).

    >[!Note]
    >Si seleccionó **Implementar a escala**, agregue el nombre del dispositivo y los detalles antes de continuar con la pestaña **Agregar módulos** en las instrucciones siguientes.

Complete cada uno de los pasos para completar la implementación de IoT Edge de Defender para IoT.

#### <a name="step-1-modules"></a>Paso 1: Módulos

1. Seleccione el módulo **AzureSecurityCenterforIoT**.
1. En la pestaña **Configuración del módulo**, cambie el **nombre** a **azureiotsecurity**.
1. En la pestaña **Variables de entorno**, agregue una variable si es necesario (por ejemplo, puede agregar la variable *debug level* y establecerla en uno de los valores siguientes: "Fatal", "Error", "Warning" o "Information").
1. En la pestaña **Opciones de creación del contenedor**, agregue la configuración siguiente:

    ``` json
    {
        "NetworkingConfig": {
            "EndpointsConfig": {
                "host": {}
            }
        },
        "HostConfig": {
            "Privileged": true,
            "NetworkMode": "host",
            "PidMode": "host",
            "Binds": [
                "/:/host"
            ]
        }
    }
    ```

1. En la pestaña **Configuración de módulos gemelos**, agregue la configuración siguiente:

   Propiedad del módulo gemelo:
   
   ``` json
     "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration"
   ```

   Contenido de propiedad del módulo gemelo: 

   ```json
     {

     }
   ```
    
   Para obtener más información sobre cómo configurar el agente, vea [Configuración de agentes de seguridad](./how-to-agent-configuration.md).

1. Seleccione **Actualizar**.

#### <a name="step-2-runtime-settings"></a>Paso 2: Configuración del entorno de ejecución

1. Seleccione **Configuración del entorno de ejecución**.
2. En **Centro de Microsoft Edge**, cambie el valor de **Imagen** a **mcr.microsoft.com/azureiotedge-hub:1.0.8.3**.

    >[!Note]
    > Actualmente, se admite la versión 1.0.8.3 o posterior.

3. Compruebe que la opción **Opciones de creación** está establecida tal y como se muestra a continuación:

    ``` json
    {
       "HostConfig":{
          "PortBindings":{
             "8883/tcp":[
                {
                   "HostPort":"8883"
                }
             ],
             "443/tcp":[
                {
                   "HostPort":"443"
                }
             ],
             "5671/tcp":[
                {
                   "HostPort":"5671"
                }
             ]
          }
       }
    }
    ```

4. Seleccione **Guardar**.

5. Seleccione **Next** (Siguiente).

#### <a name="step-3-specify-routes"></a>Paso 3: Especificación de rutas

1. En la pestaña **Especificar rutas**, asegúrese de tener una ruta (explícita o implícita) que reenvíe los mensajes del módulo **azureiotsecurity** a **$upstream** de acuerdo con los ejemplos siguientes. Solo cuando la ruta esté establecida, seleccione **Siguiente**.

   Rutas de ejemplo:

    ```Default implicit route
    "route": "FROM /messages/* INTO $upstream"
    ```

    ```Explicit route
    "ASCForIoTRoute": "FROM /messages/modules/azureiotsecurity/* INTO $upstream"
    ```

1. Seleccione **Next** (Siguiente).

#### <a name="step-4-review-deployment"></a>Paso 4: Revisión de la implementación

- En la pestaña **Revisar la implementación**, revise la información de implementación y seleccione **Crear** para completar la implementación.

## <a name="diagnostic-steps"></a>Pasos de diagnósticos

Si encuentra algún problema, los registros de contenedor son la mejor manera conocer el estado de un dispositivo de módulo de seguridad IoT Edge. Use los comandos y las herramientas de esta sección para recopilar información.

### <a name="verify-the-required-containers-are-installed-and-functioning-as-expected"></a>Comprobación de que los contenedores necesarios están instalados y funcionan según lo previsto

1. Ejecute el comando siguiente en el dispositivo de IoT Edge:

    `sudo docker ps`

1. Compruebe que los siguientes contenedores se están ejecutando:

   | Nombre | IMAGEN |
   | --- | --- |
   | azureiotsecurity | mcr.microsoft.com/ascforiot/azureiotsecurity:1.0.2 |
   | edgeHub | mcr.microsoft.com/azureiotedge-hub:1.0.8.3 |
   | edgeAgent | mcr.microsoft.com/azureiotedge-agent:1.0.1 |

   Si no están presentes los contenedores mínimos necesarios, compruebe si el manifiesto de implementación de IoT Edge está alineado con la configuración recomendada. Para más información, consulte [Implementación del módulo IoT Edge](#deployment-using-azure-portal).

### <a name="inspect-the-module-logs-for-errors"></a>Inspección de los registros de módulo en busca de errores

1. Ejecute el comando siguiente en el dispositivo de IoT Edge:

   `sudo docker logs azureiotsecurity`

1. Para los registros más detallados, agregue la siguiente variable de entorno a la implementación del módulo **azureiotsecurity**: `logLevel=Debug`.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las opciones de configuración, prosiga con la guía paso a paso para configurar el módulo.
> [!div class="nextstepaction"]
> [Guía paso a paso de configuración del módulo](./how-to-agent-configuration.md)