---
title: Creación de un panel de Customer Insights
description: Administre los comentarios de clientes, datos de redes sociales, etc. mediante la compilación de un panel de clientes con Azure Logic Apps y Azure Functions
services: logic-apps
ms.suite: integration
author: jeffhollan
ms.author: jehollan
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 03/15/2018
ms.openlocfilehash: f6da9737416eb3d23977d0e4e6b0d981019944cd
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108277253"
---
# <a name="create-a-streaming-customer-insights-dashboard-with-azure-logic-apps-and-azure-functions"></a>Creación de un panel de streaming de Customer Insights con Azure Logic Apps y Azure Functions

Azure ofrece herramientas [sin servidor](https://azure.microsoft.com/solutions/serverless/) que ayudan a compilar aplicaciones y hospedarlas rápidamente en la nube, sin necesidad de preocuparse de la infraestructura. En este tutorial, puede crear un panel para actuar en función de los comentarios de los clientes, analizarlos con Machine Learning y publicar información en un origen, como Power BI o Azure Data Lake.

Para esta solución, use estos componentes principales de Azure para las aplicaciones sin servidor: [Azure Functions](https://azure.microsoft.com/services/functions/) y [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/).
Azure Logic Apps ofrece un motor de flujo de trabajo sin servidor en la nube para poder crear orquestaciones de todos los componentes sin servidor y conectarse a más de doscientos servicios y API. Azure Functions proporciona informática sin servidor en la nube. Esta solución usa Azure Functions para marcar tweets de clientes en función de palabras clave predefinidas.

En este escenario, creará una aplicación lógica que permite encontrar comentarios de los clientes. Algunos de los conectores que pueden ayudar a responder a los comentarios de los clientes son: Outlook.com, Office 365, Survey Monkey, Twitter y [una solicitud HTTP desde un formulario web](/archive/blogs/logicapps/calling-a-logic-app-from-an-html-form). El flujo de trabajo creado supervisa un hashtag en Twitter.

Puede [compilar la solución completa en Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md) e [implementarla con la plantilla de Azure Resource Manager](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md). Para ver un tutorial en el que se explique cómo crear esta solución, [vea este vídeo de Channel 9](https://aka.ms/logicappsdemo). 

## <a name="trigger-on-customer-data"></a>Interactuación con los datos de clientes

1. En Azure Portal o Visual Studio, cree una aplicación lógica en blanco. 

   Si es la primera vez que interactúa con Logic Apps, consulte el [inicio rápido de Azure Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md) o el [inicio rápido de Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

2. En el Diseñador de aplicación lógica, busque y agregue el desencadenador de Twitter que tenga esta acción: **Cuando se publique un nuevo tweet**

3. Configure el desencadenador para escuchar tweets con una palabra clave o hashtag.

   En el caso de los desencadenadores basados en sondeos, como el desencadenador de Twitter, la propiedad de periodicidad determina la frecuencia con que la aplicación lógica busca nuevos elementos.

   ![Ejemplo de desencadenador de Twitter][1]

Esta aplicación lógica ahora se desencadena con todos los tweets nuevos. Puede usar y analizar los datos de los tweets para comprender mejor las opiniones expresadas. 

## <a name="analyze-tweet-text"></a>Análisis del texto de los tweets

Para detectar la opinión que transmite un texto, puede usar [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).

1. En el desencadenador del Diseñador de aplicación lógica, seleccione **Nuevo paso**.

2. Busque el conector **Análisis de texto**.

3. Seleccione la acción **Detectar sentimiento**.

4. Si se le solicita, proporcione una clave de Cognitive Services válida para el servicio Text Analytics.

5. En **Cuerpo de la solicitud**, seleccione el campo **Texto del tweet**, que proporciona el texto de los tweets como entrada para el análisis.

Después de obtener los datos del tweet y la información sobre el tweet, ya puede usar otros conectores relevantes y sus acciones:

* **Power BI, incorporación de filas al conjunto de datos de transmisión por secuencias**: vea los tweets entrantes en un panel de Power BI.
* **Azure Data Lake, anexado de archivos**: agregue datos de los clientes a un conjunto de datos de Azure Data Lake datos para incluirlos en los trabajos de análisis.
* **SQL, incorporación de filas**: almacene datos en una base de datos para su posterior recuperación.
* **Slack, envío de mensajes**: notifique a un canal de Slack sobre comentarios negativos que puedan requerir medidas.

También puede crear una función de Azure para poder realizar procesamientos personalizados de los datos. 

## <a name="process-data-with-azure-functions"></a>Procesamiento de datos con Azure Functions

Antes de crear una función, cree una aplicación de función en la suscripción de Azure. Además, para que la aplicación lógica llame directamente a una función, esta última debe tener un enlace de desencadenador HTTP; por ejemplo, use la plantilla **HttpTrigger**. Obtenga información sobre [cómo crear su primera aplicación de función y función en Azure Portal](../azure-functions/functions-get-started.md).

En este escenario, use el texto del tweet como el cuerpo de la solicitud de la función de Azure. En el código de la función, defina la lógica que determina si el texto del tweet contiene una palabra clave o frase. Mantenga la función tan sencilla o compleja como sea necesario para el escenario.
Al final de la función, devuelva una respuesta a la aplicación lógica con algunos datos; por ejemplo, un valor booleano sencillo, como `containsKeyword` o un objeto complejo.

> [!TIP]
> Para acceder a una respuesta compleja desde una función de una aplicación lógica, use la acción **Análisis del archivo JSON**.

Cuando haya terminado, guarde la función y, a continuación, agréguela como una acción en la aplicación lógica que se va a compilar.

## <a name="add-azure-function-to-logic-app"></a>Adición de una función de Azure a la aplicación lógica

1. En el Diseñador de aplicación lógica, en la acción **Detectar sentimiento**, seleccione **Nuevo paso**.

2. Busque el conector de **Azure Functions** y luego seleccione la función que ha creado.

3. En **Cuerpo de la solicitud**, seleccione **Texto del tweet**.

![Paso de función de Azure configurado][2]

## <a name="run-and-monitor-your-logic-app"></a>Ejecución y supervisión de la aplicación lógica

Para revisar cualquier ejecución actual o anterior de la aplicación lógica, puede usar las funcionalidades de supervisión y depuración enriquecidas que Azure Logic Apps ofrece en Azure Portal o en Visual Studio o con los SDK y las API de REST de Azure.

Para probar fácilmente la aplicación lógica, en el Diseñador de aplicación lógica, seleccione **Ejecutar desencadenador**. El desencadenador sondea los tweets según el programa especificado, hasta encontrar un tweet que cumpla sus criterios. A medida que avanza la ejecución, el diseñador muestra una vista dinámica de dicha ejecución.

Para ver historiales de ejecuciones anteriores en Visual Studio o en Azure Portal: 

* Abra Visual Studio Cloud Explorer. Busque la aplicación lógica y abra el menú contextual de la aplicación. Seleccione **Abrir el historial de ejecución**.

  > [!TIP]
  > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio.

* Busque la aplicación lógica en Azure Portal. En el menú de la aplicación lógica, elija **Información general**. 

## <a name="create-automated-deployment-templates"></a>Creación de plantillas de implementación automatizada

Después de crear una solución de aplicación lógica, puede capturar e implementar la aplicación como una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) en cualquier región de Azure del mundo. Puede utilizar esta funcionalidad para modificar los parámetros para crear versiones diferentes de la aplicación y para integrar la solución en Azure Pipelines. También puede incluir Azure Functions en la plantilla de la implementación, para poder administrar la solución completa con todas las dependencias como una única plantilla. Aprenda a [automatizar la implementación de aplicaciones lógicas](logic-apps-azure-resource-manager-templates-overview.md).

Para acceder a una plantilla de implementación de ejemplo con una función de Azure, consulte el [repositorio de plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.web/function-app-create-dynamic/azuredeploy.json).

## <a name="next-steps"></a>Pasos siguientes

* [Busque otros ejemplos y escenarios comunes de Azure Logic Apps](logic-apps-examples-and-scenarios.md)

<!-- Image References -->
[1]: ./media/logic-apps-scenario-social-serverless/twitter.png
[2]: ./media/logic-apps-scenario-social-serverless/function.png