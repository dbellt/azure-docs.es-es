---
title: Uso de flujos de trabajo para integrar la aplicación de Azure IoT Central con otros servicios en la nube | Microsoft Docs
description: En este artículo de procedimientos se muestra cómo puede, como generador, configurar las reglas y las acciones que integran la aplicación de IoT Central con otros servicios en la nube. Para crear una regla avanzada, puede usar un conector de IoT Central en Power Automate o en Azure Logic Apps.
author: dominicbetts
ms.author: dobett
ms.date: 05/12/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: af6adcf408c9790b09830e986cd1cac00774a2ec
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491940"
---
# <a name="use-workflows-to-integrate-your-azure-iot-central-application-with-other-cloud-services"></a>Uso de flujos de trabajo para integrar la aplicación de Azure IoT Central con otros servicios en la nube

*Este artículo se aplica a los generadores de soluciones.*

Puede crear reglas en IoT Central que desencadenen acciones, como el envío de un correo electrónico, en respuesta a condiciones basadas en datos de telemetría, como la temperatura de un dispositivo que supera un umbral.

El conector Azure IoT Central V3 para Power Automate y Azure Logic Apps permite crear reglas más avanzadas para automatizar operaciones en IoT Central:

- Cuando se activa una regla en la aplicación de Azure IoT Central, puede desencadenar un flujo de trabajo en Power Automate o en Azure Logic Apps. Estos flujos de trabajo pueden ejecutar acciones en otros servicios en la nube, como Microsoft 365, o en un servicio de terceros.
- Un evento en otro servicio en la nube, como Microsoft 365, puede desencadenar un flujo de trabajo en Power Automate o en Azure Logic Apps. Estos flujos de trabajo pueden ejecutar acciones o recuperar datos de la aplicación de IoT Central.

## <a name="prerequisites"></a>Requisitos previos

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

La configuración de la solución requiere una aplicación de la versión 3 de IoT Central. Para obtener información sobre cómo comprobar la versión de la aplicación, vea [Acerca de la aplicación](./howto-get-app-info.md). Para obtener información sobre cómo crear una aplicación de IoT Central, vea [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md).

> [!NOTE]
> Si usa una aplicación de la versión 2 de IoT Central, vea [Creación de flujos de trabajo con el conector de IoT Central en Azure Logic Apps](/previous-versions/azure/iot-central/core/howto-build-azure-logic-apps) en el sitio de documentación de las versiones anteriores y use el conector Azure IoT Central V2

## <a name="trigger-a-workflow-from-a-rule"></a>Desencadenamiento de un flujo de trabajo desde una regla

Para poder desencadenar un flujo de trabajo en Power Automate o en Azure Logic Apps, necesita una regla en la aplicación de IoT Central. Para más información, consulte [Configuración de reglas y acciones en Azure IoT Central](./howto-configure-rules.md).

Para agregar el conector **Azure IoT Central V3 (Versión preliminar)** como desencadenador en Power Automate:

1. En Power Automate, seleccione **+ Crear** y la pestaña **Personalizado**.
1. Busque *IoT Central* y seleccione el conector **Azure IoT Central V3 (Versión preliminar)** .
1. En la lista de desencadenadores, seleccione **When a rule is fired (preview)** (Cuando se desencadene una regla [versión preliminar]).
1. En el paso **When a rule is fired (preview)** (Cuando se desencadene una regla [versión preliminar]), seleccione la aplicación de IoT Central y la regla que esté usando.

Para agregar el conector **Azure IoT Central V3 (Versión preliminar)** como desencadenador en Azure Logic Apps:

1. En el **diseñador de Logic Apps**, seleccione la plantilla **Aplicación lógica en blanco**.
1. En el diseñador, seleccione la pestaña **Personalizado**.
1. Busque *IoT Central* y seleccione el conector **Azure IoT Central V3 (Versión preliminar)** .
1. En la lista de desencadenadores, seleccione **When a rule is fired (preview)** (Cuando se desencadene una regla [versión preliminar]).
1. En el paso **When a rule is fired (preview)** (Cuando se desencadene una regla [versión preliminar]), seleccione la aplicación de IoT Central y la regla que esté usando.

:::image type="content" source="./media/howto-configure-rules-advanced/triggers.png" alt-text="Busque el conector Azure IoT Central - preview (Azure IoT Central [Versión preliminar]) y elija el desencadenador":::

Ahora puede agregar más pasos al flujo de trabajo para crear el escenario de integración.

## <a name="run-an-action"></a>Ejecución de una acción

Puede ejecutar acciones en una aplicación de IoT Central desde flujos de trabajo de Power Automate y Azure Logic Apps. En primer lugar, cree el flujo de trabajo y use un conector para definir el desencadenador que iniciará el flujo de trabajo. Después, use el conector **Azure IoT Central V3 (Versión preliminar)** como una acción.

Para agregar el conector **Azure IoT Central V3 (Versión preliminar)** como una acción en Power Automate:

1. En Power Automate, en el panel **Elegir una acción**, seleccione la pestaña **Personalizado**.
1. Busque *IoT Central* y seleccione el conector **Azure IoT Central V3 (Versión preliminar)** .
1. En la lista de acciones, seleccione la acción de IoT Central que desee usar.
1. En el paso de la acción, complete la configuración de la acción que haya elegido. Después, seleccione **Guardar**.

Para agregar el conector **Azure IoT Central V3 (Versión preliminar)** como una acción en Azure Logic Apps:

