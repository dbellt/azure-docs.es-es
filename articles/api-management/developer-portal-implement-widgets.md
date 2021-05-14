---
title: Implementación de widgets en el portal para desarrolladores
titleSuffix: Azure API Management
description: Obtenga información sobre cómo implementar widgets que consumen datos de API externas y mostrarlo en el portal para desarrolladores de API Management.
author: dlepow
ms.author: apimpm
ms.date: 04/15/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: 4bda136a1abe8f9ffb443973731ebbe13b56ac3b
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741340"
---
# <a name="implement-widgets-in-the-developer-portal"></a>Implementación de widgets en el portal para desarrolladores

En este tutorial, implementará un widget que consume datos de una API externa y lo mostrará en el portal para desarrolladores de API Management.

El widget recuperará las descripciones de sesión de la [API de conferencia](https://conferenceapi.azurewebsites.net/?format=json) de ejemplo. El identificador de sesión se establecerá a través de un editor de widgets designado.

Para ayudarle en el proceso de desarrollo, consulte el widget completado que se encuentra en la carpeta `examples` del [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal/) del portal para desarrolladores de API Management: `/examples/widgets/conference-session`.

:::image type="content" source="media/developer-portal-implement-widgets/widget-published.png" alt-text="Captura de pantalla del widget publicado":::

## <a name="prerequisites"></a>Requisitos previos

* Configure un [entorno local ](developer-portal-self-host.md#step-1-set-up-local-environment) para la versión más reciente del portal para desarrolladores.

* Debe comprender la [anatomía dl widgets de Paperbits](https://paperbits.io/wiki/widget-anatomy).


## <a name="copy-the-scaffold"></a>Copia del scaffold

Use un scaffold `widget` de la carpeta `/scaffolds` como punto de partida para compilar el nuevo widget.

1. Copie la carpeta `/scaffolds/widget` en `/community/widgets`.
1. Cambie el nombre de la carpeta a `conference-session`.

## <a name="rename-exported-module-classes"></a>Cambio del nombre de las clases de módulo exportadas

Cambie el nombre de las clases de módulo exportadas reemplazando el prefijo `Widget` por `ConferenceSession` en estos archivos:

- `widget.design.module.ts`

- `widget.publish.module.ts`

- `widget.runtime.module.ts`
    
Por ejemplo, en el archivo `widget.design.module.ts`, cambie `WidgetDesignModule` a `ConferenceSessionDesignModule`:
    
```typescript
export class WidgetDesignModule implements IInjectorModule {
```
to 
    
```typescript
export class ConferenceSessionDesignModule implements IInjectorModule {
```
    
   
## <a name="register-the-widget"></a>Registro del widget

Para registrar los módulos del widget en los módulos raíz del portal, agregue las siguientes líneas en los archivos correspondientes:

1. `src/apim.design.module.ts`: un módulo que registra las dependencias en tiempo de diseño.

   ```typescript
   import { ConferenceSessionDesignModule } from "../community/widgets/conference-session/widget.design.module";
   
   ...
   injector.bindModule(new ConferenceSessionDesignModule());
   ```
1. `src/apim.publish.module.ts`: un módulo que registra las dependencias en tiempo de publicación.

   ```typescript
   import { ConferenceSessionPublishModule } from "../community/widgets/conference-session/widget.publish.module";

    ...

    injector.bindModule(new ConferenceSessionPublishModule());
    ```

1. `src/apim.runtime.module.ts`: dependencias en tiempo de ejecución.

    ```typescript
    import { ConferenceSessionRuntimeModule } from "../community/widgets/conference-session/widget.runtime.module";

    ...

    injector.bindModule(new ConferenceSessionRuntimeModule());
    ```

## <a name="place-the-widget-in-the-portal"></a>Colocación del widget en el portal

Ahora está listo para conectar el scaffold duplicado y usarlo en el portal para desarrolladores.

1. Ejecute el comando `npm start`.

1. Cuando se cargue la aplicación, coloque el nuevo widget en una página. Puede encontrarlo en el nombre `Your widget` en la categoría `Community` en el selector de widgets.

    :::image type="content" source="media/developer-portal-implement-widgets/widget-selector.png" alt-text="Captura de pantalla del selector de widgets":::

1. Guarde la página presionando **CTRL**+**S** (o **⌘**+**S** en macOS).

    > [!NOTE]
    > En tiempo de diseño, todavía puede interactuar con el sitio web si mantiene presionada la tecla **CTRL** (o **⌘**).

## <a name="add-custom-properties"></a>Adición de propiedades personalizadas

Para que el widget pueda capturar descripciones de sesión, debe tener conocimiento del identificador de sesión. Agregue la propiedad `Session ID` a las interfaces y clases respectivas:

Para que el widget capture la descripción de la sesión, debe tener conocimiento del identificador de sesión. Agregue la propiedad de identificador de sesión a las interfaces y clases respectivas:

1. `widgetContract.ts`: contrato de datos (capa de datos) que define cómo se conserva la configuración del widget.

    ```typescript
    export interface WidgetContract extends Contract {
        sessionNumber: string;
    }
    ```

1. `widgetModel.ts`: modelo (capa de negocio), una representación principal del widget en el sistema. Los editores lo actualizan, y se representa mediante la capa de presentación.

    ```typescript
    export class WidgetModel {
        public sessionNumber: string;
    }
    ```

1. `ko/widgetViewModel.ts`: modelo de vista (capa de presentación), un objeto específico del marco de interfaz de usuario que el portal para desarrolladores representa con la plantilla HTML.

    > [!NOTE]
    > No es necesario cambiar nada en este archivo.

## <a name="configure-binders"></a>Configuración de enlazadores

Habilite el flujo de `sessionNumber` desde el origen de datos a la presentación del widget. Edite las entidades `ModelBinder` y `ViewModelBinder`:

1. `widgetModelBinder.ts` ayuda a preparar el modelo con los datos descritos en el contrato.

    ```typescript
    export class WidgetModelBinder implements IModelBinder<WidgetModel> {
        public async contractToModel(contract: WidgetContract): Promise<WidgetModel> {
            model.sessionNumber = contract.sessionNumber || "107"; // 107 is the default session id
            ...
        }
    
        public modelToContract(model: WidgetModel): Contract {
            const contract: WidgetContract = {
                sessionNumber: model.sessionNumber
                ...
            };
            ...
        }
    }
    ```

1. `ko/widgetViewModelBinder.ts` sabe de qué manera el portal para desarrolladores tiene que presentar el modelo (como un modelo de vista) en un marco de interfaz de usuario específico.

    ```typescript
    ...
    public async updateViewModel(model: WidgetModel, viewModel: WidgetViewModel): Promise<void> {
            viewModel.runtimeConfig(JSON.stringify({
                sessionNumber: model.sessionNumber
            }));
        }
    }
    ...
    ```

## <a name="adjust-design-time-widget-template"></a>Ajuste de la plantilla de widgets en tiempo de diseño

Los componentes de cada ámbito se ejecutan de manera independiente. Tienen contenedores independientes de inserción de dependencias, su propia configuración, ciclo de vida, etc. Incluso pueden usar diferentes marcos de interfaz de usuario (en este ejemplo, es Knockout JS).

Desde la perspectiva del tiempo de diseño, cualquier componente en tiempo de ejecución es simplemente una etiqueta HTML con determinados atributos o contenido. De ser necesario, la configuración se pasa con marcado sin formato. En casos sencillos, como en este ejemplo, el parámetro se pasa en el atributo. Si la configuración es más compleja, podría usar un identificador de los valores necesarios capturados por un proveedor de configuración designado (por ejemplo, `ISettingsProvider`).

1. Actualice el archivo `ko/widgetView.html`:

    ```html
    <widget-runtime data-bind="attr: { params: runtimeConfig }"></widget-runtime>
    ```

    Cuando el portal para desarrolladores ejecuta el enlace `attr` en *tiempo de diseño* o en *tiempo de publicación*, el código HTML resultante es:

    ```html
    <widget-runtime params="{ sessionNumber: 107 }"></widget-runtime>
    ```

    A continuación, en tiempo de ejecución, el componente `widget-runtime` leerá `sessionNumber` y lo usará en el código de inicialización (consulte a continuación).

1. Actualice el archivo `widgetHandlers.ts` para asignar el identificador de sesión en el momento de la creación:

    ```typescript
    ...
    createModel: async () => {
        var model = new ConferenceSessionModel();
        model.sessionNumber = "107";
            return model;
        }
    ...
    ```

## <a name="revise-runtime-view-model"></a>Revisión del modelo de vista en tiempo de ejecución

Los componentes en tiempo de ejecución son el código que se ejecuta en el propio sitio web. Por ejemplo, en el portal para desarrolladores de API Management, son todos los scripts detrás de componentes dinámicos (por ejemplo, *detalles de la API*, *consola de la API*), las operaciones de control, como la generación de ejemplos de código, el envío de solicitudes, etc.

El modelo de vista del componente en tiempo de ejecución tiene que tener los siguientes métodos y propiedades:

- La propiedad `sessionNumber` (marcada con decorador `Param`) usada como parámetro de entrada de componentes pasado desde fuera (el marcado generado en tiempo de diseño; consulte el paso anterior).
- La propiedad `sessionDescription` enlazada a la plantilla de widgets (consulte `widget-runtime.html` más adelante en este artículo).
- El método `initialize` (con el decorador `OnMounted`) invocado después de crear el widget y asignar todos sus parámetros. Es un buen lugar para leer `sessionNumber` e invocar la API mediante `HttpClient`. `HttpClient` una dependencia que inserta el contenedor de IoC (inversión de control).

- En primer lugar, el portal para desarrolladores crea el widget y asigna todos sus parámetros. A continuación, invoca el método `initialize`.

    ```typescript
    ...
    import * as ko from "knockout";
    import { Component, RuntimeComponent, OnMounted, OnDestroyed, Param } from "@paperbits/common/ko/decorators";
    import { HttpClient, HttpRequest } from "@paperbits/common/http";
    ...

    export class WidgetRuntime {
        public readonly sessionDescription: ko.Observable<string>;

        constructor(private readonly httpClient: HttpClient) {
            ...
            this.sessionNumber = ko.observable();
            this.sessionDescription = ko.observable();
            ...
        }

        @Param()
        public readonly sessionNumber: ko.Observable<string>;

        @OnMounted()
        public async initialize(): Promise<void> {
            ...
            const sessionNumber = this.sessionNumber();

            const request: HttpRequest = {
                url: `https://conferenceapi.azurewebsites.net/session/${sessionNumber}`,
                method: "GET"
            };

            const response = await this.httpClient.send<string>(request);
            const sessionDescription = response.toText();
    
            this.sessionDescription(sessionDescription);
            ...
        }
        ...
    }
    ```

## <a name="tweak-the-widget-template"></a>Ajuste de la plantilla de widgets

Actualice el widget para mostrar la descripción de la sesión.

Use una etiqueta de párrafo y un enlace `markdown` (o `text`) en el archivo `ko/runtime/widget-runtime.html` para representar la descripción:

```html
<p data-bind="markdown: sessionDescription"></p>
```

## <a name="add-the-widget-editor"></a>Adición del editor de widgets

El widget ahora está configurado para capturar la descripción de la sesión `107`. Ha especificado `107` en el código como sesión predeterminada. Para comprobar que ha hecho todo correctamente, ejecute `npm start` y confirme que el portal para desarrolladores muestre la descripción en la página.

Ahora, lleve a cabo estos pasos para permitir que el usuario configure el identificador de sesión a través de un editor de widgets:

1. Actualice el archivo `ko/widgetEditorViewModel.ts`:

    ```typescript
    export class WidgetEditor implements WidgetEditor<WidgetModel> {
        public readonly sessionNumber: ko.Observable<string>;

        constructor() {
            this.sessionNumber = ko.observable();
        }

        @Param()
        public model: WidgetModel;

        @Event()
        public onChange: (model: WidgetModel) => void;

        @OnMounted()
        public async initialize(): Promise<void> {
            this.sessionNumber(this.model.sessionNumber);
            this.sessionNumber.subscribe(this.applyChanges);
        }

        private applyChanges(): void {
            this.model.sessionNumber = this.sessionNumber();
            this.onChange(this.model);
        }
    }
    ```

    El modelo de vista del editor usa el mismo enfoque que ha visto anteriormente, pero hay una nueva propiedad `onChange`, decorada con `@Event()`. Comunica la devolución de llamada para notificar a los agentes de escucha (en este caso, un editor de contenido) de los cambios en el modelo.

1. Actualice el archivo `ko/widgetEditorView.html`:

    ```html
    <input type="text" class="form-control" data-bind="textInput: sessionNumber" />
    ```

1. Vuelva a ejecutar `npm start`. Debería poder cambiar `sessionNumber` en el editor de widgets. Cambie el identificador a `108`, guarde los cambios y actualice la pestaña del explorador. Si tiene problemas, es posible que tenga que volver a agregar el widget a la página.

    :::image type="content" source="media/developer-portal-implement-widgets/widget-editor.png" alt-text="Captura de pantalla del editor de widgets":::

## <a name="rename-the-widget"></a>Cambio de nombre del widget

Cambie el nombre del widget en el archivo `constants.ts`:

```typescript
...
export const widgetName = "conference-session";
export const widgetDisplayName = "Conference session";
...
```

> [!NOTE]
> Si va a contribuir con el widget al repositorio, `widgetName` tiene que ser igual que su nombre de carpeta y debe derivarse del nombre para mostrar (en minúsculas y los espacios reemplazados por guiones). La categoría debe permanecer en `Community`.

## <a name="next-steps"></a>Pasos siguientes


Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)

- [Contribución de widgets](developer-portal-widget-contribution-guidelines.md): agradecemos y animamos las contribuciones de la comunidad.

- Consulte [Uso de widgets de la comunidad](developer-portal-use-community-widgets.md) para aprender a usar widgets aportados por la comunidad.
