---
title: Desencadenadores y acciones integrados para Azure Logic Apps
description: Use desencadenadores y acciones integrados para crear flujos de trabajo automatizados que integren aplicaciones, datos, servicios y sistemas, y así poder controlar flujos de trabajo y para administrar datos mediante Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, azla
ms.topic: conceptual
ms.date: 04/20/2021
ms.openlocfilehash: 045d7391c9c3c2870efddc0aed4ae7590db938d2
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107796338"
---
# <a name="built-in-triggers-and-actions-for-logic-apps"></a>Desencadenadores y acciones integrados para Logic Apps


Los [desencadenadores y las acciones integrados](apis-list.md) proporcionan [maneras de controlar la programación y la estructura del flujo de trabajo](#control-workflow), [ejecutar su propio código](#run-code-from-workflows), [administrar o manipular datos](#manage-or-manipulate-data) y completar otras tareas en los flujos de trabajo. A diferencia de los [conectores administrados](managed.md), muchas operaciones integradas no están vinculadas a un servicio, sistema o protocolo específicos. Por ejemplo, puede iniciar casi cualquier flujo de trabajo según una programación mediante el desencadenador Periodicidad. O bien, puede hacer que el flujo de trabajo espere hasta que se llame mediante el desencadenador Solicitud. Todas las operaciones integradas se ejecutan de forma nativa en el servicio Logic Apps, y la mayoría no requieren que cree una conexión antes de usarlas. 

Logic Apps también proporciona operaciones integradas para una menor cantidad de servicios, sistemas y protocolos, como Azure Service Bus, Azure Functions, SQL y AS2, entre otros. El número y el intervalo varían en función de si crea una aplicación lógica multiinquilino o una aplicación lógica de un solo inquilino. En algunos casos, una versión integrada y una versión del conector administrado están disponibles. En cambio, en la mayoría de los casos, la versión integrada proporciona un mejor rendimiento, funcionalidades y precios, entre otras cosas. Por ejemplo, para [intercambiar mensajes B2B mediante el protocolo AS2](../logic-apps/logic-apps-enterprise-integration-as2.md), seleccione la versión integrada a menos que necesite usar funcionalidades de seguimiento, puesto que solo están disponibles en la versión del conector administrado (en desuso).

En la siguiente lista solo se describen algunas de las tareas que puede realizar con [desencadenadores y acciones integrados](#understand-triggers-and-actions):

- Ejecute flujos de trabajo cuando use programaciones personalizadas y avanzadas. Para obtener más información sobre la programación, revise la [sección del comportamiento de periodicidad que cuenta con la información general del conector de Logic Apps](apis-list.md#recurrence-behavior).
- Organice y controle la estructura del flujo de trabajo, por ejemplo, mediante bucles y condiciones.
- También puede trabajar con variables, fechas, operaciones de datos, transformaciones de contenido y operaciones por lotes.
- Comuníquese con otros puntos de conexión mediante acciones y desencadenadores HTTP.
- Recibir y responder solicitudes.
- Llame a sus propias funciones (Azure Functions), como las aplicaciones web (Azure App Services), las API (Azure API Management) u otros flujos de trabajo de Logic Apps que pueden recibir solicitudes, y así sucesivamente.

## <a name="understand-triggers-and-actions"></a>Comprensión de los desencadenadores y las acciones

Logic Apps proporciona los siguientes desencadenadores y acciones integrados:

:::row:::
    :::column:::
        [![Icono de programación en Logic Apps][schedule-icon]][schedule-doc]
        \
        \
        [**Programación**][schedule-doc]
        \
        \
        [**Periodicidad:** ][schedule-recurrence-doc] permite desencadenar un flujo de trabajo en función de la periodicidad especificada.
        \
        \
        [**Ventana deslizante:** ][schedule-sliding-window-doc] permite desencadenar un flujo de trabajo que necesita controlar los datos en fragmentos continuos.
        \
        \
        [**Retraso:** ][schedule-delay-doc] permite pausar el flujo de trabajo durante el tiempo especificado.
        \
        \
        [**Retrasar hasta**][schedule-delay-until-doc]: permite pausar el flujo de trabajo hasta la fecha y hora especificadas.
    :::column-end:::
    :::column:::
        [![Icono de Batch en Logic Apps][batch-icon]][batch-doc]
        \
        \
        [**Batch**][batch-doc]
        \
        \
        [**Mensajes por lotes:** ][batch-doc] permite desencadenar un flujo de trabajo que procese los mensajes en lotes.
        \
        \
        [**Enviar mensajes al lote:**][batch-doc] permite llamar a un flujo de trabajo existente que se inicia actualmente con un **desencadenador de mensajes de Batch**.
    :::column-end:::
    :::column:::
        [![Icono de HTTP en Logic Apps][http-icon]][http-doc]
        \
        \
        [**HTTP**][http-doc]
        \
        \
        Llame a un punto de conexión HTTP o HTTPS mediante el desencadenador o la acción HTTP. 
        \
        \
        También puede usar estos otros desencadenadores y acciones HTTP integrados: 
        - [HTTP + Swagger][http-swagger-doc]
        - [HTTP + Webhook][http-webhook-doc]
    :::column-end:::
    :::column:::
        [![Icono de solicitud][http-request-icon]][http-request-doc]
        \
        \
        [**Solicitud**][http-request-doc]
        \
        \
        [**Cuando se recibe una solicitud HTTP:** ][http-request-doc] permite esperar una solicitud de otro flujo de trabajo, aplicación o servicio. Este desencadenador permite llamar al flujo de trabajo sin tener que revisarlo o sondearlo en un horario.
        \
        \
        [**Respuesta:**][http-request-doc] permite responder a una solicitud que recibe el desencadenador denominado **Cuando se recibe una solicitud HTTP** en el mismo flujo de trabajo.
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Icono de Azure API Management en Logic Apps][azure-api-management-icon]][azure-api-management-doc]
        \
        \
        [**Azure API Management**][azure-api-management-doc]
        \
        \
        Llame a sus propios desencadenadores y acciones en las API que defina, administre y publique mediante [Azure API Management](../api-management/api-management-key-concepts.md). <p><p>**Nota:** Esto no se admite cuando se usa el [nivel de consumo para API Management](../api-management/api-management-features.md).
    :::column-end:::
    :::column:::
        [![Icono de App de Azure Services en Logic Apps][azure-app-services-icon]][azure-app-services-doc]
        \
        \
        [**Azure App Services**][azure-app-services-doc]
        \
        \
        Llame a las aplicaciones que haya creado y hospédelas en [Azure App Service](../app-service/overview.md); por ejemplo, API Apps y Web Apps.
        \
        \
        Cuando Swagger se incluye, los desencadenadores y las acciones que hayan definido las aplicaciones aparecen como cualquier otro desencadenador y acción de primera clase en Azure Logic Apps.
    :::column-end:::
    :::column:::
        [![Icono de Azure Logic Apps en Logic Apps][azure-logic-apps-icon]][nested-logic-app-doc]
        \
        \
        [**Azure Logic Apps**][nested-logic-app-doc]
        \
        \
        Llame a otros flujos de trabajo que comiencen con el desencadenador se solicitud denominado **Cuando se recibe una solicitud HTTP**.
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="run-code-from-workflows"></a>Ejecución de código desde flujos de trabajo

Logic Apps proporciona acciones integradas para ejecutar código propio en el flujo de trabajo:

:::row:::
    :::column:::
        [![Icono de Azure Functions en Logic Apps][azure-functions-icon]][azure-functions-doc]
        \
        \
        [**Azure functions**][azure-functions-doc]
        \
        \
        Llame a las [funciones hospedadas en Azure](../azure-functions/functions-overview.md) para ejecutar sus propios *fragmentos de código* (C# o Node.js) en el flujo de trabajo.
    :::column-end:::
    :::column:::
        [![Icono del código en línea en Logic Apps][inline-code-icon]][inline-code-doc]
        \
        \
        [**Código en línea**][inline-code-doc]
        \
        \
        [**Ejecutar código JavaScript**][inline-code-doc]: permite agregar y ejecutar sus propios *fragmentos de código* de JavaScript insertados en el flujo de trabajo.
    :::column-end:::
    :::column:::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="control-workflow"></a>Controlar el flujo de trabajo

Logic Apps proporciona acciones integradas para estructurar y controlar las acciones en el flujo de trabajo:

:::row:::
    :::column:::
        [![Icono de acción Condition en Logic Apps][condition-icon]][condition-doc]
        \
        \
        [**Condition**][condition-doc]
        \
        \
        Evalúa una condición y ejecuta acciones diferentes según si la condición es true o false.
    :::column-end:::
    :::column:::
        [![Icono de la acción For Each en Logic Apps][for-each-icon]][for-each-doc]
        \
        \
        [**For Each**][for-each-doc]
        \
        \
        Realiza las mismas acciones en todos los elementos de una matriz.
    :::column-end:::
    :::column:::
        [![Icono de la acción de ámbito en Logic Apps][scope-icon]][scope-doc]
        \
        \
        [**Nombre**][scope-doc]
        \
        \
        Agrupa las acciones en *ámbitos*, que obtienen su propio estado después de que las acciones del ámbito terminen de ejecutarse.
    :::column-end:::
    :::column:::
        [![Icono de la acción Switch en Logic Apps][switch-icon]][switch-doc]
        \
        \
        [**Switch**][switch-doc]
        \
        \
        Agrupa las acciones en *casos* a los que se asignan valores únicos, excepto el caso predeterminado. Ejecuta solo el caso cuyo valor asignado coincida con el resultado de una expresión, objeto o token. Si no hay coincidencias, ejecuta el caso predeterminado.
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        [![Icono de la acción Terminate en Logic Apps][terminate-icon]][terminate-doc]
        \
        \
        [**Terminate**][terminate-doc]
        \
        \
        Detiene el flujo de trabajo que la aplicación lógica está ejecutando activamente. 
    :::column-end:::
    :::column:::
        [![Icono de la acción Until en Logic Apps][until-icon]][until-doc]
        \
        \
        [**Until**][until-doc]
        \
        \
        Repite las acciones hasta que la condición especificada sea true o cambie algún estado.
    :::column-end:::
    :::column:::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="manage-or-manipulate-data"></a>Administrar o manipular datos

Logic Apps proporciona acciones integradas para trabajar con salidas de datos y sus formatos:

:::row:::
    :::column:::
        [![Icono de la acción Data Operations en Logic Apps][data-operations-icon]][data-operations-doc]
        \
        \
        [**Data Operations**][data-operations-doc]
        \
        \
        Realice operaciones con datos. 
        \
        \
        **Redactar**: crea una única salida a partir de varias entradas con varios tipos. 
        \
        \
        **Crear tabla CSV**: crea una tabla con valores separados por comas (CSV) a partir de una matriz con objetos JSON. 
        \
        \
        **Crear tabla HTML**: crea una tabla HTML a partir de una matriz con objetos JSON. 
        \
        \
        **Filtrar matriz**: crea una matriz a partir de elementos de otra matriz que cumplen los criterios. 
        \
        \
        **Combinar**: crea una cadena a partir de todos los elementos de una matriz y los separa con el delimitador especificado. 
        \
        \
        **Analizar JSON**: crea tokens sencillos a partir de propiedades y sus valores en contenido JSON para que pueda usar esas propiedades en el flujo de trabajo. 
        \
        \
        **Select**: crea una matriz con objetos JSON mediante la transformación de elementos o valores de otra matriz, y la asignación de esos elementos a propiedades especificadas.
    :::column-end:::
    :::column:::
        ![Icono de la acción Date Time en Logic Apps][date-time-icon]
        \
        \
        **Date Time**
        \
        \
        Realice operaciones con marcas de tiempo.
        \
        \
        **Agregar a la hora**: agrega el número especificado de unidades a una marca de tiempo. 
        \
        \
        **Convertir la zona horaria**: Convierte una marca de tiempo de la zona horaria de origen a la zona horaria de destino. 
        \
        \
        **Hora actual**: Devuelve la marca de tiempo actual como una cadena. 
        \
        \
        **Obtener la hora futura**: Devuelve la marca de tiempo actual más las unidades de tiempo especificadas. 
        \
        \
        **Obtener la hora pasada**: Devuelve la marca de tiempo actual menos las unidades de tiempo especificadas. 
        \
        \
        **Restar de la hora**: permite restar un número de unidades de tiempo de una marca de tiempo.
    :::column-end:::
    :::column:::
        [![Icono de la acción Variables en Logic Apps][variables-icon]][variables-doc]
        \
        \
        [**Variables**][variables-doc]
        \
        \
        Realiza operaciones con variables.
        \
        \
        **Anexar a la variable de matriz**: inserta un valor como el último elemento en una matriz almacenada por una variable. 
        \
        \
        **Anexar a la variable de cadena**: inserta un valor como el último carácter en una cadena almacenada por una variable.
        \
        \
        **Reducir variable**: reduce una variable por un valor constante.
        \
        \
        **Incremento de variable**: incrementa una variable por un valor constante. 
        \
        \
        **Inicializar la variable**: crea una variable y declara su tipo de datos y el valor inicial. 
        \
        \
        **Establecer variable**: asigna otro valor a una variable existente. 
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de API personalizadas que se pueden llamar desde Logic Apps](/logic-apps/logic-apps-create-api-app)

<!-- Built-ins icons -->
[azure-api-management-icon]: ./media/apis-list/azure-api-management.png
[azure-app-services-icon]: ./media/apis-list/azure-app-services.png
[azure-functions-icon]: ./media/apis-list/azure-functions.png
[azure-logic-apps-icon]: ./media/apis-list/azure-logic-apps.png
[batch-icon]: ./media/apis-list/batch.png
[condition-icon]: ./media/apis-list/condition.png
[data-operations-icon]: ./media/apis-list/data-operations.png
[date-time-icon]: ./media/apis-list/date-time.png
[for-each-icon]: ./media/apis-list/for-each-loop.png
[http-icon]: ./media/apis-list/http.png
[http-request-icon]: ./media/apis-list/request.png
[http-response-icon]: ./media/apis-list/response.png
[http-swagger-icon]: ./media/apis-list/http-swagger.png
[http-webhook-icon]: ./media/apis-list/http-webhook.png
[inline-code-icon]: ./media/apis-list/inline-code.png
[schedule-icon]: ./media/apis-list/recurrence.png
[scope-icon]: ./media/apis-list/scope.png
[switch-icon]: ./media/apis-list/switch.png
[terminate-icon]: ./media/apis-list/terminate.png
[until-icon]: ./media/apis-list/until.png
[variables-icon]: ./media/apis-list/variables.png


<!--Built-in doc links-->
[azure-api-management-doc]: ../api-management/get-started-create-service-instance.md "Creación de una instancia del servicio Azure API Management para administrar y publicar sus API"
[azure-app-services-doc]: ../logic-apps/logic-apps-custom-api-host-deploy-call.md "Integrar aplicaciones lógicas con App Service API Apps"
[azure-functions-doc]: ../logic-apps/logic-apps-azure-functions.md "Integración de aplicaciones lógicas con Azure Functions"
[batch-doc]: ../logic-apps/logic-apps-batch-process-send-receive-messages.md "Procesamiento de mensajes en grupos o lotes"
[condition-doc]: ../logic-apps/logic-apps-control-flow-conditional-statement.md "Evaluar una condición y ejecutar acciones diferentes según si la condición es true o false"
[for-each-doc]: ../logic-apps/logic-apps-control-flow-loops.md#foreach-loop "Realizar las mismas acciones en todos los elementos de una matriz"
[http-doc]: ./connectors-native-http.md "Llamar a puntos de conexión HTTP o HTTPS desde las aplicaciones lógicas"
[http-request-doc]: ./connectors-native-reqres.md "Recibir solicitudes HTTP en las aplicaciones lógicas"
[http-response-doc]: ./connectors-native-reqres.md "Responder a solicitudes HTTP desde las aplicaciones lógicas"
[http-swagger-doc]: ./connectors-native-http-swagger.md "Llamar a puntos de conexión REST desde las aplicaciones lógicas"
[http-webhook-doc]: ./connectors-native-webhook.md "Esperar eventos específicos de los puntos de conexión HTTP o HTTPS"
[inline-code-doc]: ../logic-apps/logic-apps-add-run-inline-code.md "Agregar y ejecutar fragmentos de código JavaScript de las aplicaciones lógicas"
[nested-logic-app-doc]: ../logic-apps/logic-apps-http-endpoint.md "Integrar aplicaciones lógicas con flujos de trabajo anidados"
[query-doc]: ../logic-apps/logic-apps-perform-data-operations.md#filter-array-action "Seleccionar y filtrar matrices con la acción Consulta"
[schedule-doc]: ../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md "Ejecutar aplicaciones lógicas en función de una programación"
[schedule-delay-doc]: ./connectors-native-delay.md "Retrasar la ejecución de la siguiente acción"
[schedule-delay-until-doc]: ./connectors-native-delay.md "Retrasar la ejecución de la siguiente acción"
[schedule-recurrence-doc]:  ./connectors-native-recurrence.md "Ejecutar aplicaciones lógicas según una programación periódica"
[schedule-sliding-window-doc]: ./connectors-native-sliding-window.md "Ejecutar aplicaciones lógicas que necesitan manipular datos en fragmentos contiguos"
[scope-doc]: ../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md "Organizar las acciones en grupos, que obtienen su propio estado después de que las acciones del grupo terminen de ejecutarse"
[switch-doc]: ../logic-apps/logic-apps-control-flow-switch-statement.md "Organizar las acciones en los casos, a los que se asignan valores únicos. Ejecutar solo el caso cuyo valor asignado coincida con el resultado de una expresión, objeto o token. Si no hay coincidencias, ejecutar el caso predeterminado"
[terminate-doc]: ../logic-apps/logic-apps-workflow-actions-triggers.md#terminate-action "Detener o cancelar un flujo de trabajo que se está ejecutando activamente para la aplicación lógica"
[until-doc]: ../logic-apps/logic-apps-control-flow-loops.md#until-loop "Repetir las acciones hasta que la condición especificada sea true o cambie algún estado"
[data-operations-doc]: ../logic-apps/logic-apps-perform-data-operations.md "Realizar operaciones de datos como el filtrado de matrices o la creación de tablas CSV y HTML"
[variables-doc]: ../logic-apps/logic-apps-create-variables-store-values.md "Realizar operaciones con variables, por ejemplo, inicializar, establecer, incrementar, reducir y anexar a la variable de cadena o matriz"
