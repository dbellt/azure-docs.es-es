---
title: 'Tutorial: Personalización del panel del operador en Azure IoT Central'
description: En este tutorial se muestra cómo personalizar el panel del operador en una aplicación de IoT Central y cómo administrar dispositivos.
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
ms.openlocfilehash: cea5ed5d75bbba4e2e6cdc8793bf5ce7503a5912
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108755358"
---
# <a name="tutorial--customize-the-operator-dashboard-and-manage-devices-in-azure-iot-central"></a>Tutorial:  Personalización del panel del operador y administración de dispositivos en Azure IoT Central


En este tutorial obtendrá información sobre cómo personalizar el panel del operador en una aplicación de análisis en la tienda de Azure IoT Central. Los operadores de la aplicación pueden usar el panel personalizado para ejecutar la aplicación y administrar los dispositivos conectados.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Cambiar el nombre del panel
> * Personalizar iconos de imagen en el panel
> * Organizar iconos para modificar el diseño
> * Agregar iconos de telemetría para mostrar las condiciones
> * Agregar iconos de propiedades para mostrar detalles del dispositivo
> * Agregar iconos de comandos para ejecutar comandos

## <a name="prerequisites"></a>Prerrequisitos

El desarrollador debe realizar el primer tutorial para crear la aplicación de análisis en tienda de Azure IoT Central y agregar dispositivos:

* [Creación de una aplicación de análisis en tienda en Azure IoT Central](./tutorial-in-store-analytics-create-app.md) (requerido)

## <a name="change-the-dashboard-name"></a>Cambiar el nombre del panel
Para personalizar el panel del operador, tiene que editar el panel predeterminado en la aplicación. También puede crear nuevos paneles adicionales. El primer paso para personalizar el panel en la aplicación es cambiar el nombre.

1. Vaya al sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral).

1. Abra la aplicación de supervisión de condiciones que creó en el tutorial [Creación de una aplicación de análisis en tienda en Azure IoT Central](./tutorial-in-store-analytics-create-app.md).

1. Seleccione **Edit** (Edición) en la barra de herramientas del panel. En el modo de edición, puede personalizar la apariencia, el diseño y el contenido del panel.

    ![Edición del panel de Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/dashboard-edit.png)

1. Si lo desea, puede ocultar el panel izquierdo. Al hacerlo, se ampliará el área de trabajo para editar el panel.

1. Escriba un nombre descriptivo para el panel en **Dashboard name** (Nombre del panel). En este tutorial se utiliza una compañía ficticia llamada Contoso y el nombre del panel de ejemplo es *Contoso dashboard*. 

1. Seleccione **Guardar**. Los cambios se guardan en el panel y el modo de edición está deshabilitado.

    ![Cambio de nombre del panel de Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/dashboard-change-name.png)

## <a name="customize-image-tiles-on-the-dashboard"></a>Personalizar iconos de imagen en el panel
Un panel de aplicación de Azure IoT Central consta de uno o varios iconos. Un icono es un contenedor rectangular para mostrar contenido en un panel. Es posible asociar varios tipos de contenido con los iconos y arrastrar, colocar y cambiar el tamaño de los iconos para personalizar el diseño de un panel. Existen varios tipos de iconos para mostrar contenido. Los iconos de imagen contienen imágenes y es posible agregar una dirección URL que permita a los usuarios hacer clic en la imagen. Los iconos de etiqueta muestran texto sin formato. Los iconos de Markdown contienen contenido con formato y permiten establecer una imagen, una dirección URL, un título y un código Markdown que se representa como HTML. Los iconos de telemetría, propiedades o comandos muestran datos específicos del dispositivo. 

En esta sección, aprenderá a personalizar los iconos de imagen en el panel.

Para personalizar un icono de imagen que muestra una imagen de marca en el panel:

1. Seleccione **Edit** (Edición) en la barra de herramientas del panel. 

1. Seleccione **Configure** (Configurar) en el icono de imagen que muestra la imagen de la marca Northwind. 

    ![Edición de imagen de marca en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/brand-image-edit.png)

