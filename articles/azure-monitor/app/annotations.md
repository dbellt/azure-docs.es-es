---
title: Anotaciones de la versión de Application Insights | Microsoft Docs
description: Agregue marcadores de implementación o compilación a sus gráficos del Explorador de métricas en Application Insights.
ms.topic: conceptual
ms.date: 08/14/2020
ms.openlocfilehash: 9132e65e4705fd9125d97a5e095fe5f0850229a2
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107011057"
---
# <a name="annotations-on-metric-charts-in-application-insights"></a>Anotaciones sobre gráficos de métricas en Application Insights

Las anotaciones muestran dónde ha implementado una nueva compilación u otros eventos importantes. Las anotaciones permiten ver fácilmente si los cambios tuvieron algún efecto en el rendimiento de la aplicación. Se pueden crear automáticamente con el sistema de compilación de [Azure Pipelines](/azure/devops/pipelines/tasks/). También puede crear anotaciones para marcar los eventos que prefiera si los crea desde PowerShell.

## <a name="release-annotations-with-azure-pipelines-build"></a>Anotaciones de la versión con una compilación de Azure Pipelines

Las anotaciones de la versión son una característica del servicio Azure Pipelines basado en la nube de Azure DevOps.

### <a name="install-the-annotations-extension-one-time"></a>Instalación de la extensión Anotaciones (una vez)

Para poder crear anotaciones de la versión, debe instalar una de las muchas extensiones de Azure DevOps disponibles en Visual Studio Marketplace.

