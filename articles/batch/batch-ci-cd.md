---
title: Uso de Azure Pipelines para compilar e implementar soluciones HPC
description: Aprenda a implementar una canalización de compilación/versión de una aplicación HPC que se ejecuta en Azure Batch.
author: chrisreddington
ms.author: chredd
ms.date: 03/04/2021
ms.topic: how-to
ms.openlocfilehash: 8ea126637679974035d62cc415c4fd07f597d53f
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111961873"
---
# <a name="use-azure-pipelines-to-build-and-deploy-hpc-solutions"></a>Usar Azure Pipelines para compilar e implementar soluciones HPC

Las herramientas suministradas por Azure DevOps pueden traducirse en la compilación y la comprobación automatizadas de soluciones de informática de alto rendimiento (HPC). [Azure Pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines) proporciona una serie de procesos de integración continua (CI) e implementación continua (CD) para compilar, implementar, probar y supervisar software. Estos procesos aceleran la entrega de software, lo que permite centrarse en el código, más que en la infraestructura de soporte y las operaciones.

En este artículo se explica cómo configurar procesos de CI/CD con [Azure Pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines) para las soluciones de HPC implementadas en Azure Batch.

## <a name="prerequisites"></a>Requisitos previos

Para realizar los pasos descritos en este artículo, se necesita una [organización de Azure DevOps](/azure/devops/organizations/accounts/create-organization). También debe [crear un proyecto en Azure DevOps](/azure/devops/organizations/projects/create-project).

Resulta útil tener un conocimiento básico del [control de código fuente](/azure/devops/user-guide/source-control) y la [sintaxis de la plantilla de Azure Resource Manager](../azure-resource-manager/templates/syntax.md) antes de empezar.

## <a name="create-an-azure-pipeline"></a>Crear una canalización de Azure Pipelines

En este ejemplo, creará una canalización de compilación y versión para implementar una infraestructura de Azure Batch y lanzar un paquete de aplicación. Suponiendo que el código se desarrolla localmente, este es el flujo de implementación general:

![Diagrama que muestra el flujo de implementación en la canalización.](media/batch-ci-cd/DeploymentFlow.png)

En este ejemplo se usan varias plantillas de Azure Resource Manager y archivos binarios existentes. Puede copiar estos ejemplos en su repositorio e insertarlos en Azure DevOps.

### <a name="understand-the-azure-resource-manager-templates"></a>Descripción de las plantillas de Azure Resource Manager

En este ejemplo se usan varias plantillas de Azure Resource Manager para implementar la solución. Se usan tres plantillas de funcionalidad (similares a unidades o módulos) para implementar una parte específica de la funcionalidad. Después, se usa una plantilla de solución de un extremo a otro (deployment.jsen) para implementar esas plantillas de funcionalidad subyacentes. Esta [estructura de plantillas vinculadas](../azure-resource-manager/templates/deployment-tutorial-linked-template.md) permite que cada plantilla de funcionalidad se pueda probar y reutilizar individualmente en las soluciones.

![Diagrama que muestra una estructura de plantillas vinculadas mediante plantillas de Azure Resource Manager.](media/batch-ci-cd/ARMTemplateHierarchy.png)

Esta plantilla define una cuenta de Azure Storage, que es necesaria para implementar la aplicación en la cuenta de Batch. Para obtener información detallada, consulte la [guía de referencia de plantillas de Resource Manager para los tipos de recursos de Microsoft.Storage](/azure/templates/microsoft.storage/allversions).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[parameters('accountName')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "apiVersion": "2018-02-01",
            "location": "[resourceGroup().location]",
            "properties": {}
        }
    ],
    "outputs": {
        "blobEndpoint": {
          "type": "string",
          "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))).primaryEndpoints.blob]"
        },
        "resourceId": {
          "type": "string",
          "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))]"
        }
    }
}
```

La siguiente plantilla define una [cuenta de Azure Batch](accounts.md). La cuenta de Batch actúa como plataforma donde ejecutar muchas aplicaciones en [grupos](nodes-and-pools.md#pools). Para obtener información detallada, consulte la [guía de referencia de plantillas de Resource Manager para los tipos de recursos de Microsoft.Batch](/azure/templates/microsoft.batch/allversions).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
            }
        },
        "storageAccountId": {
           "type": "string",
           "metadata": {
                "description": "ID of the Azure Storage Account"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "[parameters('batchAccountName')]",
            "type": "Microsoft.Batch/batchAccounts",
            "apiVersion": "2017-09-01",
            "location": "[resourceGroup().location]",
            "properties": {
              "poolAllocationMode": "BatchService",
              "autoStorage": {
                  "storageAccountId": "[parameters('storageAccountId')]"
              }
            }
          }
    ],
    "outputs": {}
}
```

