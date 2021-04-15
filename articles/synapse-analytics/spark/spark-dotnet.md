---
title: Uso de .NET para Apache Spark
description: Obtenga información sobre el uso de .NET y Apache Spark para realizar el procesamiento por lotes, el streaming en tiempo real, el aprendizaje automático y la escritura de consultas ad hoc en cuadernos de Azure Synapse Analytics.
author: luisquintanilla
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 05/01/2020
ms.author: luquinta
ms.reviewer: jrasnick
ms.openlocfilehash: 8d045c1ec96bb7b31a710a28e30e3d428922b65e
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378557"
---
# <a name="use-net-for-apache-spark-with-azure-synapse-analytics"></a>Uso de .NET para Apache Spark con Azure Synapse Analytics

[.NET para Apache Spark](https://dot.net/spark) proporciona un marco gratuito, de [código abierto](https://github.com/dotnet/spark) y multiplataforma de .NET para Spark. 

Ofrece enlaces de .NET para Spark que permiten acceder a las API de Spark con C# y F#. Con .NET para Apache Spark, también puede escribir y ejecutar funciones definidas por el usuario para Spark escritas en .NET. Las API de .NET para Spark le permiten acceder a todos los aspectos de las tramas de datos Spark que le ayudan a analizar los datos, incluido Spark SQL, Delta Lake y Structured Streaming.

Puede analizar los datos con .NET para Apache Spark a través de definiciones de trabajos por lotes de Spark o con cuadernos interactivos de Azure Synapse Analytics. En este artículo, aprenderá a usar .NET para Apache Spark con Azure Synapse mediante ambas técnicas.

## <a name="submit-batch-jobs-using-the-spark-job-definition"></a>Envío de trabajos por lotes mediante la definición de trabajos de Spark

Visite el tutorial para aprender a usar Azure Synapse Analytics para [crear definiciones de trabajos de Spark para grupos de Synapse Spark](apache-spark-job-definitions.md). Si no ha empaquetado la aplicación para enviarla a Azure Synapse, complete los pasos siguientes.

1. Ejecute los siguientes comandos para publicar la aplicación. Asegúrese de reemplazar *mySparkApp* por la ruta de acceso a su aplicación.
   
   ```dotnetcli
   cd mySparkApp
   dotnet publish -c Release -f netcoreapp3.1 -r ubuntu.18.04-x64
   ```

2. Comprima el contenido de la carpeta de publicación (por ejemplo, `publish.zip`) que se creó como resultado del paso 1. Todos los elementos ensamblados deben estar en la primera capa del archivo ZIP y no debe haber ninguna capa de carpeta intermedia. Esto significa que, al descomprimir `publish.zip`, todos los elementos ensamblados se extraen en el directorio de trabajo actual.

    **En Windows:**

    Use un programa de extracción, como [7-zip](https://www.7-zip.org/) o [WinZip](https://www.winzip.com/), para extraer el archivo en el directorio bin con todos los archivos binarios publicados.

    **En Linux:**

    Abra un shell de bash, cambie al directorio bin con todos los archivos binarios publicados y ejecute el siguiente comando.

    ```bash
    zip -r publish.zip
    ```

## <a name="net-for-apache-spark-in-azure-synapse-analytics-notebooks"></a>.NET para Apache Spark en cuadernos de Azure Synapse Analytics 

Los cuadernos son una excelente opción para la generación de prototipos de canalizaciones y escenarios de .NET para Apache Spark. Puede empezar a trabajar con los datos, comprenderlos, filtrarlos, mostrarlos y visualizarlos de forma rápida y eficaz. 

Los ingenieros de datos, científicos de datos, analistas de negocios e ingenieros de aprendizaje automático pueden colaborar en un documento compartido e interactivo. Verá resultados inmediatos de la exploración de datos y podrá visualizar los datos en el mismo cuaderno.

### <a name="how-to-use-net-for-apache-spark-notebooks"></a>Cómo usar .NET para cuadernos de Apache Spark

Al crear un cuaderno, elige un kernel de lenguaje en el que quiere expresar la lógica de negocios. El kernel admite varios lenguajes, incluido C#.

Para usar .NET para Apache Spark en el cuaderno de Azure Synapse Analytics, seleccione **.NET Spark (C#)** como kernel y conecte el cuaderno a un grupo de Apache Spark sin servidor existente.

El cuaderno de .NET Spark se basa en las experiencias [interactivas de .NET](https://github.com/dotnet/interactive) y proporciona experiencias interactivas de C# con la capacidad de usar .NET para Spark de forma predefinida con la variable de sesión de Spark `spark` ya predefinida.

### <a name="install-nuget-packages-in-notebooks"></a>Instalación de paquetes NuGet en cuadernos

Puede instalar los paquetes NuGet que prefiera en el cuaderno con comando magic `#r nuget` antes del nombre del paquete NuGet. En el diagrama siguiente se muestra un ejemplo:

![Captura de pantalla que muestra cómo usar #r para instalar un paquete NuGet en un cuaderno de .NET para Spark](./media/apache-spark-development-using-notebooks/synapse-spark-dotnet-notebook-nuget.png)

Para obtener más información sobre cómo trabajar con paquetes NuGet en cuadernos, consulte la [documentación interactiva de .NET](https://github.com/dotnet/interactive/blob/main/docs/nuget-overview.md).

### <a name="net-for-apache-spark-c-kernel-features"></a>Características de .NET para el kernel C# de Apache Spark

Las siguientes características están disponibles cuando se usa .NET para Apache Spark en el cuaderno de Azure Synapse Analytics:

* HTML declarativa: Genere resultados a partir de las celdas con sintaxis HTML, como encabezados, listas con viñetas e, incluso, mostrar imágenes.
* Instrucciones simples de C# (como asignaciones, impresión en la consola, generación de excepciones, etc.).
* Bloques de código de varias líneas de C# (como instrucciones if, bucles foreach, definiciones de clase, etc.).
* Acceso a la biblioteca estándar de C# (como System, LINQ, Enumerable, etc.).
* Compatibilidad con las características del lenguaje C# 8.0.
* `spark` como variable predefinida para darle acceso a la sesión de Apache Spark.
* Compatibilidad para definir [funciones definidas por el usuario de .NET que se pueden ejecutar en Apache Spark](/dotnet/spark/how-to-guides/udf-guide). Le recomendamos leer [Escritura y llamada a UDF en entornos interactivos de .NET para Apache Spark](/dotnet/spark/how-to-guides/dotnet-interactive-udf-issue) para aprender a usar UDF en experiencias interactivas de .NET para Apache Spark.
* Compatibilidad para visualizar la salida de los trabajos de Spark con distintos gráficos (como líneas, barras o histogramas), y diseños (como único, superpuesto, etc.) mediante la biblioteca de `XPlot.Plotly`.
* Posibilidad de incluir paquetes NuGet en el cuaderno de C#.

## <a name="next-steps"></a>Pasos siguientes

* [Documentación de .NET para Apache Spark](/dotnet/spark/)
* [Guías interactivas de .NET para Apache Spark](/dotnet/spark/how-to-guides/dotnet-interactive-udf-issue)
* [Azure Synapse Analytics](https://azure.microsoft.com/services/synapse-analytics/)
* [Documentación interactiva de .NET](https://devblogs.microsoft.com/dotnet/creating-interactive-net-documentation/)
