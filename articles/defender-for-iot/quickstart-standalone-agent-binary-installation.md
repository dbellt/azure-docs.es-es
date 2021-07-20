---
title: 'Inicio rápido: Instalación del microagente de Defender para IoT (versión preliminar)'
description: En este inicio rápido, aprenderá a instalar y autenticar el microagente de Defender.
ms.date: 06/27/2021
ms.topic: quickstart
ms.openlocfilehash: 7f4215350af2e060bff4e1592eb76975e426ae2d
ms.sourcegitcommit: 3b371af4c30fce82854b3d45fd8b8e674bbe7517
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2021
ms.locfileid: "112984919"
---
# <a name="quickstart-install-defender-for-iot-micro-agent-preview"></a>Inicio rápido: Instalación del microagente de Defender para IoT (versión preliminar)

En este artículo se explica cómo instalar y autenticar el microagente de Defender.

## <a name="prerequisites"></a>Prerrequisitos

Antes de instalar el módulo Defender para IoT, debe crear una identidad de módulo en IoT Hub. Para más información sobre cómo crear una identidad de módulo, consulte [Creación de un módulo gemelo del microagente de Defender para IoT (versión preliminar)](quickstart-create-micro-agent-module-twin.md).

## <a name="install-the-package"></a>Instalar el paquete

**Para agregar el repositorio de paquetes de Microsoft adecuado**:

1. Descargue la configuración del repositorio que coincida con el sistema operativo del dispositivo.  

    - Para Ubuntu 18.04

        ```bash
        curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
        ```

    - Para Ubuntu 20.04

        ```bash
            curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list > ./microsoft-prod.list
        ```

    - Para Debian 9 (AMD64 y ARM64)

        ```bash
        curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
        ```

1. Copie la configuración del repositorio al directorio `sources.list.d`.

    ```bash
    sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
    ```

1. Actualice la lista de paquetes del repositorio que ha agregado con el siguiente comando:

    ```bash
    sudo apt-get update
    ```

Para instalar el paquete de microagente de Defender en Debian y las distribuciones de Linux basadas en Ubuntu, use el siguiente comando:

```bash
sudo apt-get install defender-iot-micro-agent 
```

## <a name="micro-agent-authentication-methods"></a>Métodos de autenticación del microagente 

Las dos opciones que se usan para autenticar el microagente de Defender para IoT son las siguientes: 

- Cadena de conexión de identidad de módulo. 

- Certificado.

### <a name="authenticate-using-a-module-identity-connection-string"></a>Autenticación mediante una cadena de conexión de identidad de módulo

