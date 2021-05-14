---
title: Realización de predicciones con un modelo AutoML de ONNX en .NET
description: Aprenda a hacer predicciones utilizando un modelo AutoML de ONNX en .NET con ML.NET
titleSuffix: Azure Machine Learning
author: luisquintanilla
ms.author: luquinta
ms.date: 10/30/2020
ms.topic: how-to
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.custom: automl
ms.openlocfilehash: 2764623142ce09a248ee1b9c0efa1e409c3c8a8c
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884361"
---
# <a name="make-predictions-with-an-automl-onnx-model-in-net"></a>Realización de predicciones con un modelo AutoML de ONNX en .NET

En este artículo, aprenderá a usar un modelo de ML automatizado (AutoML) de Neural Network Exchange (ONNX) para hacer predicciones en una aplicación de consola de .NET Core de C# con ML.NET.

[ML.NET](/dotnet/machine-learning/) es un marco de aprendizaje automático multiplataforma de código abierto del ecosistema de .NET que permite entrenar y consumir modelos de aprendizaje automático personalizados utilizando un enfoque Code First de C# o F# junto con herramientas que apenas requieren código, como [Model Builder](/dotnet/machine-learning/automate-training-with-model-builder) y la [CLI de ML.NET](/dotnet/machine-learning/automate-training-with-cli). Este marco puede ampliarse y permite utilizar otros marcos de aprendizaje automático populares, como TensorFlow y ONNX.