1. Inicie sesión en su proyecto de [Azure DevOps](https://azure.microsoft.com/services/devops/).
   
1. En la página de [extensión de anotaciones de la versión](https://marketplace.visualstudio.com/items/ms-appinsights.appinsightsreleaseannotations) de Visual Studio Marketplace, seleccione su organización de Azure DevOps y, a continuación, seleccione **Instalar** para agregar la extensión a la organización de Azure DevOps.
   
   ![Seleccione una organización de Azure DevOps y, a continuación, seleccione Instalar.](./media/annotations/1-install.png)
   
Solo deberá instalar la extensión una vez para su organización de Azure DevOps. Las anotaciones de la versión se pueden configurar ahora para cualquier proyecto de su organización.

### <a name="configure-release-annotations"></a>Configurar anotaciones de versión

Cree una clave de API independiente para cada una de las plantillas de versión de Azure Pipelines.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra el recurso de Application Insights que supervisa su aplicación. Si no tiene uno, [cree un nuevo recurso de Application Insights](./app-insights-overview.md).
   
1. Abra la pestaña **Acceso de API** y copie el **Id. de Application Insights**.
   
   ![En Acceso de API, copie el identificador de la aplicación.](./media/annotations/2-app-id.png)

1. En una ventana del explorador independiente, abra o cree la plantilla de versión que administra sus implementaciones de Azure Pipelines.
   
1. Seleccione **Agregar tarea** y, a continuación, seleccione la tarea **Anotación de versión de Application Insights** en el menú.
   
   ![Seleccione Agregar tarea y seleccione Anotación de versión de Application Insights.](./media/annotations/3-add-task.png)

   > [!NOTE]
   > Actualmente, la tarea Anotación de versión solo admite agentes basados en Windows; no se ejecuta en Linux, macOS ni otros tipos de agentes.
   
1. En **Id. de aplicación**, pegue el identificador de Application Insights que copió de la pestaña **Acceso de API**.
   
   ![Pegue el identificador de Application Insights.](./media/annotations/4-paste-app-id.png)
   
1. De vuelta en la ventana **Acceso de API** de Application Insights, seleccione **Crear clave de API**. 
   
   ![En la pestaña Acceso de API, seleccione Crear clave de API.](./media/annotations/5-create-api-key.png)
   
1. En la ventana **Crear clave de API**, escriba una descripción, seleccione **Escribir anotaciones** y, a continuación, seleccione **Generar clave**. Copie la clave nueva.
   
   ![En la ventana Crear clave de API, escriba una descripción, seleccione Escribir anotaciones y, a continuación, seleccione Generar clave.](./media/annotations/6-create-api-key.png)
   
1. En la ventana de la plantilla de versión, en la pestaña **Variables**, seleccione **Agregar** para crear una definición de variable para la nueva clave de API.

1. En **Nombre**, escriba `ApiKey` y, en **Valor**, pegue la clave de API que copió en la pestaña **Acceso de API**.
   
   ![En la pestaña Variables de Azure DevOps, seleccione Agregar, asigne a la variable el nombre ApiKey y pegue la clave de API en Valor.](./media/annotations/7-paste-api-key.png)
   
1. Seleccione **Guardar** en la ventana de la plantilla de versión principal para guardar la plantilla.


   > [!NOTE]
   > Los límites de las claves de API se describen en la [documentación de los límites de frecuencia de la API de REST](https://dev.applicationinsights.io/documentation/Authorization/Rate-limits).

## <a name="view-annotations"></a>Ver anotaciones


   > [!NOTE]
   > Las anotaciones de versión no están disponibles actualmente en el panel de métricas de Application Insights

Ahora, cuando utilice la plantilla de versión para implementar una nueva versión, se enviará una anotación a Application Insights. Las anotaciones se pueden ver en las siguientes ubicaciones:

El panel **Uso**, donde también puede crear anotaciones de versión manualmente:

![Captura de pantalla del gráfico de barras con el número de visitas de usuario mostradas durante un período de horas. Las anotaciones de versión aparecen como marcas de verificación verdes sobre el gráfico e indican el momento dado en que se produjo una versión](./media/annotations/usage-pane.png)

En cualquier consulta de libro basada en registros en la que la visualización muestre el tiempo a lo largo del eje x.

![Captura de pantalla del panel de libros con una consulta basada en registro de serie temporal con las anotaciones mostradas](./media/annotations/workbooks-annotations.png)

Para habilitar anotaciones en el libro, vaya a **Configuración avanzada** y seleccione **Mostrar anotaciones**.

![Captura de pantalla del menú Configuración avanzada con las palabras Mostrar anotaciones resaltadas con una marca de verificación junto a la opción para habilitar la opción.](./media/annotations/workbook-show-annotations.png)

Seleccione un marcador de anotación para abrir los detalles de la versión, incluidos el solicitante, la rama de control de código fuente, la canalización de la versión y el entorno.

## <a name="create-custom-annotations-from-powershell"></a>Crear anotaciones personalizadas desde PowerShell
Puede usar el script de PowerShell CreateReleaseAnnotation para crear anotaciones desde cualquier proceso que desee, sin usar Azure DevOps.

> [!IMPORTANT]
> Si usa PowerShell 7.1, agregue `-SkipHttpErrorCheck` al final de la línea 26. Por ejemplo: `$request = Invoke-WebRequest -Uri $fwLink -MaximumRedirection 0 -UseBasicParsing -ErrorAction Ignore -SkipHttpErrorCheck`.

1. Realice una copia local de CreateReleaseAnnotation.ps1:

    ```powershell
    
    # Copyright (c) Microsoft Corporation. All rights reserved. 
    # Licensed under the MIT License. See License.txt in the project root for license information. 
    
    # Sample usage .\CreateReleaseAnnotation.ps1 -applicationId "<appId>" -apiKey "<apiKey>" -releaseFilePath "<path to .exe with file version>" -releaseProperties @{"ReleaseDescription"="Release with annotation";"TriggerBy"="John Doe"}
    param(
        [parameter(Mandatory = $true)][string]$applicationId,
        [parameter(Mandatory = $true)][string]$apiKey,
        [parameter(Mandatory = $true)][string]$releaseFilePath,
        [parameter(Mandatory = $false)]$releaseProperties
    )
    
    $releaseName = (Get-Item $releaseFilePath).VersionInfo.FileVersion
    Write-Host "Creating release annotation $releaseName in ApplicationInsights" -ForegroundColor Cyan
    
    # background info on how fwlink works: After you submit a web request, many sites redirect through a series of intermediate pages before you finally land on the destination page.
    # So when calling Invoke-WebRequest, the result it returns comes from the final page in any redirect sequence. Hence, I set MaximumRedirection to 0, as this prevents the call to 
    # be redirected. By doing this, we get a response with status code 302, which indicates that there is a redirection link from the response body. We grab this redirection link and 
    # construct the url to make a release annotation.
    # Here's how this logic is going to works
    # 1. Client send http request, such as:  http://go.microsoft.com/fwlink/?LinkId=625115
    # 2. FWLink get the request and find out the destination URL for it, such as:  http://www.bing.com
    # 3. FWLink generate a new http response with status code “302” and with destination URL “http://www.bing.com”. Send it back to Client.
    # 4. Client, such as a powershell script, knows that status code “302” means redirection to new a location, and the target location is “http://www.bing.com”
    function GetRequestUrlFromFwLink($fwLink)
    {
        $request = Invoke-WebRequest -Uri $fwLink -MaximumRedirection 0 -UseBasicParsing -ErrorAction Ignore
        if ($request.StatusCode -eq "302") {
            return $request.Headers.Location
        }
        
        return $null
    }
    
    function CreateAnnotation($grpEnv)
    {
        $retries = 1
        $success = $false
        while (!$success -and $retries -lt 6) {
            $location = "$grpEnv/applications/$applicationId/Annotations?api-version=2015-11"
                
            Write-Host "Invoke a web request for $location to create a new release annotation. Attempting $retries"
            set-variable -Name createResultStatus -Force -Scope Local -Value $null
            set-variable -Name createResultStatusDescription -Force -Scope Local -Value $null
            set-variable -Name result -Force -Scope Local
    
            try {
                $result = Invoke-WebRequest -Uri $location -Method Put -Body $bodyJson -Headers $headers -ContentType "application/json; charset=utf-8" -UseBasicParsing
            } catch {
                if ($_.Exception){
                    if($_.Exception.Response) {
                        $createResultStatus = $_.Exception.Response.StatusCode.value__
                        $createResultStatusDescription = $_.Exception.Response.StatusDescription
                    }
                    else {
                        $createResultStatus = "Exception"
                        $createResultStatusDescription = $_.Exception.Message
                    }
                }
            }
    
            if ($result -eq $null) {
                if ($createResultStatus -eq $null) {
                    $createResultStatus = "Unknown"
                }
                if ($createResultStatusDescription -eq $null) {
                    $createResultStatusDescription = "Unknown"
                }
            }
            else {
                    $success = $true                     
            }
    
            if ($createResultStatus -eq 409 -or $createResultStatus -eq 404 -or $createResultStatus -eq 401) # no retry when conflict or unauthorized or not found
            {
                break
            }
    
            $retries = $retries + 1
            sleep 1
        }
    
        $createResultStatus
        $createResultStatusDescription
        return
    }
    
    # Need powershell version 3 or greater for script to run
    $minimumPowershellMajorVersion = 3
    if ($PSVersionTable.PSVersion.Major -lt $minimumPowershellMajorVersion) {
       Write-Host "Need powershell version $minimumPowershellMajorVersion or greater to create release annotation"
       return
    }
    
    $currentTime = (Get-Date).ToUniversalTime()
    $annotationDate = $currentTime.ToString("MMddyyyy_HHmmss")
    set-variable -Name requestBody -Force -Scope Script
    $requestBody = @{}
    $requestBody.Id = [GUID]::NewGuid()
    $requestBody.AnnotationName = $releaseName
    $requestBody.EventTime = $currentTime.GetDateTimeFormats("s")[0] # GetDateTimeFormats returns an array
    $requestBody.Category = "Deployment"
    
    if ($releaseProperties -eq $null) {
        $properties = @{}
    } else {
        $properties = $releaseProperties    
    }
    $properties.Add("ReleaseName", $releaseName)
    
    $requestBody.Properties = ConvertTo-Json($properties) -Compress
    
    $bodyJson = [System.Text.Encoding]::UTF8.GetBytes(($requestBody | ConvertTo-Json))
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add("X-AIAPIKEY", $apiKey)
    
    set-variable -Name createAnnotationResult1 -Force -Scope Local -Value $null
    set-variable -Name createAnnotationResultDescription -Force -Scope Local -Value ""
    
    # get redirect link from fwlink
    $requestUrl = GetRequestUrlFromFwLink("http://go.microsoft.com/fwlink/?prd=11901&pver=1.0&sbp=Application%20Insights&plcid=0x409&clcid=0x409&ar=Annotations&sar=Create%20Annotation")
    if ($requestUrl -eq $null) {
        $output = "Failed to find the redirect link to create a release annotation"
        throw $output
    }
    
    $createAnnotationResult1, $createAnnotationResultDescription = CreateAnnotation($requestUrl)
    if ($createAnnotationResult1) 
    {
         $output = "Failed to create an annotation with Id: {0}. Error {1}, Description: {2}." -f $requestBody.Id, $createAnnotationResult1, $createAnnotationResultDescription
         throw $output
    }
    
    $str = "Release annotation created. Id: {0}." -f $requestBody.Id
    Write-Host $str -ForegroundColor Green
    
    ```
   
1. Use los pasos del procedimiento anterior para obtener el identificador de Application Insights y crear una clave de API desde la pestaña **Acceso de API** de Application Insights.
   
1. Llame al script de PowerShell con el código siguiente, reemplazando los marcadores de posición entre corchetes angulares por sus valores. Los valores de `-releaseProperties` son opcionales. 
   
   ```powershell
   
        .\CreateReleaseAnnotation.ps1 `
         -applicationId "<applicationId>" `
         -apiKey "<apiKey>" `
         -releaseName "<releaseName>" `
         -releaseProperties @{
             "ReleaseDescription"="<a description>";
             "TriggerBy"="<Your name>" }
   ```

El script se puede modificar (por ejemplo, para crear anotaciones para el pasado).


## <a name="next-steps"></a>Pasos siguientes

* [Crear elementos de trabajo](./diagnostic-search.md#create-work-item)
* [Automation con PowerShell](./powershell.md)
