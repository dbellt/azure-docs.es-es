---
title: 'Tutorial: Implementación de Azure Policy as Code con GitHub'
description: En este tutorial, implementará un flujo de trabajo de directiva de Azure como código con exportaciones, acciones de GitHub y flujos de trabajo de GitHub.
ms.date: 03/31/2021
ms.topic: tutorial
ms.openlocfilehash: d7ad4d0487d8a6ead6c89834569e130c7b9945f5
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108752038"
---
# <a name="tutorial-implement-azure-policy-as-code-with-github"></a>Tutorial: Implementación de Azure Policy as Code con GitHub

Un flujo de trabajo de directiva de Azure como código permite administrar las definiciones de directiva y las asignaciones como código, controlar el ciclo de vida de la actualización de esas definiciones y automatizar la validación de los resultados de cumplimiento. En este tutorial, aprenderá a usar las características de Azure Policy con GitHub para crear un proceso del ciclo de vida. Estas tareas incluyen:

> [!div class="checklist"]
> - Exportar las definiciones y asignaciones de directivas a GitHub
> - Insertar en Azure los objetos de directivas actualizados en GitHub
> - Desencadenar un examen de cumplimiento desde una acción de GitHub

Si desea asignar una directiva para identificar el estado de cumplimiento actual de los recursos existentes, en los artículos de inicio rápido se explica cómo hacerlo.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Consulte [Diseño de flujos de trabajo de directiva como código](../concepts/policy-as-code.md) para obtener una descripción de los patrones de diseño que se usan en este tutorial.

### <a name="export-azure-policy-objects-from-the-azure-portal"></a>Exportación de objetos de Azure Policy desde Azure Portal

Para exportar una definición de directiva desde Azure Portal, siga estos pasos:

1. Inicie el servicio Azure Policy en Azure Portal. Para ello, haga clic en **Todos los servicios** y, a continuación, busque y seleccione **Directiva**.

1. Seleccione **Definiciones** en el lado izquierdo de la página de Azure Policy.

1. Use el botón **Exportar definiciones**, o bien seleccione el botón de los puntos suspensivos que encontrará en la fila de la definición de una directiva y, a continuación, seleccione **Exportar definición**.

