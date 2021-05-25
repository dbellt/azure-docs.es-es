---
title: Realización de operaciones en datos
description: Conversión, administración y manipulación de salidas y formatos de datos en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: article
ms.date: 05/11/2021
ms.openlocfilehash: cc4952acd8d5949485b9bd1fe5fac91296839493
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753658"
---
# <a name="perform-data-operations-in-azure-logic-apps"></a>Realización de operaciones de datos en Azure Logic Apps

En este artículo se muestra, entre otras cosas, cómo puede trabajar con datos en aplicaciones lógicas mediante la adición de acciones para estas tareas:

* Crear tablas a partir de matrices.
* Crear matrices a partir de otras matrices según una condición.
* Crear tokens fáciles de usar a partir de las propiedades de objetos de notación de objetos JavaScript (JSON) para que pueda usar fácilmente esas propiedades en el flujo de trabajo.

Si aquí no encuentra la acción que desea, pruebe a examinar las diversas [funciones de manipulación de datos](../logic-apps/workflow-definition-language-functions-reference.md) que proporciona Azure Logic Apps.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* La aplicación lógica donde necesita la operación para trabajar con los datos.

  Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) que será el primer paso de la aplicación lógica. 

  Las operaciones de datos están disponibles solo como acciones, así que antes de usar estas acciones, inicie la aplicación lógica con un desencadenador e incluya otras acciones necesarias para crear las salidas deseadas.

## <a name="data-operation-actions"></a>Acciones de operación de datos

En estas tablas se resumen las operaciones de datos que puede usar, organizadas según los tipos de datos de origen sobre los que trabajan las operaciones, pero cada descripción aparece en orden alfabético.

### <a name="array-actions"></a>Acciones de matriz

Estas acciones permiten trabajar con datos de matrices.

