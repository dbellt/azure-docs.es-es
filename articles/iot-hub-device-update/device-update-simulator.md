---
title: Tutorial sobre la actualización de dispositivos para Azure IoT Hub mediante el agente de referencia del simulador de Ubuntu 18.04 (x64) | Microsoft Docs
description: Introducción a la actualización de dispositivos para Azure IoT Hub mediante el agente de referencia del simulador de Ubuntu 18.04 (x64).
author: valls
ms.author: valls
ms.date: 2/11/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: 77bfab51cf7ccc9ec6ffe3b0018c0a488b601a0a
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070216"
---
# <a name="device-update-for-azure-iot-hub-tutorial-using-the-ubuntu-1804-x64-simulator-reference-agent"></a>Tutorial sobre la actualización de dispositivos para Azure IoT Hub mediante el agente de referencia del simulador de Ubuntu 18.04 (x64)

Device Update para IoT Hub admite dos formas de actualizaciones: basada en imágenes y basada en paquetes.

Las actualizaciones con imágenes proporcionan un mayor nivel de confianza en el estado final del dispositivo. Normalmente, es más fácil replicar los resultados de una actualización basada en imágenes entre un entorno de preproducción y un entorno de producción, ya que no plantea los mismos desafíos que los paquetes y sus dependencias. Debido a su naturaleza atómica, también puede adoptar fácilmente un modelo de conmutación por error A/B.

Este tutorial le guiará por los pasos necesarios para completar una actualización basada en imágenes de un extremo a otro mediante Device Update para IoT Hub. 

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Descargar e instalación de Java
> * Adición de una etiqueta a un dispositivo IoT
> * Importación de una actualización
> * Creación de un grupo de dispositivos
> * Implementación de una actualización basada en imágenes
> * Supervisión de la implementación de la actualización

## <a name="prerequisites"></a>Requisitos previos
* Si todavía no lo ha hecho, cree una [instancia y una cuenta de Device Update](create-device-update-account.md), incluida la configuración de un centro de IoT.

### <a name="download-and-install"></a>Descarga e instalación

* Cmdlets Az (CLI de Azure) para PowerShell:
  * Abra PowerShell > Instalar la CLI de Azure ("Y" para solicitar la instalación desde un origen que "no es de confianza").

```powershell
PS> Install-Module Az -Scope CurrentUser
```

### <a name="enable-wsl-on-your-windows-device-windows-subsystem-for-linux"></a>Habilitación de WSL en un dispositivo Windows (Subsistema de Windows para Linux)

1. Abra PowerShell como administrador en la máquina y ejecute el siguiente comando (es posible que se le pida que la reinicie después de cada paso; reiníciela cuando se le pida):

```powershell
PS> Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
PS> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

(*es posible que se le pida que la reinicie después de este paso*)

2. Vaya a Microsoft Store en la web e instale [Ubuntu 18.04 LTS](https://www.microsoft.com/p/ubuntu-1804-lts/9n9tngvndl3q?activetab=pivot:overviewtab`).

3. Inicie "Ubuntu 18.04 LTS" e instálelo.

4. Una vez instalado, se le pedirá que establezca el nombre de raíz (nombre de usuario) y la contraseña. Asegúrese de usar un nombre de raíz y una contraseña fáciles de recordar.

5. En PowerShell, ejecute el siguiente comando para establecer que Ubuntu sea la distribución de Linux predeterminada:

```powershell
PS> wsl --setdefault Ubuntu-18.04
```

6. Enumere todas las distribuciones de Linux y asegúrese de que Ubuntu es la predeterminada.

```powershell
PS> wsl --list
```

7. Debería ver: **Ubuntu-18.04 (valor predeterminado)**

## <a name="download-device-update-ubuntu-1804-x64-simulator-reference-agent"></a>Descarga del agente de referencia del simulador de Ubuntu 18.04 (x64) de Device Update

