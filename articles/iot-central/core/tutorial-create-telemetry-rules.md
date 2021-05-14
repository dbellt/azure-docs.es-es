---
title: 'Tutorial: Creación y administración de reglas en la aplicación de Azure IoT Central'
description: En este tutorial se muestra cómo las reglas de Azure IoT Central le permiten supervisar los dispositivos casi en tiempo real e invocar automáticamente acciones, como el envío de correo electrónico, cuando la regla se desencadena.
author: dominicbetts
ms.author: dobett
ms.date: 01/08/2021
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 0d59f50d6fa4f21676cef01ffe0dde8ed1fa4441
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108768776"
---
# <a name="tutorial-create-a-rule-and-set-up-notifications-in-your-azure-iot-central-application"></a>Tutorial: Creación de una regla y configuración de las notificaciones en la aplicación de Azure IoT Central

Puede usar Azure IoT Central para supervisar de forma remota los dispositivos conectados. Las reglas de Azure IoT Central le permiten supervisar los dispositivos casi en tiempo real e invocar acciones automáticamente, como el envío de correo electrónico. En este artículo se explica cómo crear reglas para supervisar los datos de telemetría enviados por el dispositivo.

Los dispositivos usan la telemetría para enviar datos numéricos. Cuando los datos de telemetría seleccionados superan un umbral especificado, se desencadena una regla.

En este tutorial se crea una regla que envía un mensaje de correo electrónico cuando la temperatura de un dispositivo de sensor simulado supera los 70&deg; F.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear una regla
> * Adición de una acción de correo electrónico

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, complete los inicios rápidos [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md) y [Adición de un dispositivo simulado a una aplicación de IoT Central](./quick-create-simulated-device.md) para crear la plantilla de dispositivo **Sensor Controller** con la que va a trabajar.

## <a name="create-a-rule"></a>Crear una regla

Para crear una regla de telemetría, la plantilla de dispositivo debe incluir al menos un valor de telemetría. En este tutorial se usa un dispositivo **Sensor Controller** simulado que envía datos de telemetría de temperatura y humedad. Ha agregado esta plantilla de dispositivo y creado un dispositivo simulado en el inicio rápido [Incorporación de un dispositivo simulado a la aplicación de IoT Central](./quick-create-simulated-device.md). La regla supervisa la temperatura notificada por el dispositivo y envía un correo electrónico cada vez que sube de 70 grados.

> [!NOTE]
> Hay un límite de 50 reglas por aplicación.

1. En el panel izquierdo, seleccione **Rules** (Reglas).

1. Si aún no ha creado ninguna regla, consulte la siguiente pantalla:

    :::image type="content" source="media/tutorial-create-telemetry-rules/rules-landing-page.png" alt-text="Captura de pantalla que muestra la lista vacía de reglas":::

1. Seleccione **+ Nuevo** para agregar una regla nueva.

1. Escriba el nombre _Temperature monitor_ para identificar la regla y presione Entrar.

1. Seleccione la plantilla de dispositivo **Sensor Controller**. De forma predeterminada, la regla se aplica automáticamente a todos los dispositivos asociados con la plantilla de dispositivo. Para filtrar un subconjunto de los dispositivos, seleccione **+ Filter** (+ Filtro) y use las propiedades de dispositivo para identificar los dispositivos. Para deshabilitar la regla, alterne el botón **Habilitado/Deshabilitado**:

    :::image type="content" source="media/tutorial-create-telemetry-rules/device-filters.png" alt-text="Captura de pantalla que muestra la selección de la plantilla de dispositivo en la definición de la regla":::

### <a name="configure-the-rule-conditions"></a>Configuración de las condiciones de la regla

Las condiciones definen los criterios que la regla supervisa. En este tutorial, configurará la regla para que se active cuando la temperatura supere los 70 &deg;F.

1. Seleccione **Temperature** (Temperatura) en la lista desplegable **Telemetry** (Telemetría).

1. A continuación, elija **Is greater than** (Es mayor que) como **Operator** (Operador) y escriba _70_ en **Value** (Valor).

    :::image type="content" source="media/tutorial-create-telemetry-rules/condition-filled-out.png" alt-text="Captura de pantalla que muestra la condición de temperatura de la regla":::

