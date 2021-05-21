---
title: 'Tutorial: Uso de cuadernos de estrategias con reglas de automatización en Azure Sentinel'
description: Use este tutorial para ayudarle a usar los cuadernos de estrategias junto con las reglas de automatización de Azure Sentinel para automatizar la respuesta a los incidentes y corregir las amenazas de seguridad.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: e4afc5c8-ffad-4169-8b73-98d00155fa5a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/18/2021
ms.author: yelevin
ms.openlocfilehash: b2a98e92630fcdc46228cc36579cfe9787b92daf
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786652"
---
# <a name="tutorial-use-playbooks-with-automation-rules-in-azure-sentinel"></a>Tutorial: Uso de cuadernos de estrategias con reglas de automatización en Azure Sentinel

En este tutorial se muestra cómo usar los cuadernos de estrategias junto con las reglas de automatización para automatizar la respuesta a los incidentes y corregir las amenazas de seguridad detectadas por Azure Sentinel. Después de realizar este tutorial, podrá:

> [!div class="checklist"]
>
> * Crear una regla de automatización
> * Crear un cuaderno de estrategias
> * Agregar acciones a un cuaderno de estrategias
> * Adjuntar un cuaderno de estrategias a una regla de automatización o una regla de análisis para automatizar la respuesta a las amenazas

## <a name="what-are-automation-rules-and-playbooks"></a>¿Qué son las reglas de automatización y los cuadernos de estrategias?

Las reglas de automatización ayudan a evaluar incidentes en Azure Sentinel. Puede usarlas para asignar automáticamente incidentes al personal adecuado, cerrar incidentes ruidosos o [falsos positivos](false-positives.md) conocidos, cambiar su gravedad y agregar etiquetas. También son el mecanismo por el que puede ejecutar cuadernos de estrategias en respuesta a los incidentes.

Los cuadernos de estrategias son colecciones de procedimientos que se pueden ejecutar desde Azure Sentinel en respuesta a una alerta o un incidente. Un cuaderno de estrategias puede ayudarle a automatizar y organizar su respuesta y se puede configurar para que se ejecute automáticamente cuando se generen alertas o incidentes específicos; para ello, se adjuntan a una regla de análisis o una regla de automatización, respectivamente. También se puede ejecutar manualmente a petición.

Los cuadernos de estrategias de Azure Sentinel se basan en flujos de trabajo integrados en [Azure Logic Apps](../logic-apps/logic-apps-overview.md), lo que significa que obtendrá toda la versatilidad, personalización y plantillas integradas de Logic Apps. Cada cuaderno de estrategias se crea para la suscripción específica a la que pertenece, pero en **Cuadernos de estrategia** se muestran todos los cuadernos de estrategias disponibles en todas las suscripciones seleccionadas.

> [!NOTE]
> Dado que los cuadernos de estrategias hacen uso de Azure Logic Apps, es posible que se apliquen cargos adicionales. Visite la página de precios de [Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/) para más información.

Por ejemplo, si desea detener el movimiento en la red y el robo de información por parte de usuarios potencialmente en peligro, puede crear una respuesta automatizada y de varias facetas para los incidentes generados por reglas que detecten usuarios en peligro. Comience por crear un cuaderno de estrategias que lleve a cabo las siguientes acciones:

1. Cuando una regla de automatización llama al cuaderno de estrategias y le pasa un incidente, el cuaderno de estrategias abre una incidencia en [ServiceNow](/connectors/service-now/) o en cualquier otro sistema de vales de TI.

1. Envía un mensaje al canal de operaciones de seguridad en [Microsoft Teams](/connectors/teams/) o [Slack](/connectors/slack/) para asegurarse de que los analistas de seguridad sean conscientes del incidente.

1. También envía toda la información del incidente en un mensaje de correo electrónico al administrador de seguridad y al administrador de red sénior. El mensaje de correo electrónico incluirá los botones de opciones de usuario **Bloquear** e **Ignorar**.

1. El cuaderno de estrategias espera hasta que se reciba una respuesta de los administradores y, después, continúa con los pasos siguientes.

1. Si los administradores eligen **Bloquear**, envía un comando a Azure AD para deshabilitar el usuario y uno al firewall para bloquear la dirección IP.

1. Si los administradores eligen **Ignorar**, el cuaderno de estrategias cierra el incidente en Azure Sentinel y el vale en ServiceNow.

Para desencadenar el cuaderno de estrategias, cree una regla de automatización que se ejecute cuando se generen estos incidentes. Esta regla llevará a cabo estos pasos:

1. La regla cambia el estado del incidente a **Activo**.

1. Asigna el incidente al analista encargado de administrar este tipo de incidentes.

1. Agrega la etiqueta "usuario en peligro".

1. Por último, llama al cuaderno de estrategias que acaba de crear. ([Se requieren permisos especiales para este paso](automate-responses-with-playbooks.md#incident-creation-automated-response)).

Los cuadernos de estrategias se pueden ejecutar automáticamente en respuesta a incidentes mediante la creación de reglas de automatización que llaman a los cuadernos de estrategias como acciones, como en el ejemplo anterior. También se pueden ejecutar automáticamente en respuesta a alertas, indicando a la regla de análisis que ejecute automáticamente uno o varios cuadernos de estrategias cuando se genere la alerta. 

También puede elegir ejecutar un cuaderno de estrategias manualmente a petición, como respuesta a una alerta seleccionada.

Obtenga una introducción más completa y detallada para la automatización de las respuestas a amenazas mediante [reglas de automatización](automate-incident-handling-with-automation-rules.md) y [cuadernos de estrategias](automate-responses-with-playbooks.md) en Azure Sentinel.

> [!IMPORTANT]
>
> - Las **reglas de automatización** y el uso del **desencadenador de incidentes** para los cuadernos de estrategias se encuentran actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="create-a-playbook"></a>Crear un cuaderno de estrategias

Para crear un nuevo cuaderno de estrategias en Azure Sentinel, siga estos pasos:

### <a name="prepare-the-playbook-and-logic-app"></a>Preparación del cuaderno de estrategias y la aplicación lógica

1. En el menú de navegación de **Azure Sentinel**, seleccione **Automation** (Automatización).

1. En el menú superior, seleccione **Create** (Crear) y **Add new playbook** (Agregar nuevo cuaderno de estrategias).

    :::image type="content" source="./media/tutorial-respond-threats-playbook/add-new-playbook.png" alt-text="Agregar un nuevo cuaderno de estrategias":::

    Se abrirá una nueva pestaña del explorador que le llevará al asistente para **Crear una aplicación lógica**.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/create-playbook.png" alt-text="Creación de una aplicación lógica":::

1. Escriba la **Suscripción** y el **Grupo de recursos** y asigne un nombre al cuaderno de estrategias en **Nombre de la aplicación lógica**.

1. En **Región**, seleccione la región de Azure donde se almacenará la información de la aplicación lógica.

1. Si desea supervisar la actividad de este cuaderno de estrategias con fines de diagnóstico, marque la casilla **Habilitar Log Analytics** y escriba el nombre del **Área de trabajo de Log Analytics**.

1. Si desea aplicar etiquetas al cuaderno de estrategias, haga clic en **Siguiente: Etiquetas >** (no conectado a las etiquetas que aplican las reglas de automatización. [Más información sobre etiquetas](../azure-resource-manager/management/tag-resources.md)). De lo contrario, haga clic en **Revisar y crear**. Confirme los detalles que ha proporcionado y haga clic en **Crear**.

1. Mientras se crea e implementa el cuaderno de estrategias (esto tardará unos minutos), se le dirigirá a una pantalla llamada **Microsoft.EmptyWorkflow**. Cuando aparezca el mensaje "Se completó la implementación", haga clic en **Ir al recurso**.

1. Se le dirigirá al [Diseñador de aplicaciones lógicas](../logic-apps/logic-apps-overview.md) del nuevo cuaderno de estrategias, donde puede empezar a diseñar el flujo de trabajo. Verá una pantalla con un breve vídeo introductorio y algunos desencadenadores y plantillas de aplicaciones lógicas que se usan con frecuencia. [Más información](../logic-apps/logic-apps-create-logic-apps-from-templates.md) sobre cómo crear un cuaderno de estrategias con Logic Apps.

1. Seleccione la plantilla **Aplicación lógica en blanco**.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/choose-playbook-template.png" alt-text="Galería de plantillas del Diseñador de aplicaciones lógicas":::

### <a name="choose-the-trigger"></a>Elección del desencadenador

Todos los cuadernos de estrategias deben comenzar con un desencadenador. El desencadenador define la acción que iniciará el cuaderno de estrategias y el esquema que este espera recibir.

1. En la barra de búsqueda, busque Azure Sentinel. Seleccione **Azure Sentinel** cuando aparezca en los resultados.

1. En la pestaña **Desencadenadores** resultante, verá los dos desencadenadores que ofrece Azure Sentinel:
    - When a response to an Azure Sentinel alert is triggered (Cuando se desencadena una respuesta a una alerta de Azure Sentinel)
    - When Azure Sentinel incident creation rule is triggered (Cuando se desencadena la regla de creación de incidentes de Azure Sentinel)

   Elija el desencadenador que coincida con el tipo de cuaderno de estrategias que va a crear.

    :::image type="content" source="./media/tutorial-respond-threats-playbook/choose-trigger.png" alt-text="Elección de un desencadenador para el cuaderno de estrategias":::

### <a name="add-actions"></a>Incorporación de una acción

Ahora puede definir lo que ocurre cuando se llame al cuaderno de estrategias. Puede agregar acciones, condiciones lógicas, bucles o condiciones switch-case; para ello, seleccione **Nuevo paso**. Esta selección abre un nuevo marco en el diseñador, donde puede elegir un sistema o una aplicación con la que interactuar o una condición para establecer. Escriba el nombre del sistema o la aplicación en la barra de búsqueda de la parte superior del marco y, a continuación, elija entre los resultados disponibles.

En cada uno de estos pasos, al hacer clic en cualquier campo se muestra un panel con dos menús: **Contenido dinámico** y **Expresión**. En el menú **Contenido dinámico**, puede agregar referencias a los atributos de la alerta o el incidente que se ha pasado al cuaderno de estrategias, incluidos los valores y atributos de todas las entidades implicadas. En el menú **Expresión**, puede elegir entre una gran biblioteca de funciones para agregar lógica adicional a los pasos.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/logic-app.png" alt-text="Diseñador de aplicación lógica":::

En esta captura de pantalla se muestran las acciones y condiciones que agregaría en la creación del cuaderno de estrategias descrito en el ejemplo que aparece al principio de este documento. La única diferencia es que en el cuaderno de estrategias que se muestra aquí, se usa el **desencadenador de alerta** en lugar del **desencadenador de incidente**. Esto significa que se llamará a este cuaderno de estrategias directamente desde una regla de análisis, no desde una regla de automatización. A continuación se describen las dos formas de llamar a un cuaderno de estrategias.

## <a name="automate-threat-responses"></a>Automatizar las respuestas frente a amenazas

Ha creado el cuaderno de estrategias y ha definido el desencadenador, ha establecido las condiciones y ha indicado las acciones que realizará y las salidas que generará. Ahora debe determinar los criterios según los cuales se ejecutará y configurar el mecanismo de automatización que lo ejecutará cuando se cumplan los criterios.

### <a name="respond-to-incidents"></a>Respuesta a incidentes

Puede usar un cuaderno de estrategias para responder a un **incidente** mediante la creación de una [regla de automatización](automate-incident-handling-with-automation-rules.md) que se ejecutará cuando se genere el incidente y, a su vez, llamará al cuaderno de estrategias.

Para crear una regla de automatización:

1. En la hoja **Automation** (Automatización) del menú de navegación de Azure Sentinel, seleccione **Create** (Crear) en el menú superior y luego **Add new rule** (Agregar nueva regla).

   :::image type="content" source="./media/tutorial-respond-threats-playbook/add-new-rule.png" alt-text="Adición de una nueva regla":::

1. Se abre el panel **Create new automation rule** (Creación de una regla de automatización). Escriba un nombre para la regla.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/create-automation-rule.png" alt-text="Creación de una regla de automatización":::

1. Si desea que la regla de automatización surta efecto solo en determinadas reglas de análisis, especifique cuáles; para ello, modifique la condición **If Analytics rule name** (Si el nombre de la regla de análisis).

1. Agregue cualquier otra condición de la que desee que dependa la activación de esta regla de automatización. Haga clic en **Add condition** (Agregar condición) y elija condiciones en la lista desplegable. La lista de condiciones se rellena mediante los campos de detalles de la alerta e identificador de la entidad.

1. Elija las acciones que desea que realice esta regla de automatización. Entre las acciones disponibles se incluyen **Asignar propietario**, **Cambiar estado**, **Cambiar gravedad**, **Agregar etiquetas** y **Ejecutar cuaderno de estrategias**. Puede agregar tantas acciones como desee.

1. Si agrega la acción **Ejecutar cuaderno de estrategias**, se le pedirá que elija en la lista desplegable de los cuadernos de estrategias disponibles. Solo se pueden ejecutar desde reglas de automatización los cuadernos de estrategias que comienzan por **incident trigger** (desencadenador de incidentes), por lo que solo estos aparecerán en la lista.

    > [!IMPORTANT]
    > Se deben conceder permisos explícitos a Azure Sentinel para ejecutar cuadernos de estrategias desde reglas de automatización. Si un cuaderno de estrategias aparece "atenuado" en la lista desplegable, significa que Sentinel no tiene permisos para el grupo de recursos de este cuaderno de estrategias. Haga clic en el vínculo **Manage playbook permissions** (Administrar permisos del cuaderno de estrategias) para asignar permisos.
    > En el panel **Manage permissions** (Administrar permisos) que se abre, active las casillas de los grupos de recursos que contienen los cuadernos de estrategias que desea ejecutar y haga clic en **Apply** (Aplicar).
    > :::image type="content" source="./media/tutorial-respond-threats-playbook/manage-permissions.png" alt-text="Administración de permisos":::
    > - Debe tener permisos de **propietario** en cualquier grupo de recursos al que desee conceder permisos para Azure Sentinel y debe tener el rol **Colaborador de aplicación lógica** en cualquier grupo de recursos que contenga los cuadernos de estrategias que desea ejecutar.
    > - En una implementación de varios inquilinos, si el cuaderno de estrategias que quiere ejecutar se encuentra en otro inquilino, debe conceder el permisos a Azure Sentinel para ejecutar el cuaderno de estrategias en el inquilino del cuaderno de estrategias.
    >    1. En el menú de navegación de Azure Sentinel del inquilino de los cuadernos de estrategias, seleccione **Settings** (Configuración).
    >    1. En la hoja **Settings** (Configuración), seleccione la pestaña **Settings** (Configuración) y, a continuación, el expansor de **Playbook permissions** (Permisos del cuaderno de estrategias).
    >    1. Haga clic en el botón **Configure permissions** (Configurar permisos) para abrir el panel **Manage permissions** (Administrar permisos) mencionado anteriormente y continúe tal y como se describe allí.

1. Establezca una fecha de expiración para la regla de automatización si desea que tenga una.

1. Escriba un número en **Order** (Orden) para determinar en qué punto de la secuencia de reglas de automatización se ejecutará esta regla.

1. Haga clic en **Aplicar**. ¡Y ya está!

[Descubra otras maneras](automate-incident-handling-with-automation-rules.md#creating-and-managing-automation-rules) de crear reglas de automatización.

### <a name="respond-to-alerts"></a>Respuesta a las alertas

Puede usar un cuaderno de estrategias para responder a una **alerta** mediante la creación de una **regla de análisis** o la edición de una existente, que se ejecuta cuando se genera la alerta, y la selección del cuaderno de estrategias como una respuesta automatizada en el [asistente para reglas de análisis](tutorial-detect-threats-custom.md).

1. En la hoja **Analytics** (Análisis) del menú de navegación de Azure Sentinel, seleccione la regla de análisis para la que desea automatizar la respuesta y haga clic en **Edit** (Editar) en el panel de detalles.

1. En la página **Analytics rule wizard - Edit existing rule** (Asistente para reglas de análisis: editar regla existente), seleccione la pestaña **Automated response** (Respuesta automática).

   :::image type="content" source="./media/tutorial-respond-threats-playbook/automated-response-tab.png" alt-text="Pestaña Respuesta automática":::

1. Elija el cuaderno de estrategias en la lista desplegable. Puede elegir más de un cuaderno de estrategias, pero solo estarán disponibles los cuadernos de estrategias que utilicen el **desencadenador de alertas**.

1. En la pestaña **Review and create** (Revisar y crear), seleccione **Save** (Guardar).

## <a name="run-a-playbook-on-demand"></a>Ejecución de un cuaderno de estrategias a petición

También puede ejecutar un cuaderno de estrategias a petición.

 > [!NOTE]
 > Solo se pueden ejecutar a petición los cuadernos de estrategias que usan el **desencadenador de alertas**.

Para ejecutar un cuaderno de estrategias a petición:

1. En la página **Incidents** (Incidentes), seleccione un incidente y haga clic en **View full details** (Ver detalles completos).

2. En la pestaña **Alertas**, haga clic en la alerta que desee para ejecutar el cuaderno de estrategias, desplácese completamente a la derecha, haga clic en **Ver cuadernos de estrategias** y seleccione un cuaderno de estrategias para **ejecutar** en el lista de cuadernos de estrategias disponibles en la suscripción.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a usar los cuadernos de estrategias y las reglas de automatización de Azure Sentinel para responder a las amenazas. 
- Más información en [Búsqueda de amenazas con Azure Sentinel](hunting.md).