La imagen de actualización de Ubuntu se puede descargar de la sección *Assets* (Recursos) de las notas de la versión [aquí](https://github.com/Azure/iot-hub-device-update/releases).

Hay dos versiones del agente. Si va a ejercitar un escenario basado en imágenes, use AducIotAgentSim-microsoft-swupdate, mientras que si va a ejercitar un escenario basado en paquetes, use AducIotAgentSim-microsoft-apt.

## <a name="install-device-update-agent-simulator"></a>Instalación del simulador de Device Update Agent

1. Inicie Ubuntu WSL y escriba el siguiente comando (tenga en cuenta el espacio adicional y el punto al final).

  ```shell
  explorer.exe .
  ```

2. Copie AducIotAgentSim-microsoft-swupdate (o AducIotAgentSim-microsoft-apt) de la carpeta local en que se descargó (debajo de /mnt) a su carpeta local de WSL.

3. Ejecute el siguiente comando para que se puedan ejecutar los archivos binarios.

  ```shell
  sudo chmod u+x AducIotAgentSim-microsoft-swupdate
  ```

  or

  ```shell
  sudo chmod u+x AducIotAgentSim-microsoft-apt
  ```
El software de Device Update para Azure IoT Hub está sujeto a los siguientes términos de licencia:
   * [Licencia de Device Update para IoT Hub](https://github.com/Azure/iot-hub-device-update/blob/main/LICENSE.md)
   * [Licencia de cliente de Optimización de distribución](https://github.com/microsoft/do-client/blob/main/LICENSE)
   
Lea los términos de licencia antes de usar el agente. La instalación y el uso constituyen la aceptación de estos términos. Si no está de acuerdo con los términos de licencia, no use el agente de Device Update para IoT Hub.

## <a name="add-device-to-azure-iot-hub"></a>Incorporación de un dispositivo a Azure IoT Hub

Una vez que Device Update Agent se ejecuta en un dispositivo IoT, el dispositivo debe agregarse a Azure IoT Hub.  En Azure IoT Hub, se generará una cadena de conexión para un dispositivo concreto.

1. En Azure Portal, inicie Device Update para IoT Hub.
2. Cree un dispositivo.
3. En el lado izquierdo de la página, vaya a "IoT Devices" (Dispositivos IoT) y seleccione "New" (Nuevo).
4. Especifique un nombre para el dispositivo en "ID. de dispositivo": asegúrese de que la casilla "Autogenerate keys" (Generar claves automáticamente) esté activada.
5. Seleccione "Guardar".
6. Volverá a la página "Dispositivos" y el dispositivo que ha creado debería estar en la lista. Seleccione ese dispositivo.
7. En la vista de dispositivos, seleccione el icono "Copiar", que está junto a "Cadena de conexión principal".
8. Pegue en algún lugar los caracteres copiados para su uso posterior en los pasos siguientes. **Esta cadena copiada es la cadena de conexión del dispositivo**.

## <a name="add-connection-string-to-simulator"></a>Adición de una cadena de conexión al simulador

Inicie Device Update Agent en los nuevos dispositivos de software.

1. Inicie Ubuntu.
2. Ejecute Device Update Agent y especifique la cadena de conexión del dispositivo de la sección anterior entre comillas:

Reemplace `<device connection string>` por una cadena de conexión
```shell
./AducIotAgentSim-microsoft-swupdate -c '<device connection string>'
```

o bien

```shell
./AducIotAgentSim-microsoft-apt -c '<device connection string>'
```

3. Desplácese hacia arriba y busque la cadena que indica que el dispositivo está en estado "Inactivo". El estado "Inactivo" significa que el dispositivo está listo para los comandos de servicio:

```markdown
Agent running. [main]
```

## <a name="add-a-tag-to-your-device"></a>Adición de una etiqueta a un dispositivo

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

1. Cree un manifiesto de importación siguiendo estas [instrucciones](import-update.md).
2. Seleccione la opción Actualizaciones del dispositivo en Administración de dispositivos automática en la barra de navegación izquierda.

3. Seleccione la pestaña Actualizaciones.

4. Seleccione "+ Import New Update" (Importar nueva actualización).

5. Seleccione el icono de la carpeta o el cuadro de texto en "Select an Import Manifest File" (Seleccionar un archivo de manifiesto de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione el manifiesto de importación que creó anteriormente.  Seleccione el icono de la carpeta o el cuadro de texto en "Select one or more update files" (Seleccionar uno o varios archivos de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione la imagen de actualización de Ubuntu que descargó anteriormente. 

   :::image type="content" source="media/import-update/select-update-files.png" alt-text="Captura de pantalla que muestra la selección del archivo de actualización." lightbox="media/import-update/select-update-files.png":::

6. Seleccione el icono de la carpeta o el cuadro de texto en "Select a storage container" (Seleccionar un contenedor de almacenamiento). Luego, seleccione la cuenta de almacenamiento adecuada.

7. Si ya ha creado un contenedor, puede volver a usarlo (de lo contrario, seleccione "+ Contenedor" para crear contenedor de almacenamiento para las actualizaciones).  Seleccione el contenedor que desee usar y haga clic en "Seleccionar".
  
  :::image type="content" source="media/import-update/container.png" alt-text="Captura de pantalla que muestra la selección del contenedor." lightbox="media/import-update/container.png":::

8. Seleccione "Enviar" para iniciar el proceso de importación.

9. Se inicia el proceso de importación y la pantalla cambia a la sección "Historial de importación". Seleccione "Actualizar" para ver el progreso hasta que se complete el proceso de importación. En función del tamaño de la actualización, puede completarse en unos minutos, pero puede tardar más tiempo.
   
   :::image type="content" source="media/import-update/update-publishing-sequence-2.png" alt-text="Captura de pantalla que muestra la secuencia de importación de la actualización." lightbox="media/import-update/update-publishing-sequence-2.png":::

10. Cuando la columna Estado indique que la importación se ha realizado correctamente, seleccione el encabezado "Ready to Deploy" (Listo para implementar). Debería ver la actualización importada en la lista.

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

3. Seleccione Actualizar para ver los detalles de estado más recientes. Continúe con este proceso hasta que el estado cambie a Succeded (Correcto).

Ya ha completado una actualización basada en imágenes de un extremo a otro, para lo que ha usado Device Update para IoT Hub mediante el agente de referencia del simulador Ubuntu 18.04 (x64).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, limpie la cuenta de Device Update, la instancia, IoT Hub y el dispositivo IoT. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Solución de problemas](troubleshoot-device-update.md)

