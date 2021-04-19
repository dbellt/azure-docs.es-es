---
title: 'Actualización de la versión de IoT Edge en los dispositivos: Azure IoT Edge | Microsoft Docs'
description: Cómo actualizar los dispositivos IoT Edge para ejecutar las versiones más recientes del demonio de seguridad y el entorno de ejecución de IoT Edge
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 04/07/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: e5034c228a354c98b5792492d484da9eb10b8cf2
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107310859"
---
# <a name="update-the-iot-edge-security-daemon-and-runtime"></a>Actualice el archivo de configuración del demonio de seguridad y el entorno de ejecución de IoT Edge.

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Dado que el servicio de IoT Edge lanza versiones nuevas, querrá actualizar los dispositivos IoT Edge para tener las últimas características y mejoras de seguridad. En este artículo se proporciona información sobre cómo actualizar los dispositivos IoT Edge cuando hay una versión nueva disponible.

Es necesario actualizar dos componentes de un dispositivo IoT Edge si quiere pasar a una versión más reciente. El primero es el demonio de seguridad, que se ejecuta en el dispositivo e inicia los módulos en tiempo de ejecución cuando se inicia el dispositivo. Actualmente, el demonio de seguridad solo puede actualizarse desde el propio dispositivo. El segundo componente es el entorno de ejecución, formado por los módulos Centro de IoT Edge y Agente de IoT Edge. En función del modo de estructurar la implementación, el entorno de ejecución puede actualizarse desde el dispositivo o de forma remota.

