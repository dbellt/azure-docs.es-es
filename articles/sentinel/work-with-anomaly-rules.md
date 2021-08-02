---
title: Uso de reglas de análisis de detección de anomalías en Azure Sentinel | Microsoft Docs
description: En este artículo se explica cómo ver, crear, administrar, evaluar y ajustar las reglas de análisis de detección de anomalías en Azure Sentinel.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/28/2021
ms.author: yelevin
ms.openlocfilehash: 84846aacb435cfef861acf046b7f623697445da2
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112059659"
---
# <a name="work-with-anomaly-detection-analytics-rules-in-azure-sentinel"></a>Uso de reglas de análisis de detección de anomalías en Azure Sentinel

> [!IMPORTANT]
>
> - Las reglas de anomalías se encuentran actualmente en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="view-soc-ml-anomaly-rule-templates"></a>Visualización de plantillas de reglas de anomalías de SOC-ML

La [característica de anomalías de SOC-ML](soc-ml-anomalies.md) de Azure Sentinel proporciona [plantillas de anomalías integradas](tutorial-detect-threats-built-in.md#anomaly) para obtener un valor inmediato. Estas plantillas de anomalías se desarrollaron para ser sólidas mediante miles de orígenes de datos y millones de eventos, pero esta característica también permite cambiar los umbrales y parámetros de las anomalías fácilmente en la interfaz de usuario. Las reglas de anomalías se deben activar antes de que generen anomalías, las cuales puede encontrar en la tabla **Anomalías** de la sección **Registros**.

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. En la hoja **Análisis**, seleccione la pestaña **Plantillas de regla**.

1. Filtre la lista por las plantillas de **Anomalías**:

    1. Haga clic en el filtro **Tipo de regla** y, a continuación, en la lista desplegable que aparece a continuación.

    1. Desmarque **Seleccionar todo** y, a continuación, seleccione **Anomalía**.

    1. Si es necesario, haga clic en la parte superior de la lista desplegable para retirarla y, a continuación, haga clic en **Aceptar**.

## <a name="activate-anomaly-rules"></a>Activación de reglas de anomalías

Al hacer clic en una de las plantillas de regla, verá la siguiente información en el panel de detalles, junto con un botón **Crear regla**:

- El campo **Descripción** explica cómo funciona la anomalía y los datos que necesita.

- **Orígenes de datos** indica el tipo de registros que se deben ingerir para poder analizarse.

- **Tácticas** son las tácticas del marco MITRE ATT&CK que abarca la anomalía.

- **Parámetros** son los atributos configurables de la anomalía.

- **Umbral** es un valor configurable que indica hasta qué grado debe ser inusual un evento para crear una anomalía.

- **Frecuencia de la regla** es el tiempo que transcurre entre los trabajos de procesamiento de registros que encuentran las anomalías.

- **Versión de anomalía** muestra la versión de la plantilla que usa una regla. Si desea cambiar la versión que usa una regla que ya está activa, debe volver a crear la regla.

- **Template last updated** (Fecha de la última actualización de la plantilla) es la fecha en que se cambió la versión de la anomalía.

Complete los pasos siguientes para activar una regla:

1. Elija una plantilla de regla que aún no esté etiquetada como **EN USO**. Haga clic en el botón **Crear regla** para abrir el asistente para la creación de reglas.

    El asistente para cada plantilla de regla será ligeramente diferente, pero tiene tres pasos o pestañas: **General**, **Configuración**, **Revisar y crear**.

    No se puede cambiar ninguno de los valores del asistente. Primero debe crear y activar la regla.

1. Recorra todas las pestañas, espere el mensaje "Validación superada" en la pestaña **Revisar y crear**, y seleccione el botón **Crear**.

    Solo puede crear una regla activa a partir de cada plantilla. Una vez completado el asistente, se crea una regla de anomalías activa en la pestaña **Reglas activas** y la plantilla (en la pestaña **Plantillas de regla**) se marcará como **EN USO**.

    > [!NOTE]
    > Suponiendo que los datos necesarios estén disponibles, la nueva regla puede tardar hasta 24 horas en aparecer en la pestaña **Reglas activas**. Para ver las nuevas reglas, seleccione la pestaña Reglas activas y filtre de la misma manera que filtró la lista Plantillas de regla anterior.

Una vez activada la regla de anomalías, las anomalías detectadas se almacenarán en la tabla **Anomalías** de la sección **Registros** del área de trabajo de Azure Sentinel.

Cada regla de anomalías tiene un período de entrenamiento y las anomalías no aparecerán en la tabla hasta después de ese período de entrenamiento. Puede encontrar el período de entrenamiento en la descripción de cada regla de anomalías.

## <a name="assess-the-quality-of-anomalies"></a>Evaluación de la calidad de las anomalías

Para ver el rendimiento de una regla de anomalías, revise una muestra de las anomalías creadas por una regla durante el último período de 24 horas. 

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. En la hoja **Análisis**, compruebe que la pestaña **Reglas activas** está seleccionada.

1. Filtre la lista por **Reglas de anomalías** (como se ha indicado anteriormente).

1. Seleccione la regla que desea evaluar y copie su nombre desde la parte superior del panel de detalles de la derecha.

1. En el menú de navegación de Azure Sentinel, seleccione **Registros**.

1. Si aparece una galería de **consultas** en la parte superior, ciérrela.

1. Seleccione la pestaña **Tablas** en el panel izquierdo de la hoja **Registros**.

1. Establezca el filtro **Intervalo de tiempo** en **Últimas 24 horas**.

1. Copie la consulta de Kusto siguiente y péguela en la ventana de consulta (donde dice "Escriba la consulta aquí o..."):

    ```kusto
    Anomalies 
    | where AnomalyTemplateName contains "________________________________"
    ```
    Pegue el nombre de la regla que copió anteriormente en lugar de los caracteres de subrayado situados entre las comillas.

1. Haga clic en **Ejecutar**. 

Cuando tenga algunos resultados, puede empezar a evaluar la calidad de las anomalías. Si no obtiene resultados, pruebe a aumentar el intervalo de tiempo.

Expanda los resultados de cada anomalía y, a continuación, expanda el campo **AnomalyReasons**. Esto le mostrará por qué se ha desencadenado la anomalía.

La "razonabilidad" o "utilidad" de una anomalía puede depender de las condiciones del entorno, pero una razón habitual para que una regla de anomalías produzca demasiadas anomalías es que el umbral sea demasiado bajo.

## <a name="tune-anomaly-rules"></a>Ajuste de reglas de anomalías

Aunque las reglas de anomalías están diseñadas para obtener la máxima eficacia de forma inmediata, cada situación es única y, a veces, es necesario ajustar las reglas de anomalías.

Puesto que no se puede editar una regla activa original, primero debe duplicar una regla de anomalías activa y, a continuación, personalizar la copia.

La regla de anomalías original seguirá ejecutándose hasta que la deshabilite o la elimine.

Esto es así de manera intencionada, para ofrecerle la oportunidad de comparar los resultados generados por la configuración original y la nueva. Las reglas duplicadas están deshabilitadas de manera predeterminada. Solo puede realizar una copia personalizada de cualquier regla de anomalías determinada. Se producirá un error si intenta realizar una segunda copia.

1. Para cambiar la configuración de una regla de anomalías, seleccione la regla de anomalías en la pestaña **Reglas activas**.

1. Haga clic con el botón derecho en cualquier lugar de la fila de la regla o haga clic con el botón izquierdo en los puntos suspensivos (...) al final de la fila y, a continuación, haga clic en **Duplicar**.

1. La nueva copia de la regla tendrá el sufijo " - Personalizada" en el nombre de la regla. Para personalizar realmente esta regla, selecciónela y haga clic en **Editar**.

1. La regla se abre en el asistente para reglas de Analytics. Aquí puede cambiar los parámetros de la regla y su umbral. Los parámetros que se pueden cambiar varían con cada tipo de anomalía y algoritmo.

    Puede obtener una vista previa de los resultados de los cambios en el panel **Vista previa de los resultados**. Haga clic en un **identificador de anomalía** en la vista previa de los resultados para ver por qué el modelo de ML identifica esa anomalía.

1. Habilite la regla personalizada para generar resultados. Algunos de los cambios pueden requerir que la regla se vuelva a ejecutar, por lo que debe esperar a que finalice y volver para comprobar los resultados en la página de registros. La regla de anomalías personalizada se ejecuta en modo **Lanzamiento como paquete piloto** (pruebas) de forma predeterminada. La regla original continúa ejecutándose en modo **Producción** de manera predeterminada.

1. Para comparar los resultados, vuelva a la tabla Anomalías en **Registros** para [evaluar la nueva regla como se indicó antes](#assess-the-quality-of-anomalies); busque solo las filas con el nombre de la regla original, así como con el nombre de la regla duplicada con el sufijo "- Personalizada" anexado a ella en la columna **AnomalyTemplateName**.

    Si está satisfecho con los resultados de la regla personalizada, puede volver a la pestaña **Reglas activas**, hacer clic en la regla personalizada, después, en el botón **Editar** y en la pestaña **General**, y cambiarla del modo **Lanzamiento como paquete piloto** a **Producción**. La regla original cambiará automáticamente a **Lanzamiento como paquete piloto** ya que no se pueden tener dos versiones de la misma regla en producción al mismo tiempo. 

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a trabajar con reglas de análisis de detección de anomalías de SOC-ML en Azure Sentinel.

- Obtenga información general sobre [SOC-ML](soc-ml-anomalies.md).
- Explore otros [tipos de reglas de análisis](tutorial-detect-threats-built-in.md).
