---
title: 'Aprovisionamiento automático de dispositivos Windows con DPS: Azure IoT Edge | Microsoft Docs'
description: Use un dispositivo simulado en el equipo Windows para probar el aprovisionamiento automático de dispositivos para Azure IoT Edge con Device Provisioning Service
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 4/3/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 82bd027773a5759caee19228f56ba4b3dfe8c2cf
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107482014"
---
# <a name="create-and-provision-a-simulated-iot-edge-device-with-a-virtual-tpm-on-windows"></a>Creación y aprovisionamiento de un dispositivo IoT Edge con un TPM virtual en Windows

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

Los dispositivos Azure IoT Edge pueden aprovisionarse automáticamente con [Device Provisioning Service](../iot-dps/index.yml), igual que los dispositivos que no están habilitados para Edge. Si no está familiarizado con el proceso de aprovisionamiento automático, revise la información general sobre el [aprovisionamiento](../iot-dps/about-iot-dps.md#provisioning-process) antes de continuar.

DPS ofrece soporte técnico para la atestación de clave simétrica para dispositivos IoT Edge en inscripciones de grupo e individuales. Para la inscripción de grupo, si marca la opción "Dispositivo IoT Edge" como verdadera en la atestación de clave simétrica, todos los dispositivos registrados en ese grupo de inscripción se marcarán como dispositivos IoT Edge.

En este artículo se muestra cómo probar el aprovisionamiento automático en un dispositivo IoT Edge simulado con los pasos siguientes:

* Cree una instancia de IoT Hub Device Provisioning Service (DPS).
* Cree un dispositivo simulado en una máquina Windows con un Módulo de plataforma segura (TPM) simulado para la seguridad de hardware.
* Cree una inscripción individual para el dispositivo.
* Instale el entorno de ejecución de IoT Edge y conecte el dispositivo a IoT Hub.

> [!TIP]
> En este artículo se describe la comprobación del aprovisionamiento automático mediante el uso de la atestación de TPM en los dispositivos virtuales, pero gran parte se aplica también al hardware físico de TPM.

## <a name="prerequisites"></a>Prerrequisitos

* Una máquina de desarrollo Windows. En este artículo se usa Windows 10.
* Una instancia de IoT Hub activa.

> [!NOTE]
> Se necesita la versión TPM 2.0 al usar la atestación de TPM con DPS y solo se puede usar para crear inscripciones individuales, no de grupo.

## <a name="set-up-the-iot-hub-device-provisioning-service"></a>Configuración de IoT Hub Device Provisioning Service

Cree una nueva instancia de IoT Hub Device Provisioning Service en Azure y vincúlela a IoT Hub. Puede seguir las instrucciones de [Configuración de IoT Hub DPS](../iot-dps/quick-setup-auto-provision.md).

Cuando Device Provisioning Service esté en ejecución, copie el valor de **Ámbito de id.** de la página de información general. Use este valor cuando configure el entorno de ejecución de IoT Edge.

> [!TIP]
> Si usa un dispositivo de TPM físico, debe determinar la **Clave de aprobación**, que es única para cada chip de TPM y se obtiene del fabricante del chip de TPM asociado. Puede derivar un único **identificador de registro** para el dispositivo TPM, por ejemplo, al aplicar un algoritmo hash SHA-256 a la clave de aprobación.
>
> Siga las instrucciones del artículo [Administración de inscripciones de dispositivos con Azure Portal](../iot-dps/how-to-manage-enrollments.md) para crear su inscripción en DPS y prosiga con la sección [Instalación del entorno de ejecución de IoT Edge](#install-the-iot-edge-runtime) de este artículo para continuar.

## <a name="simulate-a-tpm-device"></a>Simulación del dispositivo de TPM

Cree un dispositivo TPM simulado en la máquina de desarrollo Windows. Recupere el **Id. de registro** y la **Clave de aprobación** para el dispositivo y úselos para crear una entrada de inscripción individual en DPS.

Al crear una inscripción en DPS, tiene la oportunidad de declarar un **Estado inicial de dispositivo gemelo**. En el dispositivo gemelo, puede establecer etiquetas para agrupar dispositivos por cualquier métrica que necesite en su solución, como la región, el entorno, la ubicación o el tipo de dispositivo. Estas etiquetas se usan para crear [implementaciones automáticas](how-to-deploy-at-scale.md).

Elija el lenguaje del SDK que desea usar para crear el dispositivo simulado y siga los pasos hasta que se cree la inscripción individual.

Cuando cree la inscripción individual, seleccione **True** (Verdadero) para declarar que el dispositivo TPM simulado en la máquina de desarrollo de Windows es un **dispositivo IoT Edge**.

> [!TIP]
> En la CLI de Azure, puede crear una [inscripción](/cli/azure/iot/dps/enrollment) o un [grupo de inscripción](/cli/azure/iot/dps/enrollment-group) y usar la marca **habilitado para Edge** para especificar que un dispositivo o un grupo de dispositivos son un dispositivo IoT Edge.

Guías de dispositivos simulados e inscripción individual:

* [C](../iot-dps/quick-create-simulated-device.md)
* [Java](../iot-dps/quick-create-simulated-device-tpm-java.md)
* [C#](../iot-dps/quick-create-simulated-device-tpm-csharp.md)
* [Node.js](../iot-dps/quick-create-simulated-device-tpm-node.md)
* [Python](../iot-dps/quick-create-simulated-device-tpm-python.md)

Después de crear la inscripción individual, guarde el valor del **Id. de registro**. Use este valor cuando configure el entorno de ejecución de IoT Edge.

## <a name="install-the-iot-edge-runtime"></a>Instalación del entorno de ejecución de IoT Edge

El runtime de IoT Edge se implementa en todos los dispositivos de IoT Edge. Sus componentes se ejecutan en contenedores y permiten implementar contenedores adicionales en el dispositivo para que pueda ejecutar código en Edge. Instale el entorno de ejecución de IoT Edge en el dispositivo que ejecuta el TPM simulado.

Siga los pasos descritos en [Instalación del entorno de ejecución de Azure IoT Edge](how-to-install-iot-edge.md) y, luego, vuelva a este artículo para aprovisionar el dispositivo.

> [!TIP]
> Mantenga abierta la ventana que se está ejecutando el simulador de TPM durante la instalación y prueba.

## <a name="configure-the-device-with-provisioning-information"></a>Configuración del dispositivo con la información de aprovisionamiento

Una vez que el entorno de ejecución está instalado en el dispositivo, configure el dispositivo con la información que usa para conectarse al servicio Device Provisioning y a IoT Hub.

1. Conozca el **Ámbito de identificador** de DPS y el **Identificador de registro** del dispositivo que se recopilaron en las secciones anteriores.

1. Abra una ventana de Azure PowerShell en modo de administrador. Asegúrese de usar una sesión de AMD64 de PowerShell para instalar IoT Edge, no PowerShell (x86).

1. El comando **Deploy-IoTEdge** comprueba si la versión del equipo Windows es compatible, activa la característica de contenedores y descarga tanto el runtime de Moby como el de IoT Edge. De forma predeterminada el comando usa contenedores Windows.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Deploy-IoTEdge
   ```

1. En este momento, los dispositivos IoT Core pueden reiniciarse automáticamente. Es posible que los dispositivos Windows 10 o Windows Server soliciten su reinicio. En ese caso, reinícielo ahora. Una vez que el dispositivo esté listo, vuelva a ejecutar PowerShell como administrador.

1. El comando **Initialize-IoTEdge** configura el entorno de ejecución de Azure IoT Edge en el equipo. El comportamiento predeterminado del comando es el aprovisionamiento manual con contenedores de Windows. Utilice la marca `-Dps` para usar Device Provisioning Service, en lugar del aprovisionamiento manual.

   Reemplace los valores de marcador de posición para `{scope_id}` y `{registration_id}` con los datos que ha recopilado antes.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -Dps -ScopeId {scope ID} -RegistrationId {registration ID}
   ```

## <a name="verify-successful-installation"></a>Comprobación de instalación correcta

Si el entorno de ejecución se inició correctamente, puede ir a IoT Hub y empezar a implementar módulos de IoT Edge en el dispositivo. Use los siguientes comandos en el dispositivo para comprobar que el entorno de ejecución está instalado e iniciado correctamente.  

Compruebe el estado del servicio IoT Edge.

```powershell
Get-Service iotedge
```

Examine los registros de servicio de los últimos 5 minutos.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

Enumere los módulos en ejecución.

```powershell
iotedge list
```

## <a name="next-steps"></a>Pasos siguientes

El proceso de inscripción en Device Provisioning Service permite establecer el id. de dispositivo y las etiquetas del dispositivo gemelo al mismo tiempo que aprovisiona el nuevo dispositivo. Puede usar esos valores para dirigirse a dispositivos individuales o grupos de dispositivos con la administración automática de dispositivos. Obtenga información sobre la [Implementación y supervisión de módulos de IoT Edge a escala mediante Azure Portal](how-to-deploy-at-scale.md) o [mediante la CLI de Azure](how-to-deploy-cli-at-scale.md)
