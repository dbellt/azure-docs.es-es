---
title: Introducción a la versión preliminar de Azure Logic Apps
description: La versión preliminar de Azure Logic Apps es una solución en la nube para la creación de flujos de trabajo automatizados de un único inquilino con y sin estado que integran aplicaciones, datos, servicios y sistemas con un código mínimo para escenarios de nivel empresarial.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, az-logic-apps-dev
ms.topic: conceptual
ms.date: 03/24/2021
ms.openlocfilehash: 27889e8309c0efaf1e2869fc39d099f38f64f7c4
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764826"
---
# <a name="overview-azure-logic-apps-preview"></a>Introducción: Versión preliminar de Azure Logic Apps

> [!IMPORTANT]
> Esta funcionalidad se encuentra en versión preliminar pública, se ofrece sin contrato de nivel de servicio y no se recomienda usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Con la versión preliminar de Azure Logic Apps, puede crear soluciones de automatización e integración entre aplicaciones, datos, servicios en la nube y sistemas mediante la creación y ejecución de aplicaciones lógicas de un único inquilino con el nuevo tipo de recurso **Aplicación lógica (versión preliminar)** . Con este tipo de aplicación lógica de un único inquilino, puede crear varios [flujos de trabajo *con estado* y *sin estado*](#stateful-stateless) con la tecnología del entorno de ejecución rediseñado de la versión preliminar de Azure Logic Apps, que proporciona portabilidad, mejor rendimiento y flexibilidad para su implementación y ejecución en diversos entornos de hospedaje, no solo Azure, sino también contenedores de Docker.

¿Cómo es posible? El runtime rediseñado usa el [modelo de extensibilidad de Azure Functions](../azure-functions/functions-bindings-register.md) y se hospeda como una extensión en el runtime de Azure Functions. Esta arquitectura significa que el tipo de aplicación lógica de un único inquilino se puede ejecutar en cualquier ubicación en donde se ejecute Azure Functions. Puede hospedar el runtime rediseñado prácticamente en cualquier topología de red y elegir cualquier tamaño de proceso disponible para controlar la carga de trabajo necesaria que requieren los flujos de trabajo. Para obtener más información, vea [Introducción a Azure Functions](../azure-functions/functions-overview.md) y [Conceptos básicos sobre los enlaces y desencadenadores de Azure Functions](../azure-functions/functions-triggers-bindings.md).

Puede crear el recurso de **Logic App (versión preliminar)** si [inicia en Azure Portal](create-stateful-stateless-workflows-azure-portal.md) o [crea un proyecto en Visual Studio Code con la extensión Azure Logic Apps (versión preliminar)](create-stateful-stateless-workflows-visual-studio-code.md). Además, en Visual Studio Code, puede compilar *y ejecutar localmente* los flujos de trabajo en el entorno de desarrollo. Tanto si usa el portal como Visual Studio Code, puede implementar y ejecutar el tipo de aplicación lógica de un único inquilino en los mismos tipos de entornos de hospedaje.

En esta introducción se tratan las áreas siguientes:

* [Diferencias entre la versión preliminar de Azure Logic Apps, el entorno multiinquilino de Azure Logic Apps y el entorno del servicio de integración](#preview-differences).

* [Diferencias entre los flujos de trabajo con y sin estado](#stateful-stateless), incluidas las diferencias de comportamiento entre [flujos de trabajo anidados con y sin estado](#nested-behavior).

* [Capacidades de esta versión preliminar pública](#public-preview-contents).

* [Funcionamiento del modelo de precios](#pricing-model).

* [Capacidades modificadas, limitadas, no disponibles o no admitidas](#limited-unavailable-unsupported).

* [Límites de la versión preliminar de Azure Logic Apps](#limits).

Para más información, consulte estos otros temas:

* [Azure Logic Apps en ejecución en cualquier ubicación: el runtime en profundidad](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-runtime-deep-dive/ba-p/1835564)

* [Problemas conocidos de la versión preliminar pública de Logic Apps (GitHub)](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md)

<a name="preview-differences"></a>

## <a name="how-does-azure-logic-apps-preview-differ"></a>Diferencias de la versión preliminar de Azure Logic Apps

El runtime de la versión preliminar de Azure Logic Apps usa la extensibilidad de [Azure Functions](../azure-functions/functions-overview.md) y se hospeda como una extensión en el runtime de Azure Functions. Esta arquitectura significa que el tipo de aplicación lógica de un único inquilino se puede ejecutar en cualquier ubicación en donde se ejecute Azure Functions. Puede hospedar el runtime de la versión preliminar de Azure Logic Apps prácticamente en cualquier topología de red que quiera y elegir cualquier tamaño de proceso disponible para controlar la carga de trabajo que necesita el flujo de trabajo. Para obtener más información sobre la extensibilidad de Azure Functions, vea [SDK de WebJobs: Creación de enlaces de entrada y salida personalizados](https://github.com/Azure/azure-webjobs-sdk/wiki/Creating-custom-input-and-output-bindings).

Con este nuevo enfoque, tanto el runtime de la versión preliminar de Azure Logic Apps como los flujos de trabajo forman parte de la aplicación que se puede empaquetar. Esta capacidad permite implementar y ejecutar los flujos de trabajo con solo copiar artefactos en el entorno de hospedaje e iniciar la aplicación. Este enfoque además proporciona una experiencia más normalizada para compilar canalizaciones de implementación en torno a los proyectos de flujo de trabajo para ejecutar las pruebas y validaciones necesarias antes de implementar cambios en entornos de producción. Para obtener más información, vea [Azure Logic Apps en ejecución en cualquier ubicación: el runtime en profundidad](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-runtime-deep-dive/ba-p/1835564).

En la tabla siguiente se resumen brevemente las diferencias en la manera en que los flujos de trabajo comparten recursos, en función del entorno en el que se ejecutan. Para conocer las diferencias en cuanto a los límites, vea [Límites de la versión preliminar de Azure Logic Apps](#limits).

| Entorno | Uso compartido y consumo de recursos |
|-------------|----------------------------------|
| Azure Logic Apps (multiinquilino) | Los flujos de trabajo *de clientes en varios inquilinos* comparten el mismo procesamiento (proceso), almacenamiento, red, etc. |
| Azure Logic Apps (versión preliminar, un único inquilino) | Los flujos de trabajo *de la misma aplicación lógica y un único inquilino* comparten el mismo procesamiento (proceso), almacenamiento, red, etc. |
| Entorno del servicio de integración (no disponible en la versión preliminar) | Los flujos de trabajo del *mismo entorno* comparten el mismo procesamiento (proceso), almacenamiento, red, etc. |
|||

Mientras, puede seguir creando el tipo de aplicación lógica de varios inquilinos en Azure Portal y en Visual Studio Code mediante la extensión de Azure Logic Apps de varios inquilinos. Aunque las experiencias de desarrollo difieren entre los tipos de aplicaciones lógicas de un único inquilino y varios inquilinos, la suscripción de Azure puede incluir ambos tipos. Puede ver todas las aplicaciones lógicas implementadas en la suscripción de Azure, y acceder a ellas, pero se organizan en sus propias categorías y secciones.

<a name="stateful-stateless"></a>

## <a name="stateful-and-stateless-workflows"></a>Flujos de trabajo con y sin estado

Con el tipo de aplicación lógica de un único inquilino, puede crear estos tipos de flujo de trabajo dentro de la misma aplicación lógica:

* *Con estado*

  Cree flujos de trabajo con estado cuando necesite mantener o revisar datos de eventos anteriores, o hacer referencia a ellos. Estos flujos de trabajo guardan las entradas y las salidas de cada acción y sus estados en el almacenamiento externo, lo que facilita la revisión y ejecución de los detalles y el historial al término de cada ejecución. Los flujos de trabajo con estado proporcionan una alta resistencia en caso de interrupciones. Una vez restaurados los servicios y sistemas, puede reconstruir las ejecuciones interrumpidas a partir del estado guardado y volver a ejecutar los flujos de trabajo hasta su finalización. Los flujos de trabajo con estado pueden continuar ejecutándose durante un plazo máximo de un año.

* *Sin estado*

  Cree flujos de trabajo sin estado cuando no necesite guardar ni revisar datos de eventos anteriores, ni hacer referencia a ellos, en un almacenamiento externo para su revisión posterior. Estos flujos de trabajo guardan las entradas y las salidas de cada acción y sus estados *solo en la memoria*, en lugar de transferir estos datos al almacenamiento externo. Como consecuencia, los flujos de trabajo sin estado tienen ejecuciones más cortas que normalmente no duran más de 5 minutos, un rendimiento más rápido con menores tiempos de respuesta, mayor rendimiento y costos de ejecución menores, ya que los detalles y el historial de ejecución no se conservan en el almacenamiento externo. Pero si se producen interrupciones, las ejecuciones interrumpidas no se restauran automáticamente, por lo que el autor de la llamada tiene que volver a enviarlas manualmente. Estos flujos de trabajo solo se pueden ejecutar de manera sincrónica.

  Para facilitar la depuración, puede habilitar el historial de ejecución de un flujo de trabajo sin estado, lo que tiene algún impacto en el rendimiento, y luego deshabilitar el historial de ejecución cuando haya terminado. Para obtener más información, vea [Creación de flujos de trabajo con y sin estado en Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#enable-run-history-stateless) o [Creación de flujos de trabajo con y sin estado en Azure Portal](create-stateful-stateless-workflows-visual-studio-code.md#enable-run-history-stateless).

  > [!NOTE]
  > Actualmente, los flujos de trabajo sin estado solo admiten *acciones* de [conectores administrados](../connectors/managed.md) que se implementan en Azure, y no desencadenadores. Para iniciar el flujo de trabajo, seleccione el [desencadenador integrado Solicitud, Event Hubs o Service Bus](../connectors/built-in.md). Estos desencadenadores se ejecutan de forma nativa en el runtime de la versión preliminar de Azure Logic Apps. Para obtener más información sobre los desencadenadores, las acciones y los conectores limitados, no disponibles o no admitidos, vea [Capacidades modificadas, limitadas, no disponibles o no admitidas](#limited-unavailable-unsupported).

<a name="nested-behavior"></a>

### <a name="nested-behavior-differences-between-stateful-and-stateless-workflows"></a>Diferencias de comportamiento anidado entre flujos de trabajo con y sin estado

Puede [hacer que se pueda llamar a un flujo de trabajo](../logic-apps/logic-apps-http-endpoint.md) desde otros flujos de trabajo que existan en el mismo recurso de **Logic App (versión preliminar)** mediante el [desencadenador Solicitud](../connectors/connectors-native-reqres.md), el [desencadenador Webhook HTTP](../connectors/connectors-native-webhook.md) o los desencadenadores de conectores administrados que tienen el [tipo ApiConnectionWehook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-trigger) y pueden recibir solicitudes HTTPS.

Estos son los patrones de comportamiento que los flujos de trabajo anidados pueden seguir después de que un flujo de trabajo primario llame a uno secundario:

* Patrón de sondeo asincrónico

  El flujo de trabajo primario no espera una respuesta a su llamada inicial, pero comprueba continuamente el historial de ejecución del flujo de trabajo secundario hasta que este termina de ejecutarse. De manera predeterminada, los flujos de trabajo con estado siguen este patrón, que es ideal para flujos de trabajo secundarios de ejecución prolongada que podrían superar los [límites de tiempo de espera de las solicitudes](../logic-apps/logic-apps-limits-and-config.md).

* Patrón sincrónico ("desencadenar y olvidar")

  El flujo de trabajo secundario confirma la llamada devolviendo inmediatamente una respuesta `202 ACCEPTED`, y el flujo de trabajo primario continúa con la siguiente acción sin esperar los resultados del secundario. En su lugar, el primario recibe los resultados cuando el secundario termina de ejecutarse. Los flujos de trabajo con estado secundarios que no incluyen una acción Respuesta siempre siguen el patrón sincrónico. En el caso de los flujos de trabajo con estado secundarios, el historial de ejecución está disponible para su revisión.

  Para habilitar este comportamiento, en la definición de JSON del flujo de trabajo, establezca la propiedad `operationOptions` en `DisableAsyncPattern`. Para obtener más información, consulte [Tipos de desencadenador y de acción: Opciones de operación](../logic-apps/logic-apps-workflow-actions-triggers.md#operation-options).

* Desencadenar y esperar

  Para un flujo de trabajo sin estado secundario, el primario espera una respuesta que devuelve los resultados del secundario. Este patrón funciona de forma similar al uso del [desencadenador o acción HTTP](../connectors/connectors-native-http.md) integrados para llamar a un flujo de trabajo secundario. Los flujos de trabajo sin estado secundarios que no incluyen una acción Respuesta devuelven inmediatamente una respuesta `202 ACCEPTED`, pero el primario espera a que el secundario finalice antes de continuar con la siguiente acción. Estos comportamientos solo se aplican a los flujos de trabajo sin estado secundarios.

En esta tabla se especifica el comportamiento del flujo de trabajo secundario en función de si los flujos de trabajo primario y secundario son con estado, sin estado o tipos de flujo de trabajo mixtos:

| Flujo de trabajo primario | Flujo de trabajo secundario | Comportamiento del secundario |
|-----------------|----------------|----------------|
| Con estado | Con estado | Asincrónico o sincrónico con la configuración `"operationOptions": "DisableAsyncPattern"` |
| Con estado | Sin estado | Desencadenar y esperar |
| Sin estado | Con estado | Sincrónico |
| Sin estado | Sin estado | Desencadenar y esperar |
||||

<a name="public-preview-contents"></a>

## <a name="capabilities"></a>Funciones

La versión preliminar de Azure Logic Apps incluye muchas capacidades actuales y adicionales, por ejemplo:

* Cree aplicaciones lógicas y sus flujos de trabajo desde [más de 400 conectores](/connectors/connector-reference/connector-reference-logicapps-connectors) para aplicaciones y servicios de Software como servicio (SaaS) y Plataforma como servicio (PaaS), además de conectores para sistemas locales.

  * Algunos conectores administrados están ahora disponibles como versiones integradas, que se ejecutan de forma similar a los desencadenadores y acciones integrados, como el desencadenador de solicitudes y la acción HTTP, que se ejecutan de forma nativa en el tiempo de ejecución de la versión preliminar de Azure Logic Apps. Por ejemplo, estos nuevos conectores integrados incluyen Azure Service Bus, Azure Event Hubs, SQL Server y MQ.

    > [!NOTE]
    > En el caso del conector integrado de SQL Server, solo la acción **Ejecutar consulta** puede conectarse directamente a redes virtuales de Azure sin necesitar la [puerta de enlace de datos local](logic-apps-gateway-connection.md).

  * Cree sus propios conectores integrados para cualquier servicio que necesite mediante el [marco de extensibilidad de la versión preliminar](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-built-in-connector/ba-p/1921272). De forma similar a los conectores integrados, como Azure Service Bus y SQL Server, pero a diferencia de los [conectores personalizados](../connectors/apis-list.md#custom-apis-and-connectors), que no se admiten actualmente para la versión preliminar, estos conectores proporcionan mayor rendimiento, baja latencia, conectividad local y se ejecutan de forma nativa en el mismo proceso que el entorno de ejecución de la versión preliminar.

    La funcionalidad de creación solo está disponible actualmente en Visual Studio Code, pero no está habilitada de manera predeterminada. Para crear estos conectores, [cambie el proyecto de la extensión basada en agrupación (Node.js) a basado en paquetes NuGet (.NET)](create-stateful-stateless-workflows-visual-studio-code.md#enable-built-in-connector-authoring). Para obtener más información, vea [Azure Logic Apps en ejecución en cualquier ubicación: extensibilidad de los conectores integrados](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-built-in-connector/ba-p/1921272).

  * Puede usar las acciones B2B en operaciones de Liquid y XML sin ninguna cuenta de integración. Para usar estas acciones, debe tener asignaciones de Liquid, asignaciones XML o esquemas XML que pueda cargar por medio de las acciones respectivas en Azure Portal o agregar a la carpeta **Artefactos** del proyecto de Visual Studio Code mediante las carpetas **Asignaciones** y **Esquemas** respectivas.

  * Cree e implemente aplicaciones lógicas que se puedan ejecutar en cualquier lugar, ya que el servicio Azure Logic Apps genera cadenas de conexión de Firma de acceso compartido (SAS) que estas aplicaciones lógicas pueden usar para enviar solicitudes al punto de conexión de runtime de conexión a la nube. El servicio Logic Apps guarda estas cadenas de conexión con otras opciones de configuración de la aplicación para que pueda almacenar fácilmente estos valores en Azure Key Vault al realizar la implementación en Azure.

    > [!NOTE]
    > De manera predeterminada, un recurso de **Logic Apps (versión preliminar)** tiene su [identidad administrada asignada por el sistema](../logic-apps/create-managed-service-identity.md) habilitada automáticamente para autenticar conexiones en runtime. Esta identidad se diferencia de las credenciales de autenticación o de la cadena de conexión que se usan al crear una conexión. Si deshabilita esta identidad, las conexiones no funcionarán en runtime. Para ver este valor, en el menú de la aplicación lógica, en **Configuración**, seleccione **Identidad**.

* Cree aplicaciones lógicas con flujos de trabajo sin estado que se ejecuten solo en memoria de modo que finalicen con mayor rapidez, respondan más rápido, tengan un mejor rendimiento e incurran en menos costos de ejecución, ya que los historiales de ejecución y los datos entre acciones no se conservan en el almacenamiento externo. De manera opcional, puede habilitar el historial de ejecución para facilitar la depuración. Para obtener más información, consulte [Aplicaciones lógicas con estado y sin estado](#stateful-stateless).

* Ejecute, pruebe y depure las aplicaciones lógicas y sus flujos de trabajo localmente en el entorno de desarrollo de Visual Studio Code.

  Antes de ejecutar y probar la aplicación lógica, puede facilitar la depuración si agrega y usa puntos de interrupción dentro del archivo **workflow.json** de un flujo de trabajo. Pero los puntos de interrupción solo se admiten en las acciones en este momento, no en los desencadenadores. Para obtener más información, vea [Creación de flujos de trabajo con y sin estado en Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#manage-breakpoints).

* Publique o implemente directamente las aplicaciones lógicas y sus flujos de trabajo desde Visual Studio Code en varios entornos de hospedaje, como Azure y [contenedores de Docker](/dotnet/core/docker/introduction).

* Habilite las capacidades de registro de diagnóstico y seguimiento de la aplicación lógica mediante [Application Insights](../azure-monitor/app/app-insights-overview.md) cuando lo admitan la configuración de la aplicación lógica y la suscripción de Azure.

* Con el [plan prémium de Azure Functions](../azure-functions/functions-premium-plan.md) puede acceder a funcionalidades de red, como la conexión e integración de forma privada con redes virtuales de Azure, de manera similar a Azure Functions al crear e implementar las aplicaciones lógicas. Para obtener más información, consulte estos temas:

  * [Opciones de redes de Azure Functions](../azure-functions/functions-networking-options.md)

  * [Azure Logic Apps en ejecución en cualquier ubicación: posibilidades de conexión de red con Azure Logic Apps (versión preliminar)](https://techcommunity.microsoft.com/t5/integrations-on-azure/logic-apps-anywhere-networking-possibilities-with-logic-app/ba-p/2105047)

* Regenere las claves de acceso para las conexiones administradas usadas por los flujos de trabajo individuales de un recurso de **aplicación lógica (versión preliminar)** de un único inquilino. Para esta tarea, [siga los mismos pasos que para el recurso de **Logic Apps** de varios inquilinos, pero en el nivel de flujo de trabajo individual](logic-apps-securing-a-logic-app.md#regenerate-access-keys), no en el nivel de recurso de aplicación lógica.

* Agregue ramas paralelas en el diseñador para inquilino único con los mismos pasos que en el diseñador para varios inquilinos.

Para más información, consulte [Funcionalidades modificadas, limitadas, no disponibles e incompatibles](#limited-unavailable-unsupported) y la [página de problemas conocidos de la versión preliminar pública de Logic Apps en GitHub](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md).

<a name="pricing-model"></a>

## <a name="pricing-model"></a>Modelo de precios

Al crear el tipo de aplicación lógica de un único inquilino en Azure Portal o realizar la implementación desde Visual Studio Code, debe elegir un plan de hospedaje, ya sea [App Service o Premium](../azure-functions/functions-scale.md), para que lo use la aplicación lógica. Este plan determina el modelo de precios que se aplica a la hora de ejecutar la aplicación lógica. Si selecciona el plan de App Service, también debe elegir un [plan de tarifa](../app-service/overview-hosting-plans.md).

Los flujos de trabajo *con estado* usan [almacenamiento externo](../azure-functions/storage-considerations.md#storage-account-requirements), por lo que se aplican los [precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/) a las transacciones de almacenamiento que realiza el runtime de la versión preliminar de Azure Logic Apps. Por ejemplo, las colas se utilizan para la programación, mientras que las tablas y blobs se utilizan para almacenar los estados de los flujos de trabajo.

> [!NOTE]
> Durante la versión preliminar pública, la ejecución de aplicaciones lógicas en App Service no incurre en cargos *adicionales* al plan seleccionado.

Para más información sobre los modelos de precios que se aplican al tipo de recurso de un único inquilino, consulte estos temas:

* [Escalado y hospedaje de Azure Functions](../azure-functions/functions-scale.md)
* [Escalado de una aplicación en Azure App Service](../app-service/manage-scale-up.md)
* [Detalles de precios de Azure Functions](https://azure.microsoft.com/pricing/details/functions/)
* [Detalles de precios de App Service](https://azure.microsoft.com/pricing/details/app-service/)
* [Detalles de precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/)

<a name="limited-unavailable-unsupported"></a>

## <a name="changed-limited-unavailable-or-unsupported-capabilities"></a>Capacidades modificadas, limitadas, no disponibles o no admitidas

En la versión preliminar de Azure Logic Apps, estas capacidades han cambiado, o bien están limitadas, no están disponibles o no se admiten:

* **Compatibilidad con SO**: Actualmente, el diseñador de Visual Studio Code no funciona en el sistema operativo Linux, pero puede implementar aplicaciones lógicas que usan el entorno en tiempo de ejecución de la versión preliminar de Logic Apps en máquinas virtuales basadas en Linux. Por ahora, puede compilar las aplicaciones lógicas en Visual Studio Code en Windows o macOS y, luego, implementarlas en una máquina virtual basada en Linux.

* **Desencadenadores y acciones**: los desencadenadores y las acciones integrados se ejecutan de forma nativa en el runtime de la versión preliminar de Azure Logic Apps, mientras que los conectores administrados se implementan en Azure. algunos desencadenadores integrados no están disponibles, como Ventana deslizante y Lote.

  Para iniciar el flujo de trabajo, use el [desencadenador integrado Solicitud, HTTP, HTTP Webhook, Event Hubs o Service Bus](../connectors/apis-list.md). En el diseñador, las acciones y los desencadenadores integrados aparecen en la pestaña **Integrados**, mientras que las acciones y los desencadenadores de los conectores administrados aparecen en la pestaña **Azure**.

  > [!NOTE]
  > Para ejecutar localmente en Visual Studio Code, los desencadenadores y las acciones basados en webhook requieren configuración adicional. Para obtener más información, vea [Creación de flujos de trabajo con y sin estado en Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#webhook-setup).

  * En el caso de los *flujos de trabajo sin estado*, la pestaña **Azure** no aparece cuando se selecciona un desencadenador, ya que solo se pueden seleccionar [acciones *de conector administrado*, no desencadenadores](../connectors/managed.md). Aunque puede habilitar conectores administrados implementados en Azure para flujos de trabajo sin estado, el diseñador no muestra ningún desencadenador de conector administrado para que pueda agregarlo.

  * En el caso de los *flujos de trabajo con estado*, excepto los desencadenadores y las acciones que se muestran como no disponibles a continuación, se pueden usar [tanto desencadenadores como acciones de conector administrado](../connectors/managed.md).

  * Estos desencadenadores y acciones han cambiado o están limitados, no se admiten o no están disponibles:

    * Los [*desencadenadores* de la puerta de enlace de datos local](../connectors/managed.md#on-premises-connectors) no están disponibles, pero las acciones *sí*.

    * La acción integrada [Azure Functions: Elegir una función de Azure](logic-apps-azure-functions.md) es ahora **Operaciones de Azure Functions: Llamar a una función de Azure**. Esta acción solo funciona con las funciones que se crean a partir de la plantilla **Desencadenador HTTP**.

      En Azure Portal, puede seleccionar una función de desencadenador HTTP a la que tenga acceso mediante la creación de una conexión por medio de la experiencia del usuario. Si inspecciona la definición JSON de la acción de la función en la vista de código o el archivo **workflow.json**, la acción se refiere a la función mediante una referencia `connectionName`. Esta versión abstrae la información de la función como una conexión que puede encontrar en el archivo **connections.json** del proyecto, que está disponible después de crear una conexión.

      > [!NOTE]
      > En la versión de un único inquilino, la acción de la función solo admite la autenticación de cadena de consulta. La versión preliminar de Azure Logic Apps obtiene la clave predeterminada de la función al realizar la conexión, la almacena en la configuración de la aplicación y la usa para la autenticación cuando se llama a la función.
      >
      > Al igual que con la versión de varios inquilinos, si renueva esta clave, por ejemplo, mediante la experiencia de Azure Functions del portal, la acción de la función ya no funciona debido a la clave no válida. Para corregir este problema, debe volver a crear la conexión a la función a la que quiere llamar o actualizar la configuración de la aplicación con la nueva clave.

    * La acción integrada [Código en línea: Ejecutar código de JavaScript](logic-apps-add-run-inline-code.md) es ahora **Operaciones de código en línea: Ejecutar JavaScript en línea**.

      * Las acciones de Operaciones de código en línea ya no requieren ninguna cuenta de integración.

      * Para macOS o Linux, **Operaciones de código en línea** ya está disponible si se emplea la extensión Azure Logic Apps (versión preliminar) en Visual Studio Code.

      * Si realiza cambios en una acción de **Operaciones de código en línea**, ya no será necesario reiniciar la aplicación lógica.

      * Las acciones de **Operaciones de código** en línea tienen [límites actualizados](logic-apps-overview-preview.md#inline-code-limits).

    * Algunos [desencadenadores y acciones de B2B integrados de las cuentas de integración](../connectors/managed.md#integration-account-connectors) no están disponibles, por ejemplo, las acciones de codificación y descodificación **Archivo plano**.

    * La acción integrada [Azure Logic Apps: Elegir un flujo de trabajo de aplicación lógica](logic-apps-http-endpoint.md) es ahora **Operaciones de flujo de trabajo: Invocar un flujo de trabajo en esta aplicación de flujo de trabajo**.

* Los [conectores personalizados](../connectors/apis-list.md#custom-apis-and-connectors) no se admiten actualmente para la versión preliminar.

* **Disponibilidad del plan de hospedaje**: tanto si crea el tipo de recurso de **aplicación lógica (versión preliminar)** de un único inquilino en Azure Portal como si se implementa desde Visual Studio Code, solo puede usar el plan de hospedaje de App Service o Premium de Azure. Los planes de hospedaje de consumo no son compatibles ni están disponibles para implementar este tipo de recurso. Puede implementar desde Visual Studio Code en un contenedor de Docker, pero no en un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md).

* **Depuración de puntos de interrupción en Visual Studio Code**: aunque puede agregar y usar puntos de interrupción dentro del archivo **workflow.json** de un flujo de trabajo, los puntos de interrupción solo se admiten en las acciones en este momento, no en los desencadenadores. Para obtener más información, vea [Creación de flujos de trabajo con y sin estado en Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#manage-breakpoints).

* **Control de zoom**: el control de zoom no está disponible actualmente en el diseñador.

* **Historial de desencadenadores e historial de ejecución**: para el tipo de recurso **Logic App (versión preliminar)** , el historial de desencadenadores y el historial de ejecución en Azure Portal aparecen en el nivel del flujo de trabajo, no en el nivel de la aplicación lógica. Para encontrar estos datos históricos, siga estos pasos:

   * Para ver el historial de ejecución, abra el flujo de trabajo en la aplicación lógica. En el menú del flujo de trabajo, en **Programador**, seleccione **Pantalla**.

   * Para ver el historial de desencadenadores, abra el flujo de trabajo en la aplicación lógica. En el menú del flujo de trabajo, en **Programador**, seleccione **Historiales del desencadenador**.

<a name="firewall-permissions"></a>

## <a name="permit-traffic-in-strict-network-and-firewall-scenarios"></a>Permitir el tráfico en escenarios de red y firewall estrictos

Si el entorno tiene requisitos de red estrictos o firewalls que limitan el tráfico, debe permitir el acceso de las conexiones del desencadenador o la acción en los flujos de trabajo de la aplicación lógica.

Para buscar los nombres de dominio completos (FQDN) de estas conexiones, revise las secciones correspondientes de estos temas:

* [Permisos de firewall para aplicaciones lógicas de inquilino único: Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#firewall-setup)
* [Permisos de firewall para aplicaciones lógicas de inquilino único: Azure Portal](create-stateful-stateless-workflows-azure-portal.md#firewall-setup)

<a name="limits"></a>

## <a name="updated-limits"></a>Límites actualizados

Aunque muchos límites de la versión preliminar de Azure Logic Apps son los mismos que los [límites de Azure Logic Apps multiinquilino](logic-apps-limits-and-config.md), han cambiado en la versión preliminar de Azure Logic Apps.

<a name="http-timeout-limits"></a>

### <a name="http-timeout-limits"></a>Límites de tiempo de espera HTTP

En el caso de una única llamada entrante o saliente, el límite de tiempo de espera es de 230 segundos (3,9 minutos) para estos desencadenadores y acciones:

* Solicitud saliente: desencadenador HTTP, acción HTTP
* Solicitud entrante: desencadenador Solicitud, desencadenador HTTP Webhook, acción HTTP Webhook

En comparación, estos son los límites de tiempo de espera de estos desencadenadores y acciones en otros entornos donde se ejecutan las aplicaciones lógicas y sus flujos de trabajo:

* Azure Logic Apps multiinquilino: 120 segundos (2 minutos)
* Entorno del servicio de integración: 240 segundos (4 minutos)

Para obtener más información, vea [Límites HTTP](logic-apps-limits-and-config.md#http-limits).

<a name="managed-connector-limits"></a>

### <a name="managed-connectors"></a>Conectores administrados

Los conectores administrados se limitan a 50 solicitudes por minuto y conexión. Para solucionar problemas de limitación de conectores, vea [Solución de problemas de limitación (429 - "Demasiadas solicitudes") en Azure Logic Apps](handle-throttling-problems-429-errors.md#connector-throttling).

<a name="inline-code-limits"></a>

### <a name="inline-code-operations-execute-javascript-code"></a>Operaciones de código en línea (Ejecutar código de JavaScript)

En el caso de una sola definición de aplicación lógica, la acción Operaciones de código en línea, [**Ejecutar código de JavaScript**](logic-apps-add-run-inline-code.md) tiene estos límites actualizados:

* El número máximo de caracteres de código aumenta de 1024 a 100 000 caracteres.

* La duración máxima de la ejecución de código aumenta de 5 a 15 segundos.

Para obtener más información, vea [Límites de definición de Logic Apps](logic-apps-limits-and-config.md#definition-limits).

## <a name="next-steps"></a>Pasos siguientes

* [Creación de flujos de trabajo con y sin estado en Azure Portal](create-stateful-stateless-workflows-azure-portal.md)
* [Creación de flujos de trabajo con y sin estado en Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md)
* [Página de problemas conocidos de la versión preliminar pública de Logic Apps en GitHub](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md)

Además, nos gustaría conocer sus experiencias con la versión preliminar de Azure Logic Apps.

* En caso de errores o problemas, [dirija los problemas a GitHub](https://github.com/Azure/logicapps/issues).
* Si tiene preguntas, solicitudes, ideas y comentarios, [use este formulario de comentarios](https://aka.ms/lafeedback).
