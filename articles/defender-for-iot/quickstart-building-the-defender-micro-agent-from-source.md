---
title: 'Inicio rápido: Compilación del microagente de Defender a partir del código fuente (versión preliminar)'
description: En este inicio rápido, obtendrá información sobre el microagente que incluye una infraestructura que se puede usar para personalizar la distribución.
ms.date: 05/10/2021
ms.topic: quickstart
ms.openlocfilehash: b52c985619e317421a1597fee77911d9bc1e81d2
ms.sourcegitcommit: 19dfdfa85e92c6a34933bdd54a7c94e8b00eacfd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109664859"
---
# <a name="quickstart-build-the-defender-micro-agent-from-source-code-preview"></a>Inicio rápido: Compilación del microagente de Defender a partir del código fuente (versión preliminar)

El microagente incluye una infraestructura que se puede usar para personalizar la distribución. Para ver una lista de los parámetros de configuración disponibles, consulte el archivo `configs/LINUX_BASE.conf`.

Para una sola distribución, modifique el archivo `.conf` base. 

Si necesita más de una distribución, puede heredar de la configuración base y reemplazar sus valores. 

Para reemplazar los valores:

1. Cree un nuevo archivo `.dist`.

1. Agregue `CONF_DEFINE_BASE(${g_plat_config_path} LINUX_BASE.conf)` a la parte superior.
 
1. Defina los nuevos valores que desee, por ejemplo: 

    `set(ASC_LOW_PRIORITY_INTERVAL 60*60*24)` 

1. Asigne al archivo `.dist` una referencia al compilar. Por ejemplo, 

    `cmake -DCMAKE_BUILD_TYPE=Debug -Dlog_level=DEBUG -Dlog_level_cmdline:BOOL=ON -DIOT_SECURITY_MODULE_DIST_TARGET=UBUNTU1804 ..` 

## <a name="prerequisites"></a>Prerrequisitos

1. Póngase en contacto con su administrador de cuentas para acceder al código fuente de Defender para IoT.
 
1. Clone o extraiga el código fuente en una carpeta en el disco.

1. Vaya a ese directorio.

1. Extraiga los submódulos mediante el código siguiente:

    ```bash
    git submodule update --init
    ```
    
1. A continuación, extraiga los submódulos del SDK de Azure IoT con el código siguiente: 

    ```bash
    git -C deps/azure-iot-sdk-c/ submodule update –init
    ```
 

1. Agregue un permiso de ejecución y ejecute el script de configuración del entorno del desarrollador:

    ```bash
    chmod +x scripts/install_development_environment.sh && ./scripts/install_development_environment.sh 
    ```

1. Cree un directorio para las salidas de la compilación: 

    ```bash
    mkdir cmake 
    ```

1. (Opcional) Descargue e instale [VSCode](https://code.visualstudio.com/download ). 

1. (Opcional) Instale la [extensión de C/C++](https://code.visualstudio.com/docs/languages/cpp ) para VSCode: Ninguna.

## <a name="baseline-configuration-signing"></a>Firma de la configuración de línea de base 

De forma predeterminada, el agente comprueba la autenticidad de los archivos de configuración que se colocan en el disco para mitigar la alteración.

Puede detener este proceso mediante la definición de la marca del preprocesador `ASC_BASELINE_CONF_SIGN_CHECK_DISABLE`.

No se recomienda desactivar la comprobación de firma para entornos de producción. 

Si necesita una configuración diferente para escenarios de producción, póngase en contacto con el equipo de Defender para IoT. 

## <a name="building-the-defender-iot-micro-agent"></a>Compilación del microagente de Defender para IoT 

1. Abra el directorio con VSCode. 

1. Vaya al directorio `cmake`. 

1. Ejecute el siguiente comando: 

    ```bash
    cmake -DCMAKE_BUILD_TYPE=Debug -Dlog_level=DEBUG -Dlog_level_cmdline:BOOL=ON -DIOT_SECURITY_MODULE_DIST_TARGET<the appropriate distro configuration file name> .. 
    
    cmake --build . -- -j${env:NPROC}
    ```

    Por ejemplo: 

    ```bash
    cmake -DCMAKE_BUILD_TYPE=Debug -Dlog_level=DEBUG -Dlog_level_cmdline:BOOL=ON -DIOT_SECURITY_MODULE_DIST_TARGETUBUNTU1804 ..
    
    cmake --build . -- -j${env:NPROC}
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de la solución de Azure Defender para IoT](quickstart-configure-your-solution.md)
