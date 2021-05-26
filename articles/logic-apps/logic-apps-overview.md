---
title: Introducción a Azure Logic Apps
description: Azure Logic Apps es una plataforma en la nube para la automatización de flujos de trabajo que integran aplicaciones, datos, servicios y sistemas con un código mínimo o sin código. Los flujos de trabajo se pueden ejecutar en un entorno dedicado, multiinquilino o de inquilino único.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: overview
ms.custom: mvc
ms.date: 05/07/2021
ms.openlocfilehash: c971f184aad7bfaf6d36a00ec1fb95f474c3a61c
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110376957"
---
# <a name="what-is-azure-logic-apps"></a>¿Qué es Azure Logic Apps?

[Azure Logic Apps](https://azure.microsoft.com/services/logic-apps) es una plataforma basada en la nube para crear y ejecutar [*flujos de trabajo*](#logic-app-concepts) automatizados que integren sus aplicaciones, datos, servicios y sistemas. Esta plataforma permite desarrollar rápidamente soluciones de integración altamente escalables para escenarios intraempresariales y de negocio a negocio (B2B). Logic Apps forma parte de [Azure Integration Services](https://azure.microsoft.com/product-categories/integration/), por lo que simplifica la forma de conectar sistemas heredados, modernos y vanguardistas a través de entornos híbridos, locales y en la nube.

En la lista siguiente se describen solo algunas tareas, procesos empresariales y cargas de trabajo de ejemplo que puede automatizar con el servicio Logic Apps:

* Programación y envío de notificaciones por correo electrónico mediante Office 365 cuando se produce un evento específico, por ejemplo, cuando se ha cargado un archivo nuevo.
* Procesamiento y enrutamiento de pedidos entre sistemas locales y servicios en la nube.
* Traslado de archivos cargados de un servidor SFTP o FTP a Azure Storage.
* Supervisión de tuits, análisis de opiniones y creación de alertas o tareas para los elementos que se deben revisar.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Introducing-Azure-Logic-Apps/player]

