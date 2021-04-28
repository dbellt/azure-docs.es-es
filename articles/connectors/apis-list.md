---
title: Conectores para Azure Logic Apps
description: Introducción a los conectores para crear flujos de trabajo automatizados con Azure Logic Apps. Obtenga información sobre cómo funcionan los distintos tipos de conectores, desencadenadores y acciones.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, azla
ms.topic: conceptual
ms.date: 04/20/2021
ms.custom: contperf-fy21q4
ms.openlocfilehash: 9e138cb7fb591728e7bc6a02ff61b3167d13da30
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771360"
---
# <a name="connectors-for-azure-logic-apps"></a>Conectores para Azure Logic Apps

En Azure Logic Apps, los *conectores* le permiten acceder rápidamente a eventos, datos y acciones desde otras aplicaciones, servicios, sistemas, protocolos y plataformas. A menudo puede realizar estas tareas sin código adicional. También puede usar conectores para crear flujos de trabajo de Logic Apps que usan esta información en entornos híbridos, locales o basados en la nube.

Existen cientos de conectores disponibles para Logic Apps. Como resultado, esta documentación se centra en algunos de los conectores más populares y que se usan con más frecuencia en Logic Apps. Para obtener información completa sobre los conectores de Logic Apps, Microsoft Power Automate y Microsoft Power Apps, consulte la [documentación de conectores](/connectors). 

