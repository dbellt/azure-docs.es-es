---
title: Configuración de la supervisión de puntos de conexión de Windows
description: Enriquecimiento de los datos resueltos en dispositivos mediante el trabajo con la supervisión de puntos de conexión de Windows (WMI).
ms.date: 05/03/2021
ms.topic: how-to
ms.openlocfilehash: 4d701cfda88a2c257b001a52a19853a937661714
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108735000"
---
# <a name="configure-windows-endpoint-monitoring-wmi"></a>Configuración de la supervisión de puntos de conexión de Windows (WMI)

Con la funcionalidad de supervisión de puntos de conexión de Windows, puede configurar Azure Defender para IoT para sondear de forma selectiva los sistemas Windows. Esta actividad le proporciona información más precisa y específica sobre los dispositivos, por ejemplo, los niveles de Service Pack.

Puede configurar el sondeo con intervalos y hosts específicos, y que se realice solo con la frecuencia que se desee. El sondeo selectivo se realiza mediante el Instrumental de administración de Windows (WMI), que es el lenguaje de scripting estándar de Microsoft para administrar sistemas Windows.

> [!NOTE]
> - Solo puede ejecutar un examen cada vez.
> - Los mejores resultados se obtendrán con los usuarios que tengan privilegios de administrador local o de dominio.
> - Antes de comenzar la configuración de WMI, configure una regla de firewall que abra el tráfico saliente del sensor a la subred examinada mediante el puerto UDP 135 y todos los puertos TCP por encima de 1024.

Una vez finalizado el sondeo, se puede encontrar un archivo de registro con todos los intentos de sondeo en la opción para exportar un registro. El registro contiene todas las direcciones IP que se sondearon. Para cada dirección IP, el registro muestra la información de acierto y error. También hay un código de error, que es una cadena libre derivada de la excepción. Solo se mantiene en el sistema el examen del último registro.

Puede realizar exámenes programados o manuales. Una vez finalizado el examen, puede ver los resultados en un archivo CSV.

**Requisitos previos**

Configure una regla de firewall que abra el tráfico saliente del sensor a la subred examinada mediante el puerto UDP 135 y todos los puertos TCP por encima de 1024.

## <a name="perform-an-automatic-scan"></a>Realización de un examen automático

En esta sección se describe cómo realizar un examen automático.

**Para realizar un examen automático:**

1. En el menú lateral, seleccione **Configuración del sistema**.

2. Seleccione **Windows Endpoint Monitoring** :::image type="icon" source="media/how-to-control-what-traffic-is-monitored/windows-endpoint-monitoring-icon-v2.png" border="false"::: (Supervisión de puntos de conexión de Windows).

    :::image type="content" source="media/how-to-control-what-traffic-is-monitored/windows-endpoint-monitoring-screen-v2.png" alt-text="Captura de pantalla que muestra la selección de Windows Endpoint Monitoring (Supervisión de puntos de conexión de Windows).":::

3. En el panel **Programación de exámenes**, configure las opciones de la siguiente manera:

      - **By fixed intervals (in hours)** (Por intervalos fijos [en horas]): establezca la programación de examen según intervalos en horas.

      - **By specific times** (Por horas específicas): establezca la programación de examen según horas específicas y seleccione **Save Scan** (Guardar examen).

    :::image type="content" source="media/how-to-control-what-traffic-is-monitored/schedule-a-scan-screen-v2.png" alt-text="Captura de pantalla que muestra el botón Save Scan (Guardar examen).":::

4. Para definir el intervalo de examen, seleccione **Set scan ranges** (Establecer intervalos de examen).

5. Establezca el intervalo de direcciones IP y agregue su usuario y contraseña.

    :::image type="content" source="media/how-to-control-what-traffic-is-monitored/edit-scan-range-screen.png" alt-text="Captura de pantalla que muestra cómo agregar un usuario y una contraseña.":::

6. Para excluir un intervalo de direcciones IP de un examen, seleccione **Deshabilitar** junto al intervalo.

7. Para quitar un intervalo, seleccione :::image type="icon" source="media/how-to-control-what-traffic-is-monitored/remove-scan-icon.png" border="false"::: junto a él.

8. Seleccione **Guardar**. Se cierra el cuadro de diálogo **Edit Scan Ranges Configuration** (Editar configuración de intervalos de examen) y el número de intervalos aparece en el panel **Scan Ranges** (Intervalos de examen).

## <a name="perform-a-manual-scan"></a>Realización de un examen manual

**Para realizar un examen manual:**

1. En el menú lateral, seleccione **Configuración del sistema**.

2. Seleccione **Windows Endpoint Monitoring** :::image type="icon" source="media/how-to-control-what-traffic-is-monitored/windows-endpoint-monitoring-icon-v2.png" border="false"::: (Supervisión de puntos de conexión de Windows).

    :::image type="content" source="media/how-to-control-what-traffic-is-monitored/windows-endpoint-monitoring-screen-v2.png" alt-text="Captura de pantalla que muestra la pantalla de configuración de Windows Endpoint Monitoring (Supervisión de puntos de conexión de Windows).":::

3. En el panel **Acciones**, seleccione **Iniciar detección**. Aparece una barra de estado en el panel **Acciones** que muestra el progreso del proceso de examen.

    :::image type="content" source="media/how-to-control-what-traffic-is-monitored/started-scan-screen-v2.png" alt-text="Captura de pantalla que muestra el botón Iniciar detección.":::

## <a name="view-scan-results"></a>Visualización de los resultados de la detección

**Para ver los resultados del examen:**

1. Una vez finalizado el examen, en el panel **Acciones**, seleccione **Ver resultados del examen**. Se descarga en el equipo el archivo CSV con los resultados del examen.