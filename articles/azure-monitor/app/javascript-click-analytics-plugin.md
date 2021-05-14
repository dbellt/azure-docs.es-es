---
title: Complemento Click Analytics Auto-collection para el SDK de JavaScript para Application Insights
description: Cómo instalar y usar el complemento Click Analytics Auto-collection para el SDK de JavaScript para Application Insights.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: lagayhar
ms.openlocfilehash: 2eabbfb5928fea861874e78fa68ac196d16134d3
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108291341"
---
# <a name="click-analytics-auto-collection-plugin-for-application-insights-javascript-sdk"></a>Complemento Click Analytics Auto-collection para el SDK de JavaScript para Application Insights

Este complemento realiza automáticamente el seguimiento de los eventos de clic en páginas web y usa los atributos data-* en los elementos HTML para rellenar la telemetría de eventos.

## <a name="getting-started"></a>Introducción

Los usuarios pueden configurar el complemento Click Analytics Auto-collection a través de npm.

### <a name="npm-setup"></a>Configuración de npm

Instalación del paquete de npm:

```bash
npm install --save @microsoft/applicationinsights-clickanalytics-js @microsoft/applicationinsights-web
```

```js

import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ClickAnalyticsPlugin } from '@microsoft/applicationinsights-clickanalytics-js';

const clickPluginInstance = new ClickAnalyticsPlugin();
// Click Analytics configuration
const clickPluginConfig = {
  autoCapture: true
};
// Application Insights Configuration
const configObj = {
  instrumentationKey: "YOUR INSTRUMENTATION KEY",
  extensions: [clickPluginInstance],
  extensionConfig: {
    [clickPluginInstance.identifier]: clickPluginConfig
  },
};

const appInsights = new ApplicationInsights({ config: configObj });
appInsights.loadAppInsights();
```

## <a name="snippet-setup-ignore-if-using-npm-setup"></a>Configuración de fragmentos de código (omitir si se usa la configuración de npm)

```html
<script type="text/javascript" src="https://js.monitor.azure.com/scripts/b/ext/ai.clck.2.6.2.min.js"></script>
<script type="text/javascript">
  var clickPluginInstance = new Microsoft.ApplicationInsights.ClickAnalyticsPlugin();
  // Click Analytics configuration
  var clickPluginConfig = {
    autoCapture : true,
    dataTags: {
      useDefaultContentNameOrId: true
    }
  }
  // Application Insights Configuration
  var configObj = {
    instrumentationKey: "YOUR INSTRUMENTATION KEY",
    extensions: [
      clickPluginInstance
    ],
    extensionConfig: {
      [clickPluginInstance.identifier] : clickPluginConfig
    },
  };
  // Application Insights Snippet code
  !function(T,l,y){<!-- Removed the Snippet code for brevity -->}(window,document,{
    src: "https://js.monitor.azure.com/scripts/b/ai.2.min.js",
    crossOrigin: "anonymous",
    cfg: configObj
  });
</script>
```

## <a name="how-to-effectively-use-the-plugin"></a>Procedimiento para usar el complemento de forma eficaz

1. Los datos de telemetría que se generan en los eventos de clic se almacenan como `customEvents` en la sección Application Insights de Azure Portal.
2. Para rellenarse el elemento `name` de customEvent se siguen estas reglas:
    1.  El elemento `id` que se proporciona en `data-*-id` se usará como nombre de customEvent. Por ejemplo, si el elemento HTML en el que se ha hecho clic tiene el atributo "data-sample-id"="button1", "button1" será el nombre de customEvent.
    2. Si no existe ese atributo y `useDefaultContentNameOrId` está establecido en `true` en la configuración, se utilizará el atributo HTML `id` del elemento en el que se ha hecho clic o el nombre de contenido del elemento como nombre de customEvent. Si tanto el valor de `id` como el nombre de contenido están presentes, se da prioridad a `id`.
    3. Si el valor de `useDefaultContentNameOrId` es false, el nombre de customEvent será "not_specified".

    > [!TIP]
    > Se recomienda establecer `useDefaultContentNameOrId` en true para generar datos significativos.  
