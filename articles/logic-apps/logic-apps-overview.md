---
title: Introducción a Azure Logic Apps
description: La plataforma en la nube Logic Apps ayuda a crear y ejecutar flujos de trabajo automatizados para escenarios de integración empresarial con poco o ningún código.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: overview
ms.custom: mvc
ms.date: 04/26/2021
ms.openlocfilehash: 78a1d577be21334a14008b18bb777f3d79c2a5e1
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287201"
---
# <a name="what-is-azure-logic-apps"></a>¿Qué es Azure Logic Apps?

[Logic Apps](https://azure.microsoft.com/services/logic-apps) es una plataforma basada en la nube que le ayuda a crear y ejecutar [flujos de trabajo](#logic-app-concepts) automatizados para integrar aplicaciones, datos, servicios y sistemas. Esta plataforma permite crear soluciones de integración altamente escalables para escenarios intraempresariales y de negocio a negocio (B2B) de forma más fácil y rápida. Logic Apps forma parte de [Azure Integration Services](https://azure.microsoft.com/product-categories/integration/), por lo que proporciona una forma más sencilla de conectar sistemas heredados, modernos y vanguardistas a través de entornos híbridos, locales y en la nube.

En esta lista se describen solo algunas tareas, procesos empresariales y cargas de trabajo de ejemplo que puede automatizar con el servicio Logic Apps:

* Programación y envío de notificaciones por correo electrónico mediante Office 365 cuando se produce un evento específico, por ejemplo, cuando se ha cargado un archivo nuevo.
* Procesamiento y enrutamiento de pedidos entre sistemas locales y servicios en la nube.
* Traslado de archivos cargados de un servidor SFTP o FTP a Azure Storage.
* Supervisión de tuits, análisis de opiniones y creación de alertas o tareas para los elementos que se deben revisar.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Introducing-Azure-Logic-Apps/player]

