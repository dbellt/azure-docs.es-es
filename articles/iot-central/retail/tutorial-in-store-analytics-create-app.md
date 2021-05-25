---
title: 'Tutorial: Creación de una aplicación de análisis en tienda en Azure IoT Central'
description: En este tutorial se muestra cómo crear una aplicación de comercio minorista de análisis en tienda en IoT Central. La creará, personalizará y agregará dispositivos sensores.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.custom:
- iot-storeAnalytics-checkout
- iot-p0-scenario
ms.author: timlt
author: timlt
ms.date: 11/12/2019
ms.openlocfilehash: 183693963d66a5df13cbb4dd1e561b26f5495f08
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685700"
---
# <a name="tutorial-create-an-in-store-analytics-application-in-azure-iot-central"></a>Tutorial: Creación de una aplicación de análisis en tienda en Azure IoT Central

En el tutorial se muestra cómo crear una aplicación de análisis en tienda de Azure IoT Central. La aplicación de ejemplo es para una tienda de venta al por menor. Es una solución para la necesidad empresarial común de supervisar y adaptarse a la ocupación y las condiciones ambientales.

La aplicación de ejemplo que se crea incluye tres dispositivos reales: una puerta de enlace Rigado Cascade 500 y dos sensores RuuviTag. En el tutorial también se muestra cómo usar el sensor de ocupación simulado incluido en la plantilla de aplicación con fines de prueba. La puerta de enlace Rigado C500 sirve como centro de comunicación de la aplicación. Se comunica con los sensores de la tienda y administra sus conexiones a la nube. El RuuviTag es un sensor ambiental que proporciona datos de telemetría, como la temperatura, la humedad y la presión. El sensor de ocupación simulado proporciona una forma de realizar un seguimiento del movimiento y la presencia en las zonas de cajas de una tienda. 

En este tutorial se incluyen instrucciones para conectar los dispositivos Rigado y RuuviTag a la aplicación. Si tiene otra puerta de enlace u otros sensores, puede seguir los pasos para crear la aplicación. En el tutorial también se muestra cómo crear sensores RuuviTag simulados. Los sensores simulados le permiten crear la aplicación si no tiene dispositivos reales. 

La solución de supervisión de la zona de cajas y las condiciones se desarrolla en tres partes:

* Creación de la aplicación y conexión de los dispositivos para supervisar las condiciones
* Personalización del panel para permitir que los operadores supervisen y administren los dispositivos
* Configuración de la exportación de datos para permitir que los administradores de la tienda ejecuten el análisis y visualicen información

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Usar la plantilla **In-store analytics - checkout** (Análisis en tienda: finalización de la compra) de Azure IoT Central para crear una aplicación de tienda de venta al por menor
> * Personalizar la configuración de la aplicación
> * Crear y personalizar plantillas de dispositivo IoT
> * Conectar dispositivos a la aplicación
> * Agregar reglas y acciones para supervisar las condiciones

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta serie de tutoriales, necesitará lo siguiente:
* Se recomienda una suscripción de Azure. Opcionalmente, puede usar una evaluación gratuita de 7 días. Si no tiene una suscripción de Azure, puede crear una en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).
* Acceso a un dispositivo de puerta de enlace y dos sensores ambientales (opcionalmente, puede usar dispositivos simulados como se describe en el tutorial).
* Plantillas de dispositivo para los dispositivos que use (se proporcionan plantillas para todos los dispositivos usados en el tutorial).

## <a name="create-an-application"></a>Crear una aplicación
En esta sección, creará una nueva aplicación de Azure IoT Central a partir de una plantilla. Usará esta aplicación a lo largo de la serie de tutoriales para crear una solución completa.

Para crear una nueva aplicación de Azure IoT Central:

1. Vaya al sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral).

1. Si tiene una suscripción a Azure, inicie sesión con las credenciales que usa para acceder a ella; si no la tiene, inicie sesión con una cuenta de Microsoft:

    ![Incorporación de la cuenta de la organización](./media/tutorial-in-store-analytics-create-app/sign-in.png)

1. Para empezar a crear una nueva aplicación de Azure IoT Central, seleccione **New Application** (Nueva aplicación).

1. Seleccione **Retail** (Comercio minorista).  En la página de comercio minorista se muestran varias plantillas de aplicación de venta al por menor.

Para crear una nueva aplicación de análisis en tienda de finalización de la compra:  

