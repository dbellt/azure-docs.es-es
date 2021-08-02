---
title: Edición de una plantilla de dispositivo en su aplicación de Azure IoT Central | Microsoft Docs
description: Itere en las plantillas de dispositivo sin afectar a los dispositivos conectados en vivo.
author: dominicbetts
ms.author: dobett
ms.date: 04/26/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: fd6fb0cebe33d8511185c396c95faa8927941e6e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110104586"
---
# <a name="edit-an-existing-device-template"></a>Edición de una plantilla de dispositivo existente

*Este artículo se aplica a los generadores de soluciones y a los desarrolladores de dispositivos.*

Una plantilla de dispositivo incluye un modelo que describe el modo en que un dispositivo interactúa con IoT Central. Este modelo define las funcionalidades del dispositivo y la manera en que IoT Central interactúa con ellas. Los dispositivos pueden enviar datos de telemetría y valores de propiedad a IoT Central, IoT Central puede enviar comandos y actualizaciones de propiedades a un dispositivo. IoT Central también usa el modelo para definir interacciones con características de IoT Central como trabajos, reglas y exportaciones.

Los cambios en el modelo de una plantilla de dispositivo pueden afectar a toda la aplicación, incluidos los dispositivos conectados. Los cambios en una funcionalidad que usan las reglas, exportaciones, grupos de dispositivos o trabajos pueden hacer que estos se comporten de forma inesperada o dejen de funcionar. Por ejemplo, si quita una definición de telemetría de una plantilla:

- IoT Central ya no puede interpretar ese valor. IoT Central muestra los datos del dispositivo que no puede interpretar como **Datos no modelados** en la página **Datos sin procesar** del dispositivo.
- IoT Central ya no incluye el valor en ninguna exportación de datos.

Para ayudarle a evitar consecuencias imprevistas al editar una plantilla de dispositivo, en este artículo se incluyen recomendaciones basadas en la fase del ciclo de vida de desarrollo actual. En general, cuanto más temprano se encuentre en el ciclo de vida del desarrollo, más tolerante puede ser a los cambios de la plantilla de dispositivo.

Para más información sobre las plantillas de dispositivo y cómo crear una, consulte [¿Qué son las plantillas de dispositivo?](concepts-device-templates.md) y [Configuración de una plantilla de dispositivo](howto-set-up-template.md).

## <a name="modify-a-device-template"></a>Modificación de una plantilla de dispositivo

Los cambios aditivos, como la adición de una funcionalidad o interfaz a un modelo, son cambios no disruptivos. Puede realizar cambios aditivos en un modelo en cualquier fase del ciclo de vida de desarrollo.

Los cambios importantes incluyen la eliminación de partes de un modelo o el cambio de un nombre de funcionalidad o un tipo de esquema. Estos cambios podrían hacer que las características de la aplicación, como reglas, exportaciones o paneles, muestren mensajes de error y dejen de funcionar.

En las primeras fases del desarrollo de dispositivos, mientras sigue diseñando y probando el modelo, hay mayor tolerancia para realizar cambios directamente en el modelo de dispositivo. Antes de conectar dispositivos de producción a una plantilla de dispositivo, puede editarla directamente. IoT Central aplica estos cambios automáticamente a los dispositivos al publicar la plantilla de dispositivo.

Después de asociar dispositivos de producción a una plantilla de dispositivo, evalúe el impacto de los cambios antes de editar una plantilla de dispositivo. No debe realizar cambios importantes en una plantilla de dispositivo en producción. Para realizar estos cambios, cree una nueva versión de la plantilla de dispositivo. Pruebe la nueva plantilla de dispositivo y, a continuación, migre los dispositivos de producción a la nueva plantilla en un tiempo de inactividad programado.

## <a name="update-an-iot-edge-device-template"></a>Actualización de una plantilla de dispositivo de IoT Edge

Las plantillas de dispositivo de IoT Edge contienen un _manifiesto de implementación_ además del modelo de dispositivo. Para un dispositivo IoT Edge, el modelo agrupa las funcionalidades por módulos que corresponden a los módulos de IoT Edge que se ejecutan en el dispositivo. El manifiesto de implementación es un documento JSON independiente que indica al dispositivo IoT Edge qué módulos instalar y cómo configurarlos. Las orientaciones de la sección anterior se aplican también a los módulos del modelo de dispositivo. Además, todos los módulos definidos en el modelo de dispositivo deben incluirse en el manifiesto de implementación. Una vez publicada una plantilla de dispositivo de IoT Edge, debe crear una nueva versión si necesita reemplazar el manifiesto de implementación. Para que los dispositivos de IoT Edge reciban el nuevo manifiesto de implementación, migre a la nueva versión de la plantilla.

