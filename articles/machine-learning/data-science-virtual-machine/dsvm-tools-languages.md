---
title: Idiomas compatibles
titleSuffix: Azure Data Science Virtual Machine
description: Lenguajes de programación admitidos y herramientas relacionadas que se instalaron previamente en Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: data-science-vm
ms.custom: devx-track-python
author: timoklimmer
ms.author: tklimmer
ms.topic: conceptual
ms.date: 05/12/2021
ms.openlocfilehash: fcb2a4d9860687fb18409666f81839002329d54e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110070905"
---
# <a name="languages-supported-on-the-data-science-virtual-machine"></a>Lenguajes admitidos en Data Science Virtual Machine 

Data Science Virtual Machine (DSVM) incorpora varios lenguajes predefinidos y herramientas de desarrollo para compilar sus aplicaciones de inteligencia artificial (IA). Estos son algunos de los más importantes.

## <a name="python"></a>Python

| Category | Value |
|--|--|
| Versiones de lenguajes admitidas | Python 3.8 |
| Ediciones de DSVM admitidas | Windows Server 2019, Ubuntu 18.04 |
| ¿Cómo se configura/instala en DSVM? | Hay varios entornos `conda` en los que cada una de ellas tiene diferentes paquetes de Python preinstalados. Para enumerar todos los entornos disponibles en la máquina, ejecute `conda env list`. |

### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla

* En un símbolo del sistema:

  Abra un símbolo del sistema y use uno de los métodos siguientes, en función de la versión de Python que quiera ejecutar:

    ```
    conda activate <conda_environment_name>
    python --version
    ```
    
* Uso en un IDE:

  Las imágenes de DSVM tienen varios IDE instalados, como VS.Code o PyCharm. Puede usarlos para editar, ejecutar y depurar los scripts de Python.

* Uso en Jupyter Lab:

  Abra una pestaña de inicio en Jupyter Lab y seleccione el tipo y el kernel del nuevo documento. Si quiere que el documento se coloque en una carpeta determinada, vaya primero a esa carpeta en el Explorador de archivos del lado izquierdo.

* Instalación de paquetes de Python:

  Para instalar un nuevo paquete, primero debe activar el entorno correcto. El entorno es el lugar donde se instalará el nuevo paquete y el paquete solo estará disponible en ese entorno.

  Para activar un entorno, ejecute `conda activate <environment_name>`. Una vez activado el entorno, puede usar un administrador de paquetes como `conda` o `pip` para instalar o actualizar un paquete.

  Como alternativa, si usa Jupyter, también puede instalar paquetes directamente mediante la ejecución de `!pip install --upgrade <package_name>` en una celda.

## <a name="r"></a>R

| Category | Value |
|--|--|
| Versiones de lenguajes admitidas | CRAN R 4.0.5 |
| Ediciones de DSVM admitidas | Linux y Windows |

### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla

* En un símbolo del sistema:

  Abra un símbolo del sistema y escriba `R`.

* Uso en un IDE:

  Para editar scripts de R en un IDE, puede usar RStudio, que está instalado en las imágenes de DSVM de forma predeterminada.

* Uso en Jupyter Lab

  Abra una pestaña de inicio en Jupyter Lab y seleccione el tipo y el kernel del nuevo documento. Si quiere que el documento se coloque en una carpeta determinada, vaya primero a esa carpeta en el Explorador de archivos del lado izquierdo.

* Instalación de paquetes de R:

  Puede instalar nuevos paquetes de R mediante la función `install.packages()` o mediante RStudio.

## <a name="julia"></a>Julia

| Category | Value |
| ------------- | ------------- |
| Versiones de lenguajes admitidas | 1.0.5 |
| Ediciones de DSVM admitidas      | Linux y Windows     |


### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla    

* Ejecución en un símbolo del sistema

  Abra un símbolo del sistema y ejecute `julia`.

* Uso en Jupyter:

  Abra una pestaña de inicio en Jupyter y seleccione el tipo y el kernel del nuevo documento. Si quiere que el documento se coloque en una carpeta determinada, vaya primero a esa carpeta en el Explorador de archivos del lado izquierdo.

* Instalación de paquetes de Julia:

  Puede usar los comandos del administrador de paquetes de Julia, como `Pkg.add()`, para instalar o actualizar paquetes.


## <a name="other-languages"></a>Otros idiomas

**C#** : disponible en Windows y accesible mediante Visual Studio Community Edition o en `Developer Command Prompt for Visual Studio`, donde solo puede ejecutar el comando `csc`.

**Java**: OpenJDK está disponible en las ediciones para Windows y Linux de DSVM y se establece en la ruta de acceso. Para usar Java, escriba el comando `javac` o `java` en el símbolo del sistema en Windows o en el shell bash de Linux.

**Node.js**: Node.js está disponible en la edición para Windows y Linux de DSVM y se establece en la ruta de acceso. Para acceder a Node.js, escriba el comando `node` o `npm` en el símbolo del sistema en Windows o en el shell bash de Linux. En Windows, se instala la extensión de Visual Studio para las herramientas de Node.js para proporcionar un IDE gráfico con el fin de desarrollar la aplicación Node.js.

**F#** : disponible en Windows y accesible mediante Visual Studio Community Edition o en `Developer Command Prompt for Visual Studio`, donde solo puede ejecutar el comando `fsc`.
