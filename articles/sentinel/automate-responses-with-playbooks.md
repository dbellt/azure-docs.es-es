---
title: Automatización de la respuesta a amenazas con cuadernos de estrategias en Azure Sentinel | Microsoft Docs
description: En este artículo se explica la automatización en Azure Sentinel y se muestra cómo usar cuadernos de estrategias para automatizar la prevención y la respuesta a amenazas.
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
ms.date: 03/17/2021
ms.author: yelevin
ms.openlocfilehash: 0687b3bf486d2496763237164536be34f504f7ed
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112070892"
---
# <a name="automate-threat-response-with-playbooks-in-azure-sentinel"></a>Automatización de la respuesta a amenazas con cuadernos de estrategias en Azure Sentinel

En este artículo se explica qué son los cuadernos de estrategias de Azure Sentinel y cómo usarlos para implementar las operaciones de orquestación, automatización y respuesta de la seguridad (SOAR), logrando mejores resultados a la vez que se ahorran tiempo y recursos.

## <a name="what-is-a-playbook"></a>¿Qué es un cuaderno de estrategias?

Los equipos de SIEM y SOC normalmente se sobrecargan con alertas e incidentes de seguridad de forma regular, en volúmenes tan grandes que el personal disponible está abrumado. Esto genera, con demasiada frecuencia, situaciones en las que se ignoran muchas alertas y no se pueden investigar muchos incidentes, lo que hace que la organización sea vulnerable a ataques que pasan desapercibidos.

Muchas de estas alertas e incidentes, si no la mayoría, se ajustan a patrones recurrentes que se pueden resolver mediante conjuntos de acciones de corrección específicas y definidas.

Un cuaderno de estrategias es una colección de estas acciones correctivas que se puede ejecutar desde Azure Sentinel de forma rutinaria. Un cuaderno de estrategias puede ayudarle a [**automatizar y organizar la respuesta a las amenazas**](tutorial-respond-threats-playbook.md); se puede ejecutar manualmente o establecer para que se ejecute automáticamente en respuesta a alertas o incidentes específicos, cuando se desencadena mediante una regla de análisis o una regla de automatización, respectivamente.

Los cuadernos de estrategias se crean y se aplican en el nivel de suscripción, pero en la pestaña **Playbooks** (Cuadernos de estrategias) (en la hoja nueva **Automation** [Automatización]) se muestran todos los cuadernos de estrategias disponibles en todas las suscripciones seleccionadas.

### <a name="azure-logic-apps-basic-concepts"></a>Conceptos básicos de Azure Logic Apps

Los cuadernos de estrategias de Azure Sentinel se basan en flujos de trabajo integrados en [Azure Log Apps](../logic-apps/logic-apps-overview.md), un servicio en la nube que le ayuda a programar, automatizar y organizar tareas y flujos de trabajo en los sistemas de toda la empresa. Esto significa que los cuadernos de estrategias pueden aprovechar toda la eficacia y la personalización de las plantillas integradas de Logic Apps.

> [!NOTE]
> Dado que Azure Logic Apps son recursos independientes, es posible que se apliquen cargos adicionales. Visite la página de precios de [Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/) para más información.

Azure Logic Apps se comunica con otros sistemas y servicios mediante conectores. A continuación se explican brevemente los conectores y algunos de sus atributos más importantes:

- **Conector administrado:** conjunto de acciones y desencadenadores que se encapsulan llamadas de API a un producto o servicio determinado. Azure Logic Apps ofrece cientos de conectores para comunicarse con servicios de Microsoft y de otros fabricantes.
  - [Lista de todos los conectores de Logic Apps y su documentación](/connectors/connector-reference/)

- **Conector personalizado:** es posible que quiera comunicarse con servicios que no están disponibles como conectores integrados previamente. Para resolver esta necesidad, los conectores personalizados permiten crear (e incluso compartir) un conector y definir sus propios desencadenadores y acciones.
  - [Creación de un conector personalizado en Azure Logic Apps](/connectors/custom-connectors/create-logic-apps-connector)

