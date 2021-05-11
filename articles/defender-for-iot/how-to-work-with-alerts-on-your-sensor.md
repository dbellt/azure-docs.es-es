---
title: Descripción de alertas de sensor
description: Use alertas para mejorar la seguridad y el funcionamiento de la red.
ms.date: 3/29/2021
ms.topic: how-to
ms.openlocfilehash: 353f7fb0da85a0ffa737dc5c4afa70763b2fb185
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108208948"
---
# <a name="about-sensor-alerts"></a>Acerca de las alertas del sensor

Las alertas le ayudan a mejorar la seguridad y el funcionamiento de la red. Las alertas proporcionan información sobre lo siguiente:

- Desviaciones de la actividad de red autorizada

- Anomalías en las operaciones y los protocolos

- Tráfico de malware sospechoso

:::image type="content" source="media/how-to-work-with-alerts-sensor/address-scan-detected-screen.png" alt-text="Detección de análisis de direcciones":::

Las opciones de administración de alertas permiten a los usuarios:

- Indicar a los sensores que consideren la actividad detectada como tráfico autorizado.

- Confirmar la revisión de la alerta.

- Indicar a los sensores que desactiven los eventos detectados con dispositivos idénticos y tráfico comparable.

Hay disponibles herramientas adicionales que le ayudarán a mejorar y acelerar la investigación de alertas. Por ejemplo:

  - Agregue comentarios informativos para los revisores de alertas.

  - Cree grupos de alertas para su visualización en las soluciones de SOC. 

  - Busque alertas específicas; revise los archivos de PCAP relacionados; vea los dispositivos detectados y otros dispositivos conectados en el mapa del dispositivo o envíe los detalles de la alerta a los sistemas asociados.

  - Reenvíe alertas a los proveedores asociados: sistemas SIEM, sistemas MSSP, etc.

## <a name="alerts-and-engines"></a>Alertas y motores

Las alertas se desencadenan cuando los motores de los sensores detectan cambios en el tráfico de red y el comportamiento que requieren su atención. En este artículo se describe el tipo de alertas que desencadena cada motor.

| Tipo de alerta | Descripción |
|-|-|
| Alertas de infracción de directivas | Se desencadena cuando el motor de infracción de directivas detecta una desviación del tráfico previamente aprendido. Por ejemplo: <br /> - Se detecta un nuevo dispositivo.  <br /> - Se detecta una nueva configuración en un dispositivo. <br /> - Un dispositivo no definido como dispositivo de programación realiza un cambio de programación. <br /> - Se produce un cambio en la versión del firmware. |
| Alertas de infracción de protocolos | Se desencadenan cuando el motor de infracción de protocolos detecta valores de campo o estructuras de paquetes que no cumplen la especificación del protocolo. | 
| Alertas operativas | Se desencadenan cuando el motor operativo detecta incidentes operativos de red o un dispositivo que no funciona correctamente. Por ejemplo, un dispositivo de red se detuvo con un comando Stop PLC, o una interfaz en un sensor dejó de supervisar el tráfico. |
| Alertas de malware | Se desencadenan cuando el motor de malware detecta actividades de red malintencionadas. Por ejemplo, el motor detecta un ataque conocido, como Conficker. |
| Alertas de anomalías | Se desencadenan cuando el motor de anomalías detecta una desviación. Por ejemplo, un dispositivo realiza exámenes de red, pero no se define como un dispositivo de detección. |

Hay herramientas disponibles para habilitar y deshabilitar los motores de los sensores. No se desencadenan alertas desde motores que están deshabilitados. Vea [Control del tráfico que se supervisa](how-to-control-what-traffic-is-monitored.md).

## <a name="alerts-and-sensor-reporting"></a>Alertas y notificaciones de los sensores

La actividad que se refleja en las alertas se calcula al generar informes de minería de datos, evaluación de riesgos y vectores de ataque. Cuando se administran estos eventos, el sensor actualiza los informes en consecuencia.

Por ejemplo:

  - La conectividad no autorizada entre un dispositivo de una subred definida y los dispositivos que se encuentran fuera de la subred (pública) aparecerá en el informe de minería de datos sobre la *actividad de Internet* y en la sección *Conexiones a Internet* de la evaluación de riesgos. Una vez que estos dispositivos están autorizados (aprendidos), se calculan en la generación de estos informes.

  - Los eventos de malware detectados en los dispositivos de red se notifican en los informes de evaluación de riesgos. Cuando las alertas sobre eventos de malware se *desactivan*, los dispositivos afectados no se calcularán en el informe de evaluación de riesgos.

## <a name="next-steps"></a>Pasos siguientes

[Modos de aprendizaje y aprendizaje de TI inteligente](how-to-control-what-traffic-is-monitored.md#learning-and-smart-it-learning-modes)
[Visualización de la información proporcionada en las alertas](how-to-view-information-provided-in-alerts.md)
[Administrar eventos de alerta](how-to-manage-the-alert-event.md)
[Aceleración de los flujos de trabajo de alertas](how-to-accelerate-alert-incident-response.md)
[Tipos y descripciones de alertas](alert-engine-messages.md)
