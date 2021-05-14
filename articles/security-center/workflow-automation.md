---
title: Automatización de flujos de trabajo en Azure Security Center | Microsoft Docs
description: Aprenda a crear y automatizar flujos de trabajo en Azure Security Center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 05/03/2021
ms.author: memildin
ms.openlocfilehash: bc9c12338610b81e23d5b850fe53aa3c5731a03a
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108770288"
---
# <a name="automate-responses-to-security-center-triggers"></a>Automatización de respuestas a desencadenadores de Security Center

Cada programa de seguridad incluye varios flujos de trabajo para la respuesta a incidentes. Estos procesos pueden incluir notificaciones para las partes interesadas competentes, iniciar un proceso de administración de cambios y aplicar pasos de corrección específicos. Los expertos en seguridad recomiendan automatizar tantos pasos de esos procedimientos como sea posible. Recuerde que la automatización reduce la sobrecarga. También puede mejorar la seguridad asegurándose de que los pasos del proceso se realizan de forma rápida, coherente y según sus requisitos predefinidos.

En este artículo se describe la característica de automatización de flujos de trabajo de Azure Security Center. Esta característica puede desencadenar aplicaciones lógicas sobre alertas de seguridad, recomendaciones y cambios en el cumplimiento normativo. Por ejemplo, si quiere que Security Center envíe un correo electrónico a un usuario específico cuando se produce una alerta. También aprenderá a crear instancias de Logic Apps con [Azure Logic Apps](../logic-apps/logic-apps-overview.md).


## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general (GA)|
|Precios:|Gratuito|
|Roles y permisos necesarios:|**Rol Administrador de seguridad** o **Propietario** en el grupo de recursos<br>También debe tener permisos de escritura para el recurso de destino.<br><br>Para trabajar con flujos de trabajo de Azure Logic Apps, también debe tener los siguientes roles o permisos de Logic Apps:<br> Son necesarios los permisos de - [Operador de aplicación lógica](../role-based-access-control/built-in-roles.md#logic-app-operator) o el acceso de lectura o desencadenamiento de aplicación lógica (este rol no puede crear ni editar aplicaciones lógicas, solo *ejecutar* las existentes).<br> Los permisos de - [Colaborador de la aplicación lógica](../role-based-access-control/built-in-roles.md#logic-app-contributor) son necesarios para la creación y modificación de aplicaciones lógicas.<br>Si quiere usar conectores de aplicaciones lógicas, es posible que necesite credenciales adicionales para iniciar sesión en sus servicios respectivos (por ejemplo, en las instancias de Outlook, Teams o Slack).|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||



## <a name="create-a-logic-app-and-define-when-it-should-automatically-run"></a>Creación de una aplicación lógica y definición de cuándo debería ejecutarse automáticamente 

1. En la barra lateral de Security Center, seleccione **Automatización de flujos de trabajo**.

    :::image type="content" source="./media/workflow-automation/list-of-workflow-automations.png" alt-text="Lista de automatizaciones de flujos de trabajo":::

    Desde esta página puede crear nuevas reglas de automatización, así como habilitar, deshabilitar o eliminar las existentes.

1. Para definir un nuevo flujo de trabajo, haga clic en **Add workflow automation** (Agregar automatización de flujos de trabajo). 

    Aparecerá un panel con las opciones de la nueva automatización. Aquí puede escribir lo siguiente:
    1. Un nombre y descripción para la automatización.
    1. Los desencadenadores que iniciarán este flujo de trabajo automático. Por ejemplo, cuando quiera que la aplicación lógica se ejecute cuando se genere una alerta de seguridad que contenga "SQL".

        > [!NOTE]
        > Si el desencadenador es una recomendación que tiene "recomendaciones secundarias", por ejemplo **Se deben remediar los resultados de evaluación de vulnerabilidades en las bases de datos SQL**, la aplicación lógica no se desencadenará para cada nueva búsqueda de seguridad; solo cuando cambie el estado de la recomendación primaria.

    1. La aplicación lógica que se ejecutará cuando se cumplan las condiciones del desencadenador. 

        :::image type="content" source="./media/workflow-automation/add-workflow.png" alt-text="Panel para agregar automatizaciones del flujo de trabajo":::

1. En la sección Acciones, haga clic en **Crear una nueva** para comenzar el proceso de creación de la aplicación lógica.

    Se le dirigirá a Azure Logic Apps.

    [![Creación de una aplicación lógica nueva](media/workflow-automation/logic-apps-create-new.png)](media/workflow-automation/logic-apps-create-new.png#lightbox)

1. Escriba un nombre, un grupo de recursos y una ubicación, y haga clic en **Crear**.

1. En la nueva aplicación lógica, puede decidir si quiere usar plantillas predefinidas integradas en la categoría de seguridad. También puede definir un flujo de eventos personalizado para que se active cuando se desencadene este proceso.

    > [!TIP]
    > A veces, en una aplicación lógica, los parámetros se incluyen en el conector como parte de una cadena y no en su propio campo. Para ver un ejemplo de cómo extraer parámetros, consulte el paso 14 de [Trabajo con parámetros de aplicaciones lógicas en la creación de automatizaciones de flujo de trabajo de Azure Security Center](https://techcommunity.microsoft.com/t5/azure-security-center/working-with-logic-app-parameters-while-building-azure-security/ba-p/1342121).

    El diseñador de aplicaciones lógicas admite estos desencadenadores de Security Center:

    - **Cuando se crea o se desencadena una recomendación de Azure Security Center**: si la aplicación lógica se basa en una recomendación que entra en desuso o se reemplaza, la automatización dejará de funcionar y deberá actualizar el desencadenador. Para hacer un seguimiento de los cambios en las recomendaciones, vea las [notas de la versión de Azure Security Center](release-notes.md).

    - **Cuando se crea o se desencadena una alerta de Azure Security Center**: puede personalizar el desencadenador para que se refiera solo a las alertas con los niveles de gravedad que le interesen.
    
    - **When a Security Center regulatory compliance assessment is created or triggered** (Cuando se crea o se desencadena una evaluación de cumplimiento normativo de Security Center): se desencadenan automatizaciones según las actualizaciones de las evaluaciones de cumplimiento normativo.

    > [!NOTE]
    > Si utiliza el desencadenador heredado "Cuando se desencadena una respuesta a una alerta de Azure Security Center", la característica de automatización de flujos de trabajo no iniciará su instancia de aplicación lógica. En su lugar, use cualquiera de los desencadenadores mencionados anteriormente. 

    [![Aplicación lógica de ejemplo](media/workflow-automation/sample-logic-app.png)](media/workflow-automation/sample-logic-app.png#lightbox)

1. Una vez que haya definido la aplicación lógica, vuelva al panel de la definición de automatización del flujo de trabajo ("Agregar automatización de flujos de trabajo"). Haga clic en **Actualizar** para asegurarse de que la nueva aplicación lógica está disponible en la selección.

    ![Actualizar](media/workflow-automation/refresh-the-list-of-logic-apps.png)

1. Seleccione la aplicación lógica y guarde la automatización. Tenga en cuenta que la lista desplegable de aplicaciones lógicas solo muestra las aplicaciones lógicas con los conectores de Security Center complementarios mencionados anteriormente.


## <a name="manually-trigger-a-logic-app"></a>Desencadenar manualmente una aplicación lógica

También puede ejecutar Logic Apps manualmente al ver una alerta o recomendación de seguridad.

Para ejecutar manualmente una aplicación lógica, abra una alerta o recomendación y haga clic en **Desencadenar aplicación lógica**:

[![Desencadenar manualmente una aplicación lógica](media/workflow-automation/manually-trigger-logic-app.png)](media/workflow-automation/manually-trigger-logic-app.png#lightbox)


## <a name="configure-workflow-automation-at-scale-using-the-supplied-policies"></a>Configuración de la automatización de flujos de trabajo a escala mediante las directivas suministradas

La automatización de los procesos de supervisión y respuesta ante incidentes de la organización puede mejorar considerablemente el tiempo necesario para investigar y mitigar los incidentes de seguridad.

Para implementar las configuraciones de automatización en la organización, use las directivas "DeployIfNotExist" de Azure Policy proporcionadas que se describen a continuación para crear y configurar los procedimientos de automatización de flujos de trabajo.

Empiece a usar las [plantillas de automatización de flujos de trabajo](https://github.com/Azure/Azure-Security-Center/tree/master/Workflow%20automation).

Para implementar estas directivas:

1. Desde la tabla siguiente, seleccione la directiva que quiere aplicar:

    |Objetivo  |Directiva  |Id. de directiva  |
    |---------|---------|---------|
    |Automatización de flujos de trabajo para alertas de seguridad              |[Implementar la automatización del flujo de trabajo para las alertas de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ff1525828-9a90-4fcf-be48-268cdd02361e)|f1525828-9a90-4fcf-be48-268cdd02361e|
    |Automatización de flujos de trabajo para recomendaciones de seguridad     |[Implementar la automatización del área de trabajo para las recomendaciones de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f73d6ab6c-2475-4850-afd6-43795f3492ef)|73d6ab6c-2475-4850-afd6-43795f3492ef|
    |Automatización de flujos de trabajo para cambios de cumplimiento normativo|[Implementación de la automatización del flujo de trabajo para el cumplimiento normativo de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f509122b9-ddd9-47ba-a5f1-d0dac20be63c)|509122b9-ddd9-47ba-a5f1-d0dac20be63c|
    ||||

    > [!TIP]
    > También puede encontrarlos buscando Azure Policy:
    > 1. Abra Azure Policy.
    > :::image type="content" source="./media/continuous-export/opening-azure-policy.png" alt-text="Acceso a Azure Policy":::
    > 2. En el menú Azure Policy, seleccione **Definiciones** y realice la búsqueda por nombre. 

1. En la página pertinente de Azure Policy, seleccione **Asignar**.
    :::image type="content" source="./media/workflow-automation/export-policy-assign.png" alt-text="Asignación de Azure Policy":::

1. Abra cada pestaña y establezca los parámetros como quiera:
    1. En la pestaña **Aspectos básicos**, establezca el ámbito de la directiva. Para usar la administración centralizada, asigne la directiva al grupo de administración que contiene las suscripciones que usarán la automatización de flujos de trabajo. 
    1. En la pestaña **Parámetros**, establezca el grupo de recursos y los detalles del tipo de datos. 
        > [!TIP]
        > Cada parámetro tiene información sobre herramientas que explica las opciones disponibles.
        >
        > La pestaña Parámetros de Azure Policy (1) proporciona acceso a opciones de configuración similares, como la página de automatización de flujos de trabajo de Security Center (2).
        > :::image type="content" source="./media/workflow-automation/azure-policy-next-to-workflow-automation.png" alt-text="Comparación de los parámetros de la automatización de flujos de trabajo con Azure Policy" lightbox="./media/workflow-automation/azure-policy-next-to-workflow-automation.png":::

    1. Opcionalmente, para aplicar esta asignación a las suscripciones existentes, abra la pestaña **Corrección** y seleccione la opción para crear una tarea de corrección.

1. Revise la página de resumen y seleccione **Crear**.


## <a name="data-types-schemas"></a>Esquemas de tipos de datos

Para ver los esquemas de eventos sin procesar de las alertas de seguridad o los eventos de recomendaciones que se pasan a la instancia de aplicación lógica, consulte los [Esquemas de tipos de datos de automatización de flujo de trabajo](https://aka.ms/ASCAutomationSchemas). Puede resultar útil en los casos en que no se usan los conectores de aplicación lógica integrados de Security Center mencionados anteriormente, sino que alternativamente se usa el conector HTTP genérico de la aplicación lógica; puede usar el esquema JSON del evento para analizarlo manualmente como considere oportuno.


## <a name="faq-for-workflow-automation"></a>Preguntas más frecuentes relacionadas con la automatización de flujos de trabajo

### <a name="does-workflow-automation-support-any-business-continuity-or-disaster-recovery-bcdr-scenarios"></a>¿La automatización de flujos de trabajo es compatible con los escenarios de continuidad empresarial o recuperación ante desastres (BCDR)?

Al preparar el entorno para escenarios de BCDR, en los que el recurso de destino está experimentando una interrupción u otro desastre, la organización tiene la responsabilidad de evitar la pérdida de datos mediante la preparación de copias de seguridad de acuerdo con las instrucciones de Azure Event Hubs, el área de trabajo de Log Analytics y la aplicación lógica.

Para cada automatización activa, se recomienda crear una automatización idéntica (deshabilitada) y almacenarla en una ubicación diferente. Cuando se produce una interrupción, puede habilitar estas automatización de copias de seguridad y mantener las operaciones normales.

Obtenga más información sobre [continuidad empresarial y recuperación ante desastres para Azure Logic Apps](../logic-apps/business-continuity-disaster-recovery-guidance.md).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre cómo crear aplicaciones lógicas, automatizar su ejecución en Security Center y ejecutarlas manualmente.

Para obtener material relacionado, consulte: 

- [Módulo de Microsoft Learn sobre cómo la automatización de flujos de trabajo para automatizar una respuesta de seguridad](/learn/modules/resolve-threats-with-azure-security-center/)
- [Recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md)
- [Alertas de seguridad en el Centro de seguridad de Azure](security-center-alerts-overview.md)
- [Acerca de Azure Logic Apps](../logic-apps/logic-apps-overview.md)
- [Conectores para Azure Logic Apps](../connectors/apis-list.md)
- [Esquemas de tipos de datos de automatización de flujo de trabajo](https://aka.ms/ASCAutomationSchemas)