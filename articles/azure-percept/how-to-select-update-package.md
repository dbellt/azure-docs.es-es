---
title: Selección del paquete de actualización más adecuado para Azure Percept DK
description: Obtenga información sobre cómo identificar la versión de Azure Percept DK y seleccionar el paquete de actualización más adecuado.
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 05/04/2021
ms.custom: template-how-to
ms.openlocfilehash: 6f32e785af7f302124a11613a40e286fe3b641b9
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488058"
---
# <a name="how-to-determine-and-download-the-best-update-package-for-ota-and-usb-updates"></a>Determinación y descarga del paquete de actualización más adecuado para las actualizaciones mediante OTA y USB

En esta página se proporcionan instrucciones sobre cómo seleccionar el paquete de actualización que mejor se adapte a su kit de desarrollo y las ubicaciones de descarga de los paquetes de actualización.

Para más información sobre cómo actualizar el dispositivo, consulte estos artículos:
- [Actualización de Azure Percept DK de forma inalámbrica](https://docs.microsoft.com/azure/azure-percept/how-to-update-over-the-air)
- [Actualización de Azure Percept DK mediante USB](https://docs.microsoft.com/azure/azure-percept/how-to-update-via-usb)


## <a name="prerequisites"></a>Requisitos previos

- Un kit de desarrollo [Azure Percept DK](https://go.microsoft.com/fwlink/?linkid=2155270) que se haya [configurado y conectado a Azure Percept Studio e IoT Hub](https://docs.microsoft.com/azure/azure-percept/quickstart-percept-dk-set-up).

## <a name="identify-the-model-name-and-software-version-of-your-dev-kit"></a>Identificación del nombre del modelo y la versión de software del kit de desarrollo
Para asegurarse de aplicar el paquete de actualización correcto al kit de desarrollo, primero debe determinar qué versión de software se está ejecutando actualmente.

> [!WARNING]
> La aplicación de un paquete de actualización incorrecto podría dar lugar a que el kit de desarrollo se vuelva inoperable. Es importante que siga estos pasos para asegurarse de que aplica el paquete de actualización correcto.

1. Encienda el kit de desarrollo y asegúrese de que esté conectado a Azure Percept Studio.
1. En Azure Percept Studio, seleccione **Devices** (Dispositivos) en el menú de la izquierda.
1. En la lista de dispositivos, seleccione el nombre del dispositivo que está conectado actualmente. El estado será **Connected** (Conectado).
1. Seleccione **Open device in IoT Hub** (Abrir dispositivo en IoT Hub).
1. Es posible que tenga que volver a iniciar sesión en su cuenta de Azure.
1. Seleccione **Device twin** (Dispositivo gemelo).
1. Desplácese por las propiedades del dispositivo gemelo, busque los elementos **"model"** (modelo) y **"swVersion"** (versión de software) en **"deviceInformation"** (información del dispositivo) y anote sus valores.

## <a name="determine-the-correct-update-package"></a>Determinación del paquete de actualización correcto
Con los elementos **model** y **swVersion** identificados en la sección anterior, compruebe la tabla siguiente para determinar qué paquete de actualización va a descargar.


|model  |swVersion  |Método de actualización  |Vínculos de descarga  |
|---------|---------|---------|---------|
|PE-101     |2020.108.101.105, <br>2020.108.114.120, <br>2020.109.101.122, <br>2020.109.116.120, <br>2021.101.106.118        |**Solo USB**         |[Paquete de actualización mediante USB](https://go.microsoft.com/fwlink/?linkid=2155734)         |
|PE-101     |2021.102.108.112 <br>         |OTA o USB        |[Manifiesto de OTA](https://go.microsoft.com/fwlink/?linkid=2155625)<br>[Paquete de actualización mediante OTA](https://go.microsoft.com/fwlink/?linkid=2161538)<br>[Paquete de actualización mediante OTA](https://go.microsoft.com/fwlink/?linkid=2155734)          |
|APDK-101     |Todas las versiones de software        |OTA o USB       | [Manifiesto de OTA](https://go.microsoft.com/fwlink/?linkid=2162292)<br>[Paquete de actualización mediante OTA](https://go.microsoft.com/fwlink/?linkid=2161538)<br>[Paquete de actualización mediante OTA](https://go.microsoft.com/fwlink/?linkid=2155734)        |



## <a name="next-steps"></a>Pasos siguientes
Actualice los kits de desarrollo mediante los métodos y los paquetes de actualización determinados en la sección anterior.
- [Actualización de Azure Percept DK de forma inalámbrica](https://docs.microsoft.com/azure/azure-percept/how-to-update-over-the-air)
- [Actualización de Azure Percept DK mediante USB](https://docs.microsoft.com/azure/azure-percept/how-to-update-via-usb)

