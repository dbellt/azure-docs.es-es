---
title: Complemento Angular para el SDK de JavaScript de Application Insights
description: Instalación y uso del complemento Angular para el SDK de JavaScript de Application Insights.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: lagayhar
ms.openlocfilehash: 7ac83d0c43026b431370fab1d8c49aec1adf6659
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107312729"
---
# <a name="angular-plugin-for-application-insights-javascript-sdk"></a>Complemento Angular para el SDK de JavaScript de Application Insights

El complemento Angular para el SDK de JavaScript de Application Insights habilita lo siguiente:

- Seguimiento de cambios de enrutador
- Seguimiento de excepciones no detectadas

> [!WARNING]
> El complemento Angular NO es compatible con ECMAScript 3 (ES3).

## <a name="getting-started"></a>Introducción

Instalación del paquete de npm:

```bash
npm install @microsoft/applicationinsights-angularplugin-js
```

## <a name="basic-usage"></a>Uso básico

Configure una instancia de Application Insights en el componente de entrada de la aplicación:

```js
import { Component } from '@angular/core';
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { AngularPlugin } from '@microsoft/applicationinsights-angularplugin-js';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
    constructor(
        private router: Router
    ){
        var angularPlugin = new AngularPlugin();
        const appInsights = new ApplicationInsights({ config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [angularPlugin],
        extensionConfig: {
            [angularPlugin.identifier]: { router: this.router }
        }
        } });
        appInsights.loadAppInsights();
    }
}
```

Para realizar un seguimiento de las excepciones no detectadas, configure ApplicationinsightsAngularpluginErrorService en `app.module.ts`:

```js
import { ApplicationinsightsAngularpluginErrorService } from '@microsoft/applicationinsights-angularplugin-js';

@NgModule({
  ...
  providers: [
    {
      provide: ErrorHandler,
      useClass: ApplicationinsightsAngularpluginErrorService
    }
  ]
  ...
})
export class AppModule { }
```

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre el SDK de JavaScript, consulte la [documentación del SDK de JavaScript de Application Insights](javascript.md).
- [Complemento Angular en GitHub](https://github.com/microsoft/ApplicationInsights-JS/tree/master/extensions/applicationinsights-angularplugin-js)
