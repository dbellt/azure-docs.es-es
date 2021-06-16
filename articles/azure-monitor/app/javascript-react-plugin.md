---
title: Complemento React para el SDK de JavaScript de Application Insights
description: Instalación y uso del complemento React para el SDK de JavaScript de Application Insights
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: b703d8bc5253d8f5daeded8211e36dc119188b08
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112027792"
---
# <a name="react-plugin-for-application-insights-javascript-sdk"></a>Complemento React para el SDK de JavaScript de Application Insights

El complemento React para el SDK de JavaScript de Application Insights habilita lo siguiente:

- Seguimiento de cambios de ruta
- Estadísticas de uso de componentes de React

## <a name="getting-started"></a>Introducción

Instalación del paquete de npm:

```bash

npm install @microsoft/applicationinsights-react-js
npm install @microsoft/applicationinsights-web

```

## <a name="basic-usage"></a>Uso básico

Inicialice una conexión a Application Insights:

```javascript
// AppInsights.js
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ReactPlugin } from '@microsoft/applicationinsights-react-js';
import { createBrowserHistory } from 'history';

const browserHistory = createBrowserHistory({ basename: '' });
const reactPlugin = new ReactPlugin();
const appInsights = new ApplicationInsights({
    config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [reactPlugin],
        extensionConfig: {
          [reactPlugin.identifier]: { history: browserHistory }
        }
    }
});
appInsights.loadAppInsights();
export { reactPlugin, appInsights };
```

Ajuste el componente con la función de componente de orden superior para habilitar Application Insights en él:

```javascript
import React from 'react';
import { withAITracking } from '@microsoft/applicationinsights-react-js';
import { reactPlugin, appInsights } from './AppInsights';

// To instrument various React components usage tracking, apply the `withAITracking` higher-order
// component function.

class MyComponent extends React.Component {
    ...
}

// withAITracking takes 4 parameters ( reactPlugin, Component, ComponentName, className) 
// the first two are required and the other two are optional.

export default withAITracking(reactPlugin, MyComponent);
```

## <a name="configuration"></a>Configuración