- **Conector de Azure Sentinel:** para crear cuadernos de estrategias que interactúen con Azure Sentinel, use el conector de Azure Sentinel.
  - [Documentación del conector de Azure Sentinel](/connectors/azuresentinel/)

- **Desencadenador:** componente de un conector que inicia un cuaderno de estrategias. Define el esquema que el cuaderno de estrategias espera recibir cuando se desencadene. Actualmente, el conector de Azure Sentinel tiene dos desencadenadores:
  - [Desencadenador de alerta](/connectors/azuresentinel/#triggers): el cuaderno de estrategias recibe la alerta como entrada.
  - [Desencadenador de incidente](/connectors/azuresentinel/#triggers): el cuaderno recibe el incidente como entrada, junto con todas las alertas y entidades incluidas.

    > [!IMPORTANT]
    >
    > - La característica de **desencadenador de incidente** para los cuadernos de estrategias se encuentra actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

- **Acciones**: las acciones son todos los pasos que se producen después del desencadenador. Se pueden organizar secuencialmente, en paralelo o en una matriz de condiciones complejas.

- **Campos dinámicos:** campos temporales, determinados por el esquema de salida de los desencadenadores y las acciones, y rellenados por su salida real, que se pueden usar en las acciones siguientes.

### <a name="permissions-required"></a>Permisos necesarios

 Para dar al equipo de operaciones de seguridad la capacidad de usar Logic Apps para las operaciones de orquestación, automatización y respuesta de la seguridad (SOAR), es decir, para crear y ejecutar cuadernos de estrategias en Azure Sentinel, puede asignar roles de Azure, ya sea a miembros específicos del equipo de operaciones de seguridad o a todo el equipo. A continuación se describen los distintos roles disponibles y las tareas para las que se deben asignar:

#### <a name="azure-roles-for-logic-apps"></a>Roles de Azure para Logic Apps

- **Colaborador de aplicaciones lógicas** permite administrar aplicaciones lógicas y ejecutar cuadernos de estrategias, pero no puede cambiar el acceso a ellos (para ello necesita el rol **Propietario**).
- **Operador de aplicaciones lógicas** permite leer, habilitar y deshabilitar aplicaciones lógicas, pero no puede editarlas ni actualizarlas.

#### <a name="azure-roles-for-sentinel"></a>Roles de Azure para Sentinel

- El rol **Colaborador de Azure Sentinel** permite adjuntar un cuaderno de estrategias a una regla de análisis.
- El rol **Respondedor de Azure Sentinel** permite ejecutar un cuaderno de estrategias manualmente.
- **Colaborador de automatización de Azure Sentinel** permite que las reglas de automatización ejecuten los cuadernos de estrategias. No se usa para ningún otro fin.

#### <a name="learn-more"></a>Más información

- [Más información sobre los roles de Azure en Azure Logic Apps](../logic-apps/logic-apps-securing-a-logic-app.md#access-to-logic-app-operations).
- [Más información sobre los roles de Azure en Azure Sentinel](roles.md).

## <a name="steps-for-creating-a-playbook"></a>Pasos para crear un cuaderno de estrategias

- [Defina el escenario de automatización](#use-cases-for-playbooks).

- [Cree la aplicación lógica de Azure](tutorial-respond-threats-playbook.md).

- [Pruebe la aplicación lógica](#run-a-playbook-manually-on-an-alert).

- Adjunte el cuaderno de estrategias a una [regla de automatización](#incident-creation-automated-response) o a una [regla de análisis](#alert-creation-automated-response), o bien [ejecútelo manualmente cuando sea necesario](#run-a-playbook-manually-on-an-alert).

### <a name="use-cases-for-playbooks"></a>Casos de uso de los cuadernos de estrategias

La plataforma de Azure Logic Apps ofrece cientos de acciones y desencadenadores, por lo que se puede crear prácticamente cualquier escenario de automatización. Azure Sentinel recomienda empezar con los siguientes escenarios de SOC:

#### <a name="enrichment"></a>Enriquecimiento

**Recopile datos y asócielos al incidente para tomar decisiones más inteligentes.**

Por ejemplo:

Se ha creado un incidente de Azure Sentinel a partir de una alerta mediante una regla de análisis que genera entidades de direcciones IP.

El incidente desencadena una regla de automatización que ejecuta un cuaderno de estrategias con los pasos siguientes:

- Se inicia cuando [se crea un nuevo incidente de Azure Sentinel](/connectors/azuresentinel/#triggers). Las entidades representadas en el incidente se almacenan en los campos dinámicos del desencadenador de incidente.

- Para cada dirección IP, se consulta un proveedor externo de inteligencia sobre amenazas, como [Virus Total](https://www.virustotal.com/), para recuperar más datos.

- Se agregan los datos y los detalles devueltos como comentarios del incidente.

#### <a name="bi-directional-sync"></a>Sincronización bidireccional

**Los cuadernos de estrategias se pueden usar para sincronizar los incidentes de Azure Sentinel con otros sistemas de vales.**

Por ejemplo:

Cree una regla de automatización para cada creación de incidentes y adjunte un cuaderno de estrategias que abra una incidencia en ServiceNow:

- Se inicia cuando [se crea un nuevo incidente de Azure Sentinel](/connectors/azuresentinel/#triggers).

- Se crea un nuevo vale en [ServiceNow](/connectors/service-now/).

- Se incluye en el vale el nombre del incidente, los campos importantes y una dirección URL para el incidente de Azure Sentinel para facilitar la dinamización.

#### <a name="orchestration"></a>Orquestación

**Utilice la plataforma de chat del SOC para controlar mejor la cola de incidentes.**

Por ejemplo:

Se ha creado un incidente de Azure Sentinel a partir de una alerta mediante una regla de análisis que genera entidades de nombres de usuario y direcciones IP.

El incidente desencadena una regla de automatización que ejecuta un cuaderno de estrategias con los pasos siguientes:

- Se inicia cuando [se crea un nuevo incidente de Azure Sentinel](/connectors/azuresentinel/#triggers).

- Se envía un mensaje al canal de operaciones de seguridad en [Microsoft Teams](/connectors/teams/) o [Slack](/connectors/slack/) para asegurarse de que los analistas de seguridad son conscientes del incidente.

- Se envía toda la información de la alerta por correo electrónico al administrador de seguridad y al administrador de red sénior. El mensaje de correo electrónico incluirá los botones de opciones de usuario **Bloquear** e **Ignorar**.

- Se espera hasta que se reciba una respuesta de los administradores y, a continuación, se continúa con la ejecución.

- Si los administradores han elegido **Bloquear**, se envía un comando al firewall para bloquear la dirección IP de la alerta y otro a Azure AD para deshabilitar el usuario.

#### <a name="response"></a>Response

**Responda inmediatamente a las amenazas con las mínimas dependencias humanas.**

Dos ejemplos:

**Ejemplo 1:** Responder a una regla de análisis que indica que un usuario está en peligro, según la detección de [Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md):

   - Se inicia cuando [se crea un nuevo incidente de Azure Sentinel](/connectors/azuresentinel/#triggers).

   - Para cada entidad de usuario del incidente sospechosa de estar en peligro:

     - Se envía un mensaje de Teams al usuario solicitando la confirmación de que el usuario realizó la acción sospechosa.

     - Se comprueba con Azure AD Identity Protection para [confirmar que el estado del usuario está en peligro](/connectors/azureadip/#confirm-a-risky-user-as-compromised). Azure AD Identity Protection etiquetará al usuario como **de riesgo** y aplicará cualquier directiva de cumplimiento ya configurada (por ejemplo, para requerir que el usuario use MFA la próxima vez que inicie sesión).

        > [!NOTE]
        > El cuaderno de estrategias no inicia ninguna acción de cumplimiento en el usuario, ni inicia ninguna configuración de directiva de cumplimiento. Solo indica a Azure AD Identity Protection que aplique las directivas ya definidas según corresponda. Cualquier medida de cumplimiento depende en su totalidad de las directivas correspondientes definidas en Azure AD Identity Protection.

**Ejemplo 2:** Responder a una regla de análisis que indica que una máquina está en peligro, según la detección de [Microsoft Defender para punto de conexión](/windows/security/threat-protection/):

   - Se inicia cuando [se crea un nuevo incidente de Azure Sentinel](/connectors/azuresentinel/#triggers).

   - Se usa la acción **Entidades: obtener hosts** de Azure Sentinel para analizar las máquinas sospechosas que se incluyen en las entidades del incidente.

   - Se emite un comando a Microsoft Defender para punto de conexión para [aislar las máquinas](/connectors/wdatp/#actions---isolate-machine) de la alerta.

## <a name="how-to-run-a-playbook"></a>Ejecución de un cuaderno de estrategias

Los cuadernos de estrategias se pueden ejecutar de forma **manual** o **automática**.

La ejecución manual de los mismos significa que cuando obtiene una alerta, puede optar por ejecutar un cuaderno de estrategias a petición como respuesta a la alerta seleccionada. Actualmente, esta característica solo se admite para alertas, no para incidentes.

Ejecutarlos automáticamente significa establecerlos como una respuesta automatizada de una regla de análisis (para las alertas) o como una acción de una regla de automatización (para los incidentes). [Más información sobre las reglas de automatización](automate-incident-handling-with-automation-rules.md).

### <a name="set-an-automated-response"></a>Establecimiento de una respuesta automatizada

Los equipos de operaciones de seguridad pueden reducir de forma significativa su carga de trabajo al automatizar completamente las respuestas rutinarias a tipos de incidentes y alertas recurrentes, lo que le permite concentrarse más en las alertas y los incidentes singulares, analizar patrones, buscar amenazas y mucho más.

La configuración de la respuesta automatizada significa que cada vez que se desencadena una regla de análisis, además de crear una alerta, la regla ejecutará un cuaderno de estrategias, que recibirá como entrada la alerta creada por la regla.

Si la alerta crea un incidente, el incidente desencadenará una regla de automatización que, a su vez, puede ejecutar un cuaderno de estrategias que recibirá como entrada el incidente creado por la alerta.

#### <a name="alert-creation-automated-response"></a>Respuesta automatizada en la creación de alertas

En el caso de los cuadernos de estrategias desencadenados por la creación de alertas y que reciben alertas como entradas (el primer paso es "Cuando se desencadena una alerta de Azure Sentinel"), adjunte el cuaderno de estrategias a una regla de análisis:

1. Edite la [regla de análisis](tutorial-detect-threats-custom.md) que genera la alerta para la que desea definir una respuesta automatizada.

1. En **Alert automation** (Automatización de alertas), en la pestaña **Automated response** (Respuesta automática), seleccione el o los cuadernos de estrategias que desencadenará esta regla de análisis cuando se cree una alerta.

#### <a name="incident-creation-automated-response"></a>Respuesta automatizada en la creación de incidentes

En el caso de los cuadernos de estrategias desencadenados por la creación de incidentes y que reciben incidentes como entradas (el primer paso es "Cuando se desencadena un incidente de Azure Sentinel"), cree una regla de automatización y defina una acción **Ejecutar cuaderno de estrategias** en ella. Esto se puede llevar a cabo de 2 maneras:

- Edite la regla de análisis que genera el incidente para el que desea definir una respuesta automatizada. En **Incident automation** (Automatización de incidentes), en la pestaña **Automated response** (Respuesta automática), cree una regla de automatización. Esto creará una respuesta automatizada solo para esta regla de análisis.

- En la pestaña **Automation rules** (Reglas de automatización) de la hoja **Automation** (Automatización), cree una nueva regla de automatización y especifique las condiciones adecuadas y las acciones deseadas. Esta regla de automatización se aplicará a cualquier regla de análisis que cumpla las condiciones especificadas.

    > [!NOTE]
    > **Las reglas de automatización de Azure Sentinel requieren permisos para ejecutar los cuadernos de estrategias.**
    >
    > Para ejecutar un cuaderno de estrategias desde una regla de automatización, Azure Sentinel usa una cuenta de servicio específicamente autorizada para ello. El uso de esta cuenta (en contraposición a su cuenta de usuario) aumenta el nivel de seguridad del servicio y permite que la API de reglas de automatización admita los casos de uso de CI/CD.
    >
    > A esta cuenta se le deben conceder permisos explícitos (con la forma del rol **Colaborador de automatización de Azure Sentinel**) en el grupo de recursos donde reside el cuaderno de estrategias. En ese momento, cualquier regla de automatización podrá ejecutar cualquier cuaderno de estrategias de ese grupo de recursos.
    >
    > Al agregar la acción **Ejecutar cuaderno de estrategias** a una regla de automatización, aparecerá una lista desplegable de cuadernos de estrategias para su selección. Los cuadernos de estrategias en los que Azure Sentinel no tiene permisos se mostrarán como no disponibles ("atenuados"). Puede conceder permisos a Azure Sentinel aquí mismo; para ello, seleccione el vínculo **Manage playbook permissions** (Administrar permisos del cuaderno de estrategias).
    >
    > En un escenario de varios inquilinos ([Lighthouse](extend-sentinel-across-workspaces-tenants.md#managing-workspaces-across-tenants-using-azure-lighthouse)), debe definir los permisos en el inquilino en el que reside el cuaderno de estrategias, incluso si la regla de automatización que llama al cuaderno de estrategias está en otro inquilino. Para ello, debe tener permisos de **Propietario** en el grupo de recursos del cuaderno de estrategias.
    >
    > Hay un escenario único al que se enfrenta un **proveedor de servicios de seguridad administrada (MSSP)** , en el que un proveedor de servicios, que ha iniciado sesión en su propio inquilino, crea una regla de automatización en el área de trabajo de un cliente mediante [Azure Lighthouse](../lighthouse/index.yml). A continuación, esta regla de automatización llama a un cuaderno de estrategias que pertenece al inquilino del cliente. En este caso, se debe conceder permisos a Azure Sentinel en **_ambos inquilinos_ *. En el inquilino del cliente, se conceden en el panel* Manage playbook permissions** (Administrar permisos del cuaderno de estrategias), al igual que en el escenario multiinquilino normal. Para conceder los permisos pertinentes en el inquilino del proveedor de servicios, debe agregar una delegación de Azure Lighthouse adicional que conceda derechos de acceso a la aplicación de **Azure Security Insights**, con el rol **Colaborador de Azure Sentinel Automation**, en el grupo de recursos donde reside el cuaderno de estrategias. [Más información sobre cómo agregar esta delegación](tutorial-respond-threats-playbook.md#permissions-to-run-playbooks).

Consulte las [instrucciones completas para crear reglas de automatización](tutorial-respond-threats-playbook.md#respond-to-incidents).

### <a name="run-a-playbook-manually-on-an-alert"></a>Ejecución manual de un cuaderno de estrategias en una alerta

La activación manual está disponible en el portal de Azure Sentinel en las siguientes hojas:

- En la vista **Incidents** (Incidentes), elija un incidente específico, abra su pestaña **Alerts** (Alertas) y elija una alerta.

- En **Investigation** (Investigación), elija una alerta específica.

1. Haga clic en **View playbooks** (Ver cuadernos de estrategias) de la alerta seleccionada. Obtendrá una lista de todos los cuadernos de estrategias que comienzan por **Cuando se desencadena una alerta de Azure Sentinel** a los que tiene acceso.

1. Haga clic en **Run** (Ejecutar) en la línea de un cuaderno de estrategias específico para desencadenarlo.

1. Seleccione la pestaña **Runs** (Ejecuciones) para ver una lista de todas las veces que se ha ejecutado un cuaderno de estrategias en esta alerta. Una ejecución completada recientemente puede tardar unos segundos en aparecer en esta lista.

1. Al hacer clic en una ejecución específica, se abrirá el registro de ejecución completo en Logic Apps.

### <a name="run-a-playbook-manually-on-an-incident"></a>Ejecución manual de un cuaderno de estrategias en un incidente

Aún no se admite. <!--make this a note instead? -->

## <a name="manage-your-playbooks"></a>Administración de los cuadernos de estrategias

En la pestaña **Playbooks** (Cuadernos de estrategias), aparece una lista de todos los cuadernos de estrategias a los que tiene acceso, filtrados por las suscripciones que se muestran actualmente en Azure. El filtro de suscripciones está disponible en el menú **Directory + subscription** (Directorio + suscripción) del encabezado de página global.

Al hacer clic en un nombre de cuaderno de estrategias, se le dirigirá a la página principal del cuaderno de estrategias en Logic Apps. La columna **Estado** indica si está habilitado o deshabilitado.

El **tipo de desencadenador** representa el desencadenador de Logic apps que inicia este cuaderno de estrategias.

| Tipo de desencadenador | Indica los tipos de componente del cuaderno de estrategias |
|-|-|
| **Incidente o alerta de Azure Sentinel** | El cuaderno de estrategias se inicia con uno de los desencadenadores de Sentinel (alerta o incidente). |
| **Uso de una acción de Azure Sentinel** | El cuaderno de estrategias se inicia con un desencadenador que no es de Sentinel, pero usa una acción de Azure Sentinel. |
| **Otros** | El cuaderno de estrategias no incluye ningún componente de Sentinel. |
| **No inicializado** | El cuaderno de estrategias se ha creado, pero no contiene componentes (desencadenadores o acciones). |
|

En la página de la aplicación lógica del cuaderno de estrategias, puede ver más información sobre el cuaderno de estrategias, incluido un registro de todas las veces que se ha ejecutado y el resultado (éxito o error, y otros detalles). También puede abrir el diseñador de aplicaciones lógicas y editar el cuaderno de estrategias directamente, si tiene los permisos adecuados.

### <a name="api-connections"></a>Conexiones de API

Las conexiones de API se usan para conectar Logic Apps a otros servicios. Cada vez que se realiza una nueva autenticación en un conector de Logic Apps, se crea un nuevo recurso de tipo **conexión de API**, que contiene la información proporcionada al configurar el acceso al servicio.

Para ver todas las conexiones de API, escriba *conexiones de API* en el cuadro de búsqueda del encabezado de Azure Portal. Anote las columnas de interés:

- Nombre para mostrar: el nombre "descriptivo" que se asigna a la conexión cada vez que se crea una.
- Estado: indica el estado de la conexión: error o conectado.
- Grupo de recursos: las conexiones de API se crean en el grupo de recursos del recurso del cuaderno de estrategias (Logic Apps).

Otra manera de ver las conexiones de API sería ir a la hoja **Todos los recursos** y filtrar por el tipo *Conexión de API*. De esta manera se permite la selección, el etiquetado y la eliminación de varias conexiones a la vez.

Para cambiar la autorización de una conexión existente, escriba el recurso de conexión y seleccione **Editar conexión de API**.

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Uso de cuadernos de estrategias con reglas de automatización en Azure Sentinel](tutorial-respond-threats-playbook.md)
