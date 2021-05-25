---
title: Tutorial de Device Update para Azure IoT Hub con la imagen de referencia de Yocto para Raspberry Pi 3 B+ | Microsoft Docs
description: Introducción a Device Update para Azure IoT Hub con la imagen de referencia de Yocto para Raspberry Pi 3 B+.
author: ValOlson
ms.author: valls
ms.date: 2/11/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: 05b42a3dd6d5df7bf4484e23b2780732d55ab731
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109789162"
---
# <a name="device-update-for-azure-iot-hub-tutorial-using-the-raspberry-pi-3-b-reference-image"></a>Tutorial de Device Update para Azure IoT Hub con la imagen de referencia para Raspberry Pi 3 B+

Device Update para IoT Hub admite dos formas de actualizaciones: basada en imágenes y basada en paquetes.

Las actualizaciones con imágenes proporcionan un mayor nivel de confianza en el estado final del dispositivo. Normalmente, es más fácil replicar los resultados de una actualización basada en imágenes entre un entorno de preproducción y un entorno de producción, ya que no plantea los mismos desafíos que los paquetes y sus dependencias. Debido a su naturaleza atómica, también puede adoptar fácilmente un modelo de conmutación por error A/B.

Este tutorial le guiará por los pasos necesarios para completar una actualización basada en imágenes completa mediante Device Update for IoT Hub en una placa Raspberry Pi 3 B+. 

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Descarga de la imagen
> * Adición de una etiqueta al dispositivo IoT
> * Importación de una actualización
> * Creación de un grupo de dispositivos
> * Implementación de una actualización basada en imágenes
> * Supervisión de la implementación de la actualización

## <a name="prerequisites"></a>Requisitos previos
* Si todavía no lo ha hecho, cree una [instancia y una cuenta de Device Update](create-device-update-account.md), incluida la configuración de un centro de IoT.

## <a name="download-image"></a>Descarga de la imagen

