---
title: 'Inicio rápido: Crear una aplicación web de Node.js'
description: Implementación de su primera aplicación Hola mundo de Node.js en Azure App Service en cuestión de minutos.
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 08/01/2020
ms.custom: mvc, devcenter, seodec18
zone_pivot_groups: app-service-platform-windows-linux
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 7cd9add699d9bd4a3eff9cdcf1e65af0d754b70a
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/03/2021
ms.locfileid: "101747791"
---
# <a name="create-a-nodejs-web-app-in-azure"></a>Creación de una aplicación web de Node.js en Azure

Introducción a <abbr title="Un servicio basado en HTTP para hospedar aplicaciones web, API REST y aplicaciones de back-end para dispositivos móviles.">Azure App Service</abbr> mediante la creación de una aplicación de Node.js/Express localmente o con Visual Studio Code y, a continuación, implemente la aplicación en la nube de Azure. Dado que usa un <abbr title="En Azure App Service, un nivel básico mediante el que la aplicación se ejecuta en las mismas máquinas virtuales que otras aplicaciones, incluidas las aplicaciones de otros clientes. Este nivel está pensado para las fases de desarrollo y pruebas.">nivel Gratis,</abbr>puede incurrir en costos al realizar el inicio rápido.

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

- Una cuenta de Azure con una suscripción activa. <abbr title="Una suscripción de Azure es un contenedor lógico que se usa para aprovisionar recursos en Azure. Contiene los detalles de todos los recursos, como máquinas virtuales, bases de datos, etc.">subscription</abbr>. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension).
- <a href="https://git-scm.com/" target="_blank">Instalación de Git</a>
- [Node.js y npm](https://nodejs.org) Ejecute el comando `node --version` para comprobar que Node.js está instalado.
- [Visual Studio Code](https://code.visualstudio.com/).
- La [extensión Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) para Visual Studio Code.

[Notificar un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&prepare-your-environment)




<br>
<hr/>

## <a name="2-clone-and-run-a-local-nodejs-application"></a>2. Clonación y ejecución de una aplicación de Node.js local

1. En el equipo local, abra un terminal y clone el repositorio de ejemplo:

    ```bash
    git clone https://github.com/Azure-Samples/nodejs-docs-hello-world
    ```

1. Vaya a la carpeta de la nueva aplicación:

    ```bash
    cd nodejs-docs-hello-world
    ```

1. Instale las dependencias:

    ```bash
    npm install
    ```

1. Inicie la aplicación para probarla localmente:

    ```bash
    npm start
    ```
    
1. Abra el explorador web y vaya a `http://localhost:1337`. El explorador debe mostrar "¡Hola mundo!".

1. Presione <kbd>Ctrl</kbd> + <kbd>C</kbd> en el terminal para detener el servidor.

[Notificar un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&prepare-your-environment)


<br>
<hr/>




<!-- VS Code extension works differently for Windows/Linus - Step 3 -->

::: zone pivot="platform-windows"  

[!INCLUDE [Windows](./includes/quickstart-nodejs-uiex-windows.md)]


::: zone-end

::: zone pivot="platform-linux"  

[!INCLUDE [Windows](./includes/quickstart-nodejs-uiex-linux.md)]

::: zone-end


## <a name="4-viewing-logs-from-visual-studio-code"></a>4. Ver registros desde Visual Studio Code

Visualización del registro <abbr title="Las llamadas realizadas a `console.log` en la aplicación se muestran en la ventana de salida en Visual Studio Code.">log</abbr> de la aplicación de App Service en ejecución.

1. Busque la aplicación en el explorador de **AZURE APP SERVICE**, haga clic con el botón derecho en el nombre de la aplicación y seleccione **Inicio de transmisión de registros**.

1. Se abre la ventana de salida de Visual Studio Code.

    ![Visualización de los registros de streaming](./media/quickstart-nodejs/view-logs.png)

    :::image type="content" source="./media/quickstart-nodejs/enable-restart.png" alt-text="Captura de pantalla del símbolo del sistema de VS Code para habilitar el registro de archivos y reiniciar la aplicación web con el botón Sí seleccionado.":::

1. Después de unos segundos, verá un mensaje donde se indica que está conectado al servicio de streaming de registros. 
1. Actualice la página varias veces para ver más actividad.

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    2020-09-20 20:37:39.574 INFO  - Initiating warmup request to container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node
    2020-09-20 20:37:55.011 INFO  - Waiting for response to warmup request for container msdocs-vscode-node_2_00ac292a. Elapsed time = 15.4373071 sec
    2020-09-20 20:38:08.233 INFO  - Container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node initialized successfully and is ready to serve requests.
    2020-09-20T20:38:21  Startup Request, url: /Default.cshtml, method: GET, type: request, pid: 61,1,7, SCM_SKIP_SSL_VALIDATION: 0, SCM_BIN_PATH: /opt/Kudu/bin, ScmType: None
    </pre>

<br>

[Notificar un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&prepare-your-environment)

<br>
<hr/>

## <a name="5-clean-up-resources"></a>5. Limpieza de recursos

Busque la aplicación en el explorador de **AZURE APP SERVICE**, haga clic con el botón derecho en el nombre de la aplicación y seleccione **Eliminar**. 

:::image type="content" source="./media/quickstart-nodejs/delete-resource-ieux.png" alt-text=" ":::Busque la aplicación en el explorador de `**`AZURE APP SERVICE`**`, haga clic con el botón derecho en el nombre de la aplicación y seleccione `Eliminar`

## <a name="next-steps"></a>Pasos siguientes

Ha completado correctamente este inicio rápido. Para implementar los cambios en esta aplicación, siga el mismo proceso y seleccione la aplicación existente, en lugar de crear otra.

A continuación, vea otras extensiones de Azure.

* [Cosmos DB](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Funciones de Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Herramientas de Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Herramientas de la CLI de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

También puede conseguirlas todas si instala el paquete de extensiones [Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).
