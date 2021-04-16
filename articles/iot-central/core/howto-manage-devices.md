---
title: Administración de dispositivos en una aplicación de Azure IoT Central | Microsoft Docs
description: Como operador, aprenda a administrar dispositivos en la aplicación de Azure IoT Central. Obtenga información acerca de cómo administrar dispositivos y realizar importaciones y exportaciones en bloque de los dispositivos a la aplicación.
author: dominicbetts
ms.author: dobett
ms.date: 10/08/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: peterpr
ms.custom: contperf-fy21q2
ms.openlocfilehash: 0072e699107ea51764a728ce3e9987b86b2a6806
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106492331"
---
# <a name="manage-devices-in-your-azure-iot-central-application"></a>Administración de dispositivos en la aplicación de Azure IoT Central

En este artículo se describe cómo puede administrar dispositivos en la aplicación de Azure IoT Central como operador. Como operador, puede hacer lo siguiente:

- Usar la página **Devices** (Dispositivos) para ver, agregar y eliminar los dispositivos conectados a una aplicación de Azure IoT Central.
- Importar y exportar dispositivos en bloque.
- Mantener un inventario actualizado de los dispositivos.
- Mantenga actualizados los metadatos del dispositivo cambiando los valores almacenados en las propiedades del dispositivo desde sus vistas.
- Controlar el comportamiento de los dispositivos mediante la actualización de una configuración en un dispositivo específico desde sus vistas.

Para obtener información sobre cómo administrar grupos personalizados de dispositivos, consulte [Tutorial: Uso de grupos de dispositivos para analizar la telemetría de dispositivo](tutorial-use-device-groups.md).

## <a name="view-your-devices"></a>Visualización de los dispositivos

Para ver un dispositivo determinado:

1. Elija **Dispositivos** en el panel izquierdo. Verá una lista con todos los dispositivos y sus plantillas de dispositivo.

1. Elija una plantilla.

1. En el panel derecho de la página **Devices** (Dispositivos), verá una lista de los dispositivos creados con esa plantilla. Elija un dispositivo para ver la página de detalles de ese dispositivo:

    ![Página Detalles del dispositivo](./media/howto-manage-devices/devicelist.png)

## <a name="add-a-device"></a>Agregar un dispositivo

Para agregar un dispositivo real a una aplicación de Azure IoT Central:

1. Elija **Dispositivos** en el panel izquierdo.

1. Elija la plantilla de dispositivo desde la que va a crear un dispositivo.

1. Elija + **Nuevo**.

1. Cambie el botón de alternancia **Simulated** **On** (Activado) u **Off** (Desactivado). Un dispositivo real hace referencia a un dispositivo físico que se conecta a la aplicación de Azure IoT Central. Un dispositivo simulado tiene datos de ejemplo generados por Azure IoT Central, que los pone a su disposición.

1. Seleccione **Crear**.

1. El dispositivo aparece ahora en la lista de dispositivos de esta plantilla. Selecciónelo para ver su página de detalles, que contiene todas las vistas del dispositivo.

## <a name="import-devices"></a>Importación de dispositivos

Para conectar un gran número de dispositivos a la aplicación, puede importar dispositivos en bloque desde un archivo CSV. El archivo CSV debe tener las siguientes columnas y encabezados:

* **IOTC_DeviceID**: el identificador de dispositivo puede contener letras, números y el carácter `-`.
* **IOTC_DeviceName**: esta columna es opcional.

Para registrar dispositivos en la aplicación en bloque:

1. Elija **Dispositivos** en el panel izquierdo.

1. En el panel izquierdo, elija la plantilla de dispositivo para la que desea crear dispositivos en bloque.

    > [!NOTE]
    > Si aún no tiene una plantilla de dispositivo, puede importar dispositivos en **Todos los dispositivos** y registrarlos sin plantilla. Una vez que los dispositivos se han importado, puede migrarlos a una plantilla.

1. Seleccione **Import** (Importar).

    ![Acción de importación](./media/howto-manage-devices/bulkimport1a.png)


1. Seleccione el archivo CSV que tiene la lista de identificadores de dispositivo que desea importar.

1. La importación de dispositivos se inicia una vez que se ha cargado el archivo. En el panel Device Operations (Operaciones de dispositivo) puede realizar un seguimiento del estado de la importación. Este panel aparece automáticamente cuando se inicia la importación, o bien se puede acceder a él a través del icono de la campana de la esquina superior derecha.

1. Una vez que la importación finalizada, se muestra un mensaje de confirmación en el panel Device Operations (Operaciones de dispositivo).

    ![Importación correcta](./media/howto-manage-devices/bulkimport3a.png)

Si se produce un error en la operación de importación de dispositivos, aparece un mensaje de error en el panel Device Operations (Operaciones de dispositivo). Se genera un archivo de registro descargable con todos los errores.

## <a name="migrate-devices-to-a-template"></a>Migración de dispositivos a una plantilla

Si para registrar dispositivos inicia la importación en **All devices** (Todos los dispositivos), los dispositivos se crean sin asociación con ninguna plantilla. Los dispositivos deben estar asociados con una plantilla para explorar los datos y otros detalles sobre ellos. Siga estos pasos para asociar dispositivos con una plantilla:

1. Elija **Dispositivos** en el panel izquierdo.

1. Elija **All devices** (Todos los servicios) en el panel izquierdo:

    ![Dispositivos no asociados](./media/howto-manage-devices/unassociateddevices1a.png)

1. Use el filtro de la cuadrícula para determinar si el valor de la columna **Plantilla de dispositivo** es **Sin asociar** en cualquiera de los dispositivos.

1. Seleccione los dispositivos que quiere asociar con una plantilla:

1. Seleccione **Migrate** (Migrar):

    ![Asociar dispositivos](./media/howto-manage-devices/unassociateddevices2a.png)

1. Elija la plantilla en la lista de plantillas disponibles y seleccione **Migrate** (Migrar).

1. Los dispositivos seleccionados están asociados con la plantilla de dispositivo que eligió.

## <a name="export-devices"></a>Exportación de dispositivos

Para conectar un dispositivo real a IoT Central, necesita su cadena de conexión. Puede exportar los detalles del dispositivo de forma masiva para obtener la información que necesita para crear las cadenas de conexión del dispositivo. El proceso de exportación crea un archivo CSV con la identidad, el nombre y las claves de todos los dispositivos seleccionados.

Para realizar la exportación masiva de dispositivos desde la aplicación:

1. Elija **Dispositivos** en el panel izquierdo.

1. En el panel izquierdo, elija la plantilla de dispositivo desde la que desea exportar los dispositivos.

1. Seleccione los dispositivos que desea exportar y la acción **Exportar**.

    ![Exportación](./media/howto-manage-devices/export1a.png)

1. Se inicia el proceso de exportación. Para realizar un seguimiento del estado, utilice el panel Device Operations (Operaciones de dispositivo).

1. Cuando la exportación finaliza, se muestra un mensaje de confirmación junto con un vínculo para descargar el archivo generado.

1. Seleccione el vínculo **Download File** (Descargar archivo) para descargar el archivo en una carpeta local del disco.

    ![Exportación correcta](./media/howto-manage-devices/export2a.png)

1. El archivo CSV exportado contiene las siguientes columnas: Id. de dispositivo, nombre del dispositivo, claves de dispositivo y huellas digitales de certificado X509:

    * IOTC_DEVICEID
    * IOTC_DEVICENAME
    * IOTC_SASKEY_PRIMARY
    * IOTC_SASKEY_SECONDARY
    * IOTC_X509THUMBPRINT_PRIMARY
    * IOTC_X509THUMBPRINT_SECONDARY

Para más información sobre las cadenas de conexión y la conexión de dispositivos reales a la aplicación de IoT Central, consulte [Conectividad de dispositivos en Azure IoT Central](concepts-get-connected.md).

## <a name="delete-a-device"></a>Eliminar un dispositivo

Para eliminar ya sea un dispositivo real o simulado de la aplicación de Azure IoT Central:

1. Elija **Dispositivos** en el panel izquierdo.

1. Elija la plantilla de dispositivo del dispositivo que desea eliminar.

1. Use las herramientas de filtro para filtrar y buscar los dispositivos. Active la casilla situada junto al dispositivo que desea eliminar.

1. Elija **Eliminar**. Puede realizar un seguimiento del estado de la eliminación en el panel Device Operations (Operaciones de dispositivo).

## <a name="change-a-property"></a>Cambio de una propiedad

Las propiedades de la nube son los metadatos de dispositivo asociados al dispositivo, como la ciudad y el número de serie. Las propiedades de la nube solo existen en la aplicación IoT Central y no se sincronizan con sus dispositivos. Las propiedades que se pueden escribir controlan el comportamiento de un dispositivo y permiten establecer el estado de un dispositivo de forma remota, por ejemplo, estableciendo la temperatura objetivo de un dispositivo de termostato.  Las propiedades del dispositivo las establece el dispositivo y son de solo lectura en IoT Central. Puede ver y actualizar las propiedades en las vistas de **Detalles del dispositivo**.

1. Elija **Dispositivos** en el panel izquierdo.

1. Elija la plantilla de dispositivo del dispositivo cuyas propiedades desea modificar y seleccione el dispositivo de destino.

1. Elija la vista que contiene las propiedades del dispositivo. Esta vista le permite especificar valores y seleccionar **Guardar** en la parte superior de la página. Aquí verá las propiedades del dispositivo y sus valores actuales. Tanto las propiedades de la nube como las propiedades que se pueden escribir tienen campos editables, mientras que las propiedades del dispositivo son de solo lectura. En el caso de las propiedades que se pueden escribir, puede ver su estado de sincronización en la parte inferior del campo. 

1. Modifique las propiedades con los valores que necesite. Puede modificar varias propiedades a la vez y actualizarlas todas al mismo tiempo.

1. Elija **Guardar**. Si guardó las propiedades que se pueden escribir, los valores se envían al dispositivo. Cuando el dispositivo confirma el cambio de la propiedad que se puede escribir, el estado vuelve a **synced** (sincronizado). Si guardó una propiedad de la nube, el valor se actualiza.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar dispositivos en la aplicación Azure IoT Central, el paso siguiente que sugerimos es que obtenga más información sobre cómo [Configurar reglas](howto-configure-rules.md) para sus dispositivos.
