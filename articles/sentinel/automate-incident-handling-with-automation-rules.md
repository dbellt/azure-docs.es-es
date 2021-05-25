---
title: Automatización del control de incidentes en Azure Sentinel | Microsoft Docs
description: En este artículo se explica cómo usar las reglas de automatización para automatizar el control de incidentes a fin de maximizar la eficiencia y eficacia de su centro de operaciones de seguridad en respuesta a las amenazas de seguridad.
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
ms.topic: conceptual
ms.date: 03/14/2021
ms.author: yelevin
ms.openlocfilehash: 5ef5f465cbb8dfd044482bfad1eb72f1aa78df39
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783736"
---
# <a name="automate-incident-handling-in-azure-sentinel-with-automation-rules"></a>Automatización del control de incidentes en Azure Sentinel con las reglas de automatización

En este artículo se explica qué son las reglas de automatización de Azure Sentinel y cómo usarlas para implementar las operaciones de orquestación de seguridad, automatización y respuesta (SOAR), lo que aumenta la eficacia de su centro de operaciones de seguridad y ahorra tiempo y recursos.

> [!IMPORTANT]
>
> - La característica de **reglas de automatización** se encuentra actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="what-are-automation-rules"></a>¿Qué son las reglas de automatización?

Las reglas de automatización son un nuevo concepto en Azure Sentinel. Esta característica permite a los usuarios administrar de forma centralizada la automatización del control de incidentes. Además de permitirle asignar cuadernos de estrategias a incidentes (no solo a las alertas, como hasta ahora), las reglas de automatización también permiten automatizar las respuestas de varias reglas de análisis a la vez, etiquetar, asignar o cerrar incidentes automáticamente sin necesidad de cuadernos de estrategias y controlar el orden de las acciones que se ejecutan. Las reglas de automatización agilizarán el uso de la automatización en Azure Sentinel y le permitirán simplificar flujos de trabajo complejos de los procesos de orquestación de incidentes.

## <a name="components"></a>Componentes

Las reglas de automatización están conformadas por varios componentes:

### <a name="trigger"></a>Desencadenador

La creación de un incidente desencadena reglas de automatización. 

Tema de revisión: las reglas de análisis crean los incidentes a partir de las alertas, de las cuales hay cuatro tipos, como se explica en el tutorial [Detección de amenazas con reglas de análisis integradas en Azure Sentinel](tutorial-detect-threats-built-in.md).

### <a name="conditions"></a>Condiciones

Se pueden definir conjuntos complejos de condiciones para controlar cuándo se deberían ejecutarse las acciones (véase a continuación). Estas condiciones se basan normalmente en los estados o valores de los atributos de los incidentes y sus entidades, y pueden incluir operadores `AND` / `OR` / `NOT` / `CONTAINS`.

### <a name="actions"></a>Acciones

Se pueden definir acciones para que se ejecuten cuando se cumplan las condiciones (véase más arriba). Puede definir muchas acciones en una regla y puede elegir el orden en el que se ejecutarán (véase a continuación). Las siguientes acciones se pueden definir mediante reglas de automatización, sin necesidad de la [funcionalidad avanzada de un cuaderno de estrategias](automate-responses-with-playbooks.md):

