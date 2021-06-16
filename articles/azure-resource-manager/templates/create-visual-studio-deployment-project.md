---
title: Creación e implementación de proyectos de grupos de recursos de Visual Studio
description: Use Visual Studio para crear un proyecto del grupo de recursos de Azure e implementar los recursos en Azure.
ms.topic: conceptual
ms.date: 04/12/2021
ms.openlocfilehash: 71474578853523245b4859680b739254702afde6
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958110"
---
# <a name="creating-and-deploying-azure-resource-groups-through-visual-studio"></a>Creación e implementación de grupos de recursos de Azure mediante Visual Studio

Con Visual Studio, puede crear un proyecto que implementa su infraestructura y código en Azure. Por ejemplo, puede implementar el host web, el sitio web y el código del sitio web. Visual Studio proporciona muchas plantillas de inicio diferentes para la implementación de escenarios comunes. En este artículo se implementa una aplicación web.

En este artículo se muestra cómo usar [Visual Studio 2019 o posterior con el desarrollo de Azure y las cargas de trabajo de ASP.NET instalados](/visualstudio/install/install-visual-studio). Si utiliza Visual Studio 2017, su experiencia es muy similar.

## <a name="create-azure-resource-group-project"></a>Creación de un proyecto de grupo de recursos de Azure

En esta sección, va a crear un proyecto de Grupo de recursos de Azure con una plantilla **Aplicación web**.

1. En Visual Studio, seleccione **Archivo**>**Nuevo**>**Proyecto**.
1. Seleccione la plantilla del proyecto **Grupo de recursos de Azure**  y **Siguiente**.

    ![Captura de pantalla muestra la ventana Crear un proyecto con Grupo de recursos de Azure y el botón Siguiente resaltado.](./media/create-visual-studio-deployment-project/create-project.png)

1. Asigne un nombre a su proyecto. El resto de las opciones de configuración predeterminadas probablemente sea correctas, pero revíselas para realizar cómo funcionan en su entorno. Cuando haya terminado, seleccione **Crear**.

    ![Crear proyecto](./media/create-visual-studio-deployment-project/name-project.png)

1. Elija la plantilla que desea implementar en Azure Resource Manager. Observe que hay muchas opciones diferentes basadas en el tipo de proyecto que desee implementar. Para este artículo, elija la plantilla **Aplicación web** y **Aceptar**.

    ![Seleccionar una plantilla](./media/create-visual-studio-deployment-project/select-project.png)

    La plantilla que elija es simplemente un punto de partida, puede agregar y quitar recursos para adaptarse a su escenario específico.

1. Visual Studio crea un proyecto de implementación de grupo de recursos para la aplicación web. Para ver los archivos del proyecto, examine el nodo del proyecto de implementación.

    ![Mostrar nodos](./media/create-visual-studio-deployment-project/show-items.png)

    Dado que eligió la plantilla de aplicación web, verá los archivos a continuación:

   | Nombre de archivo | Descripción |
   | --- | --- |
   | Deploy-AzureResourceGroup.ps1 |Un script de PowerShell que ejecuta los comandos de PowerShell que se implementarán en Azure Resource Manager. Visual Studio usa este script de PowerShell para implementar su plantilla. |
   | WebSite.json |La plantilla de Resource Manager que define la infraestructura que desea implementar en Azure y los parámetros que puede proporcionar durante la implementación. También define las dependencias entre los recursos, de modo que Resource Manager implemente los recursos en el orden correcto. |
   | WebSite.parameters.json |Un archivo de parámetros que tiene valores necesarios para la plantilla. Transferirá los valores de los parámetros para personalizar cada implementación. Observe que **Acción de compilación** se establece en **Contenido**. Si agrega más archivos de parámetros, asegúrese de que la acción de compilación está establecida en **Contenido**. |

    Todos los proyectos de implementación del grupo de recursos tienen estos archivos básicos. Otros proyectos pueden tener más archivos para admitir otras funcionalidades.

## <a name="customize-resource-manager-template"></a>Personalización de la plantilla de Resource Manager

Puede personalizar un proyecto de implementación si modifica la plantilla de Resource Manager que describen los recursos que desea implementar. Para más información sobre los elementos de la plantilla de Resource Manager, consulte [Creación de plantillas de Azure Resource Manager](./syntax.md).

