---
title: Implementar directivas personalizadas con Azure Pipelines
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo implementar directivas personalizadas de Azure AD B2C en una canalización de CI/CD mediante Azure Pipelines en Azure DevOps Services.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 913f21b90043209cae1ec9963619389bcb452781
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529435"
---
# <a name="deploy-custom-policies-with-azure-pipelines"></a>Implementar directivas personalizadas con Azure Pipelines

Mediante el uso de una canalización de integración y entrega continuas (CI/CD) configurada en [Azure Pipelines][devops-pipelines], puede incluir las directivas personalizadas de Azure AD B2C en la entrega de software y la automatización del control de código. A medida que se implementan en diferentes entornos de Azure AD B2C, por ejemplo, en desarrollo, prueba y producción, se recomienda quitar los procesos manuales y realizar pruebas automatizadas con Azure Pipelines.

Son necesarios tres pasos principales para permitir que Azure Pipelines administre las directivas personalizadas en Azure AD B2C:

1. Crear un registro de aplicación web en el inquilino de Azure AD B2C
1. Configurar un repositorio de Azure
1. Configurar una canalización de Azure

> [!IMPORTANT]
> La administración de las directivas personalizadas de Azure AD B2C con una canalización de Azure usa actualmente la **vista previa** de las operaciones de disponibles en el punto de conexión `/beta` de Microsoft Graph API. No se admite su uso en aplicaciones de producción. Para obtener más información, consulte [Referencia del punto de conexión de la API de REST Microsoft Graph beta](/graph/api/overview?toc=.%2fref%2ftoc.json&view=graph-rest-beta&preserve-view=true).

## <a name="prerequisites"></a>Prerrequisitos

