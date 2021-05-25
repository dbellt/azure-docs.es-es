---
title: Creación de reglas de análisis personalizadas para detectar amenazas con Azure Sentinel | Microsoft Docs
description: Use este tutorial para aprender a crear reglas de análisis personalizadas para detectar amenazas de seguridad con Azure Sentinel. Aproveche las ventajas de la agrupación de eventos, la agrupación de alertas y el enriquecimiento de alertas y comprenderá qué significa el texto AUTO DISABLED (deshabilitada automáticamente).
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/21/2021
ms.author: yelevin
ms.openlocfilehash: 9115c31b3eb24e9d0e3afd5984ea60308c424a77
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786760"
---
# <a name="tutorial-create-custom-analytics-rules-to-detect-threats"></a>Tutorial: Creación de reglas de análisis personalizadas para detectar amenazas

Ahora que ha [conectado los orígenes de datos](quickstart-onboard.md) a Azure Sentinel, puede crear reglas de análisis personalizadas que le ayuden a detectar las amenazas y los comportamientos anómalos que se producen en su entorno. Estas reglas buscan eventos o conjuntos de eventos específicos en el entorno, le avisan cuando se alcanzan determinados umbrales de eventos o condiciones, generan incidentes para que el SOC evalúe e investigue, y responden a amenazas con procesos de seguimiento y corrección automatizados. 

Este tutorial le ayuda a crear reglas personalizadas para detectar amenazas con Azure Sentinel.

Después de completar este tutorial, podrá realizar las operaciones siguientes:
> [!div class="checklist"]
> * Creación de reglas de análisis
> * Definición de la forma en que se procesan los eventos y las alertas
> * Definición de la forma en que se generan las alertas y los incidentes
> * Elección de respuestas a amenazas automatizadas para las reglas

## <a name="create-a-custom-analytics-rule-with-a-scheduled-query"></a>Creación de una regla de análisis personalizada con una consulta programada

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. En la barra de acciones de la parte superior, seleccione **+ Crear** y elija **Regla de consulta programada**. Así se abre el **Asistente para reglas de Analytics**.

    :::image type="content" source="media/tutorial-detect-threats-custom/create-scheduled-query-small.png" alt-text="Crear una consulta programada" lightbox="media/tutorial-detect-threats-custom/create-scheduled-query-full.png":::

### <a name="analytics-rule-wizard---general-tab"></a>Asistente para reglas de análisis: pestaña General

- Proporcione un **Nombre** único y una **Descripción**. 

