---
title: 'Importación de actualizaciones a Device Update para IoT Hub: esquema y otra información | Microsoft Docs'
description: Esquema y otra información relacionada (incluidos los objetos) que se usan al importar actualizaciones a Device Update para IoT Hub.
author: andrewbrownmsft
ms.author: andbrown
ms.date: 2/25/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: 1d58d0b0ecb614779b2fd046a44ad16afd8ebeb9
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955999"
---
# <a name="importing-updates-into-device-update-for-iot-hub---schema-and-other-information"></a>Importación de actualizaciones a Device Update para IoT Hub: esquema y otra información
Si quiere importar una actualización a Device Update para IoT Hub, asegúrese de revisar primero los [conceptos](import-concepts.md) y la [guía paso a paso](import-update.md). Si le interesan los detalles del esquema que se usa al construir un manifiesto de importación, así como la información sobre los objetos relacionados, vea las secciones que se presentan a continuación.

## <a name="import-manifest-schema"></a>Importación del esquema de manifiesto

| Nombre | Tipo | Descripción | Restricciones |
| --------- | --------- | --------- | --------- |
| UpdateId | Objecto `UpdateId` | Identidad de actualización. |
| UpdateType | string | Tipo de actualización: <br/><br/> * Especifique `microsoft/apt:1` al realizar una actualización basada en paquetes mediante un agente de referencia.<br/> * Especifique `microsoft/swupdate:1` al realizar una actualización basada en imágenes mediante un agente de referencia.<br/> * Especifique `microsoft/simulator:1` al usar un simulador de agente de ejemplo.<br/> * Especifique un tipo personalizado si va a desarrollar un agente personalizado. | Formato: <br/> `{provider}/{type}:{typeVersion}`<br/><br/> Un máximo de 32 caracteres en total. |
| InstalledCriteria | string | Cadena que interpreta el agente para determinar si la actualización se aplicó correctamente:  <br/> * Especifique el **valor** de SWVersion para el tipo de actualización `microsoft/swupdate:1`.<br/> * Especifique `{name}-{version}` para el tipo de actualización `microsoft/apt:1`, cuyo nombre y versión se obtienen del archivo de APT.<br/> * Especifique una cadena personalizada si va a desarrollar un agente personalizado.<br/> | 64 caracteres como máximo |
| Compatibilidad | Matriz de `CompatibilityInfo` [objetos](#compatibilityinfo-object) | Información de compatibilidad del dispositivo compatible con esta actualización. | Un máximo de 10 elementos |
| CreatedDateTime | date/time | Fecha y hora en que se creó la carpeta. | Formato de fecha y hora delimitado que sigue la norma ISO 8601, en UTC |
| ManifestVersion | string | Versión del esquema de manifiesto de importación. Especifique `2.0`, que será compatible tanto con la interfaz de `urn:azureiot:AzureDeviceUpdateCore:1`, como con la interfaz de `urn:azureiot:AzureDeviceUpdateCore:4`. | Debe ser `2.0` |
| Archivos | Matriz de objetos `File` | Archivos de carga de Update | Un máximo de cinco archivos |

## <a name="updateid-object"></a>Objeto UpdateId

| Nombre | Tipo | Descripción | Restricciones |
| --------- | --------- | --------- | --------- |
| Proveedor | string | Parte del proveedor de la identidad de actualización. | De 1 a 64 caracteres, alfanuméricos, puntos y guiones. |
| Nombre | string | Parte del nombre de la identidad de actualización. | De 1 a 64 caracteres, alfanuméricos, puntos y guiones. |
| Versión | version | Parte de la versión de la identidad de actualización. | Número de versión de 2 a 4 partes separadas por puntos entre 0 y 2147483647. Se quitarán los ceros iniciales. |

## <a name="file-object"></a>Objeto File

| Nombre | Tipo | Descripción | Restricciones |
| --------- | --------- | --------- | --------- |
| Nombre de archivo | string | Nombre del archivo | Debe ser único en una actualización |
| SizeInBytes | Int64 | Tamaño del archivo en bytes | Máximo de 800 MB por archivo individual, o 800 MB colectivamente por actualización |
| Códigos hash | Objecto `Hashes` | Objeto JSON que contiene los hash del archivo |

## <a name="compatibilityinfo-object"></a>Objeto CompatibilityInfo

| Nombre | Tipo | Descripción | Restricciones |
| --- | --- | --- | --- |
| DeviceManufacturer | string | Fabricante del dispositivo con el que la actualización es compatible. | De 1 a 64 caracteres, alfanuméricos, puntos y guiones. |
| DeviceModel | string | Modelo del dispositivo con el que la actualización es compatible. | De 1 a 64 caracteres, alfanuméricos, puntos y guiones. |

## <a name="hashes-object"></a>Objeto Hashes

| Nombre | Obligatorio | Tipo | Descripción |
| --------- | --------- | --------- | --------- |
| Sha256 | True | string | Hash codificado en base64 del archivo con el algoritmo SHA-256. |

## <a name="example-import-request-body"></a>Ejemplo de cuerpo de solicitud de importación

Si usa la salida del manifiesto de importación de ejemplo de la página [Adición de una nueva actualización](./import-update.md#review-the-generated-import-manifest) y desea llamar directamente a la [API REST](/rest/api/deviceupdate/updates) Device Update para realizar la importación, el cuerpo de la solicitud correspondiente debe tener el siguiente aspecto:

```json
{
  "importManifest": {
    "url": "http://<your Azure Storage location file path>/importManifest.json",
    "sizeInBytes": <size of import manifest file>,
    "hashes": {
      "sha256": "<hash of import manifest file>"
    }
  },
  "files": [
    {
      "filename": "file1.json",
      "url": "http://<your Azure Storage location file path>/file1.json"
    },
    {
          "filename": "file2.zip",
          "url": "http://<your Azure Storage location file path>/file2.zip"
    },
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más Información sobre los [conceptos de la importación](./import-concepts.md).

Cuando está listo, pruebe la [guía de procedimientos de importación](./import-update.md), que le guiará a través del proceso de importación paso a paso.