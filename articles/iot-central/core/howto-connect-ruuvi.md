---
title: Conexión de un sensor RuuviTag en Azure IoT Central | Microsoft Docs
description: Aprenda a conectar un sensor ambiental RuuviTag a la aplicación de IoT Central.
services: iot-central
ms.service: iot-central
ms.topic: how-to
ms.custom:
- iot-storeAnalytics-conditionMonitor
- iot-p0-scenario
ms.author: avneets
author: avneet723
ms.date: 11/27/2019
ms.openlocfilehash: 13b1e22f1e1e5f51d524e80d0bf102b744fbec3d
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109684602"
---
# <a name="connect-a-ruuvitag-sensor-to-your-azure-iot-central-application"></a>Conexión de un sensor RuuviTag a la aplicación de Azure IoT Central

En este artículo se describe cómo puede conectar un sensor RuuviTag a la aplicación de Microsoft Azure IoT Central.

¿Qué es RuuviTag?

RuuviTag es una plataforma avanzada de señales de sensor de código abierto diseñada para satisfacer las necesidades de clientes empresariales, desarrolladores, creadores, alumnos y aficionados. El dispositivo está configurado para funcionar tan pronto como se saca de su caja y está listo para implementarse donde lo necesite. Se trata de una señal Bluetooth LE con un sensor ambiental y un acelerómetro integrados.

RuuviTag se comunica a través de BLE (Bluetooth de bajo consumo) y requiere un dispositivo de puerta de enlace para comunicarse con Azure IoT Central. Asegúrese de que tiene un dispositivo de puerta de enlace, como Rigado Cascade 500, e instálelo para habilitar la conexión de un sensor RuuviTag a IoT Central.

Si quiere instalar un dispositivo de puerta de enlace Rigado Cascade 500, siga [estas instrucciones](./howto-connect-rigado-cascade-500.md).

## <a name="prerequisites"></a>Prerrequisitos

Para conectar sensores RuuviTag, necesita los siguientes recursos:

[!INCLUDE [iot-central-prerequisites-basic](../../../includes/iot-central-prerequisites-basic.md)]

- Un sensor RuuviTag. Para más información, visite el sitio web de [RuuviTag](https://ruuvi.com/).

- Un dispositivo Rigado Cascade 500 u otra puerta de enlace BLE. Para más información, visite el sitio web de [Rigado](https://www.rigado.com/).


## <a name="add-a-ruuvitag-device-template"></a>Adición de una plantilla de dispositivo RuuviTag

Para incorporar un sensor RuuviTag a la instancia de la aplicación de Azure IoT Central, debe configurar una plantilla de dispositivo correspondiente dentro de la aplicación.

Para agregar una plantilla de dispositivo RuuviTag, siga estos pasos:

1. Vaya a la pestaña ***Plantillas de dispositivo** _ en el panel izquierdo y seleccione _*+ Nuevo**: ![Creación de una plantilla de dispositivo](./media/howto-connect-ruuvi/devicetemplate-new.png) La página ofrece una opción para * **crear una plantilla personalizada**_ o _ *_usar una plantilla de dispositivo preconfigurada_**
1. Seleccione la plantilla de dispositivo RuuviTag de la lista de plantillas de dispositivo preconfiguradas, como se muestra a continuación:  ![Selección de la plantilla de dispositivo RuuviTag](./media/howto-connect-ruuvi/devicetemplate-preconfigured.png)
1. Seleccione ***Siguiente: Customize*** (Personalizar) para continuar con el paso siguiente.
1. En la siguiente pantalla, seleccione ***Create*** (Crear) para incorporar la plantilla de dispositivo C500 a la aplicación IoT Central.

## <a name="connect-a-ruuvitag-sensor"></a>Conexión de un sensor RuuviTag

Como se mencionó anteriormente, para conectar el sensor RuuviTag con la aplicación de IoT Central, debe configurar un dispositivo de puerta de enlace. En los pasos siguientes se supone que ha configurado un dispositivo de puerta de enlace Rigado Cascade 500.  

1. Encienda el dispositivo Rigado Cascade 500 y conéctelo a la red (Ethernet o inalámbrica).
1. Retire la cubierta del sensor RuuviTag y tire de la lengüeta de plástico para proteger la conexión con la batería.
1. Coloque el sensor RuuviTag cerca de una puerta de enlace Rigado Cascade 500 ya configurada en la aplicación de IoT Central.
1. En unos segundos, el sensor RuuviTag aparecerá en la lista de dispositivos dentro de IoT Central.  
    ![Lista de dispositivos RuuviTag](./media/howto-connect-ruuvi/ruuvi-devicelist.png)

Ahora puede usar este sensor RuuviTag en la aplicación de IoT Central.  

## <a name="create-a-simulated-ruuvitag"></a>Creación de un sensor RuuviTag simulado

Si no tiene a mano un dispositivo RuuviTag físico, puede crear uno simulado para realizar pruebas en la aplicación de Azure IoT Central.

Para crear un sensor RuuviTag simulado, siga estos pasos:

1. Seleccione **Devices (Dispositivos) > RuuviTag**.
1. Seleccione **+ Nuevo**.
1. Especifique un identificador único en **Device ID** (Id. de dispositivo) y un nombre descriptivo en **Device name** (Nombre de dispositivo).  
1. Habilite la opción **Simulated** (Simulado).
1. Seleccione **Crear**.  

## <a name="next-steps"></a>Pasos siguientes

Algunos pasos siguientes sugeridos son:

- Consultar sobre la [Conectividad de dispositivos en Azure IoT Central](./concepts-get-connected.md)
- Obtener información sobre la [Supervisión de la conectividad de dispositivos mediante la CLI de Azure](./howto-monitor-devices-azure-cli.md)