1. Para trabajar en la plantilla, abra **WebSite.json**.

1. El editor de Visual Studio proporciona herramientas para ayudarle con la edición de la plantilla de Resource Manager. La ventana **Esquema JSON** facilita la visualización de los elementos definidos en la plantilla.

   ![Mostrar el esquema JSON](./media/create-visual-studio-deployment-project/show-json-outline.png)

1. Seleccione un elemento en el esquema para ir a esa parte de la plantilla.

   ![Navegar en JSON](./media/create-visual-studio-deployment-project/navigate-json.png)

1. Para agregar un recurso, haga clic en el botón **Agregar recurso** de la parte superior de la ventana Esquema JSON, o haga clic con el botón derecho en **recursos** y seleccione **Agregar nuevo recurso**.

   ![Captura de pantalla que muestra la ventana Esquema JSON con la opción Agregar nuevo recurso resaltada.](./media/create-visual-studio-deployment-project/add-resource.png)

1. Seleccione **Cuenta de almacenamiento** y asígnele un nombre. Proporcione un nombre que no tenga más de 11 caracteres y que solo contenga números y letras minúsculas.

   ![Agregue almacenamiento](./media/create-visual-studio-deployment-project/add-storage.png)

1. Tenga en cuenta que no solo se agregó el recurso, sino que también se agregó un parámetro para la cuenta de almacenamiento de tipo y una variable para el nombre de la cuenta de almacenamiento.

   ![Mostrar el esquema](./media/create-visual-studio-deployment-project/show-new-items.png)

1. El parámetro para el tipo de cuenta de almacenamiento está predefinido con los tipos permitidos y un tipo predeterminado. Puede dejar estos valores o modificarlos para su escenario específico. Si no quiere que nadie pueda implementar una cuenta de almacenamiento **Premium_LRS** a través de esta plantilla, quítela de los tipos permitidos.

   ```json
   "demoaccountType": {
     "type": "string",
     "defaultValue": "Standard_LRS",
     "allowedValues": [
       "Standard_LRS",
       "Standard_ZRS",
       "Standard_GRS",
       "Standard_RAGRS"
     ]
   }
   ```

1. Visual Studio también proporciona IntelliSense para ayudarle a entender qué propiedades están disponibles al editar la plantilla. Por ejemplo, para editar las propiedades de su plan de App Service, navegue al recurso **HostingPlan** y agregue un valor a las **propiedades**. Observe que IntelliSense muestra los valores disponibles y proporciona una descripción de cada valor.

   ![Mostrar Intellisense](./media/create-visual-studio-deployment-project/show-intellisense.png)

   Puede establecer **numberOfWorkers** en 1 y guardar el archivo.

   ```json
   "properties": {
     "name": "[parameters('hostingPlanName')]",
     "numberOfWorkers": 1
   }
   ```