1. En el **diseñador de Logic Apps**, en el panel **Elegir una acción**, seleccione la pestaña **Personalizado**.
1. Busque *IoT Central* y seleccione el conector **Azure IoT Central V3 (Versión preliminar)** .
1. En la lista de acciones, seleccione la acción de IoT Central que desee usar.
1. En el paso de la acción, complete la configuración de la acción que haya elegido. Después, seleccione **Guardar**.

:::image type="content" source="./media/howto-configure-rules-advanced/actions.png" alt-text="Búsqueda del conector Azure IoT Central V3 (Versión preliminar) y selección de una acción":::

## <a name="list-of-actions"></a>Lista de acciones

En la lista siguiente se muestran todas las acciones de IoT Central disponibles en el conector **Azure IoT Central V3 (Versión preliminar)** y sus opciones de configuración. Muchos de los campos pueden tener contenido generado dinámicamente. Por ejemplo, un paso anterior podría haber determinado el identificador del dispositivo en el que actúa el paso actual.

### <a name="create-or-update-a-device"></a>Create or update a device (Crear o actualizar un dispositivo)

Use esta acción para crear o actualizar un dispositivo en la aplicación de IoT Central.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a crear o actualizar. |
| Aprobado | Elija si el dispositivo se ha aprobado para conectarse a IoT Central. |
| Device Description (Descripción del dispositivo) | Descripción detallada del dispositivo. |
| Device Name (Nombre del dispositivo) | Nombre para mostrar del dispositivo. |
| Plantilla de dispositivo | Elija en la lista de plantillas de dispositivo de la aplicación de IoT Central. |
| Simulated (Simulado) | Elija si el dispositivo es simulado. |

### <a name="delete-a-device"></a>Eliminar un dispositivo

Use esta acción para eliminar un dispositivo de la aplicación de IoT Central.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |

### <a name="execute-a-device-command"></a>Execute a device command (Ejecutar un comando de dispositivo)

Use esta acción para ejecutar un comando definido en una de las interfaces del dispositivo.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |
| Device Component (Componente de dispositivo) | La interfaz de la plantilla de dispositivo que contiene el comando. |
| Comando de dispositivo | Elija uno de los comandos de la interfaz seleccionada. |
| Plantilla de dispositivo | Elija en la lista de plantillas de dispositivo de la aplicación de IoT Central. |
| Device Command Request Payload (Carga de solicitud de comando de dispositivo) | Si el comando requiere una carga de solicitud, agréguela aquí.  |

> [!NOTE]
> No puede elegir un componente de dispositivo mientras no haya elegido una plantilla de dispositivo.

### <a name="get-a-device-by-id"></a>Get a device by ID (Obtener un dispositivo por identificador)

Use esta acción para recuperar los detalles del dispositivo.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |

Puede usar los detalles que devuelven las expresiones dinámicas en otras acciones. Los detalles del dispositivo que se devuelven incluyen: **Approved** (Aprobado), **body** (cuerpo), **Device Description** (Descripción del dispositivo), **Device Name** (Nombre del dispositivo), **Device Template** (Plantilla de dispositivo), **Provisioned** (Aprovisionado) y **Simulated** (Simulado).

### <a name="get-device-cloud-properties"></a>Get device cloud properties (Obtener propiedades en la nube del dispositivo)

Use esta acción para recuperar los valores de propiedad en la nube de un dispositivo específico.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |
| Plantilla de dispositivo | Elija en la lista de plantillas de dispositivo de la aplicación de IoT Central. |

Puede usar los valores de propiedad en la nube que devuelven las expresiones dinámicas en otras acciones.

### <a name="get-device-properties"></a>Get device properties (Obtener propiedades del dispositivo)

Use esta acción para recuperar los valores de propiedad de un dispositivo específico.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |
| Plantilla de dispositivo | Elija en la lista de plantillas de dispositivo de la aplicación de IoT Central. |

Puede usar los valores de propiedad que devuelven las expresiones dinámicas en otras acciones.

### <a name="get-device-telemetry-value"></a>Get device telemetry value (Obtener valor de telemetría del dispositivo)

Use esta acción para recuperar los valores de telemetría de un dispositivo específico.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |
| Plantilla de dispositivo | Elija en la lista de plantillas de dispositivo de la aplicación de IoT Central. |

Puede usar los valores de telemetría que devuelven las expresiones dinámicas en otras acciones.

### <a name="update-device-cloud-properties"></a>Update device cloud properties (Actualizar propiedades en la nube del dispositivo)

Use esta acción para actualizar los valores de propiedad en la nube de un dispositivo específico.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |
| Plantilla de dispositivo | Elija en la lista de plantillas de dispositivo de la aplicación de IoT Central. |
| Propiedades de la nube | Después de elegir una plantilla de dispositivo, se agrega un campo por cada propiedad en la nube definida en la plantilla. |

### <a name="update-device-properties"></a>Update device properties (Actualizar propiedades del dispositivo)

Use esta acción para actualizar los valores de propiedad grabables de un dispositivo específico.

| Campo | Descripción |
| ----- | ----------- |
| Application | Elija en la lista de aplicaciones de IoT Central. |
| Dispositivo | Identificador único del dispositivo que se va a eliminar. |
| Plantilla de dispositivo | Elija en la lista de plantillas de dispositivo de la aplicación de IoT Central. |
| Propiedades editables | Después de elegir una plantilla de dispositivo, se agrega un campo por cada propiedad grabable definida en la plantilla. |

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a crear una regla avanzada en la aplicación de Azure IoT Central, puede aprender a [analizar los datos de los dispositivos en la aplicación de Azure IoT Central](howto-create-analytics.md).