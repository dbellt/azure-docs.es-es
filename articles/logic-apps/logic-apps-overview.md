---
title: Introducción a Azure Logic Apps
description: Logic Apps es una plataforma en la nube para la creación de flujos de trabajo automatizados que integran aplicaciones, datos, servicios y sistemas para escenarios empresariales con un código mínimo.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: overview
ms.custom: mvc
ms.date: 04/26/2021
ms.openlocfilehash: 377abf4809e01c3931de393ff97718e0638c5c36
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108065515"
---
# <a name="what-is-azure-logic-apps"></a>¿Qué es Azure Logic Apps?

[Logic Apps](https://azure.microsoft.com/services/logic-apps) es una plataforma basada en la nube que le ayuda a integrar de forma rápida y sencilla aplicaciones, datos, sistemas y servicios mediante la creación y ejecución de [flujos de trabajo](#logic-app-concepts) automatizados. Como parte de [Azure Integration Services](https://azure.microsoft.com/product-categories/integration/), Logic Apps proporciona una manera más sencilla de crear, hospedar y administrar soluciones de integración altamente escalables para escenarios empresariales y de negocio a negocio (B2B) en entornos híbridos, locales y en la nube.

En esta lista se describen solo algunas tareas, procesos empresariales y cargas de trabajo de ejemplo que puede automatizar con el servicio Logic Apps:

* Procesamiento y enrutamiento de pedidos entre sistemas locales y servicios en la nube.
* Programación y envío de notificaciones por correo electrónico mediante Office 365 cuando se produce un evento específico.
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

Para escenarios de integración B2B, Logic Apps incluye funcionalidades de [BizTalk Server](/biztalk/core/introducing-biztalk-server). Puede crear una [cuenta de integración](logic-apps-enterprise-integration-create-integration-account.md) en la que definir socios comerciales, contratos, esquemas, mapas y otros artefactos B2B. Al vincular esta cuenta a una aplicación lógica, puede crear flujos de trabajo que funcionen con estos artefactos e intercambiar mensajes mediante protocolos como AS2, EDIFACT y X12.

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

* **Conector administrado**: un conector administrado por Microsoft que proporciona acceso a una aplicación, datos, servicio o sistema específicos. Para poder usarlos, la mayoría de los conectores administrados requieren que primero cree una conexión desde el flujo de trabajo y autentique su identidad.

  Por ejemplo, puede iniciar un flujo de trabajo con un desencadenador o incluir una acción que trabaje con Azure Blob Storage, Office 365, Salesforce o servidores SFTP. Para más información, consulte [Conectores administrados para Logic Apps](../connectors/managed.md).

* **Desencadenador o acción integrados**: una operación de Logic Apps que se ejecuta de forma nativa que proporciona una manera de controlar la programación o estructura del flujo de trabajo, ejecutar su propio código, administrar o manipular datos o completar otras tareas del flujo de trabajo. La mayoría de las operaciones integradas no están asociadas a ningún servicio o sistema. Muchas tampoco requieren que primero cree una conexión desde el flujo de trabajo y autentique su identidad. Las operaciones integradas también están disponibles para algunos servicios, sistemas y protocolos, como Azure Functions, Azure API Management, Azure App Services, etc.

  Por ejemplo, puede iniciar casi cualquier flujo de trabajo según una programación mediante el desencadenador Periodicidad. O bien, puede hacer que el flujo de trabajo espere hasta que se le llame mediante el desencadenador Solicitud. Para más información, consulte [Desencadenadores y acciones integrados para Logic Apps](../connectors/built-in.md).

<a name="how-do-logic-apps-work"></a>

## <a name="how-logic-apps-work"></a>Cómo funcionan las aplicaciones lógicas

En una aplicación lógica, cada flujo de trabajo siempre se inicia con un único [desencadenador](#logic-app-concepts). Un desencadenador se desencadena cuando se cumple una condición, por ejemplo, cuando se produce un evento específico o cuando los datos cumplen criterios específicos. Muchos desencadenadores incluyen [funcionalidades de programación](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md) que controlan la frecuencia con la que se ejecuta el flujo de trabajo. Después del desencadenador, hay una o varias [acciones](#logic-app-concepts) que ejecutan operaciones que, por ejemplo, procesan, controlan o convierten datos que recorren el flujo de trabajo, o que avanzan el flujo de trabajo al paso siguiente.

Por ejemplo, el flujo de trabajo siguiente comienza con un desencadenador de Dynamics que tiene una condición integrada llamada **Cuando se actualiza un registro**. Las acciones incluyen la transformación de código XML, la llamada a una aplicación web que actualiza los datos, la evaluación de una condición que controla qué acciones realizar y el envío de una notificación por correo electrónico con los resultados. Cuando el desencadenador detecta un evento que cumple la condición, el desencadenador se activa y las acciones del flujo de trabajo comienzan a ejecutarse. Cada vez que el desencadenador se activa, Azure Logic Apps crea una instancia del flujo de trabajo que ejecuta la aplicación lógica.

![Diseñador de Logic Apps: ejemplo de flujo de trabajo](./media/logic-apps-overview/azure-logic-apps-designer.png)

Puede crear visualmente flujos de trabajo mediante el diseñador Logic Apps en Azure Portal, Visual Studio o Visual Studio Code. Cada flujo de trabajo también tiene una definición subyacente que se describe mediante notación de objetos JavaScript (JSON). Si lo prefiere, puede crear y personalizar flujos de trabajo mediante el editor de vistas de código de Azure Portal, Visual Studio o Visual Studio Code. Para algunas tareas de creación y administración, Logic Apps proporciona compatibilidad con comandos de Azure PowerShell y la CLI de Azure. Para la implementación automatizada, Logic Apps admite plantillas de Azure Resource Manager.

## <a name="why-use-logic-apps"></a>¿Por qué usar Logic Apps?

Ante la creciente digitalización que están experimentando las empresas, la plataforma de Logic Apps le ayuda a conectar sistemas heredados, modernos y de vanguardia de manera más rápida y sencilla al proporcionarle API integradas como conectores administrados por Microsoft. De este modo, se puede centrar en las funcionalidades y en la lógica de negocios de sus aplicaciones. No tiene que preocuparse de compilar, hospedar, escalar, administrar, mantener ni supervisar las aplicaciones. Logic Apps hace todo eso por usted. Además, solo pagará por lo que utilice según un [modelo de precios](../logic-apps/logic-apps-pricing.md) de consumo.

Normalmente, no tendrá que escribir código. No obstante, si tiene que hacerlo, puede crear fragmentos de código con [Azure Functions](../azure-functions/functions-overview.md) y ejecutar ese código a petición desde los flujos de trabajo. Además, si los flujos de trabajo deben interactuar con eventos de los servicios de Azure, aplicaciones personalizadas u otras soluciones, puede usar [Azure Event Grid](../event-grid/overview.md) con los flujos de trabajo para supervisar, enrutar y publicar eventos.

Logic Apps, Functions y Event Grid están completamente administrados por Microsoft Azure, que le permite despreocuparse de compilar, hospedar, escalar, administrar, supervisar y mantener sus soluciones. Con la funcionalidad de crear [aplicaciones y soluciones "sin servidor"](../logic-apps/logic-apps-serverless-overview.md), solo tendrá que centrarse en la lógica de negocios. Estos servicios se escalan automáticamente para satisfacer sus necesidades, realizar integraciones con mayor rapidez y ayudarle a crear aplicaciones sólidas en la nube con un código mínimo.

Para ver cómo algunas empresas han conseguido mejorar su agilidad y centrarse en sus negocios principales al combinar Logic Apps con otros servicios de Azure y productos de Microsoft, consulte estos [testimonios de clientes](https://aka.ms/logic-apps-customer-stories).

Aquí encontrará más detalles sobre las funcionalidades y ventajas que obtendrá con Logic Apps:

### <a name="visually-build-workflows-with-easy-to-use-tools"></a>Creación visual de flujos de trabajo con herramientas fáciles de utilizar

Ahorre tiempo y simplifique procesos complejos con herramientas de diseño visuales. Cree flujos de trabajo de principio a fin mediante el diseñador de aplicaciones lógicas desde Azure Portal con el explorador o en Visual Studio. Inicie su flujo de trabajo con un desencadenador y añada una serie de acciones de la [galería de conectores](../connectors/apis-list.md).

### <a name="get-started-faster-with-logic-apps-templates"></a>Plantillas de Logic Apps para avanzar más rápidamente

Cree soluciones frecuentes más rápidamente al seleccionar flujos de trabajo predefinidos de la [galería de plantillas](../logic-apps/logic-apps-create-logic-apps-from-templates.md). En las plantillas puede encontrar desde sencillas aplicaciones de conectividad para software como servicio (SaaS) hasta soluciones B2B avanzadas, además de otras plantillas con las que divertirse. Descubra cómo [crear aplicaciones lógicas a partir de plantillas precompiladas](../logic-apps/logic-apps-create-logic-apps-from-templates.md).

### <a name="connect-disparate-systems-across-different-environments"></a>Conexión de sistemas dispares en diferentes entornos

Determinados patrones y flujos de trabajo son fáciles de describir pero difíciles de implementar en el código. Los flujos de trabajo de Logic Apps le ayudan a conectar fácilmente sistemas dispares en entornos locales y en la nube. Por ejemplo, puede conectar una solución de marketing de la nube en un sistema de facturación local, o centralizar la mensajería a través de las API y los sistemas con un bus de servicio empresarial. La plataforma de Logic Apps ofrece una manera rápida, fiable y coherente de proporcionar soluciones que se pueden volver a utilizar y configurar para estos escenarios.

### <a name="first-class-support-for-enterprise-integration-and-b2b-scenarios"></a>Soporte de primera clase para escenarios B2B y de integración empresarial

Las empresas y organizaciones se comunican electrónicamente entre sí mediante el uso de estándares del sector, pero con diferentes protocolos y formatos de mensajes, como EDIFACT, AS2 y X12. Con las características de [Enterprise Integration Pack (EIP)](../logic-apps/logic-apps-enterprise-integration-overview.md), puede crear flujos de trabajo que transformen los formatos de mensaje que usan sus asociados en formatos que los sistemas de su organización puedan interpretar y procesar. Logic Apps administra estos intercambios de forma fácil y también segura mediante cifrado y firmas digitales.

Empiece poco a poco con sus servicios y sistemas actuales y crezca de forma gradual a su propio ritmo. Cuando esté listo, Logic Apps y EIP le ayudarán a implementar y escalar verticalmente escenarios de integración más avanzados mediante estas funcionalidades y muchas más:

* Utilice estos productos y servicios para compilar:

  * [Microsoft BizTalk Server](/biztalk/core/introducing-biztalk-server)
  * [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)
  * [Funciones de Azure](../azure-functions/functions-overview.md)
  * [Azure API Management](../api-management/api-management-key-concepts.md)

* Procese [mensajes XML](../logic-apps/logic-apps-enterprise-integration-xml.md).

* Procese [archivos sin formato](../logic-apps/logic-apps-enterprise-integration-flatfile.md).

* Intercambie mensajes con los protocolos [EDIFACT](../logic-apps/logic-apps-enterprise-integration-edifact.md), [AS2](../logic-apps/logic-apps-enterprise-integration-as2.md) y [X12](../logic-apps/logic-apps-enterprise-integration-x12.md).

* Almacene y administre estos artefactos B2B y mucho más en un solo lugar con [cuentas de integración](./logic-apps-enterprise-integration-create-integration-account.md):

  * [Asociados](../logic-apps/logic-apps-enterprise-integration-partners.md)
  * [Contratos](../logic-apps/logic-apps-enterprise-integration-agreements.md) 
  * [Asignaciones de transformaciones XML](../logic-apps/logic-apps-enterprise-integration-maps.md)
  * [Esquemas de validación XML](../logic-apps/logic-apps-enterprise-integration-schemas.md)

Por ejemplo, si usa Microsoft BizTalk Server, Logic Apps podrá comunicarse con BizTalk Server utilizando el [conector de BizTalk Server](../connectors/managed.md#on-premises-connectors). Posteriormente, podrá ampliar o realizar operaciones como en BizTalk mediante los flujos de trabajo. Para ello, debe incluir [conectores de cuentas de integración](../connectors/managed.md#integration-account-connectors), que están disponibles con Enterprise Integration Pack.

Por otro lado, BizTalk Server puede establecer conexión y comunicación con Logic Apps utilizando el [adaptador de Microsoft BizTalk Server para Logic Apps](https://www.microsoft.com/download/details.aspx?id=54287). Aprenda a [configurar y usar el adaptador de BizTalk Server](/biztalk/core/logic-app-adapter).

### <a name="write-once-reuse-often"></a>Escriba una vez y úselo tanto como quiera

Cree aplicaciones lógicas como plantillas de Azure Resource Manager para que pueda [automatizar la implementación de aplicaciones lógicas](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) en varios entornos y regiones.

### <a name="access-resources-inside-azure-virtual-networks"></a>Acceso a recursos dentro de redes virtuales de Azure

Las aplicaciones lógicas pueden acceder a recursos protegidos, como máquinas virtuales y otros sistemas o servicios dentro de una red de [Azure Virtual Network](../virtual-network/virtual-networks-overview.md) al crear un [*entorno del servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md). Un ISE es una instancia dedicada del servicio Logic Apps que usa recursos dedicados y se ejecuta de forma independiente del servicio Logic Apps público, "global" y multiinquilino.

La ejecución de aplicaciones lógicas en una instancia dedicada e individual ayuda a reducir el impacto que podrían tener otros inquilinos de Azure en el rendimiento de la aplicación, también conocido como el [efecto "vecinos ruidosos"](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors). Un ISE también proporciona estas ventajas:

* Direcciones IP estáticas propias, que son independientes de las direcciones IP estáticas que comparten las aplicaciones lógicas en el servicio multiinquilino. Puede configurar una dirección IP de salida única, pública, estática y predecible para comunicarse con los sistemas de destino. De ese modo, no es necesario configurar aperturas adicionales del firewall en los sistemas de destino para cada ISE.

* Se aumentan los límites de duración de ejecución, retención de almacenamiento, rendimiento, solicitud HTTP y tiempos de espera de respuesta, tamaños de mensaje y solicitudes de conector personalizado. Para más información, consulte el artículo de [límites y configuración para Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md).

Cuando se crea un ISE, Azure *inserta* o implementa dicho ISE en su red virtual de Azure. A continuación, puede usar ese ISE como la ubicación de las aplicaciones lógicas y cuentas de integración que necesitan acceso. Para más información sobre cómo crear un ISE, vea [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

### <a name="built-in-extensibility"></a>Extensibilidad integrada

Si no encuentra el conector que desea para ejecutar código personalizado, puede ampliar las aplicaciones lógicas creando y llamando a sus propios fragmentos de código a petición mediante [Azure Functions](../azure-functions/functions-overview.md). Cree sus propias [API](../logic-apps/logic-apps-create-api-app.md) y [conectores personalizados](../logic-apps/custom-connector-overview.md) a los que puede llamar desde aplicaciones lógicas.

### <a name="pay-only-for-what-you-use"></a>Pague solo por lo que usa
  
Logic Apps usa un modelo de [precios y valoración](../logic-apps/logic-apps-pricing.md) basado en el consumo, a menos que haya creado anteriormente aplicaciones lógicas con planes de App Service.

Obtenga más información sobre Logic Apps con estos vídeos de introducción:

* [Integration with Logic Apps - Go from zero to hero](https://channel9.msdn.com/Events/Build/2017/C9R17) (Integración con Logic Apps: de la nada al todo)
* [Enterprise integration with Microsoft Azure Logic Apps](https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK2188) (Integración empresarial con Microsoft Azure Logic Apps)
* [Building advanced business processes with Logic Apps](https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK3179) (Creación de procesos empresariales avanzados con Logic Apps)

## <a name="how-does-logic-apps-differ-from-functions-webjobs-and-power-automate"></a>¿En qué se diferencia Logic Apps de Functions, WebJobs y Power Automate?

Todos estos servicios le ayudan a "unir" y conectar sistemas dispares. Cada servicio tiene sus ventajas y beneficios, por lo que la combinación de sus funcionalidades es la mejor manera de crear rápidamente un sistema de integración escalable y completo. Para más información, consulte [Elección entre Logic Apps, Functions, WebJobs y Power Automate](../azure-functions/functions-compare-logic-apps-ms-flow-webjobs.md).

## <a name="get-started"></a>Introducción

Logic Apps es uno de los muchos servicios hospedados en Microsoft Azure. Por lo tanto, para empezar, necesita una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

Si tiene una suscripción de Azure, pruebe la siguiente [guía de inicio rápido para crear su primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md), que supervisa el nuevo contenido de un sitio web a través de una fuente RSS y envía correo electrónico cuando aparece nuevo contenido.

## <a name="next-steps"></a>Pasos siguientes

* [Comprobación del tráfico con una aplicación lógica basada en una programación](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)
* Más información sobre [soluciones sin servidor con Azure](../logic-apps/logic-apps-serverless-overview.md)
* Más información sobre [integración B2B con Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