Para obtener información sobre los precios, consulte el [Modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md) y los [Detalles de precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

Para integrar la aplicación lógica con un servicio o API que no tiene conector, puede llamar directamente al servicio a través de un protocolo como HTTP, o bien [crear un conector personalizado](#custom-apis-and-connectors).

## <a name="what-are-connectors"></a>¿Qué son los conectores?

En Azure Logic Apps, los conectores proporcionan *desencadenadores* y *acciones* que se usan para realizar tareas en el flujo de trabajo de la aplicación lógica. Cada desencadenador y acción tienen propiedades que puede configurar. Algunos desencadenadores y acciones requieren que [cree y configure conexiones](#connection-configuration) para que el flujo de trabajo pueda obtener acceso a un servicio o sistema específico.

### <a name="triggers"></a>Desencadenadores

Un *desencadenador* siempre es el primer paso de cualquier flujo de trabajo, que especifica el evento que inicia ese flujo de trabajo. Hay varios tipos de desencadenadores:

- Los *desencadenadores de sondeo* comprueban periódicamente un servicio o sistema específico según una programación especificada, para comprobar si hay nuevos datos o un evento específico. Si hay nuevos datos disponibles o se produce un evento específico, estos desencadenadores crean y ejecutan una nueva instancia del flujo de trabajo. Esta nueva instancia puede usar los datos que se pasan como entrada.
- Los *desencadenadores de inserción* escuchan datos nuevos o si se produce un evento, sin tener que realizar un sondeo. Si hay nuevos datos disponibles o se produce un evento específico, estos desencadenadores crean y ejecutan una nueva instancia de la aplicación lógica. Esta nueva instancia puede usar los datos que se pasan como entrada.

Por ejemplo, es posible que quiera crear un flujo de trabajo que se active cuando se carga un archivo en el servidor FTP. Puede agregar el desencadenador del conector FTP denominado **Cuando se agrega o modifica un archivo**, como primer paso del flujo de trabajo. A continuación, puede especificar una programación para comprobar periódicamente si hay eventos de carga.

Un desencadenador también pasa todas las entradas y otros datos necesarios al flujo de trabajo, donde las acciones posteriores pueden hacer referencia a los datos y usarlos a lo largo del flujo de trabajo. Por ejemplo, puede usar el desencadenador de Office 365 Outlook denominado **Cuando llega un nuevo correo electrónico**. Puede configurar este desencadenador para que se active según el contenido de cada nuevo correo electrónico, como el remitente, la línea de asunto, el cuerpo, los datos adjuntos, etc. A continuación, puede procesar esa información en la aplicación lógica mediante las acciones.

### <a name="actions"></a>Acciones

Una *acción* es una operación que sigue al desencadenador y realiza algún tipo de tarea en el flujo de trabajo. Puede usar varias acciones en la aplicación lógica. Por ejemplo, puede tener un desencadenador de SQL que detecte nuevos datos de cliente en una base de datos SQL. El flujo de trabajo puede incluir una primera acción de SQL que obtenga los datos del cliente, además de otra acción que no es necesariamente una acción de SQL, que procesa los datos.

## <a name="connector-categories"></a>Categorías del conector

En Logic Apps, normalmente hay versiones integradas o administradas del conector de desencadenadores y de acciones. Existe un pequeño número de desencadenadores y acciones disponibles en ambas versiones. Las versiones específicas dependen de si va a crear una aplicación lógica multiinquilino o de un solo inquilino, que actualmente solo está disponible en la [versión preliminar de Logic Apps](../logic-apps/logic-apps-overview-preview.md).

Los [desencadenadores y las acciones integrados](built-in.md) se ejecutan de forma nativa en el entorno de ejecución de Logic Apps, y no requieren la creación de conexiones; asimismo, realizan estos tipos de tareas:

- [Ejecución del código en los flujos de trabajo](built-in.md#run-code-from-workflows).
- [Organización y control de los datos](built-in.md#control-workflow).
- [Administración o manipulación de los datos](built-in.md#manage-or-manipulate-data).

Microsoft se encarga de implementar, hospedar y administrar los [conectores administrados](managed.md). Estos conectores proporcionan desencadenadores y acciones para los servicios en la nube, los sistemas locales o ambos. Entre ellas se incluyen las siguientes:

- [Conectores locales](managed.md#on-premises-connectors), que le ayudan a obtener acceso a los datos y recursos en sistemas locales.
- [Conectores empresariales](managed.md#enterprise-connectors), que son versiones de algunos conectores de Logic Apps que proporcionan acceso a sistemas empresariales.
- [Conectores de cuentas de integración](managed.md#integration-account-connectors), que admiten escenarios de comunicación de negocio a negocio (B2B).
- [Conectores del Entorno del servicio de integración (ISE)](managed.md#ise-connectors), que son un pequeño grupo de [conectores administrados disponibles solo para ISE](#ise-and-connectors).

## <a name="connection-configuration"></a>Configuración de la conexión

La mayoría de los conectores requieren que primero cree una *conexión* al servicio o sistema de destino antes de poder usar sus activadores o acciones en la aplicación lógica. Para crear una conexión, debe autenticar su identidad con las credenciales de cuenta y, a veces, usar otra información de conexión. Por ejemplo, para que el flujo de trabajo obtenga acceso a la cuenta de correo electrónico de Office 365 Outlook y trabaje con ella en una aplicación lógica, debe autorizar una conexión a esa cuenta.

En el caso de los conectores que usan la autenticación OAuth de Azure Active Directory (Azure AD), como Office 365, Salesforce o GitHub debe iniciar sesión en el servicio donde el token de acceso se [cifra](../security/fundamentals/encryption-overview.md) y se almacena de forma segura en un almacén de secretos de Azure. Otros conectores (como FTP y SQL) requieren una conexión con detalles de configuración como la dirección del servidor, el nombre de usuario y la contraseña. Estos detalles sobre la configuración de la conexión también se [cifran y se almacenan de forma segura en Azure](../security/fundamentals/encryption-overview.md).

Las conexiones establecidas pueden obtener acceso al servicio o sistema de destino, siempre que ese servicio o sistema lo permita. En el caso de los servicios que usan conexiones de Azure AD OAuth, como Office 365 y Dynamics, Logic Apps actualiza los tokens de acceso de forma indefinida. Es posible que otros servicios tengan límites con respecto a cuánto tiempo puede usar Logic Apps un token sin actualizarse. Por lo general, algunas acciones invalidarán todos los tokens de acceso como, por ejemplo, el cambio de la contraseña.

Aunque cree conexiones desde un flujo de trabajo, las conexiones son recursos de Azure independientes que cuentan con sus propias definiciones de recursos. Para revisar estas definiciones de recursos de conexión, puede [descargar la aplicación lógica desde Azure en Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md). Este método es la forma más sencilla de crear una plantilla válida de aplicación lógica con parámetros que esté prácticamente lista para la implementación.

> [!TIP]
> Si su organización no le permite acceder a recursos específicos a través de los conectores de Logic Apps, puede [bloquear la capacidad de crear dichas conexiones](../logic-apps/block-connections-connectors.md) mediante [Azure Policy](../governance/policy/overview.md).

## <a name="recurrence-behavior"></a>Comportamiento de periodicidad

Los desencadenadores incorporados recurrentes, como el [Desencadenador de recurrencia](../connectors/connectors-native-recurrence.md), se ejecutan de forma nativa en Azure Logic Apps y son diferentes de los desencadenadores periódicos basados en conexiones, como el desencadenador del conector de Office 365 Outlook, donde primero debe crear una conexión.

Sin embargo, para ambos tipos de desencadenadores, si una periodicidad no proporciona una fecha y hora de inicio específicas, la primera periodicidad se ejecuta inmediatamente al guardar o implementar la aplicación lógica, independientemente de la configuración de periodicidad del desencadenador. Para evitar este comportamiento, proporcione una fecha y hora de inicio para cuando quiera que se ejecute la primera periodicidad.

### <a name="recurrence-for-built-in-triggers"></a>Periodicidad de los desencadenadores integrados

Los desencadenadores periódicos integrados siguen la programación que establezca, incluida cualquier zona horaria especificada. Sin embargo, si una periodicidad no especifica ninguna otra opción de programación avanzada, como horas específicas para ejecutar futuras repeticiones, esas repeticiones se basan en la última ejecución del desencadenador. Como resultado, las horas de inicio de estas periodicidades pueden cambiar debido a factores como la latencia durante las llamadas de almacenamiento. Para obtener ayuda para solucionar problemas, consulte la sección [Problemas de periodicidad](#recurrence-issues).

### <a name="recurrence-for-connection-based-triggers"></a>Periodicidad de los desencadenadores basados en conexión

En cuanto a los desencadenadores periódicos basados en conexiones, como Office 365 Outlook, la programación no es el único controlador que administra la ejecución. Asimismo, la zona horaria solo determina la hora de inicio inicial. Las ejecuciones posteriores dependen de la programación de periodicidad, de la última ejecución del desencadenador, y de otros factores que pueden provocar que haya un desfase o un comportamiento inesperado en los tiempos de ejecución. Entre ellas se incluyen las siguientes:

- Si el desencadenador tiene acceso a un servidor que tiene más datos, que el desencadenador intenta capturar inmediatamente.
- Los errores o reintentos en que incurre el desencadenador.
- La latencia durante las llamadas de almacenamiento.
- No mantener la programación especificada cuando se inicia y finaliza el horario de verano (DST).
- Otros factores que pueden afectar al siguiente tiempo de ejecución.

Para obtener ayuda para solucionar problemas, consulte la sección [Problemas de periodicidad](#recurrence-issues). 

### <a name="recurrence-issues"></a>Problemas de periodicidad

Para asegurarse de que el flujo de trabajo se ejecuta a la hora de inicio especificada y no pierde una periodicidad, especialmente cuando la frecuencia se especifica en días o unidades superiores, pruebe con estas soluciones.

Cuando se aplica el horario de verano, ajuste manualmente la periodicidad para que el flujo de trabajo siga ejecutándose en el momento esperado. De lo contrario, la hora de inicio se desplazará una hora hacia delante cuando se inicie el DST y una hora hacia atrás cuando finalice el DST. Para obtener más información y ejemplos, consulte [Periodicidad de horario de verano y hora estándar](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#daylight-saving-standard-time).

Si usa un desencadenador de **periodicidad**, especifique una zona horaria y una fecha y hora de inicio. Asimismo, configure las horas específicas en las que se ejecutarán las repeticiones posteriores mediante las propiedades denominadas **A estas horas** y **En estos minutos**, que solo están disponibles para las frecuencias **Día** y **Semana**. Sin embargo, es posible que algunas ventanas de tiempo sigan provocando problemas cuando se cambia la hora. 

Considere la posibilidad de usar un desencadenador de [**Ventana deslizante**](../connectors/connectors-native-sliding-window.md) en lugar de un desencadenador de **periodicidad** para evitar periodicidades perdidas.

## <a name="custom-apis-and-connectors"></a>Conectores y API personalizadas

Para llamar a las API que ejecutan código personalizado o que no están disponibles como conectores, puede extender la plataforma de Logic Apps mediante la [creación de aplicaciones de API personalizadas](../logic-apps/logic-apps-create-api-app.md). 

También puede [crear conectores personalizados](../logic-apps/custom-connector-overview.md) para cualquier API REST o SOAP, para que esas API estén disponibles para cualquier aplicación lógica en su suscripción de Azure. 

Para que las aplicaciones de API o los conectores personalizados sean públicos y cualquier persona pueda usarlos en Azure, [envíe los conectores para que Microsoft los certifique](/connectors/custom-connectors/submit-certification).

## <a name="ise-and-connectors"></a>ISE y conectores

En el caso de los flujos de trabajo que necesitan acceso directo a los recursos de una red virtual de Azure, puede crear un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) dedicado en el que puede compilar, implementar y ejecutar los flujos de trabajo en recursos dedicados. Para más información sobre cómo crear ISE, vea [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

Los conectores personalizados creados dentro de un ISE no funcionan con la puerta de enlace de datos local. Pero estos conectores pueden acceder directamente a orígenes de datos locales que están conectados a una red virtual de Azure en la que se hospeda el ISE. Por tanto, es muy probable que las aplicaciones lógicas en un ISE no necesiten la puerta de enlace de datos cuando se comuniquen con esos recursos. Si tiene conectores personalizados que creó fuera de un ISE que requieran una puerta de enlace de datos local, las aplicaciones lógicas de un ISE pueden usar esos conectores.

En el Diseñador de Logic Apps, al examinar los conectores que quiere usar para las aplicaciones lógicas de un ISE, aparece una etiqueta **CORE** en desencadenadores y acciones integrados, mientras que la etiqueta **ISE** aparece en algunos conectores.

:::row:::
    :::column:::
        ![Conector de ejemplo CORE](./media/apis-list/example-core-connector.png)
        \
        \
        **CORE**
        \
        \
        Los desencadenadores y acciones integrados con esta etiqueta se ejecutan en el mismo ISE que las aplicaciones lógicas.
    :::column-end:::
    :::column:::
        ![Ejemplo de conector de ISE](./media/apis-list/example-ise-connector.png)
        \
        \
        **ISE**
        \
        \
        Los conectores administrados con esta etiqueta se ejecutan en el mismo ISE que las aplicaciones lógicas. Si tiene un sistema local que está conectado a una red virtual de Azure, un ISE permite a las aplicaciones lógicas obtener acceso directo a dicho sistema sin la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-connection.md). En su lugar, puede usar el conector de **ISE** si está disponible, una acción HTTP o un [conector personalizado](connectors-overview.md#custom-apis-and-connectors). En el caso de los sistemas locales que no tienen conectores de **ISE**, use la puerta de enlace de datos local. Para revisar los conectores de ISE disponibles, consulte[conectores de ISE](#ise-and-connectors).
    :::column-end:::
    :::column:::
        ![Ejemplo de conector multiinquilino](./media/apis-list/example-multi-tenant-connector.png)
        \
        \
        Sin etiqueta     \
        \
        Todos los demás conectores que no tienen la etiqueta **CORE** o **ISE**, que puede seguir usando, se ejecutan en el servicio de Logic Apps global multiinquilino.
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="known-issues"></a>Problemas conocidos

Los siguientes son problemas conocidos de los conectores de Logic Apps.

#### <a name="error-badgateway-client-request-id-guid"></a>Error: BadGateway. Id. de solicitud de cliente: "{GUID}"

Este error se produce cuando se actualizan las etiquetas de una aplicación lógica en la que una o varias conexiones no admiten la autenticación OAuth de Azure Active Directory (Azure AD), como SFTP ad SQL, con lo que se interrumpen esas conexiones. Para evitar este comportamiento, evite actualizar esas etiquetas.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de API personalizadas que se pueden llamar desde Logic Apps](/logic-apps/logic-apps-create-api-app)