1. Seleccione la plantilla de aplicación **In-store analytics - checkout** (Análisis en tienda: finalización de la compra). Esta plantilla incluye plantillas de dispositivo para todos los dispositivos usados en el tutorial, excepto los sensores RuuviTag. La plantilla también proporciona un panel del operador para la supervisión de la zona de cajas, las condiciones ambientales y el estado del dispositivo. 

1. Opcionalmente, elija un valor descriptivo para **Application name** (Nombre de la aplicación). Esta aplicación se basa en una tienda al por menor ficticia llamada Contoso. El tutorial usa el **Nombre de aplicación** *Contoso checkout*. La plantilla de la aplicación se basa en la compañía ficticia Northwind. En este tutorial, usará Contoso para aprender a personalizar la aplicación.

    > [!NOTE]
    > Aunque use un **nombre de aplicación** descriptivo, debe usar un valor único para la dirección **URL** de la aplicación.

1. Si tiene una suscripción de Azure, escriba el *directorio, la suscripción de Azure y la región*. Si no tiene una suscripción, puede activar **7-day free trial** (Evaluación gratuita de 7 días) y completar la información de contacto necesaria.  

1. Seleccione **Crear**.

    ![Página de creación de una aplicación de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/preview-application-template.png)

    ![Página de información de facturación de la creación de una aplicación de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/preview-application-template-billinginfo.png)

## <a name="customize-application-settings"></a>Personalización de la configuración de la aplicación

Como creador, puede cambiar varias opciones de configuración para personalizar la experiencia del usuario en la aplicación. En esta sección, seleccionará un tema de aplicación predefinido. Opcionalmente, aprenderá a crear un tema personalizado y a actualizar la imagen de la aplicación. Un tema personalizado le permite establecer los colores del explorador de la aplicación, el icono del explorador y el logotipo de la aplicación que aparece en la cabecera.

Para seleccionar un tema de aplicación predefinido:

1. Seleccione **Settings** (Configuración) en la cabecera.

    ![Configuración de la aplicación de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/settings-icon.png)

2. Seleccione un nuevo **Theme** (Tema).

3. Seleccione **Guardar**.

En lugar de usar un tema predefinido, puede crear un tema personalizado. Si desea usar un conjunto de imágenes de ejemplo para personalizar la aplicación y completar el tutorial, descargue las [imágenes de ejemplo de Contoso](https://github.com/Azure-Samples/iot-central-docs-samples/tree/master/retail).

Para crear un tema personalizado:

1. Expanda el panel izquierdo, si aún no está expandido.

    ![Panel izquierdo de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/dashboard-expand.png)

1. Seleccione **Administration > Customize your application** (Administración > Personalizar la aplicación).

1. Use el botón **Change** (Cambiar) para elegir una imagen para cargar en el campo **Application logo** (Logotipo de la aplicación). Opcionalmente, especifique un valor para **Logo alt text** (Texto alternativo del logotipo). 

1. Use el botón **Change** (Cambiar) para elegir una imagen para **Browser icon** (Icono del explorador) que aparecerá en las pestañas del explorador.

1. Opcionalmente, reemplace los valores de **Browser colors** (Colores del explorador) mediante la adición de códigos de color HTML en hexadecimal. En **Header** (Encabezado), agregue *#008575*.  En **Accent** (Resaltar), agregue *#A1F3EA*. 

1. Seleccione **Guardar**. 

    ![Logotipo personalizado de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/select-application-logo.png)

    Después de guardar, la aplicación actualiza los colores del explorador, el logotipo de la cabecera y el icono del explorador. 

    ![Configuración de la aplicación actualizada de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/saved-application-settings.png)

Para actualizar la imagen de la aplicación:

1. Seleccione **Administration > Application settings** (Administración > Configuración de la aplicación).

1. Use el botón **Select image** (Seleccionar imagen) para elegir una imagen para cargar como la imagen de la aplicación. Esta imagen aparece en el icono de la aplicación en la página **My Apps** (Mis aplicaciones) del administrador de aplicaciones de IoT Central.

1. Seleccione **Guardar**.

1. Opcionalmente, vaya a la vista **My Apps** (Mis aplicaciones) del sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral). El icono de la aplicación muestra la imagen de la aplicación actualizada.

    ![Imagen de la aplicación personalizada de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/customize-application-image.png)