Asegúrese de que se cumplen los [requisitos previos](#prerequisites) para este artículo y de que crea una identidad de módulo antes de iniciar estos pasos. 

#### <a name="get-the-module-identity-connection-string"></a>Obtención de la cadena de conexión de identidad de módulo

Para obtener la cadena de conexión de la identidad de módulo desde IoT Hub: 

1. Vaya a IoT Hub y seleccione el centro de conectividad.

1. En el menú de la izquierda, en la sección **Exploradores**, seleccione **Dispositivos IOT**.

   :::image type="content" source="media/quickstart-standalone-agent-binary-installation/iot-devices.png" alt-text="Seleccione dispositivos IoT en el menú izquierdo.":::

1. Seleccione un dispositivo en la lista Id. de dispositivo para ver la página **Detalles del dispositivo**.

1. Seleccione la pestaña  **Identidades de módulo** .

1. Seleccione el módulo  **DefenderIotMicroAgent**  de la lista de identidades de módulo asociadas al dispositivo.

   :::image type="content" source="media/quickstart-standalone-agent-binary-installation/module-identities.png" alt-text="Seleccione las pestaña de identidades de módulo.":::

1. En la página **Detalles de identidad de módulo**, copie la cadena de conexión (clave principal) seleccionando el botón **Copiar**.

   :::image type="content" source="media/quickstart-standalone-agent-binary-installation/copy-button.png" alt-text="Selección del botón de copia para copiar la cadena de conexión (clave principal).":::

#### <a name="configure-authentication-using-a-module-identity-connection-string"></a>Configuración de la autenticación mediante una cadena de conexión de identidad de módulo

Para configurar el agente para autenticación mediante una cadena de conexión de identidad de módulo:

1. Escriba el comando siguiente para colocar un archivo denominado `connection_string.txt` que contenga la cadena de conexión codificada en UTF-8 en la ruta de acceso `/var/defender_iot_micro_agent` del directorio del agente de Defender:

    ```bash
    sudo bash -c 'echo "<connection string>" > /var/defender_iot_micro_agent/connection_string.txt'
    ```

    `connection_string.txt` debe encontrarse en la siguiente ubicación de ruta de acceso `/var/defender_iot_micro_agent/connection_string.txt`.

1. Reinicie el servicio con este comando:  

    ```bash
    sudo systemctl restart defender-iot-micro-agent.service 
    ```

### <a name="authenticate-using-a-certificate"></a>Autenticación mediante un certificado

Para la autenticación mediante un certificado:

1. Siga [estas instrucciones](../iot-hub/tutorial-x509-scripts.md) para conseguir un certificado.

1. Coloque la parte pública con codificación PEM del certificado y la clave privada en el directorio del agente de Defender del archivo denominado `certificate_public.pem` y `certificate_private.pem`. 

1. Copie la cadena de conexión adecuada en el archivo `connection_string.txt`. La cadena de conexión debe tener este aspecto: 

    `HostName=<the host name of the iot hub>;DeviceId=<the id of the device>;ModuleId=<the id of the module>;x509=true` 

    Esta cadena indica al agente de Defender que espere a que se proporcione un certificado para la autenticación. 

1. Reinicie el servicio mediante el comando siguiente:  

    ```bash
    sudo systemctl restart defender-iot-micro-agent.service
    ```

### <a name="validate-your-installation"></a>Validación de la instalación

Para la validación de la instalación:

1. Asegurándose de que el microagente se ejecuta correctamente con el siguiente comando:  

    ```bash
    systemctl status defender-iot-micro-agent.service
    ```

1. Asegúrese de que el servicio es estable. Para ello, debe asegurarse de que está `active` y de que el tiempo de actividad del proceso es el adecuado.

    :::image type="content" source="media/quickstart-standalone-agent-binary-installation/active-running.png" alt-text="Compruebe que el servicio es estable y está activo.":::
 
## <a name="testing-the-system-end-to-end"></a>Prueba integral del sistema 

Puede probar el sistema de manera integral mediante la creación de un archivo de desencadenador en el dispositivo. El archivo de desencadenador hará que el examen de línea de base del agente detecte el archivo como infracción de línea base. 

Cree un archivo en el sistema de archivos con el comando siguiente:

```bash
sudo touch /tmp/DefenderForIoTOSBaselineTrigger.txt 
```

Se producirá una recomendación de error de validación de línea de base en el centro de conectividad, con un `CceId` de CIS-Debian-9-DEFENDER_FOR_IOT_TEST_CHECKS-0.0: 

:::image type="content" source="media/quickstart-standalone-agent-binary-installation/validation-failure.png" alt-text="Recomendación de error de validación de línea de base que se produce en el centro de conectividad." lightbox="media/quickstart-standalone-agent-binary-installation/validation-failure-expanded.png":::

Espere una hora como máximo para que la recomendación aparezca en el centro de conectividad. 

## <a name="micro-agent-versioning"></a>Control de versiones del microagente 

Para instalar una versión específica del microagente de Defender para IoT, ejecute el siguiente comando: 

```bash
sudo apt-get install defender-iot-micro-agent=<version>
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación del microagente de Defender a partir de código fuente](quickstart-building-the-defender-micro-agent-from-source.md)