La siguiente plantilla crea un grupo de Batch en la cuenta de Batch. Para obtener información detallada, consulte la [guía de referencia de plantillas de Resource Manager para los tipos de recursos de Microsoft.Batch](/azure/templates/microsoft.batch/allversions).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
           }
        },
        "batchAccountPoolName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account Pool"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "name": "[concat(parameters('batchAccountName'),'/', parameters('batchAccountPoolName'))]",
            "type": "Microsoft.Batch/batchAccounts/pools",
            "apiVersion": "2017-09-01",
            "properties": {
                "deploymentConfiguration": {
                    "virtualMachineConfiguration": {
                        "imageReference": {
                            "publisher": "Canonical",
                            "offer": "UbuntuServer",
                            "sku": "18.04-LTS",
                            "version": "latest"
                        },
                        "nodeAgentSkuId": "batch.node.ubuntu 18.04"
                    }
                },
                "vmSize": "Standard_D1_v2"
            }
          }
    ],
    "outputs": {}
}
```

La plantilla final actúa como un orquestador, implementando las tres plantillas de funcionalidad subyacentes.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "templateContainerUri": {
           "type": "string",
           "metadata": {
                "description": "URI of the Blob Storage Container containing the Azure Resource Manager templates"
            }
        },
        "templateContainerSasToken": {
           "type": "string",
           "metadata": {
                "description": "The SAS token of the container containing the Azure Resource Manager templates"
            }
        },
        "applicationStorageAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
            }
         },
        "batchAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account"
            }
         },
         "batchAccountPoolName": {
             "type": "string",
             "metadata": {
                  "description": "Name of the Azure Batch Account Pool"
              }
          }
    },
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "storageAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/storageAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "accountName": {"value": "[parameters('applicationStorageAccountName')]"}
                }
            }
        },  
        {
            "apiVersion": "2017-05-10",
            "name": "batchAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "storageAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "storageAccountId": {"value": "[reference('storageAccountDeployment').outputs.resourceId.value]"}
                }
            }
        },
        {
            "apiVersion": "2017-05-10",
            "name": "poolDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "batchAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccountPool.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "batchAccountPoolName": {"value": "[parameters('batchAccountPoolName')]"}
                }
            }
        }
    ],
    "outputs": {}
}
```

### <a name="understand-the-hpc-solution"></a>Descripción de la solución HPC

Como se mencionó anteriormente, en este ejemplo se usan varias plantillas de Azure Resource Manager y archivos binarios existentes. Puede copiar estos ejemplos en su repositorio e insertarlos en Azure DevOps.