- Cambiar el estado de un incidente para mantener el flujo de trabajo actualizado.

  - Al cambiar a "closed", especifique el [motivo de cierre](tutorial-investigate-cases.md#closing-an-incident) y agregue un comentario. Esto le ayudará a realizar un seguimiento del rendimiento y la eficacia, y a realizar ajustes para reducir los [falsos positivos](false-positives.md).

- Cambiar la gravedad de un incidente: puede volver a evaluar y cambiar la prioridad en función de la presencia, la ausencia, los valores o los atributos de las entidades implicadas en el incidente.

- Asignar un incidente a un propietario: esto le ayuda a dirigir los tipos de incidentes al personal más adecuado para que los solucionen, o al personal disponible.

- Agregar una etiqueta a un incidente: resulta útil para clasificar incidentes por asunto, por atacante o por cualquier otro denominador común.

Además, puede definir una acción para ejecutar un [cuaderno de estrategias](tutorial-respond-threats-playbook.md) a fin de realizar acciones de respuesta más complejas, incluidas las que impliquen sistemas externos. En las reglas de automatización solo se pueden usar los cuadernos de estrategias que se activan a través del [desencadenador de incidentes](automate-responses-with-playbooks.md#azure-logic-apps-basic-concepts). Puede definir una acción para que incluya varios cuadernos de estrategias, o combinaciones de cuadernos de estrategias y otras acciones, así como el orden en que se ejecutarán.

### <a name="expiration-date"></a>Fecha de expiración

Puede definir una fecha de expiración para una regla de automatización. La regla se deshabilitará después de esa fecha. Esto resulta útil para controlar (es decir, cerrar) los incidentes "irrelevantes" causados por las actividades planeadas de tiempo limitado, como las pruebas de penetración.

### <a name="order"></a>Pedido

Puede definir el orden en el que se ejecutarán las reglas de automatización. Las reglas de automatización posteriores evaluarán las condiciones del incidente en función de su estado después de que las reglas de automatización anteriores hayan actuado.

Por ejemplo, si la "Primera regla de automatización" cambió la gravedad de un incidente de Media a Baja y se definió una "Segunda regla de automatización" para que se ejecute solo en incidentes con una gravedad Media o superior, la segunda no se ejecutará en dicho incidente.

## <a name="common-use-cases-and-scenarios"></a>Casos de uso y escenarios comunes

### <a name="incident-triggered-automation"></a>Automatización desencadenada por incidentes

Hasta ahora, solo las alertas podían desencadenar una respuesta automatizada mediante los cuadernos de estrategias. Con las reglas de automatización, los incidentes ahora pueden activar cadenas de respuesta automatizadas, que pueden incluir nuevos cuadernos de estrategias desencadenados por incidentes ([se requieren permisos especiales](#permissions-for-automation-rules-to-run-playbooks)) cuando se crea un incidente. 

### <a name="trigger-playbooks-for-microsoft-providers"></a>Desencadenamiento de cuadernos de estrategias para proveedores de Microsoft

Las reglas de automatización proporcionan una forma de automatizar el control de las alertas de seguridad de Microsoft mediante la aplicación de estas reglas a los incidentes creados a partir de alertas. Las reglas de automatización pueden llamar a los cuadernos de estrategias ([se requieren permisos especiales](#permissions-for-automation-rules-to-run-playbooks)) y pasarles los incidentes junto con todos sus detalles, incluidas las alertas y entidades. En general, los procedimientos recomendados de Azure Sentinel dictan el uso de la cola de incidentes como punto crucial de las operaciones de seguridad.

Las siguientes son algunas alertas de seguridad de Microsoft:

- Microsoft Cloud App Security (MCAS)
- Azure AD Identity Protection
- Azure Defender (ASC)
- Defender para IoT (anteriormente ASC para IoT)
- Microsoft Defender para Office 365 (anteriormente Office 365 ATP)
- Microsoft Defender para punto de conexión (anteriormente MDATP)
- Microsoft Defender para identidad (anteriormente Azure ATP)

### <a name="multiple-sequenced-playbooksactions-in-a-single-rule"></a>Secuencia de varios cuadernos de estrategias o acciones en una sola regla

Ahora puede tener un control casi completo sobre el orden de ejecución de las acciones y cuadernos de estrategias en una única regla de automatización. También puede controlar el orden de ejecución de las reglas de automatización. Esto le permite simplificar en gran medida los cuadernos de estrategias, ya que puede reducirlos a una sola tarea o a una secuencia de tareas sencilla y directa, así como combinar estos pequeños cuadernos de estrategias de distintas maneras en reglas de automatización diferentes.

### <a name="assign-one-playbook-to-multiple-analytics-rules-at-once"></a>Asignación de un cuaderno de estrategias a varias reglas de análisis a la vez

Si tiene una tarea que quiere automatizar en todas las reglas de análisis (por ejemplo, la creación de una incidencia de soporte técnico en un sistema externo de incidencias), puede aplicar de una sola vez un cuaderno de estrategias a cualquiera de las reglas de análisis, incluidas todas las reglas futuras. Esto hace menos frustrantes las tareas de mantenimiento que son sencillas, pero repetitivas.

### <a name="automatic-assignment-of-incidents"></a>Asignación automática de incidentes

Puede asignar incidentes al propietario adecuado automáticamente. Si el centro de operaciones de seguridad tiene un analista que se especializa en una plataforma determinada, todos los incidentes relacionados con esa plataforma se pueden asignar automáticamente a dicho analista.

### <a name="incident-suppression"></a>Eliminación de incidentes

Puede usar reglas para resolver automáticamente los incidentes que son falsos positivos o positivos inofensivos sin usar los cuadernos de estrategias. Por ejemplo, al ejecutar pruebas de penetración, realizar tareas de mantenimiento o actualizaciones programadas, o probar procedimientos de automatización, se pueden crear muchos incidentes de tipo falso positivo que el centro de operaciones de seguridad quiere omitir. Una regla de automatización de tiempo limitado puede cerrar automáticamente estos incidentes a medida que se crean, a la vez que los etiqueta con un descriptor de la causa de su origen.

### <a name="time-limited-automation"></a>Automatización de tiempo limitado

Puede agregar fechas de expiración a las reglas de automatización. Puede haber casos distintos de la eliminación de incidentes que requieran de la automatización de tiempo limitado. Es posible que quiera asignar un tipo de incidente específico a un usuario determinado (por ejemplo, un empleado en prácticas o un consultor) durante un período de tiempo concreto. Si se conoce de antemano el período de tiempo, puede hacer que la regla se deshabilite cuando deje de ser relevante sin necesidad de recordar que debe hacerlo.

### <a name="automatically-tag-incidents"></a>Etiquetado automático de incidentes

Puede agregar automáticamente etiquetas de texto libre a los incidentes para agruparlos o clasificarlos según los criterios que elija.

## <a name="automation-rules-execution"></a>Ejecución de reglas de automatización

Las reglas de automatización se ejecutan secuencialmente, según el orden que se determine. Cada regla de automatización se ejecuta na vez finalizada la ejecución de la anterior. Dentro de una regla de automatización, todas las acciones se ejecutan de manera secuencial en el orden en que se definieron.

En el caso de las acciones del cuaderno de estrategias, hay un retraso de dos minutos entre el inicio de una acción del cuaderno y la siguiente acción de la lista.

### <a name="permissions-for-automation-rules-to-run-playbooks"></a>Permisos para que las reglas de automatización ejecuten cuadernos de estrategias

Cuando una regla de automatización de Azure Sentinel ejecuta un cuaderno de estrategias, usa una cuenta de servicio de Azure Sentinel especial autorizada específicamente para hacerlo. El uso de esta cuenta (en lugar de su cuenta de usuario) aumenta el nivel de seguridad del servicio.

Para que una regla de automatización ejecute un cuaderno de estrategias, se deben conceder a esta cuenta permisos explícitos para acceder al grupo de recursos en el que se encuentra el cuaderno. En ese momento, cualquier regla de automatización podrá ejecutar cualquier cuaderno de estrategias de ese grupo de recursos.

Al configurar una regla de automatización y agregar una acción para **ejecutar un cuaderno de estrategias**, aparecerá una lista desplegable de cuadernos. Los cuadernos de estrategias en los que Azure Sentinel no tiene permisos se mostrarán como no disponibles ("atenuados"). Puede conceder permiso a Azure Sentinel para acceder a los grupos de recursos de los cuadernos de estrategias en ese momento si selecciona el vínculo **Manage playbook permissions** (administrar permisos del cuaderno de estrategias).

> [!NOTE]
> **Permisos en una arquitectura de varios inquilinos**
>
> Las reglas de automatización son totalmente compatibles con las implementaciones en varias áreas de trabajo y multiinquilino (en el caso de varios inquilinos, mediante [Azure Lighthouse](extend-sentinel-across-workspaces-tenants.md#managing-workspaces-across-tenants-using-azure-lighthouse)).
>
> Por lo tanto, si la implementación de Azure Sentinel usa una arquitectura multiinquilino (por ejemplo, si es un MSSP), una regla de automatización en un inquilino puede ejecutar un cuaderno de estrategias ubicado en un inquilino diferente. No obstante, los permisos de Sentinel para ejecutar los cuadernos de estrategias deben definirse en el inquilino en el que se encuentran los cuadernos de estrategias, no en el inquilino en el que están definidas las reglas.

## <a name="creating-and-managing-automation-rules"></a>Creación y administración de reglas de automatización

Puede crear y administrar reglas de automatización desde distintas partes en la experiencia de Azure Sentinel, en función de su caso de uso y requisitos específicos.

- **Hoja Automatización**

    Las reglas de automatización se pueden administrar de forma centralizada en la nueva hoja **Automatización** (que reemplaza a la hoja **Cuadernos de estrategias**), en la pestaña **Reglas de automatización** (ahora también es posible administrar cuadernos de estrategias en esta hoja, en la pestaña **Cuadernos de estrategias**). En dicha hoja se pueden crear reglas de automatización y editar las existentes. También puede arrastrar las reglas de automatización para cambiar el orden de ejecución, además de habilitarlas o deshabilitarlas.

    En la hoja **Automatización**, verá todas las reglas que están definidas en el área de trabajo, junto con su estado (Habilitada/Deshabilitada) y a qué reglas de análisis se aplican.

    Cuando necesite una regla de automatización para aplicarla a muchas reglas de análisis, créela directamente en la hoja **Automatización**. En el menú superior, haga clic en **Crear** y **Agregar nueva regla** para abrir el panel **Creación de una regla de automatización**. Desde aquí tiene total flexibilidad para configurar la regla: puede aplicarla a cualquier regla de análisis (incluidas las futuras) y definir la más amplia gama de condiciones y acciones.

- **Asistente para reglas de análisis**

    En la pestaña **Respuesta automatizada** del Asistente para reglas de análisis puede ver, administrar y crear reglas de automatización que se aplican a la regla de análisis específica que se está creando o editando en el Asistente.

    El Asistente para reglas se abrirá si hace clic en **Crear** y en uno de los tipos de reglas (**Regla de consulta programada** o **Regla de creación de incidentes de Microsoft**) desde el menú superior de la hoja **Análisis**, o bien si selecciona una regla de análisis existente y hace clic en **Editar**. Al seleccionar la pestaña **Respuesta automatizada**, verá una sección llamada **Automatización de los incidentes**, en la que se mostrarán las reglas de automatización que se aplican actualmente a esta regla. Puede seleccionar una regla de automatización existente para editarla o hacer clic en **Agregar nueva** para crear una.

    Notará que, al crear la regla de automatización desde aquí, el panel **Creación de una regla de automatización** muestra la condición de la **regla de análisis** como no disponible, ya que esta regla está configurada para aplicarse solo a la regla de análisis que está editando en el asistente. Todas las demás opciones de configuración siguen disponibles.

- **Hoja Incidentes**

    También puede crear una regla de automatización desde la hoja **Incidentes** con el fin de responder a un único incidente repetitivo. Esto resulta útil cuando se crean [reglas de eliminación](#incident-suppression) para cerrar automáticamente los incidentes "irrelevantes". Seleccione un incidente de la cola y haga clic en **Crear regla de automatización** en el menú superior.

    Observará que el panel **Crear nueva regla de automatización** ha rellenado todos los campos con los valores del incidente. Asigna a la regla el mismo nombre que el incidente, la aplica a la regla de análisis que generó el incidente y usa todas las entidades disponibles en el incidente como condiciones de la regla. También sugiere una acción de eliminación (cierre) de manera predeterminada y sugiere una fecha de expiración para la regla. Puede agregar o quitar condiciones y acciones, así como cambiar la fecha de expiración como desee.

## <a name="auditing-automation-rule-activity"></a>Auditoría de la actividad de la regla de automatización

Puede que le interese saber lo que ha ocurrido en un incidente determinado y qué acciones realizó o no una regla de automatización concreta. Tiene un registro completo de crónicas del incidente a su disposición en la tabla *SecurityIncident* de la hoja **Registros**. Use la siguiente consulta para ver toda la actividad de su regla de automatización:

```kusto
SecurityIncident
| where ModifiedBy contains "Automation"
```

## <a name="next-steps"></a>Pasos siguientes

En este documento, aprendió a usar las reglas de automatización para administrar la cola de incidentes de Azure Sentinel e implementar una automatización básica para el control de incidentes.

- Para más información sobre las opciones avanzadas de automatización, consulte [Automatización de la respuesta a amenazas con los cuadernos de estrategias de Azure Sentinel](automate-responses-with-playbooks.md).
- Para obtener ayuda sobre la implementación de reglas de automatización y cuadernos de estrategias, consulte [Tutorial: Uso de cuadernos de estrategias para automatizar las respuestas a amenazas en Azure Sentinel](tutorial-respond-threats-playbook.md).
