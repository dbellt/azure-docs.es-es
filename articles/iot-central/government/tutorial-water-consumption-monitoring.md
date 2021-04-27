---
title: 'Tutorial: Creación de una aplicación de supervisión del consumo de agua con Azure IoT Central'
description: 'Tutorial: Aprenda a crear una aplicación de supervisión del consumo de agua mediante plantillas de aplicación de Azure IoT Central.'
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: b65013bbf21faa8bffdcf799a991952b69f5fead
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714477"
---
# <a name="tutorial-create-a-water-consumption-monitoring-application-with-azure-iot-central"></a>Tutorial: Creación de una aplicación de supervisión del consumo de agua con Azure IoT Central

Este tutorial le muestra cómo crear una aplicación de supervisión del consumo de agua de Azure IoT Central mediante la plantilla de aplicación de supervisión del consumo de agua de Azure IoT Central.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar la plantilla de supervisión del consumo de agua de Azure IoT Central para crear la aplicación de supervisión del consumo de agua.
> * Explorar y personalizar el panel del operador.
> * Explorar las plantillas de dispositivo.
> * Explorar los dispositivos simulados
> * Explorar y configurar reglas
> * Configurar trabajos
> * Realizar la personalización de marca de la aplicación mediante etiquetado.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

- Suscripción a Azure. Si no tiene una suscripción de Azure, puede crear una en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).

## <a name="create-a-water-consumption-monitoring-app-with-azure-iot-central"></a>Creación de una aplicación de supervisión del consumo de agua con Azure IoT Central

En esta sección, usará la plantilla de supervisión del consumo de agua de Azure IoT Central para crear la aplicación de supervisión del consumo de agua en Azure IoT Central.

Para crear una aplicación de supervisión del consumo de agua de Azure IoT Central, siga estos pasos:

1. Vaya a la [página principal de Azure IoT Central](https://aka.ms/iotcentral).

    Si tiene una suscripción a Azure, inicie sesión con las credenciales que usa para acceder a ella. Si no, inicie sesión con una cuenta de Microsoft.

    ![Incorporación de la cuenta de la organización](media/tutorial-waterconsumptionmonitoring/sign-in.png)

1. Haga clic en **Crear** en el panel izquierdo y seleccione la pestaña **Administración Pública**. En la página **Administración Pública** se muestran varias plantillas de aplicación para la Administración Pública.

   ![Plantillas de creación de aplicaciones de la Administración Pública](./media/tutorial-waterconsumptionmonitoring/iotcentral-government-tab-overview1.png)

1. Seleccione la plantilla de aplicación **Water consumption monitoring**.
Esta plantilla incluye una plantilla de dispositivo de consumo de agua de ejemplo, un dispositivo simulado, un panel del operador y reglas de supervisión preconfiguradas.

1. Haga clic en **Crear aplicación** para abrir el formulario de creación **Nueva aplicación** con los siguientes campos:
    * **Nombre de la aplicación**: De forma predeterminada, la aplicación usa *Water consumption monitoring* seguido de una cadena de identificación única que genera Azure IoT Central. Tiene la opción de elegir un nombre descriptivo para la aplicación. También puede cambiar el nombre de la aplicación más adelante.
    * **URL**: Azure IoT Central genera automáticamente una dirección URL basada en el nombre de la aplicación. Puede optar por actualizar la dirección URL a su gusto. También puede cambiar la dirección URL más adelante.
    * Si tiene una suscripción de Azure, escriba el **Directorio**, la **suscripción de Azure** y la información de **Ubicación**. Si no tiene una suscripción, puede seleccionar la opción **Evaluación gratuita de 7 días** y completar la información de contacto necesaria.

    Para más información sobre los directorios y las suscripciones, consulte la guía de inicio rápido [Creación de una aplicación](../core/quick-deploy-iot-central.md).

1. En la parte inferior de la página, seleccione **Crear**.

    ![Página Nueva aplicación de Azure IoT Central](./media/tutorial-waterconsumptionmonitoring/new-application-waterconsumptionmonitoring.png)

    ![Página de información de facturación de Azure IoT Central](./media/tutorial-waterconsumptionmonitoring/new-application-waterconsumptionmonitoring-billinginfo.png)

Ahora ha creado una aplicación de supervisión del consumo de agua mediante la plantilla de supervisión del consumo de agua de Azure IoT Central.

La aplicación de supervisión del consumo de agua trae los siguientes elementos preconfigurados:

* Paneles del operador de ejemplo.
* Plantillas de dispositivo predefinidas de ejemplo de caudal de agua y válvula.
* Dispositivos simulados de caudal de agua y válvula inteligente.
* Reglas y trabajos.
* Ejemplo de personalización de marca mediante etiquetado.

Como es su aplicación, puede modificarla en cualquier momento. Ahora, explore la aplicación y realice algunas personalizaciones.

## <a name="explore-and-customize-the-operator-dashboard"></a>Exploración y personalización del panel del operador

Después de crear la aplicación, se abre el panel de ejemplo **Wide World water consumption dashboard**.

   ![Panel de supervisión del consumo de agua](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-dashboardfull.png)

Como desarrollador, puede crear y personalizar las vistas del panel para los operadores. Vamos a explorar el panel antes de intentar personalizarlo.

> [!NOTE]
> Todos los datos que se muestran en el panel se basan en los datos del dispositivo simulado, que exploraremos en la sección siguiente.
  
El panel consta de diferentes tipos de iconos:

* **Icono de imagen de Wide World Water Utility**: el primer icono del panel es un icono de imagen de un servicio de agua ficticio, "Wide World Water". Puede personalizar el icono y colocar su propia imagen o quitarlo.
* **Icono Average water flow KPI** (KPI de caudal de agua promedio): el icono de KPI está configurado para mostrar como ejemplo *el caudal medio en los últimos 30 minutos*. Puede personalizar el icono de KPI y establecerlo en un tipo y un intervalo de tiempo diferentes.
* **Iconos de comando de dispositivo**: estos iconos incluyen los iconos **Close valve** (Cerrar válvula), **Open valve** (Abrir válvula) y **Set valve position** (Establecer posición de la válvula). Al seleccionar los comandos, irá a la página de comandos del dispositivo simulado. In Azure IoT Central, un *comando* es un tipo de *funcionalidad del dispositivo*. Exploraremos este concepto más adelante en la sección "Plantilla de dispositivo" de este tutorial.
* **Mapa de área de distribución de agua**: el mapa usa Azure Maps, que puede configurar directamente en Azure IoT Central. El icono del mapa muestra la ubicación del dispositivo. Mantenga el puntero sobre el mapa y pruebe los controles del mapa, como *acercar*, *alejar* o *ampliar*.

    ![Mapa del panel de supervisión del consumo de agua](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-dashboard-map.png)

* **Gráfico de líneas de flujo de agua promedio** y **Gráfico de líneas de condiciones ambientales**: puede visualizar los datos de telemetría de uno o varios dispositivos representados como un gráfico de líneas durante el intervalo de tiempo deseado.
* **Gráfico de mapa térmico de presión de válvula promedio**: puede elegir el tipo de visualización del mapa térmico de los datos de telemetría del dispositivo que desea ver distribuidos en un intervalo de tiempo con un índice de colores.
* **Icono de contenido Restablecer umbrales de alertas**: puede incluir iconos de contenido de llamada a acciones e insertar un vínculo a una página de acciones. En este caso, Restablecer umbrales de alertas le lleva a los **Trabajos** de la aplicación, donde puede ejecutar las actualizaciones de las propiedades del dispositivo. Exploraremos esta opción más adelante en la sección "Configurar trabajos" de este tutorial.
* **Iconos de propiedades**: el panel muestra los iconos **Valve operational info** (Información operativa de válvulas), **Flow alert thresholds** (Umbrales de alerta de caudal) y **Maintenance info** (Información de mantenimiento).

### <a name="customize-the-dashboard"></a>Personalización del panel

Como desarrollador, puede personalizar las vistas del panel para los operadores.

1. Seleccione **Edit** (Editar) para personalizar el panel **Wide World water consumption**. Para personalizar el panel, seleccione el menú **Edit** (Editar). Una vez que el panel esté en modo de **edición**, puede agregar nuevos iconos o configurarlo.

     ![Editar panel](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-edit-dashboard.png)

1. Seleccione **+ Nuevo** para crear un nuevo panel y configurarlo desde el principio. Puede tener varios paneles y puede moverse entre ellos desde el menú del panel.

## <a name="explore-the-device-template"></a>Exploración de la plantilla de dispositivo

Una plantilla de dispositivo de Azure IoT Central define la funcionalidad de un dispositivo, que puede ser telemetría, propiedad o comando. Como desarrollador, puede definir una o varias plantillas de dispositivo en Azure IoT Central que representen la funcionalidad de los dispositivos que se van a conectar.

La aplicación de supervisión del consumo de agua incluye dos plantillas de dispositivo de referencia que representan un *medidor de caudal* y una *válvula inteligente*.

Para ver la plantilla de dispositivo, siga estos pasos:

1. Seleccione **Plantillas de dispositivo** en el panel izquierdo de la aplicación en Azure IoT Central. En la lista **Plantillas de dispositivo**, verá dos plantillas de dispositivo, **Smart Valve** (Válvula inteligente) y **Flow meter** (Medidor de caudal).

   ![Plantilla de dispositivo](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devicetemplate.png)

1. Seleccione la plantilla de dispositivo **Flow meter** (Medidor de caudal) y familiarícese con las funcionalidades del dispositivo.

     ![Plantilla de dispositivo Flow meter (Medidor de caudal)](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devicetemplate-flowmeter.png)

### <a name="customize-the-device-template"></a>Personalización de la plantilla de dispositivo

Para personalizar la plantilla de dispositivo:

1. Vaya a **Personalizar** en el menú **Plantillas de dispositivo**.
1. Busque el tipo de telemetría `Temperature`.
1. Actualice el valor de **Nombre para mostrar** de `Temperature` a `Reported temperature`.
1. Actualice la unidad de medida o establezca los campos **Valor mínimo** y **Valor máximo**.
1. Para guardar los cambios, seleccione **Guardar**.

### <a name="add-a-cloud-property"></a>Adición de una propiedad de la nube

1. Vaya a **Propiedades en la nube** en el menú **Plantillas de dispositivo**.
1. Agregue una nueva propiedad en la nube; para ello, seleccione **+ Agregar propiedad en la nube**.
    En Azure IoT Central, puede agregar una propiedad que sea pertinente para el dispositivo. Por ejemplo, una propiedad en la nube podría ser un umbral de alerta específico de un área de la instalación, información de recursos u otra información de mantenimiento.
1. Para guardar los cambios, seleccione **Guardar**.

### <a name="views"></a>Vistas

La plantilla de dispositivo de supervisión del consumo de agua incluye vistas predefinidas. Puede explorar las vistas y puede hacer actualizaciones. Las vistas definen cómo verán los operadores los datos del dispositivo y también las propiedades en la nube de entrada.

  ![Vistas de la plantilla de dispositivo](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devicetemplate-views.png)

### <a name="publish"></a>Publicar

Si realizó algún cambio, seleccione **Publicar** para publicar la plantilla de dispositivo.

### <a name="create-a-new-device-template"></a>Creación de una nueva plantilla de dispositivo

Seleccione **+ New** (+ Nueva) para crear una plantilla de dispositivo y siga el proceso de creación.
Puede crear una plantilla de dispositivo personalizada desde el principio o elegir una del catálogo de dispositivos de Azure.

## <a name="explore-simulated-devices"></a>Explorar los dispositivos simulados

En Azure IoT Central, puede crear dispositivos simulados para probar la aplicación y la plantilla de dispositivo. La aplicación de supervisión del consumo de agua tiene dos dispositivos simulados asignados a las plantillas de dispositivo **Flow meter** (Medidor de caudal) y **Smart Valve** (Válvula inteligente).

### <a name="view-the-devices"></a>Visualización de los dispositivos

1. Seleccione **Dispositivos** > **Todos los dispositivos** en el panel izquierdo.

   ![Panel Todos los dispositivos](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devices.png)

1. Seleccione **Smart Valve 1** (Válvula inteligente 1).

    ![Válvula inteligente 1](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitor-device1.png)

1. En la pestaña **Comandos**, puede ver los tres comandos de dispositivo, **Close Valve** (Cerrar válvula), **Open Valve** (Abrir válvula) y **Set Valve position** (Establecer posición de la válvula), que son funcionalidades definidas en la plantilla de dispositivo **Smart Valve** (Válvula inteligente).

1. Explore las pestañas **Device Properties** (Propiedades del dispositivo) **Device Dashboard** (Panel del dispositivo).

> [!NOTE]
> Tenga en cuenta que todas las pestañas se han configurado a partir de las vistas de la plantilla de dispositivo.

### <a name="add-new-devices"></a>Adición de nuevos dispositivos

Para agregar nuevos dispositivos, seleccione **(+ Nuevo)** en la pestaña **Dispositivos**.

## <a name="explore-and-configure-rules"></a>Exploración y configuración de reglas

En Azure IoT Central puede crear reglas para supervisar automáticamente los datos de telemetría de los dispositivos y desencadenar acciones cuando se cumplan una o varias condiciones. Las acciones pueden incluir el envío de notificaciones por correo electrónico, el desencadenamiento de una acción de Microsoft Power Automate o una acción de webhook para enviar datos a otros servicios.

La aplicación de supervisión del consumo de agua que ha creado tiene tres reglas preconfiguradas.

### <a name="view-rules"></a>Visualización de reglas

1. En el panel izquierdo, seleccione **Reglas**.

   ![Panel Reglas](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-rules.png)

1. Seleccione **High water flow alert** (Alerta de flujo de agua elevado), que es una de las reglas preconfiguradas en la aplicación.

     ![Alerta de pH alto](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-highflowalert.png)

    La regla `High water flow alert` está configurada para compararse con la condición `Flow` es `greater than``Max flow threshold`. Max flow threshold es una propiedad en la nube definida en la plantilla de dispositivo **Smart Valve** (Válvula inteligente). El valor de `Max flow threshold` se establece por instancia de dispositivo.

Ahora, se creará una acción de correo electrónico.

Para agregar una acción a la regla:

1. Seleccione **+ Email** (+ Correo electrónico).
1. Escriba **High flow alert** (Alerta de flujo elevado) como valor descriptivo de **Display name** (Nombre para mostrar) de la acción.
1. Escriba la dirección de correo electrónico asociada a la cuenta de Azure IoT Central en **To** (Para).
1. Tiene la opción de escribir una nota e incluirla en el texto del correo electrónico.
1. Seleccione **Done** (Listo) para completar la acción.
1. Seleccione **Guardar** para guardar la regla nueva.
1. Habilite la regla.

A los pocos minutos de que se cumpla la condición configurada, recibirá un correo electrónico.

> [!NOTE]
> La aplicación envía un correo electrónico cada vez que se cumple una condición. Seleccione **Deshabilitar** para deshabilitar la regla para dejar de recibir correo electrónico de la regla automatizada.
  
Para crear una regla, siga estos pasos:

* Seleccione **+ Nueva** en la pestaña **Reglas** en el panel izquierdo.

## <a name="configure-jobs"></a>Trabajos de configuración

En Azure IoT Central, los trabajos permiten desencadenar actualizaciones de propiedades en la nube o del dispositivo en varios dispositivos. Además de las propiedades, los trabajos se pueden usar también para desencadenar comandos de dispositivo en varios dispositivos. Azure IoT Central automatiza el flujo de trabajo.

1. Seleccione **Trabajos** en el panel izquierdo.
1. Seleccione **+ Nuevo** y configure uno o varios trabajos.

## <a name="customize-your-application"></a>Personalización de la aplicación

Como creador, puede cambiar varias opciones de configuración para personalizar la experiencia del usuario en la aplicación.

1. Seleccione **Administration** > **Customize your application** (Administración > Personalizar la aplicación).
1. Seleccione el botón **Change** (Cambiar) para elegir una imagen para cargar en el campo **Application logo** (Logotipo de la aplicación).
1. Seleccione el botón **Change** (Cambiar) para elegir una imagen para **Browser icon** (Icono del explorador) que aparecerá en las pestañas del explorador.
1. También puede reemplazar los valores predeterminados de **Browser colors** (Colores del explorador) mediante la adición de códigos de color HTML hexadecimales.

   ![Selecciones para el logotipo de la aplicación, el icono del explorador y los colores del explorador](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-customize-your-application.png)

1. También puede cambiar las imágenes de la aplicación; para ello, seleccione **Administration** > **Application settings** (Administración > Configuración de la aplicación). Seleccione el botón **Select image** (Seleccionar imagen) para elegir una imagen para cargar como la imagen de la aplicación.
1. Por último, también puede cambiar el **Tema**; para ello, seleccione el icono **Settings** (Configuración) en la esquina superior derecha de la aplicación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimínela.

1. Seleccione **Administration** (Administración) en el panel izquierdo de la aplicación de Azure IoT Central.
1. Seleccione **Application settings** (Configuración de la aplicación) y, a continuación, seleccione **Delete** (Eliminar) en la parte inferior de la página.

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de 

> [!div class="nextstepaction"]
> [Conceptos de supervisión del consumo de agua](./concepts-waterconsumptionmonitoring-architecture.md).