En esta solución, ffmpeg se utiliza como el paquete de aplicación. Puede [descargar el paquete ffmpeg](https://github.com/GyanD/codexffmpeg/releases/tag/4.3.1-2020-11-08) si aún no lo tiene.

![Captura de pantalla de la estructura de repositorio.](media/batch-ci-cd/git-repository.jpg)

En este repositorio hay cuatro secciones principales:

- Una carpeta **arm-templates**, que contiene plantillas de Azure Resource Manager.
- Una carpeta **hpc-application**, que contiene la versión de Windows de 64 bits de [ffmpeg 4.3.1](https://github.com/GyanD/codexffmpeg/releases/tag/4.3.1-2020-11-08).
- Una carpeta de **canalizaciones** que contiene un archivo YAML que define el proceso de canalización de la compilación.
- Opcional: una carpeta **client-application**, que es una copia del ejemplo de [procesamiento de archivos .NET de Azure Batch con ffmpeg](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial). No necesitaremos la aplicación en este artículo.


> [!NOTE]
> Este es simplemente un ejemplo de estructura de un código base. Este método se usa con el fin de demostrar que la aplicación, la infraestructura y el código de la canalización se almacenan en el mismo repositorio.

Ahora que ya tenemos el código fuente configurado, puede empezar la primera compilación.

## <a name="continuous-integration"></a>Integración continua

[Azure Pipelines](/azure/devops/pipelines/get-started/), que forma parte Azure DevOps Services, sirve para implementar una canalización de compilación, prueba e implementación para sus aplicaciones.

En esta fase de la canalización se suelen ejecutar pruebas para validar el código y compilar los fragmentos de software adecuados. La cantidad y los tipos de pruebas, así como cualquier otra tarea que ejecute, dependerán de la estrategia de compilación y versión más amplia que haya trazado.

## <a name="prepare-the-hpc-application"></a>Preparación de la aplicación HPC

En esta sección, trabajará con la carpeta **hpc-application**. Esta carpeta contiene el software (ffmpeg) que se ejecutará desde la cuenta de Azure Batch.

1. Vaya a la sección Compilaciones de Azure Pipelines de su organización de Azure DevOps. Cree una **canalización**.

    ![Captura de pantalla de la pantalla de nueva canalización.](media/batch-ci-cd/new-build-pipeline.jpg)

1. Existen dos formas de crear una canalización de compilación:

    a. [Use el diseñador visual](/azure/devops/pipelines/get-started-designer). Para ello, seleccione "Usar el diseñador visual" en la página **Nueva canalización**.

    b. [Use compilaciones YAML](/azure/devops/pipelines/get-started-yaml). Para crear una canalización YAML, haga clic en la opción Azure Repos o en la opción GitHub en la página **Nueva canalización**. Si lo desea, puede almacenar el siguiente ejemplo en su control de código fuente y hacer referencia a un archivo YAML existente; para ello, seleccione el diseñador visual y, después, use la plantilla YAML.

    ```yml
    # To publish an application into Azure Batch, we need to
    # first zip the file, and then publish an artifact, so that
    # we can take the necessary steps in our release pipeline.
    steps:
    # First, we Zip up the files required in the Batch Account
    # For this instance, those are the ffmpeg files
    - task: ArchiveFiles@2
      displayName: 'Archive applications'
      inputs:
        rootFolderOrFile: hpc-application
        includeRootFolder: false
        archiveFile: '$(Build.ArtifactStagingDirectory)/package/$(Build.BuildId).zip'
    # Publish that zip file, so that we can use it as part
    # of our Release Pipeline later
    - task: PublishPipelineArtifact@0
      inputs:
        artifactName: 'hpc-application'
        targetPath: '$(Build.ArtifactStagingDirectory)/package'
    ```

1. Una vez configurada la compilación según sea necesario, seleccione **Guardar y poner en cola**. Si tiene habilitada la integración continua (en la sección **Desencadenadores**), la compilación se desencadenará automáticamente cuando haya una nueva confirmación en el repositorio que cumpla las condiciones establecidas en la compilación.

    ![Captura de pantalla de una canalización de compilación existente.](media/batch-ci-cd/existing-build-pipeline.jpg)

1. Si quiere ver actualizaciones directas del progreso de la compilación en Azure DevOps, vaya a la sección **Compilación** de Azure Pipelines. Seleccione la compilación que quiera en la definición de compilación.

    ![Captura de pantalla de las salidas en directo de la compilación en Azure DevOps.](media/batch-ci-cd/Build-1.jpg)

> [!NOTE]
> Si usa una aplicación cliente para ejecutar la solución HPC, deberá crear una definición de compilación aparte para esa aplicación. Encontrará varias guías de procedimientos en la documentación de [Azure Pipelines](/azure/devops/pipelines/get-started/index).

## <a name="continuous-deployment"></a>Implementación continua

Azure Pipelines también sirve para implementar la aplicación y la infraestructura subyacente. [Las canalizaciones de versiones](/azure/devops/pipelines/release) permiten una implementación continua y que automatiza el proceso de publicación.

### <a name="deploy-your-application-and-underlying-infrastructure"></a>Implementación de la aplicación y la infraestructura subyacente

Implementar la infraestructura conlleva una serie de pasos. Como esta solución utiliza [plantillas vinculadas](../azure-resource-manager/templates/linked-templates.md), estas deberán estar accesibles desde un punto de conexión público (HTTP o HTTPS). ya sea un repositorio de GitHub, una cuenta de Azure Blob Storage u otra ubicación de almacenamiento. Los artefactos de plantilla cargados están protegidos porque se pueden mantener en modo privado, pero se puede permitir el acceso a ellos mediante algún tipo de token de firma de acceso compartido (SAS).

En el siguiente ejemplo se muestra cómo implementar una infraestructura con plantillas de un Azure Storage Blob.

1. Cree una **nueva definición de versión** y seleccione una definición vacía. Cambien el nombre del entorno recién creado por uno que sea pertinente en nuestra canalización.

    ![Captura de pantalla de la canalización de versión inicial.](media/batch-ci-cd/Release-0.jpg)

1. Cree una dependencia en la canalización de compilación para obtener la salida de la aplicación HPC.

    > [!NOTE]
    > Anote el **Alias de origen**, ya que nos hará falta cuando se creen tareas dentro de la definición de versión.

    ![Captura de pantalla que muestra un vínculo del artefacto a HPCApplicationPackage en la canalización de compilación apropiada](media/batch-ci-cd/Release-1.jpg)

1. Cree un vínculo a otro artefacto, esta vez, a una instancia de Azure Repos. Esto es necesario para tener acceso a las plantillas de Resource Manager almacenadas en el repositorio. Como las plantillas de Resource Manager no requieren compilación, no hay que insertarlas a través de una canalización de compilación.

    > [!NOTE]
    > De nuevo, anote el **Alias de origen**, ya que nos hará falta más adelante.

    ![Captura de pantalla que muestra un vínculo de artefacto a la instancia de Azure Repos.](media/batch-ci-cd/Release-2.jpg)

1. Vaya a la sección **Variables**. Querrá crear una serie de variables en la canalización para no tener que volver a escribir la misma información en varias tareas. En este ejemplo se utilizan las variables siguientes:

   - **applicationStorageAccountName**: nombre de la cuenta de almacenamiento que contiene los archivos binarios de la aplicación de HPC.
   - **batchAccountApplicationName**: nombre de la aplicación en la cuenta de Batch.
   - **batchAccountName**: nombre de la cuenta de Batch.
   - **batchAccountPoolName**: nombre del grupo de máquinas virtuales encargadas del procesamiento.
   - **batchApplicationId**: identificador único de la aplicación de Batch.
   - **batchApplicationVersion**: versión semántica de la aplicación de Batch (es decir, los archivos binarios de ffmpeg).
   - **location**: ubicación donde se van a implementar los recursos de Azure.
   - **resourceGroupName**: nombre del grupo de recursos que se creará y donde se implementarán los recursos.
   - **storageAccountName**: nombre de la cuenta de almacenamiento que contiene las plantillas vinculadas de Resource Manager.

   ![Captura de pantalla que muestra las variables definidas para la versión de Azure Pipelines.](media/batch-ci-cd/Release-4.jpg)

1. Vaya a las tareas del entorno de desarrollo. En la captura de pantalla de abajo se ven seis tareas. Estas tareas harán lo siguiente: descargar los archivos de ffmpeg comprimidos en un zip; implementar una cuenta de almacenamiento donde hospedar las plantillas de Resource Manager anidadas; copiar las plantillas de Resource Manager en la cuenta de almacenamiento; implementar la cuenta de Batch y las dependencias necesarias; crear una aplicación en la cuenta de Azure Batch y cargar el paquete de aplicación a esa cuenta de Azure Batch.

    ![Captura de pantalla que muestra las tareas utilizadas para publicar la aplicación HPC en Azure Batch.](media/batch-ci-cd/Release-3.jpg)

1. Agregue la tarea **Descargar artefacto de canalización, versión preliminar** y establezca las siguientes propiedades:
    - **Nombre para mostrar**: "Descargar ApplicationPackage en el agente".
    - **Nombre del artefacto que se va a descargar**: hpc-application.
    - **Ruta de acceso de descarga a**: $(System.DefaultWorkingDirectory).

1. Cree una cuenta de almacenamiento para almacenar las plantillas de Azure Resource Manager. Puede usar una cuenta de almacenamiento existente de la solución, pero en nuestro ejemplo autónomo, y de cara al aislamiento de contenido, crearemos una cuenta de almacenamiento dedicada.

    Agregue la tarea **Implementación de un grupo de recursos de Azure** de y establezca las siguientes propiedades:
    - **Nombre para mostrar**: "Implementar la cuenta de almacenamiento de plantillas de Resource Manager"
    - **Suscripción de Azure**: seleccione la suscripción de Azure que proceda.
    - **Acción**: Creación o actualización del grupo de recursos
    - **Grupo de recursos**: $(resourceGroupName).
    - **Ubicación**: $(location).
    - **Plantilla**: $(System.ArtifactsDirectory)/ **{SuAliasDeOrigenDeArtefactoDeAzureRepos}** /arm-templates/storageAccount.json.
    - **Reemplazar parámetros de plantilla**: accountName $(storageAccountName).

1. Cargue los artefactos desde el control de código fuente en la cuenta de almacenamiento mediante Azure Pipelines. Como parte de esta tarea de Azure Pipelines, el URI del contenedor de la cuenta de almacenamiento y el token de SAS se pueden enviar a una variable en Azure Pipelines, lo que permite que se reutilicen en esta fase del agente.

    Agregue la tarea **Copia de archivos de Azure** y establezca las siguientes propiedades:
    - **Origen**: $(System.ArtifactsDirectory)/ **{SuAliasDeOrigenDeArtefactoDeAzureRepos}** /arm-templates/.
    - **Tipo de conexión de Azure**: Azure Resource Manager
    - **Suscripción de Azure**: seleccione la suscripción de Azure que proceda.
    - **Tipo de destino**: Blob de Azure
    - **Cuenta de almacenamiento del Administrador de recursos**: $(storageAccountName).
    - **Nombre del contenedor**: templates.
    - **URI del contenedor de almacenamiento**: templateContainerUri.
    - **Token de SAS del contenedor de almacenamiento**: templateContainerSasToken.

1. Implemente la plantilla que va a actuar como orquestador. Esta plantilla incluye parámetros para el URI del contenedor de la cuenta de almacenamiento y el token de SAS. Las variables necesarias en la plantilla de Resource Manager se conservan en la sección de variables de la definición de la versión, o bien se han establecido en otra tarea de Azure Pipelines (por ejemplo, como parte de la tarea de copia de blob de Azure).

    Agregue la tarea **Implementación de un grupo de recursos de Azure** de y establezca las siguientes propiedades:
    - **Nombre para mostrar**: Implementar Azure Batch
    - **Suscripción de Azure**: seleccione la suscripción de Azure que proceda.
    - **Acción**: Creación o actualización del grupo de recursos
    - **Grupo de recursos**: $(resourceGroupName).
    - **Ubicación**: $(location).
    - **Plantilla**: $(System.ArtifactsDirectory)/ **{SuAliasDeOrigenDeArtefactoDeAzureRepos}** /arm-templates/deployment.json.
    - **Reemplazar parámetros de plantilla**: `-templateContainerUri $(templateContainerUri) -templateContainerSasToken $(templateContainerSasToken) -batchAccountName $(batchAccountName) -batchAccountPoolName $(batchAccountPoolName) -applicationStorageAccountName $(applicationStorageAccountName)`.

   Una práctica habitual es usar tareas de Azure Key Vault. Si la entidad de servicio conectada a su suscripción de Azure tiene establecido un conjunto de directivas de acceso adecuado, esta puede descargar secretos de Azure Key Vault y usarlos como variables en la canalización. El nombre del secreto se establecerá con el valor asociado. Por ejemplo, se puede hacer referencia a un secreto sshPassword con $(sshPassword) en la definición de la versión.

1. En los siguientes pasos se llama a la CLI de Azure. El primer paso sirve para crear una aplicación en Azure Batch y cargar los paquetes asociados.

    Agregue la tarea **CLI de Azure** y establezca las siguientes propiedades:
    - **Nombre para mostrar**: "Crear la aplicación en la cuenta de Azure Batch"
    - **Suscripción de Azure**: seleccione la suscripción de Azure que proceda.
    - **Ubicación del script**: script en línea.
    - **Script en línea**: `az batch application create --application-id $(batchApplicationId) --name $(batchAccountName) --resource-group $(resourceGroupName)`.

1. El segundo paso sirve para cargar los paquetes asociados en la aplicación (en este caso, los archivos ffmpeg).

    Agregue la tarea **CLI de Azure** y establezca las siguientes propiedades:
    - **Nombre para mostrar**: "Cargar el paquete en la cuenta de Azure Batch"
    - **Suscripción de Azure**: seleccione la suscripción de Azure que proceda.
    - **Ubicación del script**: script en línea.
    - **Script en línea**: `az batch application package create --application-id $(batchApplicationId)  --name $(batchAccountName)  --resource-group $(resourceGroupName) --version $(batchApplicationVersion) --package-file=$(System.DefaultWorkingDirectory)/$(Release.Artifacts.{YourBuildArtifactSourceAlias}.BuildId).zip`.

    > [!NOTE]
    > El número de versión del paquete de aplicación se establece en una variable. Esto permite sobrescribir versiones anteriores del paquete y controlar manualmente el número de versión del paquete insertado en Azure Batch.

1. Cree una versión seleccionando **Versión > Crear una versión nueva**. Una vez desencadenada, seleccione el vínculo a la nueva versión para ver su estado.

1. Para ver la salida directa desde el agente, seleccione el botón **Registros** debajo del entorno.

    ![Captura de pantalla que muestra el estado de la versión.](media/batch-ci-cd/Release-5.jpg)

## <a name="test-the-environment"></a>Prueba del entorno

Una vez configurado el entorno, confirme que las siguientes pruebas finalizan correctamente.

Conéctese a la nueva cuenta de Azure Batch usando la CLI de Azure desde un símbolo del sistema de PowerShell.

- Inicie sesión en su cuenta de Azure con `az login` y siga las instrucciones para autenticarse.
- Ahora, autentique la cuenta de Batch: `az batch account login -g <resourceGroup> -n <batchAccount>`.

#### <a name="list-the-available-applications"></a>Mostrar las aplicaciones disponibles

```azurecli
az batch application list -g <resourcegroup> -n <batchaccountname>
```

#### <a name="check-the-pool-is-valid"></a>Comprobar que el grupo es válido

```azurecli
az batch pool list
```

Anote el valor de `currentDedicatedNodes` de la salida de este comando. Este valor se ajusta en la siguiente prueba.

#### <a name="resize-the-pool"></a>Cambiar el tamaño del grupo

Cambie el tamaño del grupo de forma que haya nodos disponibles para realizar pruebas de trabajos y tareas; consulte el comando de lista de grupo para conocer el estado actual hasta que la operación de cambio de tamaño se haya completado y haya nodos disponibles.

```azurecli
az batch pool resize --pool-id <poolname> --target-dedicated-nodes 4
```

## <a name="next-steps"></a>Pasos siguientes

Consulte estos tutoriales para aprender a interactuar con una cuenta de Batch a través de una aplicación sencilla.

- [Ejecución de una carga de trabajo paralela con Azure Batch mediante Python API](tutorial-parallel-python.md)
- [Ejecución de una carga de trabajo paralela con Azure Batch mediante la API de .NET](tutorial-parallel-dotnet.md)