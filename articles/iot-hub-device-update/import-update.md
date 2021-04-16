---
title: Procedimientos para la importación de una actualización nueva | Microsoft Docs
description: Guía de procedimientos para importar una nueva actualización en Hub Device Update para IoT Hub.
author: andrewbrownmsft
ms.author: andbrown
ms.date: 2/11/2021
ms.topic: how-to
ms.service: iot-hub-device-update
ms.openlocfilehash: ede0d279b8769f49afcdae1cb9352c1b47fb59b5
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105932410"
---
# <a name="import-new-update"></a>Importación de una nueva actualización
Aprenda a importar una nueva actualización en Device Update para IoT Hub. Si aún no lo ha hecho, asegúrese de familiarizarse con los [conceptos de importación](import-concepts.md) básicos.

## <a name="prerequisites"></a>Requisitos previos

* [Acceso a una instancia de IoT Hub con Device Update para IoT Hub habilitado](create-device-update-account.md). 
* Un dispositivo IoT (o simulador) aprovisionado para Device Update en IoT Hub.
   * Si se usa un dispositivo real, necesitará un archivo de imagen de actualización para la actualización de la imagen o un [archivo de manifiesto APT](device-update-apt-manifest.md) para la actualización del paquete.
* [PowerShell 5](/powershell/scripting/install/installing-powershell) o posterior (incluye las instalaciones de Linux, macOS y Windows).
* Exploradores admitidos:
  * [Microsoft Edge](https://www.microsoft.com/edge)
  * Google Chrome

> [!NOTE]
> Es posible que algunos de los datos que se envíen a este servicio se procesen en una región que no es en la que se creó esta instancia.

## <a name="create-device-update-import-manifest"></a>Creación de un manifiesto de importación Device Update

1. Asegúrese de que el archivo de imagen de actualización o el archivo de manifiesto APT se encuentra en un directorio al que se pueda acceder desde PowerShell.

2. Cree un archivo de texto denominado **AduUpdate.psm1** en el directorio donde se encuentra el archivo de imagen de actualización o el archivo de manifiesto APT. Después, abra el cmdlet de PowerShell [AduUpdate.psm1](https://github.com/Azure/iot-hub-device-update/tree/main/tools/AduCmdlets), copie el contenido en el archivo de texto y, a continuación, guarde el archivo de texto.

3. En PowerShell, navegue hasta el directorio en el que ha creado el cmdlet de PowerShell en el paso 2. Use la opción Copiar que aparece a continuación y péguela en PowerShell para ejecutar los comandos:

    ```powershell
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
    Import-Module .\AduUpdate.psm1
    ```

4. Ejecute los siguientes comandos, para lo que debe reemplazar los valores de los parámetros de ejemplo para generar un manifiesto de importación, un archivo JSON que describe la actualización:
    ```powershell
    $compat = New-AduUpdateCompatibility -DeviceManufacturer 'deviceManufacturer' -DeviceModel 'deviceModel'

    $importManifest = New-AduImportManifest -Provider 'updateProvider' -Name 'updateName' -Version 'updateVersion' `
                                            -UpdateType 'updateType' -InstalledCriteria 'installedCriteria' `
                                            -Compatibility $compat -Files 'updateFilePath(s)'

    $importManifest | Out-File '.\importManifest.json' -Encoding UTF8
    ```

    Para obtener una referencia rápida, estos son algunos valores de ejemplo de los parámetros anteriores. También puede consultar el [esquema del manifiesto de importación](import-schema.md) completo para obtener más información.

    | Parámetro | Descripción |
    | --------- | ----------- |
    | deviceManufacturer | El fabricante del dispositivo con el que la actualización es compatible, por ejemplo, Contoso. Debe coincidir con la [propiedad del dispositivo](./device-update-plug-and-play.md#device-properties) _manufacturer_.
    | deviceModel | El modelo del dispositivo con el que la actualización es compatible, por ejemplo, Tostador. Debe coincidir con la [propiedad del dispositivo](./device-update-plug-and-play.md#device-properties) _model_.
    | updateProvider | Entidad que está creando la actualización o que es directamente responsable de ella. Suele ser un nombre de empresa.
    | updateName | Identificador de una clase de actualizaciones. La clase puede ser cualquier cosa que elija. Suele ser un nombre de dispositivo o de modelo.
    | updateVersion | Número de versión que diferencia a esta actualización de otras con el mismo proveedor y el mismo nombre. No coincide con ninguna versión de un componente de software individual en el dispositivo (pero puede coincidir si elige uno).
    | updateType | <ul><li>Especifique `microsoft/swupdate:1` para la actualización de la imagen.</li><li>Especifique `microsoft/apt:1` para la actualización del paquete.</li></ul>
    | installedCriteria | <ul><li>Especifique el valor de SWVersion para el tipo de actualización `microsoft/swupdate:1`.</li><li>Especifique el valor **name-version**, donde _name_ es el nombre del manifiesto APT y _version_ es la versión del manifiesto APT. Por ejemplo, contoso-iot-edge-1.0.0.0.
    | updateFilePath(s) | Ruta de acceso a los archivos de actualización del equipo


## <a name="review-generated-import-manifest"></a>Revisión de manifiesto de importación generado

Ejemplo:
```json
{
  "updateId": {
    "provider": "Microsoft",
    "name": "Toaster",
    "version": "2.0"
  },
  "updateType": "microsoft/swupdate:1",
  "installedCriteria": "5",
  "compatibility": [
    {
      "deviceManufacturer": "Fabrikam",
      "deviceModel": "Toaster"
    },
    {
      "deviceManufacturer": "Contoso",
      "deviceModel": "Toaster"
    }
  ],
  "files": [
    {
      "filename": "file1.json",
      "sizeInBytes": 7,
      "hashes": {
        "sha256": "K2mn97qWmKSaSaM9SFdhC0QIEJ/wluXV7CoTlM8zMUo="
      }
    },
    {
      "filename": "file2.zip",
      "sizeInBytes": 11,
      "hashes": {
        "sha256": "gbG9pxCr9RMH2Pv57vBxKjm89uhUstD06wvQSioLMgU="
      }
    }
  ],
  "createdDateTime": "2020-10-08T03:32:52.477Z",
  "manifestVersion": "2.0"
}
```

## <a name="import-update"></a>Importación de actualización

[!NOTE]
En las siguientes instrucciones se muestra cómo importar una actualización a través de la interfaz de usuario de Azure Portal. También puede usar las [API de actualización de dispositivos para IOT Hub](https://github.com/Azure/iot-hub-device-update/tree/main/docs/publish-api-reference) para importar una actualización. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a su instancia de IoT Hub con Device Update.

2. En el lado izquierdo de la página, seleccione "Actualizaciones de dispositivos" en "Administración de dispositivos automática".

   :::image type="content" source="media/import-update/import-updates-3.png" alt-text="Importación de actualizaciones" lightbox="media/import-update/import-updates-3.png":::

3. Verá varias pestañas en la parte superior de la pantalla. Seleccione la pestaña Actualizaciones.

   :::image type="content" source="media/import-update/updates-tab.png" alt-text="Actualizaciones" lightbox="media/import-update/updates-tab.png":::

4. Seleccione "+ Import New Update" (Importar nueva actualización) en el encabezado "Preparado para implementar".

   :::image type="content" source="media/import-update/import-new-update-2.png" alt-text="Importación de una nueva actualización" lightbox="media/import-update/import-new-update-2.png":::

5. Seleccione el icono de la carpeta o el cuadro de texto en "Select an Import Manifest File" (Seleccionar un archivo de manifiesto de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione el manifiesto de importación que creó anteriormente mediante el cmdlet de PowerShell. Seleccione el icono de la carpeta o el cuadro de texto en "Select one or more update files" (Seleccionar uno o varios archivos de importación). Verá un cuadro de diálogo para seleccionar archivos. Seleccione los archivos de actualización.

   :::image type="content" source="media/import-update/select-update-files.png" alt-text="Seleccionar archivos de actualización" lightbox="media/import-update/select-update-files.png":::

6. Seleccione el icono de la carpeta o el cuadro de texto en "Select a storage container" (Seleccionar un contenedor de almacenamiento). Luego, seleccione la cuenta de almacenamiento adecuada. El contenedor de almacenamiento se usa para almacenar temporalmente los archivos de actualización.

   :::image type="content" source="media/import-update/storage-account.png" alt-text="Storage Account" lightbox="media/import-update/storage-account.png":::

7. Si ya ha creado un contenedor, puede volver a usarlo (de lo contrario, seleccione "+ Contenedor" para crear contenedor de almacenamiento para las actualizaciones).  Seleccione el contenedor que desee usar y haga clic en "Seleccionar".

   :::image type="content" source="media/import-update/container.png" alt-text="Seleccionar contenedor" lightbox="media/import-update/container.png":::

8. Seleccione "Enviar" para iniciar el proceso de importación.

   :::image type="content" source="media/import-update/publish-update.png" alt-text="Publicar actualización" lightbox="media/import-update/publish-update.png":::

9. Se inicia el proceso de importación y la pantalla cambia a la sección "Historial de importación". Seleccione "Actualizar" para ver el progreso hasta que finalice el proceso de importación (en función del tamaño de la actualización, esta operación puede tardar unos minutos o prolongarse mucho tiempo).

   :::image type="content" source="media/import-update/update-publishing-sequence-2.png" alt-text="Actualizar secuenciación de importación" lightbox="media/import-update/update-publishing-sequence-2.png":::

10. Cuando la columna Estado indique que la importación se ha realizado correctamente, seleccione el encabezado "Ready to Deploy" (Listo para implementar). Debería ver la actualización importada en la lista.

   :::image type="content" source="media/import-update/update-ready.png" alt-text="Estado del trabajo" lightbox="media/import-update/update-ready.png":::

## <a name="next-steps"></a>Pasos siguientes

[Creación de grupos](create-update-group.md)

[Información sobre los conceptos de la importación](import-concepts.md)