1. Abra el archivo **WebSite.parameters.json**. Utilice el archivo de parámetros para pasar valores durante la implementación que personalizan el recurso que se va a implementar. Asigne un nombre al plan de hospedaje y guarde el archivo.

   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "hostingPlanName": {
         "value": "demoHostPlan"
       }
     }
   }
   ```

## <a name="deploy-project-to-azure"></a>Implementación del proyecto en Azure

Ahora está preparado para implementar el proyecto en un grupo de recursos.

De forma predeterminada, el script de PowerShell (Deploy-AzureResourceGroup.ps1) en el proyecto usa el módulo de AzureRM. Si todavía tiene instalado el módulo de AzureRM y desea seguir utilizándolo, puede utilizar este script predeterminado. Con este script, puede usar la interfaz de Visual Studio para implementar la solución.

Sin embargo, si ha migrado al nuevo [módulo Az](/powershell/azure/new-azureps-module-az), deberá agregar un nuevo script al proyecto. Para agregar un script que usa el módulo de Az, copie el script [Deploy-AzTemplate.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzTemplate.ps1) y agréguelo al proyecto. Para utilizar este script para la implementación, debe ejecutarlo desde una consola de PowerShell, en lugar de utilizar la interfaz de la implementación de Visual Studio.

En este artículo se muestran ambos enfoques. Este artículo se refiere al script predeterminado como script del módulo de AzureRM y el nuevo script como el del módulo de Az.

### <a name="az-module-script"></a>Script de módulo de Az

Para el script del módulo Az, abra una consola de PowerShell y ejecute:

```powershell
.\Deploy-AzTemplate.ps1 -ArtifactStagingDirectory . -Location centralus -TemplateFile WebSite.json -TemplateParametersFile WebSite.parameters.json
```

### <a name="azurerm-module-script"></a>Script del módulo AzureRM

Para el script del módulo AzureRM, use Visual Studio:

1. En el menú contextual del nodo del proyecto de implementación, elija **Implementar** > **Nueva**.

    ![Nueva implementación del elemento de menú](./media/create-visual-studio-deployment-project/deploy.png)

1. Aparece el cuadro de diálogo **Implementar en grupo de recursos**. En el cuadro de lista desplegable **Grupo de recursos**, elija un grupo de recursos existente o cree uno nuevo. Seleccione **Implementar**.

    ![Cuadro de diálogo Implementar en grupo de recursos](./media/create-visual-studio-deployment-project/show-deployment.png)

1. En la ventana **Salida** puede ver el estado de la implementación. Cuando se termina la implementación, el último mensaje indica una implementación satisfactoria de una forma parecida a esta:

   ```output
   18:00:58 - Successfully deployed template 'website.json' to resource group 'ExampleAppDeploy'.
   ```

## <a name="view-deployed-resources"></a>Visualización de los recursos implementados

Comprobemos los resultados.

1. En un explorador, abra [Azure Portal](https://portal.azure.com/) e inicie sesión en su cuenta. Para ver el grupo de recursos, seleccione **Grupos de recursos** y el grupo de recursos que implementó.

1. Verá todos los recursos implementados. Observe que el nombre de la cuenta de almacenamiento no es exactamente el que especificó al agregar ese recurso. La cuenta de almacenamiento debe ser única. La plantilla agrega automáticamente una cadena de caracteres al nombre que proporcionó para crear uno único.

    ![Mostrar recursos](./media/create-visual-studio-deployment-project/show-deployed-resources.png)

## <a name="add-code-to-project"></a>Agregar código al proyecto

A estas alturas ha implementado la infraestructura de la aplicación, pero no hay ningún código real que se haya implementado con el proyecto.

1. Agregue un proyecto a la solución de Visual Studio. Haga clic con el botón derecho en la solución y seleccione **Agregar** > **Nuevo proyecto**.

    ![Agregar proyecto](./media/create-visual-studio-deployment-project/add-project.png)

1. Agregue una **aplicación web ASP.NET Core**.

    ![Agregar aplicación web](./media/create-visual-studio-deployment-project/add-app.png)

1. Asigne un nombre a la aplicación web y seleccione **Crear**.

    ![Asignar nombre a aplicación web](./media/create-visual-studio-deployment-project/name-web-app.png)

1. Seleccione **Aplicación web** y **Crear**.

    ![Seleccionar aplicación web](./media/create-visual-studio-deployment-project/select-project-type.png)

1. Después de que Visual Studio cree la aplicación web, podrá ver ambos proyectos en la solución.

    ![Mostrar proyectos](./media/create-visual-studio-deployment-project/show-projects.png)

1. Ahora, debe asegurarse de que el proyecto del grupo de recursos reconoce el nuevo proyecto. Vuelva al proyecto del grupo de recursos (ExampleAppDeploy). Haga clic con el botón derecho en **Referencias** y seleccione **Agregar referencia**.

    ![Captura de pantalla que muestra el menú ExampleAppDeploy con la opción Agregar referencia resaltada.](./media/create-visual-studio-deployment-project/add-new-reference.png)

1. Seleccione el proyecto de la aplicación web que ha creado.

   ![Agregar referencia](./media/create-visual-studio-deployment-project/add-reference.png)

   Al agregar una referencia, se vincula el proyecto de aplicación web con el proyecto del grupo de recursos y automáticamente se establecen algunas propiedades. Puede ver estas propiedades en la ventana **Propiedades** de la referencia. **Include File Path** (Incluir ruta del archivo) contiene la ruta de acceso donde se crea el paquete. Observe la carpeta (ExampleApp) y el archivo (package.zip). Debe conocer estos valores porque los proporcionará como parámetros al implementar la aplicación.

   ![Ver referencia](./media/create-visual-studio-deployment-project/see-reference.png)

1. Vuelva a su plantilla (WebSite.json) y agréguele un recurso.

    ![Agregar recurso](./media/create-visual-studio-deployment-project/add-resource-2.png)

1. En esta ocasión, seleccione **Web Deploy para Web Apps**.

    ![Agregar implementación web](./media/create-visual-studio-deployment-project/add-web-deploy.png)

   Guarde la plantilla.

1. Hay algunos parámetros nuevos en la plantilla. Se han agregado en el paso anterior. No es necesario especificar valores en **_artifactsLocation** o **_artifactsLocationSasToken**, ya que se generan automáticamente. Sin embargo, debe establecer la carpeta y el nombre de archivo con la ruta de acceso que contiene el paquete de implementación. Los nombres de estos parámetros terminan con **PackageFolder** y **PackageFileName**. La primera parte del nombre es el nombre del recurso Web Deploy que agregó. En este artículo, se llaman **ExampleAppPackageFolder** y **ExampleAppPackageFileName**.

   Abra **Website.parameters.json** y establezca esos parámetros en los valores que ha visto en las propiedades de referencia. Establezca **ExampleAppPackageFolder** en el nombre de la carpeta. Establezca **ExampleAppPackageFileName** en el nombre del archivo ZIP.

   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "hostingPlanName": {
         "value": "demoHostPlan"
       },
       "ExampleAppPackageFolder": {
         "value": "ExampleApp"
       },
       "ExampleAppPackageFileName": {
         "value": "package.zip"
       }
     }
   }
   ```