1. Seleccione el botón **Iniciar sesión con GitHub**. Si aún no se ha autenticado en GitHub para autorizar a Azure Policy a exportar el recurso, revise el acceso que [Acciones de GitHub](https://github.com/features/actions) requiere en la nueva ventana que se abre y seleccione **Autorizar AzureGitHubActions** para continuar con el proceso de exportación. Una vez completado este paso, la nueva ventana se cerrará automáticamente.

1. En la pestaña **Aspectos básicos**, establezca las siguientes opciones y,a continuación, seleccione la pestaña **Directivas** o el botón **Siguiente: Directivas** situado en la parte inferior de la página.

   - **Filtro del repositorio**: establezca esta opción en _Mis repositorios_ si solo quiere ver los repositorios que posee, o bien en _Todos los repositorios_ para ver todos aquellos a los que concedió acceso a Acciones de GitHub.
   - **Repositorio**: establezca esta opción en el repositorio en el que quiera exportar los recursos de Azure Policy.
   - **Rama**: establezca la rama del repositorio. Usar una rama distinta de la predeterminada es una buena manera de validar las actualizaciones antes de combinarlas en el código fuente.
   - **Directorio**: se trata de la _carpeta de nivel raíz_ en la que se exportarán los recursos de Azure Policy. Las subcarpetas de este directorio se crean en función de los recursos que se exportan.

1. En la pestaña **Directivas**, establezca el ámbito de búsqueda; para ello, seleccione el botón de los puntos suspensivos y elija una combinación de grupos de administración, suscripciones o grupos de recursos.

1. Use el botón **Agregar definiciones de directiva** para realizar la búsqueda en el ámbito cuyos objetos se van a exportar. En la ventana lateral que se abre, seleccione cada uno de los objetos que quiera exportar. Filtre la selección mediante el cuadro de búsqueda o el tipo. Una vez que haya seleccionado todos los objetos que se van a exportar, use el botón **Agregar** de la parte inferior de la página.

1. Para cada objeto seleccionado, seleccione las opciones de exportación que prefiera, como _Solo definición_ o _Definición y asignaciones_, para una definición de directiva. Luego, seleccione la pestaña **Revisar y exportar** o el botón **Siguiente: Revisar y exportar** situado en la parte inferior de la página.

   > [!NOTE]
   > Si se elige la opción _Definición y asignaciones_, solo se exportarán las asignaciones de directiva comprendidas en el ámbito establecido por el filtro al agregar la definición de directiva.

1. En la pestaña **Revisar y exportar**, compruebe que los detalles coincidan y, luego, use el botón **Exportar** situado en la parte inferior de la página.

1. Compruebe el repositorio de GitHub, la rama y la _carpeta de nivel raíz_ para ver que los recursos seleccionados se hayan exportado en el control de código fuente.

Los recursos de Azure Policy se exportan en la siguiente estructura dentro del repositorio y la _carpeta de nivel raíz_ de GitHub que haya seleccionado:

```text
|
|- <root level folder>/  ________________ # Root level folder set by Directory property
|  |- policies/  ________________________ # Subfolder for policy objects
|     |- <displayName>_<name>____________ # Subfolder based on policy displayName and name properties
|        |- policy.json _________________ # Policy definition
|        |- assign.<displayName>_<name>__ # Each assignment (if selected) based on displayName and name properties
|
```

### <a name="push-policy-objects-updated-in-github-to-azure"></a>Insertar en Azure los objetos de directivas actualizados en GitHub

1. Cuando se exportan objetos de directiva, también se crea un archivo de [flujo de trabajo de GitHub](https://docs.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow#about-workflows) llamado `.github/workflows/manage-azure-policy-<randomLetters>.yml` para comenzar.

   > [!NOTE]
   > El archivo de flujo de trabajo de GitHub se crea cada vez que se usa la exportación. Cada instancia del archivo es específica de las opciones establecidas durante la acción de exportación.

1. Este archivo de flujo de trabajo usa la acción [Administrar Azure Policy](https://github.com/marketplace/actions/manage-azure-policy) para enviar los cambios realizados en los objetos de directiva exportados del repositorio de GitHub de nuevo a Azure Policy. De forma predeterminada, la acción tiene en cuenta y sincroniza solo los archivos que son diferentes de los existentes en Azure. También puede usar el parámetro `assignments` en la acción para sincronizar solo los cambios realizados en archivos de asignación específicos. Este parámetro se puede usar para aplicar asignaciones de directivas solo para un entorno específico. Para más información, consulte el [archivo Léame del repositorio Administrar Azure Policy](https://github.com/Azure/manage-azure-policy).

1. De forma predeterminada, el flujo de trabajo se debe desencadenar manualmente. Para ello, use la opción **Actions** (Acciones) de GitHub, seleccione el flujo de trabajo `manage-azure-policy-<randomLetters>`, seleccione **Run workflow** (Ejecutar flujo de trabajo) y, a continuación, **Run workflow** (Ejecutar flujo de trabajo) de nuevo.

   :::image type="content" source="../media/policy-as-code-github/manually-trigger-workflow.png" alt-text="Captura de pantalla de la pestaña de la acción, el flujo de trabajo y los botones para ejecutar el flujo de trabajo en la interfaz web de GitHub.":::

   > [!NOTE]
   > El archivo de flujo de trabajo debe estar en la rama predeterminada para que se detecte y se ejecute manualmente.

1. El flujo de trabajo sincroniza los cambios realizados en los objetos de directivas con Azure y le proporciona el estado en los registros.

   :::image type="content" source="../media/policy-as-code-github/workflow-logging.png" alt-text="Captura de pantalla del flujo de trabajo en acción y los detalles registrados en los registros.":::

1. El flujo de trabajo también agrega los detalles en el elemento `properties.metadata` de los objetos de Azure Policy para el seguimiento.

   :::image type="content" source="../media/policy-as-code-github/updated-definition-metadata.png" alt-text="Captura de pantalla de la definición de directiva de Azure en Azure Portal actualizada con los metadatos específicos de la acción de GitHub.":::

### <a name="trigger-compliance-scans-using-github-action"></a>Desencadenado de exámenes de cumplimiento mediante la acción de GitHub

Use la acción de [Azure Policy de análisis de cumplimiento](https://github.com/marketplace/actions/azure-policy-compliance-scan) para desencadenar un análisis de evaluación de cumplimiento a petición desde el [flujo de trabajo de GitHub](https://docs.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow#about-workflows) en uno o varios recursos, grupos de recursos o suscripciones, y modifique la ruta de acceso del flujo de trabajo en función del estado de cumplimiento de esos recursos. También puede configurar el flujo de trabajo para que se ejecute a una hora programada para obtener el estado de cumplimiento más reciente en un momento adecuado. Opcionalmente, esta acción de GitHub también puede generar un informe sobre el estado de cumplimiento de los recursos examinados para analizarlos o archivarlos más adelante.

En el ejemplo siguiente se ejecuta el examen de cumplimiento de una suscripción.

```yaml

on:
  schedule:
    - cron:  '0 8 * * *'  # runs every morning 8am
jobs:
  assess-policy-compliance:
    runs-on: ubuntu-latest
    steps:
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{secrets.AZURE_CREDENTIALS}}

    - name: Check for resource compliance
      uses: azure/policy-compliance-scan@v0
      with:
        scopes: |
          /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## <a name="review"></a>Revisar

En este tutorial, ha realizado correctamente las tareas siguientes:

> [!div class="checklist"]
> - Ha exportado las definiciones y asignaciones de directivas a GitHub.
> - Ha insertado en Azure los objetos de directivas actualizados en GitHub.
> - Ha desencadenado un examen de cumplimiento desde una acción de GitHub.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las estructuras de las definiciones de directiva, consulte este artículo:

> [!div class="nextstepaction"]
> [Estructura de definición de Azure Policy](../concepts/definition-structure.md)
