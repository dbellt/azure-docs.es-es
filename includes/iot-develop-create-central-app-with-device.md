---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 04/28/2021
ms.author: timlt
ms.custom: include file
ms.openlocfilehash: 981ebfece7bc12584adfdf1e565bd275d8e1b2fb
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108226751"
---
## <a name="create-an-application"></a>Crear una aplicación
Existen varias formas de conectar los dispositivos a Azure IoT. En esta sección, aprenderá a conectar un dispositivo con Azure IoT Central. IoT Central es una plataforma de aplicaciones IoT que reduce el costo y la complejidad de administrar dispositivos en una solución IoT.

Para crear una aplicación:
1. Vaya a [Azure IoT Central](https://apps.azureiotcentral.com/) e inicie sesión con una cuenta personal, profesional o educativa de Microsoft.
1. Vaya a **Compilar** y seleccione **Aplicaciones personalizadas**.
   :::image type="content" source="media/iot-develop-create-central-app-with-device/iot-central-build.png" alt-text="Página de inicio de IoT Central":::
1. En **Nombre de aplicación**, escriba un nombre único o use el nombre generado.
1. En **URL**, escriba un prefijo de dirección URL de aplicación fácil de recordar o use el prefijo de dirección URL generado.
1. En **Plantilla de aplicación**, deje la opción *Aplicación personalizada*. 
1. Seleccione un opción en **Plan de tarifa**. 
    - Para usar la aplicación de forma gratuita durante siete días, seleccione **Gratis**. Las aplicaciones gratuitas se pueden convertir a los precios estándar antes de que expiren.
    - Opcionalmente, se puede seleccionar un plan de tarifas estándar. Si selecciona los precios estándar, aparecerán más opciones y deberá establecer un **directorio**, una **suscripción de Azure** y una **ubicación**. Para obtener información sobre los precios, consulte [Precios de Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/). 
        - **Directorio** es la instancia de Azure Active Directory en que se crea la aplicación. Azure Active Directory contiene identidades de usuario, credenciales y otra información de la organización. Si no tiene Azure Active Directory, se crea automáticamente al crear una suscripción de Azure.
        - Una **suscripción de Azure** permite crear instancias de los servicios de Azure. IoT Central aprovisiona los recursos de su suscripción. Si no tiene una suscripción, puede crear una [gratis](https://aka.ms/createazuresubscription). Si tiene una suscripción, puede seleccionarla en la lista desplegable.
        - **Ubicación** es la [ubicación geográfica de Azure](https://azure.microsoft.com/global-infrastructure/geographies/) en la que se crea una aplicación. Seleccione una ubicación que se encuentre lo más cercana físicamente posible a los dispositivos a fin de obtener un rendimiento óptimo. Una vez elegida una ubicación, no podrá mover la aplicación a otra.

    :::image type="content" source="media/iot-develop-create-central-app-with-device/iot-central-pricing.png" alt-text="Cuadro de diálogo de una nueva aplicación de IoT Central":::
1. Seleccione **Crear**.
    
    Una vez que IoT Central crea la aplicación, le redirige al panel de la aplicación.
    :::image type="content" source="media/iot-develop-create-central-app-with-device/iot-central-created.png" alt-text="Panel de aplicación nueva de IoT Central":::

## <a name="add-a-device"></a>Agregar un dispositivo
En esta sección, se agrega un nuevo dispositivo a una aplicación de IoT Central. El dispositivo es una instancia de una plantilla de dispositivo que representa un dispositivo real o simulado que se conectará a la aplicación. 

Para crear un dispositivo:
1. En el panel izquierdo, seleccione **Dispositivos** y, después, seleccione **+ Nuevo**.
1. En **Plantilla de dispositivo** deje la opción *Sin asignar*.

    > [!NOTE]
    > En aras de la sencillez, en este inicio rápido, se conecta un dispositivo simulado que usa una plantilla sin asignar. Si sigue usando IoT Central para administrar dispositivos, aprenderá a usar plantillas de dispositivo. Para obtener información general sobre cómo trabajar con plantillas de dispositivo, consulte [Inicio rápido: Adición de un dispositivo simulado a una aplicación de IoT Central](../articles/iot-central/core/quick-create-simulated-device.md).
1. Establezca un **nombre de dispositivo** y un **identificador de dispositivo** descriptivos. Opcionalmente, use los valores generados.
    :::image type="content" source="media/iot-develop-create-central-app-with-device/iot-central-create-device.png" alt-text="Cuadro de diálogo de nuevo dispositivo de IoT Central":::
1. Seleccione **Crear**.

    El dispositivo creado aparece en la lista **Todos los dispositivos**.
    :::image type="content" source="media/iot-develop-create-central-app-with-device/iot-central-devices-list.png" alt-text="Lista de todos los dispositivos de IoT Central":::
    
Para obtener los detalles de la conexión del nuevo dispositivo:
1. En la lista **Todos los dispositivos**, haga clic en el nombre del dispositivo vinculado para mostrar los detalles. 
1. En el menú superior, seleccione **Conectar**.

    En el cuadro de diálogo **Conexión del dispositivo** se muestran los detalles de la conexión: :::image type="content" source="media/iot-develop-create-central-app-with-device/iot-central-device-connect.png" alt-text="Detalles de conexión de dispositivo de IoT Central":::
1. Copie los siguientes valores del cuadro de diálogo **Conexión del dispositivo** en una ubicación segura, ya que los usará para conectar el dispositivo a IoT Central.
    * `ID scope`
    * `Device ID`
    * `Primary key`