Se proporcionan imágenes de ejemplo en "Recursos" en la [página de versiones de GitHub de Device Update](https://github.com/Azure/iot-hub-device-update/releases). El archivo .gz es la imagen base que puede grabar en memoria flash en una placa Raspberry Pi B3+ y el archivo swUpdate es la actualización que se importaría mediante Device Update for IoT Hub. 

## <a name="flash-sd-card-with-image"></a>Escritura de la memoria flash de la tarjeta SD con la imagen

Con su herramienta de escritura en memoria flash del sistema operativo favorita, instale la imagen base de Device Update (adu-base-image) en la tarjeta SD que se usará en el dispositivo Raspberry Pi 3 B+.

### <a name="using-bmaptool-to-flash-sd-card"></a>Uso de bmaptool para escribir la memoria flash de la tarjeta SD

1. Si aún no lo ha hecho, instale la utilidad `bmaptool`.

   ```shell
   sudo apt-get install bmap-tools
   ```

2. Busque la ruta de acceso de la tarjeta SD en `/dev`. La ruta de acceso debe tener un aspecto similar a `/dev/sd*` o `/dev/mmcblk*`. Puede usar la utilidad `dmesg` para buscar la ruta de acceso correcta.

3. Deberá desmontar todas las particiones montadas antes de escribir la memoria flash.

   ```shell
   sudo umount /dev/<device>
   ```

4. Asegúrese de que tiene permisos de escritura en el dispositivo.

   ```shell
   sudo chmod a+rw /dev/<device>
   ```

5. Opcional. Para acelerar la escritura de la memoria flash, descargue el archivo bimap junto con el archivo de la imagen y colóquelos en el mismo directorio.

6. Escriba la memoria flash de la tarjeta SD.

   ```shell
   sudo bmaptool copy <path to image> /dev/<device>
   ```
   
El software de Device Update para Azure IoT Hub está sujeto a los siguientes términos de licencia:
   * [Licencia de Device Update para IoT Hub](https://github.com/Azure/iot-hub-device-update/blob/main/LICENSE.md)
   * [Licencia de cliente de Optimización de distribución](https://github.com/microsoft/do-client/blob/main/LICENSE)
   
Lea los términos de licencia antes de usar el agente. La instalación y el uso constituyen la aceptación de estos términos. Si no está de acuerdo con los términos de licencia, no use Device Update Agent para IoT Hub.

## <a name="create-device-or-module-in-iot-hub-and-get-connection-string"></a>Creación de un dispositivo o un módulo en IoT Hub y obtención de la cadena de conexión

Ahora, se debe agregar el dispositivo a Azure IoT Hub.  En Azure IoT Hub, se generará una cadena de conexión para el dispositivo.

1. En Azure Portal, inicie Azure IoT Hub.
2. Cree un dispositivo.
3. En el lado izquierdo de la página, vaya a "IoT Devices" (Dispositivos IoT) y seleccione "New" (Nuevo).
4. Especifique un nombre para el dispositivo en "ID. de dispositivo": asegúrese de que la casilla "Autogenerate keys" (Generar claves automáticamente) esté activada.
5. Seleccione "Save" (Guardar).
6. Ahora se le devolverá a la página "Devices" (Dispositivos) y el dispositivo que ha creado debe estar en la lista. 
7. Obtención de la cadena de conexión del dispositivo:
    - Opción 1: Uso del agente de Device Update con una identidad de módulo: en la misma página "Devices" (Dispositivos), haga clic en "+ Add Module Identity" (+ Agregar identidad de módulo) en la parte superior. Cree un nuevo módulo de Device Update con el nombre "IoTHubDeviceUpdate", elija otras opciones según sea necesario para su caso de uso y, a continuación, haga clic en "Guardar" (Save). Haga clic en el módulo recién creado y, en la vista del módulo, seleccione el icono de copia junto a "Primary Connection String" (Cadena de conexión principal).
    - Opción 2: Uso del agente de Device Update con la identidad del dispositivo: en la vista del dispositivo, seleccione el icono de copia junto a "Primary Connection String" (Cadena de conexión principal).
8. Pegue en algún lugar los caracteres copiados para su uso posterior en los pasos siguientes.
   **Esta cadena copiada es la cadena de conexión del dispositivo**.

## <a name="provision-connection-string-on-sd-card"></a>Aprovisionamiento de la cadena de conexión en la tarjeta SD

1. Asegúrese de que el dispositivo Raspberry Pi3 esté conectado a la red.
2. En PowerShell, use el siguiente comando para conectarse mediante ssh al dispositivo.
   ```markdown
   ssh raspberrypi3 -l root
      ```
4. Escriba "root" como nombre de inicio de sesión y la contraseña se debe dejar vacía.
5. Después de conectarse correctamente mediante ssh al dispositivo, ejecute los siguientes comandos.
 
Reemplace `<device connection string>` por la cadena de conexión.
 ```markdown
    echo "connection_string=<device connection string>" > /adu/adu-conf.txt  
    echo "aduc_manufacturer=ADUTeam" >> /adu/adu-conf.txt
    echo "aduc_model=RefDevice" >> /adu/adu-conf.txt
   ```

## <a name="connect-the-device-in-device-update-iot-hub"></a>Conexión del dispositivo en Device Update para IoT Hub

1. En el lado izquierdo de la página, seleccione "IoT Devices" (Dispositivos IoT).
2. Seleccione el vínculo con el nombre del dispositivo.
3. En la parte superior de la página, seleccione "Device Twin" (Dispositivo gemelo) si se conecta directamente a Device Update con la identidad del dispositivo IoT. De lo contrario, seleccione el módulo que creó anteriormente y haga clic en "Module Twin" (Módulo gemelo).
4. En la sección "reported" (Notificado) de las propiedades del dispositivo gemelo, busque la versión del kernel de Linux.
Para un nuevo dispositivo, que no ha recibido una actualización desde Device Update, el valor de [DeviceManagement:DeviceInformation:1.swVersion](device-update-plug-and-play.md) representa la versión de firmware que se ejecuta en el dispositivo.  Una vez que se haya aplicado una actualización a un dispositivo, Device Update usará el valor de la propiedad [AzureDeviceUpdateCore:ClientMetadata:4.installedUpdateId](device-update-plug-and-play.md) para representar la versión de firmware que se ejecuta en el dispositivo.
5. Los archivos de la imagen base y la de actualización tienen un número de versión en el nombre de archivo.

   ```markdown
   adu-<image type>-image-<machine>-<version number>.<extension>
   ```

Use ese número de versión en el paso Importación de la actualización que aparece a continuación.

## <a name="add-a-tag-to-your-device"></a>Adición de una etiqueta al dispositivo

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al centro de IoT.

2. En "Dispositivos IoT" o "IoT Edge" en el panel de navegación izquierdo, busque el dispositivo IoT y vaya al dispositivo o módulo gemelo.

3. En el módulo gemelo del agente de Device Update, cambie los valores de etiqueta de Device Update existentes a null para eliminarlos. Si usa la identidad de dispositivo con el agente de Device Update, realice estos cambios en el dispositivo gemelo.

4. Agregue un nuevo valor de etiqueta de Device Update como se muestra a continuación.

```JSON
    "tags": {
            "ADUGroup": "<CustomTagValue>"
            }
```

## <a name="import-update"></a>Importación de actualización

1. Descargue el [manifiesto de importación de ejemplo](https://github.com/Azure/iot-hub-device-update/releases/download/0.7.0-rc1/TutorialImportManifest.json) y la [actualización de imagen de ejemplo](https://github.com/Azure/iot-hub-device-update/releases/download/0.7.0-rc1/adu-update-image-raspberrypi3-0.6.5073.1.swu).
2. Inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya a su instancia de IoT Hub con Device Update. A continuación, seleccione la opción Actualizaciones de dispositivos en Administración de dispositivos automática en la barra de navegación izquierda.
3. Seleccione la pestaña Actualizaciones.
4. Seleccione "+ Import New Update" (Importar nueva actualización).
5. Seleccione el icono de la carpeta o el cuadro de texto en "Select an Import Manifest File" (Seleccionar un archivo de manifiesto de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione el _manifiesto de importación de ejemplo_ que ha descargado en el paso 1 anterior.  Seleccione el icono de la carpeta o el cuadro de texto en "Select one or more update files" (Seleccionar uno o varios archivos de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione el _archivo de actualización de ejemplo_ que ha descargado en el paso 1 anterior.
   
   :::image type="content" source="media/import-update/select-update-files.png" alt-text="Captura de pantalla que muestra la selección del archivo de actualización." lightbox="media/import-update/select-update-files.png":::

5. Seleccione el icono de la carpeta o el cuadro de texto en "Select a storage container" (Seleccionar un contenedor de almacenamiento). Luego, seleccione la cuenta de almacenamiento adecuada.

6. Si ya ha creado un contenedor, puede volver a usarlo (de lo contrario, seleccione "+ Contenedor" para crear contenedor de almacenamiento para las actualizaciones).  Seleccione el contenedor que desee usar y haga clic en "Seleccionar".
  
  :::image type="content" source="media/import-update/container.png" alt-text="Captura de pantalla que muestra la selección del contenedor." lightbox="media/import-update/container.png":::

7. Seleccione "Enviar" para iniciar el proceso de importación.

8. Se inicia el proceso de importación y la pantalla cambia a la sección "Historial de importación". Seleccione "Actualizar" para ver el progreso hasta que se complete el proceso de importación. En función del tamaño de la actualización, puede completarse en unos minutos, pero puede tardar más tiempo.
   
   :::image type="content" source="media/import-update/update-publishing-sequence-2.png" alt-text="Captura de pantalla que muestra la secuencia de importación de la actualización." lightbox="media/import-update/update-publishing-sequence-2.png":::

9. Cuando la columna Estado indique que la importación se ha realizado correctamente, seleccione el encabezado "Ready to Deploy" (Listo para implementar). Debería ver la actualización importada en la lista.

[Más información](import-update.md) sobre la importación de actualizaciones.

## <a name="create-update-group"></a>Creación de un grupo de actualización

1. Vaya a la instancia de IoT Hub que conectó anteriormente a la instancia de Device Update.

2. Seleccione la opción Actualizaciones del dispositivo en Administración de dispositivos automática en la barra de navegación izquierda.

3. Seleccione la pestaña Groups (Grupos) en la parte superior de la página. 

4. Seleccione el botón Add (Agregar) para crear un grupo.

5. Seleccione en la lista la etiqueta IoT Hub que ha creado en el paso anterior. Seleccione Create update group (Crear grupo de actualización).

   :::image type="content" source="media/create-update-group/select-tag.PNG" alt-text="Captura de pantalla que muestra la selección de etiquetas." lightbox="media/create-update-group/select-tag.PNG":::

[Más información](create-update-group.md) sobre cómo agregar etiquetas y crear grupos de actualización


## <a name="deploy-update"></a>Implementación de la actualización

1. Una vez creado el grupo, debería ver que hay una actualización nueva disponible para el grupo de dispositivos, con un vínculo a la actualización en Actualizaciones pendientes. Es posible que tenga que actualizar una vez. 

2. Haga clic en la actualización disponible.

3. Confirme que se ha seleccionado el grupo correcto como grupo de destino. Programe la implementación y, a continuación, seleccione Deploy update (Implementar actualización).

   :::image type="content" source="media/deploy-update/select-update.png" alt-text="Select update" lightbox="media/deploy-update/select-update.png"::: (Seleccionar actualización)

4. Consulte el gráfico de compatibilidad. Debería ver que la actualización está en curso. 

   :::image type="content" source="media/deploy-update/update-in-progress.png" alt-text="Actualización en curso" lightbox="media/deploy-update/update-in-progress.png":::

5. Una vez que el dispositivo se haya actualizado correctamente, debería ver que el gráfico de compatibilidad y la actualización de los detalles de implementación reflejan lo mismo. 

   :::image type="content" source="media/deploy-update/update-succeeded.png" alt-text="Actualización correcta" lightbox="media/deploy-update/update-succeeded.png":::

## <a name="monitor-an-update-deployment"></a>Supervisión de una implementación de actualizaciones

1. Seleccione la pestaña Implementaciones en la parte superior de la página.

   :::image type="content" source="media/deploy-update/deployments-tab.png" alt-text="Pestaña Implementaciones" lightbox="media/deploy-update/deployments-tab.png":::

2. Seleccione la implementación que creó para ver los detalles de la implementación.

   :::image type="content" source="media/deploy-update/deployment-details.png" alt-text="Detalles de implementación" lightbox="media/deploy-update/deployment-details.png":::

3. Seleccione Actualizar para ver los detalles de estado más recientes. Continúe con este proceso hasta que el estado cambie a Succeeded (Correcto).

Ahora ha completado una actualización correcta de la imagen de un extremo a otro con Device Update para IoT Hub en un dispositivo Raspberry Pi 3 B+. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, limpie la cuenta de Device Update, la instancia, IoT Hub y el dispositivo IoT. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agente de referencia del simulador](device-update-simulator.md)
