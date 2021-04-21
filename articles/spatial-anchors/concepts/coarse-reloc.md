---
title: Relocalización general
description: Aprenda cómo y cuándo usar la relocalización general. La relocalización general le ayuda a encontrar anclajes cercanos.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 01/28/2021
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: d2737f58fa95d1aa45d9952e8b501c1b9be4d1b0
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107600359"
---
# <a name="coarse-relocalization"></a>Relocalización general

La relocalización general es una característica que permite la localización a gran escala proporcionando una respuesta aproximada pero rápida a estas preguntas: 
- *¿Dónde está ahora mi dispositivo?* 
- *¿Qué contenido debo observar?* 
 
La respuesta no es precisa. En este formulario: *Está cerca de estos anclajes. Intente localizar uno*.

La relocalización general funciona mediante el etiquetado de los delimitadores con varias lecturas de sensor en el dispositivo que se usan posteriormente para realizar consultas rápidas. En el caso de los escenarios exteriores, los datos del sensor suelen ser la posición GPS (sistema de posición global) del dispositivo. Cuando el GPS no está disponible o no es confiable, por ejemplo cuando se encuentra en interiores, los datos del sensor están formados por los puntos de acceso Wi-Fi y las balizas Bluetooth dentro del alcance. Los datos recopilados del sensor contribuyen al mantenimiento de un índice espacial que Azure Spatial Anchors usa para determinar rápidamente los anclajes que se encuentran cerca del dispositivo.

## <a name="when-to-use-coarse-relocalization"></a>Cuándo usar la relocalización general

Si tiene previsto controlar más de 35 anclajes espaciales en un espacio mayor que una pista de tenis, probablemente le resultará útil la indexación espacial de relocalización general.

La búsqueda rápida de los anclajes habilitados por la relocalización general está diseñada para simplificar el desarrollo de aplicaciones respaldadas por colecciones a escala mundial de, por ejemplo, millones de anclajes distribuidos geográficamente. La complejidad de la indexación espacial está completamente oculta, lo que le permite centrarse en la lógica de aplicación. Todo el trabajo difícil se realiza en segundo plano mediante Azure Spatial Anchors.

## <a name="using-coarse-relocalization"></a>Uso de la relocalización general

El flujo de trabajo típico para crear y consultar Azure Spatial Anchors con la relocalización general es el siguiente:
1.  Cree y configure un proveedor de huellas digitales de sensor para recopilar los datos de sensor que desee.
2.  Inicie una sesión de Azure Spatial Anchors y cree anclajes. Como la huella digital de sensores está habilitada, los delimitadores se indexan espacialmente mediante la relocalización general.
3.  Consulte los anclajes cercanos mediante la relocalización general a través de los criterios de búsqueda dedicados de la sesión de Spatial Anchors.

Puede consultar uno de los siguientes tutoriales para configurar la relocalización general en la aplicación:
* [Relocalización general en Unity](../how-tos/set-up-coarse-reloc-unity.md)
* [Relocalización general en Objective-C](../how-tos/set-up-coarse-reloc-objc.md)
* [Relocalización general en Swift](../how-tos/set-up-coarse-reloc-swift.md)
* [Relocalización general en Java](../how-tos/set-up-coarse-reloc-java.md)
* [Relocalización general en C++ y NDK](../how-tos/set-up-coarse-reloc-cpp-ndk.md)
* [Relocalización general en C++ y WinRT](../how-tos/set-up-coarse-reloc-cpp-winrt.md)

## <a name="sensors-and-platforms"></a>Sensores y plataformas

### <a name="platform-availability"></a>Disponibilidad de la plataforma

Puede enviar los tipos de datos de sensor siguientes al servicio de anclaje:

* Posición del GPS: latitud, longitud, altitud.
* Intensidad de la señal de los puntos de acceso Wi-Fi dentro del alcance.
* Intensidad de la señal de las balizas Bluetooth dentro del alcance.

En esta tabla se resume la disponibilidad de los datos del sensor en las plataformas admitidas y se proporciona información que debe tener en cuenta:

|                 | HoloLens | Android | iOS |
|-----------------|----------|---------|-----|
| **GPS**         | No<sup>1</sup>  | Sí<sup>2</sup> | Sí<sup>3</sup> |
| **Wi-Fi**        | Sí<sup>4</sup> | Sí<sup>5</sup> | No  |
| **Balizas de BLE** | Sí<sup>6</sup> | Sí<sup>6</sup> | Sí<sup>6</sup>|