1. Cambie el título en **Title** (Título). El título aparece cuando un usuario mantiene el puntero sobre la imagen.

1. Seleccione **Image** (Imagen). Se abrirá un cuadro de diálogo que le permitirá cargar una imagen personalizada. 

1. Opcionalmente, especifique una dirección URL para la imagen.

1. Seleccione **Update configuration** (Actualización de la configuración). El botón **Update configuration** (Actualización de la configuración) guarda los cambios en el panel y deja habilitado el modo de edición.

    ![Guardar imagen de marca en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/brand-image-save.png)

1. Opcionalmente, seleccione **Configure** (Configurar) en el icono titulado **Documentation** (Documentación) y especifique una dirección URL para contenido de soporte técnico. 

Para personalizar un icono de imagen que muestra un mapa de las zonas de sensores en el almacén:

1. Seleccione **Configure** (Configurar) en el icono de imagen que muestra el mapa de las zonas de la tienda predeterminado. 

1. Seleccione **Image** (Imagen) y use el cuadro de diálogo para cargar una imagen personalizada de un mapa de zonas de la tienda. 

1. Seleccione **Update configuration** (Actualización de la configuración).

    ![Guardar mapa de zonas en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/store-map-save.png)

    En el mapa de la tienda de Contoso de ejemplo se muestran cuatro zonas: dos zonas de cajas, una zona para prendas y productos de cuidado personal y otra para comestibles. En este tutorial, asociará sensores con estas zonas para proporcionar datos de telemetría.

    ![Zonas de tienda en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/store-zones.png)

1. Seleccione **Guardar**. 

## <a name="arrange-tiles-to-modify-the-layout"></a>Organizar iconos para modificar el diseño
Un paso clave en la personalización de un panel es reorganizar los iconos para crear una vista útil. Los operadores de la aplicación usan el panel para visualizar los datos de telemetría de los dispositivos, administrar dispositivos y supervisar las condiciones de una tienda. Azure IoT Central simplifica la tarea de creación de un panel del desarrollador de aplicaciones. El modo de edición del panel le permite agregar, mover, cambiar de tamaño y eliminar iconos rápidamente. La plantilla de aplicación **Análisis del almacén: finalización de la compra** también simplifica la tarea de crear un panel. Proporciona un diseño de panel de trabajo, con sensores conectados e iconos que muestran los recuentos de líneas de caja y las condiciones ambientales.

En esta sección, reorganizará el panel en la plantilla de aplicación **Análisis del almacén: finalización de la compra** para crear un diseño personalizado.

Para quitar los iconos que no tenga previsto usar en la aplicación:

1. Seleccione **Edit** (Edición) en la barra de herramientas del panel. 

1. Seleccione **X** (Eliminar) para quitar los iconos siguientes: **Back to all zones** (Volver a todas las zonas), **Visit store dashboard** (Visitar panel de tienda), **Wait time** (Tiempo de espera) y los tres iconos asociados con **Checkout 3** (Caja 3). El panel de la tienda de Contoso no usa estos iconos. 

    ![Eliminación de iconos en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/delete-tiles.png)

1. Desplácese para mostrar los restantes iconos del panel.

1. Seleccione **X** (Eliminar) para quitar los iconos siguientes: **Warm-up checkout zone** (Subir temperatura de zona de cajas), **Cool-down checkout zone** (Bajar temperatura de zona de cajas), **Occupancy sensor settings** (Configuración de sensores de ocupación), **Thermostat sensor settings** (Configuración de sensor de termostato) y **Environment conditions** (Condiciones ambientales). 

   ![Eliminación de iconos restantes en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/delete-tiles-2.png)

1. Seleccione **Guardar**. Al quitar los iconos sin usar se libera espacio en la página de edición y se simplifica la vista del panel para los operadores.

1. Vea los cambios en el panel.

   ![Azure IoT Central después de eliminar iconos](./media/tutorial-in-store-analytics-customize-dashboard/after-delete-tiles.png)

