---
title: Administración de sensores en el portal de Defender para IoT
description: Aprenda a incorporar, ver y administrar sensores en el portal de Defender para IoT.
ms.date: 04/29/2021
ms.topic: how-to
ms.openlocfilehash: 5db2b440a70b2e172a1167f6b8759b78d0d7878e
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109654911"
---
# <a name="manage-sensors-in-the-defender-for-iot-portal"></a>Administración de sensores en el portal de Defender para IoT

En este artículo se describe cómo incorporar, ver y administrar sensores en el [portal de Defender para IoT](https://portal.azure.com/#blade/Microsoft_Azure_IoT_Defender/IoTDefenderDashboard/Getting_Started).

## <a name="onboard-sensors"></a>Sensores incorporados

Para incorporar un sensor, debe registrarlo en Azure Defender para IoT y descargar un archivo de activación de sensor.

### <a name="register-the-sensor"></a>Registro del sensor

**Para registrarse:**

1. Vaya a la **página principal** del [portal de Defender para IoT](https://portal.azure.com/#blade/Microsoft_Azure_IoT_Defender/IoTDefenderDashboard/Getting_Started).

1. Seleccione **Onboard sensor** (Incorporar sensor).

   :::image type="content" source="media/how-to-manage-sensors-on-the-cloud/onboard-a-sensor.png" alt-text="Seleccione Incorporar sensor para iniciar el proceso de incorporación del sensor.":::

1. Cree un nombre de sensor. 

    Se recomienda incluir la dirección IP del sensor instalado como parte del nombre o usar un nombre que sea fácil de identificar. De esta forma, se garantiza un seguimiento más sencillo y una nomenclatura coherente entre el nombre de registro en el [portal de Azure Defender para IoT](https://portal.azure.com/#blade/Microsoft_Azure_IoT_Defender/IoTDefenderDashboard/Getting_Started) y la dirección IP del sensor implementado que se muestra en la consola del sensor.

1. Asocie el sensor a una suscripción de Azure.

    :::image type="content" source="media/how-to-manage-sensors-on-the-cloud/name-subscription.png" alt-text="Escriba un nombre descriptivo y conecte el sensor a una suscripción.":::

1. Elija un modo de conexión del sensor mediante el botón de alternancia **Cloud connected** (Conectado a la nube). Si el botón de alternancia está activado, el sensor está conectado a la nube. Si el botón de alternancia está desactivado, el sensor está administrado de forma local.

   - **Cloud-connected sensors** (Sensores conectados a la nube): la información que detecta el sensor se muestra en la consola del sensor. Además, se proporciona información de alertas mediante un centro de IoT, que puede compartirse con otros servicios de Azure, como Azure Sentinel. Asimismo, los paquetes de inteligencia sobre amenazas pueden insertarse en los sensores desde el portal de Azure Defender para IoT. Por el contrario, cuando el sensor no está conectado a la nube, debe descargar los paquetes de inteligencia sobre amenazas y, después, cargarlos en los sensores de la empresa. Para permitir que Defender para IoT inserte los paquetes en los sensores, habilite el botón de alternancia **Automatic Threat Intelligence Updates** (Actualizaciones automáticas de inteligencia sobre amenazas). Para obtener más información, vea [Investigación y paquetes de inteligencia sobre amenazas](how-to-work-with-threat-intelligence-packages.md).
   
   Elija un centro de IoT que sirva como puerta de enlace entre este sensor y el portal de Azure Defender para IoT. Defina un nombre de sitio y una zona. También puede agregar etiquetas descriptivas. El nombre del sitio, la zona y las etiquetas son entradas descriptivas en la página [Sites and sensors](#view-onboarded-sensors) (Sitios y sensores).

   - **Locally managed sensors** (Sensores administrados de forma local): la información que detectan los sensores se muestra en la consola del sensor. Si trabaja en una red aislada y quiere una vista unificada de toda la información detectada por varios sensores administrados de forma local, trabaje con la consola de administración local.

   En el caso de los sensores conectados a la nube, el nombre que se define durante la incorporación es el nombre que aparece en la consola del sensor. Este nombre no se puede cambiar directamente desde la consola. En el caso de los sensores administrados de forma local, el nombre que se aplique durante la incorporación se almacenará en Azure, pero se puede actualizar en la consola del sensor.

### <a name="download-the-sensor-activation-file"></a>Descarga del archivo de activación del sensor

El archivo de activación del sensor contiene instrucciones sobre el modo de administración del sensor. Descargue un archivo de activación único para cada sensor que implemente. Un usuario que inicia sesión en la consola del sensor por primera vez carga el archivo de activación en el sensor.

**Para descargar un archivo de activación:**

1. En la página **Onboard sensor** (Incorporar sensor), seleccione **Download activation file** (Descargar archivo de activación).

1. Ponga el archivo a disposición del usuario que inicia sesión en la consola del sensor por primera vez.

## <a name="view-onboarded-sensors"></a>Visualización sensores incorporados

En el [portal de Defender para IoT](https://portal.azure.com/#blade/Microsoft_Azure_IoT_Defender/IoTDefenderDashboard/Getting_Started), puede ver información operativa importante sobre los sensores incorporados.

1. Seleccione **Sites and sensors** (Sitios y sensores). La página muestra el número de sensores incorporados, el número de sensores conectados a la nube y administrados localmente, así como:

    :::image type="content" source="media/how-to-manage-sensors-on-the-cloud/sites-and-sensors.png" alt-text="Seleccione la página de sitios y sensores para ver todos los sensores asociados.":::

    - El nombre de sensor asignado durante la incorporación.
    - El tipo de conexión (conectado a la nube o administrado localmente).
    - La zona asociada al sensor.
    - La versión instalada del sensor.
    - El estado de conexión del sensor a la nube.
    - La última vez que se detectó que el sensor se conectaba a la nube.

## <a name="manage-onboarded-sensors"></a>Administración de sensores incorporados

Use el [portal de Defender para IoT](https://portal.azure.com/#blade/Microsoft_Azure_IoT_Defender/IoTDefenderDashboard/Getting_Started) para realizar las tareas de administración relacionadas con los sensores.

Los sensores incorporados se pueden ver en la página **Sites and Sensors** (Sitios y sensores). También se puede editar la información del sensor desde esta página.

### <a name="export-sensor-details"></a>Exportación de los detalles del sensor

Para exportar información del sensor incorporado, seleccione el icono **Exportar** en la parte superior de la página **Sites and sensors** (Sitios y sensores).

:::image type="content" source="media/how-to-manage-sensors-on-the-cloud/export-sensors.png" alt-text="Seleccione el botón Exportar para exportar la información del sensor.":::

### <a name="edit-sensor-zone-details"></a>Edición de los detalles de la zona del sensor

Use las opciones de edición **Sites and Sensors** (Sitios y sensores) para editar el nombre y la zona del sensor.

**Para editar:**

1. Seleccione los **puntos suspensivos** ( **...** ) para el sensor que quiera editar.

1. Seleccione **Editar**.

1. Actualice la zona del sensor o cree una nueva.

### <a name="delete-a-sensor"></a>Eliminación de un sensor

Si elimina un sensor conectado a la nube, no se enviará información al centro de IoT. Elimine los sensores conectados localmente cuando deje de trabajar con ellos.

**Para eliminar un sensor:**

1. Haga clic en el botón de puntos suspensivos **(...)** del sensor que quiera administrar.

1. Seleccione **Eliminar sensor**.

### <a name="reactivate-a-sensor"></a>Reactivación de un sensor 

Es posible que tenga que reactivar el sensor porque quiera:

- **Work in cloud-connected mode instead of locally managed mode** (Trabajar en el modo conectado a la nube en lugar del modo administrado localmente): Después de la reactivación, las detecciones del sensor se muestran en el sensor y la información de alertas recién detectadas se entrega a través de IoT Hub. Esta información se puede compartir con otros servicios de Azure, como Azure Sentinel.

- **Trabajar en modo administrado localmente en lugar de en modo conectado a la nube**: Después de la reactivación, la información de detección del sensor solo se mostrará en el sensor.

- **Associate the sensor to a new IoT hub** (Asociar el sensor a un nuevo centro de IoT):  Para ello, vuelva a registrar el sensor con un nuevo centro de conectividad y, luego, descargue un nuevo archivo de activación.

**Para reactivar un sensor:**

1. Vaya a la página **Sites and sensors** (Sitios y sensores) del [portal de Defender para IoT](https://portal.azure.com/#blade/Microsoft_Azure_IoT_Defender/IoTDefenderDashboard/Getting_Started).

1. Seleccione el sensor para el que quiere cargar un nuevo archivo de activación.

1. Seleccione los **puntos suspensivos** ( **...** ) y, a continuación, seleccione **Eliminar sensor**.

    :::image type="content" source="media/how-to-manage-sensors-on-the-cloud/delete-a-sensor.png" alt-text="Seleccione los puntos suspensivos y, a continuación, Eliminar sensor.":::

1. Vuelva a [incorporar el sensor](#onboard-sensors) en el nuevo modo o con un nuevo centro de IoT al seleccionar **Onboard a sensor** (incorporar un sensor) en la página Introducción.

1. Descargue el archivo de activación.

1. Inicie sesión en la consola del sensor de Defender para IoT.

1. En la consola del sensor, seleccione **Configuración del sistema** y, a continuación, seleccione **Reactivación**.

   :::image type="content" source="media/how-to-manage-sensors-on-the-cloud/reactivate.png" alt-text="Carga del archivo de activación para reactivar el sensor.":::

1. Seleccione **Cargar** y elija el archivo que guardó en la página de incorporación del sensor.

1. Seleccione **Activar**.

## <a name="next-steps"></a>Pasos siguientes

[Activación y configuración del sensor](how-to-activate-and-set-up-your-sensor.md)