* [Inquilino de Azure AD B2C](tutorial-create-tenant.md) y credenciales para un usuario en el directorio con el rol [Administrador de directivas IEF de B2C](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator)
* [Directivas personalizadas](tutorial-create-user-flows.md?pivots=b2c-custom-policy) cargadas en el inquilino
* [Aplicación de administración](microsoft-graph-get-started.md) registrada en el inquilino con el permiso de Microsoft Graph API *Policy.ReadWrite.TrustFramework*
* [Canalización de Azure](https://azure.microsoft.com/services/devops/pipelines/) y acceso a un [proyecto de Azure DevOps Services][devops-create-project]

## <a name="client-credentials-grant-flow"></a>Flujo de concesión de credenciales del cliente

El escenario que se describe aquí usa las llamadas de servicio a servicio entre Azure Pipelines y Azure AD B2C mediante el [flujo de concesión de credenciales de cliente](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) de OAuth 2.0. Este flujo de concesión permite que un servicio web como Azure Pipelines (cliente confidencial) use sus propias credenciales para autenticarse al llamar a otro servicio web, en lugar de suplantar a un usuario (Microsoft Graph API, en este caso). Azure Pipelines obtiene un token de forma no interactiva y, a continuación, realiza solicitudes a Microsoft Graph API.

## <a name="register-an-application-for-management-tasks"></a>Registrar una aplicación para tareas de administración

Como se menciona en los [Requisitos previos](#prerequisites), necesita un registro de aplicación que los scripts de PowerShell (ejecutados por Azure Pipelines) puedan usar para acceder a los recursos de su inquilino.

Si ya tiene un registro de aplicación que usa para las tareas de automatización, asegúrese de que se le ha concedido el permiso **Microsoft Graph** > **Directiva** > **Policy.ReadWrite.TrustFramework** en **Permisos de API** del registro de aplicación.

Para obtener instrucciones sobre cómo registrar una aplicación de administración, consulte [Administrar Azure AD B2C con Microsoft Graph](microsoft-graph-get-started.md).

## <a name="configure-an-azure-repo"></a>Configurar un repositorio de Azure

Una vez registrada la aplicación de administración, está listo para configurar un repositorio para los archivos de directivas.

1. Inicie sesión en su organización de Azure DevOps Services.
1. [Cree un nuevo proyecto][devops-create-project] o seleccione uno existente.
1. En el proyecto, navegue hasta **Repos** y seleccione la página **Archivos**. Seleccione un repositorio existente o cree uno para este ejercicio.
1. Cree una carpeta denominada *B2CAssets*. Asigne al archivo de marcador de posición necesario el nombre *README.md* y **confirme** el archivo. Puede quitar este archivo más adelante, si lo desea.
1. Agregue los archivos de directiva de Azure AD B2C a la carpeta *B2CAssets*. Entre ellos se incluyen *TrustFrameworkBase.xml*, *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* y *PasswordReset.xml* y otras directivas que ha creado. Registre el nombre de archivo de cada archivo de directiva de Azure AD B2C para usarlo en un paso posterior (se usan como argumentos de script de PowerShell).
1. Cree una carpeta denominada *Scripts* en el directorio raíz del repositorio, asigne al archivo de marcador de posición el nombre *DeployToB2c.ps1*. No confirme el archivo en este momento, lo hará en un paso posterior.
1. Pegue el siguiente script de PowerShell en *DeployToB2c.ps1* y, después, **confirme** el archivo. El script adquiere un token de Azure AD y llama a Microsoft Graph API para cargar las directivas dentro de la carpeta *B2CAssets* en el inquilino de Azure AD B2C.

    ```PowerShell
    [Cmdletbinding()]
    Param(
        [Parameter(Mandatory = $true)][string]$ClientID,
        [Parameter(Mandatory = $true)][string]$ClientSecret,
        [Parameter(Mandatory = $true)][string]$TenantId,
        [Parameter(Mandatory = $true)][string]$PolicyId,
        [Parameter(Mandatory = $true)][string]$PathToFile
    )

    try {
        $body = @{grant_type = "client_credentials"; scope = "https://graph.microsoft.com/.default"; client_id = $ClientID; client_secret = $ClientSecret }

        $response = Invoke-RestMethod -Uri https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token -Method Post -Body $body
        $token = $response.access_token

        $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
        $headers.Add("Content-Type", 'application/xml')
        $headers.Add("Authorization", 'Bearer ' + $token)

        $graphuri = 'https://graph.microsoft.com/beta/trustframework/policies/' + $PolicyId + '/$value'
        $policycontent = Get-Content $PathToFile
        $response = Invoke-RestMethod -Uri $graphuri -Method Put -Body $policycontent -Headers $headers

        Write-Host "Policy" $PolicyId "uploaded successfully."
    }
    catch {
        Write-Host "StatusCode:" $_.Exception.Response.StatusCode.value__

        $_

        $streamReader = [System.IO.StreamReader]::new($_.Exception.Response.GetResponseStream())
        $streamReader.BaseStream.Position = 0
        $streamReader.DiscardBufferedData()
        $errResp = $streamReader.ReadToEnd()
        $streamReader.Close()

        $ErrResp

        exit 1
    }

    exit 0
    ```

## <a name="configure-your-azure-pipeline"></a>Configurar la canalización de Azure

Con el repositorio inicializado y rellenado con los archivos de directivas personalizadas, está listo para configurar la canalización de versión.

### <a name="create-pipeline"></a>Creación de una canalización

1. Inicie sesión en su organización de Azure DevOps Services y vaya a su proyecto.
1. En el proyecto, seleccione **Canalizaciones** > **Versiones** > **Nueva canalización**.
1. En **Seleccionar una plantilla**, seleccione **Empty job** (Trabajo vacío).
1. Escriba un **Nombre de fase**, por ejemplo, *DeployCustomPolicies*, y luego cierre el panel.
1. Seleccione **Agregar un artefacto** y, en **Tipo de origen**, seleccione **Azure Repository** (Repositorio de Azure).
    1. Elija el repositorio de origen que contiene la carpeta *Scripts* que ha rellenado con el script de PowerShell.
    1. Seleccione una **Rama predeterminada**. Si ha creado un nuevo repositorio en la sección anterior, la rama predeterminada es *maestro*.
    1. Deje la configuración de **Versión predeterminada** en *Más reciente desde la rama predeterminada*.
    1. Escriba un **Alias de origen** para el repositorio. Por ejemplo, *policyRepo*. No incluya espacios en el nombre de alias.
1. Seleccione **Agregar**.
1. Cambie el nombre de la canalización para que refleje su intención. Por ejemplo, *Implementar la canalización de directivas personalizadas*.
1. Para guardar la configuración de canalización, seleccione **Guardar**.

### <a name="configure-pipeline-variables"></a>Configurar variables de canalización

1. Seleccione la pestaña **Variables**.
1. Agregue las siguientes variables en **Variables de canalización** y establezca sus valores como se especifica a continuación:

    | Nombre | Value |
    | ---- | ----- |
    | `clientId` | **Id. de aplicación (cliente)** de la aplicación que registró anteriormente. |
    | `clientSecret` | El valor del **secreto de cliente** que creó anteriormente. <br /> Cambie el tipo de variable a **secreto** (seleccione el icono de candado). |
    | `tenantId` | `your-b2c-tenant.onmicrosoft.com`, donde *your-b2c-tenant* es el nombre del inquilino de Azure AD B2C. |

1. Seleccione **Guardar** para guardar las variables.

### <a name="add-pipeline-tasks"></a>Agregar tareas de canalización

A continuación, agregue una tarea para implementar un archivo de directiva.

1. Seleccione la ficha **Tareas**.
1. Seleccione **Trabajo de agente** y, a continuación, seleccione el signo más ( **+** ) para agregar una tarea al trabajo del agente.
1. Busque y seleccione **PowerShell**. No seleccione "Azure PowerShell", "PowerShell en equipos de destino" ni otra entrada de PowerShell.
1. Seleccione la tarea **Script de PowerShell** recién agregada.
1. Escriba los siguientes valores para la tarea Script de PowerShell:
    * **Versión de la tarea**: 2.*
    * **Nombre para mostrar**: El nombre de la directiva que debe cargar esta tarea. Por ejemplo, *B2C_1A_TrustFrameworkBase*.
    * **Tipo**: Ruta del archivo
    * **Ruta de acceso del script**: Seleccione los puntos suspensivos (**_..._* _), vaya a la carpeta _Scripts* y, a continuación, seleccione el archivo *DeployToB2C.ps1*.
    * **Argumentos**:

        Escriba los siguientes valores para **Argumentos**. Reemplace `{alias-name}` por el alias que especificó en la sección anterior.

        ```PowerShell
        # Before
        -ClientID $(clientId) -ClientSecret $(clientSecret) -TenantId $(tenantId) -PolicyId B2C_1A_TrustFrameworkBase -PathToFile $(System.DefaultWorkingDirectory)/{alias-name}/B2CAssets/TrustFrameworkBase.xml
        ```

        Por ejemplo, si el alias especificado es *policyRepo*, la línea de argumento debe ser:

        ```PowerShell
        # After
        -ClientID $(clientId) -ClientSecret $(clientSecret) -TenantId $(tenantId) -PolicyId B2C_1A_TrustFrameworkBase -PathToFile $(System.DefaultWorkingDirectory)/policyRepo/B2CAssets/TrustFrameworkBase.xml
        ```

1. Seleccione **Guardar** para guardar el trabajo de agente.

La tarea que acaba de agregar carga *un* archivo de directiva en Azure AD B2C. Antes de continuar, desencadene manualmente el trabajo (**Crear versión**) para asegurarse de que se completa correctamente antes de crear tareas adicionales.

Si la tarea se completa correctamente, agregue tareas de implementación siguiendo los pasos anteriores para cada uno de los archivos de directiva personalizada. Modifique los valores de los argumentos `-PolicyId` y `-PathToFile` para cada directiva.

`PolicyId` es un valor que se encuentra al principio de un archivo de directiva XML dentro del nodo TrustFrameworkPolicy. Por ejemplo, `PolicyId` en el siguiente XML de directiva es *B2C_1A_TrustFrameworkBase*:

```xml
<TrustFrameworkPolicy
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:xsd="http://www.w3.org/2001/XMLSchema"
xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
PolicySchemaVersion="0.3.0.0"
TenantId="contoso.onmicrosoft.com"
PolicyId= "B2C_1A_TrustFrameworkBase"
PublicPolicyUri="http://contoso.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
```

Al ejecutar los agentes y cargar los archivos de directivas, asegúrese de que se cargan en este orden:

1. *TrustFrameworkBase.xml*
1. *TrustFrameworkExtensions.xml*
1. *SignUpOrSignin.xml*
1. *ProfileEdit.xml*
1. *PasswordReset.xml*

Identity Experience Framework exige este orden, ya que la estructura de archivos se basa en una cadena jerárquica.

## <a name="test-your-pipeline"></a>Prueba de la canalización

Para probar la canalización de versión:

1. Seleccione **Canalizaciones** y luego **Versiones**.
1. Seleccione la canalización que creó anteriormente, por ejemplo *DeployCustomPolicies*.
1. Seleccione **Crear versión** y, a continuación, seleccione **Crear** para poner en cola la versión.

Debería ver un banner de notificación que indica que una versión se ha puesto en cola. Para ver su estado, seleccione el vínculo en el banner de notificación o selecciónelo en la lista de la pestaña **Versiones**.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre:

* [Llamadas entre servicios mediante las credenciales del cliente](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md)
* [Azure DevOps Services](/azure/devops/user-guide/)

<!-- LINKS - External -->
[devops]: /azure/devops/
[devops-create-project]:  /azure/devops/organizations/projects/create-project
[devops-pipelines]: /azure/devops/pipelines