Para obtener la versión más reciente de Azure IoT Edge, consulte [Versiones de Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

## <a name="update-the-security-daemon"></a>Actualización del demonio de seguridad

El demonio de seguridad de IoT Edge es un componente nativo que debe actualizarse mediante el administrador de paquetes en el dispositivo IoT Edge.

Compruebe la versión del demonio de seguridad que se ejecuta en el dispositivo mediante el comando `iotedge version`.

>[!IMPORTANT]
>Si va a actualizar un dispositivo de la versión 1.0 o 1.1 a la versión 1.2, existen diferencias en los procesos de instalación y configuración que requieren pasos adicionales. Para más información, consulte los pasos que se indican más adelante en este artículo: [Caso especial: actualización de 1.0 o 1.1 a 1.2](#special-case-update-from-10-or-11-to-12).

# <a name="linux"></a>[Linux](#tab/linux)

En los dispositivos Linux x64, use apt-get o el administrador de paquetes adecuado para actualizar el demonio de seguridad a la versión más reciente.

Obtenga la configuración del repositorio más reciente de Microsoft:

* **Ubuntu Server 18.04**:

   ```bash
   curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
   ```

* **Raspberry Pi OS Stretch**:

   ```bash
   curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
   ```

Copie la lista generada.

   ```bash
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   ```

Instale de la clave pública de GPG de Microsoft.

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

Actualice apt.

   ```bash
   sudo apt-get update
   ```

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

Compruebe qué versiones de IoT Edge están disponibles.

   ```bash
   apt list -a iotedge
   ```

Si desea actualizar a la versión más reciente del demonio de seguridad, use el siguiente comando, que también actualiza **libiothsm-std** a la versión más reciente:

   ```bash
   sudo apt-get install iotedge
   ```

Si desea actualizar el demonio de seguridad a una versión específica, especifique la versión en la salida de la lista de apt. Cada vez que **iotedge** se actualiza, se intenta actualizar automáticamente el paquete **libiothsm-std** a su versión más reciente, lo que puede provocar un conflicto de dependencia. Si no va a la versión más reciente, asegúrese de tener como destino ambos paquetes para la misma versión. Por ejemplo, el siguiente comando instala una versión específica de la versión 1.1:

   ```bash
   sudo apt-get install iotedge=1.1.1 libiothsm-std=1.1.1
   ```

Si la versión que desea instalar no está disponible a través de apt-get, puede usar curl para tener como destino cualquier versión del repositorio de [versiones de IoT Edge](https://github.com/Azure/azure-iotedge/releases). Para cualquier versión que desee instalar, localice los archivos **libiothsm-std** y **iotedge** adecuados para el dispositivo. En cada archivo, haga clic con el botón derecho sobre el vínculo del archivo y copie la dirección del vínculo. Use la dirección del vínculo para instalar las versiones específicas de esos componentes:

```bash
curl -L <libiothsm-std link> -o libiothsm-std.deb && sudo apt-get install ./libiothsm-std.deb
curl -L <iotedge link> -o iotedge.deb && sudo apt-get install ./iotedge.deb
```
<!-- end 1.1 -->
:::moniker-end

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

Compruebe qué versiones de IoT Edge están disponibles.

   ```bash
   apt list -a aziot-edge
   ```

Si quiere actualizar a la versión más reciente de IoT Edge, use el siguiente comando, que también actualiza el servicio de identidad a la versión más reciente:

   ```bash
   sudo apt-get install aziot-edge
   ```
<!-- end 1.2 -->
:::moniker-end

# <a name="windows"></a>[Windows](#tab/windows)

<!-- 1.1 -->
::: moniker range="iotedge-2018-06"

Con IoT Edge para Linux en Windows, IoT Edge se ejecuta en una máquina virtual Linux hospedada en un dispositivo Windows. Esta máquina virtual está instalada previamente con IoT Edge, y se administra con Microsoft Update para mantener actualizados los componentes. Si tiene actualizaciones automáticas habilitadas, las nuevas actualizaciones se descargarán e instalarán siempre que estén disponibles.

Con IoT Edge para Windows, IoT Edge se ejecuta directamente en el dispositivo Windows. Para obtener instrucciones para la actualización con los scripts de PowerShell, consulte [Instalación y administración de Azure IoT Edge para Windows](how-to-install-iot-edge-windows-on-windows.md).
:::moniker-end

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

Actualmente, no hay soporte técnico para IoT Edge versión 1.2 en ejecución en dispositivos Windows.

:::moniker-end

---

## <a name="update-the-runtime-containers"></a>Actualización de los contenedores del entorno de ejecución

El modo de actualizar los contenedores del centro de IoT Edge y del agente de IoT Edge depende de si usa etiquetas graduales (por ejemplo, 1.1) o etiquetas específicas (por ejemplo, 1.1.1) en la implementación.

Compruebe la versión de los módulos Agente de IoT Edge y Centro de IoT Edge instalada actualmente en el dispositivo mediante los comandos `iotedge logs edgeAgent` o `iotedge logs edgeHub`.

  ![Búsqueda de la versión del contenedor en los registros](./media/how-to-update-iot-edge/container-version.png)

### <a name="understand-iot-edge-tags"></a>Información sobre las etiquetas de IoT Edge

Las imágenes del Agente de IoT Edge y del centro de IoT Edge se etiquetan con la versión de IoT Edge a la que están asociadas. Hay dos maneras diferentes de usar etiquetas con las imágenes del entorno de ejecución:

* **Etiquetas graduales**: use solo los dos primeros valores del número de versión para obtener la imagen más reciente que coincida con esos dígitos. Por ejemplo, 1.1 se actualiza cada vez que hay una nueva versión para que apunte a la versión 1.1.x más reciente. Si el entorno de ejecución del contenedor en el dispositivo IoT Edge extrae la imagen de nuevo, se actualizan los módulos del entorno de ejecución a la versión más reciente. Las implementaciones de Azure Portal tienen como valor predeterminado las etiquetas graduales. *Este enfoque se recomienda para fines de desarrollo.*

* **Etiquetas específicas**: use los tres valores del número de versión para establecer explícitamente la versión de la imagen. Por ejemplo, la versión 1.1.0 no cambiará después de su lanzamiento inicial. Puede declarar un nuevo número de versión del manifiesto de implementación cuando esté listo para actualizar. *Este enfoque se recomienda para fines de producción.*

### <a name="update-a-rolling-tag-image"></a>Actualización de una imagen de etiqueta gradual

Si usa etiquetas graduales en la implementación (por ejemplo, mcr.microsoft.com/azureiotedge-hub:**1.1**), deberá forzar el entorno de ejecución del contenedor en el dispositivo para extraer la versión más reciente de la imagen.

Elimine la versión local de la imagen del dispositivo IoT Edge. En equipos con Windows, al desinstalar el demonio de seguridad también se quitan las imágenes en tiempo de ejecución, por lo que no es necesario volver a realizar este paso.

```bash
docker rmi mcr.microsoft.com/azureiotedge-hub:1.1
docker rmi mcr.microsoft.com/azureiotedge-agent:1.1
```

Es posible que tenga que usar la marca `-f` de fuerza para eliminar las imágenes.

El servicio de IoT Edge extraerá las versiones más recientes de las imágenes del entorno de ejecución y volverá a iniciarlas automáticamente en el dispositivo.

### <a name="update-a-specific-tag-image"></a>Actualización de una etiqueta específica

Si usa etiquetas específicas en la implementación (por ejemplo, mcr.microsoft.com/azureiotedge-hub:**1.1.1**), solo tiene que actualizar la etiqueta en el manifiesto de implementación y aplicar los cambios en el dispositivo.

1. En IoT Hub en Azure Portal, seleccione el dispositivo IoT Edge y luego **Establecer módulos**.

1. En la sección **Módulos de IoT Edge**, seleccione **Configuración del entorno de ejecución**.

   ![Configuración de los parámetros de ejecución](./media/how-to-update-iot-edge/configure-runtime.png)

1. En **Configuración del entorno de ejecución**, actualice el valor **Image** del **Centro de Microsoft Edge** con la versión deseada. No seleccione **Guardar** todavía.

   ![Actualización de la versión de la imagen del Centro de Microsoft Edge](./media/how-to-update-iot-edge/runtime-settings-edgehub.png)

1. Contraiga la configuración del **Centro de Microsoft Edge**, o desplácese hacia abajo, y actualice el valor de **Image** para el **Agente de Edge** con la misma versión deseada.

   ![Actualización de la versión del agente del Centro de Microsoft Edge](./media/how-to-update-iot-edge/runtime-settings-edgeagent.png)

1. Seleccione **Guardar**.

1. Seleccione **Revisar y crear**, revise la implementación y, después, seleccione **Crear**.

## <a name="special-case-update-from-10-or-11-to-12"></a>Caso especial: actualización de 1.0 o 1.1 a 1.2

A partir de la versión 1.2, el servicio IoT Edge usa un nombre de paquete nuevo y tiene algunas diferencias en los procesos de instalación y configuración. Si dispone de un dispositivo IoT Edge que ejecuta la versión 1.0 o 1.1, siga estas instrucciones para obtener información sobre cómo actualizar a 1.2.

>[!NOTE]
>Actualmente, no hay soporte técnico para IoT Edge versión 1.2 en ejecución en dispositivos Windows.

Algunas de las diferencias principales entre la versión 1.2 y las anteriores son:

* El nombre del paquete ha cambiado de **iotedge** a **aziot-edge**.
* El paquete **libiothsm-std** ya no se usa. Si usó el paquete estándar proporcionado como parte de la versión de IoT Edge, las configuraciones se pueden transferir a la nueva versión. Si usó una implementación diferente de libiothsm-std, deberá volver a configurar los certificados proporcionados por el usuario como el certificado de identidad de dispositivo, la CA de dispositivo y el paquete de confianza.
* Un nuevo servicio de identidad, **aziot-identity-service**, se introdujo como parte de la versión 1.2. Este servicio controla el aprovisionamiento y la administración de identidades para IoT Edge y otros componentes de dispositivo que necesitan comunicarse con IoT Hub, como la [actualización de dispositivos para IoT Hub](../iot-hub-device-update/understand-device-update.md).
* El archivo de configuración predeterminado tiene un nombre y una ubicación nuevos. La información de configuración de dispositivo que anteriormente se encontraba en `/etc/iotedge/config.yaml`, ahora se espera que esté en `/etc/aziot/config.toml` de manera predeterminada. El comando `iotedge config import` se puede usar para ayudar a migrar la información de configuración de la ubicación y la sintaxis anteriores a las nuevas.
  * El comando de importación no puede detectar ni modificar las reglas de acceso al módulo de plataforma segura (TPM) de un dispositivo. Si el dispositivo usa la atestación de TPM, debe actualizar manualmente el archivo /etc/udev/rules.d/tpmaccess.rules para proporcionar acceso al servicio aziottpm. Para más información, vea [Concesión a IoT Edge el acceso a TPM](how-to-auto-provision-simulated-device-linux.md?view=iotedge-2020-11&preserve-view=true#give-iot-edge-access-to-the-tpm).
* La API de carga de trabajo de la versión 1.2 guarda los secretos cifrados en un nuevo formato. Si actualiza desde una versión anterior a la versión 1.2, se importa la clave de cifrado maestra existente. La API de carga de trabajo puede leer los secretos guardados en el formato anterior mediante la clave de cifrado importada. Sin embargo, la API de carga de trabajo no puede escribir secretos cifrados en el formato anterior. Una vez que un módulo vuelve a cifrar un secreto, se guarda en el nuevo formato. El mismo módulo de la versión 1.1 no puede leer los secretos cifrados en la versión 1.2. Si conserva los datos cifrados en una carpeta o volumen montados en host, cree siempre una copia de seguridad de los datos *antes* de la actualización para conservar la capacidad de cambiar a una versión anterior, si es necesario.

Antes de automatizar los procesos de actualización, compruebe que funciona en máquinas de prueba.

Cuando esté listo, siga estos pasos para actualizar IoT Edge en los dispositivos:

1. Obtenga la configuración del repositorio más reciente de Microsoft:

   * **Ubuntu Server 18.04**:

     ```bash
     curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
     ```

   * **Raspberry Pi OS Stretch**:

     ```bash
     curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
     ```

2. Copie la lista generada.

   ```bash
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   ```

3. Instale de la clave pública de GPG de Microsoft.

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

4. Actualice apt.

   ```bash
   sudo apt-get update
   ```

5. Desinstale la versión anterior de IoT Edge, manteniendo los archivos de configuración en su lugar.

   ```bash
   sudo apt-get remove iotedge
   ```

6. Instale la versión más reciente de IoT Edge, junto con el servicio de identidad de IoT.

   ```bash
   sudo apt-get install aziot-edge
   ```

7. Importe el archivo config.yaml antiguo a su nuevo formato y aplique la información de configuración.

   ```bash
   sudo iotedge config import
   ```

Ahora que se ha actualizado el servicio IoT Edge que se ejecuta en los dispositivos, siga los pasos de este artículo para [Actualizar los contenedores en tiempo de ejecución](#update-the-runtime-containers) también.

## <a name="special-case-update-to-a-release-candidate-version"></a>Caso especial: actualización a una versión candidata para lanzamiento

Azure IoT Edge lanza periódicamente nuevas versiones del servicio IoT Edge. Antes de cada versión estable, se publican una o varias versiones candidatas para lanzamiento (RC). Las versiones RC incluyen todas las características planeadas para la versión, pero aún se están sometiendo a pruebas y validación. Si quiere probar una nueva característica de manera anticipada, puede instalar una versión RC y dejar comentarios a través de GitHub.

Las versiones candidatas para lanzamiento siguen la misma convención de numeración que las versiones, pero tienen **-rc** más un número incremental anexados al final. Puede ver las versiones candidatas para lanzamiento en la misma lista de [versiones de Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases) que las versiones estables. Por ejemplo, busque **1.2.0-rc4**, una de las versiones candidatas para lanzamiento publicada antes de **1.2.0**. También puede ver que las versiones RC se marcan con etiquetas de **versión preliminar**.

Los módulos agente y centro de IoT Edge tienen versiones RC que se etiquetan siguiendo la misma convención. Por ejemplo, **mcr.microsoft.com/azureiotedge-hub:1.2.0-rc4**.

Igual que las versiones preliminares, las versiones candidatas para lanzamiento no se incluyen como la versión más reciente que utilizan los instaladores habituales. En su lugar, debe especificar manualmente los recursos para la versión RC que quiere probar. Por lo general, la instalación o actualización de una versión RC equivale a seleccionar cualquier otra versión específica de IoT Edge.

Use las secciones de este artículo para obtener información sobre cómo actualizar un dispositivo IoT Edge a una versión específica del demonio de seguridad o de los módulos del entorno de ejecución.

Si va a instalar IoT Edge, en lugar de actualizar una instalación existente, use los pasos de la [instalación sin conexión o específica de la versión](how-to-install-iot-edge.md#offline-or-specific-version-installation-optional).

## <a name="next-steps"></a>Pasos siguientes

Consulte las últimas [versiones de Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

Permanezca actualizado con los anuncios y las actualizaciones recientes del [blog de Internet de las cosas](https://azure.microsoft.com/blog/topics/internet-of-things/).
