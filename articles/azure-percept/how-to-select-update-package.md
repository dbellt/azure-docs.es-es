---
title: Selección del paquete de actualización más adecuado para Azure Percept DK
description: Obtenga información sobre cómo identificar la versión de Azure Percept DK y seleccionar el paquete de actualización más adecuado.
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 05/04/2021
ms.custom: template-how-to
ms.openlocfilehash: 7528e11e24485fecc3e83773d98e119d98ca63cf
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111968756"
---
# <a name="how-to-determine-and-download-the-best-update-package-for-ota-and-usb-updates"></a>Determinación y descarga del paquete de actualización más adecuado para las actualizaciones mediante OTA y USB

En esta página se proporcionan instrucciones sobre cómo seleccionar el paquete de actualización que mejor se adapte a su kit de desarrollo y las ubicaciones de descarga de los paquetes de actualización.

Para más información sobre cómo actualizar el dispositivo, consulte estos artículos:
- [Actualización de Azure Percept DK de forma inalámbrica](./how-to-update-over-the-air.md)
- [Actualización de Azure Percept DK mediante USB](./how-to-update-via-usb.md)


## <a name="prerequisites"></a>Requisitos previos

- Un kit de desarrollo [Azure Percept DK](https://go.microsoft.com/fwlink/?linkid=2155270) que se haya [configurado y conectado a Azure Percept Studio e IoT Hub](./quickstart-percept-dk-set-up.md).

## <a name="identify-the-model-name-and-software-version-of-your-dev-kit"></a>Identificación del nombre del modelo y la versión de software del kit de desarrollo
Para asegurarse de aplicar el paquete de actualización correcto al kit de desarrollo, primero debe determinar qué versión de software se está ejecutando actualmente.

> [!WARNING]
> La aplicación de un paquete de actualización incorrecto podría dar lugar a que el kit de desarrollo se vuelva inoperable. Es importante que siga estos pasos para asegurarse de que aplica el paquete de actualización correcto.

Opción 1:
1. Inicie sesión en [Azure Percept Studio](./overview-azure-percept-studio.md).
2. En **Dispositivos**, elija el dispositivo devkit.
3. En la pestaña **General**, busque la información del **modelo** y la **versión de SW**.

Opción 2:
1. Consulte el **dispositivo IoT Edge** del servicio **IoT Hub** en Microsoft Azure Portal.
2. Elija el dispositivo devkit en la lista de dispositivos.
3. Seleccione **Device twin** (Dispositivo gemelo).
4. Desplácese por las propiedades del dispositivo gemelo, busque los elementos **"model"** (modelo) y **"swVersion"** (versión de software) en **"deviceInformation"** (información del dispositivo) y anote sus valores.

## <a name="determine-the-correct-update-package"></a>Determinación del paquete de actualización correcto
Con los elementos **model** y **swVersion** identificados en la sección anterior, compruebe la tabla siguiente para determinar qué paquete de actualización va a descargar.


|model  |swVersion  |Método de actualización  |Vínculos de descarga  |Nota  |
|---------|---------|---------|---------|---------|
|PE-101     |2020.108.101.105, <br>2020.108.114.120, <br>2020.109.101.122, <br>2020.109.116.120, <br>2021.101.106.118        |**Solo USB**         |[Paquete de actualización mediante USB 2021.105.111.112](https://go.microsoft.com/fwlink/?linkid=2155734)         |Versión de mayo (2105)         |
|PE-101     |2021.102.108.112 <br>         |OTA o USB        |[Manifiesto de OTA 2021.105.111.112 (PE-101)](https://go.microsoft.com/fwlink/?linkid=2155625)<br>[Paquete de actualización mediante OTA 2021.105.111.112](https://go.microsoft.com/fwlink/?linkid=2161538)<br>[Paquete de actualización mediante USB 2021.105.111.112](https://go.microsoft.com/fwlink/?linkid=2155734)          |Versión de mayo (2105)         |
|APDK-101     |Todas las versiones de software        |OTA o USB       | [Manifiesto de OTA 2021.105.111.112 (APDK-101)](https://go.microsoft.com/fwlink/?linkid=2163554)<br>[Paquete de actualización mediante OTA 2021.105.111.112](https://go.microsoft.com/fwlink/?linkid=2163456)<br>[Paquete de actualización mediante USB 2021.105.111.112](https://go.microsoft.com/fwlink/?linkid=2163555)        |Versión de mayo (2105)         |


## <a name="next-steps"></a>Pasos siguientes
Actualice los kits de desarrollo mediante los métodos y los paquetes de actualización determinados en la sección anterior.
- [Actualización de Azure Percept DK de forma inalámbrica](./how-to-update-over-the-air.md)
- [Actualización de Azure Percept DK mediante USB](./how-to-update-via-usb.md)