---
title: Definición de un nuevo tipo de dispositivo IoT en Azure IoT Central | Microsoft Docs
description: En este artículo se muestra cómo crear una plantilla de dispositivo Azure IoT en la aplicación de Azure IoT Central. Se definen la telemetría, el estado, las propiedades y los comandos del tipo.
author: dominicbetts
ms.author: dobett
ms.date: 12/06/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom:
- contperf-fy21q1
- device-developer
ms.openlocfilehash: 8d51297b353a72861f864e42bf87cad4f1a712b3
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108747204"
---
# <a name="define-a-new-iot-device-type-in-your-azure-iot-central-application"></a>Definición de un nuevo tipo de dispositivo IoT en la aplicación de Azure IoT Central

Una plantilla de dispositivo es un plano técnico que define las características y los comportamientos de un tipo de dispositivo que se conecta a una [aplicación de Azure IoT Central](concepts-app-templates.md).

Por ejemplo, un generador puede crear una plantilla de dispositivo para un ventilador conectado que tenga las siguientes características:

- Envía datos de telemetría de temperatura.
- Envía la propiedad de ubicación.
- Envía eventos de error del motor del ventilador.
- Envía el estado de funcionamiento del ventilador.
- Proporciona una propiedad de velocidad del ventilador que se puede modificar
- Proporciona un comando para reiniciar el dispositivo
- Le ofrece una vista global del dispositivo mediante una vista

