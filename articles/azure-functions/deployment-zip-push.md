---
title: Implementación para insertar archivos ZIP en Azure Functions
description: Use las funciones de implementación de archivos ZIP del servicio de implementación de Kudu para publicar sus instancias de Azure Functions.
ms.topic: conceptual
ms.date: 08/12/2018
ms.openlocfilehash: a8a8b6e0ad1cd70ae6fe2f0025afcba6fc9e44a5
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110465840"
---
# <a name="zip-deployment-for-azure-functions"></a>Implementación de archivos ZIP en Azure Functions

En este artículo se describe la manera de implementar los archivos de proyecto de la aplicación de función en Azure desde un archivo ZIP (comprimido). Aprenda a realizar una implementación de inserción mediante el uso de la CLI de Azure y con las API de REST. [Azure Functions Core Tools](functions-run-local.md) también usa estas API de implementación al publicar un proyecto local en Azure.

Azure Functions tiene toda una gama de opciones de implementación e integración continuas que se proporcionan con Azure App Service. Para más información, vea [Implementación continua para Azure Functions](functions-continuous-deployment.md).

Para acelerar el desarrollo, puede que le resulte más fácil implementar los archivos de proyecto de la aplicación de funciones directamente desde un archivo ZIP. La API de implementación de ZIP toma el contenido de un archivo ZIP y lo extrae en la carpeta `wwwroot` de la aplicación de función. Esta implementación del archivo .zip usa el servicio Kudu que permite realizar implementaciones basadas en integraciones continuas, entre las que se incluye:

+ La eliminación de archivos que se omitieron en implementaciones anteriores.
+ La personalización de la implementación, incluyendo la ejecución de scripts de implementación.
+ Registros de implementación.
+ Los desencadenadores de la función de sincronización de una aplicación de función en un [plan de consumo](functions-scale.md).

Para más información, vea la [referencia sobre implementaciones con archivos ZIP](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file).

## <a name="deployment-zip-file-requirements"></a>Requisitos de los .zip de implementación

El archivo ZIP que use para la implementación de inserción debe contener todos los archivos necesarios para ejecutar la función.

>[!IMPORTANT]
> Cuando realice la implementación del archivo ZIP, los archivos de cualquier otra implementación existente que no se encuentren en el archivo ZIP se eliminarán de la aplicación de función.  

[!INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

Una aplicación de función incluye todos los archivos y carpetas del directorio `wwwroot`. Una implementación de archivo .zip incluye el contenido del directorio `wwwroot`, pero no el propio directorio. Al implementar un proyecto de la biblioteca de clases de C#, debe incluir las dependencias y los archivos de biblioteca compilados en una subcarpeta `bin` del paquete ZIP.

Al desarrollar contenido en un equipo local, puede crear manualmente un archivo .zip de la carpeta del proyecto de la aplicación de funciones mediante la funcionalidad de compresión .zip integrada o herramientas de terceros.

## <a name="download-your-function-app-files"></a>Descarga de los archivos de aplicación de función

Si creó las funciones mediante el editor de Azure Portal, puede descargar el proyecto de la aplicación de funciones existente como un archivo .zip de una de estas maneras:

+ **Desde Azure Portal**

  1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la aplicación de función.

  2. En la pestaña **Introducción**, seleccione **Descargar contenido de la aplicación**. Seleccione las opciones de descarga y, a continuación, seleccione **Descargar**.

      ![Descargue el proyecto de aplicación de función.](./media/deployment-zip-push/download-project.png)

     El archivo .zip descargado tiene el formato correcto para volver a publicarlo en la aplicación de función mediante la implementación de inserción de archivos .zip. La descarga del portal también puede agregar los archivos necesarios para abrir la aplicación de función directamente en Visual Studio.

+ **Uso de las API REST:**

    Use la siguiente API GET de implementación para descargar los archivos del proyecto `<function_app>`: 

    ```http
    https://<function_app>.scm.azurewebsites.net/api/zip/site/wwwroot/
    ```

    Al incluir `/site/wwwroot/` se garantiza que el archivo ZIP solo incorpora los archivos de proyecto de la aplicación de función y no el sitio entero. Si aún no ha iniciado sesión en Azure, se le pedirá que lo haga.  

Igualmente, también puede descargar un archivo .zip desde un repositorio de GitHub. Al descargar un repositorio de GitHub como un archivo ZIP, GitHub agrega un nivel de carpeta adicional para la rama. Este nivel de carpetas adicional significa que no puede implementar el archivo .zip directamente ya que se descargó desde GitHub. Si usa un repositorio de GitHub para mantener la aplicación de función, debe usar la [integración continua](functions-continuous-deployment.md) para implementar la aplicación.  

## <a name="deploy-by-using-azure-cli"></a><a name="cli"></a>Implementación con la CLI de Azure

Puede usar la CLI de Azure para desencadenar una implementación de inserción. Para implementar el archivo .zip en la aplicación de función, use el comando [az functionapp deployment source config-zip](/cli/azure/functionapp/deployment/source#az_functionapp_deployment_source_config_zip). Para usar este comando, debe usar la versión 2.0.21 o posterior de la CLI de Azure. Para ver qué versión de CLI de Azure está usando, use el comando `az --version`.

En el siguiente comando, reemplace el marcador de posición `<zip_file_path>` por la ruta de acceso a la ubicación del archivo .zip. Asimismo, reemplace `<app_name>` por el nombre único de su aplicación de funciones y `<resource_group>` por el nombre del grupo de recursos.

```azurecli-interactive
az functionapp deployment source config-zip -g <resource_group> -n \
<app_name> --src <zip_file_path>
```

Este comando implementa los archivos de proyecto del archivo .zip descargado en la aplicación de función en Azure. A continuación, se reinicia la aplicación. Para ver la lista de implementaciones de esta aplicación de función, debe usar las API de REST.

Cuando se usa la CLI de Azure en el equipo local, `<zip_file_path>` es la ruta de acceso al archivo .zip en el equipo. También puede ejecutar la CLI de Azure en [Azure Cloud Shell](../cloud-shell/overview.md). Cuando use Cloud Shell, recuerde que debe cargar primero el archivo .zip de implementación en la cuenta de Azure Files que esté asociada con Cloud Shell. En ese caso, `<zip_file_path>` será la ubicación de almacenamiento que utiliza la cuenta de Cloud Shell. Para obtener más información, consulte [Persistencia de archivos en Azure Cloud Shell](../cloud-shell/persisting-shell-storage.md).

[!INCLUDE [app-service-deploy-zip-push-rest](../../includes/app-service-deploy-zip-push-rest.md)]

## <a name="run-functions-from-the-deployment-package"></a>Ejecución de funciones desde el paquete de implementación

También puede ejecutar las funciones directamente desde el archivo del paquete de implementación. Este método omite el paso de implementación de la copia de archivos del paquete en el directorio `wwwroot` de la aplicación de función. En su lugar, Functions Runtime monta el archivo del paquete y el contenido el directorio `wwwroot` será de solo lectura.  

La implementación de ZIP se integra con esta característica, que se puede habilitar al configurar la opción de la aplicación de función `WEBSITE_RUN_FROM_PACKAGE` en un valor de `1`. Para más información, consulte [Run your functions from a deployment package file](run-functions-from-deployment-package.md) (Ejecución de Azure Functions desde un archivo de paquete de implementación).

[!INCLUDE [app-service-deploy-zip-push-custom](../../includes/app-service-deploy-zip-push-custom.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Implementación continua para Azure Functions](functions-continuous-deployment.md)

[.zip push deployment reference topic]: https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file