Para obtener más información, consulte [Manifiestos de implementación de IoT Edge y plantillas de dispositivos de IoT Central](concepts-iot-edge.md#iot-edge-deployment-manifests-and-iot-central-device-templates).

### <a name="edit-and-publish-actions"></a>Edición y publicación de acciones

Las siguientes acciones son útiles al editar una plantilla de dispositivo:

- Seleccione _Guardar_. Cuando cambia parte de la plantilla de dispositivo, al guardar los cambios se crea un borrador al que puede volver. Estos cambios aún no afectan a los dispositivos conectados. Los dispositivos creados a partir de esta plantilla no tendrán los cambios guardados hasta que la publique.
- _Publicación_. Al publicar la plantilla de dispositivo, se aplican los cambios guardados a las instancias de dispositivo existentes. Las instancias de dispositivo recién creadas siempre usan la plantilla publicada más reciente.
- _Control de versiones de una plantilla_. Cuando se genera una nue versión de una plantilla de dispositivo, se crea una plantilla con todos los cambios guardados más recientes. Las instancias de dispositivo existentes no se verán afectadas por los cambios realizados en una nueva versión. Para obtener más información, consulte [Control de versiones de una plantilla de dispositivo](#version-a-device-template).
- _Control de las versiones de una interfaz_. Al crear una versión de una interfaz, se crea una nueva interfaz con todas las funcionalidades guardadas más recientes. Puede reutilizar una interfaz en varias ubicaciones dentro de una plantilla. Este es el motivo por el que un cambio realizado en una referencia a una interfaz cambia todos los lugares de la plantilla que usan la interfaz. Cuando se controlan las versiones de una interfaz, este comportamiento cambia porque la nueva versión es ahora una interfaz independiente. Para más información, consulte [Control de las versiones de una interfaz](#version-an-interface).
- _Migración de un dispositivo_. Al migrar un dispositivo, la instancia de dispositivo cambia de una plantilla de dispositivo a otra. La migración de los dispositivos puede tardar un poco mientras IoT Central procesa los cambios. Para más información, consulte [Migración de un dispositivo entre versiones](#migrate-a-device-across-versions).

### <a name="version-numbers"></a>Números de versión

Ambos modelos de dispositivo e interfaces tienen números de versión. Los distintos números de versión permiten que los modelos o interfaces compartan un valor `@id`, al tiempo que proporcionan un historial de actualizaciones. Los números de versión solo se incrementan si decide controlar las versiones de la plantilla o la interfaz, o si cambia deliberadamente el número de versión. Debe cambiar un número de versión al realizar un cambio importante en una plantilla o interfaz.

En el fragmento de código siguiente se muestra el modelo de dispositivo de un dispositivo termostato. El modelo de dispositivo tiene una sola interfaz. Puede ver el número de versión, `1`, al final del campo `@id`.

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;1",
  "@type": "Interface",
  "displayName": "Thermostat",
  "description": "Reports current temperature and provides desired temperature control.",
  "contents": [
    // ...
  ]
}
```

Para ver esta información en la interfaz de usuario de IoT Central, seleccione **View identity** (Ver identidad) en el editor de plantillas de dispositivo:

:::image type="content" source="media/howto-edit-device-template/view-identity.png" alt-text="Visualización de la identidad de una interfaz para ver el número de versión":::

## <a name="version-a-device-template"></a>Versión de una plantilla de dispositivo

Para controlar la versión de una plantilla de dispositivo:

1. Vaya a la página **Plantillas de dispositivo**.
1. Seleccione la plantilla de dispositivo cuya versión desea controlar.
1. Seleccione **Versión** en la parte superior de la página y asigne un nuevo nombre a la plantilla. IoT Central sugiere un nombre nuevo, que se puede editar.
1. Seleccione **Crear**.

Ahora ha creado una plantilla con una identidad única que no está asociada a ningún dispositivo existente.

## <a name="version-an-interface"></a>Control de las versiones de una interfaz

Para controlar las versiones de una interfaz:

1. Vaya a la página **Plantillas de dispositivo**.
1. Seleccione la plantilla de dispositivo que tiene en modo borrador.
1. Seleccione la interfaz publicada cuya versión quiere controlar y modificar.
1. Seleccione **Versión** en la parte superior de la página de la interfaz.
1. Seleccione **Crear**.

Ahora ha creado una nueva interfaz con una identidad única, no sincronizada con la versión anterior de la interfaz.

## <a name="migrate-a-device-across-versions"></a>Migración de un dispositivo entre versiones

Puede crear varias versiones de la plantilla de dispositivo. Con el tiempo, tendrá varios dispositivos conectados mediante estas plantillas de dispositivo. Puede migrar los dispositivos de una versión de la plantilla de dispositivo a otra. Los pasos siguientes describen cómo migrar un dispositivo:

1. Vaya a la página **Dispositivos**.
1. Seleccione el dispositivo que necesita migrar a otra versión.
1. Elija **Migrate** (Migrar):

    :::image type="content" source="media/howto-edit-device-template/migrate-device.png" alt-text="Selección de la opción para iniciar la migración de un dispositivo":::

1. Seleccione la plantilla de dispositivo con la versión a la que quiere migrar el dispositivo y elija **Migrar**.

## <a name="next-steps"></a>Pasos siguientes

Si es operador o generador de soluciones, el siguiente paso sugerido es aprender [cómo administrar los dispositivos](./howto-manage-devices.md).

Si es desarrollador de dispositivos, el siguiente paso sugerido es leer acerca de los [dispositivos de Azure IoT Edge y Azure IoT Central](./concepts-iot-edge.md).