## <a name="create-device-templates"></a>Creación de plantillas de dispositivo
Como creador, puede crear plantillas de dispositivo que le permitan a usted y a los operadores de la aplicación configurar y administrar los dispositivos. Para crear una plantilla se puede crear una personalizada, importar un archivo de plantilla existente o importar una plantilla desde el catálogo de dispositivos IoT de Azure. Después de crear y personalizar una plantilla de dispositivo, úsela para conectar dispositivos reales a la aplicación. También puede usar una plantilla de dispositivo para generar dispositivos simulados para realizar pruebas.

La plantilla de aplicación **In-store analytics - checkout** (Análisis en tienda: finalización de la compra) tiene plantillas de dispositivo para varios dispositivos.  Hay plantillas de dispositivo para dos de los tres dispositivos que se usan en la aplicación. La plantilla del dispositivo RuuviTag no se incluye en la plantilla de aplicación **In-store analytics - checkout** (Análisis en tienda: finalización de la compra). En esta sección, agregará una plantilla de dispositivo para los sensores RuuviTag a la aplicación.

Para agregar una plantilla de dispositivo RuuviTag a la aplicación:

1. Seleccione **Device Templates** (Plantillas de dispositivo) en el panel izquierdo.

1. Seleccione **+ New** (+ Nuevo) para crear una nueva plantilla de dispositivo.

1. Busque y seleccione la plantilla del dispositivo sensor **RuuviTag** en el catálogo de dispositivos IoT de Azure. 

1. Seleccione **Siguiente: Customize** (Personalizar)

    ![Captura de pantalla que resalta el botón Next: Customize (Siguiente: Personalizar).](./media/tutorial-in-store-analytics-create-app/ruuvitag-device-template.png)

1. Seleccione **Crear**. La aplicación agrega la plantilla del dispositivo RuuviTag.

1. Seleccione **Device templates** (Plantillas de dispositivo) en el panel izquierdo. En la página se muestran todas las plantillas de dispositivo incluidas en la plantilla de aplicación y la plantilla del dispositivo RuuviTag que acaba de agregar.

    ![Plantilla de dispositivo del sensor RuuviTag de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/device-templates-list.png)

## <a name="customize-device-templates"></a>Personalización de plantillas de dispositivo
Puede personalizar las plantillas de dispositivo en la aplicación de tres maneras. En primer lugar, puede personalizar las interfaces nativas integradas en los dispositivos cambiando para ello las funcionalidades del dispositivo. Por ejemplo, con un sensor de temperatura, puede cambiar detalles como el nombre para mostrar de la interfaz de temperatura, el tipo de datos, las unidades de medida y los intervalos de funcionamiento mínimo y máximo. 

En segundo lugar, puede personalizar las plantillas de dispositivo mediante la adición de propiedades en la nube. Las propiedades en la nube no forman parte de las funcionalidades integradas del dispositivo. Las propiedades en la nube son datos personalizados que la aplicación de Azure IoT Central crea, almacena y asocia a los dispositivos. Un ejemplo de una propiedad en la nube podría ser un valor calculado o metadatos, como una ubicación que desea asociar a un conjunto de dispositivos. 

En tercer lugar, puede personalizar las plantillas de dispositivo mediante la creación de vistas personalizadas. Las vistas proporcionan una manera para que los operadores visualicen los datos de telemetría y los metadatos de los dispositivos, como las métricas del dispositivo y el estado.

Aquí se usan los dos primeros métodos para personalizar la plantilla de dispositivo de los sensores RuuviTag. Para obtener información sobre la creación de vistas para los sensores, consulte la guía de inicio rápido [Adición de un dispositivo simulado a la aplicación de IoT Central](../core/quick-create-simulated-device.md).

Para personalizar las interfaces integradas de la plantilla del dispositivo RuuviTag:

1. Seleccione **Device Templates** (Plantillas de dispositivo) en el panel izquierdo. 

1. Seleccione la plantilla de los sensores RuuviTag. 

1. Oculte el panel izquierdo. La vista Resumen de la plantilla muestra las funcionalidades del dispositivo.

    ![Vista Resumen de la plantilla del dispositivo RuuviTag de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/ruuvitag-device-summary-view.png)

1. Seleccione **Customize** (Personalizar) en el menú de la plantilla del dispositivo RuuviTag. 

