---
title: CI/CD con Azure Pipelines y plantillas
description: Describe cómo configurar la integración continua en Azure Pipelines mediante plantillas de Azure Resource Manager. Muestra cómo usar un script de PowerShell o copiar archivos en una ubicación de almacenamiento provisional e implementar desde allí.
ms.topic: conceptual
ms.date: 03/09/2021
ms.openlocfilehash: 8264fd53bde168972cdddd9bdf15fccc8d536651
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111984471"
---
# <a name="integrate-arm-templates-with-azure-pipelines"></a>Integración de plantillas de ARM con Azure Pipelines

Puede integrar plantillas de Azure Resource Manager (plantillas de Resource Manager) con Azure Pipelines para la integración continua e implementación continua (CI/CD). En este artículo, aprenderá dos formas avanzadas adicionales de implementar plantillas con Azure Pipelines.

## <a name="select-your-option"></a>Selección de la opción

Antes de avanzar en este artículo, vamos a tener en cuenta las distintas opciones para implementar una plantilla de ARM desde una canalización.

* **Usar la tarea de implementación de plantilla de ARM**. Esta opción es la más sencilla. Este enfoque funciona cuando se desea implementar una plantilla directamente desde un repositorio. Esta opción no se trata en este artículo, sino que se trata en el tutorial [Integración continua de plantillas de ARM con Azure Pipelines](deployment-tutorial-pipeline.md). Muestra cómo usar la [tarea de implementación de plantilla de ARM](https://github.com/microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceManagerTemplateDeploymentV3/README.md) para implementar una plantilla desde el repositorio de GitHub.

* **Agregar una tarea que ejecute un script de Azure PowerShell**. Esta opción tiene la ventaja de ofrecer coherencia en todo el ciclo de vida de desarrollo debido a que puede usar el mismo script que usó al ejecutar pruebas locales. El script implementa la plantilla, pero también puede realizar otras operaciones como obtener valores para usarlos como parámetros. Esta opción se muestra en este artículo. Consulte [Tarea de Azure PowerShell](#azure-powershell-task).

   Visual Studio proporciona el [proyecto de grupo de recursos de Azure](create-visual-studio-deployment-project.md) que incluye un script de PowerShell. El script agrega los artefactos del proyecto al "stage" en una cuenta de almacenamiento a la que Resource Manager puede acceder. Los artefactos son elementos del proyecto tales como plantillas vinculadas, scripts y archivos binarios de aplicación. Si desea seguir usando el script del proyecto, use la tarea Script de PowerShell mostrada en este artículo.

* **Agregar tareas para copiar e implementar tareas**. Esta opción ofrece una alternativa conveniente al script del proyecto. Configure dos tareas en la canalización. Una tarea organiza los artefactos en una ubicación accesible. La otra tarea implementa la plantilla desde esa ubicación. Esta opción se muestra en este artículo. Consulte [Tareas de copia y de implementación](#copy-and-deploy-tasks).

## <a name="prepare-your-project"></a>Preparación del proyecto

En este artículo se presupone que su plantilla de Resource Manager y la organización de Azure DevOps están listas para crear la canalización. Los pasos siguientes muestran cómo asegurarse de que está listo:

* Tiene una organización de Azure DevOps. Si no tiene ninguna, [cree una gratis](/azure/devops/pipelines/get-started/pipelines-sign-up). Si su equipo ya tiene una organización de Azure DevOps, compruebe que es administrador del proyecto de Azure DevOps que quiere utilizar.

* Ha configurado una [conexión al servicio](/azure/devops/pipelines/library/connect-to-azure) en su suscripción de Azure. Las tareas en la canalización se ejecutan con la identidad de la entidad de servicio. Para conocer los pasos para crear la conexión, consulte [Crear un proyecto de DevOps](deployment-tutorial-pipeline.md#create-a-devops-project).

* Tiene una [plantilla de Resource Manager](quickstart-create-templates-use-visual-studio-code.md) que define la estructura del proyecto.

## <a name="create-pipeline"></a>Creación de una canalización

1. Si no ha agregado una canalización anteriormente, tiene que crear una. En la organización de Azure DevOps, seleccione **Canalizaciones** y **Nueva canalización**.

   ![Adición de una nueva canalización](./media/add-template-to-azure-pipelines/new-pipeline.png)

1. Especifique dónde está almacenado el código. En la imagen siguiente se muestra la selección de **GIT de Azure Repos**.

   ![Selección del origen del código](./media/add-template-to-azure-pipelines/select-source.png)

1. Desde ese origen, seleccione el repositorio que tiene el código para el proyecto.

   ![Selección del repositorio](./media/add-template-to-azure-pipelines/select-repo.png)

1. Seleccione el tipo de canalización que quiere crear. Puede seleccionar **Canalización inicial**.

   ![Selección de la canalización](./media/add-template-to-azure-pipelines/select-pipeline.png)

Está listo para agregar una tarea de Azure PowerShell o las tareas de copia de archivos y de implementación.

## <a name="azure-powershell-task"></a>Tarea de Azure PowerShell

En esta sección se muestra cómo configurar la implementación continua mediante el uso de una sola tarea que ejecuta el script de PowerShell en el proyecto. Si necesita un script de PowerShell que implemente una plantilla, consulte [Deploy-AzTemplate.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzTemplate.ps1) o [Deploy-AzureResourceGroup.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzureResourceGroup.ps1).

El siguiente archivo YAML crea una [tarea de Azure PowerShell](/azure/devops/pipelines/tasks/deploy/azure-powershell):

```yml
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzurePowerShell@5
  inputs:
    azureSubscription: 'script-connection'
    ScriptType: 'FilePath'
    ScriptPath: './Deploy-AzTemplate.ps1'
    ScriptArguments: -Location 'centralus' -ResourceGroupName 'demogroup' -TemplateFile templates\mainTemplate.json
    azurePowerShellVersion: 'LatestVersion'
```

Al establecer la tarea en `AzurePowerShell@5`, la canalización usa el [Módulo Az](/powershell/azure/new-azureps-module-az). Si usa el módulo AzureRM en el script, establezca la tarea en `AzurePowerShell@3`.

```yaml
steps:
- task: AzurePowerShell@3
```

En `azureSubscription`, proporcione el nombre de la conexión de servicio que creó.

```yaml
inputs:
    azureSubscription: '<your-connection-name>'
```

Para `scriptPath`, proporcione la ruta de acceso relativa desde el archivo de canalización hasta el script. Puede buscar la ruta de acceso en el repositorio.

```yaml
ScriptPath: '<your-relative-path>/<script-file-name>.ps1'
```

En `ScriptArguments`, proporcione los parámetros necesarios para el script. En el ejemplo siguiente se muestran algunos de los parámetros de un script, pero tendrá que personalizar los parámetros de su script.

```yaml
ScriptArguments: -Location 'centralus' -ResourceGroupName 'demogroup' -TemplateFile templates\mainTemplate.json
```

Al seleccionar **Guardar**, la canalización de compilación se ejecuta automáticamente. Vuelva al resumen de la canalización de compilación y vea el estado.

![Vista de resultados](./media/add-template-to-azure-pipelines/view-results.png)

Puede seleccionar la canalización en ejecución para ver información detallada sobre las tareas. Cuando haya terminado, verá los resultados de cada paso.

## <a name="copy-and-deploy-tasks"></a>Tareas de copia y de implementación

En esta sección se muestra cómo configurar la implementación continua mediante el uso de dos tareas. En la primera tarea se organizan los artefactos en una cuenta de almacenamiento y en la segunda tarea se implementa la plantilla.

Para copiar archivos en una cuenta de almacenamiento, a la entidad de servicio de la conexión de servicio se le debe asignar el rol Colaborador de datos de Storage Blob o Propietario de datos de Storage Blob. Para más información, consulte [Introducción a AzCopy](../../storage/common/storage-use-azcopy-v10.md).

El siguiente código YAML muestra la [tarea de copia de archivos de Azure](/azure/devops/pipelines/tasks/deploy/azure-file-copy).

```yml
trigger:
- master

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureFileCopy@4
  inputs:
    SourcePath: 'templates'
    azureSubscription: 'copy-connection'
    Destination: 'AzureBlob'
    storage: 'demostorage'
    ContainerName: 'projecttemplates'
  name: AzureFileCopy
```

Hay varias partes de esta tarea que debe revisar para su entorno. El elemento `SourcePath` indica la ubicación de los artefactos en relación con el archivo de canalización.

```yaml
SourcePath: '<path-to-artifacts>'
```

En `azureSubscription`, proporcione el nombre de la conexión de servicio que creó.

```yaml
azureSubscription: '<your-connection-name>'
```

En cuanto al almacenamiento y el nombre del contenedor, proporcione los nombres de la cuenta de almacenamiento y del contenedor que quiere usar para almacenar los artefactos. La cuenta de almacenamiento debe existir.

```yaml
storage: '<your-storage-account-name>'
ContainerName: '<container-name>'
```

Después de crear la tarea de copia de archivos, estará listo para agregar la tarea a fin de implementar la plantilla preconfigurada.

El siguiente código YAML muestra la [tarea de implementación de la plantilla de Resource Manager](https://github.com/microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceManagerTemplateDeploymentV3/README.md):

```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'copy-connection'
    subscriptionId: '00000000-0000-0000-0000-000000000000'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'demogroup'
    location: 'West US'
    templateLocation: 'URL of the file'
    csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
    csmParametersFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.parameters.json$(AzureFileCopy.StorageContainerSasToken)'
    deploymentMode: 'Incremental'
    deploymentName: 'deploy1'
```

Hay varias partes de esta tarea que debe revisar con mayor detalle.

- `deploymentScope`: seleccione el ámbito de implementación entre las opciones: `Management Group`, `Subscription` y `Resource Group`. Para más información sobre los ámbitos, consulte [Ámbitos de implementación](deploy-rest.md#deployment-scope).

- `azureResourceManagerConnection`: proporcione el nombre de la conexión de servicio que creó.

- `subscriptionId`: especifique el identificador de la suscripción de destino. Esta propiedad solo se aplica al ámbito de implementación del grupo de recursos y de la implementación de la suscripción.

- `resourceGroupName` y `location`: especifique el nombre y la ubicación del grupo de recursos en donde quiere realizar la implementación. La tarea crea el grupo de recursos si no existe.

   ```yml
   resourceGroupName: '<resource-group-name>'
   location: '<location>'
   ```

- `csmFileLink`: proporcione el vínculo para la plantilla preconfigurada. Al establecer el valor, use variables devueltas desde la tarea de copia de archivos. El ejemplo siguiente se vincula a una plantilla denominada mainTemplate.json. La carpeta denominada **Plantillas** se incluye porque allí es donde la tarea de copia de archivos copió el archivo. En la canalización, proporcione la ruta de acceso a la plantilla y el nombre de la plantilla.

   ```yml
   csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
   ```

La canalización tiene el siguiente aspecto:

```yml
trigger:
- master

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureFileCopy@4
  inputs:
    SourcePath: 'templates'
    azureSubscription: 'copy-connection'
    Destination: 'AzureBlob'
    storage: 'demostorage'
    ContainerName: 'projecttemplates'
  name: AzureFileCopy
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'copy-connection'
    subscriptionId: '00000000-0000-0000-0000-000000000000'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'demogroup'
    location: 'West US'
    templateLocation: 'URL of the file'
    csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
    csmParametersFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.parameters.json$(AzureFileCopy.StorageContainerSasToken)'
    deploymentMode: 'Incremental'
    deploymentName: 'deploy1'
```

Al seleccionar **Guardar**, la canalización de compilación se ejecuta automáticamente. Vuelva al resumen de la canalización de compilación y vea el estado.

## <a name="next-steps"></a>Pasos siguientes

* Para usar la operación what-if en una canalización, consulte [Prueba de plantillas de Resouce Manager con What-If en una canalización](https://4bes.nl/2021/03/06/test-arm-templates-with-what-if/).
* Para obtener información sobre cómo usar plantillas de Resource Manager con Acciones de GitHub, consulte [Implementación de plantillas de Azure Resource Manager mediante Acciones de GitHub](deploy-github-actions.md).