<sup>1</sup> Se puede asociar un dispositivo GPS externo a HoloLens. Póngase en contacto con [nuestro soporte técnico](../spatial-anchor-support.md) si quiere usar HoloLens con un dispositivo de seguimiento GPS.<br/>
<sup>2</sup> Compatible con las API [LocationManager][3] (tanto GPS como NETWORK).<br/>
<sup>3</sup> Compatible con las API [CLLocationManager][4].<br/>
<sup>4</sup> Compatible con una frecuencia de aproximadamente un examen cada 3 segundos. <br/>
<sup>5</sup> A partir del nivel 28 de API, los exámenes Wi-Fi se limitan a 4 llamadas cada 2 minutos. A partir de Android 10, puede deshabilitar esta limitación en el menú **Configuración de desarrollador**. Para más información, consulte la [documentación de Android][5].<br/>
<sup>6</sup> Limitado a [Eddystone][1] y [iBeacon][2].

### <a name="which-sensor-to-enable"></a>Qué sensor habilitar

La elección del sensor es específica de la aplicación que se desarrolla y de la plataforma.
En el diagrama siguiente se proporciona un punto de partida para determinar la combinación de sensores que se puede habilitar según el escenario de localización:

![Diagrama que muestra sensores habilitados para distintos escenarios.](media/coarse-relocalization-enabling-sensors.png)

En las secciones siguientes se proporciona más información sobre las ventajas y las limitaciones de cada tipo de sensor.

### <a name="gps"></a>GPS

GPS es la opción para escenarios de exteriores.
Al usar el GPS en la aplicación, tenga en cuenta que las lecturas que proporciona el hardware suelen ser:

* Frecuencia asincrónica y baja (menos de 1 Hz).
* No confiable/ruidoso (promedio, desviación estándar de 7 m).

En general, tanto el sistema operativo del dispositivo como Spatial Anchors realizarán algo de filtrado y extrapolación en la señal de GPS sin procesar en un intento de mitigar estos problemas. Este procesamiento adicional requiere tiempo para la convergencia, por lo que debe intentar lo siguiente para obtener mejores resultados:

* Crear un proveedor de huellas digitales del sensor lo antes posible en su aplicación.
* Mantener el proveedor de huellas digitales del sensor activo entre varias sesiones.
* Compartir el proveedor de huellas digitales del sensor entre varias sesiones.

Los dispositivos GPS de nivel de consumidor suelen ser imprecisos. Un estudio de [Zandenbergen y Barbeau (2011)][6] indica que la precisión media de los teléfonos móviles que tienen GPS asistido (A-GPS) es aproximadamente de 7 metros. Un valor lo suficientemente alto para ignorarlo. Para tener en cuenta estos errores de medición, el servicio trata los anclajes como distribuciones de probabilidad en el espacio GPS. Por tanto, un anclaje es la región del espacio que más probablemente (con más del 95 % de confianza) contiene su posición GPS verdadera y desconocida.

El mismo razonamiento se aplica cuando se realiza una consulta mediante GPS. El dispositivo se representa como otra región de confianza espacial en torno a su verdadera posición GPS desconocida. Descubrir los anclajes cercanos significa encontrar los anclajes con regiones de confianza *suficientemente cercanas* a la región de confianza del dispositivo, como se ilustra en la imagen siguiente:

![Diagrama que muestra la búsqueda de candidatos de anclaje mediante GPS.](media/coarse-reloc-gps-separation-distance.png)

### <a name="wi-fi"></a>Wi-Fi

En HoloLens y Android, la intensidad de la señal Wi-Fi puede ser una opción correcta para habilitar la relocalización general en interiores.
La ventaja es la posible disponibilidad inmediata de los puntos de acceso Wi-Fi (habituales, por ejemplo, en oficinas o centros comerciales) sin necesidad de configuración adicional.

> [!NOTE]
> iOS no proporciona ninguna API para leer la intensidad de la señal Wi-Fi, por lo que no se puede usar para la relocalización general habilitada a través de Wi-Fi.

Al usar Wi-Fi en la aplicación, tenga en cuenta que las lecturas que proporciona el hardware suelen ser:

* Frecuencia asincrónica y baja (menos de 0,1 Hz).
* Posible limitación en el nivel del sistema operativo.
* No confiable/ruidoso (promedio, desviación estándar de 3 dBm).

Spatial Anchors intentará crear un mapa de intensidad de señal Wi-Fi filtrado durante una sesión para intentar mitigar estos problemas. Para obtener unos resultados óptimos, intente lo siguiente:

* Cree la sesión antes de colocar el primer anclaje.
* Mantenga la sesión activa el máximo tiempo posible. (Es decir, cree todos los anclajes y la consulta en una sesión).

