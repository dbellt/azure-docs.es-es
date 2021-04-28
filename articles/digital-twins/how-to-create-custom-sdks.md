---
title: Creación de SDK de lenguaje personalizado con AutoRest
titleSuffix: Azure Digital Twins
description: Aprenda a usar AutoRest para generar SDK de lenguaje personalizado, para escribir código de Azure Digital Twins en otros lenguajes que no tengan SDK publicados.
author: baanders
ms.author: baanders
ms.date: 3/9/2021
ms.topic: how-to
ms.service: digital-twins
ms.custom:
- devx-track-js
- contperf-fy21q3
ms.openlocfilehash: 4e91bf5acc5290229afa8dc7a849e8953257bcfd
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751119"
---
# <a name="create-custom-language-sdks-for-azure-digital-twins-using-autorest"></a>Creación de SDK de lenguaje personalizado para Azure Digital Twins con AutoRest

Si necesita trabajar con Azure Digital Twins con un lenguaje que no tenga un [SDK de Azure Digital Twins publicado](how-to-use-apis-sdks.md), este artículo le muestra cómo usar AutoRest para generar su propio SDK en el lenguaje que prefiera. 

En los ejemplos de este artículo se muestra la creación de un [SDK de plano de datos](how-to-use-apis-sdks.md#overview-data-plane-apis), pero este proceso también funciona para generar un [ SDK de plano de control](how-to-use-apis-sdks.md#overview-control-plane-apis).

## <a name="prerequisites"></a>Requisitos previos

Para generar un SDK, primero debe completar la siguiente configuración en la máquina local:
* Instale [**AutoRest**](https://github.com/Azure/autorest), versión 2.0.4413 (la versión 3 no se admite por el momento).
* Instale [**Node.js**](https://nodejs.org), que es un requisito previo para usar AutoRest.
* Instale [**Visual Studio**](https://visualstudio.microsoft.com/downloads/).
* Descargue el archivo de **plano de datos de Swagger** (OpenAPI) más reciente de Azure Digital Twins desde la [carpeta correspondiente](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/data-plane/Microsoft.DigitalTwins), junto con su carpeta de ejemplos. El archivo Swagger es el que se llama *digitaltwins.jsen*.

>[!TIP]
> Para crear mejor un **SDK de plano de control**, realice los pasos de este artículo usando el archivo de **plano de control de Swagger** (OpenAPI) más reciente de la [carpeta correspondiente](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/resource-manager/Microsoft.DigitalTwins/), en lugar del plano de datos uno.

Una vez que la máquina esté equipada con todos los elementos de la lista anterior, está listo para crear el SDK con AutoRest.

## <a name="create-the-sdk-using-autorest"></a>Creación del SDK con AutoRest 

Cuando se haya instalado Node.js, puede ejecutar este comando para asegurarse de que tiene instalada la versión correcta de AutoRest:
```cmd/sh
npm install -g autorest@2.0.4413
```

Para ejecutar AutoRest con el archivo Swagger de Azure Digital Twins, siga estos pasos:
1. Copie el archivo Swagger de Azure Digital Twins y su correspondiente carpeta de ejemplos en un directorio de trabajo.
2. Use una ventana del símbolo del sistema para cambiar a ese directorio de trabajo.
3. Ejecute AutoRest con el siguiente comando. Reemplace el marcador de posición `<language>` por el lenguaje que prefiera: `python`, `java`, `go`, etc. (Puede encontrar la lista completa de opciones en el archivo [Léame de AutoRest](https://github.com/Azure/autorest)).

```cmd/sh
autorest --input-file=digitaltwins.json --<language> --output-folder=DigitalTwinsApi --add-credentials --azure-arm --namespace=DigitalTwinsApi
```

Como resultado, verá una nueva carpeta denominada *DigitalTwinsApi* en el directorio de trabajo. Los archivos de SDK generados tendrán el espacio de nombres *DigitalTwinsApi*. Seguirá usando ese espacio de nombres en el resto de los ejemplos de uso de este artículo.

AutoRest es compatible con una amplia gama de generadores de código de lenguaje.

## <a name="make-the-sdk-into-a-class-library"></a>Conversión del SDK en una biblioteca de clases

Puede incluir los archivos generados mediante AutoRest directamente en una solución de .NET. Sin embargo, es probable que quiera incluir el SDK de Azure Digital Twins en varios proyectos distintos (aplicaciones cliente, aplicaciones de Azure Functions, etc.). Por esta razón, puede ser útil crear un proyecto independiente (una biblioteca de clases .NET) a partir de los archivos generados. A continuación, puede incluir este proyecto de biblioteca de clases en varias soluciones como referencia de proyecto.

En esta sección se proporcionan instrucciones sobre cómo compilar el SDK como una biblioteca de clases, que es su propio proyecto y se puede incluir en otros proyectos. Estos pasos se basan en **Visual Studio**.

He aquí los pasos:

1. Creación de una nueva solución de Visual Studio para una biblioteca de clases
2. Use *DigitalTwinsApi* como nombre del proyecto.
3. En el Explorador de soluciones, seleccione con el botón derecho el proyecto *DigitalTwinsApi* de la solución generada y elija *Agregar > Elemento existente...*
4. Busque la carpeta en la que generó el SDK y seleccione los archivos en el nivel raíz.
5. Presione "Aceptar".
6. Agregue una carpeta al proyecto (haga clic con el botón derecho para seleccionar el proyecto en el Explorador de soluciones y elija *Agregar > Nueva carpeta*).
7. Asigne a la carpeta el nombre *Models*.
8. En el Explorador de soluciones, seleccione con el botón derecho la carpeta *Models* y elija *Agregar > Elemento existente...*
9. Seleccione los archivos de la carpeta *Models* del SDK generado y presione "Aceptar".

Para compilar el SDK correctamente, el proyecto necesitará estas referencias:
* `Microsoft.Rest.ClientRuntime`
* `Microsoft.Rest.ClientRuntime.Azure`

Para agregarlas, abra *Herramientas > Administrador de paquetes NuGet > Administrar paquetes NuGet para la solución...*

1. En el panel, asegúrese de que esté seleccionada la pestaña *Examinar*.
2. Busque *Microsoft.Rest*.
3. Seleccione los paquetes `ClientRuntime` y `ClientRuntime.Azure`, y agréguelos a la solución.

Ahora puede compilar el proyecto e incluirlo como una referencia de proyecto en cualquier aplicación de Azure Digital Twins que escriba.

## <a name="tips-for-using-the-sdk"></a>Sugerencias de uso del SDK

En esta sección se incluye información general sobre el SDK generado, así como instrucciones para usarlo.

### <a name="synchronous-and-asynchronous-calls"></a>Llamadas sincrónicas y asincrónicas

Todas las funciones del SDK cuentan con versiones sincrónicas y asincrónicas.

### <a name="typed-and-untyped-data"></a>Datos con tipo y sin tipo

Por lo general, las llamadas API de REST devuelven objetos fuertemente tipados. Sin embargo, dado que Azure Digital Twins permite a los usuarios definir sus propios tipos personalizados para gemelos, no hay ninguna manera de predefinir datos devueltos estáticos para muchas llamadas de Azure Digital Twins. En su lugar, las API devuelven tipos de contenedor fuertemente tipados donde corresponde, y los datos de gemelos con tipos personalizados están en objetos Json.NET (que se usan siempre que aparece el tipo de datos "object" en las firmas de API). Puede convertir estos objetos según corresponda en el código.

### <a name="error-handling"></a>Control de errores

Siempre que se produzca un error en el SDK (incluidos los errores HTTP, como 404), el SDK devolverá una excepción. Por esta razón, es importante encapsular todas las llamadas API dentro de bloques try/catch.

A continuación se muestra un fragmento de código que intenta agregar un gemelo y detecta cualquier error en este proceso:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="CreateTwin_errorHandling":::

### <a name="paging"></a>Paginación

AutoRest genera dos tipos de patrones de paginación para el SDK:
* Uno para todas las API, excepto la API de consulta
* Uno para la API de consulta

En el patrón de paginación sin consultas se incluye un método de ejemplo que muestra cómo recuperar una lista paginada de relaciones de salida de Azure Digital Twins:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FindOutgoingRelationshipsMethod":::

El segundo patrón solo se genera para la API de consulta. Usa `continuationToken` explícitamente.

>[!TIP]
> Una de las razones principales de obtener páginas es calcular los [cargos por unidad de consulta](concepts-query-units.md) por una llamada a la API de consulta.

A continuación, se indica un ejemplo con este patrón:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/queries.cs" id="PagedQuery":::

## <a name="next-steps"></a>Pasos siguientes

Siga los pasos para crear una aplicación cliente donde pueda usar el SDK:
* [*Tutorial: Programación de una aplicación cliente*](tutorial-code.md)