- En el campo **Tactics** (Táctica), puede elegir cualquiera de las categorías de ataques por las que se clasifica la regla. Se basan en las tácticas del marco [MITRE ATT&CK](https://attack.mitre.org/).

- Establezca la **Gravedad** de la alerta según sea necesario. 

- Cuando cree la regla, el valor predeterminado del campo **Status** (Estado) es **Enabled** (Habilitado), lo que significa que se ejecutará inmediatamente después de que termine de crearla. Si no desea ejecutarla de inmediato, seleccione **Disabled** (Deshabilitado) para agregar la regla a la pestaña **Active rules** (Reglas activas), desde donde podrá habilitarla cuando sea necesario.

   :::image type="content" source="media/tutorial-detect-threats-custom/general-tab.png" alt-text="Inicio de la creación de una regla de análisis personalizada":::

## <a name="define-the-rule-query-logic-and-configure-settings"></a>Definición de la lógica de consulta de regla y configuración de los valores

En la pestaña **Establecer la lógica de la regla**, puede escribir una consulta directamente en el campo **Consulta de la regla**, o bien crearla en Log Analytics y, después, copiarla y pegarla aquí.

- Las consultas se escriben en el lenguaje de consulta de Kusto (KQL). Obtenga más información sobre los [conceptos](/azure/data-explorer/kusto/concepts/) y las [consultas](/azure/data-explorer/kusto/query/) de KQL, y vea esta práctica [guía de referencia rápida](/azure/data-explorer/kql-quick-reference).

- En el ejemplo que se muestra en esta captura de pantalla se consulta la tabla *SecurityEvent* para mostrar un tipo de [eventos de inicio de sesión de Windows con errores](/windows/security/threat-protection/auditing/event-4625).

   :::image type="content" source="media/tutorial-detect-threats-custom/set-rule-logic-tab-1-new.png" alt-text="Configuración de la lógica de consulta de regla y sus valores" lightbox="media/tutorial-detect-threats-custom/set-rule-logic-tab-all-1-new.png":::

- Aquí tiene otra consulta de ejemplo que le alertará cuando se cree una cantidad anómala de recursos en [Azure Activity](../azure-monitor/essentials/activity-log.md).

    ```kusto
    AzureActivity
    | where OperationName == "Create or Update Virtual Machine" or OperationName =="Create Deployment"
    | where ActivityStatus == "Succeeded"
    | make-series dcount(ResourceId)  default=0 on EventSubmissionTimestamp in range(ago(7d), now(), 1d) by Caller
    ```

    > [!NOTE]
    > #### <a name="rule-query-best-practices"></a>Procedimientos recomendados de consulta de regla
    > - La longitud de la consulta debe ser de entre 1 y 10 000 caracteres y no puede contener "`search *`" ni "`union *`". Puede usar [funciones definidas por el usuario](/azure/data-explorer/kusto/query/functions/user-defined-functions) para superar la limitación de longitud de consulta.
    >
    > - **No se admite** el uso de funciones ADX para crear consultas de Azure Data Explorer en la ventana de consulta de Log Analytics.
    >
    > - Al usar la función **`bag_unpack`** en una consulta, si se proyectan las columnas como campos mediante "`project field1`" y la columna no existe, se producirá un error en la consulta. Para evitar que esto suceda, debe proyectar la columna de la siguiente manera:
    >   - `project field1 = column_ifexists("field1","")`

### <a name="alert-enrichment"></a>Enriquecimiento de alertas

> [!IMPORTANT]
> Las características de enriquecimiento de alertas se encuentran actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.
    
- Use la sección de configuración **Asignación de entidades** para asignar parámetros de los resultados de las consultas a entidades reconocidas por Azure Sentinel. Las entidades enriquecen la salida de las reglas (alertas e incidentes) con datos esenciales que actúan como bloques de creación de cualquier proceso de investigación y de las acciones de corrección posteriores. También son los criterios por los que puede agrupar las alertas en incidentes en la pestaña **Configuración de los incidentes**.

    Obtenga más información sobre las [entidades en Azure Sentinel](entities-in-azure-sentinel.md).

    Consulte [Asignación de campos de datos a entidades en Azure Sentinel](map-data-fields-to-entities.md) para obtener instrucciones completas de asignación de entidades, junto con información importante sobre la [compatibilidad con versiones anteriores](map-data-fields-to-entities.md#notes-on-the-new-version).

- Use la sección configuración de **Detalles personalizados** para extraer elementos de datos de eventos de la consulta y mostrarlos en las alertas generadas por esta regla, lo que le proporciona visibilidad inmediata del contenido del evento en sus alertas e incidentes.

    Obtenga más información sobre la exposición de detalles personalizados en alertas y consulte las [instrucciones completas](surface-custom-details-in-alerts.md).

### <a name="query-scheduling-and-alert-threshold"></a>Programación de consultas y umbral de alertas

- En la sección **Query scheduling** (Programación de consultas), establezca los siguientes parámetros:

   :::image type="content" source="media/tutorial-detect-threats-custom/set-rule-logic-tab-2.png" alt-text="Establecer la programación de consultas y la agrupación de eventos" lightbox="media/tutorial-detect-threats-custom/set-rule-logic-tab-all-2-new.png":::

    - Establezca la opción **Ejecutar la consulta cada** para controlar la frecuencia de ejecución de la consulta (puede establecer frecuencias de 5 minutos o de una vez cada 14 días).

    - Set **Lookup data from the last** (Datos de la búsqueda a partir del último) para determinar el periodo de los datos que cubre la consulta (por ejemplo, puede consultar los 10 últimos minutos de datos o las 6 últimas horas de datos). El máximo es de 14 días.

        > [!NOTE]
        > **Intervalos de consulta y período de retrospectiva**
        >
        >  Estos dos valores son independientes entre sí, hasta cierto punto. Puede ejecutar una consulta a un intervalo corto que abarque un periodo mayor que el intervalo (teniendo en efecto consultas que se solapan), pero no puede ejecutar una consulta a un intervalo que supere el periodo de cobertura, ya que, de lo contrario tendrá lagunas en la cobertura general de la consulta.
        >
        > **Retraso de la ingesta**
        >
        > Para tener en cuenta la **latencia** que puede producirse entre la generación de un evento en el origen y su ingesta en Azure Sentinel, y para garantizar una cobertura completa sin duplicación de datos, Azure Sentinel ejecuta reglas de análisis programadas con un **retraso de cinco minutos** desde su hora programada.
        >
        > Para obtener una explicación técnica detallada de por qué este retraso es necesario y cómo soluciona este problema, consulte la excelente entrada de blog de Ron Marsiano sobre el tema del [control del retraso de la ingesta en reglas de alertas programadas de Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/handling-ingestion-delay-in-azure-sentinel-scheduled-alert-rules/ba-p/2052851).

- Use la sección **Umbral de alerta** para definir el nivel de confidencialidad de la regla. Por ejemplo, en **Generar alerta cuando el número de resultados de consulta**, seleccione **Es mayor que** y escriba el número 1000 si desea que la regla genere una alerta solo si la consulta genera más de 1000 resultados cada vez que se ejecuta. Este es un campo obligatorio, por lo que si no desea establecer un umbral (es decir, si desea que su alerta registre todos los eventos), escriba 0 en el campo numérico.
    
### <a name="results-simulation"></a>Simulación de resultados

En el área **Results simulation** (Simulación de resultados) de la derecha del asistente, seleccione **Test with current data** (Probar con los datos actuales) y Azure Sentinel mostrará un gráfico de los resultados (eventos de registro) que la consulta habría generado en las últimas 50 ejecuciones, según la programación definida actualmente. Si modifica la consulta, vuelva a seleccionar **Probar con los datos actuales** para actualizar el gráfico. El gráfico muestra el número de resultados en un periodo definido, lo que determina la configuración de la sección **Query scheduling** (Programación de consultas).
  
Este es el aspecto que podría tener la simulación de resultados en la consulta de la captura de pantalla anterior. El lado izquierdo es la vista predeterminada, mientras que el lado derecho es lo que se ve al mantener el mouse sobre un momento dado del gráfico.

:::image type="content" source="media/tutorial-detect-threats-custom/results-simulation.png" alt-text="Capturas de pantallas de simulación de resultados":::

Si percibe que la consulta desencadenaría alertas excesivas o demasiado frecuentes, puede experimentar con la configuración de las [secciones](#query-scheduling-and-alert-threshold) **Programación de consultas** y **Umbral de alerta** y seleccionar volver a **Test with current data** (Probar con los datos actuales).

### <a name="event-grouping-and-rule-suppression"></a>Agrupación de eventos y supresión de reglas

> [!IMPORTANT]
> La agrupación de eventos se encuentra actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.
    
- En **Agrupación de eventos**, elija una de las dos formas de controlar la agrupación de **eventos** en **alertas**: 

    - **Agrupar todos los eventos en una misma alerta** (la configuración predeterminada). La regla genera una única alerta cada vez que se ejecuta, siempre y cuando la consulta devuelva más resultados de los especificados en el **umbral de alerta** anterior. La alerta incluye un resumen de todos los eventos que se devuelven en los resultados. 

    - **Desencadenar una alerta para cada evento**. La regla genera una alerta única para cada evento devuelto por la consulta. Esto resulta útil si desea que los eventos se muestren individualmente, o si desea agruparlos por determinados parámetros por usuario, nombre de host o cualquier otro elemento. Puede definir estos parámetros en la consulta.
    
        En la actualidad, el número de alertas que puede generar una regla se limita a veinte. Si en una regla determinada, **Agrupación de eventos** se establece en **Desencadenar una alerta para cada evento** y la consulta de la regla devuelve más de 20 eventos, cada uno de los primeros 19 eventos generará una alerta única y la vigésima alerta resumirá todo el conjunto de eventos devueltos. En otras palabras, la vigésima alerta es lo que habría generado en la opción **Agrupar todos los eventos en una misma alerta**.

    > [!NOTE]
    > ¿En qué se diferencian los **eventos** y las **alertas**?
    >
    > - Un **evento** es una descripción de una única repetición de una acción. Por ejemplo, una sola entrada de un archivo de registro podría contar como un evento. En este contexto, un evento hace referencia a un único resultado devuelto por una consulta en una regla de análisis.
    >
    > - Una **alerta** es una colección de eventos que, en conjunto, son importantes desde el punto de vista de la seguridad. Una alerta podría contener un solo evento si el evento tuviera implicaciones de seguridad importantes (por ejemplo, un inicio de sesión administrativo de un país extranjero fuera de horas de oficina).
    >
    > - Por cierto, ¿qué son los **incidentes**? La lógica interna de Azure Sentinel crea **incidentes** a partir de **alertas** o de grupos de alertas. La cola de incidentes es el punto focal del trabajo del analista del centro de operaciones de seguridad: evaluación de prioridades, investigación y corrección.
    > 
    > Azure Sentinel ingiere eventos sin procesar de algunos orígenes de datos y alertas ya procesadas de otros usuarios. Es importante tener en cuenta con cuál es el que se está tratando en cualquier momento.

- In the **Suppression** (Supresión), en el valor **Stop running query after alert is generated** (Detener la ejecución de la consulta después de que se genere una alerta), seleccione **On** (Activar) si, una vez que recibe la alerta, desea suspender la operación de esta reglar durante un periodo que supere el intervalo de consulta. Si lo activa, en **Stop running query for** (Detener la ejecución de la consulta durante), seleccione el periodo durante el que debe detenerse la consulta, con un máximo de 24 horas.

## <a name="configure-the-incident-creation-settings"></a>Configuración de la creación de incidentes

En la pestaña **Incident Settings** (Configuración de incidentes), puede elegir si Azure Sentinel convierte las alertas en incidentes sobre los que se pueden realizar acciones y cómo lo hace. Si esta pestaña no se modifica, Azure Sentinel creará un solo incidente independiente a partir de todas y cada una de las alertas. Puede elegir que no se creen incidentes o agrupar varias alertas en un solo incidente. Para ello, solo debe cambiar el valor de esta pestaña.

> [!IMPORTANT]
> La pestaña Configuración de los incidentes está actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

:::image type="content" source="media/tutorial-detect-threats-custom/incident-settings-tab.png" alt-text="Definir la configuración de creación de incidentes y la agrupación de alertas":::

### <a name="incident-settings"></a>Configuración de los incidentes

En la sección **Configuración de los incidentes**, el valor predeterminado de **Crear incidentes a partir de las alertas desencadenadas por esta regla de análisis** es **Habilitado**, lo que significa que Azure Sentinel creará un solo incidente independiente por cada una de las alertas que desencadene la regla.
    
- Si no desea que esta regla provoque la aparición de incidentes (por ejemplo, si esta regla es solo para recopilar información para su posterior análisis), seleccione **Disabled** (Deshabilitado).

- Si desea que se cree un solo incidente a partir de un grupo de alertas, en lugar de uno para cada alerta, consulte la sección siguiente.

### <a name="alert-grouping"></a>Agrupación de alertas

En la sección **Alert grouping** (Agrupación de alertas), si desea que se genere un solo incidente a partir de un grupo de hasta 150 alertas similares o recurrentes (consulte la nota), en **Group related alerts, triggered by this analytics rule, into incidents** (Agrupar en incidentes alertas relacionadas desencadenadas por esta regla de análisis) seleccione **Enabled** (Habilitado) y establezca los siguientes parámetros.

- **Limite el grupo a las alertas creadas en el período de tiempo seleccionado**: Determine el período de tiempo en el que se agruparán las alertas similares o periódicas. Todas las alertas correspondientes que se encuentren dentro de este periodo de tiempo generarán colectivamente un incidente o un conjunto de incidentes (en función de la configuración de agrupación que encontrará a continuación). Las alertas que aparezcan fuera de este periodo de tiempo generarán un incidente, o conjunto de incidentes, independientes.

- **Agrupe las alertas desencadenadas por esta regla de análisis en un único incidente por**: elija el motivo por el que se agruparán las alertas:

    - **Agrupar las alertas en un solo incidente si todas las entidades coinciden**: Las alertas se agrupan si comparten los mismos valores entre todas las entidades asignadas (definidas en la pestaña Set rule logic [Establecer lógica de regla] anterior). Esta es la configuración recomendada.

    - **Agrupar todas las alertas desencadenadas por esta regla en un único incidente**: Todas las alertas que genera esta regla se agrupan aunque no compartan valores idénticos.

    - **Agrupar las alertas en un solo incidente si las entidades seleccionadas coinciden**: Las alertas se agrupan si comparten valores idénticos en algunas de las entidades asignadas (que puede seleccionar en la lista desplegable). Es posible que desee usar esta opción si, por ejemplo, desea crear incidentes independientes basados en las direcciones IP de origen o de destino.

- **Vuelva a abrir los incidentes coincidentes cerrados**: si se ha resuelto y cerrado un incidente y, posteriormente, se genera otra alerta que habría pertenecido a ese incidente, establezca esta opción en **Enabled** (Habilitado) si quiere que el incidente cerrado se vuelva a abrir, o bien déjela como **Disabled** (Deshabilitado) si quiere que la alerta cree un incidente nuevo.
    
    > [!NOTE]
    > Se pueden agrupar **hasta 150 alertas** en un solo incidente. Si hay más de 150 alertas generadas por una regla que las agrupa en un solo incidente, se generará un nuevo incidente con los mismos detalles del incidente que el original, y las alertas sobrantes se agruparán en el nuevo incidente.

## <a name="set-automated-responses-and-create-the-rule"></a>Establecimiento de respuestas automatizadas y creación de la regla

1. En la pestaña **Automated responses** (Respuestas automatizadas), seleccione todos los cuadernos de estrategias que desee ejecutar automáticamente cuando la regla personalizada genere una alerta. Para obtener más información sobre la creación y automatización de cuadernos de estrategias, vea [Respuesta a amenazas](tutorial-respond-threats-playbook.md).

    :::image type="content" source="media/tutorial-detect-threats-custom/automated-response-tab.png" alt-text="Definir la configuración de respuesta automatizada":::

1. Seleccione **Revisar y crear** para revisar todos los valores de configuración de la nueva regla de alerta. Cuando aparezca el mensaje "Validación superada", seleccione **Crear** para inicializar la regla de alerta.

    :::image type="content" source="media/tutorial-detect-threats-custom/review-and-create-tab.png" alt-text="Examinar toda la configuración y crear la regla":::

## <a name="view-the-rule-and-its-output"></a>Visualización de la regla y su salida
  
- Puede encontrar la regla personalizada recién creada (de tipo "Programado") en la tabla en la pestaña **Reglas activas** de la pantalla **Análisis** principal. Desde esta lista puede habilitar, deshabilitar o eliminar cada regla.

- Para ver los resultados de las reglas de alertas que cree, vaya a la página de **incidentes**, donde puede evaluar las prioridades, [investigar los incidentes](tutorial-investigate-cases.md) y solucionar las amenazas.

- Puede actualizar la consulta de regla para excluir falsos positivos. Para obtener más información, consulte [Controlar falsos positivos en Azure Sentinel](false-positives.md).

> [!NOTE]
> Las alertas generadas en Azure Sentinel están disponibles a través de [Seguridad de Microsoft Graph](/graph/security-concept-overview). Para obtener más información, vea la [documentación sobre alertas de Seguridad de Microsoft Graph](/graph/api/resources/security-api-overview).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="issue-no-events-appear-in-query-results"></a>Problema: no aparece ningún evento en los resultados de la consulta

Si la **agrupación de eventos** se establece para **desencadenar una alerta para cada evento**, en determinados escenarios, al ver los resultados de la consulta posteriormente (por ejemplo, al volver a las alertas de un incidente), es posible que no aparezca ningún resultado de la consulta. Esto se debe a que la conexión del evento a la alerta se logra mediante el hash de la información del evento concreto y la inclusión del hash en la consulta. Si los resultados de la consulta han cambiado desde que se generó la alerta, el hash dejará de ser válido y no se mostrará ningún resultado. 

Para ver los eventos, quite manualmente la línea con el hash de la consulta de la regla y ejecute la consulta.

### <a name="issue-a-scheduled-rule-failed-to-execute-or-appears-with-auto-disabled-added-to-the-name"></a>Problema: No se pudo ejecutar una regla programada o aparece con el texto AUTO DISABLED agregado al nombre

Es muy poco frecuente que una regla de consulta programada no se ejecute, pero puede ocurrir. Azure Sentinel clasifica los errores como transitorios o permanentes, en función del tipo específico del error y de las circunstancias que condujeron a él.

#### <a name="transient-failure"></a>Error transitorio

El error transitorio se produce debido a una circunstancia que es temporal y pronto volverá a la normalidad, momento en el que la ejecución de la regla se realizará correctamente. Algunos ejemplos de errores que Azure Sentinel clasifica como transitorios son los siguientes:

- Una consulta de regla tarda demasiado tiempo en ejecutarse y se agota el tiempo de espera.
- Problemas de conectividad entre orígenes de datos y Log Analytics, o entre Log Analytics y Azure Sentinel.
- Cualquier otro error nuevo y desconocido se considera transitorio.

Cuando se produce un error transitorio, Azure Sentinel sigue intentando volver a ejecutar la regla tras intervalos predeterminados y cada vez mayores, hasta un punto determinado. Después, la regla se volverá a ejecutar solo en la siguiente hora programada. Una regla nunca se deshabilitará automáticamente debido a un error transitorio.

#### <a name="permanent-failure---rule-auto-disabled"></a>Error permanente: deshabilitación automática de la regla

El error permanente se produce debido a un cambio en las condiciones que permiten la ejecución de la regla y que, sin intervención humana, no volverán a su estado anterior. A continuación se muestran algunos ejemplos de errores que se clasifican como permanentes:

- El área de trabajo de destino (en la que opera la consulta de regla) se ha eliminado.
- La tabla de destino (en la que opera la consulta de regla) se ha eliminado.
- Se quitó Azure Sentinel del área de trabajo de destino.
- Una función usada por la consulta de la regla ya no es válida porque se ha modificado o quitado.
- Se cambiaron los permisos para uno de los orígenes de datos de la consulta de la regla.
- Uno de los orígenes de datos de la consulta de la regla se ha eliminado o desconectado.

**En el caso de un número predeterminado de errores permanentes consecutivos, del mismo tipo y en la misma regla,** Azure Sentinel deja de intentar ejecutar la regla y también lleva a cabo los siguientes pasos:

- Deshabilita la regla.
- Agrega las palabras **"AUTO DISABLED"** (deshabilitada automáticamente) al principio del nombre de la regla.
- Agrega el motivo del error (y la deshabilitación) a la descripción de la regla.

Puede determinar fácilmente si se ha deshabilitado automáticamente alguna regla si ordena la lista de reglas por nombre. Las reglas deshabilitadas automáticamente estarán en la parte superior de la lista o cerca de ella.

Los administradores de SOC deben asegurarse de comprobar la lista de reglas periódicamente para comprobar si hay reglas deshabilitadas automáticamente.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a detectar amenazas casos mediante Azure Sentinel.

- Obtenga información sobre cómo [investigar incidentes en Azure Sentinel](tutorial-investigate-cases.md).
- Obtenga información sobre las [entidades en Azure Sentinel](entities-in-azure-sentinel.md).
- Obtenga información sobre la [configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](tutorial-respond-threats-playbook.md).