Después de quitar los iconos sin usar, reorganice los iconos restantes para crear un diseño organizado. El nuevo diseño incluye espacio para los iconos que agregue en un paso posterior.

Para reorganizar los iconos restantes:

1. Seleccione **Editar**.

1. Seleccione el icono **Occupancy firmware** (Firmware de ocupación) y arrástrelo a la derecha del icono de batería **Occupancy** (Ocupación).

1. Seleccione el icono **Thermostat firmware** (Firmware de termostato) y arrástrelo a la derecha del icono de batería **Thermostat** (termostato).

1. Seleccione **Guardar**.

1. Vea los cambios en el diseño. 

    ![Iconos de batería de firmware de Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/firmware-battery-tiles.png)

## <a name="add-telemetry-tiles-to-display-conditions"></a>Agregar iconos de telemetría para mostrar las condiciones
Después de personalizar el diseño del panel, estará listo para agregar iconos para mostrar los datos de telemetría. Para crear un icono de telemetría, seleccione una plantilla de dispositivo y una instancia de dispositivo y, luego, seleccione los datos de telemetría específicos del dispositivo que desea mostrar en el icono. En la plantilla de aplicación **Análisis del almacén: finalización de la compra** se incluyen varios iconos de telemetría en el panel. Los cuatro iconos de las dos zonas de cajas muestran los datos de telemetría del sensor de ocupación simulado. El icono **People traffic** (Tráfico de personas) muestra los totales de las dos zonas de cajas. 

En esta sección, agregará dos iconos de telemetría más para mostrar los datos de telemetría ambiental de los sensores RuuviTag que agregó en el tutorial [Creación de una aplicación de análisis en tienda en Azure IoT Central](./tutorial-in-store-analytics-create-app.md). 

Para agregar iconos y mostrar datos ambientales de los sensores RuuviTag:

1. Seleccione **Editar**.

1. Seleccione `RuuviTag` en la lista **Device template** (Plantilla de dispositivo). 

1. En **Device instance** (Instancia del dispositivo), seleccione una instancia del dispositivo de uno de los dos sensores RuuviTag. En el ejemplo de la tienda de Contoso, seleccione `Zone 1 Ruuvi` para crear un icono de telemetría para la zona 1. 

1. Seleccione `Relative humidity` y `temperature` en la lista **Telemetry** (Telemetría). Estos son los elementos de telemetría que se muestran para cada zona en el icono.

1. Seleccione **Combine** (Combinar). 

    ![Adición del icono 1 de RuuviTag en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/add-zone1-ruuvi.png)

    Aparece un nuevo icono para mostrar los datos de telemetría de humedad y temperatura combinados del sensor seleccionado. 

1. Seleccione **Configure** (Configurar) en el nuevo icono para el sensor RuuviTag. 

1. En **Title** (Título), cambie el título a *Entorno de zona 1*. 

1. Seleccione **Update configuration** (Actualización de la configuración).

1. Repita los pasos anteriores para crear un icono para la segunda instancia del sensor. En **Title** (Título), cambie el título a *Entorno de zona 2* y, a continuación, seleccione **Update configuration** (Actualización de la configuración).

1. Arrastre el icono titulado **Entorno de zona 2** y sitúelo debajo del icono **Thermostat firmware** (Firmware de termostato). 

1. Arrastre el icono titulado **Entorno de zona 1** y sitúelo debajo del icono **People traffic** (Tráfico de personas). 

1. Seleccione **Guardar**. El panel muestra los datos de telemetría de zona en los dos nuevos iconos.

    ![Todos los iconos de RuuviTag en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/all-ruuvitag-tiles.png)

Para editar el icono **People traffic** (Tráfico de personas) para mostrar los datos de telemetría solo para dos zonas de cajas:

1. Seleccione **Editar**. 

1. Seleccione **Configure** (Configurar) en el icono **People traffic** (Tráfico de personas).

1. En **Telemetry** (Telemetría), seleccione **count 1** (total 1), **count 2** (total 2) y **count 3** (total 3). 