| Acción | Descripción |
|--------|-------------|
| [**Crear tabla CSV**](#create-csv-table-action) | Crea una tabla de valores separados por comas (CSV) a partir de una matriz. |
| [**Crear tabla HTML**](#create-html-table-action) | Crea una tabla HTML a partir de una matriz. |
| [**Filtrar matriz**](#filter-array-action) | Crea un subconjunto de matriz a partir de una matriz en función del filtro o condición especificados. |
| [**Combinar**](#join-action) | Crea una cadena a partir de todos los elementos de una matriz y separa cada elemento con el carácter especificado. |
| [**Seleccionar**](#select-action) | Crea una matriz a partir de las propiedades especificadas para todos los elementos de una matriz distinta. |
||| 

### <a name="json-actions"></a>Acciones JSON

Estas acciones le ayudan a trabajar con datos en formato de notación de objetos JavaScript (JSON).

| Acción | Descripción |
|--------|-------------|
| [**Redactar**](#compose-action) | Crea un mensaje o cadena, a partir de varias entradas que pueden tener varios tipos de datos. Luego puede usar esta cadena como una única entrada, en lugar de especificar varias veces las mismas entradas. Por ejemplo, puede crear un único mensaje JSON a partir de varias entradas. |
| [**Análisis del archivo JSON**](#parse-json-action) | Crea tokens de datos fáciles de usar para propiedades de contenido JSON para que pueda usar las propiedades más fácilmente en las aplicaciones lógicas. |
|||

Para crear transformaciones JSON más complejas, consulte [Perform advanced JSON transformations with Liquid templates](../logic-apps/logic-apps-enterprise-integration-liquid-transform.md) (Realización de transformaciones JSON avanzadas con plantillas de Liquid).

<a name="compose-action"></a>

## <a name="compose-action"></a>Acción Compose

Para crear una salida única, como un objeto JSON de varias entradas, se puede usar la acción **Redactar**. Las entradas pueden tener varios tipos, como enteros, booleanos, matrices, objetos JSON y cualquier otro tipo nativo que admita Azure Logic Apps, por ejemplo, binario y XML. Luego puede usar la salida en las acciones que siguen a la acción **Redactar**. La acción **Redactar** también puede evitarle tener que escribir una y otra vez las mismas entradas mientras compila el flujo de trabajo de la aplicación lógica.

Por ejemplo, puede construir un mensaje JSON a partir de varias variables, como variables de cadena que almacenan los nombres y apellidos de la gente, y una variable de entero que almacena las edades de la gente. En este caso, la acción **Redactar** acepta estas entradas:

`{ "age": <ageVar>, "fullName": "<lastNameVar>, <firstNameVar>" }`

y crea esta salida:

`{"age":35,"fullName":"Owens,Sophie"}`

Para probar un ejemplo, siga estos pasos con el Diseñador de aplicación lógica. O bien, si prefiere trabajar en el editor de vista de código, puede copiar las definiciones de acción de ejemplo **Redactar** e **Inicializar variable** de este artículo en su propia definición de flujo de trabajo subyacente de la aplicación lógica: [Ejemplos de código de operación de datos: Redactar](../logic-apps/logic-apps-data-operations-code-samples.md#compose-action-example) 

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas.

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador **Periodicidad** y varias acciones **Inicializar variable**. Estas acciones se configuran para crear dos variables de cadena y una variable de entero. Cuando más tarde pruebe la aplicación lógica, puede ejecutar manualmente la aplicación sin esperar a que se active el desencadenador.

   ![Inicio de la aplicación lógica de ejemplo para la acción "Redactar"](./media/logic-apps-perform-data-operations/sample-starting-logic-app-compose-action.png)

1. En la aplicación lógica donde desea crear la salida, siga uno de estos pasos: 

   * Para agregar una acción en el último paso, elija **Nuevo paso**.

     ![Selección de "Nuevo paso" para la acción "Redactar"](./media/logic-apps-perform-data-operations/add-compose-operation-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el puntero sobre la flecha de conexión para que el signo más ( **+** ) aparezca. Haga clic en el signo más y, a continuación, seleccione **Agregar una acción**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `compose` como filtro. En la lista de acciones, seleccione la acción **Redactar**.

   ![Seleccionar la acción "Redactar"](./media/logic-apps-perform-data-operations/select-compose-action.png)

1. En el cuadro **Entradas**, proporcione las entradas que desee para crear la salida.

   En este ejemplo, al hacer clic en el cuadro **Entradas**, aparece la lista de contenido dinámico, por lo que puede seleccionar las variables creadas anteriormente:

   ![Selección de entradas para usar en la acción "Redactar"](./media/logic-apps-perform-data-operations/configure-compose-action.png)

   Esta es la acción **Redactar** de ejemplo finalizada: 

   ![Ejemplo finalizado para la acción "Redactar"](./media/logic-apps-perform-data-operations/finished-compose-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Para más información sobre esta acción en la definición del flujo de trabajo subyacente, consulte la [acción Redactar](../logic-apps/logic-apps-workflow-actions-triggers.md#compose-action).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para confirmar si la acción **Redactar** crea los resultados esperados, envíese una notificación que incluya la salida de esta acción.

1. En la aplicación lógica, agregue una acción que pueda enviarle los resultados de la acción **Redactar**.

1. En dicha acción, haga clic en cualquier lugar donde quiere que aparezcan los resultados. Cuando se abra la lista de contenido dinámico, en la acción **Redactar**, seleccione **Salida**.

   En este ejemplo se usa la acción **Enviar un correo electrónico** y se incluyen los campos de **Salida** en el cuerpo y el asunto del correo electrónico:

   ![Campos "Salida" para la acción "Redactar"](./media/logic-apps-perform-data-operations/send-email-compose-action.png)

1. Ahora, ejecute la aplicación lógica manualmente. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar).

   Según el conector de correo electrónico que ha usado, estos son los resultados que obtiene:

   ![Resultados de enviar correo electrónico con la acción "Redactar"](./media/logic-apps-perform-data-operations/compose-email-results.png)

<a name="create-csv-table-action"></a>

## <a name="create-csv-table-action"></a>Acción Crear tabla CSV

Para crear una tabla de valores separados por comas (CSV) que tenga las propiedades y los valores de objetos de notación de objetos JavaScript (JSON) en una matriz, use la acción **Crear tabla CSV**. Luego puede usar la tabla resultante en las acciones que siguen a la acción **Crear tabla CSV**.

Si prefiere trabajar en el editor de vista de código, puede copiar las definiciones de acción de ejemplo **Crear tabla CSV** e **Inicializar variable** de este artículo en su propia definición de flujo de trabajo subyacente de la aplicación lógica: [Ejemplos de código de operación de datos: Crear tabla CSV](../logic-apps/logic-apps-data-operations-code-samples.md#create-csv-table-action-example)

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas.

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador **Periodicidad** y una acción **Inicializar variable**. La acción está configurada para la creación de una variable cuyo valor inicial es una matriz que tiene algunas propiedades y valores en formato JSON. Cuando más tarde pruebe la aplicación lógica, puede ejecutar manualmente la aplicación sin esperar a que se active el desencadenador.

   ![Inicio de la aplicación lógica de ejemplo para la acción "Crear tabla CSV"](./media/logic-apps-perform-data-operations/sample-starting-logic-app-create-table-action.png)

1. En la aplicación lógica donde quiere crear la tabla CSV, siga uno de estos pasos: 

   * Para agregar una acción en el último paso, elija **Nuevo paso**.

     ![Selección de "Nuevo paso" para la acción "Crear tabla CSV"](./media/logic-apps-perform-data-operations/add-create-table-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el puntero sobre la flecha de conexión para que el signo más ( **+** ) aparezca. Haga clic en el signo más y, a continuación, seleccione **Agregar una acción**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `create csv table` como filtro. En la lista de acciones, seleccione la acción **Crear tabla CSV**.

   ![Seleccionar la acción "Crear tabla CSV"](./media/logic-apps-perform-data-operations/select-create-csv-table-action.png)

1. En el cuadro **Desde**, proporcione la matriz o expresión que desea para crear la tabla.

   En este ejemplo, al hacer clic dentro del cuadro **Desde**, aparece la lista de contenido dinámico, por lo que puede seleccionar la variable creada anteriormente:

   ![Seleccionar la salida de matriz para crear la tabla CSV](./media/logic-apps-perform-data-operations/configure-create-csv-table-action.png)

   > [!TIP]
   > Para crear tokens fáciles de usar para las propiedades de objetos JSON de modo que pueda seleccionar esas propiedades como entradas, use la acción [Análisis del archivo JSON](#parse-json-action) antes de llamar a la acción **Crear tabla CSV**.

   Esta es la acción **Crear tabla CSV** de ejemplo finalizada: 

   ![Ejemplo finalizado de la acción "Crear tabla CSV"](./media/logic-apps-perform-data-operations/finished-create-csv-table-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

### <a name="customize-table-format"></a>Personalización del formato de tabla

De forma predeterminada, la propiedad **Columnas** está configurada para crear automáticamente las columnas de la tabla en función de los elementos de la matriz. Para especificar los valores y los encabezados personalizados, siga estos pasos:

1. Abra la lista **Columnas** y seleccione **Personalizar**.

1. En la propiedad **Encabezado**, especifique el texto de encabezado personalizado que se va a usar en su lugar.

1. En la propiedad **Value**, especifique el valor personalizado que se va a usar en su lugar.

Para devolver valores de la matriz, puede usar la función [`item()`](../logic-apps/workflow-definition-language-functions-reference.md#item) con la acción **Crear tabla CSV**. En un bucle `For_each`, puede usar la función [`items()`](../logic-apps/workflow-definition-language-functions-reference.md#items).

Por ejemplo, supongamos que desea columnas de tabla que solo tengan los valores de propiedad y no los nombres de propiedad de una matriz. Para devolver solo estos valores, siga estos pasos para trabajar en la vista de diseñador o en la vista de código. Este es el resultado que devuelve este ejemplo:

```text
Apples,1
Oranges,2
```

#### <a name="work-in-designer-view"></a>Trabajar en la Vista de diseñador

En la acción, mantenga la columna **Header** vacía. En cada fila de la columna **Valor**, desreferenciar cada propiedad de matriz que desee. Cada fila debajo de **Valor** devuelve todos los valores para la propiedad de matriz especificada y se convierte en una columna en su tabla.

1. En **Valor**, en cada fila que desee, haga clic dentro del cuadro de edición para que aparezca la lista de contenido dinámico.

1. En la lista de contenido dinámico, seleccione **Expresión**.

1. En el editor de expresiones, ingrese esta expresión que especifica el valor de propiedad de la matriz que desea y seleccione **Aceptar**.

   `item()?['<array-property-name>']`

   Por ejemplo:

   * `item()?['Description']`
   * `item()?['Product_ID']`

   ![Desreferenciar "Descripción" para "Crear tabla CSV"](./media/logic-apps-perform-data-operations/csv-table-expression.png)

1. Repita los pasos anteriores para cada propiedad de matriz que desee. Una vez que haya terminado, la acción debe ser parecida a la de este ejemplo:

   ![Función "item ()" en "Crear tabla CSV"](./media/logic-apps-perform-data-operations/finished-csv-expression.png)

1. Para resolver expresiones en versiones más descriptivas, cambie a la vista de código y vuelva a la vista de diseñador, y luego vuelva a abrir la acción contraída:

   La acción **Crear tabla CSV** ahora aparece como este ejemplo:

   !["Crear tabla CSV": expresiones resueltas, sin encabezados](./media/logic-apps-perform-data-operations/resolved-csv-expression.png)

#### <a name="work-in-code-view"></a>Trabajar en la Vista código

En la definición JSON de la acción, dentro de la matriz `columns`, establezca la propiedad `header` en una cadena vacía. Para cada propiedad `value`, desreferenciar cada propiedad de matriz que desee.

1. En la barra de herramientas del diseñador, seleccione **vista Código**.

1. En el editor de código, en la matriz `columns` de la acción, agregue la propiedad `header` vacía y esta expresión `value` para cada columna de valores de matriz que desee:

   ```json
   {
      "header": "",
      "value": "@item()?['<array-property-name>']"
   }
   ```

   Por ejemplo:

   ```json
   "Create_CSV_table": {
      "inputs": {
         "columns": [
            { 
               "header": "",
               "value": "@item()?['Description']"
            },
            { 
               "header": "",
               "value": "@item()?['Product_ID']"
            }
         ],
         "format": "CSV",
         "from": "@variables('myJSONArray')"
      }
   }
   ```

1. Vuelva a la vista de diseñador y vuelva a abrir la acción contraída.

   La acción **Crear tabla CVS** ahora aparece como en este ejemplo y las expresiones se han resuelto como versiones más descriptivas:

   !["Crear tabla CSV": expresiones resueltas y sin encabezados](./media/logic-apps-perform-data-operations/resolved-csv-expression.png)

Para más información sobre esta acción en la definición del flujo de trabajo subyacente, consulte la [acción Tabla](../logic-apps/logic-apps-workflow-actions-triggers.md#table-action).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para confirmar si la acción **Crear tabla CSV** crea los resultados esperados, envíese una notificación que incluya la salida de esta acción.

1. En la aplicación lógica, agregue una acción que pueda enviarle los resultados de la acción **Crear tabla CSV**.

1. En dicha acción, haga clic en cualquier lugar donde quiere que aparezcan los resultados. Cuando se abra la lista de contenido dinámico, en la acción **Crear tabla CSV**, seleccione **Salida**. 

   En este ejemplo se usa la acción de Office 365 Outlook **Enviar un correo electrónico** y se incluyen los campos de **Salida** en el cuerpo del correo electrónico:

   ![Campos "Salida" para la acción "Crear tabla CSV"](./media/logic-apps-perform-data-operations/send-email-create-csv-table-action.png)

   > [!NOTE]
    > Si la tabla se devuelve con formato incorrecto, consulte [cómo comprobar el formato de los datos de la tabla](#format-table-data).

1. Ahora, ejecute la aplicación lógica manualmente. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar).

   Según el conector de correo electrónico que ha usado, estos son los resultados que obtiene:

   ![Resultados de enviar por correo electrónico con la acción "Crear tabla CSV"](./media/logic-apps-perform-data-operations/create-csv-table-email-results.png)

<a name="create-html-table-action"></a>

## <a name="create-html-table-action"></a>Acción Crear tabla HTML

Para crear una tabla HTML que tenga las propiedades y los valores de objetos de notación de objetos JavaScript (JSON) en una matriz, use la acción **Crear tabla HTML**. Luego puede usar la tabla resultante en las acciones que siguen a la acción **Crear tabla HTML**.

Si prefiere trabajar en el editor de vista de código, puede copiar las definiciones de acción de ejemplo **Crear tabla HTML** e **Inicializar variable** de este artículo en su propia definición de flujo de trabajo subyacente de la aplicación lógica: [Ejemplos de código de operación de datos: Crear tabla HTML](../logic-apps/logic-apps-data-operations-code-samples.md#create-html-table-action-example) 

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas.

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador **Periodicidad** y una acción **Inicializar variable**. La acción está configurada para la creación de una variable cuyo valor inicial es una matriz que tiene algunas propiedades y valores en formato JSON. Cuando más tarde pruebe la aplicación lógica, puede ejecutar manualmente la aplicación sin esperar a que se active el desencadenador.

   ![Inicio de la aplicación lógica de ejemplo para "Crear tabla HTML"](./media/logic-apps-perform-data-operations/sample-starting-logic-app-create-table-action.png)

1. En la aplicación lógica donde quiere crear la tabla HTML, siga uno de estos pasos:

   * Para agregar una acción en el último paso, elija **Nuevo paso**.

     ![Selección de "Nuevo paso" para la acción "Crear tabla HTML"](./media/logic-apps-perform-data-operations/add-create-table-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el puntero sobre la flecha de conexión para que el signo más ( **+** ) aparezca. Haga clic en el signo más y, a continuación, seleccione **Agregar una acción**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `create html table` como filtro. En la lista de acciones, seleccione la acción **Crear tabla HTML**.

   ![Seleccionar la acción "Crear tabla HTML"](./media/logic-apps-perform-data-operations/select-create-html-table-action.png)

1. En el cuadro **Desde**, proporcione la matriz o expresión que desea para crear la tabla.

   En este ejemplo, al hacer clic dentro del cuadro **Desde**, aparece la lista de contenido dinámico, por lo que puede seleccionar la variable creada anteriormente:

   ![Seleccionar la salida de matriz para crear la tabla HTML](./media/logic-apps-perform-data-operations/configure-create-html-table-action.png)

   > [!TIP]
   > Para crear tokens fáciles de usar para las propiedades de objetos JSON de modo que pueda seleccionar esas propiedades como entradas, use la acción [Análisis del archivo JSON](#parse-json-action) antes de llamar a la acción **Crear tabla HTML**.

   Esta es la acción **Crear tabla HTML** de ejemplo finalizada:

   ![Ejemplo finalizado para "Crear tabla HTML"](./media/logic-apps-perform-data-operations/finished-create-html-table-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

### <a name="customize-table-format"></a>Personalización del formato de tabla

De forma predeterminada, la propiedad **Columnas** está configurada para crear automáticamente las columnas de la tabla en función de los elementos de la matriz. Para especificar los valores y los encabezados personalizados, siga estos pasos:

1. Abra la lista **Columnas** y seleccione **Personalizar**.

1. En la propiedad **Encabezado**, especifique el texto de encabezado personalizado que se va a usar en su lugar.

1. En la propiedad **Value**, especifique el valor personalizado que se va a usar en su lugar.

Para devolver valores de la matriz, puede usar la función [`item()`](../logic-apps/workflow-definition-language-functions-reference.md#item) con la acción **Crear tabla HTML**. En un bucle `For_each`, puede usar la función [`items()`](../logic-apps/workflow-definition-language-functions-reference.md#items).

Por ejemplo, supongamos que desea columnas de tabla que solo tengan los valores de propiedad y no los nombres de propiedad de una matriz. Para devolver solo estos valores, siga estos pasos para trabajar en la vista de diseñador o en la vista de código. Este es el resultado que devuelve este ejemplo:

```text
Apples,1
Oranges,2
```

#### <a name="work-in-designer-view"></a>Trabajar en la Vista de diseñador

En la acción, mantenga la columna **Header** vacía. En cada fila de la columna **Valor**, desreferenciar cada propiedad de matriz que desee. Cada fila debajo de **Valor** devuelve todos los valores para la propiedad especificada y se convierte en una columna en su tabla.

1. En **Valor**, en cada fila que desee, haga clic dentro del cuadro de edición para que aparezca la lista de contenido dinámico.

1. En la lista de contenido dinámico, seleccione **Expresión**.

1. En el editor de expresiones, ingrese esta expresión que especifica el valor de propiedad de la matriz que desea y seleccione **Aceptar**.

   `item()?['<array-property-name>']`

   Por ejemplo:

   * `item()?['Description']`
   * `item()?['Product_ID']`

   ![Desreferenciar propiedad en la acción "Crear tabla HTML"](./media/logic-apps-perform-data-operations/html-table-expression.png)

1. Repita los pasos anteriores para cada propiedad de matriz que desee. Una vez que haya terminado, la acción debe ser parecida a la de este ejemplo:

   ![Función "item ()" en "Crear tabla HTML"](./media/logic-apps-perform-data-operations/finished-html-expression.png)

1. Para resolver expresiones en versiones más descriptivas, cambie a la vista de código y vuelva a la vista de diseñador, y luego vuelva a abrir la acción contraída:

   La acción **Crear tabla HTML** ahora aparece como este ejemplo:

   !["Crear tabla HTML": expresiones resueltas, sin encabezados](./media/logic-apps-perform-data-operations/resolved-html-expression.png)

#### <a name="work-in-code-view"></a>Trabajar en la Vista código

En la definición JSON de la acción, dentro de la matriz `columns`, establezca la propiedad `header` en una cadena vacía. Para cada propiedad `value`, desreferenciar cada propiedad de matriz que desee.

1. En la barra de herramientas del diseñador, seleccione **vista Código**.

1. En el editor de código, en la matriz `columns` de la acción, agregue la propiedad `header` vacía y esta expresión `value` para cada columna de valores de matriz que desee:

   ```json
   {
      "header": "",
      "value": "@item()?['<array-property-name>']"
   }
   ```

   Por ejemplo:

   ```json
   "Create_HTML_table": {
      "inputs": {
         "columns": [
            { 
               "header": "",
               "value": "@item()?['Description']"
            },
            { 
               "header": "",
               "value": "@item()?['Product_ID']"
            }
         ],
         "format": "HTML",
         "from": "@variables('myJSONArray')"
      }
   }
   ```

1. Vuelva a la vista de diseñador y vuelva a abrir la acción contraída.

   La acción **Crear tabla HTML** ahora aparece como en este ejemplo y las expresiones se han resuelto como versiones más descriptivas:

   !["Crear tabla HTML": expresiones resueltas y sin encabezados](./media/logic-apps-perform-data-operations/resolved-html-expression.png)

Para más información sobre esta acción en la definición del flujo de trabajo subyacente, consulte la [acción Tabla](../logic-apps/logic-apps-workflow-actions-triggers.md#table-action).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para confirmar si la acción **Crear tabla HTML** crea los resultados esperados, envíese una notificación que incluya la salida de esta acción.

1. En la aplicación lógica, agregue una acción que pueda enviarle los resultados de la acción **Crear tabla HTML**.

1. En dicha acción, haga clic en cualquier lugar donde quiere que aparezcan los resultados. Cuando se abra la lista de contenido dinámico, en la acción **Crear tabla HTML**, seleccione **Salida**. 

   En este ejemplo se usa la acción de Office 365 Outlook **Enviar un correo electrónico** y se incluyen los campos de **Salida** en el cuerpo del correo electrónico:

   ![Campos "Salida" para "Crear tabla HTML"](./media/logic-apps-perform-data-operations/send-email-create-html-table-action.png)

   > [!NOTE]
   > Al incluir la salida de la tabla HTML en una acción de correo electrónico, asegúrese de establecer la propiedad **Es HTML** en **Sí** en las opciones avanzadas de la acción de correo electrónico. De este modo, la acción de correo electrónico aplica el formato correcto a la tabla HTML.

   > [!NOTE]
   > Si la tabla se devuelve con formato incorrecto, consulte [cómo comprobar el formato de los datos de la tabla](#format-table-data).

1. Ahora, ejecute la aplicación lógica manualmente. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar).

   Según el conector de correo electrónico que ha usado, estos son los resultados que obtiene:

   ![Correo electrónico con los resultados de "Crear tabla HTML"](./media/logic-apps-perform-data-operations/create-html-table-email-results.png)

<a name="filter-array-action"></a>

## <a name="filter-array-action"></a>Acción Filtrar matriz

Para crear una matriz más pequeña que contenga elementos que satisfagan determinados criterios, en una matriz existente, use la acción **Filtrar matriz**. Luego puede usar la matriz filtrada en las acciones que siguen a la acción **Filtrar matriz**.

> [!NOTE]
> Cualquier texto de filtro que use en la condición distingue mayúsculas de minúsculas. Además, esta acción no puede cambiar el formato o los componentes de los elementos de la matriz. 
> 
> Para que las acciones usen la salida de matriz de la acción **Filtrar matriz**, esas acciones deben aceptar matrices como entrada, o puede que tenga que transformar la matriz de salida en otro formato compatible.
> 
> Si llama a un punto de conexión HTTP y recibe una respuesta JSON, use la acción **analizar JSON** para procesar la respuesta JSON. 
> De lo contrario, la acción del **filtro de la matriz** solo puede leer el cuerpo de la respuesta y no la estructura de la carga JSON.

Si prefiere trabajar en el editor de vista de código, puede copiar las definiciones de acción de ejemplo **Filtrar matriz** e **Inicializar variable** de este artículo en su propia definición de flujo de trabajo subyacente de la aplicación lógica: [Ejemplos de código de operación de datos: Filtrar matriz](../logic-apps/logic-apps-data-operations-code-samples.md#filter-array-action-example)

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas.

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador **Periodicidad** y una acción **Inicializar variable**. La acción está configurada para la creación de una variable cuyo valor inicial es una matriz que contiene algunos enteros de ejemplo. Cuando más tarde pruebe la aplicación lógica, puede ejecutar manualmente la aplicación sin esperar a que se active el desencadenador.

   > [!NOTE]
   > Aunque en este ejemplo se usa una matriz de enteros sencilla, esta acción es especialmente útil para matrices de objetos JSON, donde puede filtrar según las propiedades y los valores de los objetos.

   ![Inicio de la aplicación lógica de ejemplo para la acción "Filtrar matriz"](./media/logic-apps-perform-data-operations/sample-starting-logic-app-filter-array-action.png)

1. En la aplicación lógica donde quiere crear la matriz filtrada, siga uno de estos pasos: 

   * Para agregar una acción en el último paso, elija **Nuevo paso**.

     ![Selección de "Nuevo paso" para la acción "Filtrar matriz"](./media/logic-apps-perform-data-operations/add-filter-array-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el puntero sobre la flecha de conexión para que el signo más ( **+** ) aparezca. Haga clic en el signo más y, a continuación, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba `filter array` como filtro. En la lista de acciones, seleccione la acción **Filtrar matriz**.

   ![Seleccionar la acción "Filtrar matriz"](./media/logic-apps-perform-data-operations/select-filter-array-action.png)

1. En el cuadro **Desde**, proporcione la matriz o expresión que desea filtrar.

   En este ejemplo, al hacer clic dentro del cuadro **Desde**, aparece la lista de contenido dinámico, por lo que puede seleccionar la variable creada anteriormente:

   ![Seleccionar la salida de matriz para crear la matriz filtrada](./media/logic-apps-perform-data-operations/configure-filter-array-action.png)

1. Para la condición, especifique los elementos de matriz para comparar, seleccione el operador de comparación y especifique el valor de comparación.

   En este ejemplo se usa la función `item()` para acceder a cada elemento de la matriz mientras la acción **Filtrar matriz** busca los elementos de matriz cuyo valor es superior a 1:

   ![Ejemplo terminado para la acción "Filtrar matriz"](./media/logic-apps-perform-data-operations/finished-filter-array-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Para más información sobre esta acción en la definición del flujo de trabajo subyacente, consulte la [acción Consulta](../logic-apps/logic-apps-workflow-actions-triggers.md#query-action).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para confirmar si la acción **Filtrar matriz** crea los resultados esperados, envíese una notificación que incluya la salida de esta acción.

1. En la aplicación lógica, agregue una acción que pueda enviarle los resultados de la acción **Filtrar matriz**.

1. En dicha acción, haga clic en cualquier lugar donde quiere que aparezcan los resultados. Cuando se abra la lista de contenido dinámico, seleccione **Expresión**. Para obtener la salida de matriz de la acción **Filtrar matriz**, escriba esta expresión que incluye el nombre de dicha acción:

   `@actionBody('Filter_array')`

   En este ejemplo se usa la acción de Office 365 **Enviar un correo electrónico** y se incluyen las salidas de la expresión **actionBody('Filter_array')** en el cuerpo del correo electrónico:

   ![Salidas de acción de la acción "Filtrar matriz"](./media/logic-apps-perform-data-operations/send-email-filter-array-action.png)

1. Ahora, ejecute la aplicación lógica manualmente. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar).

   Según el conector de correo electrónico que ha usado, estos son los resultados que obtiene:

   ![Resultados de enviar por correo electrónico con la acción "Filtrar matriz"](./media/logic-apps-perform-data-operations/filter-array-email-results.png)

<a name="join-action"></a>

## <a name="join-action"></a>Acción Combinar

Para crear una cadena que contenga todos los elementos de una matriz y separe esos elementos con un carácter delimitador específico, use la acción **Combinar**. Luego puede usar la cadena en las acciones que siguen a la acción **Combinar**.

Si prefiere trabajar en el editor de vista de código, puede copiar las definiciones de acción de ejemplo **Unir** e **Inicializar variable** de este artículo en su propia definición de flujo de trabajo subyacente de la aplicación lógica: [Ejemplos de código de operación de datos: Unir](../logic-apps/logic-apps-data-operations-code-samples.md#join-action-example)

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas.

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador **Periodicidad** y una acción **Inicializar variable**. La acción está configurada para la creación de una variable cuyo valor inicial es una matriz que contiene algunos enteros de ejemplo. Cuando más tarde pruebe la aplicación lógica, puede ejecutar manualmente la aplicación sin esperar a que se active el desencadenador.

   ![Inicio de la aplicación lógica de ejemplo para la acción "Unirse"](./media/logic-apps-perform-data-operations/sample-starting-logic-app-join-action.png)

1. En la aplicación lógica donde desea crear la cadena desde una matriz, siga uno de estos pasos:

   * Para agregar una acción en el último paso, elija **Nuevo paso**.

     ![Selección de "Nuevo paso" para la acción "Unirse"](./media/logic-apps-perform-data-operations/new-step-add-join-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el puntero sobre la flecha de conexión para que el signo más ( **+** ) aparezca. Haga clic en el signo más y, a continuación, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba `join` como filtro. En la lista de acciones, seleccione esta acción: **Join**

   ![Selección de la acción "Combinar"](./media/logic-apps-perform-data-operations/select-join-operation-action.png)

1. En el cuadro **Desde**, proporcione la matriz que contiene los elementos que quiere combinar como una cadena.

   En este ejemplo, al hacer clic dentro del cuadro **Desde**, aparece la lista de contenido dinámico, por lo que puede seleccionar la variable creada anteriormente:  

   ![Seleccionar la salida de matriz para crear la cadena](./media/logic-apps-perform-data-operations/configure-join-action.png)

1. En el cuadro **Unir con**, escriba el carácter deseado para separar cada elemento de matriz. 

   En este ejemplo se usan los dos puntos (:) como separador.

   ![Proporcionar el carácter separador](./media/logic-apps-perform-data-operations/finished-join-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Para más información sobre esta acción en la definición del flujo de trabajo subyacente, consulte la [acción Combinar](../logic-apps/logic-apps-workflow-actions-triggers.md#join-action).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para confirmar si la acción **Combinar** crea los resultados esperados, envíese una notificación que incluya la salida de esta acción.

1. En la aplicación lógica, agregue una acción que pueda enviarle los resultados de la acción **Combinar**.

1. En dicha acción, haga clic en cualquier lugar donde quiere que aparezcan los resultados. Cuando se abra la lista de contenido dinámico, en la acción **Combinar**, seleccione **Salida**. 

   En este ejemplo se usa la acción de Office 365 Outlook **Enviar un correo electrónico** y se incluyen los campos de **Salida** en el cuerpo del correo electrónico:

   ![Campos "Salida" para la acción "Unirse"](./media/logic-apps-perform-data-operations/send-email-join-action.png)

1. Ahora, ejecute la aplicación lógica manualmente. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar).

   Según el conector de correo electrónico que ha usado, estos son los resultados que obtiene:

   ![Resultados de enviar correo electrónico con la acción "Combinar"](./media/logic-apps-perform-data-operations/join-send-email-results.png)

<a name="parse-json-action"></a>

## <a name="parse-json-action"></a>Acción Análisis del archivo JSON

Para hacer referencia a las propiedades en contenido de notación de objetos JavaScript (JSON) o acceder a ellas, puede crear campos o tokens fáciles de usar para esas propiedades mediante la acción **Análisis del archivo JSON**. De este modo, puede seleccionar esas propiedades de la lista de contenido dinámico al especificar entradas para la aplicación lógica. Para esta acción, puede proporcionar un esquema JSON o generar uno a partir de la carga o el contenido JSON de ejemplo.

Si prefiere trabajar en el editor de vista de código, puede copiar las definiciones de acción de ejemplo **Analizar JSON** e **Inicializar variable** de este artículo en su propia definición de flujo de trabajo subyacente de la aplicación lógica: [Ejemplos de código de operación de datos: Analizar JSON](../logic-apps/logic-apps-data-operations-code-samples.md#parse-json-action-example)

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas.

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador **Periodicidad** y una acción **Inicializar variable**. La acción está configurada para la creación de una variable cuyo valor inicial es un objeto JSON que tiene propiedades y valores. Cuando más tarde pruebe la aplicación lógica, puede ejecutar manualmente la aplicación sin esperar a que se active el desencadenador.

   ![Inicio de la aplicación lógica de ejemplo para la acción "Analizar JSON"](./media/logic-apps-perform-data-operations/sample-starting-logic-app-parse-json-action.png)

1. En la aplicación lógica donde desee analizar el contenido JSON, siga uno de estos pasos:

   * Para agregar una acción en el último paso, elija **Nuevo paso**.

     ![Selección de "Nuevo paso" para una acción "Analizar JSON"](./media/logic-apps-perform-data-operations/add-parse-json-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el puntero sobre la flecha de conexión para que el signo más ( **+** ) aparezca. Haga clic en el signo más y, a continuación, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba `parse json` como filtro. En la lista de acciones, seleccione la acción **Análisis del archivo JSON**.

   ![Seleccionar la acción "Análisis del archivo JSON"](./media/logic-apps-perform-data-operations/select-parse-json-action.png)

1. En el cuadro **Contenido**, proporcione el contenido JSON que quiere analizar.

   En este ejemplo, al hacer clic dentro del cuadro **Contenido**, aparece la lista de contenido dinámico, por lo que puede seleccionar la variable creada anteriormente:

   ![Seleccionar el objeto JSON para la acción Análisis del archivo JSON](./media/logic-apps-perform-data-operations/configure-parse-json-action.png)

1. Especifique el esquema JSON que describe el contenido JSON que se va a analizar.

   En este ejemplo, este es el esquema JSON:

   ![Proporcionar el esquema JSON del objeto JSON que quiere analizar](./media/logic-apps-perform-data-operations/provide-schema-parse-json-action.png)

   Si no tiene el esquema, puede generarlo a partir del contenido JSON, o la *carga*, que se va a analizar. 
   
   1. En la acción **Análisis del archivo JSON**, seleccione **Usar una carga de ejemplo para generar el esquema**.

   1. En **Permite especificar o pegar una carga de JSON de ejemplo**, proporcione el contenido JSON y luego elija **Listo**.

      ![Especificar el contenido JSON para generar el esquema](./media/logic-apps-perform-data-operations/generate-schema-parse-json-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Para más información sobre esta acción en la definición del flujo de trabajo subyacente, consulte la [acción Análisis del archivo JSON](../logic-apps/logic-apps-workflow-actions-triggers.md).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para confirmar si la acción **Análisis del archivo JSON** crea los resultados esperados, envíese una notificación que incluya la salida de esta acción.

1. En la aplicación lógica, agregue una acción que pueda enviarle los resultados de la acción **Análisis del archivo JSON**.

1. En dicha acción, haga clic en cualquier lugar donde quiere que aparezcan los resultados. Cuando se abra la lista de contenido dinámico, en la acción **Análisis del archivo JSON**, ahora puede seleccionar las propiedades del contenido JSON analizado.

   En este ejemplo se usa la acción de Office 365 Outlook **Enviar un correo electrónico** y se incluyen los campos **Nombre**, **Apellido** y **Correo electrónico** en el cuerpo del correo electrónico:

   ![Propiedades JSON en la acción "Send an email" (Enviar un correo electrónico)](./media/logic-apps-perform-data-operations/send-email-parse-json-action.png)

   Esta es la acción de correo electrónico finalizada:

   ![Ejemplo finalizado para la acción de correo electrónico](./media/logic-apps-perform-data-operations/send-email-parse-json-action-2.png)

1. Ahora, ejecute la aplicación lógica manualmente. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar). 

   Según el conector de correo electrónico que ha usado, estos son los resultados que obtiene:

   ![Correo electrónico con los resultados de la acción "Analizar JSON"](./media/logic-apps-perform-data-operations/parse-json-email-results.png)

<a name="select-action"></a>

## <a name="select-action"></a>Acción Select

Para crear una matriz que contenga objetos JSON generados a partir de valores de una matriz existente, use la acción **Seleccionar**. Por ejemplo, puede crear un objeto JSON para cada valor de una matriz de enteros mediante la especificación de las propiedades que debe tener cada objeto JSON y cómo asignar los valores de la matriz de origen a esas propiedades. Y aunque puede cambiar los componentes de esos objetos JSON, la matriz de salida tiene siempre el mismo número de elementos que la matriz de origen.

> [!NOTE]
> Para que las acciones usen la salida de matriz de la acción **Seleccionar**, esas acciones deben aceptar matrices como entrada, o puede que tenga que transformar la matriz de salida en otro formato compatible. 

Si prefiere trabajar en el editor de vista de código, puede copiar las definiciones de acción de ejemplo **Seleccionar** e **Inicializar variable** de este artículo en su propia definición de flujo de trabajo subyacente de la aplicación lógica: [Ejemplos de código de operación de datos: Seleccionar](../logic-apps/logic-apps-data-operations-code-samples.md#select-action-example) 

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas.

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador **Periodicidad** y una acción **Inicializar variable**. La acción está configurada para la creación de una variable cuyo valor inicial es una matriz que contiene algunos enteros de ejemplo. Cuando más tarde pruebe la aplicación lógica, puede ejecutar manualmente la aplicación sin esperar a que se active el desencadenador.

   ![Inicio de la aplicación lógica de ejemplo para la acción "Seleccionar"](./media/logic-apps-perform-data-operations/sample-starting-logic-app-select-action.png)

1. En la aplicación lógica donde desea crear la matriz, siga uno de estos pasos: 

   * Para agregar una acción en el último paso, elija **Nuevo paso**.

     ![Selección de "Nuevo paso" para la acción "Seleccionar"](./media/logic-apps-perform-data-operations/add-select-operation-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el puntero sobre la flecha de conexión para que el signo más ( **+** ) aparezca. Haga clic en el signo más y, a continuación, seleccione **Agregar una acción**.

1. En **Elegir una acción**, seleccione **Integrado**. En el cuadro de búsqueda, escriba `select` como filtro. En la lista **Acciones**, seleccione la acción Seleccionar.

   ![Seleccionar la acción "Seleccionar"](./media/logic-apps-perform-data-operations/select-select-action.png)

1. En el cuadro **Desde**, especifique la matriz de origen que quiera.

   En este ejemplo, al hacer clic dentro del cuadro **Desde**, aparece la lista de contenido dinámico, por lo que puede seleccionar la variable creada anteriormente:

   ![Seleccionar la matriz de origen para la acción Seleccionar](./media/logic-apps-perform-data-operations/configure-select-action.png)

1. En la columna izquierda del cuadro **Asignar**, proporcione el nombre de la propiedad que quiere asignar a cada valor de la matriz de origen. En la columna derecha, especifique una expresión que represente el valor que quiere asignar a la propiedad.

   En este ejemplo se especifica "Product_ID" como nombre de la propiedad para asignar a cada valor de la matriz de enteros mediante la función `item()` en una expresión que accede a cada elemento de la matriz. 

   ![Especificación de la propiedad de objeto JSON y los valores para crear una matriz](./media/logic-apps-perform-data-operations/configure-select-action-2.png)

   Esta es la acción finalizada:

   ![Ejemplo finalizado para la acción "Seleccionar"](./media/logic-apps-perform-data-operations/finished-select-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Para más información sobre esta acción en la definición del flujo de trabajo subyacente, consulte la [acción Seleccionar](../logic-apps/logic-apps-workflow-actions-triggers.md).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para confirmar si la acción **Seleccionar** crea los resultados esperados, envíese una notificación que incluya la salida de esta acción.

1. En la aplicación lógica, agregue una acción que pueda enviarle los resultados de la acción **Seleccionar**.

1. En dicha acción, haga clic en cualquier lugar donde quiere que aparezcan los resultados. Cuando se abra la lista de contenido dinámico, seleccione **Expresión**. Para obtener la salida de matriz de la acción **Seleccionar**, escriba esta expresión que incluye el nombre de dicha acción.

   `@actionBody('Select')`

   En este ejemplo se usa la acción de Office 365 Outlook **Enviar un correo electrónico** y se incluyen las salidas de la expresión `@actionBody('Select')` en el cuerpo del correo electrónico:

   ![Salidas de la acción "Seleccionar"](./media/logic-apps-perform-data-operations/send-email-select-action.png)

1. Ahora, ejecute la aplicación lógica manualmente. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar).

   Según el conector de correo electrónico que ha usado, estos son los resultados que obtiene:

   ![Resultados de enviar correo electrónico con la acción "Seleccionar"](./media/logic-apps-perform-data-operations/select-email-results.png)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="format-table-data"></a>Datos de la tabla de formato

Si la [tabla CSV](#create-csv-table-action) o la [tabla HTML](#create-html-table-action) se devuelven con un formato incorrecto, asegúrese de que los datos de entrada tengan saltos de línea entre las filas. 

Formato incorrecto:

```text
Fruit,Number Apples,1 Oranges,2
```

Formato correcto:

```text
Fruit,Number
Apples,1
Oranges,2
```

Para agregar saltos de línea entre las filas, agregue una de las siguientes expresiones a la tabla:

```text
replace(body('Create_CSV_table'),'','<br/>')
```

```text
replace(body('Create_HTML_table'),'','<br/>')
```

Por ejemplo: 

```json
{
    "Send_an_email_": {
        "inputs": {
            "body": {
                "Body": "<p>Results from Create CSV table action:<br/>\n<br/>\n<br/>\n@{replace(body('Create_CSV_table'),'\r\n','<br/>')}</p>",
                "Subject": "Create CSV table results",
                "To": "sophia.owen@fabrikam.com"
            }
        }
    }
}
```
## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre los [conectores de Logic Apps](../connectors/apis-list.md)