En función del tipo de recurso de aplicación lógica que elija y cree, las aplicaciones lógicas se ejecutan en un Entorno del servicio de integración dedicado, multiinquilino o de inquilino único. Por ejemplo, al crear en contenedores aplicaciones lógicas basadas en inquilino único, puede implementar las aplicaciones como contenedores y ejecutarlas en cualquier lugar que Azure Functions pueda ejecutar. Para obtener más información, consulte [Diferencias entre el tipo de recurso y el entorno de host para las aplicaciones lógicas](#resource-environment-differences).

Para acceder y ejecutar operaciones de forma segura en tiempo real en varios orígenes de datos, puede elegir [*conectores administrados*](#logic-app-concepts) de un [ecosistema de más de 400 conectores de Azure y cada vez mayor](/connectors/connector-reference/connector-reference-logicapps-connectors) para usarlos en los flujos de trabajo, por ejemplo:

* Servicios de Azure, como Blob Storage y Service Bus
* Servicios de Office 365, como Outlook, Excel y SharePoint
* Servidores de bases de datos, como SQL y Oracle
* Sistemas empresariales, como SAP e IBM MQ
* Recursos compartidos de archivos, como FTP y SFTP

Para comunicarse con cualquier punto de conexión de servicio, ejecutar su propio código, organizar el flujo de trabajo o manipular los datos, puede usar desencadenadores y acciones [*integrados*](#logic-app-concepts), que se ejecutan de forma nativa dentro del servicio Logic Apps. Por ejemplo, los desencadenadores integrados incluyen Solicitud, HTTP y Periodicidad. Las acciones integradas incluyen Condición, Bucle For each, Ejecutar código JavaScript y operaciones que llaman a funciones de Azure, aplicaciones web o aplicaciones de API hospedadas en Azure y otros flujos de trabajo de Logic Apps.

Para escenarios de integración B2B, Logic Apps incluye funcionalidades de [BizTalk Server](/biztalk/core/introducing-biztalk-server). Para definir artefactos de negocio a negocio (B2B), cree una [*cuenta de integración*](#logic-app-concepts) donde almacene estos artefactos. Después de vincular esta cuenta a la aplicación lógica, los flujos de trabajo pueden usar estos artefactos B2B e intercambiar mensajes que cumplan con los estándares de intercambio electrónico de datos (EDI) y Enterprise Application Integration (EAI).

Para más información sobre las formas en las que los flujos de trabajo pueden acceder y trabajar con aplicaciones, datos, servicios y sistemas, consulte la siguiente documentación:

* [Conectores para Azure Logic Apps](../connectors/apis-list.md)
* [Desencadenadores y acciones integrados para Logic Apps](../connectors/built-in.md)
* [Conectores administrados para Logic Apps](../connectors/managed.md)
* [Soluciones de integración empresarial B2B con Azure Logic Apps y Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md)

<a name="logic-app-concepts"></a>

## <a name="key-terms"></a>Términos clave

* *Aplicación lógica*: recurso de Azure que se va a crear cuando se desea desarrollar un flujo de trabajo. En función de las necesidades del escenario y de los requisitos de la solución, puede crear aplicaciones lógicas que se ejecuten en el entorno de Azure Logic Apps multiinquilino o de inquilino único, o bien que se ejecuten en un Entorno del servicio de integración (ISE). Para obtener más información, consulte [Diferencias entre el tipo de recurso y el entorno de host para las aplicaciones lógicas](#resource-environment-differences).

* *Flujo de trabajo*: una serie de pasos que definen una tarea o proceso, empezando por un único desencadenador y seguido de una o varias acciones.

* *Desencadenador*: el primer paso que inicia cada flujo de trabajo y especifica la condición que se debe cumplir antes de ejecutar cualquier acción en el flujo de trabajo. Por ejemplo, un evento de desencadenador podría recibir un correo electrónico en la bandeja de entrada o detectar un nuevo archivo en una cuenta de almacenamiento.

* *Acción*: cada paso posterior que sigue después del desencadenador y ejecuta alguna operación en un flujo de trabajo.

* *Desencadenador o acción integrados*: una operación de Logic Apps que se ejecuta de forma nativa que proporciona una manera de controlar la programación o estructura del flujo de trabajo, ejecutar su propio código, administrar o manipular datos o completar otras tareas del flujo de trabajo. La mayoría de las operaciones integradas no están asociadas a ningún servicio o sistema. Muchas tampoco requieren que primero cree una conexión desde el flujo de trabajo y autentique su identidad. Sin embargo, las operaciones integradas también están disponibles para algunos servicios, sistemas y protocolos usados frecuentemente, como Azure Functions, Azure API Management, Azure App Service, etc.

  Por ejemplo, puede iniciar casi cualquier flujo de trabajo según una programación mediante el desencadenador Periodicidad. O bien, puede hacer que el flujo de trabajo espere hasta que se le llame mediante el desencadenador Solicitud. Para más información, consulte [Desencadenadores y acciones integrados para Logic Apps](../connectors/built-in.md).

* *Conector administrado*: un proxy o contenedor creado previamente en torno a una API REST que proporciona desencadenadores y acciones precompilados para que el flujo de trabajo acceda a una aplicación, datos, servicio o sistema específicos. Para poder crear la mayoría de los conectores administrados, primero debe crear una conexión desde el flujo de trabajo y autenticar su identidad.

  Por ejemplo, puede iniciar un flujo de trabajo con un desencadenador o agregar una acción que trabaje con Azure Blob Storage, Office 365, Salesforce o servidores SFTP. Microsoft se encarga de hospedar y administrar los conectores administrados. Para más información, consulte [Conectores administrados para Logic Apps](../connectors/managed.md).

* *Cuenta de integración*: recurso de Azure que se va a crear cuando desee definir y almacenar artefactos B2B para su uso en los flujos de trabajo. Después de vincular esta cuenta a la aplicación lógica, los flujos de trabajo pueden usar estos artefactos B2B e intercambiar mensajes que cumplan con los estándares de intercambio electrónico de datos (EDI) y Enterprise Application Integration (EAI).

  Por ejemplo, puede definir socios comerciales, contratos, esquemas, mapas y otros artefactos B2B. Puede crear flujos de trabajo que usen estos artefactos e intercambiar mensajes a través de protocolos como AS2, EDIFACT, X12 y RosettaNet. Para más información, consulte [Creación y administración de cuentas de integración para la integración empresarial B2B](logic-apps-enterprise-integration-create-integration-account.md).

<a name="how-do-logic-apps-work"></a>

## <a name="how-logic-apps-work"></a>Cómo funcionan las aplicaciones lógicas

En una aplicación lógica, cada flujo de trabajo siempre se inicia con un único [desencadenador](#logic-app-concepts). Un desencadenador se desencadena cuando se cumple una condición, por ejemplo, cuando se produce un evento específico o cuando los datos cumplen criterios específicos. Muchos desencadenadores incluyen [funcionalidades de programación](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md) que controlan la frecuencia con la que se ejecuta el flujo de trabajo. Después del desencadenador, hay una o varias [acciones](#logic-app-concepts) que ejecutan operaciones que, por ejemplo, procesan, controlan o convierten datos que recorren el flujo de trabajo, o que avanzan el flujo de trabajo al paso siguiente.

Por ejemplo, el flujo de trabajo siguiente comienza con un desencadenador de Dynamics que tiene una condición integrada llamada **Cuando se actualiza un registro**. Las acciones incluyen la transformación de código XML, la llamada a una aplicación web que actualiza los datos, la evaluación de una condición que controla qué acciones realizar y el envío de una notificación por correo electrónico con los resultados. Cuando el desencadenador detecta un evento que cumple la condición, el desencadenador se activa y las acciones del flujo de trabajo comienzan a ejecutarse. Cada vez que el desencadenador se activa, Azure Logic Apps crea una instancia del flujo de trabajo que ejecuta la aplicación lógica.

![Diseñador de Logic Apps: ejemplo de flujo de trabajo](./media/logic-apps-overview/azure-logic-apps-designer.png)

Puede crear visualmente flujos de trabajo mediante el diseñador de Logic Apps en Azure Portal, Visual Studio Code o Visual Studio. Cada flujo de trabajo también tiene una definición subyacente que se describe mediante notación de objetos JavaScript (JSON). Si lo prefiere, puede editar flujos de trabajo cambiando la definición de este JSON. Para algunas tareas de creación y administración, Logic Apps proporciona compatibilidad con comandos de Azure PowerShell y la CLI de Azure. Para la implementación automatizada, Logic Apps admite plantillas de Azure Resource Manager.

<a name="resource-environment-differences"></a>

## <a name="resource-type-and-host-environment-differences"></a>Diferencias entre el tipo de recurso y el entorno de host

Para crear flujos de trabajo de aplicación lógica, elija el tipo de recurso **Logic Apps** en función del escenario, los requisitos de la solución, las funcionalidades que desea y el entorno donde quiere ejecutar los flujos de trabajo.

En la tabla siguiente se resumen brevemente las diferencias entre el tipo de recurso original **Logic Apps (consumo)** y el tipo de recurso **Logic Apps (estándar)** . También aprenderá en qué se diferencia el modelo de *inquilino único* del modelo *multiinquilino* y el *Entorno del servicio de integración (ISE)* para implementar, hospedar y ejecutar los flujos de trabajo de la aplicación lógica.

[!INCLUDE [Logic app resource type and environment differences](../../includes/logic-apps-resource-environment-differences-table.md)]

## <a name="why-use-logic-apps"></a>Motivos para usar Logic Apps

La plataforma de integración de Logic Apps proporciona conectores de API administrados por Microsoft creados previamente y operaciones integradas, con el fin de que pueda conectar e integrar aplicaciones, datos, servicios y sistemas de forma más fácil y rápida. Puede centrarse más en el diseño e implementación de la funcionalidad y lógica de negocios de la solución, no en averiguar cómo acceder a los recursos.

Normalmente, no tendrá que escribir código. Sin embargo, si necesita escribir código, puede utilizar [Azure Functions ](../azure-functions/functions-overview.md) para crear fragmentos de código y ejecutar ese código desde un flujo de trabajo. También puede crear fragmentos de código que se ejecuten en el flujo de trabajo mediante la acción [**Código en línea**](logic-apps-add-run-inline-code.md). Si el flujo de trabajo necesita interactuar con eventos de servicios de Azure, aplicaciones personalizadas u otras soluciones, puede supervisar, enrutar y publicar eventos mediante [Azure Event Grid](../event-grid/overview.md).

Microsoft Azure administra completamente Logic Apps, lo que le permite despreocuparse de la compilación, hospedaje, escalado, administración, supervisión y mantenimiento de estos servicios. Si usa estas funcionalidades para crear [aplicaciones y soluciones "sin servidor"](../logic-apps/logic-apps-serverless-overview.md), puede centrarse en la lógica de negocios y en la funcionalidad. Estos servicios se escalan automáticamente para satisfacer sus necesidades, agilizar las integraciones y ayudarle a crear sólidas aplicaciones en la nube con poco código, o sin código alguno.

Para ver la forma en que otras empresas han mejorado su agilidad y se han centrado en sus negocios principales al combinar Logic Apps con otros servicios de Azure y productos de Microsoft, consulte estos [testimonios de clientes](https://aka.ms/logic-apps-customer-stories).

Las secciones siguientes proporcionan más información acerca de las funcionalidades y beneficios de Logic Apps:

#### <a name="visually-create-and-edit-workflows-with-easy-to-use-tools"></a>Creación y edición visual de flujos de trabajo con herramientas fáciles de usar

Ahorre tiempo y simplifique los procesos complejos mediante el uso de herramientas de diseño visual de Logic Apps. Cree flujos de trabajo de principio a fin mediante el diseñador de Logic Apps en Azure Portal, Visual Studio Code o Visual Studio. Inicie su flujo de trabajo con un desencadenador y agregue varias acciones desde la [galería de conectores](/connectors/connector-reference/connector-reference-logicapps-connectors).

Si va a crear una aplicación lógica basada en multiinquilino, empiece a trabajar más rápidamente al [crear un flujo de trabajo desde la galería de plantillas](../logic-apps/logic-apps-create-logic-apps-from-templates.md). Estas plantillas están disponibles para patrones de flujo de trabajo comunes, que van desde una conectividad sencilla para aplicaciones de software como servicio (SaaS) hasta soluciones B2B avanzadas, además de plantillas "solo para disfrutar".

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

Los flujos de trabajo de aplicaciones lógicas pueden acceder a recursos protegidos, como máquinas virtuales y otros sistemas o servicios dentro de una [red virtual de Azure](../virtual-network/virtual-networks-overview.md) al crear un [*entorno del servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md). Un ISE es una instancia dedicada del servicio Azure Logic Apps que usa recursos dedicados y se ejecuta de forma independiente del servicio Azure Logic Apps público, global y multiinquilino.

La ejecución de aplicaciones lógicas en una instancia dedicada ayuda a reducir el impacto que podrían tener otros inquilinos de Azure en el rendimiento de las aplicaciones, también conocido como el [efecto "vecinos ruidosos"](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors). Un ISE también proporciona estas ventajas:

* Direcciones IP estáticas propias, que son independientes de las direcciones IP estáticas que comparten las aplicaciones lógicas en el servicio multiinquilino. Puede configurar una dirección IP de salida única, pública, estática y predecible para comunicarse con los sistemas de destino. De ese modo, no es necesario configurar aperturas adicionales del firewall en los sistemas de destino para cada ISE.

* Se aumentan los límites de duración de ejecución, retención de almacenamiento, rendimiento, solicitud HTTP y tiempos de espera de respuesta, tamaños de mensaje y solicitudes de conector personalizado. Para más información, consulte el artículo sobre los [límites y la configuración de Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md).

Cuando se crea un ISE, Azure *inserta* o implementa dicho ISE en su red virtual de Azure. A continuación, puede usar ese ISE como la ubicación de las aplicaciones lógicas y cuentas de integración que necesitan acceso. Para más información sobre cómo crear un ISE, consulte [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

#### <a name="pricing-options"></a>Opciones de precios

Cada tipo de aplicación lógica, que difiere en función de las funcionalidades y dónde se ejecutan (multiinquilino, inquilino único, entorno de servicio de integración), tiene un [modelo de precios](../logic-apps/logic-apps-pricing.md) diferente. Por ejemplo, las aplicaciones lógicas basadas en multiinquilino usan precios basados en el consumo, mientras que las que están en un Entorno del servicio de integración usan precios fijos. Obtenga más información sobre [precios y medición](../logic-apps/logic-apps-pricing.md) de Logic Apps.

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