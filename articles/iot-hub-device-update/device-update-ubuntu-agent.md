---
title: Tutorial sobre Device Update para Azure IoT Hub mediante el agente de paquetes de Ubuntu Server 18.04 (x64) | Microsoft Docs
description: Introducción a Device Update para Azure IoT Hub mediante el agente de paquetes de Ubuntu Server 18.04 (x64).
author: vimeht
ms.author: vimeht
ms.date: 2/16/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: b8fc67076418678f6c53331dc6d0fbe0e7e7a0b1
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111410034"
---
# <a name="device-update-for-azure-iot-hub-tutorial-using-the-package-agent-on-ubuntu-server-1804-x64"></a>Tutorial sobre Device Update para Azure IoT Hub con el agente de paquetes en Ubuntu Server 18.04 (x64)

Device Update para IoT Hub admite dos formas de actualizaciones: basada en imágenes y basada en paquetes.

Las actualizaciones basadas en paquetes son actualizaciones dirigidas que solo modifican una aplicación o un componente específicos en el dispositivo. Suponen un menor consumo de ancho de banda y ayudan a reducir el tiempo necesario para descargar e instalar la actualización. Normalmente, las actualizaciones de paquetes implican menos tiempo de inactividad de los dispositivos al aplicar una actualización y evitan la sobrecarga asociada a la creación de imágenes.

Este tutorial integral a otro le guía por la actualización de Azure IoT Edge en Ubuntu Server 18.04 (x64) mediante el agente de paquetes de Device Update. Aunque en el tutorial se muestra cómo actualizar Iot Edge, con pasos similares puede actualizar otros paquetes, como el motor de contenedor que use.

Las herramientas y los conceptos de este tutorial se siguen aplicando, aunque tenga previsto usar una configuración de plataforma de sistema operativo diferente. Complete esta introducción al proceso de actualización de un extremo a otro y, a continuación, elija su forma preferida de actualizar una plataforma de sistema operativo para profundizar en los detalles.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Descargar e instalar Device Update Agent y sus dependencias
> * Adición de una etiqueta a un dispositivo
> * Importación de una actualización
> * Creación de un grupo de dispositivos
> * Implementar una actualización del paquete
> * Supervisar la implementación de actualizaciones

## <a name="prerequisites"></a>Requisitos previos

* Si todavía no lo ha hecho, cree una [instancia y una cuenta de Device Update](create-device-update-account.md), incluida la configuración de un centro de IoT.
* La [cadena de conexión de un dispositivo de IoT Edge](../iot-edge/how-to-register-device.md?view=iotedge-2020-11&preserve-view=true#view-registered-devices-and-retrieve-connection-strings).

## <a name="prepare-a-device"></a>Preparación de un dispositivo
### <a name="using-the-automated-deploy-to-azure-button"></a>Uso del botón de implementación automatizada en Azure

Para mayor comodidad, en este tutorial se usa una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) basada en [cloud-init](../virtual-machines/linux/using-cloud-init.md) que tiene como fin ayudarle a configurar rápidamente una máquina virtual de Ubuntu 18.04 LTS. Esta plantilla instala el entorno de ejecución de Azure IoT Edge y el agente de paquetes de Device Update para, a continuación, configurar automáticamente el dispositivo con la información de aprovisionamiento mediante la cadena de conexión de un dispositivo IoT Edge (requisito previo) que proporcione. La plantilla de Azure Resource Manager también evita la necesidad de iniciar una sesión de SSH para completar la instalación.