Para acceder de forma segura y ejecutar operaciones en tiempo real en varios orígenes de datos, elija entre una [galería en constante crecimiento](/connectors/connector-reference/connector-reference-logicapps-connectors) de [conectores administrados por Microsoft](#logic-app-concepts), por ejemplo:

* Servicios de Azure, como Blob Storage y Service Bus
* Servicios de Office, como Outlook, Excel y SharePoint
* Servidores de bases de datos, como SQL y Oracle
* Sistemas empresariales, como SAP e IBM MQ
* Recursos compartidos de archivos, como FTP y SFTP

Para comunicarse con cualquier punto de conexión de servicio, ejecutar su propio código, organizar el flujo de trabajo o manipular los datos, puede usar [desencadenadores y acciones integrados](#logic-app-concepts), que se ejecutan de forma nativa dentro del servicio Logic Apps. Por ejemplo, los desencadenadores integrados incluyen Solicitud, HTTP y Periodicidad. Las acciones integradas incluyen Condición, Bucle For each, Ejecutar código JavaScript y operaciones que llaman a funciones de Azure, aplicaciones web o aplicaciones de API hospedadas en Azure y otros flujos de trabajo de Logic Apps.

Para escenarios de integración B2B, Logic Apps incluye funcionalidades de [BizTalk Server](/biztalk/core/introducing-biztalk-server). Puede crear una [cuenta de integración](logic-apps-enterprise-integration-create-integration-account.md) en la que definir socios comerciales, contratos, esquemas, mapas y otros artefactos B2B. Si vincula esta cuenta a una aplicación lógica, puede crear flujos de trabajo que funcionen con estos artefactos e intercambien mensajes mediante protocolos como AS2, EDIFACT, X12 y RosettaNet.

Para más información sobre las formas en las que los flujos de trabajo pueden acceder y trabajar con aplicaciones, datos, servicios y sistemas, consulte la siguiente documentación:

* [Conectores para Azure Logic Apps](../connectors/apis-list.md)
* [Desencadenadores y acciones integrados para Logic Apps](../connectors/built-in.md)
* [Conectores administrados para Logic Apps](../connectors/managed.md)
* [Soluciones de integración empresarial B2B con Azure Logic Apps y Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md)

<a name="logic-app-concepts"></a>

## <a name="key-terms"></a>Términos clave

* **Flujo de trabajo**: una serie de pasos que definen una tarea o proceso, empezando por un único desencadenador y seguido de una o varias acciones

* **Desencadenador**: el primer paso que inicia cada flujo de trabajo y especifica la condición que se debe cumplir antes de ejecutar cualquier acción en el flujo de trabajo. Por ejemplo, un evento de desencadenador podría recibir un correo electrónico en la bandeja de entrada o detectar un nuevo archivo en una cuenta de almacenamiento.

* **Acción**: cada paso posterior que sigue después del desencadenador y ejecuta alguna operación en un flujo de trabajo

* **Conector administrado**: una API REST administrada por Microsoft que proporciona acceso a una aplicación, datos, servicio o sistema específicos. Para poder usarlos, la mayoría de los conectores administrados requieren que primero cree una conexión desde el flujo de trabajo y autentique su identidad.

  Por ejemplo, puede iniciar un flujo de trabajo con un desencadenador o incluir una acción que trabaje con Azure Blob Storage, Office 365, Salesforce o servidores SFTP. Para más información, consulte [Conectores administrados para Logic Apps](../connectors/managed.md).

* **Desencadenador o acción integrados**: una operación de Logic Apps que se ejecuta de forma nativa que proporciona una manera de controlar la programación o estructura del flujo de trabajo, ejecutar su propio código, administrar o manipular datos o completar otras tareas del flujo de trabajo. La mayoría de las operaciones integradas no están asociadas a ningún servicio o sistema. Muchas tampoco requieren que primero cree una conexión desde el flujo de trabajo y autentique su identidad. Las operaciones integradas también están disponibles para algunos servicios, sistemas y protocolos, como Azure Functions, Azure API Management, Azure App Service, etc.

  Por ejemplo, puede iniciar casi cualquier flujo de trabajo según una programación mediante el desencadenador Periodicidad. O bien, puede hacer que el flujo de trabajo espere hasta que se le llame mediante el desencadenador Solicitud. Para más información, consulte [Desencadenadores y acciones integrados para Logic Apps](../connectors/built-in.md).

* **Aplicación lógica**: el recurso de Azure que se crea para compilar flujos de trabajo. En función de las necesidades del escenario y de los requisitos de la solución, puede crear aplicaciones lógicas que se ejecuten en el entorno del servicio Logic Apps multiinquilino o de un solo inquilino, o bien que se ejecuten en un entorno de servicio de integración. Para más información, consulte [Entornos de host para aplicaciones lógicas](#host-environments).

<a name="how-do-logic-apps-work"></a>

## <a name="how-logic-apps-work"></a>Cómo funcionan las aplicaciones lógicas

En una aplicación lógica, cada flujo de trabajo siempre se inicia con un único [desencadenador](#logic-app-concepts). Un desencadenador se desencadena cuando se cumple una condición, por ejemplo, cuando se produce un evento específico o cuando los datos cumplen criterios específicos. Muchos desencadenadores incluyen [funcionalidades de programación](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md) que controlan la frecuencia con la que se ejecuta el flujo de trabajo. Después del desencadenador, hay una o varias [acciones](#logic-app-concepts) que ejecutan operaciones que, por ejemplo, procesan, controlan o convierten datos que recorren el flujo de trabajo, o que avanzan el flujo de trabajo al paso siguiente.

Por ejemplo, el flujo de trabajo siguiente comienza con un desencadenador de Dynamics que tiene una condición integrada llamada **Cuando se actualiza un registro**. Las acciones incluyen la transformación de código XML, la llamada a una aplicación web que actualiza los datos, la evaluación de una condición que controla qué acciones realizar y el envío de una notificación por correo electrónico con los resultados. Cuando el desencadenador detecta un evento que cumple la condición, el desencadenador se activa y las acciones del flujo de trabajo comienzan a ejecutarse. Cada vez que el desencadenador se activa, Azure Logic Apps crea una instancia del flujo de trabajo que ejecuta la aplicación lógica.

![Diseñador de Logic Apps: ejemplo de flujo de trabajo](./media/logic-apps-overview/azure-logic-apps-designer.png)

Puede crear visualmente flujos de trabajo mediante el diseñador de Logic Apps en Azure Portal, Visual Studio Code o Visual Studio. Cada flujo de trabajo también tiene una definición subyacente que se describe mediante notación de objetos JavaScript (JSON). Si lo prefiere, puede editar flujos de trabajo cambiando la definición de este JSON. Para algunas tareas de creación y administración, Logic Apps proporciona compatibilidad con comandos de Azure PowerShell y la CLI de Azure. Para la implementación automatizada, Logic Apps admite plantillas de Azure Resource Manager.

<a name="host-environments"></a>

## <a name="host-environments"></a>Entornos de host

En función de los requisitos del escenario y de la solución, puede crear aplicaciones lógicas que difieren en el entorno del servicio Logic Apps en que se ejecutan y en la forma en que los flujos de trabajo usan los recursos. En la tabla siguiente se resumen las diferencias.

| Entorno | [Modelo de precios](logic-apps-pricing.md) | Descripción |
|-------------|----------------------------------------|-------------|
| Azure Logic Apps (multiinquilino) | Consumo | Ninguna aplicación lógica puede tener más de un flujo de trabajo. <p><p>Los flujos de trabajo de diferentes aplicaciones lógicas de *varios inquilinos* comparten el mismo procesamiento (proceso), almacenamiento, red, etc. |
| Azure Logic Apps [[un único inquilino (versión preliminar)](logic-apps-overview-preview.md)] | [Versión preliminar](logic-apps-overview-preview.md#pricing-model) | Una aplicación lógica puede tener varios flujos de trabajo. <p><p>Los flujos de trabajo de la *misma aplicación lógica que se encuentran en un único inquilino* comparten el mismo procesamiento (proceso), almacenamiento, red, etc. |
| [Entorno del servicio de integración (ISE)](connect-virtual-network-vnet-isolated-environment-overview.md) | Fijo | Ninguna aplicación lógica puede tener más de un flujo de trabajo. <p><p>Los flujos de trabajo de distintas aplicaciones lógicas  *que se encuentran en un único inquilino* comparten el mismo procesamiento (proceso), almacenamiento, red, etc. |
||||

Las aplicaciones lógicas hospedadas en entornos de servicio de Logic Apps también tienen límites diferentes. Para más información, consulte el artículo sobre los [límites de Logic Apps](logic-apps-limits-and-config.md) y la sección sobre los [límites en Logic Apps (versión preliminar)](logic-apps-overview-preview.md#limits).

## <a name="why-use-logic-apps"></a>Motivos para usar Logic Apps

La plataforma de integración de Logic Apps proporciona conectores de API administrados por Microsoft creados previamente y operaciones integradas, con el fin de que pueda conectar e integrar aplicaciones, datos, servicios y sistemas de forma más fácil y rápida. Puede centrarse más en el diseño e implementación de la funcionalidad y lógica de negocios de la solución, no en averiguar cómo acceder a los recursos.

Normalmente, no tendrá que escribir código. Sin embargo, si necesita escribir código, puede utilizar [Azure Functions ](../azure-functions/functions-overview.md) para crear fragmentos de código y ejecutar ese código desde un flujo de trabajo. También puede crear fragmentos de código que se ejecuten en el flujo de trabajo mediante la acción [**Código en línea**](logic-apps-add-run-inline-code.md). Si el flujo de trabajo necesita interactuar con eventos de servicios de Azure, aplicaciones personalizadas u otras soluciones, puede supervisar, enrutar y publicar eventos mediante [Azure Event Grid](../event-grid/overview.md).

Microsoft Azure administra completamente Logic Apps, lo que le permite despreocuparse de la compilación, hospedaje, escalado, administración, supervisión y mantenimiento de estos servicios. Si usa estas funcionalidades para crear [aplicaciones y soluciones "sin servidor"](../logic-apps/logic-apps-serverless-overview.md), puede centrarse en la lógica de negocios y en la funcionalidad. Estos servicios se escalan automáticamente para satisfacer sus necesidades, agilizar las integraciones y ayudarle a crear sólidas aplicaciones en la nube con poco código, o sin código alguno.

Para ver la forma en que otras empresas han mejorado su agilidad y se han centrado en sus negocios principales al combinar Logic Apps con otros servicios de Azure y productos de Microsoft, consulte estos [testimonios de clientes](https://aka.ms/logic-apps-customer-stories).

Las secciones siguientes proporcionan más información acerca de las funcionalidades y beneficios de Logic Apps:

#### <a name="visually-create-and-edit-workflows-with-easy-to-use-tools"></a>Creación y edición visual de flujos de trabajo con herramientas fáciles de usar

Ahorre tiempo y simplifique los procesos complejos mediante el uso de herramientas de diseño visual de Logic Apps. Cree flujos de trabajo de principio a fin mediante el diseñador de Logic Apps en Azure Portal, Visual Studio Code o Visual Studio. Inicie su flujo de trabajo con un desencadenador y agregue varias acciones desde la [galería de conectores](/connectors/connector-reference/connector-reference-logicapps-connectors).

Si va a crear una aplicación lógica multiinquilino y a empezar a trabajar más rápidamente al [crear un flujo de trabajo desde la galería de plantillas](../logic-apps/logic-apps-create-logic-apps-from-templates.md). Estas plantillas están disponibles para patrones de flujo de trabajo comunes, que van desde una conectividad sencilla para aplicaciones de software como servicio (SaaS) hasta soluciones B2B avanzadas, además de plantillas "solo para disfrutar".

#### <a name="connect-different-systems-across-various-environments"></a>Conexión de distintos sistemas en varios entornos

Determinados patrones y flujos de trabajo son fáciles de describir pero difíciles de implementar en el código. La plataforma Logic Apps le ayuda a conectar sin problemas sistemas dispares en entornos híbridos, locales y en la nube. Por ejemplo, puede conectar una solución de marketing de la nube con un sistema de facturación local, o bien centralizar la mensajería a través de las API y los sistemas mediante Azure Service Bus. Logic Apps ofrece una forma rápida, fiable y coherente de proporcionar soluciones que se pueden volver a utilizar y configurar para estos escenarios.

#### <a name="write-once-reuse-often"></a>Escriba una vez y úselo tanto como quiera

Cree aplicaciones lógicas como plantillas de Azure Resource Manager para que pueda [configurar y automatizar las implementaciones](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) en varios entornos y regiones.

#### <a name="first-class-support-for-enterprise-integration-and-b2b-scenarios"></a>Soporte de primera clase para escenarios B2B y de integración empresarial

Las empresas y organizaciones se comunican electrónicamente entre sí mediante el uso de protocolos y formatos de mensajes estándares del sector, pero diferentes, como EDIFACT, AS2, X12 y RosettaNet. Mediante el uso de las [funcionalidades de integración empresarial](../logic-apps/logic-apps-enterprise-integration-overview.md) compatibles con Logic Apps, puede crear flujos de trabajo que transformen los formatos de mensaje que usan las entidades en formatos que los sistemas de su organización puedan interpretar y procesar. Logic Apps administra estos intercambios de forma fácil y segura mediante cifrado y firmas digitales.

Empiece poco a poco con sus servicios y sistemas actuales y crezca de forma gradual a su propio ritmo. Cuando esté listo, la plataforma Logic Apps le ayudará a implementar y realizar un escalado vertical a escenarios de integración más avanzados mediante estas funcionalidades y muchas más:

* Integre y compile a partir de [Microsoft BizTalk Server](/biztalk/core/introducing-biztalk-server), [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md), [Azure Functions](../azure-functions/functions-overview.md), [Azure API Management](../api-management/api-management-key-concepts.md), etc.
* Intercambie mensajes mediante los protocolos [EDIFACT](../logic-apps/logic-apps-enterprise-integration-edifact.md), [AS2,](../logic-apps/logic-apps-enterprise-integration-as2.md) [X12](../logic-apps/logic-apps-enterprise-integration-x12.md)y [RosettaNet](logic-apps-enterprise-integration-rosettanet.md).
* Procese [mensajes XML](../logic-apps/logic-apps-enterprise-integration-xml.md) y [archivos planos](../logic-apps/logic-apps-enterprise-integration-flatfile.md).
* Cree una [cuenta de integración](./logic-apps-enterprise-integration-create-integration-account.md) para almacenar y administrar artefactos B2B, como [entidades](../logic-apps/logic-apps-enterprise-integration-partners.md), [acuerdos](../logic-apps/logic-apps-enterprise-integration-agreements.md), [mapas de transformación](../logic-apps/logic-apps-enterprise-integration-maps.md), [esquemas de validación](../logic-apps/logic-apps-enterprise-integration-schemas.md), etc.

Por ejemplo, si usa Microsoft BizTalk Server, los flujos de trabajo pueden comunicarse con BizTalk Server mediante el [conector de BizTalk Server](../connectors/managed.md#on-premises-connectors). Luego, puede ampliar o realizar operaciones similares a las de BizTalk en los flujos de trabajo mediante los [conectores de la cuenta de integración](../connectors/managed.md#integration-account-connectors). Por otra parte, BizTalk Server puede comunicarse con sus flujos de trabajo mediante el [adaptador de Microsoft BizTalk Server Adapter para Logic Apps](https://www.microsoft.com/download/details.aspx?id=54287). Aprenda a [configurar y usar el adaptador de BizTalk Server](/biztalk/core/logic-app-adapter).

#### <a name="built-in-extensibility"></a>Extensibilidad integrada

Si no hay ningún conector adecuado disponible para ejecutar el código que desee, puede crear y llamar a sus propios fragmentos de código desde el flujo de trabajo mediante [Azure Functions](../azure-functions/functions-overview.md). O bien, cree sus propias [API](../logic-apps/logic-apps-create-api-app.md) y [conectores personalizados](../logic-apps/custom-connector-overview.md), a los que puede llamar desde sus flujos de trabajo.

#### <a name="access-resources-inside-azure-virtual-networks"></a>Acceso a recursos dentro de redes virtuales de Azure

Los flujos de trabajo de aplicaciones lógicas pueden acceder a recursos protegidos, como máquinas virtuales y otros sistemas o servicios dentro de una [red virtual de Azure](../virtual-network/virtual-networks-overview.md) al crear un [*entorno del servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md). Un ISE es una instancia dedicada del servicio Logic Apps que usa recursos dedicados y se ejecuta de forma independiente del servicio Logic Apps público, global y multiinquilino.

La ejecución de aplicaciones lógicas en una instancia dedicada ayuda a reducir el impacto que podrían tener otros inquilinos de Azure en el rendimiento de las aplicaciones, también conocido como el [efecto "vecinos ruidosos"](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors). Un ISE también proporciona estas ventajas:

* Direcciones IP estáticas propias, que son independientes de las direcciones IP estáticas que comparten las aplicaciones lógicas en el servicio multiinquilino. Puede configurar una dirección IP de salida única, pública, estática y predecible para comunicarse con los sistemas de destino. De ese modo, no es necesario configurar aperturas adicionales del firewall en los sistemas de destino para cada ISE.

* Se aumentan los límites de duración de ejecución, retención de almacenamiento, rendimiento, solicitud HTTP y tiempos de espera de respuesta, tamaños de mensaje y solicitudes de conector personalizado. Para más información, consulte el artículo sobre los [límites y la configuración de Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md).

Cuando se crea un ISE, Azure *inserta* o implementa dicho ISE en su red virtual de Azure. A continuación, puede usar ese ISE como la ubicación de las aplicaciones lógicas y cuentas de integración que necesitan acceso. Para más información sobre cómo crear un ISE, consulte [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

#### <a name="pricing-options"></a>Opciones de precios

Cada tipo de aplicación lógica, que difiere en función de las funcionalidades y dónde se ejecutan (multiinquilino, inquilino único, entorno de servicio de integración), tiene un [modelo de precios](../logic-apps/logic-apps-pricing.md) diferente. Por ejemplo, las aplicaciones lógicas multiinquilino usan precios basados en el consumo, mientras que las que están en un entorno de servicio de integración usan precios fijos. Obtenga más información sobre [precios y medición](../logic-apps/logic-apps-pricing.md) de Logic Apps.

## <a name="how-does-logic-apps-differ-from-functions-webjobs-and-power-automate"></a>¿En qué se diferencia Logic Apps de Functions, WebJobs y Power Automate?

Todos estos servicios le ayudan a conectarse y a reunir sistemas dispares. Cada servicio tiene sus ventajas y beneficios, por lo que la combinación de sus funcionalidades es la mejor manera de crear rápidamente un sistema de integración escalable y completo. Para más información, consulte el artículo en el que se proporciona la información necesaria para [elegir entre Logic Apps, Functions, WebJobs y Power Automate](../azure-functions/functions-compare-logic-apps-ms-flow-webjobs.md).

## <a name="get-started"></a>Primeros pasos

Para poder empezar a usar Azure Logic Apps, necesita una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/). Si la tiene, pruebe este [inicio rápido para crear su primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md), que supervisa el nuevo contenido de un sitio web a través de una fuente RSS y envía correo electrónico cuando aparece nuevo contenido.

## <a name="other-resources"></a>Otros recursos

Obtenga más información sobre la plataforma de Logic Apps con estos vídeos introductorios:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Go-serverless-Enterprise-integration-with-Azure-Logic-Apps/player]
>
> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Connect-and-extend-your-mainframe-to-the-cloud-with-Logic-Apps/player]

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación del primer flujo de trabajo de Logic Apps en Azure Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* Más información sobre [soluciones sin servidor con Azure](../logic-apps/logic-apps-serverless-overview.md)
* Más información sobre la [integración de B2B con Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