1. Desplácese por la lista de funcionalidades y busque el tipo de telemetría `humidity`. Es el elemento de fila con el campo **Display name** (Nombre para mostrar) editable para *humidity* (humedad).

En los pasos siguientes, se personaliza el tipo de telemetría `humidity` para los sensores RuuviTag. Opcionalmente, puede personalizar algunos de los demás tipos de telemetría.

En el caso del tipo de telemetría `humidity`, realice los cambios siguientes:

1. Seleccione el control **Expand** (Expandir) para expandir los detalles del esquema de la fila.

1. Actualice el valor de **Display Name** (Nombre para mostrar) de *humedad* a un valor personalizado como *Humedad relativa*.

1. Cambie la opción **Semantic Type** (Tipo de semántica) de *None* (Ninguno) a *Humidity* (Humedad).  Opcionalmente, establezca los valores de esquema para el tipo de telemetría de humedad en la vista de esquema expandida. La configuración de esquema le permite crear requisitos de validación detallados para los datos de los que los sensores realizan un seguimiento. Por ejemplo, puede establecer los valores de intervalo de funcionamiento mínimo y máximo para una interfaz determinada.

1. Haga clic en **Guardar** para guardar los cambios.

    ![Captura de pantalla que muestra la pantalla para personalizar y resalta el botón para guardar.](./media/tutorial-in-store-analytics-create-app/ruuvitag-device-template-customize.png)

Para agregar una propiedad en la nube a una plantilla de dispositivo en la aplicación:

1. Seleccione **Cloud Properties** (Propiedades en la nube) en el menú de la plantilla del dispositivo RuuviTag.

1. Seleccione **Add Cloud Property** (Agregar propiedad en la nube). 

Especifique los siguientes valores para crear una propiedad personalizada para almacenar la ubicación de cada dispositivo:

1. Escriba el valor *Ubicación* en el campo **Display Name** (Nombre para mostrar). Este valor se copia automáticamente en el campo **Name** (Nombre), que es un nombre descriptivo para la propiedad. Puede usar el valor copiado o cambiarlo.

1. Seleccione *String* (Cadena) en el desplegable **Schema** (Esquema). Un tipo de cadena permite asociar una cadena de nombre de ubicación a cualquier dispositivo basado en la plantilla. Por ejemplo, puede asociar un área de una tienda a cada dispositivo. Opcionalmente, puede establecer el campo **Semantic Type** (Tipo de semántica) de la propiedad en *Location* (Ubicación) y se establece automáticamente **Schema** (Esquema) en *Geopoint*. Esto permite asociar coordenadas GPS a un dispositivo. 

1. Establezca **Minimum Length** (Longitud mínima) en *2*. 

1. Establezca **Trim Whitespace** (Recortar espacios en blanco) en **On**.

1. Seleccione **Save** (Guardar) para guardar la propiedad en la nube personalizada.

    ![Personalización de la plantilla del dispositivo RuuviTag de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/ruuvitag-device-template-cloud-property.png)

1. Seleccione **Publicar**. 

    La publicación de una plantilla de dispositivo hace que sea visible para los operadores de la aplicación. Después de publicar una plantilla, úsela para generar dispositivos simulados para realizar pruebas o para conectar dispositivos reales a la aplicación. Si ya tiene dispositivos conectados a la aplicación, al publicar una plantilla personalizada se envían los cambios a los dispositivos.

## <a name="add-devices"></a>Adición de dispositivos
Después de crear y personalizar las plantillas de dispositivo, es el momento de agregar dispositivos. 

En este tutorial, se usa el siguiente conjunto de dispositivos reales y simulados para crear la aplicación:
- Una puerta de enlace Rigado C500 real
- Dos sensores RuuviTag reales
- Un sensor de **Ocupación** simulado. La plantilla de aplicación incluye el sensor simulado, por lo que no es necesario crearlo. 

> [!NOTE]
> Si no tiene dispositivos reales, puede crear sensores RuuviTag simulados para seguir usando este tutorial. Las siguientes instrucciones incluyen los pasos para crear un sensor RuuviTag simulado. No es necesario crear una puerta de enlace simulada.

Complete los pasos de los dos artículos siguientes para conectar una puerta de enlace Rigado real y sensores RuuviTag. Cuando haya terminado, vuelva a este tutorial. Dado que ya ha creado las plantillas de dispositivo en este tutorial, no es necesario crearlas de nuevo en el siguiente conjunto de instrucciones.