### <a name="bluetooth-beacons"></a>Balizas Bluetooth
<a name="beaconsDetails"></a>

La implementación adecuada de balizas Bluetooth es una buena solución para escenarios de relocalización general a gran escala en interiores, donde no hay GPS o es inexacto. También es el único método para interiores que se admite en las tres plataformas.

Las balizas suelen ser dispositivos versátiles en los que se puede configurar todo, incluidos los UUID y las direcciones MAC. Azure Spatial Anchors espera que las balizas se identifiquen de forma única por su UUID. Si no garantiza esta unicidad, es probable que obtenga resultados incorrectos. Para obtener mejores resultados:

* Asigne UUID únicos a las balizas.
* Implemente las balizas de forma que abarquen el espacio de manera uniforme y de modo que se pueda acceder al menos a tres balizas desde cualquier punto del espacio.
* Pase la lista de UUID de baliza únicos al proveedor de huellas digitales del sensor.

Las señales de radio, como las de Bluetooth, se ven afectadas por los obstáculos y pueden interferir con otras. Por tanto, puede ser difícil adivinar si el espacio se abarca de forma uniforme. Para garantizar una mejor experiencia del cliente, se recomienda probar manualmente la cobertura de las balizas. Para realizar una prueba puede recorrer el espacio con los dispositivos candidatos y una aplicación que muestre Bluetooth dentro del alcance. Mientras prueba la cobertura, asegúrese de que puede alcanzar al menos tres balizas desde cualquier posición estratégica del espacio. La configuración de demasiadas balizas puede producir más interferencias entre ellas y quizás no mejore la precisión de la relocalización general.

Las balizas Bluetooth suelen tener una cobertura de 80 metros si no hay obstáculos en el espacio.
Por tanto, en el caso de un espacio sin grandes obstáculos, puede implementar balizas en un patrón de cuadrícula cada 40 metros.

Una baliza que se quede sin batería afectará negativamente a los resultados, por lo que debe asegurarse de supervisar la implementación periódicamente en busca de baterías bajas o agotadas.

Azure Spatial Anchors solo realizará un seguimiento de las balizas Bluetooth que se encuentran en la lista de UUID conocidos de proximidad de las balizas. No obstante, las balizas malintencionadas programadas para tener UUID en la lista de permitidos pueden afectar negativamente a la calidad del servicio. Por lo tanto, los mejores resultados se obtendrán en espacios seleccionados en los que pueda controlar la implementación de las balizas.

### <a name="sensor-accuracy"></a>Precisión del sensor

La precisión de la señal del GPS, tanto durante la creación del anclaje como en las consultas, tiene una gran influencia sobre el conjunto de anclajes devueltos. Por el contrario, las consultas basadas en Wi-Fi o balizas considerarán todos los anclajes que tengan al menos un punto de acceso o una baliza en común con la consulta. En ese sentido, el resultado de una consulta basada en Wi-Fi o balizas estará determinado principalmente por el alcance físico de los puntos de acceso o las balizas y los obstáculos ambientales.
En esta tabla se calcula el espacio de búsqueda esperado para cada tipo de sensor:

| Sensor      | Radio del espacio de búsqueda (aproximado) | Detalles |
|-------------|:-------:|---------|
| **GPS**         | De 20 a 30 m | Determinado por la incertidumbre del GPS entre otros factores. Los números indicados se estiman para la precisión media del GPS de los teléfonos móviles con A-GPS: 7 metros. |
| **Wi-Fi**        | De 50 a 100 m | Determinado por el intervalo de puntos de acceso inalámbrico. Depende de la frecuencia, la intensidad del transmisor, los obstáculos físicos, las interferencias, etc. |
| **Balizas de BLE** |  70 m | Determinado por el intervalo de la señalización. Depende de la frecuencia, la intensidad de transmisión, los obstáculos físicos, las interferencias, etc. |

<!-- Reference links in article -->
[1]: https://developer.estimote.com/eddystone/
[2]: https://developer.apple.com/ibeacon/
[3]: https://developer.android.com/reference/android/location/LocationManager
[4]: https://developer.apple.com/documentation/corelocation/cllocationmanager?language=objc
[5]: https://developer.android.com/guide/topics/connectivity/wifi-scan
[6]: https://www.cambridge.org/core/journals/journal-of-navigation/article/positional-accuracy-of-assisted-gps-data-from-highsensitivity-gpsenabled-mobile-phones/E1EE20CD1A301C537BEE8EC66766B0A9