1. Seleccione **Update configuration** (Actualización de la configuración). Al hacerlo, se borrará la configuración existente en el icono. 

1. Seleccione **Configure** (Configurar) de nuevo en el icono **People traffic** (Tráfico de personas).

1. En **Telemetry** (Telemetría), seleccione **count 1** (total 1) y **count 2** (total 2). 

1. Seleccione **Update configuration** (Actualización de la configuración). 

1. Seleccione **Guardar**.  El panel actualizado muestra los totales de solo dos zonas de cajas, que se basan en el sensor de ocupación simulado.

    ![Dos carriles de tráfico de personas en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/people-traffic-two-lanes.png)

## <a name="add-property-tiles-to-display-device-details"></a>Agregar iconos de propiedades para mostrar detalles del dispositivo
Los operadores de la aplicación usan el panel para administrar dispositivos y supervisar el estado. Agregue un icono para cada instancia de RuuviTag para que los operadores puedan ver la versión de software. 

Para agregar un icono de propiedad para cada instancia de RuuviTag:

1. Seleccione **Editar**.

1. Seleccione `RuuviTag` en la lista **Device template** (Plantilla de dispositivo). 

1. En **Device instance** (Instancia del dispositivo), seleccione una instancia del dispositivo de uno de los dos sensores RuuviTag. En el ejemplo de la tienda de Contoso, seleccione `Zone 1 Ruuvi` para crear un icono de telemetría para la zona 1. 

1. Seleccione **Properties > Software version** (Propiedades > Versión del software).

1. Seleccione **Combine** (Combinar). 

1. Seleccione **Configure** (Configurar) en el icono recién creado titulado **Software version** (Versión del software). 

1. En **Title** (Título), cambie el título a *Versión de software 1 de Ruuvi*.

1. Seleccione **Update configuration** (Actualización de la configuración). 

1. Arrastre el icono titulado **Versión de software 1 de Ruuvi** debajo del icono **Entorno zona 1**.

1. Repita los pasos anteriores para crear un icono de propiedad de versión del software para la segunda instancia de RuuviTag. 

1. Seleccione **Guardar**.  

    ![Iconos de propiedades de RuuviTag en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/add-ruuvi-property-tiles.png)

## <a name="add-command-tiles-to-run-commands"></a>Agregar iconos de comandos para ejecutar comandos
Los operadores de la aplicación también usan el panel para administrar dispositivos mediante la ejecución de comandos. Puede agregar iconos de comando al panel que ejecutarán comandos predefinidos en un dispositivo. En esta sección, agregará un icono de comando para que los operadores puedan reiniciar la puerta de enlace de Rigado. 

Para agregar un icono de comando para reiniciar la puerta de enlace:

1. Seleccione **Editar**. 

1. Seleccione `C500` en la lista **Device template** (Plantilla de dispositivo). Esta es la plantilla para la puerta de enlace de Rigado C500. 

1. Seleccione la instancia de puerta de enlace en **Device instance** (Instancia del dispositivo).

1. Seleccione **Command > Reboot** (Comando > Reiniciar) y arrástrelo al panel junto al mapa de la tienda. 

1. Seleccione **Guardar**. 

1. Vea el panel de Contoso completado. 

    ![Personalización del panel completada en Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/completed-dashboard.png)

1. Opcionalmente, seleccione el icono **Reboot** (Reiniciar) para ejecutar el comando de reinicio en la puerta de enlace.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

* Cambiar el nombre del panel
* Personalizar iconos de imagen en el panel
* Organizar iconos para modificar el diseño
* Agregar iconos de telemetría para mostrar las condiciones
* Agregar iconos de propiedades para mostrar detalles del dispositivo
* Agregar iconos de comandos para ejecutar comandos

Ahora que ha personalizado el panel en la aplicación de análisis en tienda de Azure IoT Central, este es el siguiente paso sugerido:

> [!div class="nextstepaction"]
> [Exportación de datos y visualización de conclusiones](./tutorial-in-store-analytics-export-data-visualize-insights.md)
