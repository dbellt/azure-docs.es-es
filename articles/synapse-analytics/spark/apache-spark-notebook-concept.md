---
title: Introducción a los cuadernos de Azure Synapse Analytics
description: En este artículo se proporciona información general sobre las funcionalidades a las que se puede acceder desde los cuadernos de Azure Synapse Analytics.
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 11/18/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.subservice: spark
ms.openlocfilehash: dbdfb7e13249de21b890c6f4476cb82a5c91a23e
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109809911"
---
# <a name="azure-synapse-analytics-notebooks"></a>Cuadernos de Azure Synapse Analytics

Un cuaderno de Synapse Studio es una interfaz web para crear archivos que contengan código activo, visualizaciones y texto narrativo. Los cuadernos son un buen lugar para validar ideas y aplicar experimentos rápidos para sacar conclusiones a partir de los datos. 

Con un cuaderno de Synapse Studio, puede hacer lo siguiente:

* Empezar a trabajar sin esfuerzo alguno de configuración.
* Mantener los datos protegidos con las características de seguridad empresarial integradas.
* Analizar datos en formatos sin procesar (CSV, TXT, JSON, etc.), formatos de archivos procesados (parquet, Delta Lake, ORC, etc.) y archivos de datos tabulares de SQL en Spark y SQL.
* Ser productivo con funcionalidades de creación mejoradas y visualización de datos integrada.

Esta sección contiene artículos sobre la combinación de lenguajes, la creación de visualizaciones de datos, la parametrización de cuadernos, la creación de canalizaciones, etc. También contiene referencias y tutoriales para que pueda empezar el desarrollo de cuadernos.

## <a name="create-manage-and-use-notebooks"></a>Creación, administración y uso de cuadernos
Los cuadernos se pueden administrar desde la interfaz de usuario de Synapse Studio. 

Para más información sobre cómo crear y administrar cuadernos, consulte los siguientes artículos:
  - Administración de cuadernos
    - [Creación de cuadernos](./spark/../apache-spark-development-using-notebooks.md#create-a-notebook)
    - [Desarrollo de cuadernos](./spark/../apache-spark-development-using-notebooks.md#develop-notebooks)
    - [Traslado de los datos a un cuaderno](./spark/../apache-spark-development-using-notebooks.md#bring-data-to-a-notebook)
    - [Uso de varios lenguajes mediante comandos magic y tablas temporales](./spark/../apache-spark-development-using-notebooks.md#integrate-a-notebook)
    - [Uso de comandos magic de celda](./spark/../apache-spark-development-using-notebooks.md#magic-commands)
  - Desarrollo
    - [Configuración de los valores de sesión de Spark](./spark/../apache-spark-development-using-notebooks.md#spark-session-config)
    - [Uso de las utilidades Spark para Microsoft](./spark/../microsoft-spark-utilities.md)
    - [Visualización de datos mediante cuadernos y bibliotecas](./spark/../apache-spark-data-visualization.md)
    - [Integración de un cuaderno en canalizaciones](./spark/../apache-spark-development-using-notebooks.md#integrate-a-notebook)


## <a name="next-steps"></a>Pasos siguientes
Los cuadernos también se usan profusamente en la preparación y visualización de datos, el aprendizaje automático y otros escenarios de macrodatos. Para más información sobre cómo usar cuadernos en escenarios de análisis de datos y macrodatos, consulte los tutoriales siguientes:
  - [Creación de un cuaderno](./spark/../../quickstart-apache-spark-notebook.md)
  - [Creación de visualizaciones mediante cuadernos de Synapse Studio](./spark/../apache-spark-data-visualization-tutorial.md)
  - [Compilación de modelos de Machine Learning con Apache Spark MLlib](./spark/../apache-spark-machine-learning-mllib-notebook.md)
  - [Compilación de modelos de Machine Learning con el aprendizaje automático automatizado de Azure](./spark/../apache-spark-azure-machine-learning-tutorial.md)