| Nombre    | Valor predeterminado | Descripción                                                                                                    |
|---------|---------|----------------------------------------------------------------------------------------------------------------|
| history | null    | Historial de react-router. Para obtener más información, consulte la [documentación del paquete de react-router](https://reactrouter.com/web/api/history). Para obtener información sobre cómo tener acceso al objeto de historial fuera de los componentes, consulte las [preguntas frecuentes de react-router](https://github.com/ReactTraining/react-router/blob/master/FAQ.md#how-do-i-access-the-history-object-outside-of-components).    |

### <a name="react-components-usage-tracking"></a>Seguimiento de uso de componentes de React

Para instrumentar el seguimiento del uso de varios componentes de React, aplique la función de componente de orden superior `withAITracking`.

Medirá el tiempo desde el evento de `ComponentDidMount` a través del evento `ComponentWillUnmount`. Sin embargo, para que esto sea más preciso, restará el tiempo en el que el usuario estuvo inactivo `React Component Engaged Time = ComponentWillUnmount timestamp - ComponentDidMount timestamp - idle time`.

Para ver esta métrica en Azure Portal, tiene que ir al recurso de Application Insights, seleccionar la pestaña "Métricas" y configurar los gráficos vacíos para mostrar el nombre de la métrica personalizada "React Component Engaged Time (seconds)" (Tiempo de interactividad del componente de React [segundos]), seleccionar la agregación (suma, promedio, etc.) de la métrica y aplicar la división "Component Name" (Nombre del componente).

![Captura de pantalla del gráfico que muestra la métrica personalizada"React Component Engaged Time (seconds)" (Tiempo de interactividad del componente de React [segundos]) dividida por "Component Name" (Nombre del componente).](./media/javascript-react-plugin/chart.png)

También puede ejecutar consultas personalizadas para dividir los datos de Application Insights con el fin de generar informes y visualizaciones según sus requisitos. En Azure Portal navegue hasta el recurso de Application Insights, seleccione "Análisis" en el menú superior de la pestaña de información general y ejecute la consulta.

![Captura de pantalla de los resultados de la consulta de métricas personalizadas.](./media/javascript-react-plugin/query.png)

> [!NOTE]
> Las nuevas métricas personalizadas pueden tardar hasta 10 minutos en aparecer en Azure Portal.

## <a name="using-react-hooks"></a>Uso de los enlaces de React

Los [enlaces de React](https://reactjs.org/docs/hooks-reference.html) son un enfoque de administración del estado y del ciclo de vida en una aplicación React sin depender de los componentes de React basados en clases. El complemento de React en Application Insights proporciona una serie de integraciones de enlaces que funcionan de manera similar al enfoque de los componentes de orden superior.

### <a name="using-react-context"></a>Uso del contexto de React

Los enlaces de React para Application Insights están diseñados para usar el [contexto de React](https://reactjs.org/docs/context.html) como un aspecto integrado en él. Para usar el contexto, inicialice Application Insights como se ha descrito anteriormente y, después, importe el objeto de contexto:

```javascript
import React from "react";
import { AppInsightsContext } from "@microsoft/applicationinsights-react-js";
import { reactPlugin } from "./AppInsights";

const App = () => {
    return (
        <AppInsightsContext.Provider value={reactPlugin}>
            /* your application here */
        </AppInsightsContext.Provider>
    );
};
```

Este proveedor de contexto hará que Application Insights esté disponible como un enlace `useContext` en todos los componentes secundarios del mismo.

```javascript
import React from "react";
import { useAppInsightsContext } from "@microsoft/applicationinsights-react-js";

const MyComponent = () => {
    const appInsights = useAppInsightsContext();
    
    appInsights.trackMetric("Component 'MyComponent' is in use");
    
    return (
        <h1>My Component</h1>
    );
}
export default MyComponent;
```

### `useTrackMetric`

El enlace `useTrackMetric` replica la funcionalidad del componente de orden superior `withAITracking`, sin agregar un componente adicional a la estructura de componentes. El enlace adopta dos argumentos, el primero es la instancia de Application Insights (que se puede obtener del enlace `useAppInsightsContext`) y un identificador del componente para el seguimiento (como su nombre).

```javascript
import React from "react";
import { useAppInsightsContext, useTrackMetric } from "@microsoft/applicationinsights-react-js";

const MyComponent = () => {
    const appInsights = useAppInsightsContext();
    const trackComponent = useTrackMetric(appInsights, "MyComponent");
    
    return (
        <h1 onHover={trackComponent} onClick={trackComponent}>My Component</h1>
    );
}
export default MyComponent;
```

Funcionará como el componente de orden superior, pero responderá a los eventos de ciclo de vida de los enlaces, y no al ciclo de vida de un componente. El enlace se debe proporcionar explícitamente para los eventos de usuario, si es necesario ejecutarlo en determinadas interacciones.

### `useTrackEvent`

El enlace `useTrackEvent` se usa para realizar el seguimiento de cualquier evento personalizado que una aplicación pueda necesitar para realizar el seguimiento, como un clic de botón u otra llamada API. Cuenta con cuatro argumentos:
-   La instancia de Application Insights (que se puede obtener del enlace `useAppInsightsContext`).
-   El nombre para el evento.
-   El objeto de datos de evento que encapsula los cambios de los que se va a realizar el seguimiento.
-   La marca skipFirstRun (opcional) para omitir la llamada a `trackEvent` en la inicialización. El valor predeterminado está establecido en `true`.

```javascript
import React, { useState, useEffect } from "react";
import { useAppInsightsContext, useTrackEvent } from "@microsoft/applicationinsights-react-js";

const MyComponent = () => {
    const appInsights = useAppInsightsContext();
    const [cart, setCart] = useState([]);
    const trackCheckout = useTrackEvent(appInsights, "Checkout", cart);
    const trackCartUpdate = useTrackEvent(appInsights, "Cart Updated", cart);
    useEffect(() => {
        trackCartUpdate({ cartCount: cart.length });
    }, [cart]);
    
    const performCheckout = () => {
        trackCheckout();
        // submit data
    };
    
    return (
        <div>
            <ul>
                <li>Product 1 <button onClick={() => setCart([...cart, "Product 1"])}>Add to Cart</button></li>
                <li>Product 2 <button onClick={() => setCart([...cart, "Product 2"])}>Add to Cart</button></li>
                <li>Product 3 <button onClick={() => setCart([...cart, "Product 3"])}>Add to Cart</button></li>
                <li>Product 4 <button onClick={() => setCart([...cart, "Product 4"])}>Add to Cart</button></li>
            </ul>
            <button onClick={performCheckout}>Checkout</button>
        </div>
    );
}

export default MyComponent;
```

Cuando se usa el enlace, se puede proporcionar una carga de datos para agregar datos adicionales al evento cuando se almacena en Application Insights.

## <a name="react-error-boundaries"></a>Límites de error de React

Los [límites de errores](https://reactjs.org/docs/error-boundaries.html) proporcionan una manera de controlar correctamente una excepción cuando se produce dentro de una aplicación React y, cuando se produce este error, es probable que se deba registrar la excepción. El complemento de React para Application Insights proporciona un componente de límite de error que registrará automáticamente el error en caso de que se produzca.

```javascript
import React from "react";
import { reactPlugin } from "./AppInsights";
import { AppInsightsErrorBoundary } from "@microsoft/applicationinsights-react-js";

const App = () => {
    return (
        <AppInsightsErrorBoundary onError={() => <h1>I believe something went wrong</h1>} appInsights={reactPlugin}>
            /* app here */
        </AppInsightsErrorBoundary>
    );
};
```

`AppInsightsErrorBoundary` requiere que se le pasen dos propiedades, la instancia de `ReactPlugin` creada para la aplicación y un componente que se representará cuando se produzca un error. Cuando se produce un error no controlado, se llama a `trackException` con la información proporcionada al límite del error y se muestra el componente `onError`.

## <a name="sample-app"></a>Aplicación de ejemplo

Consulte la [demostración de React en Application Insights](https://github.com/Azure-Samples/application-insights-react-demo).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre el SDK de JavaScript, consulte la [documentación del SDK de JavaScript de Application Insights](javascript.md).
- Para obtener información sobre el lenguaje de consulta de Kusto y la consulta de datos en Log Analytics, consulte la [información general sobre la consulta de registro](../../azure-monitor/logs/log-query-overview.md).