## <a name="deploy-code-with-infrastructure"></a>Implementación de código con la infraestructura

Dado que ha agregado código para el proyecto, la implementación es un poco diferente en este momento. Durante la implementación, almacena provisionalmente los artefactos para el proyecto en un lugar al que Resource Manager pueda tener acceso. Los artefactos se almacenan en una cuenta de almacenamiento.

### <a name="az-module-script"></a>Script de módulo de Az

Hay un pequeño cambio que debe realizar en la plantilla si está utilizando el script del módulo Az. Este script agrega una barra diagonal a la ubicación de artefactos, pero la plantilla no la espera. Abra WebSite.json y busque las propiedades de la extensión de MSDeploy. Tiene una propiedad denominada **packageUri**. Quite la barra diagonal entre la ubicación de los artefactos y la carpeta del paquete.

Debe verse de una manera similar a:

```json
"packageUri": "[concat(parameters('_artifactsLocation'), parameters('ExampleAppPackageFolder'), '/', parameters('ExampleAppPackageFileName'), parameters('_artifactsLocationSasToken'))]",
```

Observe que, en el ejemplo anterior, no hay ningún `'/',` entre **parameters('_artifactsLocation')** y **parameters('ExampleAppPackageFolder')** .

Recompile el proyecto. Compilar el proyecto garantiza que los archivos que necesita implementar se agregan a la carpeta provisional.

Ahora, abra una consola de PowerShell y ejecute:

```powershell
.\Deploy-AzTemplate.ps1 -ArtifactStagingDirectory .\bin\Debug\staging\ExampleAppDeploy -Location centralus -TemplateFile WebSite.json -TemplateParametersFile WebSite.parameters.json -UploadArtifacts -StorageAccountName <storage-account-name>
```

### <a name="azurerm-module-script"></a>Script del módulo AzureRM

Para el script del módulo AzureRM, use Visual Studio:

1. Para volver a implementar, elija **Implementar** y el grupo de recursos que implementó anteriormente.

    ![Volver a implementar un proyecto](./media/create-visual-studio-deployment-project/redeploy.png)

1. Seleccione la cuenta de almacenamiento implementada con este grupo de recursos para la **Cuenta de almacenamiento de artefactos**.

   ![Volver a implementar la implementación web](./media/create-visual-studio-deployment-project/redeploy-web-app.png)

## <a name="view-web-app"></a>Ver aplicación web

