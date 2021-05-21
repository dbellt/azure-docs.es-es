---
title: Migración de una aplicación de Azure IoT Central v2 a v3 Microsoft Docs
description: Aprenda a migrar una aplicación de Azure IoT Central v2 a v3.
author: troyhopwood
ms.author: troyhop
ms.date: 01/18/2021
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 0868720668f5db09748e6976327f1500bc8a4781
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108733362"
---
# <a name="migrate-your-v2-iot-central-application-to-v3"></a>Migración de una aplicación de IoT Central v2 a v3

Actualmente, cuando se crea una nueva aplicación de IoT Central, se trata de una aplicación de la versión 3. Si anteriormente ha creado una aplicación, en función del momento en que lo hiciera, puede ser de la versión 2. En este artículo se explica cómo migrar una aplicación de la versión 2 a la 3 para asegurarse de usar las características más recientes de IoT Central.

Para aprender a identificar la versión de una aplicación de IoT Central, vea [Acerca de su aplicación](howto-get-app-info.md).

Los pasos necesarios para migrar una aplicación de v2 a v3 son:

1. Crear una nueva aplicación de la versión 3 a partir de la aplicación de la versión 2.
1. Configurar la aplicación de la versión 3.
1. Eliminar la aplicación de la versión 2.

## <a name="create-a-new-v3-application"></a>Creación de una nueva aplicación de la versión 3

Use el asistente para migración para crear una nueva aplicación de la versión 3.

IoT Central no admite la migración a una aplicación existente de v3. Para migrar automáticamente los dispositivos existentes, use el asistente para migración a fin de crear la aplicación de la versión v3.

El asistente para migración:

- Crea una nueva aplicación de la versión 3.
- Comprueba la compatibilidad de las plantillas de dispositivo con v3.
- Copia todas las plantillas de dispositivo en la nueva aplicación de la versión 3.
- Copia todas las asignaciones de usuarios y roles en la nueva aplicación de la versión 3.

> [!NOTE]
> Para garantizar la compatibilidad de los dispositivos con v3, los tipos primitivos de la plantilla de dispositivo se convierten en propiedades de objeto. No es necesario hacer ningún cambio en el código del dispositivo.

Debe ser administrador para migrar una aplicación a v3.

1. En el menú **Administración**, seleccione **Migrate to a V3 application** (Migrar a una aplicación de v3):

    :::image type="content" source="media/howto-migrate/migrate-menu.png" alt-text="Captura de pantalla que muestra el asistente para migración de aplicaciones":::

1. Escriba un nuevo **Nombre de aplicación** y, si quiere, cambie la **URL** generada automáticamente. La dirección URL no puede ser la misma que la de la aplicación actual de la versión 2. Pero puede cambiar la dirección URL más adelante, después de eliminar la aplicación de v2.

1. Seleccione **Create a new V3 app** (Crear una nueva aplicación de v3).

    :::image type="content" source="media/howto-migrate/create-app.png" alt-text="Captura de pantalla que muestra las opciones del asistente para migración de aplicaciones":::

    En función del número y la complejidad de las plantillas de dispositivo, este proceso puede tardar varios minutos en terminar.

    > [!Warning]
    > Se produce un error en la creación de la aplicación de v3 si alguna plantilla de dispositivo tiene campos que comienzan por un número o contienen cualquiera de los siguientes caracteres (`+`, `*`, `?`, `^`, `$`, `(`, `)`, `[`, `]`, `{`, `}`, `|`, `\`). El esquema de la plantilla de dispositivo DTDL que usan las aplicaciones de v3 no permite estos caracteres. Actualice la plantilla de dispositivo para resolver este problema antes de migrar a v3.

1. Cuando la nueva aplicación de v3 esté lista, seleccione **Open your new app** (Abrir nueva aplicación) para abrirla.

    :::image type="content" source="media/howto-migrate/open-app.png" alt-text="Captura de pantalla que muestra el asistente para migración de aplicaciones tras la migración de la aplicación":::

## <a name="configure-the-v3-application"></a>Configuración de la aplicación de la versión 3

Una vez creada la nueva aplicación de v3, realice los cambios de configuración necesarios antes de migrar los dispositivos de la aplicación de v2 a la aplicación de v3.

> [!TIP]
> Dedique un momento a [familiarizarse con la versión 3](overview-iot-central-tour.md#navigate-your-application), ya que tiene algunas diferencias con respecto a la versión anterior.

Estos son algunos de los pasos de configuración recomendados que se deben tener en cuenta:

- [Configuración del panel de la aplicación](howto-add-tiles-to-your-dashboard.md)
- [Configuración de la exportación de datos](howto-export-data.md)
- [Configuración de reglas y acciones](quick-configure-rules.md)
- [Personalización de la interfaz de usuario de la aplicación](howto-customize-ui.md)

Una vez que la aplicación de v3 esté configurada para satisfacer sus necesidades, estará listo para migrar los dispositivos de la aplicación de v2 a la aplicación de v3.

## <a name="move-your-devices-to-the-v3-application"></a>Migración de los dispositivos a la aplicación de la versión 3

Una vez completado este paso, todos los dispositivos se comunican únicamente con la nueva aplicación de v3.

> [!IMPORTANT]
> Antes de migrar los dispositivos a la aplicación de v3, elimine todos aquellos que haya creado en la aplicación de v3.

En este paso se mueven todos los dispositivos existentes a la nueva aplicación de v3. No se copian los datos del dispositivo.

Seleccione **Move all devices** (Migrar todos los dispositivos) para empezar a trasladar los dispositivos. Este paso puede tardar varios minutos en finalizar.

:::image type="content" source="media/howto-migrate/move-devices.png" alt-text="Captura de pantalla que muestra el asistente para migración de aplicaciones con la opción para migrar dispositivos":::

Una vez terminado el traslado, reinicie todos los dispositivos para asegurarse de que se conectan a la nueva aplicación de v3.

> [!WARNING]
> No elimine la aplicación de v3, ya que la de v2 ya no funciona.

## <a name="delete-your-old-v2-application"></a>Eliminación de la antigua aplicación de la versión 2

Una vez que haya validado que todo funciona según lo previsto en la nueva aplicación de v3, elimine la aplicación de v2 antigua. Este paso garantiza que no se le facture por una aplicación que ya no usa.

> [!Note]
> Para eliminar una aplicación, debe tener permisos para eliminar recursos en la suscripción de Azure que eligiera al crear la aplicación. Para obtener más información, vea el artículo sobre el [uso del control de acceso basado en rol para administrar el acceso a los recursos de la suscripción de Azure](../../role-based-access-control/role-assignments-portal.md).

1. En la aplicación de v2, seleccione la pestaña **Administración** en el menú.
2. Seleccione **Eliminar** para quitar de forma permanente la aplicación de IoT Central. Con esta acción se eliminan de manera permanente todos los datos asociados a esa aplicación.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a migrar la aplicación, el siguiente paso sugerido es revisar la [interfaz de usuario de Azure IoT Central](overview-iot-central-tour.md) para ver lo que ha cambiado en v3.