1. Para empezar, haga clic en el siguiente botón:

   [![Botón Implementar en Azure de iotedge-vm-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fiotedge-vm-deploy%2Fdevice-update-tutorial%2FedgeDeploy.json)

1. En la ventana que se abre, rellene los campos de formulario disponibles:

    > [!div class="mx-imgBorder"]
    > [![Captura de pantalla de la plantilla iotedge-vm-deploy](../iot-edge/media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-deploy.png)](../iot-edge/media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-deploy.png)

    **Suscripción**: suscripción de Azure activa en la que se va a implementar la máquina virtual.

    **Grupo de recursos**: grupo de recursos existente o recién creado que va a contener la máquina virtual y sus recursos asociados.

    **Prefijo de etiqueta DNS**: valor requerido (de su elección) que se usa para colocar el nombre de host de la máquina virtual a modo de prefijo.

    **Nombre de usuario administrador**: nombre de usuario al que se proporcionarán privilegios raíz en la implementación.

    **Cadena de conexión de dispositivo**: [cadena de conexión de un dispositivo](../iot-edge/how-to-register-device.md) que se ha creado en la instancia de [IoT Hub](../iot-hub/about-iot-hub.md) de su elección.

    **Tamaño de VM**: [tamaño](../cloud-services/cloud-services-sizes-specs.md) de la máquina virtual que se va a implementar.

    **Versión del SO Ubuntu**: versión del sistema operativo Ubuntu que se va a instalar en la máquina virtual base. Deje el valor predeterminado sin modificar, ya que se establecerá en Ubuntu 18.04-LTS.

    **Ubicación**: [región geográfica](https://azure.microsoft.com/global-infrastructure/locations/) donde se va a implementar la máquina virtual. Este valor se establece de forma predeterminada en la ubicación del grupo de recursos seleccionado.

    **Tipo de autenticación**: elija **sshPublicKey** o **password**, según cuál sea su preferencia.

    **Contraseña o clave de administrador**: valor de la clave pública SSH o de la contraseña en función del tipo de autenticación que se haya escogido.

    Cuando se hayan rellenado todos los campos, active la casilla situada en la parte inferior de la página para aceptar los términos y seleccione **Adquirir** para iniciar la implementación.

1. Confirme que la implementación finaliza correctamente. Aguarde unos minutos a que se complete la implementación para que las actividades posteriores a la instalación y la configuración terminen de instalar IoT Edge y el agente de actualización del paquete de dispositivos.

   Se debe haber implementado un recurso de máquina virtual en el grupo de recursos seleccionado.  Tome nota del nombre de la máquina, que debe tener el formato `vm-0000000000000`. Anote también el **Nombre DNS** asociado, que debe tener el formato `<dnsLabelPrefix>`.`<location>`.cloudapp.azure.com.

    El **Nombre DNS** se puede obtener en la sección **Información general** de la máquina virtual recién implementada en Azure Portal.

    > [!div class="mx-imgBorder"]
    > [![Captura de pantalla con el nombre DNS de la máquina virtual iotedge](../iot-edge/media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-dns-name.png)](../iot-edge/media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-dns-name.png)

   > [!TIP]
   > Si quiere conectarse a través de SSH a la máquina virtual después de la instalación, use el **nombre DNS** asociado con el comando `ssh <adminUsername>@<DNS_Name>`.

### <a name="optional-manually-prepare-a-device"></a>(Opcional) Preparación manual de un dispositivo
Como los pasos automatizados por el [script cloud-init](https://github.com/Azure/iotedge-vm-deploy/blob/1.2.0-rc4/cloud-init.txt), a continuación se indican los pasos manuales para instalar y configurar el dispositivo. Estos pasos se pueden usar para preparar un dispositivo físico.

1. Siga las instrucciones que se indican en [Instalación del entorno de ejecución de Azure IoT Edge](../iot-edge/how-to-install-iot-edge.md?view=iotedge-2020-11&preserve-view=true).
   > [!NOTE]
   > El agente de paquetes de Device Update no depende de IoT Edge. Sin embargo, se basa en el demonio del servicio de identidad de IoT que se instala con IoT Edge (1.2.0 y versiones posteriores) para obtener una identidad y conectarse a IoT Hub.
   >
   > Aunque no se trata en este tutorial, el [demonio del servicio de identidad de IoT se puede instalar de forma independiente en dispositivos IoT basados en Linux](https://azure.github.io/iot-identity-service/installation.html). La secuencia de instalación es importante. El agente de paquetes de Device Update debe instalarse _después_ del servicio de identidad de IoT. De lo contrario, el agente de paquetes no se registrará como componente autorizado para establecer una conexión a IoT Hub.

1. A continuación, instale los paquetes .deb de Device Update Agent.

   ```bash
   sudo apt-get install deviceupdate-agent deliveryoptimization-plugin-apt 
   ```

Los paquetes de software de Device Update para Azure IoT Hub están sujetos a los siguientes términos de licencia:
  * [Licencia de actualización de dispositivos para IoT Hub](https://github.com/Azure/iot-hub-device-update/blob/main/LICENSE.md)
  * [Licencia de cliente de optimización de distribución](https://github.com/microsoft/do-client/blob/main/LICENSE)

Lea los términos de licencia antes de usar el paquete. La instalación y el uso de un paquete constituyen la aceptación de estos términos. Si no acepta los términos de licencia, no utilice el paquete.

## <a name="add-a-tag-to-your-device"></a>Adición de una etiqueta a un dispositivo

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al centro de IoT.

2. En "IoT Edge", en el panel de navegación izquierdo, busque el dispositivo de IoT Edge y vaya al dispositivo o módulo gemelo.

3. En el módulo gemelo del agente de Device Update, cambie los valores de etiqueta de Device Update existentes a null para eliminarlos. Si usa la identidad de dispositivo con el agente de Device Update, realice estos cambios en el dispositivo gemelo.

4. Agregue un nuevo valor de etiqueta de Device Update como se muestra a continuación.

```JSON
    "tags": {
            "ADUGroup": "<CustomTagValue>"
            },
```

## <a name="import-update"></a>Importación de actualización

1. Vaya a [Versiones de Device Update](https://github.com/Azure/iot-hub-device-update/releases) en GitHub y haga clic en la lista desplegable "Recursos".

3. Haga clic en el archivo `Edge.package.update.samples.zip` para descargarlo.

5. Extraiga el contenido de la carpeta para ver un ejemplo de actualización y sus manifiestos de importación correspondientes. 

2. En Azure Portal, seleccione la opción Actualizaciones del dispositivo en Administración de dispositivos automática, en la barra de navegación izquierda de IoT Hub.

3. Seleccione la pestaña Actualizaciones.

4. Seleccione "+ Import New Update" (Importar nueva actualización).

5. Seleccione el icono de la carpeta o el cuadro de texto en "Select an Import Manifest File" (Seleccionar un archivo de manifiesto de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione el archivo de manifiesto de importación `sample-1.0.1-aziot-edge-importManifest.json` desde la carpeta que descargó anteriormente. Seleccione el icono de la carpeta o el cuadro de texto en "Select one or more update files" (Seleccionar uno o varios archivos de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione el archivo de actualización del manifiesto APT `sample-1.0.1-aziot-edge-apt-manifest.json`desde la carpeta que descargó anteriormente.
Se actualizarán los paquetes `aziot-identity-service` y `aziot-edge` a la versión 1.2.0~rc4-1 en el dispositivo.

   :::image type="content" source="media/import-update/select-update-files.png" alt-text="Captura de pantalla que muestra la selección del archivo de actualización." lightbox="media/import-update/select-update-files.png":::

6. Seleccione el icono de la carpeta o el cuadro de texto en "Select a storage container" (Seleccionar un contenedor de almacenamiento). Luego, seleccione la cuenta de almacenamiento adecuada.

7. Si ya ha creado un contenedor, puede volver a usarlo (de lo contrario, seleccione "+ Contenedor" para crear contenedor de almacenamiento para las actualizaciones).  Seleccione el contenedor que desee usar y haga clic en "Seleccionar".

   :::image type="content" source="media/import-update/container.png" alt-text="Captura de pantalla que muestra la selección del contenedor." lightbox="media/import-update/container.png":::

8. Seleccione "Enviar" para iniciar el proceso de importación.

9. Se inicia el proceso de importación y la pantalla cambia a la sección "Historial de importación". Seleccione "Actualizar" para ver el progreso hasta que se complete el proceso de importación. Según sea el tamaño de la actualización, la importación puede tardar pocos minutos o más tiempo.

   :::image type="content" source="media/import-update/update-publishing-sequence-2.png" alt-text="Captura de pantalla que muestra la secuencia de importación de la actualización." lightbox="media/import-update/update-publishing-sequence-2.png":::

10. Cuando la columna Estado indique que la importación se ha realizado correctamente, seleccione el encabezado "Ready to Deploy" (Listo para implementar). Debería ver la actualización importada en la lista.

[Más información](import-update.md) sobre la importación de actualizaciones.

## <a name="create-update-group"></a>Creación de un grupo de actualización

1. Vaya a la instancia de IoT Hub que conectó anteriormente a la instancia de Device Update.

1. Seleccione la opción Actualizaciones del dispositivo en Administración de dispositivos automática en la barra de navegación izquierda.

1. Seleccione la pestaña Groups (Grupos) en la parte superior de la página.

1. Seleccione el botón Add (Agregar) para crear un grupo.

1. Seleccione en la lista la etiqueta IoT Hub que ha creado en el paso anterior. Seleccione Create update group (Crear grupo de actualización).

   :::image type="content" source="media/create-update-group/select-tag.PNG" alt-text="Captura de pantalla que muestra la selección de etiquetas." lightbox="media/create-update-group/select-tag.PNG":::

[Más información](create-update-group.md) sobre cómo agregar etiquetas y crear grupos de actualización

## <a name="deploy-update"></a>Implementación de la actualización

1. Una vez creado el grupo, debería ver una nueva actualización disponible para el grupo de dispositivos, con un vínculo a la actualización en la columna _Actualizaciones disponibles_. Es posible que tenga que actualizar una vez.

1. Haga clic en el vínculo a la actualización disponible.

1. Confirme que se ha seleccionado el grupo correcto como grupo de destino y programe la implementación.

   :::image type="content" source="media/deploy-update/select-update.png" alt-text="Select update" lightbox="media/deploy-update/select-update.png"::: (Seleccionar actualización)

   > [!TIP]
   > De forma predeterminada, la fecha y hora de inicio es de 24 horas a partir de la hora actual. Asegúrese de seleccionar una fecha u hora diferentes si quiere que la implementación comience antes.

1. Seleccione Deploy update (Implementar actualización).

1. Consulte el gráfico de compatibilidad. Debería ver que la actualización está en curso. 

   :::image type="content" source="media/deploy-update/update-in-progress.png" alt-text="Actualización en curso" lightbox="media/deploy-update/update-in-progress.png":::

1. Una vez que el dispositivo se haya actualizado correctamente, debería ver que el gráfico de compatibilidad y la actualización de los detalles de implementación reflejan lo mismo. 

   :::image type="content" source="media/deploy-update/update-succeeded.png" alt-text="Actualización correcta" lightbox="media/deploy-update/update-succeeded.png":::

## <a name="monitor-an-update-deployment"></a>Supervisión de una implementación de actualizaciones

1. Seleccione la pestaña Implementaciones en la parte superior de la página.

   :::image type="content" source="media/deploy-update/deployments-tab.png" alt-text="Pestaña Implementaciones" lightbox="media/deploy-update/deployments-tab.png":::

1. Seleccione la implementación que creó para ver los detalles de la implementación.

   :::image type="content" source="media/deploy-update/deployment-details.png" alt-text="Detalles de implementación" lightbox="media/deploy-update/deployment-details.png":::

1. Seleccione Actualizar para ver los detalles de estado más recientes. Continúe con este proceso hasta que el estado cambie a Correcto.

Ha completado correctamente una actualización integral del paquete mediante Device Update para IoT Hub en un dispositivo Ubuntu Server 18.04 (x64). 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, limpie la cuenta de Device Update, la instancia, el centro de IoT y el dispositivo de IoT Edge (si creó la máquina virtual mediante el botón Implementar en Azure). Para ello, vaya a cada recurso individual y seleccione "Delete" (Eliminar). Antes de limpiar la cuenta de Device Update, debe limpiar la instancia de Device Update.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial de actualización de imágenes en Raspberry PI 3 B+](device-update-raspberry-pi.md)