A partir de esta plantilla de dispositivo, un operador puede crear y conectar dispositivos de ventilador reales. Todos estos ventiladores tienen medidas, propiedades y comandos que los operadores utilizan para supervisar y administrar. Los operadores usan las [vistas de dispositivos](#add-views) y los formularios para interactuar con los dispositivos de ventilación. Un desarrollador de dispositivos usa la plantilla para comprender cómo interactúa el dispositivo con la aplicación. Para más información, consulte [Cargas de telemetría, propiedades y comandos](concepts-telemetry-properties-commands.md).

> [!NOTE]
> Solo los generadores y administradores pueden crear, editar y eliminar plantillas de dispositivo. Cualquier usuario puede crear dispositivos en la página **Devices** (Dispositivos) a partir de las plantillas de dispositivo existentes.

En las aplicaciones de IoT Central, las plantillas de dispositivo usan modelos de dispositivo para describir las funcionalidades de los dispositivos. Como generador, tiene varias opciones para crear plantillas de dispositivo:

- Diseñe la plantilla de dispositivo en IoT Central e [implemente su modelo de dispositivo en el código del dispositivo](concepts-telemetry-properties-commands.md).
- Importe una plantilla de dispositivo del [catálogo de dispositivos Azure Certified for IoT](https://aka.ms/iotdevcat). Personalice la plantilla de dispositivos según sus requisitos en IoT Central.
> [!NOTE]
> IoT Central requiere el modelo completo con todas las interfaces a las que se hace referencia en el mismo archivo; al importar un modelo desde el repositorio de modelos, use la palabra clave "expanded" para obtener la versión completa.
Por ejemplo. https://devicemodels.azure.com/dtmi/com/example/thermostat-1.expanded.json

- Cree un modelo de dispositivo, para lo que debe usar el [lenguaje de definición de Digital Twins (DTDL), versión 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). Visual Studio Code tiene una extensión que admite la creación de modelos DTDL. Para obtener más información, consulte [Instalación y uso de las herramientas de creación de DTDL](../../iot-pnp/howto-use-dtdl-authoring-tools.md). Luego, publique el modelo en el repositorio de modelos público. Para más información, consulte [Repositorio de modelos de dispositivo](../../iot-pnp/concepts-model-repository.md). Implemente el código del dispositivo desde el modelo y conecte el dispositivo real a la aplicación de IoT Central. IoT Central busca e importa automáticamente el modelo del dispositivo desde el repositorio público y genera una plantilla de dispositivo. Después, puede agregar las propiedades, las personalizaciones y las vistas de la nube que la aplicación de IoT Central necesita a la plantilla del dispositivo.
- Cree un modelo de dispositivo mediante DTDL. Implemente el código del dispositivo a partir del modelo. Importe manualmente el modelo de dispositivo en la aplicación de IoT Central y agregue las propiedades, las personalizaciones y las vistas de la nube que la aplicación de IoT Central necesite.

> [!TIP]
> IoT Central requiere el modelo completo con todas las interfaces a las que se hace referencia en el mismo archivo. Al importar un modelo desde el repositorio de modelos, use la palabra clave *expanded* para obtener la versión completa.
> Por ejemplo, [https://devicemodels.azure.com/dtmi/com/example/thermostat-1.expanded.json](https://devicemodels.azure.com/dtmi/com/example/thermostat-1.expanded.json).

También puede agregar plantillas de dispositivo a una aplicación de IoT Central mediante la [API de REST](/learn/modules/manage-iot-central-apps-with-rest-api/) o la [CLI](howto-manage-iot-central-from-cli.md).

Algunas [plantillas de aplicación](concepts-app-templates.md) ya incluyen plantillas de dispositivo que son útiles en el escenario que admite la plantilla de aplicación. Por ejemplo, consulte [Arquitectura de análisis en tienda](../retail/store-analytics-architecture.md).

## <a name="create-a-device-template-from-the-device-catalog"></a>Creación de una plantilla de dispositivo desde el catálogo de dispositivos

Como desarrollador, puede empezar a crear rápidamente la solución mediante un dispositivo certificado. Consulte la lista en el [catálogo de dispositivos IoT de Azure](https://devicecatalog.azure.com). IoT Central se integra con el catálogo de dispositivos para que pueda importar un modelo de dispositivo desde cualquiera de los dispositivos certificados. Para crear una plantilla de dispositivo desde uno de estos dispositivos en IoT Central:

1. Vaya a la página **Device Templates** (Plantillas de dispositivo) de la aplicación de IoT Central.
1. Seleccione **+ Nuevo** y, después, seleccione cualquiera de los dispositivos certificados del catálogo. IoT Central crea una plantilla de dispositivo basada en este modelo de dispositivo.
1. Agregue las propiedades, las personalizaciones o las vistas en la nube a la plantilla de dispositivo.
1. Seleccione **Publicar** para que la plantilla esté disponible para que los operadores vean y conecten los dispositivos.

## <a name="create-a-device-template-from-scratch"></a>Creación de una plantilla de dispositivo desde cero

Una plantilla de dispositivo contiene:

- Un _modelo de dispositivo_ que especifica la telemetría, las propiedades y los comandos que implementa el dispositivo. Estas funcionalidades están organizadas en uno o varios componentes.
- _Propiedades en la nube_ que definen la información que almacena la aplicación de IoT Central acerca de los dispositivos. Por ejemplo, una propiedad en la nube podría registrar la fecha en la que se realizó la última revisión de un dispositivo. Esta información nunca se comparte con el dispositivo.
- Las _personalizaciones_ dejan que el generador reemplace algunas de las definiciones del modelo de dispositivo. Por ejemplo, el desarrollador puede reemplazar el nombre de una propiedad del dispositivo. Los nombres de propiedad aparecen en las vistas y formularios de IoT Central.
- Las _vistas y formularios_ permiten al generador crear una interfaz de usuario para que los operadores supervisen y administren los dispositivos conectados a la aplicación.

Para crear una plantilla de dispositivo en IoT Central:

1. Vaya a la página **Device Templates** (Plantillas de dispositivo) en la aplicación de IoT Central.
1. Seleccione **+ Nuevo** > **Dispositivo IoT**. Después, seleccione **Next: Customize** (Personalizar)
1. Escriba un nombre para la plantilla, como **Termostato**. Después, seleccione **Next: Examine**  y seleccione **Crear**.
1. IoT Central crea una plantilla de dispositivo vacía y le permite elegir entre crear un modelo personalizado desde cero o importar un modelo de DTDL.

## <a name="manage-a-device-template"></a>Administración de una plantilla de dispositivo

Puede cambiar el nombre o eliminar una plantilla desde la página principal de la plantilla.

Después de agregar un modelo de dispositivo a la plantilla, puede publicarlo. No se puede conectar un dispositivo basado en esta plantilla para que los operadores puedan verlo en la página **Dispositivos** hasta que se haya publicado la plantilla.

## <a name="create-a-capability-model"></a>Creación de un modelo de funcionalidad

Para crear un modelo de dispositivo, puede:

- Usar IoT Central para crear un modelo personalizado desde cero.
- Importar un modelo de DTDL desde un archivo JSON. Un desarrollador de dispositivos puede haber usado Visual Studio Code para crear un modelo de dispositivo para la aplicación.
- Seleccione uno de los dispositivos en el catálogo de dispositivos. Esta opción importa el modelo de dispositivo que el fabricante ha publicado para este dispositivo. Un modelo de dispositivo importado como este se publica automáticamente.

## <a name="manage-a-capability-model"></a>Administración de un modelo de funcionalidad

Después de crear un modelo de dispositivo, puede:

- Agregar componentes al modelo. Todos los modelos deben tener al menos un componente.
- Editar los metadatos del modelo, como el identificador, el espacio de nombres y el nombre.
- Eliminar el modelo.

## <a name="create-a-component"></a>Crear un componente

Todos los modelos de dispositivo deben tener al menos un componente predeterminado. Un componente es una colección reutilizable de funcionalidades.

Para crear un componente:

1. Vaya a su modelo de dispositivo y elija **+ Agregar componente**.

1. En la página **Add a component interface** (Agregar interfaz de componente), puede:

    - Crear un componente personalizado desde cero.
    - Importar un componente existente de un archivo DTDL. Un desarrollador de dispositivos puede haber usado Visual Studio Code para crear una interfaz de componente para el dispositivo.

1. Después de crear un componente, elija **Edit Identity** (Editar identidad) para cambiar el nombre para mostrar del componente.

1. Si decide crear un componente personalizado desde cero, puede agregar las funcionalidades del dispositivo. Las funcionalidades del dispositivo son la telemetría, las propiedades y los comandos.

### <a name="telemetry"></a>Telemetría

La telemetría es un flujo de valores enviados desde el dispositivo, normalmente desde un sensor. Por ejemplo, un sensor podría informar de la temperatura ambiente.

En la siguiente tabla se muestran las opciones de configuración de una funcionalidad de telemetría:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | Nombre para mostrar del valor de telemetría que se usa en las vistas y formularios. |
| Nombre | El nombre del campo en el mensaje de telemetría. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico. |
| Capability Type (Tipo de funcionalidad) | Telemetría. |
| Semantic Type (Tipo semántico) | El tipo semántico de la telemetría, como la temperatura, el estado o el evento. La elección del tipo semántico determina cuál de los campos siguientes está disponible. |
| Schema | El tipo de datos de telemetría, como doble, cadena o vector. Las opciones disponibles vienen determinadas por el tipo semántico. El esquema no está disponible para los tipos semánticos de evento y estado. |
| severity | Solo está disponible para el tipo semántico de evento. Los niveles de gravedad son **Error**, **Información** o **Advertencia**. |
| State Values (Valores de estado) | Solo está disponible para el tipo semántico de estado. Defina los valores de estado posibles, cada uno de los cuales tiene el nombre para mostrar, el nombre, el tipo de enumeración y el valor. |
| Unidad | Una unidad para el valor de telemetría, como **mph**, **%** o **&deg;C**. |
| Display Unit (Unidad de visualización) | Unidad de visualización para usarse en vistas y formularios. |
| Comentario | Cualquier comentario sobre la funcionalidad de telemetría. |
| Descripción | Una descripción de la funcionalidad de telemetría. |

### <a name="properties"></a>Propiedades

Las propiedades representan valores de un momento dado. Por ejemplo, un dispositivo puede usar una propiedad para notificar la temperatura objetivo que está intentando alcanzar. Puede establecer las propiedades que se pueden editar desde IoT Central.

En la siguiente tabla se muestran las opciones de configuración de una funcionalidad de propiedad:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | Nombre para mostrar del valor de propiedad que se usa en las vistas y formularios. |
| Nombre | El nombre de la propiedad. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico. |
| Capability Type (Tipo de funcionalidad) | Propiedad. |
| Semantic Type (Tipo semántico) | El tipo semántico de la propiedad, como la temperatura, el estado o el evento. La elección del tipo semántico determina cuál de los campos siguientes está disponible. |
| Schema | El tipo de datos de la propiedad, como doble, cadena o vector. Las opciones disponibles vienen determinadas por el tipo semántico. El esquema no está disponible para los tipos semánticos de evento y estado. |
| Editable | Si la propiedad no se puede modificar, el dispositivo puede notificar los valores de la propiedad a IoT Central. Si la propiedad se puede modificar, el dispositivo puede notificar los valores de la propiedad a IoT Central y este puede enviar actualizaciones de propiedades al dispositivo.
| severity | Solo está disponible para el tipo semántico de evento. Los niveles de gravedad son **Error**, **Información** o **Advertencia**. |
| State Values (Valores de estado) | Solo está disponible para el tipo semántico de estado. Defina los valores de estado posibles, cada uno de los cuales tiene el nombre para mostrar, el nombre, el tipo de enumeración y el valor. |
| Unidad | Una unidad para el valor de propiedad, como **mph**, **%** o **&deg;C**. |
| Display Unit (Unidad de visualización) | Unidad de visualización para usarse en vistas y formularios. |
| Comentario | Cualquier comentario sobre la funcionalidad de propiedad. |
| Descripción | Una descripción de la funcionalidad de propiedad. |

### <a name="commands"></a>Comandos:

Puede llamar a los comandos de dispositivo desde IoT Central. Los comandos, opcionalmente, pasan parámetros al dispositivo y reciben una respuesta del dispositivo. Por ejemplo, puede llamar a un comando para que reinicie un dispositivo en 10 segundos.

En la tabla siguiente se muestran las opciones de configuración de una funcionalidad de comando:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | Nombre para mostrar del comando que se usa en las vistas y formularios. |
| Nombre | El nombre del comando. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico. |
| Capability Type (Tipo de funcionalidad) | Comando. |
| Comentario | Cualquier comentario sobre la funcionalidad del comando. |
| Descripción | Una descripción de la funcionalidad del comando: |
| Solicitud | Si está habilitada, una definición del parámetro de solicitud que incluye lo siguiente: nombre, nombre para mostrar, esquema, unidad y unidad de visualización. |
| Response | Si está habilitada, una definición de la respuesta del comando que incluye lo siguiente: nombre, nombre para mostrar, esquema, unidad y unidad de visualización. |

Para más información sobre la forma en que los dispositivos implementan comandos, consulte [Cargas de telemetría, propiedades y comandos > Comandos y comandos de larga duración](concepts-telemetry-properties-commands.md#commands).

#### <a name="offline-commands"></a>Comandos sin conexión

Puede elegir comandos de cola si un dispositivo está sin conexión; para ello, habilite la opción **Queue if offline** (Poner en cola si no está conectado) en un comando de la plantilla de dispositivo.

Esta opción usa los mensajes de la nube al dispositivo de IoT Hub para enviar notificaciones a los dispositivos. Para obtener más información, consulte el artículo de IoT Hub [Envío de mensajes de la nube al dispositivo](../../iot-hub/iot-hub-devguide-messages-c2d.md).

Los mensajes de la nube al dispositivo:

- Son notificaciones unidireccionales al dispositivo de la solución.
- Ofrecen la garantía de que el mensaje se entrega al menos una vez. IoT Hub conserva los mensajes de la nube al dispositivo en las colas por dispositivo, garantizando así la resistencia frente a errores de conectividad y de dispositivo.
- Requieren que el dispositivo implemente un controlador de mensajes para procesar el mensaje de la nube al dispositivo.

> [!NOTE]
> Esta opción solo está disponible en la interfaz de usuario web de IoT Central. Esta configuración no se incluye si exporta un modelo o un componente desde la plantilla de dispositivo.

## <a name="manage-a-component"></a>Administración de componentes

Si no ha publicado el componente, podrá editar las funcionalidades que define este. Después de publicar el componente, si desea realizar cualquier cambio, deberá crear una nueva versión de la plantilla del dispositivo y [controlar la versión](howto-version-device-template.md) del componente. Los cambios que no requieren el control de versiones, como nombres para mostrar o unidades, se pueden realizar en la sección **Personalizar**.

También puede exportar el componente como un archivo JSON si desea reutilizarlo en otro modelo de funcionalidad.

## <a name="add-cloud-properties"></a>Adición de propiedades de nube

Use las propiedades en la nube para almacenar información acerca de los dispositivos en IoT Central. Las propiedades en la nube no se envían nunca a un dispositivo. Por ejemplo, puede usar las propiedades en la nube para almacenar el nombre del cliente que ha instalado el dispositivo o la fecha del último servicio del dispositivo.

En la tabla siguiente se muestran las opciones de configuración de una propiedad en la nube:

| Campo | Descripción |
| ----- | ----------- |
| Display Name (Nombre para mostrar) | Nombre para mostrar del valor de propiedad en la nube que se usa en las vistas y formularios. |
| Nombre | El nombre de la propiedad en la nube. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. |
| Semantic Type (Tipo semántico) | El tipo semántico de la propiedad, como la temperatura, el estado o el evento. La elección del tipo semántico determina cuál de los campos siguientes está disponible. |
| Schema | El tipo de datos de la propiedad en la nube, como doble, cadena o vector. Las opciones disponibles vienen determinadas por el tipo semántico. |

## <a name="add-customizations"></a>Adición de personalizaciones

Use las personalizaciones cuando necesite modificar un componente importado o agregar características específicas de IoT Central a una funcionalidad. Y solo se pueden personalizar aquellos que no interrumpan la compatibilidad del componente. Por ejemplo, puede:

- Personalizar el nombre para mostrar y las unidades de una funcionalidad.
- Agregar un color predeterminado para usarlo cuando el valor aparezca en un gráfico.
- Especificar los valores inicial, mínimo y máximo de una propiedad.

No se puede personalizar el tipo o el nombre de la funcionalidad. Si hay cambios que no se pueden realizar en la sección **Customize** (Personalizar), deberá crear una versión de la plantilla de dispositivo y el componente para modificar la funcionalidad.

### <a name="generate-default-views"></a>Generación de vistas predeterminadas

La generación de vistas predeterminadas es una forma rápida de visualizar la información importante del dispositivo. Puede generar hasta tres vistas predeterminadas para la plantilla de dispositivo:

- **Comandos**: una vista con los comandos del dispositivo que permite que el operador los envíe al dispositivo.
- **Información general**: una vista con los datos de telemetría del dispositivo que muestra gráficos y métricas.
- **Acerca de**: una vista con información del dispositivo que muestra sus propiedades.

Una vez que haya seleccionado **Generar vistas predeterminadas**, verá que se han agregado automáticamente en la sección **Vistas** de la plantilla del dispositivo.

## <a name="add-views"></a>Adición de vistas

Agregue vistas a una plantilla de dispositivo para permitir que los operadores visualicen un dispositivo mediante gráficos y métricas. Puede tener varias vistas para una plantilla de dispositivo.

Para agregar una vista a una plantilla de dispositivo:

1. Vaya a la plantilla del dispositivo y seleccione **Vistas**.
1. Elija **Visualización del dispositivo**.
1. Escriba un nombre para la vista en **Nombre de vista**.
1. Agregue iconos a la vista en la lista de iconos estáticos, de propiedades, de propiedades en la nube, de telemetría y de comandos. Arrastre y coloque los iconos que quiera agregar a la vista.
1. Para trazar varios valores de telemetría en un único icono de gráfico, seleccione los valores de telemetría y, a continuación, seleccione **Combinar**.
1. Configure cada icono que agregue para personalizar cómo muestra los datos. Para acceder a esta opción, seleccione el icono de engranaje o **Cambiar configuración** en el icono del gráfico.
1. Organice y cambie el tamaño de los iconos en la vista.
1. Guarde los cambios.

### <a name="configure-preview-device-to-view"></a>Configuración del dispositivo de versión preliminar para ver la vista

Para ver y probar la vista, seleccione **Configure preview device** (Configurar la vista previa de dispositivo). Esta característica le permite ver la vista del mismo modo que el operador una vez publicada. Use esta característica para asegurarse de que las vistas muestran los datos correctos. Puede elegir entre las siguientes opciones:

- Ningún dispositivo de versión preliminar.
- El dispositivo de prueba real que ha configurado para la plantilla de dispositivo.
- Un dispositivo existente en la aplicación mediante el uso del identificador de dispositivo.

## <a name="add-forms"></a>Adición de formularios

Agregue formularios a una plantilla de dispositivo para permitir a los operadores administrar un dispositivo con la visualización y configuración de propiedades. Los operadores solo pueden editar las propiedades en la nube y las propiedades del dispositivo que se pueden modificar. Puede tener varios formularios para una plantilla de dispositivo.

Para agregar un formulario a una plantilla de dispositivo:

1. Vaya a la plantilla del dispositivo y seleccione **Vistas**.
1. Seleccione **Editar datos del dispositivo y la nube**.
1. Escriba un nombre para el formulario en **Form Name** (Nombre de formulario).
1. Seleccione el número de columnas que se usará para diseñar el formulario.
1. Agregue propiedades a una sección existente del formulario o seleccione las propiedades y elija **Add section** (Agregar sección). Use las secciones para agrupar las propiedades del formulario. Puede agregar un título a una sección.
1. Configure cada propiedad en el formulario para personalizar su comportamiento.
1. Organice las propiedades en el formulario.
1. Guarde los cambios.

## <a name="publish-a-device-template"></a>Publicación de una plantilla de dispositivo

Antes de poder conectar un dispositivo que implementa el modelo de dispositivo, debe publicar la plantilla del dispositivo.

Después de publicar una plantilla de dispositivo, solo puede realizar cambios limitados en el modelo de dispositivo. Para modificar un componente, debe [crear y publicar una nueva versión](./howto-version-device-template.md).

Para publicar una plantilla de dispositivo, vaya a su plantilla de dispositivo y seleccione **Publicar**.

Después de publicar una plantilla de dispositivo, el operador puede ir a la página **Dispositivos** y agregar dispositivos reales o simulados que usan la plantilla de dispositivo. Puede seguir modificando y guardando la plantilla de dispositivo mientras realiza los cambios. Si desea enviar estos cambios al operador para que los vea en la página **Dispositivos**, debe seleccionar **Publicar** cada vez.

## <a name="next-steps"></a>Pasos siguientes

Un próximo paso sugerido es leer sobre [control de versiones de plantilla de dispositivo](./howto-version-device-template.md).