1. Una vez finalizada la implementación, seleccione la aplicación web en el portal. Seleccione la dirección URL para navegar al sitio.

   ![Examinar sitio](./media/create-visual-studio-deployment-project/browse-site.png)

1. Observe que ha implementado correctamente la aplicación ASP.NET predeterminada.

   ![Mostrar la aplicación implementada](./media/create-visual-studio-deployment-project/show-deployed-app.png)

## <a name="add-operations-dashboard"></a>Agregar panel de operaciones

No está limitado únicamente a los recursos que están disponibles a través de la interfaz de Visual Studio. Puede personalizar la implementación si agrega un recurso personalizado a la plantilla. Para mostrar la incorporación de un recurso, agregue un panel de operaciones para administrar el recurso que implementó.

1. Abra el archivo Website.json y agregue el JSON siguiente después del recurso de la cuenta de almacenamiento y antes del símbolo `]` de cierre de la sección de recursos.

   ```json
    ,{
      "properties": {
        "lenses": {
          "0": {
            "order": 0,
            "parts": {
              "0": {
                "position": {
                  "x": 0,
                  "y": 0,
                  "colSpan": 4,
                  "rowSpan": 6
                },
                "metadata": {
                  "inputs": [
                    {
                      "name": "resourceGroup",
                      "isOptional": true
                    },
                    {
                      "name": "id",
                      "value": "[resourceGroup().id]",
                      "isOptional": true
                    }
                  ],
                  "type": "Extension/HubsExtension/PartType/ResourceGroupMapPinnedPart"
                }
              },
              "1": {
                "position": {
                  "x": 4,
                  "y": 0,
                  "rowSpan": 3,
                  "colSpan": 4
                },
                "metadata": {
                  "inputs": [],
                  "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                  "settings": {
                    "content": {
                      "settings": {
                        "content": "__Customizations__\n\nUse this dashboard to create and share the operational views of services critical to the application performing. To customize simply pin components to the dashboard and then publish when you're done. Others will see your changes when you publish and share the dashboard.\n\nYou can customize this text too. It supports plain text, __Markdown__, and even limited HTML like images <img width='10' src='https://portal.azure.com/favicon.ico'/> and <a href='https://azure.microsoft.com' target='_blank'>links</a> that open in a new tab.\n",
                        "title": "Operations",
                        "subtitle": "[resourceGroup().name]"
                      }
                    }
                  }
                }
              }
            }
          }
        },
        "metadata": {
          "model": {
            "timeRange": {
              "value": {
                "relative": {
                  "duration": 24,
                  "timeUnit": 1
                }
              },
              "type": "MsPortalFx.Composition.Configuration.ValueTypes.TimeRange"
            }
          }
        }
      },
      "type": "Microsoft.Portal/dashboards",
      "apiVersion": "2015-08-01-preview",
      "name": "[concat('ARM-',resourceGroup().name)]",
      "location": "[resourceGroup().location]",
      "tags": {
        "hidden-title": "[concat('OPS-',resourceGroup().name)]"
      }
    }
   ```

1. Vuelva a implementar el proyecto.

1. Una vez finalizada la implementación, puede ver el panel en el portal. Seleccione **Panel** y elija el que haya implementado.

   ![Captura de pantalla que muestra la página de panel con un panel personalizado de ejemplo resaltado.](./media/create-visual-studio-deployment-project/view-custom-dashboards.png)

1. Verá el panel personalizado.

   ![Panel personalizado](./media/create-visual-studio-deployment-project/Ops-DemoSiteGroup-dashboard.png)

Puede administrar el acceso al panel mediante el control de acceso basado en rol de Azure (Azure RBAC). También puede personalizar la apariencia del panel después de implementarlo. Sin embargo, si vuelve a implementar el grupo de recursos, el panel se restablece a su estado predeterminado en la plantilla. Para más información sobre cómo crear paneles, consulte [Creación mediante programación de paneles de Azure](../../azure-portal/azure-portal-dashboards-create-programmatically.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.

1. Seleccione el nombre del grupo de recursos.

1. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a crear e implementar plantillas con Visual Studio. Para más información sobre el desarrollo de plantillas, consulte nuestra nueva serie de tutoriales para principiantes:

> [!div class="nextstepaction"]
> [Tutoriales para principiantes](./template-tutorial-create-first-template.md)