ONNX es un formato de código abierto para modelos de IA. ONNX admite la interoperabilidad entre marcos. Esto significa que puede entrenar un modelo en uno de los muchos marcos de aprendizaje automático que son más conocidos, como PyTorch, convertirlo a un formato ONNX y consumir el modelo de ONNX en un marco de trabajo diferente, como ML.NET. Para más información, visite el [sitio web de ONNX](https://onnx.ai/).

## <a name="prerequisites"></a>Requisitos previos

- [SDK de .NET Core 3.1 o posterior](https://dotnet.microsoft.com/download)
- Editor de texto o IDE (como [Visual Studio](https://visualstudio.microsoft.com/vs/) o [Visual Studio Code](https://code.visualstudio.com/Download))
- Modelo de ONNX. Para más información acerca de cómo entrenar un modelo AutoML de ONNX, consulte el siguiente [cuaderno de clasificación para marketing bancario](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features/auto-ml-classification-bank-marketing-all-features.ipynb).
- [Netron](https://github.com/lutzroeder/netron) (opcional)

## <a name="create-a-c-console-application"></a>Creación de una aplicación de consola en C#

En este ejemplo, va a utilizar la CLI de .NET Core para compilar la aplicación, pero puede realizar las mismas tareas con Visual Studio. Más información sobre la [CLI de .NET Core](/dotnet/core/tools/).

1. Abra un terminal y cree una aplicación de consola de .NET Core en C#. En este ejemplo, el nombre de la aplicación es `AutoMLONNXConsoleApp`. Se creará un directorio que tendrá el mismo nombre con el contenido de la aplicación.

    ```dotnetcli
    dotnet new console -o AutoMLONNXConsoleApp
    ```

1. En el terminal, vaya al directorio *AutoMLONNXConsoleApp*.

    ```bash
    cd AutoMLONNXConsoleApp
    ```

## <a name="add-software-packages"></a>Incorporación de los paquetes de software

1. Instale los paquetes de NuGet **Microsoft.ML**, **Microsoft.ML.OnnxRuntime** y **Microsoft.ML.OnnxTransformer** mediante la CLI de .NET Core.

    ```dotnetcli
    dotnet add package Microsoft.ML
    dotnet add package Microsoft.ML.OnnxRuntime
    dotnet add package Microsoft.ML.OnnxTransformer
    ```

    Estos paquetes contienen las dependencias necesarias para usar un modelo de ONNX en una aplicación .NET. ML.NET cuenta con una API que usa el [entorno de ejecución de ONNX](https://github.com/Microsoft/onnxruntime) para realizar las predicciones.

1. Abra el archivo *Program.cs* y agregue al comienzo las siguientes instrucciones `using` para hacer referencia a los paquetes correspondientes.

    ```csharp
    using System.Linq;
    using Microsoft.ML;
    using Microsoft.ML.Data;
    using Microsoft.ML.Transforms.Onnx;
    ```

## <a name="add-a-reference-to-the-onnx-model"></a>Incorporación de una referencia al modelo de ONNX

Una forma de conseguir que la aplicación de consola acceda al modelo de ONNX es agregarla al directorio de salida de la compilación.  Para más información sobre los elementos comunes de MSBuild, consulte la [guía de MSBuild](/visualstudio/msbuild/common-msbuild-project-items).

Agregue una referencia al archivo del modelo de ONNX de la aplicación.

1. Copie el modelo ONNX en el directorio raíz *AutoMLONNXConsoleApp* de la aplicación.
1. Abra el archivo *AutoMLONNXConsoleApp. csproj* y agregue el siguiente contenido dentro del nodo `Project`.

    ```xml
    <ItemGroup>
        <None Include="automl-model.onnx">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
        </None>
    </ItemGroup>
    ```

    En este caso, el nombre del archivo de modelo de ONNX es *automl-model.onnx*.

1. Abra el archivo *Program.cs* y agregue la línea siguiente dentro de la clase `Program`.

    ```csharp
    static string ONNX_MODEL_PATH = "automl-model.onnx";
    ```

## <a name="initialize-mlcontext"></a>Inicialización de MLContext

Dentro del método `Main` de la clase `Program`, cree una nueva instancia de [`MLContext`](xref:Microsoft.ML.MLContext).

```csharp
MLContext mlContext = new MLContext();
```

La clase [`MLContext`](xref:Microsoft.ML.MLContext) es el punto de partida de todas las operaciones de ML.NET, mientras que la inicialización de `mlContext` crea un nuevo entorno de ML.NET que puede compartirse durante el ciclo de vida del modelo. Conceptualmente, es similar a DbContext en Entity Framework.

## <a name="define-the-model-data-schema"></a>Definición del esquema de datos del modelo

El modelo espera que los datos de entrada y salida estén en un formato específico. ML.NET permite definir el formato de los datos mediante clases. En ocasiones, es posible que ya sepa cuál es el formato. De lo contrario, puede usar herramientas como Netron para inspeccionar el modelo de ONNX.

El modelo de este ejemplo utiliza la información del conjunto de datos NYC TLC Taxi Trip. A continuación, se muestra un ejemplo de los datos:

|vendor_id|rate_code|passenger_count|trip_time_in_secs|trip_distance|payment_type|fare_amount|
|---|---|---|---|---|---|---|
|VTS|1|1|1140|3,75|CRD|15,5|
|VTS|1|1|480|2.72|CRD|10.0|
|VTS|1|1|1680|7.8|CSH|26,5|

### <a name="inspect-the-onnx-model-optional"></a>Inspección del modelo de ONNX (opcional)

Utilice una herramienta como Netron para inspeccionar las entradas y salidas del modelo.

1. Abra Netron.
1. En la barra de menús superior, seleccione **File > Open** (Archivo > Abrir) y use el explorador de archivos para seleccionar el modelo.
1. Se abre el modelo. Por ejemplo, la estructura del modelo *automl-model.onnx* se parece a la siguiente:

    :::image type="content" source="media/how-to-use-automl-onnx-model-dotnet/netron-automl-onnx-model.png" alt-text="Modelo AutoML de ONNX en Netron":::

1. Seleccione el último nodo de la parte inferior del grafo (`variable_out1` en este caso) para ver los metadatos del modelo. Las entradas y salidas de la barra lateral representan las entradas, salidas y tipos de datos esperados del modelo. Utilice esta información para definir el esquema de entrada y salida del modelo.

### <a name="define-model-input-schema"></a>Definición del esquema de entrada del modelo

Cree una nueva clase llamada `OnnxInput` con las siguientes propiedades en el archivo *Program.cs*.

```csharp
public class OnnxInput
{
    [ColumnName("vendor_id")]
    public string VendorId { get; set; }

    [ColumnName("rate_code"),OnnxMapType(typeof(Int64),typeof(Single))]
    public Int64 RateCode { get; set; }

    [ColumnName("passenger_count"), OnnxMapType(typeof(Int64), typeof(Single))]
    public Int64 PassengerCount { get; set; }

    [ColumnName("trip_time_in_secs"), OnnxMapType(typeof(Int64), typeof(Single))]
    public Int64 TripTimeInSecs { get; set; }

    [ColumnName("trip_distance")]
    public float TripDistance { get; set; }

    [ColumnName("payment_type")]
    public string PaymentType { get; set; }
}
```

Cada una de las propiedades se asigna a una columna del conjunto de datos. Además, se incluirán atributos en las notas de las propiedades.

El atributo [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) permite especificar cómo ML.NET debe hacer referencia a la columna al trabajar con los datos. Por ejemplo, aunque la propiedad `TripDistance` sigue las convenciones de nomenclatura estándar de .NET, el modelo solo tiene información de una columna o característica llamada `trip_distance`. Para solucionar esta discrepancia de nomenclatura, el atributo [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) asigna la propiedad `TripDistance` a una columna o característica con el nombre `trip_distance`.
  
En el caso de los valores numéricos, ML.NET solo trabaja con tipos de valor [`Single`](xref:System.Single). Sin embargo, el tipo de datos original de algunas de las columnas es un entero. El atributo [`OnnxMapType`](xref:Microsoft.ML.Transforms.Onnx.OnnxMapTypeAttribute) asigna tipos entre ONNX y ML.NET.

Para más información sobre los atributos de datos, consulte la [guía de datos de carga de ML.NET](/dotnet/machine-learning/how-to-guides/load-data-ml-net).

### <a name="define-model-output-schema"></a>Definición del esquema de salida del modelo

Una vez procesados los datos, se genera una salida con un formato determinado. Defina el esquema de salida de los datos. Cree una nueva clase llamada `OnnxOutput` con las siguientes propiedades en el archivo *Program.cs*.

```csharp
public class OnnxOutput
{
    [ColumnName("variable_out1")]
    public float[] PredictedFare { get; set; }
}
```

Al igual que con `OnnxInput`, utilice el atributo [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) para asignar a la salida `variable_out1` un nombre más descriptivo: `PredictedFare`.

## <a name="define-a-prediction-pipeline"></a>Definición de una canalización de predicciones

Normalmente, las canalizaciones de ML.NET son una serie de transformaciones en cadena que se realizan en los datos de entrada para generar una salida. Para más información sobre las transformaciones de datos, consulte la [guía de transformación de datos de ML.NET](/dotnet/machine-learning/resources/transforms).

1. Cree un nuevo método llamado `GetPredictionPipeline` dentro de la clase `Program`.

    ```csharp
    static ITransformer GetPredictionPipeline(MLContext mlContext)
    {

    }
    ```

1. Defina el nombre de las columnas de entrada y de salida. Agregue el código siguiente dentro del método `GetPredictionPipeline`.

    ```csharp
    var inputColumns = new string []
    {
        "vendor_id", "rate_code", "passenger_count", "trip_time_in_secs", "trip_distance", "payment_type"
    };

    var outputColumns = new string [] { "variable_out1" };
    ```

1. Defina la canalización. Un objeto [`IEstimator`](xref:Microsoft.ML.IEstimator%601) proporciona un proyecto de las operaciones y de los esquemas de entrada y salida de la canalización.

    ```csharp
    var onnxPredictionPipeline =
        mlContext
            .Transforms
            .ApplyOnnxModel(
                outputColumnNames: outputColumns,
                inputColumnNames: inputColumns,
                ONNX_MODEL_PATH);
    ```

    En este caso, [`ApplyOnnxModel`](xref:Microsoft.ML.OnnxCatalog.ApplyOnnxModel%2A) es la única transformación de la canalización, que toma los nombres de las columnas de entrada y salida, y la ruta de acceso al archivo del modelo de ONNX.

1. Los objetos [`IEstimator`](xref:Microsoft.ML.IEstimator%601) solo definen el conjunto de operaciones que se van a aplicar a los datos. Lo que realmente trabaja con los datos se conoce como [`ITransformer`](xref:Microsoft.ML.ITransformer). Utilice el método `Fit` para crear uno a partir de `onnxPredictionPipeline`.

    ```csharp
    var emptyDv = mlContext.Data.LoadFromEnumerable(new OnnxInput[] {});

    return onnxPredictionPipeline.Fit(emptyDv);
    ```

    El método [`Fit`](xref:Microsoft.ML.IEstimator%601.Fit%2A) espera un objeto [`IDataView`](xref:Microsoft.ML.IDataView) como entrada para realizar las operaciones. Los objetos [`IDataView`](xref:Microsoft.ML.IDataView) permiten representar los datos en ML.NET con un formato tabular. Dado que en este caso la canalización solo se usa para realizar predicciones, puede proporcionar un objeto [`IDataView`](xref:Microsoft.ML.IDataView) para que [`ITransformer`](xref:Microsoft.ML.ITransformer) tenga la información necesaria del esquema de entrada y de salida. A continuación, se devolverá el objeto [`ITransformer`](xref:Microsoft.ML.ITransformer) al que se le ha aplicado el método Fit para que siga utilizándose en la aplicación.

    > [!TIP]
    > En este ejemplo, la canalización se define y se utiliza dentro de la misma aplicación. Sin embargo, cuando se creen predicciones, se recomienda usar diferentes aplicaciones para definir y usar la canalización. En ML.NET, las canalizaciones se pueden serializar y guardar para utilizarse en otras aplicaciones NET dirigidas a usuarios finales. ML.NET admite varios destinos de implementación, como aplicaciones de escritorio, servicios web, aplicaciones de WebAssembly * y muchos otros. Para más información acerca de cómo guardar las canalizaciones, consulte la [guía para guardar y cargar modelos entrenados de ML.NET](/dotnet/machine-learning/how-to-guides/save-load-machine-learning-models-ml-net).
    >
    > *WebAssembly solo se admite en .NET Core 5 o versiones posteriores

1. En el método `Main`, llame al método `GetPredictionPipeline` con los parámetros necesarios.

    ```csharp
    var onnxPredictionPipeline = GetPredictionPipeline(mlContext);
    ```

## <a name="use-the-model-to-make-predictions"></a>Uso del modelo para realizar predicciones

Ahora que tiene una canalización, es el momento de utilizarla para realizar predicciones. ML.NET dispone de una práctica API que permite llevar a cabo predicciones en una única instancia de datos denominada [`PredictionEngine`](xref:Microsoft.ML.PredictionEngine%602).

1. En el método `Main`, cree un objeto [`PredictionEngine`](xref:Microsoft.ML.PredictionEngine%602) utilizando el método [`CreatePredictionEngine`](xref:Microsoft.ML.ModelOperationsCatalog.CreatePredictionEngine%2A).

    ```csharp
    var onnxPredictionEngine = mlContext.Model.CreatePredictionEngine<OnnxInput, OnnxOutput>(onnxPredictionPipeline);
    ```

1. Cree una entrada de datos de prueba.

    ```csharp
    var testInput = new OnnxInput
    {
        VendorId = "CMT",
        RateCode = 1,
        PassengerCount = 1,
        TripTimeInSecs = 1271,
        TripDistance = 3.8f,
        PaymentType = "CRD"
    };
    ```

1. Utilice el objeto `predictionEngine` para hacer predicciones basadas en los nuevos datos de `testInput` con el método [`Predict`](xref:Microsoft.ML.PredictionEngineBase%602.Predict%2A).

    ```csharp
    var prediction = onnxPredictionEngine.Predict(testInput);
    ```

1. Envíe la salida de la predicción a la consola.

    ```csharp
    Console.WriteLine($"Predicted Fare: {prediction.PredictedFare.First()}");
    ```

1. Use la CLI de .NET Core para ejecutar la aplicación.

    ```dotnetcli
    dotnet run
    ```

    La salida debe ser parecida a esta:

    ```text
    Predicted Fare: 15.621523
    ```

Para más información acerca de cómo realizar predicciones en ML.NET, consulte [la guía sobre el uso de un modelo para hacer predicciones](/dotnet/machine-learning/how-to-guides/machine-learning-model-predictions-ml-net).

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de un modelo en una ASP.NET Core Web API](/dotnet/machine-learning/how-to-guides/serve-model-web-api-ml-net)
- [Implementación de un modelo como una instancia .NET sin servidor de Azure Functions](/dotnet/machine-learning/how-to-guides/serve-model-serverless-azure-functions-ml-net)