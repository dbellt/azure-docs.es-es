---
title: Administrar eventos de alerta
description: Administrar los eventos de alerta detectados en la red.
ms.date: 12/07/2020
ms.topic: how-to
ms.openlocfilehash: 124584026bd9ee1662ab39130b5b4dea8b61896c
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108209074"
---
# <a name="manage-alert-events"></a>Administrar eventos de alerta

Las opciones siguientes están disponibles para administrar eventos de alerta:

 | Acción | Descripción |
 |--|--|
 | **Learn** | Autorice el evento detectado. Para obtener más información, vea [Acerca del aprendizaje y el desaprendizaje de eventos](#about-learning-and-unlearning-events). |
 | **Confirmación** | Oculte la alerta una vez para el evento detectado. La alerta se volverá a desencadenar si el evento se vuelve a detectar. Para obtener más información, vea [Acerca de la confirmación y el desconocimiento de eventos](#about-acknowledging-and-unacknowledging-events). |
 | **Silencio** | Omita continuamente la actividad con dispositivos idénticos y tráficos comparables. Para obtener más información, vea [Acerca del silenciamiento y el no silenciamiento de eventos](#about-muting-and-unmuting-events). |
 
También puede exportar información de alertas.
## <a name="about-learning-and-unlearning-events"></a>Acerca del aprendizaje y el no aprendizaje de eventos

Los eventos que indican desviaciones de la red aprendida podrían reflejar cambios de red válidos. Cuando quiera aprobar estos cambios, puede indicar a Defender para IoT que *aprenda* el comportamiento. Algunos ejemplos pueden incluir:

- Se ha detectado una nueva actividad en el dispositivo existente. Por ejemplo, un dispositivo autorizado ha intentado acceder a un nuevo recurso en otro dispositivo.

- Se ha agregado un nuevo dispositivo a la red.  

- Se han producido cambios en la versión del firmware después de procedimientos de mantenimiento estándar.

- Un nuevo dispositivo ha realizado una operación de lectura o escritura en un controlador de destino.

- Un nuevo dispositivo realiza una operación de lectura o escritura en un controlador de destino y debe definirse como dispositivo de programación.

- Se lleva a cabo una nueva digitalización legítima y el dispositivo debe definirse como dispositivo de digitalización.


:::image type="content" source="media/how-to-work-with-alerts-sensor/detected.png" alt-text="La ventana de examen de dirección detectada.":::

La actividad que se refleja en las alertas se calcula al generar informes de minería de datos, evaluación de riesgos y vectores de ataque. Cuando se administran estos eventos, el sensor actualiza los informes en consecuencia.

Al seleccionar **Learn**, el sensor tiene en cuenta el tráfico, las configuraciones o las actividades válidas. Esto significa que ya no se desencadenarán alertas para el evento. También significa que el evento no se calculará cuando el sensor genere la evaluación de riesgos, el vector de ataque y otros informes.

Los eventos aprendidos se pueden dejar de reconocer. Cuando el sensor desconozca los eventos, volverá a desencadenar las alertas relacionadas con dicho evento.

## <a name="about-acknowledging-and-unacknowledging-events"></a>Acerca de la confirmación y el desconocimiento de eventos

En determinadas situaciones, es posible que no desee que un sensor aprenda un evento detectado o que la opción no esté disponible. En su lugar, es posible que el incidente requiera mitigación. Por ejemplo:

- **Mitigar una configuración de red o un dispositivo**: Recibirá una alerta que indica que se ha detectado un nuevo dispositivo en la red. Al investigar, descubre que el dispositivo es un dispositivo de red no autorizado. Controla el incidente desconectando el dispositivo de la red.
- **Actualizar una configuración del sensor**: Recibirá una alerta que indica que un servidor ha iniciado un número excesivo de conexiones remotas. Esta alerta se desencadena porque los umbrales de anomalías del sensor se definieron para desencadenar alertas por encima de un determinado número de sesiones en un minuto. Para controlar el incidente, actualice los umbrales.

Después de llevar a cabo la mitigación o la investigación, puede indicar al sensor que oculte la alerta seleccionando **Confirmación**. La alerta se volverá a desencadenar si el evento se vuelve a detectar.

Para desactivar la alerta:

  - Seleccione **Confirmación**.

Para volver a ver la alerta:

  - Acceda a la alerta y seleccione **Desconocer**.

Desconozca alertas si se requiere más investigación.

## <a name="about-muting-and-unmuting-events"></a>Acerca de los eventos de silenciamiento y no silenciamiento

En determinadas circunstancias, es posible que desee indicar al sensor que omita un escenario específico en la red. Por ejemplo:

  - El motor **Anomaly** desencadena una alerta sobre un pico en el ancho de banda entre dos dispositivos, pero el pico es válido para estos dispositivos.

  - El motor de **infracción de protocolo** desencadena una alerta en una desviación de protocolo detectada entre dos dispositivos, pero la desviación es válida entre los dispositivos.

En estas situaciones, el aprendizaje no está disponible. Cuando no se puede realizar el aprendizaje y desea suprimir la alerta y eliminar el dispositivo al calcular los riesgos y los vectores de ataque, puede silenciar el evento de alerta en su lugar.

> [!NOTE] 
> No se puede deshacer el silenciamiento de los eventos en los que se define un dispositivo de Internet como origen o destino.

### <a name="what-alert-activity-is-muted"></a>¿Qué actividad de alerta se silencia?

Un escenario silenciado incluye los dispositivos de red y el tráfico detectado para un evento. El título de la alerta describe el tráfico que se está silenciando.

El dispositivo o los dispositivos que se están silenciando se mostrarán como una imagen en la alerta. Si se muestran dos dispositivos, se silencia el tráfico del que se ha recibido la alerta entre ellos.

**Ejemplo 1**

Cuando un evento está silenciado, se omite siempre que el principal (origen) envía al secundario (destino) un código de función no válido tal y como lo define el proveedor.

:::image type="content" source="media/how-to-work-with-alerts-sensor/secondary-device-connected.png" alt-text="Dispositivo secundario recibido.":::

**Ejemplo 2**

Cuando se silencia un evento, se omite cada vez que el origen envía un encabezado HTTP con contenido no válido, independientemente del destino.

:::image type="content" source="media/how-to-work-with-alerts-sensor/illegal-http-header-content.png" alt-text="Captura de pantalla del contenido del encabezado HTTP no válido.":::

**Después de silenciar un evento:**

- La alerta será accesible en la vista de alertas **confirmadas** hasta que no esté silenciada.

- La acción de silenciar aparecerá en la **escala de tiempo del evento**.

  :::image type="content" source="media/how-to-work-with-alerts-sensor/muted-event-notification-screenshot.png" alt-text="Evento detectado y silenciado.":::

- El sensor volverá a calcular los dispositivos al generar la evaluación de riesgos, el vector de ataque y otros informes. Por ejemplo, si ha silenciado una alerta que ha detectado tráfico malintencionado en un dispositivo, ese dispositivo no se calculará en el informe de evaluación de riesgos.

**Para silenciar y deshacer el silenciamiento de una alerta:**

- Seleccione el icono **Silenciar** de la alerta.

**Para ver las alertas silenciadas:**

1. Seleccione la opción **Confirmado** en la pantalla principal de **Alertas**.

2. Mantenga el mouse sobre una alerta para ver si está silenciada.  

## <a name="export-alert-information"></a>Exportación de la información de alertas

Exporte la información de alertas a un archivo .csv. Puede exportar la información de todas las alertas detectadas o exportar información basada en la vista filtrada. Se exporta la siguiente información:

- Dirección de origen
- Dirección de destino
- Título de la alerta
- Gravedad de la alerta
- Mensaje de alerta
- Información adicional
- Estado confirmado
- Disponibilidad de PCAP

Para realizar la exportación:

1. Seleccione Alertas en el menú lateral.
1. Seleccione Exportar.
1. Seleccione Export Extended Alerts (Exportar alertas extendidas) para exportar información de alertas de filas independientes para cada alerta que abarque varios dispositivos. Cuando se seleccione esta opción, el archivo. .csv creará una fila duplicada del evento de alerta con los elementos únicos de cada fila. El uso de esta opción facilita la investigación de eventos de alertas exportadas.

## <a name="see-also"></a>Consulte también

[Tipos y descripciones de alertas](alert-engine-messages.md)

[Control del tráfico que se supervisa](how-to-control-what-traffic-is-monitored.md)
