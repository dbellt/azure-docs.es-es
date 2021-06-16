---
title: API de reproducción de Azure Kinect
description: Aprenda a usar el SDK del sensor de Azure Kinect para abrir un archivo de grabación mediante la API de reproducción.
author: martinekuan
ms.author: martinek
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, sensor, sdk, depth, rgb, record, playback, matroska, mkv
ms.openlocfilehash: 61d7f850554207effeff14f890efe1dd59ab109b
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747012"
---
# <a name="the-azure-kinect-playback-api"></a>API de reproducción de Azure Kinect

El SDK del sensor proporciona una API para grabar los datos del dispositivo en un archivo Matroska (.mkv). El formato de contenedor Matroska almacena pistas de vídeo, muestras de IMU e información de calibración del dispositivo. Las grabaciones se pueden generar mediante la utilidad de línea de comandos ([k4arecorder](record-sensor-streams-file.md)) proporcionada. Las grabaciones también se pueden personalizar y registrar directamente con la API de grabación.

Para más información sobre la API de grabación, consulte [`k4a_record_create()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gae14f4181e9688e710d1c80b215413831.html#gae14f4181e9688e710d1c80b215413831).

Para más información sobre las especificaciones de formato de archivo Matroska, consulte la página [Formato de archivo de grabación](record-file-format.md).

## <a name="use-the-playback-api"></a>Uso de la API de reproducción

Los archivos de grabación pueden abrirse mediante la API de reproducción. La API de reproducción proporciona acceso a los datos del sensor en el mismo formato que el resto del SDK del sensor.

### <a name="open-a-record-file"></a>Apertura de un archivo de grabación

En el ejemplo siguiente, se abre una grabación mediante [`k4a_playback_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gacb254ac941b2ab3c202ca68f4537f368.html#gacb254ac941b2ab3c202ca68f4537f368), se imprime la longitud de la grabación y, luego, se cierra el archivo con [`k4a_playback_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga76f415f2076f1c8c544e094a649306ff.html#ga76f415f2076f1c8c544e094a649306ff).

```C
k4a_playback_t playback_handle = NULL;
if (k4a_playback_open("recording.mkv", &playback_handle) != K4A_RESULT_SUCCEEDED)
{
    printf("Failed to open recording\n");
    return 1;
}

uint64_t recording_length = k4a_playback_get_last_timestamp_usec(playback_handle);
printf("Recording is %lld seconds long\n", recording_length / 1000000);

k4a_playback_close(playback_handle);
```

### <a name="read-captures"></a>Lectura de las capturas

Una vez abierto el archivo, podemos empezar a leer las capturas de la grabación. En el siguiente ejemplo se leerá cada una de las capturas del archivo.

```C
k4a_capture_t capture = NULL;
k4a_stream_result_t result = K4A_STREAM_RESULT_SUCCEEDED;
while (result == K4A_STREAM_RESULT_SUCCEEDED)
{
    result = k4a_playback_get_next_capture(playback_handle, &capture);
    if (result == K4A_STREAM_RESULT_SUCCEEDED)
    {
        // Process capture here
        k4a_capture_release(capture);
    }
    else if (result == K4A_STREAM_RESULT_EOF)
    {
        // End of file reached
        break;
    }
}
if (result == K4A_STREAM_RESULT_FAILED)
{
    printf("Failed to read entire recording\n");
    return 1;
}
```

### <a name="seek-within-a-recording"></a>Búsqueda en una grabación

Una vez que se llega al final del archivo, es posible que quiera retroceder y leerlo de nuevo. Este proceso puede realizarse leyendo hacia atrás con [`k4a_playback_get_previous_capture()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga54732e3aa0717e1ca4eb76ee385e878c.html#ga54732e3aa0717e1ca4eb76ee385e878c), pero podría ser muy lento en función de la longitud de la grabación.
Puede usar mejor la función [`k4a_playback_seek_timestamp()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaea748994a121543bd77f90417cf428f6.html#gaea748994a121543bd77f90417cf428f6) para desplazarse a un punto específico del archivo.

En este ejemplo, se especifican las marcas de tiempo en microsegundos para buscar varios puntos en el archivo.

```C
// Seek to the beginning of the file
if (k4a_playback_seek_timestamp(playback_handle, 0, K4A_PLAYBACK_SEEK_BEGIN) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to the end of the file
if (k4a_playback_seek_timestamp(playback_handle, 0, K4A_PLAYBACK_SEEK_END) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to 10 seconds from the start
if (k4a_playback_seek_timestamp(playback_handle, 10 * 1000000, K4A_PLAYBACK_SEEK_BEGIN) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to 10 seconds from the end
if (k4a_playback_seek_timestamp(playback_handle, -10 * 1000000, K4A_PLAYBACK_SEEK_END) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}
```

### <a name="read-tag-information"></a>Lectura de la información de etiqueta

Las grabaciones también pueden contener varios metadatos, como el número de serie y las versiones de firmware del dispositivo. Estos metadatos se almacenan en etiquetas de grabación, a las que se puede acceder con la función [`k4a_playback_get_tag()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga320f966fc89b4ba0d758f787f70d5143.html#ga320f966fc89b4ba0d758f787f70d5143).

```C
// Print the serial number of the device used to record
char serial_number[256];
size_t serial_number_size = 256;
k4a_buffer_result_t buffer_result = k4a_playback_get_tag(playback_handle, "K4A_DEVICE_SERIAL_NUMBER", &serial_number, &serial_number_size);
if (buffer_result == K4A_BUFFER_RESULT_SUCCEEDED)
{
    printf("Device serial number: %s\n", serial_number);
}
else if (buffer_result == K4A_BUFFER_RESULT_TOO_SMALL)
{
    printf("Device serial number too long.\n");
}
else
{
    printf("Tag does not exist. Device serial number was not recorded.\n");
}
```

### <a name="record-tag-list"></a>Lista de etiquetas de grabación

A continuación, se muestra una lista de todas las etiquetas predeterminadas que se pueden incluir en un archivo de grabación. Muchos de estos valores están disponibles como parte de la estructura [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html) y se pueden leer con la función [`k4a_playback_get_record_configuration()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaab54a85c1f1e98d170d009042b449255.html#gaab54a85c1f1e98d170d009042b449255).

Si no existe una etiqueta, se supone que tiene el valor predeterminado.

| Nombre de etiqueta                     | Valor predeterminado      | [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html) Campo | Notas     |
|------------------------------|--------------------|--------------------------------------|----------------------------------------------------------------------------------------------------------------|
| `K4A_COLOR_MODE`             | "OFF"              | `color_format` / `color_resolution`  | Valores posibles: "OFF", "MJPG_1080P", "NV12_720P", "YUY2_720P", etc.                                      |
| `K4A_DEPTH_MODE`             | "OFF"              | `depth_mode` / `depth_track_enabled` | Valores posibles: "OFF, "NFOV_UNBINNED", "PASSIVE_IR", etc.                                                |
| `K4A_IR_MODE`                | "OFF"              | `depth_mode` / `ir_track_enabled`    | Valores posibles: "OFF", "ACTIVE", "PASSIVE"                                                                    |
| `K4A_IMU_MODE`               | "OFF"              | `imu_track_enabled`                  | Valores posibles: "ON", "OFF"                                                                                   |
| `K4A_CALIBRATION_FILE`       | "calibration.json" | N/D                                  | Consulte [`k4a_device_get_raw_calibration()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga8c4e46642cee3115aeb0b33e2b43b24f.html#ga8c4e46642cee3115aeb0b33e2b43b24f). |
| `K4A_DEPTH_DELAY_NS`         | "0"                | `depth_delay_off_color_usec`         | Valor almacenado en nanosegundos; la API proporciona microsegundos.                                                        |
| `K4A_WIRED_SYNC_MODE`        | "STANDALONE"       | `wired_sync_mode`                    | Valores posibles: "STANDALONE", "MASTER", "SUBORDINATE"                                                         |
| `K4A_SUBORDINATE_DELAY_NS`   | "0"                | `subordinate_delay_off_master_usec`  | Valor almacenado en nanosegundos; la API proporciona microsegundos.                                                        |
| `K4A_COLOR_FIRMWARE_VERSION` | ""                 | N/D                                  | Versión de firmware del color del dispositivo, por ejemplo "1.x.xx".                                                            |
| `K4A_DEPTH_FIRMWARE_VERSION` | ""                 | N/D                                  | Versión de firmware de la profundidad del dispositivo, por ejemplo "1.x.xx".                                                            |
| `K4A_DEVICE_SERIAL_NUMBER`   | ""                 | N/D                                  | Número de serie del dispositivo de grabación.                                                                                 |
| `K4A_START_OFFSET_NS`        | "0"                | `start_timestamp_offset_usec`        | Consulte a continuación [Sincronización de las marcas de tiempo](record-playback-api.md#timestamp-synchronization).                       |
| `K4A_COLOR_TRACK`            | None               | N/D                                  | Consulte [Formato del archivo de grabación: identificación de las pistas](record-file-format.md#identifying-tracks).                     |
| `K4A_DEPTH_TRACK`            | None               | N/D                                  | Consulte [Formato del archivo de grabación: identificación de las pistas](record-file-format.md#identifying-tracks).                     |
| `K4A_IR_TRACK`               | None               | N/D                                  | Consulte [Formato del archivo de grabación: identificación de las pistas](record-file-format.md#identifying-tracks).                     |
| `K4A_IMU_TRACK`              | None               | N/D                                  | Consulte [Formato del archivo de grabación: identificación de las pistas](record-file-format.md#identifying-tracks).                     |

## <a name="timestamp-synchronization"></a>Sincronización de las marcas de tiempo

El formato Matroska requiere que las grabaciones comiencen con una marca de tiempo de cero. Al [sincronizar las cámaras de forma externa](record-external-synchronized-units.md), la primera marca de tiempo de cada dispositivo puede ser distinta de cero.

Para conservar las marcas de tiempo originales de los dispositivos entre grabación y reproducción, el archivo almacena un desplazamiento que se aplica a las marcas de tiempo.

La etiqueta `K4A_START_OFFSET_NS` se usa para especificar un desplazamiento de la marca de tiempo de forma que los archivos se puedan volver a sincronizar después de la grabación. Este desplazamiento se puede agregar a cada marca de tiempo del archivo para reconstruir las marcas de tiempo del dispositivo originales.

El desplazamiento inicial también está disponible en la estructura [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html).