1. Opcionalmente, puede establecer un valor de **Time aggregation** (Agregación de tiempo). Al seleccionar una agregación de tiempo, también debe seleccionar un tipo de agregación, como la media o la suma, en la lista desplegable de agregación.

    * Sin la agregación, la regla se desencadena para cada punto de datos de telemetría que cumple la condición. Por ejemplo, si configura la regla para desencadenarse cuando la temperatura está por encima de 70, la regla se desencadena casi al instante cuando la temperatura del dispositivo supere este valor.
    * Con la agregación, la regla se desencadena si el valor agregado de los puntos de datos de telemetría de la ventana de tiempo cumple la condición. Por ejemplo, si configura la regla para desencadenarse cuando la temperatura sea superior a 70 con una agregación de tiempo media de 10 minutos, la regla se desencadena cuando el dispositivo informa de una temperatura media por encima de 70, calculada a lo largo de un intervalo de 10 minutos.

    :::image type="content" source="media/tutorial-create-telemetry-rules/aggregate-condition-filled-out.png" alt-text="Captura de pantalla que muestra la condición de agregado rellenada":::

Puede agregar varias condiciones a una regla seleccionando **+ Condition** (+ Condición). Cuando se especifican varias condiciones, deben cumplirse todas ellas para que la regla se desencadene. Cada condición está unida por una cláusula `AND` implícita. Si usa la agregación de tiempo con varias condiciones, se deben agregar todos los valores de telemetría.

### <a name="configure-actions"></a>Configuración de acciones

Después de definir la condición, configure las acciones que deben llevarse a cabo cuando se desencadene la regla. Las acciones se invocan cuando todas las condiciones especificadas en la regla se evalúan como verdaderas.

1. Seleccione **+ Email** (+ Correo electrónico) en la sección **Actions** (Acciones).

1. Escriba _Temperature warning_ (Advertencia de temperatura) como nombre para mostrar de la acción, la dirección de correo electrónico en el campo **To** (Para) y _You should check the device!_ (Debe comprobar el dispositivo.) como la nota que va a aparecer en el cuerpo del correo electrónico.

    > [!NOTE]
    > Solo se envían mensajes de correo electrónico a los usuarios que se han agregado a la aplicación y han iniciado sesión al menos una vez. Obtenga más información sobre la [administración de usuarios](howto-administer.md) en Azure IoT Central.

    :::image type="content" source="media/tutorial-create-telemetry-rules/configure-action.png" alt-text="Captura de pantalla que muestra la acción de correo electrónico para la regla":::

1. Para guardar la acción, elija **Listo**. Puede agregar varias acciones a una regla.

1. Para guardar la regla, elija **Guardar**. La regla está activa en unos minutos e inicia la supervisión de telemetría que se envía a la aplicación. Cuando se cumple la condición especificada en la regla, la regla desencadena la acción de correo electrónico configurada.

Después de un tiempo, recibirá un mensaje de correo electrónico cuando se desencadene la regla:

:::image type="content" source="media/tutorial-create-telemetry-rules/email.png" alt-text="Captura de pantalla que muestra el correo electrónico de notificación":::

## <a name="delete-a-rule"></a>Eliminar una regla

Si ya no necesita una regla, elimínela; para ello, abra la regla y seleccione **Eliminar**.

## <a name="enable-or-disable-a-rule"></a>Habilitación o deshabilitación de una regla

Elija la regla que quiere habilitar o deshabilitar. Alterne el botón **Habilitado/Deshabilitado** en la regla para habilitar o deshabilitar la regla para todos los dispositivos que tengan el ámbito de esta.

## <a name="enable-or-disable-a-rule-for-specific-devices"></a>Habilitación o deshabilitación de una regla para dispositivo específicos

Elija la regla que quiere personalizar. Use uno o varios filtros de la sección **Dispositivos de destino** para restringir el ámbito de la regla a los dispositivos que desea supervisar.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

* Crear una regla basada en la telemetría
* Agregar una acción

Ahora que ha definido una regla basada en umbral, el siguiente paso que se recomienda dar es:

> [!div class="nextstepaction"]
> [Creación de webhooks en reglas](./howto-create-webhooks.md).