- Para conectar una puerta de enlace Rigado, consulte [Conexión de una puerta de enlace Rigado Cascade 500 a la aplicación de Azure IoT Central](../core/howto-connect-rigado-cascade-500.md).
- Para conectar sensores RuuviTag, consulte [Conexión de un sensor RuuviTag a la aplicación de Azure IoT Central](../core/howto-connect-ruuvi.md). También puede usar estas instrucciones para crear dos sensores simulados, si es necesario.

## <a name="add-rules-and-actions"></a>Adición de reglas y acciones
Como parte del uso de sensores en la aplicación de Azure IoT Central para supervisar las condiciones, puede crear reglas para ejecutar acciones cuando se cumplan determinadas condiciones. Una regla está asociada a una plantilla de dispositivo y a uno o varios dispositivos y contiene condiciones que deben cumplirse en función de los eventos o los datos de telemetría del dispositivo. Una regla también tiene una o más acciones asociadas. Las acciones pueden incluir el envío de notificaciones por correo electrónico o el desencadenamiento de una acción de webhook para enviar datos a otros servicios. La plantilla de aplicación **In-store analytics - checkout** (Análisis en tienda: finalización de la compra) incluye algunas reglas predefinidas para los dispositivos de la aplicación.

En esta sección, creará una nueva regla que comprueba el nivel de humedad relativa máximo en función de los datos de telemetría del sensor RuuviTag. Agregará una acción a la regla para que, si la humedad supera el máximo, la aplicación envíe un correo electrónico. 

Para crear una regla: 

1. Expanda el panel izquierdo.

1. Seleccione **Reglas**.

1. Seleccione **+ Nuevo**.

1. Escriba *Nivel de humedad* como el nombre de la regla. 

1. Seleccione la plantilla del dispositivo RuuviTag en **Scopes** (Ámbitos). La regla que defina se aplicará a todos los sensores basados en esa plantilla. Opcionalmente, puede crear un filtro que aplique la regla solo a un subconjunto definido de sensores. 

1. Elija `Relative humidity` para el campo **Telemetry** (Telemetría). Esta es la funcionalidad del dispositivo que ha personalizado en un paso anterior.

1. Elija `Is greater than` para **Operator** (Operador). 

1. Especifique un nivel de humedad interior de un rango superior típico para el entorno como **Value** (Valor). Por ejemplo, escriba *65*. Ha establecido una condición para la regla que se produce cuando la humedad relativa de cualquier sensor RuuviTag real o simulado supera este valor. Es posible que tenga que ajustar el valor hacia arriba o hacia abajo según el intervalo de humedad normal del entorno.  

   ![Agregar condiciones de regla de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/rules-add-conditions.png)

Para agregar una acción a la regla:

1. Seleccione **+ Email** (+ Correo electrónico). 

1. Escriba *Notificación de humedad alta* en el campo **Display name** (Nombre para mostrar) de la acción. 

1. Escriba la dirección de correo electrónico asociada a su cuenta en el campo **To** (Para). Si usa un correo electrónico diferente, la dirección que use debe ser de un usuario que se haya agregado a la aplicación. El usuario también debe iniciar y cerrar sesión al menos una vez.

1. Opcionalmente, escriba una nota para incluirla en el texto del correo electrónico.

1. Seleccione **Done** (Listo) para completar la acción.

   ![Agregar acciones a reglas de Azure IoT Central](./media/tutorial-in-store-analytics-create-app/rules-add-action.png)

1. Seleccione **Save** (Guardar) para guardar y activar la nueva regla. 

    En unos minutos, la cuenta de correo electrónico especificada debe empezar a recibir correos electrónicos. La aplicación envía un correo electrónico cada vez que un sensor indica que el nivel de humedad superó el valor de la condición.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

* Usar la plantilla **In-store analytics - checkout** (Análisis en tienda: finalización de la compra) de Azure IoT Central para crear una aplicación de tienda de venta al por menor
* Personalizar la configuración de la aplicación
* Crear y personalizar plantillas de dispositivo IoT
* Conectar dispositivos a la aplicación
* Agregar reglas y acciones para supervisar las condiciones

Ahora que ha creado una aplicación de supervisión de las condiciones de Azure IoT Central, este es el siguiente paso sugerido:

> [!div class="nextstepaction"]
> [Personalización del panel del operador](./tutorial-in-store-analytics-customize-dashboard.md)