3. `parentDataTag` hace dos cosas:
    1. Si esta etiqueta está presente, el complemento capturará los atributos y valores de `data-*` de todos los elementos HTML primarios del elemento en el que se ha hecho clic.
    2. Para mejorar la eficacia, el complemento usa esta etiqueta como marca y cuando la encuentra, deja de procesar el DOM (Document Object Model).
    
    > [!CAUTION]
    > Una vez que se use `parentDataTag`, el SDK comenzará a buscar etiquetas principales en toda la aplicación y no solo el elemento HTML donde se usó.
4. Si `customDataPrefix` lo proporciona el usuario, debe comenzar por `data-`, por ejemplo `data-sample-`. En HTML, los atributos globales de `data-*` forman una clase de atributos denominados atributos de datos personalizados, que permiten intercambiar información propietaria entre el código HTML y su representación DOM mediante scripts. Los exploradores más antiguos (Internet Explorer, Safari) eliminarán los atributos que no comprenda, salvo que empiecen por `data-`.

    El signo `*` de `data-*`  se pueden reemplazar por cualquier nombre que siga la [regla de producción de nombres XML](https://www.w3.org/TR/REC-xml/#NT-Name), pero existen las siguientes restricciones:
    - El nombre no debe empezar por "xml", y da igual si se usan mayúsculas o minúsculas.
    - El nombre no debe contener ningún signo de punto y coma (U + 003A).
    - El nombre no debe contener mayúsculas.

## <a name="what-data-does-the-plugin-collect"></a>¿Qué datos recopila el complemento?

Estas son algunas de las propiedades clave capturadas de forma predeterminada cuando el complemento está habilitado:

### <a name="custom-event-properties"></a>Propiedades de eventos personalizados
| Nombre                  | Descripción                            | Ejemplo          |
| --------------------- | ---------------------------------------|-----------------|
| name                  | El valor de `name` de customEvent. [Aquí](#how-to-effectively-use-the-plugin) se muestra más información sobre cómo se rellena.| Acerca de              |
| itemType              | Tipo de evento.                                      | customEvent      |
|sdkVersion             | Versión del SDK de Application Insights junto con el complemento click|javascript:2.6.2_ClickPlugin2.6.2|

### <a name="custom-dimensions"></a>Dimensiones personalizadas
| Nombre                  | Descripción                            | Ejemplo          |
| --------------------- | ---------------------------------------|-----------------|
| actionType            | Tipo de acción que produjo el evento de clic. Puede ser clic con el botón izquierdo o con el botón derecho. | CL              |
| baseTypeSource        | Origen del tipo base del evento personalizado.                                      | ClickEvent      |
| clickCoordinates      | Coordenadas donde se desencadena el evento de clic.                            | 659X47          |
| contenido               | Marcador de posición para almacenar atributos y valores `data-*` adicionales.            | [{sample1:value1, sample2:value2}] |
| pageName              | Título de la página donde se desencadena el evento de clic.                      | Título de ejemplo    |
| parentId              | Identificador o nombre del elemento principal                                           | navbarContainer |

### <a name="custom-measurements"></a>Medidas personalizadas
| Nombre                  | Descripción                            | Ejemplo          |
| --------------------- | ---------------------------------------|-----------------|
| timeToAction          | Tiempo que el usuario ha necesitado en milisegundos para hacer clic en el elemento desde la carga inicial de la página | 87407              |

## <a name="configuration"></a>Configuración

| Nombre                  | Tipo                               | Valor predeterminado | Descripción                                                                                                                              |
| --------------------- | -----------------------------------| --------| ---------------------------------------------------------------------------------------------------------------------------------------- |
| autoCapture           | boolean                            | true    | Configuración de captura automática.                                |
| devolución de llamada              | [IValueCallback](#ivaluecallback)  | null    | Configuración de las devoluciones de llamada.                               |
| pageTags              | string                             | null    | Etiquetas de página.                                             |
| dataTags              | [ICustomDataTags](#icustomdatatags)| null    | Etiquetas de datos personalizadas que se proporcionan para invalidar las etiquetas predeterminadas que se usan para capturar datos de clic. |
| urlCollectHash        | boolean                            | false   | Habilita el registro de todos los valores que haya después de un carácter "#" de la dirección URL.                |
| urlCollectQuery       | boolean                            | false   | Habilita el registro de la cadena de consulta de la dirección URL.                            |
| behaviorValidator     | Función                           | null  | Función de devolución de llamada que se va a usar para la validación del valor `data-*-bhvr`. Para más información, vaya a la [sección behaviorValidator](#behaviorvalidator).|
| defaultRightClickBhvr | cadena (o) número                 | ''      | El valor predeterminado de Behavior cuando se ha producido un evento de clic con el botón derecho. Este valor se reemplazará si el elemento tiene el atributo `data-*-bhvr`. |
| dropInvalidEvents     | boolean                            | false   | Marca para eliminar todos aquellos eventos que no tengan datos de clic útiles.                                                                                   |

### <a name="ivaluecallback"></a>IValueCallback

| Nombre               | Tipo     | Valor predeterminado | Descripción                                                                             |
| ------------------ | -------- | ------- | --------------------------------------------------------------------------------------- |
| pageName           | Función | null    | Función para reemplazar el comportamiento predeterminado de captura de pageName.                           |
| pageActionPageTags | Función | null    | Función de devolución de llamada para aumentar el valor predeterminado de pageTags que se recopila en el evento pageAction.  |
| contentName        | Función | null    | Función de devolución de llamada para rellenar el valor de contentName personalizado.                                 |

### <a name="icustomdatatags"></a>ICustomDataTags

| Nombre                      | Tipo    | Valor predeterminado   | Etiqueta predeterminada que se usará en HTML |   Descripción                                                                                |
|---------------------------|---------|-----------|-------------|----------------------------------------------------------------------------------------------|
| useDefaultContentNameOrId | boolean | false     | No aplicable         |Recopila el atributo HTML estándar para contentName cuando un elemento determinado no está etiquetado con el valor de customDataPrefix predeterminado o el usuario no proporciona el valor de customDataPrefix. |
| customDataPrefix          | string  | `data-`   | `data-*`| El nombre de contenido de captura automática y el valor de los elementos que se etiquetan con el prefijo proporcionado. Por ejemplo, `data-*-id`, `data-<yourcustomattribute>` se pueden usar en las etiquetas HTML.   |
| aiBlobAttributeTag        | string  | `ai-blob` |  `data-ai-blob`| El complemento admite un atributo Blob JSON en lugar de los atributos individuales de`data-*`. |
| metaDataPrefix            | string  | null      | N/D  | Nombre y contenido del elemento meta del encabezado HTML de captura automática con prefijo proporcionado al realizarse la captura. Por ejemplo, `custom-` se puede usar en la etiqueta meta HTML. |
| captureAllMetaDataContent | boolean | false     | No aplicable   | Captura automática de todos los nombres y el contenido de los elementos meta del encabezado HTML. El valor predeterminado es False. Si está habilitado, invalidará el valor de metaDataPrefix proporcionado. |
| parentDataTag             | string  | null      |  N/D  | Detiene el recorrido hasta llegar al DOM para capturar el nombre de contenido y el valor de los elementos cuando encuentran esta etiqueta. Por ejemplo, `data-<yourparentDataTag>` puede usarse en las etiquetas HTML.|
| dntDataTag                | string  | `ai-dnt`  |  `data-ai-dnt`| El complemento omitirá los elementos HTML con este atributo al capturar datos de telemetría.|

### <a name="behaviorvalidator"></a>behaviorValidator

Las funciones behaviorValidator comprueban automáticamente que los comportamientos etiquetados en el código se ajustan a una lista predefinida. Esto garantiza que los comportamientos etiquetados sean coherentes con la taxonomía establecida de la empresa. No es necesario, ni se espera, que la mayoría de los clientes de Azure Monitor lo usen, pero está disponible para escenarios avanzados. Hay tres funciones de devolución de llamada de behaviorValidator que se exponen como parte de esta extensión. Sin embargo, si ninguna de ellas cubre sus necesidades, los usuarios pueden usar las suyas propias. La intención es usar su propia estructura de datos de comportamientos, el complemento usa esta función del validador al extraer los comportamientos de las etiquetas de datos.

| Nombre                   | Descripción                                                                        |
| ---------------------- | -----------------------------------------------------------------------------------|
| BehaviorValueValidator | Esta función de devolución de llamada se usa si la estructura de datos de comportamientos es una matriz de cadenas.|
| BehaviorMapValidator   | Esta función de devolución de llamada se usa si la estructura de datos de comportamientos es un diccionario.       |
| BehaviorEnumValidator  | Esta función de devolución de llamada se usa si la estructura de datos de comportamientos es una enumeración.            |

#### <a name="sample-usage-with-behaviorvalidator"></a>Ejemplo de uso con behaviorValidator

```js
var clickPlugin = Microsoft.ApplicationInsights.ClickAnalyticsPlugin;
var clickPluginInstance = new clickPlugin();

// Behavior enum values
var behaviorMap = {
  UNDEFINED: 0, // default, Undefined

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Page Experience [1-19]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  NAVIGATIONBACK: 1, // Advancing to the previous index position within a webpage
  NAVIGATION: 2, // Advancing to a specific index position within a webpage
  NAVIGATIONFORWARD: 3, // Advancing to the next index position within a webpage
  APPLY: 4, // Applying filter(s) or making selections
  REMOVE: 5, // Applying filter(s) or removing selections
  SORT: 6, // Sorting content
  EXPAND: 7, // Expanding content or content container
  REDUCE: 8, // Sorting content
  CONTEXTMENU: 9, // Context Menu
  TAB: 10, // Tab control
  COPY: 11, // Copy the contents of a page
  EXPERIMENTATION: 12, // Used to identify a third party experimentation event
  PRINT: 13, // User printed page
  SHOW: 14, //  Displaying an overlay
  HIDE: 15, //  Hiding an overlay
  MAXIMIZE: 16, //  Maximizing an overlay
  MINIMIZE: 17, // Minimizing an overlay
  BACKBUTTON: 18, //  Clicking the back button

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Scenario Process [20-39]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  STARTPROCESS: 20, // Initiate a web process unique to adopter
  PROCESSCHECKPOINT: 21, // Represents a checkpoint in a web process unique to adopter
  COMPLETEPROCESS: 22, // Page Actions that complete a web process unique to adopter
  SCENARIOCANCEL: 23, // Actions resulting from cancelling a process/scenario

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Download [40-59]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  DOWNLOADCOMMIT: 40, // Initiating an unmeasurable off-network download
  DOWNLOAD: 41, // Initiating a download

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Search [60-79]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SEARCHAUTOCOMPLETE: 60, // Auto-completing a search query during user input
  SEARCH: 61, // Submitting a search query
  SEARCHINITIATE: 62, // Initiating a search query
  TEXTBOXINPUT: 63, // Typing or entering text in the text box

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Commerce [80-99]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  VIEWCART: 82, // Viewing the cart
  ADDWISHLIST: 83, // Adding a physical or digital good or services to a wishlist
  FINDSTORE: 84, // Finding a physical store
  CHECKOUT: 85, // Before you fill in credit card info
  REMOVEFROMCART: 86, // Remove an item from the cart
  PURCHASECOMPLETE: 87, // Used to track the pageView event that happens when the CongratsPage or Thank You page loads after a successful purchase
  VIEWCHECKOUTPAGE: 88, // View the checkout page
  VIEWCARTPAGE: 89, // View the cart page
  VIEWPDP: 90, // View a PDP
  UPDATEITEMQUANTITY: 91, // Update an item's quantity
  INTENTTOBUY: 92, // User has the intent to buy an item
  PUSHTOINSTALL: 93, // User has selected the push to install option

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Authentication [100-119]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SIGNIN: 100, // User sign-in
  SIGNOUT: 101, // User sign-out

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Social [120-139]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SOCIALSHARE: 120, // "Sharing" content for a specific social channel
  SOCIALLIKE: 121, // "Liking" content for a specific social channel
  SOCIALREPLY: 122, // "Replying" content for a specific social channel
  CALL: 123, // Click on a "call" link
  EMAIL: 124, // Click on an "email" link
  COMMUNITY: 125, // Click on a "community" link

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Feedback [140-159]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  VOTE: 140, // Rating content or voting for content
  SURVEYCHECKPOINT: 145, // reaching the survey page/form

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Registration, Contact [160-179]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  REGISTRATIONINITIATE: 161, // Initiating a registration process
  REGISTRATIONCOMPLETE: 162, // Completing a registration process
  CANCELSUBSCRIPTION: 163, // Canceling a subscription
  RENEWSUBSCRIPTION: 164, // Renewing a subscription
  CHANGESUBSCRIPTION: 165, // Changing a subscription
  REGISTRATIONCHECKPOINT: 166, // Reaching the registration page/form

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Chat [180-199]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  CHATINITIATE: 180, // Initiating a chat experience
  CHATEND: 181, // Ending a chat experience

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Trial [200-209]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  TRIALSIGNUP: 200, // Signing-up for a trial
  TRIALINITIATE: 201, // Initiating a trial

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Signup [210-219]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SIGNUP: 210, // Signing-up for a notification or service
  FREESIGNUP: 211, // Signing-up for a free service

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Referals [220-229]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  PARTNERREFERRAL: 220, // Navigating to a partner's web property

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Intents [230-239]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  LEARNLOWFUNNEL: 230, // Engaging in learning behavior on a commerce page (ex. "Learn more click")
  LEARNHIGHFUNNEL: 231, // Engaging in learning behavior on a non-commerce page (ex. "Learn more click")
  SHOPPINGINTENT: 232, // Shopping behavior prior to landing on a commerce page

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Video [240-259]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  VIDEOSTART: 240, // Initiating a video
  VIDEOPAUSE: 241, // Pausing a video
  VIDEOCONTINUE: 242, // Pausing or resuming a video.
  VIDEOCHECKPOINT: 243, // Capturing predetermined video percentage complete.
  VIDEOJUMP: 244, // Jumping to a new video location.
  VIDEOCOMPLETE: 245, // Completing a video (or % proxy)
  VIDEOBUFFERING: 246, // Capturing a video buffer event
  VIDEOERROR: 247, // Capturing a video error
  VIDEOMUTE: 248, // Muting a video
  VIDEOUNMUTE: 249, // Unmuting a video
  VIDEOFULLSCREEN: 250, // Making a video full screen
  VIDEOUNFULLSCREEN: 251, // Making a video return from full screen to original size
  VIDEOREPLAY: 252, // Making a video replay
  VIDEOPLAYERLOAD: 253, // Loading the video player
  VIDEOPLAYERCLICK: 254, //  Click on a button within the interactive player
  VIDEOVOLUMECONTROL: 255, //  Click on video volume control
  VIDEOAUDIOTRACKCONTROL: 256, // Click on audio control within a video
  VIDEOCLOSEDCAPTIONCONTROL: 257, //  Click on the closed caption control
  VIDEOCLOSEDCAPTIONSTYLE: 258, //  Click to change closed caption style
  VIDEORESOLUTIONCONTROL: 259, //  Click to change resolution

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  //    Advertisement Engagement [280-299]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  ADBUFFERING: 283, // Ad is buffering
  ADERROR: 284, // Ad error
  ADSTART: 285, // Ad start
  ADCOMPLETE: 286, // Ad complete
  ADSKIP: 287, // Ad skipped
  ADTIMEOUT: 288, // Ad timed-out
  OTHER: 300 // Other
};

// Application Insights Configuration
var configObj = {
  instrumentationKey: "YOUR INSTRUMENTATION KEY",
  extensions: [clickPluginInstance],
  extensionConfig: {
    [clickPluginInstance.identifier]: {
      behaviorValidator: Microsoft.ApplicationInsights.BehaviorMapValidator(behaviorMap),
      defaultRightClickBhvr: 9
    },
  },
};
var appInsights = new Microsoft.ApplicationInsights.ApplicationInsights({
  config: configObj
});
appInsights.loadAppInsights();
```

## <a name="sample-app"></a>Aplicación de ejemplo

[Sencilla aplicación web con el complemento Click Analytics Auto-collection habilitado](https://go.microsoft.com/fwlink/?linkid=2152871).

## <a name="next-steps"></a>Pasos siguientes

- Consulte el [repositorio de GitHub](https://github.com/microsoft/ApplicationInsights-JS/tree/master/extensions/applicationinsights-clickanalytics-js) y el [paquete NPM](https://www.npmjs.com/package/@microsoft/applicationinsights-clickanalytics-js) para el complemento Click Analytics Auto-Collection.
- Use el [análisis de eventos en la experiencia de uso](usage-segmentation.md) para analizar los principales clics y segmentarlos por dimensiones disponibles.
- Busque los datos de clic en el campo de contenido del atributo customDimensions de la tabla CustomEvents de [Log Analytics](../logs/log-analytics-tutorial.md#write-a-query). Consulte la [aplicación de ejemplo](https://go.microsoft.com/fwlink/?linkid=2152871) para obtener orientación adicional.
- Cree un [libro](../visualize/workbooks-overview.md) o [exporte a Power BI](../logs/log-powerbi.md#integrating-queries) para crear visualizaciones personalizadas de los datos de clics.
