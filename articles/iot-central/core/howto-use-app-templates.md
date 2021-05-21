---
title: Exportación de una aplicación de Azure IoT Central | Microsoft Docs
description: Como administrador de soluciones, quiero exportar una plantilla de aplicación para poder reutilizarla.
author: dominicbetts
ms.author: dobett
ms.date: 12/09/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: a7ad5188e891786ebe1bc1680365a891c256979a
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109683594"
---
# <a name="export-your-application"></a>Exportación de la aplicación

En este artículo se describe cómo exportar una aplicación de IoT Central para poder reutilizarla.

Tiene dos opciones:

- Puede crear una copia de la aplicación si solo necesita crear una copia duplicada de ella.
- Puede crear una plantilla de aplicación a partir la aplicación si tiene previsto crear varias copias.

## <a name="copy-your-application"></a>Copia de la aplicación

Puede crear una copia de cualquier aplicación, menos las instancias de dispositivo, el historial de datos del dispositivo y los datos de usuario. La copia usa un plan de tarifa estándar que se le facturará. No se puede crear una aplicación que use el plan de tarifa gratis mediante la copia de una aplicación.

Seleccione **Copiar**. En el cuadro de diálogo, escriba los detalles de la nueva aplicación. Después, seleccione **Copiar** para confirmar que quiere continuar. Para obtener más información sobre los campos del formulario, consulte [Creación de una aplicación](howto-create-iot-central-application.md).

![Captura de pantalla que muestra la página de configuración "Copiar aplicación".](media/howto-use-app-templates/appcopy2.png)

Una vez que la operación de copia de la aplicación se realiza correctamente, puede navegar a la nueva aplicación mediante el vínculo.

![Página Configuración de la aplicación](media/howto-use-app-templates/appcopy3a.png)

Al copiar una aplicación, también se copia la definición de las reglas y la acción de correo electrónico. Algunas acciones, como Flow y Logic Apps, están asociadas a reglas específicas a través del identificador de regla. Cuando una regla se copia en otra aplicación, obtiene su propio identificador de regla. En este caso, los usuarios tendrán que crear una nueva acción y, a continuación, asociar la nueva regla a ella. En general, es recomendable comprobar las reglas y las acciones para garantizar que están actualizadas en la nueva aplicación.

> [!WARNING]
> Si un panel incluye iconos que muestran información acerca de dispositivos específicos, estos iconos muestran el mensaje **No se encontró el recurso solicitado** en la nueva aplicación. Debe volver a configurar estos iconos para mostrar información acerca de los dispositivos en la nueva aplicación.

## <a name="create-an-application-template"></a>Elija una plantilla de aplicación.

Cuando crea una aplicación de Azure IoT Central, tiene la posibilidad de usar plantillas de ejemplo integradas. También puede crear sus propias plantillas de aplicación a partir de las aplicaciones existentes de IoT Central. A continuación, puede usar sus propias plantillas de aplicación al crear nuevas aplicaciones.

Cuando crea una plantilla de aplicación, esta incluye los siguientes elementos de su aplicación existente:

- El panel de la aplicación predeterminado, incluidos el diseño del panel y todos los iconos que ha definido.
- Las plantillas del dispositivo, incluidas las medidas, la configuración, las propiedades, los comandos y el panel.
- Las reglas. Se incluyen todas las definiciones de las reglas. Sin embargo, no se incluyen las acciones, excepto las de correo electrónico.
- Los conjuntos de dispositivos, incluidas sus condiciones y paneles.

> [!WARNING]
> Si un panel incluye iconos que muestran información acerca de dispositivos específicos, estos iconos muestran el mensaje **No se encontró el recurso solicitado** en la nueva aplicación. Debe volver a configurar estos iconos para mostrar información acerca de los dispositivos en la nueva aplicación.

Cuando crea una plantilla de aplicación, no incluye los siguientes elementos:

- Dispositivos
- Usuarios
- Definiciones de exportaciones de datos continuas

Agregue estos elementos manualmente a cualquier aplicación creada a partir de una plantilla de aplicación.

Para crear una plantilla de aplicación a partir de una aplicación existente de IoT Central:

1. Vaya a la sección **Administración** en su aplicación.
1. Seleccione **Application Template Export** (Exportación de la plantilla de aplicación).
1. En la página **Application Template Export** (Exportación de la plantilla de aplicación), escriba un nombre y una descripción para la plantilla.
1. Seleccione el botón **Exportar** para crear la plantilla de aplicación. Ahora puede copiar el **vínculo que se puede compartir** que permite que otras personas puedan crear una nueva aplicación a partir de la plantilla:

![Elija una plantilla de aplicación.](media/howto-use-app-templates/create-template.png)

### <a name="use-an-application-template"></a>Uso de una plantilla de aplicación

Para usar una plantilla de aplicación para crear una nueva aplicación de IoT Central, necesita un **vínculo que se puede compartir** que haya creado anteriormente. Pegue el **vínculo que se puede compartir** en la barra de direcciones de su explorador. La página **Crear una aplicación**  se muestra con su plantilla de aplicación personalizada seleccionada:

![Crear una aplicación a partir de una plantilla](media/howto-use-app-templates/create-app.png)

Seleccione el plan de tarifa y rellene los demás campos del formulario. A continuación, seleccione **Crear** para crear una nueva aplicación de IoT Central a partir de la plantilla de aplicación.

### <a name="manage-application-templates"></a>Administración de las plantillas de aplicaciones

En la página **Application Template Export** (Exportación de la plantilla de aplicación), puede eliminar o actualizar la plantilla de aplicación.

Si elimina una plantilla de aplicación, ya no podrá usar el vínculo que se puede compartir generado anteriormente para crear nuevas aplicaciones.

Para actualizar la plantilla de aplicación, cambie el nombre o la descripción de la plantilla en la página **Application Template Export** (Exportación de la plantilla de aplicación). A continuación, vuelva a seleccionar el botón **Exportar**. Esta acción genera un nuevo **vínculo que se puede compartir** e invalida cualquier URL anterior de **vínculo que se puede compartir vínculo**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a usar las plantillas de aplicaciones, el siguiente paso recomendado es aprender a [supervisar el estado general de los dispositivos conectados a una aplicación de IoT Central](howto-monitor-application-health.md).
