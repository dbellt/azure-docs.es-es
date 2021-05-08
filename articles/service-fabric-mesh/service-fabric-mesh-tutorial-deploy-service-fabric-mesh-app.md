---
title: 'Tutorial: Implementación de una aplicación de Service Fabric Mesh'
description: Obtenga información sobre cómo utilizar Visual Studio para publicar una aplicación de Azure Service Fabric Mesh que consta de un sitio web de ASP.NET Core que se comunica con un servicio web de back-end.
author: georgewallace
ms.topic: tutorial
ms.date: 09/18/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: daf94087a7041fe7a21f96c56d2e7bf211bcdb45
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165648"
---
# <a name="tutorial-deploy-a-service-fabric-mesh-application"></a>Tutorial: Implementación de una aplicación Service Fabric Mesh

> [!IMPORTANT]
> Se ha retirado la versión preliminar de Azure Service Fabric Mesh. Ya no se permitirán nuevas implementaciones a través de la API de Service Fabric Mesh. La compatibilidad con las implementaciones existentes continuará hasta el 28 de abril de 2021.
>
> Para más información, consulte este artículo sobre la [retirada de la versión preliminar de Azure Service Fabric Mesh](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Este tutorial es la tercera parte de una serie y muestra cómo publicar una aplicación web de Azure Service Fabric Mesh directamente desde Visual Studio.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Publicar la aplicación en Azure mediante Visual Studio.
> * Comprobar el estado de implementación de la aplicación
> * Consulta de todas las aplicaciones implementadas actualmente en la suscripción

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * [Crear una aplicación de Service Fabric Mesh en Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [Depurar una aplicación de Service Fabric Mesh que se ejecuta en el clúster de desarrollo local](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * Implementación de una aplicación de Service Fabric Mesh
> * [Actualizar una aplicación de Service Fabric Mesh](service-fabric-mesh-tutorial-upgrade.md)
> * [Limpiar los recursos de Service Fabric Mesh](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:

* Si no tiene una suscripción a Azure, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* Asegúrese de que se haya [configurado el entorno de desarrollo](service-fabric-mesh-howto-setup-developer-environment-sdk.md), que incluye la instalación del entorno de ejecución de Service Fabric, el SDK, Docker y Visual Studio 2017.

## <a name="download-the-to-do-sample-application"></a>Descarga de la aplicación de ejemplo de tareas pendientes

Si no compiló la aplicación de ejemplo de tareas pendientes en la [segunda parte de esta serie de tutoriales](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md), puede descargarla. En una ventana Comandos, ejecute el comando siguiente para clonar el repositorio de la aplicación de ejemplo en la máquina local.

```
git clone https://github.com/azure-samples/service-fabric-mesh
```

La aplicación se encuentra en el directorio `src\todolistapp`.

## <a name="publish-to-azure"></a>Publicación en Azure

Para publicar el proyecto de Service Fabric Mesh en Azure, haga clic con el botón derecho en **todolistapp** en Visual Studio y seleccione **Publicar...**

A continuación, verá un cuadro de diálogo **Publicación de la aplicación de Service Fabric**.

![Cuadro de diálogo de publicación de Service Fabric mesh en Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

Seleccione la cuenta y la suscripción de Azure. Elija una **ubicación**. En este artículo se utiliza **Este de EE. UU**.

En **Grupo de recursos**, seleccione **\<Create New Resource Group...>** . Aparece un cuadro de diálogo donde se va a crear un nuevo grupo de recursos. En este artículo se utiliza la ubicación **Este de EE. UU.** y llama al grupo **sfmeshTutorial1RG** (si la organización tiene varias personas con la misma suscripción, elija un nombre de grupo único).  Presione **Crear** para crear el grupo de recursos y volver al cuadro de diálogo Publicar.

![Cuadro de diálogo del nuevo grupo de recursos de Service Fabric mesh en Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-new-resource-group-dialog.png)

De nuevo en el cuadro de diálogo **Publicación de la aplicación de Service Fabric**, en **Azure Container Registry**, seleccione **\<Create New Container Registry...>** (Crear una instancia de Azure Container Registry...>). En el cuadro de diálogo **Crear Registro de contenedor**, utilice un nombre único para el **nombre de la instancia de Container Registry**. Especifique una **ubicación** (en este tutorial se utiliza **Este de EE. UU.** ). Seleccione el **grupo de recursos** que creó en el paso anterior en la lista desplegable, por ejemplo, **sfmeshTutorial1RG**. Establezca **SKU** en **Básica** y, a continuación, presione **Crear** para crear el registro de contenedor privado de Azure y volver al cuadro de diálogo Publicar.

![Cuadro de diálogo del nuevo registro de contenedor de Service Fabric Mesh de Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-new-container-registry-dialog.png)

Si obtiene un error que indica que un proveedor de recursos no se ha registrado en la suscripción, puede registrarlo. Consulte en primer lugar si el proveedor de recursos está disponible para su suscripción:

```powershell
Get-AzureRmResourceProvider -ListAvailable
```

Si el proveedor del registro de contenedor (`Microsoft.ContainerRegistry`) está disponible, regístrelo desde PowerShell:

```powershell
Connect-AzureRmAccount
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ContainerRegistry
```

En el cuadro de diálogo Publicar, presione el botón **Publicar** para implementar la aplicación de Service Fabric en Azure.

Al publicar en Azure por primera vez, la imagen de Docker se inserta en Azure Container Registry (ACR), que tarda según el tamaño de la imagen. Las publicaciones posteriores del mismo proyecto serán más rápidas. Para supervisar el progreso de la implementación, seleccione el panel **Herramientas de Service Fabric** en la ventana **Salida** de Visual Studio. Una vez finalizada la implementación, la salida de **Herramientas de Service Fabric** mostrará la dirección IP y el puerto de la aplicación en forma de dirección URL.

```
Packaging Application...
Building Images...
Web1 -> C:\Code\ServiceFabricMeshApp\ToDoService\bin\Any CPU\Release\netcoreapp2.0\ToDoService.dll
Uploading the images to Azure Container Registry...
Deploying application to remote endpoint...
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

Abra un explorador web y vaya a la dirección URL para ver el sitio web que se ejecuta en Azure.

## <a name="set-up-service-fabric-mesh-cli"></a>Configuración de la CLI de Service Fabric Mesh

Puede usar Azure Cloud Shell o una instalación local de la CLI de Azure para el resto de los pasos. Instale el módulo de extensión de la CLI de Azure Service Fabric Mesh siguiendo estas [instrucciones](service-fabric-mesh-howto-setup-cli.md).

## <a name="check-application-deployment-status"></a>Comprobar el estado de implementación de la aplicación

En este momento, se ha implementado la aplicación. También puede comprobar el estado mediante el uso del comando `app show`.

El nombre de la aplicación para la aplicación del tutorial es `todolistapp`. Recopile los detalles sobre la aplicación con el siguiente comando:

```azurecli-interactive
az mesh app show --resource-group $rg --name todolistapp
```

## <a name="get-the-ip-address-of-your-deployment"></a>Obtención de la dirección de la implementación

Si desea obtener la dirección IP para la aplicación, use el siguiente comando:

```azurecli-interactive
az mesh gateway show --resource-group myResourceGroup --name todolistappGateway
```

## <a name="see-all-applications-currently-deployed-to-your-subscription"></a>Consulta de todas las aplicaciones implementadas actualmente en la suscripción

Puede usar el comando "app list" para obtener una lista de aplicaciones que ha implementado en la suscripción.

```azurecli-interactive
az mesh app list --output table
```

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:
> [!div class="checklist"]
> * Publicar la aplicación en Azure
> * Comprobar el estado de implementación de la aplicación
> * Consultar toda la aplicación que ha implementado en la suscripción

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Actualizar una aplicación de Service Fabric Mesh](service-fabric-mesh-tutorial-upgrade.md)

[azure-cli-install]: /cli/azure/install-azure-cli
