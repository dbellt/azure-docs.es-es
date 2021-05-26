---
title: Herramientas de aprendizaje automático y ciencia de datos
titleSuffix: Azure Data Science Virtual Machine
description: Obtenga información sobre los marcos y las herramientas de aprendizaje automático instalados previamente en Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: data-science-vm
author: timoklimmer
ms.author: tklimmer
ms.topic: conceptual
ms.date: 05/12/2021
ms.openlocfilehash: 03ab10c4ee4ab4db1054650630ba347abbc7f25e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110068849"
---
# <a name="machine-learning-and-data-science-tools-on-azure-data-science-virtual-machines"></a>Herramientas de aprendizaje automático y ciencia de datos en Azure Data Science Virtual Machines
Azure Data Science Virtual Machine (DSVM) tiene un amplio conjunto de herramientas y bibliotecas para el aprendizaje automático, disponibles en lenguajes conocidos como Python, R o Julia.

A continuación se muestran algunas de las herramientas y bibliotecas de aprendizaje automático de DSVM.

## <a name="azure-machine-learning-sdk-for-python"></a>SDK de Azure Machine Learning para Python

Consulte la referencia completa del [SDK de Azure Machine Learning para Python](../overview-what-is-azure-ml.md).

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   |   Azure Machine Learning es un servicio en la nube que puede usar para desarrollar e implementar modelos de aprendizaje automático. Puede realizar un seguimiento de los modelos mientras los compila, entrena, escala y administra mediante el SDK de Python. Implemente modelos como contenedores y ejecútelos en la nube, de forma local o en Azure IoT Edge.   |
| Ediciones compatibles     | Windows (entorno de conda: AzureML), Linux (entorno de conda: py36)    |
| Usos típicos      | Plataforma de aprendizaje automático general      |
| ¿Cómo se configura o instala?      |  Se instala con la compatibilidad de GPU   |
| ¿Cómo se usa o ejecuta?      | Como un SDK de Python y en la CLI de Azure. Active el entorno de Conda `AzureML` en la edición de Windows *o* a `py36`, en la edición de Linux.      |
| Vínculos a ejemplos      | Se incluyen ejemplos de cuadernos de Jupyter Notebook en el directorio `AzureML` bajo los cuadernos.  |

## <a name="h2o"></a>H2O

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | Plataforma de IA de código abierto que admite aprendizaje automático en memoria, distribuido, rápido y escalable.  |
| Versiones compatibles      | Linux   |
| Usos típicos      | Aprendizaje automático escalable y distribuido de uso general   |
| ¿Cómo se configura o instala?      | H2O se instala en `/dsvm/tools/h2o`.      |
| ¿Cómo se usa o ejecuta?      | Conéctese a la máquina virtual con X2Go. Inicie un nuevo terminal y ejecute `java -jar /dsvm/tools/h2o/current/h2o.jar`. Luego, inicie un explorador web y conéctese a `http://localhost:54321`.      |
| Vínculos a ejemplos      | Hay ejemplos disponibles en la máquina virtual de Jupyter en el directorio `h2o`.      |

Hay otras bibliotecas de aprendizaje automático en DSVM, como el conocido paquete `scikit-learn`, que forma parte de la distribución de Anaconda Python para DSVM. Para extraer del repositorio la lista de paquetes disponibles en Python, R y Julia, ejecute los administradores de paquetes correspondientes.

## <a name="lightgbm"></a>LightGBM

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | Un marco de potenciación del gradiente (GBDT, GBRT, GBM o MART) rápido, distribuido y de alto rendimiento, basado en algoritmos de árbol de decisión. Se utiliza para la categorización, la clasificación y muchas otras tareas de aprendizaje automático.    |
| Versiones compatibles      | Windows, Linux    |
| Usos típicos      | Marco de potenciación del gradiente de uso general      |
| ¿Cómo se configura o instala?      | En Windows, LightGBM se instala en forma de paquete de Python. En Linux, el archivo ejecutable de la línea de comandos se encuentra en `/opt/LightGBM/lightgbm`; luego se instala el paquete de R y los paquetes de Python.     |
| Vínculos a ejemplos      | [Guía de LightGBM](https://github.com/Microsoft/LightGBM/tree/master/examples/python-guide)   |

## <a name="rattle"></a>Rattle
| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   |   Una interfaz gráfica de usuario para la minería de datos con R.   |
| Ediciones compatibles     | Windows, Linux     |
| Usos típicos      | Herramienta general de minería de datos de la IU para R    |
| ¿Cómo se usa o ejecuta?      | Como herramienta de interfaz de usuario. En Windows, inicie un símbolo del sistema, ejecute R y, dentro de R, ejecute `rattle()`. En Linux, conéctese a X2Go, inicie un terminal, ejecute R y, dentro de R, ejecute `rattle()`. |
| Vínculos a ejemplos      | [Rattle](https://togaware.com/onepager/) |

## <a name="vowpal-wabbit"></a>Vowpal Wabbit
| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   |   Una biblioteca de sistema de aprendizaje rápida, de código abierto y situada fuera de núcleo    |
| Ediciones compatibles     | Windows, Linux     |
| Usos típicos      | Biblioteca de aprendizaje automático general      |
| ¿Cómo se configura o instala?      |  Windows: instalador msi<br/>Linux: apt-get |
| ¿Cómo se usa o ejecuta?      | Como una herramienta de línea de comandos de ruta de acceso (`C:\Program Files\VowpalWabbit\vw.exe` en Windows y `/usr/bin/vw` en Linux)    |
| Vínculos a ejemplos      | [Ejemplos de VowPal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Examples) |

## <a name="weka"></a>Weka
| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   |  Una colección de algoritmos de aprendizaje automático para las tareas de minería de datos. Los algoritmos se pueden aplicar directamente a un conjunto de datos o se pueden llamar desde su propio código de Java. Weka contiene herramientas para el preprocesamiento, la clasificación, la regresión, la agrupación en clústeres, las reglas de asociación y la visualización de datos. |
| Ediciones compatibles     | Windows, Linux     |
| Usos típicos      | Herramienta de aprendizaje automático general     |
| ¿Cómo se usa o ejecuta?      | En Windows, busque Weka en el menú **Inicio**. En Linux, inicie sesión con X2Go y, a continuación, vaya a **Applications** (Aplicaciones) > **Development** (Desarrollo) > **Weka**. |
| Vínculos a ejemplos      | [Ejemplos de Weka](https://www.cs.waikato.ac.nz/ml/weka/documentation.html) |

## <a name="xgboost"></a>XGBoost 
| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   |   Una biblioteca de potenciación del gradiente (GBDT, GBRT o GBM) rápida, portátil y distribuida para Python, R, Java, Scala, C++, etc. Se ejecuta en una única máquina, y en Apache Hadoop y Spark.    |
| Ediciones compatibles     | Windows, Linux     |
| Usos típicos      | Biblioteca de aprendizaje automático general      |
| ¿Cómo se configura o instala?      |  Se instala con la compatibilidad de GPU   |
| ¿Cómo se usa o ejecuta?      | Como biblioteca de Python (2.7 y 3.6+), paquete de R y en la herramienta de línea de comandos de ruta de acceso (`C:\dsvm\tools\xgboost\bin\xgboost.exe` para Windows y `/dsvm/tools/xgboost/xgboost` para Linux)    |
| Vínculos a ejemplos      | Se incluyen ejemplos en la máquina virtual, en `/dsvm/tools/xgboost/demo` en Linux y `C:\dsvm\tools\xgboost\demo` en Windows.   |
