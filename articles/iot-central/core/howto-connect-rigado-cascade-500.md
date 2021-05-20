---
title: Conexión de un dispositivo Rigado Cascade 500 en Azure IoT Central | Microsoft Docs
description: Aprenda a conectar un dispositivo de puerta de enlace Rigado Cascade 500 a la aplicación de IoT Central.
services: iot-central
ms.service: iot-central
ms.topic: how-to
ms.custom:
- iot-storeAnalytics-conditionMonitor
- iot-p0-scenario
ms.author: avneets
author: avneet723
ms.date: 11/27/2019
ms.openlocfilehash: ee22c1e68f407eb2b508cd97e611ea83b82174b2
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109684134"
---
# <a name="connect-a-rigado-cascade-500-gateway-device-to-your-azure-iot-central-application"></a>Conexión de un dispositivo de puerta de enlace Rigado Cascade 500 a la aplicación de Azure IoT Central

En este artículo se describe cómo puede conectar un dispositivo de puerta de enlace Rigado Cascade 500 a la aplicación de Microsoft Azure IoT Central.

## <a name="what-is-cascade-500"></a>¿Qué es Cascade 500?

La puerta de enlace IoT Cascade 500 es una oferta de hardware de Rigado que se incluye como parte de la solución EaaS (Edge-as-a-Service) de Cascade. Este dispositivo proporciona a los equipos comerciales de proyecto y producto de IoT una eficacia de computación perimetral flexible, un sólido entorno de aplicación en contenedor y una amplia variedad de opciones de conectividad de dispositivos inalámbricos, como Bluetooth 5, LTE y Wi-Fi.

Cascade 500 está certificado para Azure IoT Plug and Play y permite incorporar fácilmente el dispositivo a sus soluciones integrales. La puerta de enlace de Cascade permite conectarse de forma inalámbrica a una variedad de sensores de supervisión de condiciones que están cerca del dispositivo de puerta de enlace. Estos sensores se pueden incorporar a IoT Central mediante el dispositivo de puerta de enlace.

## <a name="prerequisites"></a>Prerrequisitos

Necesitará lo siguiente para completar los pasos de esta guía:

[!INCLUDE [iot-central-prerequisites-basic](../../../includes/iot-central-prerequisites-basic.md)]

- Un dispositivo Rigado Cascade 500. Para más información, visite el sitio web de [Rigado](https://www.rigado.com/).

## <a name="add-a-device-template"></a>Incorporación de una plantilla de dispositivo

Para incorporar un dispositivo de puerta de enlace Cascade 500 a la instancia de la aplicación de Azure IoT Central, debe configurar una plantilla de dispositivo correspondiente dentro de la aplicación.

Para agregar una plantilla de dispositivo Cascade 500, siga estos pasos: 

1. Vaya a la pestaña ***Plantillas de dispositivo** _ en el panel izquierdo y seleccione _*+ Nueva**: ![Creación de una plantilla de dispositivo](./media/howto-connect-rigado-cascade-500/device-template-new.png)
1. La página le brinda la opción de ***crear una plantilla personalizada** _ o *_usar una plantilla de dispositivo preconfigurada_**
1. Seleccione la plantilla de dispositivo C500 en la lista de plantillas de dispositivo preconfiguradas, como se muestra a continuación: ![Selección de la plantilla de dispositivo C500](./media/howto-connect-rigado-cascade-500/device-template-preconfigured.png)
1. Seleccione ***Siguiente: Customize*** (Personalizar) para continuar con el paso siguiente. 
1. En la siguiente pantalla, seleccione ***Create*** (Crear) para incorporar la plantilla de dispositivo C500 a la aplicación IoT Central.

## <a name="retrieve-application-connection-details"></a>Recuperación de los detalles de conexión de la aplicación

Ahora deberá recuperar los valores de **Scope ID** (Id. de ámbito) y **Primary key** (Clave principal) para la aplicación de Azure IoT Central a fin de conectar el dispositivo Cascade 500. 

1. Vaya a **Administration** (Administración) en el panel izquierdo y haga clic en **Device connection** (Conexión del dispositivo). 
2. Anote el valor de **Scope ID** (Id. de ámbito) de la aplicación de IoT Central.
![Identificador de ámbito de la aplicación](./media/howto-connect-rigado-cascade-500/app-scope-id.png)
3. Ahora, haga clic en **View Keys** (Ver claves) y anote el valor de **Primary key** (Clave principal) 
![Clave principal](./media/howto-connect-rigado-cascade-500/primary-key-sas.png)  

## <a name="contact-rigado-to-connect-the-gateway"></a>Ponerse en contacto con Rigado para conectar la puerta de enlace 

Para conectar el dispositivo Cascade 500 a la aplicación de IoT Central, deberá ponerse en contacto con Rigado y proporcionarle los detalles de conexión de la aplicación de los pasos anteriores. 

Una vez que el dispositivo está conectado a Internet, Rigado podrá insertar una actualización de la configuración en el dispositivo de puerta de enlace Cascade 500 a través de un canal seguro. 

Esta actualización aplicará los detalles de conexión de IoT Central al dispositivo Cascade 500 y este aparecerá en la lista de dispositivos. 

![Lista de dispositivos](./media/howto-connect-rigado-cascade-500/devices-list-c500.png)  

Ya está listo para usar el dispositivo C500 en la aplicación de IoT Central.

## <a name="next-steps"></a>Pasos siguientes

Algunos pasos siguientes sugeridos son:

- Consultar sobre la [Conectividad de dispositivos en Azure IoT Central](./concepts-get-connected.md)
- Obtener información sobre la [Supervisión de la conectividad de dispositivos mediante la CLI de Azure](./howto-monitor-devices-azure-cli.md)
