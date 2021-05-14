---
title: 'Tutorial: Creación de una aplicación de administración de residuos conectada con Azure IoT Central'
description: 'Tutorial: Aprenda a crear una aplicación de administración de residuos conectada con las plantillas de aplicación de Azure IoT Central.'
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 42a82a59483a582e0d395092558665b12e1cd5a3
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109684368"
---
# <a name="tutorial-create-a-connected-waste-management-app"></a>Tutorial: Creación de una aplicación de administración de residuos conectada

En este tutorial se muestra cómo usar Azure IoT Central para crear una aplicación de administración de residuos conectada. 

Específicamente, aprenderá sobre lo siguiente: 

> [!div class="checklist"]
> * Usar la plantilla *Administración de residuos conectada* de Azure IoT Central para crear la aplicación.
> * Explorar y personalizar el panel del operador. 
> * Explorar la plantilla de dispositivo de cubo de basura conectado.
> * Explorar los dispositivos simulados
> * Explorar y configurar reglas
> * Configurar trabajos
> * Personalización de marca de la aplicación.

## <a name="prerequisites"></a>Prerrequisitos

Se recomienda una suscripción de Azure. Como alternativa, puede usar una versión de prueba gratuita de 7 días. Si no tiene una suscripción de Azure, puede crear una en la [página de suscripción a Azure](https://aka.ms/createazuresubscription).

## <a name="create-your-app-in-azure-iot-central"></a>Creación de la aplicación en Azure IoT Central

En esta sección, va a usar la plantilla Administración de residuos conectada para crear la aplicación en Azure IoT Central. A continuación, se indica cómo puede hacerlo.

1. Vaya a [Azure IoT Central](https://aka.ms/iotcentral).

    Si tiene una suscripción a Azure, inicie sesión con las credenciales que usa para acceder a ella. De lo contrario, inicie sesión mediante una cuenta Microsoft:

    ![Captura de pantalla del inicio de sesión de Microsoft.](./media/tutorial-connectedwastemanagement/sign-in.png)

1. En el panel izquierdo, seleccione **Build** (Compilar). A continuación, seleccione la pestaña **Government** (Administración Pública). En la página Government (Administración Pública) se muestran varias plantillas de aplicación.

    ![Captura de pantalla de la página de compilación de Azure IoT Central.](./media/tutorial-connectedwastemanagement/iotcentral-government-tab-overview.png)

1. Seleccione la plantilla de aplicación **Connected Waste Management** (Administración de residuos conectada). Esta plantilla incluye una plantilla de dispositivo de cubo de basura conectado de ejemplo, un dispositivo simulado, un panel del operador y reglas de supervisión preconfiguradas.    

1. Seleccione **Create app** (Crear aplicación), lo que abre el cuadro de diálogo **New application** (Nueva aplicación). Rellene la información de los campos siguientes:
    * **Application name** (Nombre de la aplicación). De forma predeterminada, la aplicación usa **Connected waste management** (Administración de residuos conectada) seguido de una cadena de identificador única que genera Azure IoT Central. Tiene la opción de elegir un nombre descriptivo para la aplicación. También puede cambiar el nombre de la aplicación más adelante.
    * **Dirección URL**. Opcionalmente, puede elegir la dirección URL deseada. Puede cambiar la dirección URL más adelante. 
    * **Pricing plan** (Plan de precios). Si tiene una suscripción de Azure, escriba el directorio, la suscripción de Azure y la región en los campos correspondientes del cuadro de diálogo **Billing info** (Información de facturación). Si no tiene una suscripción, seleccione **Free** (Gratis) para habilitar la suscripción de prueba de 7 días y complete la información de contacto necesaria.  

1. En la parte inferior de la página, seleccione **Create** (Crear). 

    ![Captura de pantalla del cuadro de diálogo Create New application (Crear nueva aplicación) de Azure IoT Central.](./media/tutorial-connectedwastemanagement/new-application-connectedwastemanagement.png)
    
    ![Captura de pantalla del cuadro de diálogo Billing info (Información de facturación) de Azure IoT Central.](./media/tutorial-connectedwastemanagement/new-application-connectedwastemanagement-billinginfo.png)

 
La aplicación recién creada incluye la siguiente configuración:
* Paneles del operador de ejemplo.
* Plantillas de dispositivo de ejemplo predefinidas de cubo de basura conectado.
* Dispositivos simulados de cubo de basura conectado.
* Reglas y trabajos.
* Personalización de marca de ejemplo. 

Como es su aplicación, puede modificarla en cualquier momento. Ahora, explore la aplicación y realice algunas personalizaciones.  

## <a name="explore-and-customize-the-operator-dashboard"></a>Exploración y personalización del panel del operador 

Eche un vistazo al **Panel de administración de residuos Wide World**, que verá después de crear la aplicación.

   ![Captura de pantalla del panel de administración de residuos Wide World.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-dashboard1.png)

Como desarrollador, puede crear y personalizar las vistas del panel para los operadores. En primer lugar, vamos a explorar el panel. 

>[!NOTE]
>Todos los datos que se muestran en el panel se basan en los datos del dispositivo simulado, que se verán con más detalle en la sección siguiente. 

El panel consta de diferentes iconos:

* **Icono de imagen del servicio Wide World Water**: el primer icono del panel es un icono de imagen de un servicio de agua ficticio, "Wide World Water". Puede personalizar el icono y colocar su propia imagen o eliminarlo. 

* **Icono de imagen de cubo de basura**: puede usar iconos de imagen y contenido para crear una representación visual del dispositivo que se va a supervisar, junto con un texto descriptivo. 

* **Icono de indicador KPI de nivel de llenado**: este icono muestra el valor devuelto por un sensor de *nivel de llenado* de un cubo de basura. Fill level (Nivel de llenado) y otros sensores como *Odor meter* (Medidor de olor) o *Weight* (Peso) de un cubo de basura se pueden supervisar de forma remota. El operador puede tomar medidas, como enviar un camión de recogida de basuras. 

* **Mapa del área de supervisión de residuos**: este icono usa Azure Maps y se puede configurar directamente en Azure IoT Central. El icono del mapa muestra la ubicación del dispositivo. Intente mantener el puntero sobre el mapa y pruebe los controles: acercar, alejar o ampliar.

     ![Captura de pantalla del mapa del panel de la plantilla de administración de residuos conectada.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-dashboard-map.png)


* **Gráfico de barras del nivel de llenado, olor y peso**: puede visualizar uno o varios tipos de datos de telemetría de los dispositivos en un gráfico de barras. También puede expandir este gráfico.  

  ![Captura de pantalla del gráfico de barras del panel de la plantilla de administración de residuos conectada.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-dashboard-barchart.png)


* **Icono de contenido de Field Services**: el panel incluye un vínculo a cómo realizar la integración con los servicios de campo de Dynamics 365 desde la aplicación de Azure IoT Central. Por ejemplo, puede usar Field Services para crear vales para enviar los servicios de recogida de basura. 


### <a name="customize-the-dashboard"></a>Personalización del panel 

Para personalizar el panel, seleccione el menú **Edit** (Editar). A continuación, puede agregar nuevos iconos o configurar los existentes. Este es el aspecto del panel en el modo de edición: 

![Captura de pantalla del panel de la plantilla de administración de residuos conectada en el modo de edición.](./media/tutorial-connectedwastemanagement/edit-dashboard.png)

También puede seleccionar **+ New** (+ Nuevo) para crear un panel y configurarlo desde el principio. Puede tener varios paneles y cambiar entre ellos desde el menú del panel. 

## <a name="explore-the-device-template"></a>Exploración de la plantilla de dispositivo

Una plantilla de dispositivo de Azure IoT Central define las funcionalidades de un dispositivo, entre ellas, la telemetría, las propiedades o los comandos. Como desarrollador, puede definir plantillas de dispositivo que representen la funcionalidad de los dispositivos que se van a conectar. 

La aplicación Administración de residuos conectada incluye una plantilla de dispositivo de cubo de basura conectado de ejemplo.

Para ver la plantilla de dispositivo, siga estos pasos:

1. Seleccione **Device templates** (Plantillas de dispositivo) en el panel izquierdo de la aplicación en Azure IoT Central. 

    ![Captura de pantalla que muestra la lista de plantillas de dispositivo de la aplicación.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devicetemplate.png)

1. En la lista **Device templates** (Plantillas de dispositivo), seleccione **Connected Waste Bin** (Cubo de basura conectado).

1. Examine las funcionalidades de la plantilla de dispositivo. Puede ver que se definen sensores como **Fill level** (Nivel de llenado), **Odor meter** (Medidor de olor), **Weight** (Peso) y **Location** (Ubicación).

   ![Captura de pantalla que muestra los detalles de la plantilla de dispositivo del cubo de basura conectado.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devicetemplate-connectedbin.png)


### <a name="customize-the-device-template"></a>Personalización de la plantilla de dispositivo

Pruebe a personalizar lo siguiente:
1. En el menú de la plantilla de dispositivo, seleccione **Customize** (Personalizar).
1. Busque el tipo de telemetría **Odor meter** (Medidor de olor).
1. Actualice el campo **Display name** (Nombre para mostrar) de **Odor meter** (Medidor de olor) a **Odor level** (Nivel de olor).
1. Pruebe a actualizar la unidad de medida o establezca los campos **Min value** (Valor mínimo) y **Max value** (Valor máximo).
1. Seleccione **Guardar**. 

### <a name="add-a-cloud-property"></a>Adición de una propiedad de la nube 

A continuación, se indica cómo puede hacerlo.
1. En el menú de la plantilla de dispositivo, seleccione **Cloud property** (Propiedad de la nube).
1. Seleccione **+ Add Cloud Property** (+ Agregar propiedad de la nube). En Azure IoT Central, puede agregar a un dispositivo una propiedad pertinente, aunque no se espera que el dispositivo la envíe. Por ejemplo, una propiedad de la nube podría ser un umbral de alerta específico del área de instalación, información de recursos o información de mantenimiento. 
1. Seleccione **Guardar**. 
 
### <a name="views"></a>Vistas 
La plantilla de dispositivo de cubo de basura conectado incluye vistas predefinidas. Explore las vistas y actualícelas si lo desea. Las vistas definen cómo verán los operadores los datos del dispositivo y las propiedades de la nube de entrada. 

  ![Captura de pantalla de las vistas de plantillas de dispositivo de la plantilla de administración de residuos conectada.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devicetemplate-views.png)

### <a name="publish"></a>Publicar 

Si ha realizado algún cambio, recuerde publicar la plantilla de dispositivo. 

### <a name="create-a-new-device-template"></a>Creación de una nueva plantilla de dispositivo 

Para crear una nueva plantilla de dispositivo, seleccione **+ New** (+ Nueva) y siga los pasos. Puede crear una plantilla de dispositivo personalizada desde el principio o elegir una del catálogo de dispositivos de Azure. 

## <a name="explore-simulated-devices"></a>Explorar los dispositivos simulados

En Azure IoT Central, puede crear dispositivos simulados para probar la aplicación y la plantilla de dispositivo. 

La aplicación Administración de residuos conectada tiene dos dispositivos simulados asociados a la plantilla de dispositivo de cubo de basura conectado. 

### <a name="view-the-devices"></a>Visualización de los dispositivos

1. En el panel izquierdo de Azure IoT Central, seleccione **Device** (Dispositivo). 

   ![Captura de pantalla de los dispositivos de la plantilla de administración de residuos conectada.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devices.png)

1. Seleccione el dispositivo **Connected Waste Bin** (Cubo de basura conectado).  

     ![Captura de pantalla de las propiedades del dispositivo de la plantilla de administración de residuos conectada.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devices-bin1.png)

1. Vaya a la pestaña **Cloud Properties** (Propiedades de la nube). Actualice el valor de **Bin full alert threshold** (Umbral de alerta de cubo lleno) de **95** a **100**. 

Explore las pestañas **Device Properties** (Propiedades del dispositivo) **Device Dashboard** (Panel del dispositivo). 

> [!NOTE]
> Todas las pestañas se han configurado a partir de las vistas de la plantilla de dispositivo.

### <a name="add-new-devices"></a>Adición de nuevos dispositivos

Para agregar nuevos dispositivos, seleccione **+ New** (+ Nuevo) en la pestaña **Devices** (Dispositivos). 

## <a name="explore-and-configure-rules"></a>Exploración y configuración de reglas

En Azure IoT Central, puede crear reglas para supervisar automáticamente los datos de telemetría de los dispositivos y desencadenar acciones cuando se cumplan una o varias condiciones. Las acciones pueden incluir el envío de notificaciones por correo electrónico, el desencadenamiento de una acción de Power Automate o iniciar una acción de webhook para enviar datos a otros servicios.

La aplicación Administración de residuos conectada tiene cuatro reglas de ejemplo.

### <a name="view-rules"></a>Visualización de reglas
1. En el panel izquierdo de Azure IoT Central, seleccione **Rules** (Reglas).

   ![Captura de pantalla de las reglas de la plantilla de administración de residuos conectada.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-rules.png)

1. Seleccione **Bin full alert** (Alerta de cubo lleno).

     ![Captura de pantalla de la alerta de cubo lleno.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-binfullalert.png)

 1. **Bin full alert** (Alerta de cubo lleno) comprueba la condición siguiente: **Fill level is greater than or equal to Bin full alert threshold** (Nivel de llenado es mayor o igual que Umbral de alerta de cubo lleno).

    **Bin full alert threshold** (Umbral de alerta de cubo lleno) es una propiedad de la nube que se define en la plantilla del dispositivo cubo de basura conectado. 

Ahora, se creará una acción de correo electrónico.

### <a name="create-an-email-action"></a>Creación de una acción de correo electrónico

Puede configurar una acción de correo electrónico en la lista **Actions** (Acciones) de la regla:
1. Seleccione **+ Email** (+ Correo electrónico). 
1. En el campo **Display name** (Nombre para mostrar), escriba **High pH alert** (Alerta de pH alto).
1. Escriba la dirección de correo electrónico asociada a la cuenta de Azure IoT Central en el campo **To** (Para). 
1. Tiene la opción de escribir una nota e incluirla en el texto del correo electrónico.
1. Seleccione **Done** > **Save** (Listo > Guardar). 

Ahora recibirá un correo electrónico cuando se cumpla la condición configurada.

>[!NOTE]
>La aplicación envía un correo electrónico cada vez que se cumple una condición. Deshabilite la regla para dejar de recibir correo electrónico de la regla automatizada. 
  
Para crear una nueva regla, en el panel izquierdo de **Rules** (Reglas), seleccione **+New** (+Nuevo).

## <a name="configure-jobs"></a>Trabajos de configuración

En Azure IoT Central, los trabajos permiten desencadenar actualizaciones de propiedades de la nube o del dispositivo en varios dispositivos. También puede usar trabajos para desencadenar comandos de dispositivo en varios dispositivos. Azure IoT Central automatiza el flujo de trabajo. 

1. En el panel izquierdo de Azure IoT Central, seleccione **Jobs** (Trabajos). 
1. Seleccione **+New** (+Nuevo) y configure uno o varios trabajos. 

## <a name="customize-your-application"></a>Personalización de la aplicación 

Como creador, puede cambiar varias opciones de configuración para personalizar la experiencia del usuario en la aplicación.

### <a name="change-the-application-theme"></a>Cambio del tema de la aplicación

A continuación, se indica cómo puede hacerlo.
1. Vaya a **Administration** > **Customize your application** (Administración > Personalizar la aplicación).
1. Seleccione **Change** (Cambiar) para elegir una imagen para cargar en el campo **Application logo** (Logotipo de la aplicación).
1. Seleccione **Change** (Cambiar) para elegir una imagen para cargar en el **icono del explorador** (una imagen que aparecerá en las pestañas del explorador).
1. También puede reemplazar los colores predeterminados del explorador mediante la adición de códigos de color HTML hexadecimales. Use los campos **Header** (Encabezado) y **Accent** (Color de énfasis) para este fin.

   ![Captura de pantalla de la personalización de la aplicación de la plantilla de administración de residuos conectada.](./media/tutorial-connectedwastemanagement/connectedwastemanagement-customize-your-application.png)

1. También puede cambiar las imágenes de la aplicación. Seleccione **Administration** > **Application settings** > **Select image** (Administración > Configuración de la aplicación > Seleccionar imagen) para elegir la imagen que se va a cargar como imagen de la aplicación.
1. Por último, también puede cambiar el tema; para ello, seleccione **Settings** (Configuración) en la cabecera de la aplicación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminarla mediante estos pasos:

1. Seleccione **Administration** (Administración) en el panel izquierdo de la aplicación de Azure IoT Central.
1. Seleccione **Application settings** > **Delete** (Configuración de la aplicación > Eliminar).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conceptos de administración de residuos conectados](./concepts-connectedwastemanagement-architecture.md)
