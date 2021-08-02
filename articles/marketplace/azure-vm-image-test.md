---
title: Prueba de una imagen de máquina virtual de Azure para Azure Marketplace
description: Pruebe y envíe una oferta de máquina virtual de Azure en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: kriti-ms
ms.author: krsh
ms.date: 03/10/2021
ms.openlocfilehash: c885f7e953849825c9a0ef87f68df6e9e2d3360a
ms.sourcegitcommit: 70ce9237435df04b03dd0f739f23d34930059fef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2021
ms.locfileid: "111528583"
---
# <a name="test-a-virtual-machine-image"></a>Prueba de una imagen de máquina virtual

En este tema se explican los pasos para probar una imagen de máquina virtual (VM) para la implementación en Azure Marketplace.

## <a name="deploy-an-azure-vm"></a>Implementación de una máquina virtual de Azure

Para implementar una máquina virtual desde la imagen de Shared Image Gallery:

1. Vaya a versión de la imagen de Shared Image Gallery.
1. Haga clic en Crear máquina virtual.
1. Proporcione un nombre de máquina virtual y seleccione un tamaño de máquina virtual.
1. Haga clic en Revisar y crear. Una vez superada la validación, haga clic en Crear.

> [!NOTE]
> Si necesita crear una máquina virtual a partir de un archivo VHD, siga las instrucciones de los siguientes artículos: [Preparación de una plantilla de Azure Resource Manager](#connect-the-certification-tool-to-a-vm-image) o [Implementación de una máquina virtual de Azure con PowerShell](#how-to-use-powershell-to-consume-the-self-test-api).

En este artículo se describe cómo probar y enviar una imagen de máquina virtual (VM) en Marketplace comercial para asegurarse de que cumple los requisitos de publicación más recientes de Azure Marketplace.

Siga estos pasos antes de enviar la oferta de máquina virtual:

- Implemente una máquina virtual de Azure con la imagen generalizada; consulte [Creación de una máquina virtual con una base aprobada](azure-vm-create-using-approved-base.md) o [Creación de una máquina virtual con su propia imagen](azure-vm-create-using-own-image.md).
- Ejecute las validaciones.

## <a name="run-validations"></a>Ejecución de validaciones

Hay dos maneras de ejecutar validaciones en la imagen implementada.

### <a name="use-certification-test-tool-for-azure-certified"></a>Usar Certification Test Tool for Azure Certified

#### <a name="download-and-run-the-certification-test-tool"></a>Descargar y ejecutar Certification Test Tool

Certification Test Tool for Azure Certified se ejecuta en un equipo Windows local, pero prueba una máquina virtual Linux o Windows basada en Azure. Verifica que la imagen de máquina virtual de usuario se puede usar con Microsoft Azure y que se han cumplido las instrucciones y los requisitos relacionados con la preparación del VHD.

1. Descargue e instale la versión más reciente de [Certification Test Tool for Azure Certified](https://www.microsoft.com/download/details.aspx?id=44299).
2. Abra la herramienta de certificación y seleccione **Iniciar nueva prueba**.
3. En la pantalla Información de la prueba, escriba un **Nombre de prueba** para la serie de pruebas.
4. Seleccione la plataforma de la máquina virtual, ya sea **Windows Server** o **Linux**. La elección de plataforma afecta a las opciones restantes.
5. Si la máquina virtual usa este servicio de base de datos, seleccione la casilla **Prueba de Azure SQL Database**.

#### <a name="connect-the-certification-tool-to-a-vm-image"></a>Conectar la herramienta de certificación a una imagen de máquina virtual

1. Seleccione el modo Autenticación SSH: Autenticación de contraseña o autenticación de archivo de clave.
2. Si usa la autenticación de contraseña, escriba los valores para **Nombre DNS de máquina virtual**, **Nombre de usuario** y **Contraseña**. También puede cambiar el número predeterminado de Puerto SSH.

    :::image type="content" source="media/vm/azure-vm-cert-2.png" alt-text="Muestra la selección de la información de la prueba de la máquina virtual.":::

3. Si usa autenticación con archivo de clave, escriba valores para Nombre DNS de máquina virtual, Nombre de usuario y Ubicación de clave privada. También puede incluir una Frase de contraseña o cambiar el número predeterminado de Puerto SSH.
4. Escriba el nombre DNS completo de la máquina virtual (por ejemplo, MyVMName.ClOudapp.net).
5. Escriba el **Nombre de usuario** y la **Contraseña**.

    :::image type="content" source="media/vm/azure-vm-cert-4.png" alt-text="Muestra la selección del nombre de usuario y contraseña de la máquina virtual.":::

6. Seleccione **Next** (Siguiente).

#### <a name="run-a-certification-test"></a>Ejecutar una prueba de certificación

Una vez proporcionados los valores de parámetro para la imagen de máquina virtual en la herramienta de certificación, seleccione Probar conexión para crear una conexión válida a la máquina virtual. Una vez verificada la conexión, seleccione **Siguiente** para iniciar la prueba. Una vez completada la prueba, los resultados se muestran en una tabla. La columna Estado se muestra (Correcto/Error/Advertencia) para cada prueba. Si se produce un error en alguna de las pruebas, la imagen no se certifica. En ese caso, revise los requisitos y los mensajes de error, realice los cambios sugeridos y vuelva a ejecutar la prueba.

Una vez completada la prueba automatizada, proporcione información adicional sobre la imagen de máquina virtual en las dos pestañas de la pantalla Cuestionario, es decir, Evaluación general y Personalización del kernel, y, después, seleccione Siguiente.

La última pantalla permite proporcionar más información, como datos de acceso SSH para una imagen de máquina virtual Linux y una explicación sobre las evaluaciones con errores, en caso de que busque excepciones.

Por último, seleccione Generar informe para descargar los resultados de las pruebas y los archivos de registro de los casos de prueba ejecutados, junto con las respuestas al cuestionario.
> [!Note]
> algunos publicadores tienen escenarios donde es necesario bloquear las máquinas virtuales, ya que tienen software como firewalls instalados en la máquina virtual. En este caso, descargue la [herramienta de prueba certificada](https://aka.ms/AzureCertificationTestTool) aquí y envíe el informe al [soporte técnico](https://aka.ms/marketplacepublishersupport) del Centro de partners.

## <a name="how-to-use-powershell-to-consume-the-self-test-api"></a>Cómo usar PowerShell para consumir la API de prueba automática

### <a name="on-linux-os"></a>En el SO Linux

Llame a la API en PowerShell:

1. Use el comando Invoke-WebRequest para llamar a la API.
2. El método es Post y el tipo de contenido es JSON, tal como se muestra en la siguiente captura de pantalla y el ejemplo de código.
3. Especifique los parámetros del cuerpo en formato JSON.

El ejemplo siguiente muestra una llamada de PowerShell a la API:

```POWERSHELL
$accesstoken = "token"
$headers = @{ "Authorization" = "Bearer $accesstoken" }
$DNSName = "<Machine DNS Name>"
$UserName = "<User ID>"
$Password = "<Password>"
$OS = "Linux"
$PortNo = "22"
$CompanyName = "ABCD"
$AppID = "<Application ID>"
$TenantId = "<Tenant ID>"

$body = @{
   "DNSName" = $DNSName
   "UserName" = $UserName
   "Password" = $Password
   "OS" = $OS
   "PortNo" = $PortNo
   "CompanyName" = $CompanyName
   "AppID" = $AppID
   "TenantId" = $TenantId
} | ConvertTo-Json

$body

$uri = "URL"

$res = (Invoke-WebRequest -Method "Post" -Uri $uri -Body $body -ContentType "application/json" -Headers $headers).Content
```

<br>Este es un ejemplo de llamada a la API en PowerShell:

[![Ejemplo de pantalla de una llamada a la API en PowerShell.](media/vm/call-api-in-powershell.png)](media/vm/call-api-in-powershell.png#lightbox)

<br>Mediante el ejemplo anterior, puede recuperar el archivo JSON y analizarlo para obtener los detalles siguientes:

```PowerShell
$resVar = $res | ConvertFrom-Json
$actualresult = $resVar.Response | ConvertFrom-Json

Write-Host "OSName: $($actualresult.OSName)"
Write-Host "OSVersion: $($actualresult.OSVersion)"
Write-Host "Overall Test Result: $($actualresult.TestResult)"

For ($i = 0; $i -lt $actualresult.Tests.Length; $i++) {
   Write-Host "TestID: $($actualresult.Tests[$i].TestID)"
   Write-Host "TestCaseName: $($actualresult.Tests[$i].TestCaseName)"
   Write-Host "Description: $($actualresult.Tests[$i].Description)"
   Write-Host "Result: $($actualresult.Tests[$i].Result)"
   Write-Host "ActualValue: $($actualresult.Tests[$i].ActualValue)"
}
```

<br>En esta pantalla de ejemplo, que muestra `$res.Content`, se pueden ver los detalles de los resultados de las pruebas en formato JSON:

[![Ejemplo de pantalla de una llamada a la API en PowerShell con los detalles de los resultados de la prueba.](media/vm/call-api-in-powershell-details.png)](media/vm/call-api-in-powershell-details.png#lightbox)

<br>Este es un ejemplo de los resultados JSON de las pruebas que se ven en un visor JSON en línea (por ejemplo, [Code Beautify](https://codebeautify.org/jsonviewer) o [JSON Viewer](https://jsonformatter.org/json-viewer)).

![Más resultados de pruebas en un visor JSON en línea.](media/vm/test-results-json-viewer-1.png)

### <a name="on-windows-os"></a>En el SO Windows

Llame a la API en PowerShell:

1. Use el comando Invoke-WebRequest para llamar a la API.
2. El método es Post y el tipo de contenido es JSON, tal como se muestra en la siguiente pantalla de ejemplo y en el ejemplo de código.
3. Cree los parámetros del cuerpo en formato JSON.

El siguiente ejemplo de código muestra una llamada de PowerShell a la API:

```PowerShell
$accesstoken = "Get token for your Client AAD App"
$headers = @{ "Authorization" = "Bearer $accesstoken" }
$Body = @{ 
   "DNSName" = "XXXX.westus.cloudapp.azure.com"
   "UserName" = "XXX"
   "Password" = "XXX@123456"
   "OS" = "Windows"
   "PortNo" = "5986"
   "CompanyName" = "ABCD"
   "AppID" = "XXXX-XXXX-XXXX"
   "TenantId" = "XXXX-XXXX-XXXX"
} | ConvertTo-Json

$res = Invoke-WebRequest -Method "Post" -Uri $uri -Body $Body -ContentType "application/json" –Headers $headers;
$Content = $res | ConvertFrom-Json
```

Estos ejemplos de pantalla muestra una llamada a la API en PowerShell:

**Con la clave SSH**:

 :::image type="content" source="media/vm/call-api-with-ssh-key.png" alt-text="Llamar a la API en PowerShell con una clave SSH.":::

**Con contraseña**:

 :::image type="content" source="media/vm/call-api-with-password.png" alt-text="Llamar a la API en PowerShell con una contraseña.":::

<br>Mediante el ejemplo anterior, puede recuperar el archivo JSON y analizarlo para obtener los detalles siguientes:

```PowerShell
$resVar = $res | ConvertFrom-Json
$actualresult = $resVar.Response | ConvertFrom-Json

Write-Host "OSName: $($actualresult.OSName)"
Write-Host "OSVersion: $($actualresult.OSVersion)"
Write-Host "Overall Test Result: $($actualresult.TestResult)"

For ($i = 0; $i -lt $actualresult.Tests.Length; $i++) {
   Write-Host "TestID: $($actualresult.Tests[$i].TestID)"
   Write-Host "TestCaseName: $($actualresult.Tests[$i].TestCaseName)"
   Write-Host "Description: $($actualresult.Tests[$i].Description)"
   Write-Host "Result: $($actualresult.Tests[$i].Result)"
   Write-Host "ActualValue: $($actualresult.Tests[$i].ActualValue)"
}
```

<br>En esta pantalla, que muestra `$res.Content`, se pueden ver los detalles de los resultados de las pruebas en formato JSON:

 :::image type="content" source="media/vm/test-results-json-format.png" alt-text="Detalles de los resultados de la prueba en formato JSON.":::

<br>Este es un ejemplo de los resultados de las pruebas que se ven en un visor JSON en línea (por ejemplo, [Code Beautify](https://codebeautify.org/jsonviewer) o [JSON Viewer](https://jsonformatter.org/json-viewer)).

![Resultados de pruebas en un visor JSON en línea.](media/vm/test-results-json-viewer-2.png)

## <a name="how-to-use-curl-to-consume-the-self-test-api-on-linux-os"></a>Cómo usar cURL para consumir la API de prueba automática en el SO Linux

En este ejemplo, se usará curl para realizar una llamada de API POST a Azure Active Directory y la máquina virtual autohospedada.

1. Solicitar un token de Azure AD para autenticarse en una máquina virtual autohospedada

   Asegúrese de que se sustituyen los valores correctos en la solicitud de curl.

   ```JSON
   curl --location --request POST 'https://login.microsoftonline.com/{TENANT_ID}/oauth2/token' \
   --header 'Content-Type: application/x-www-form-urlencoded' \
   --data-urlencode 'grant_type=client_credentials' \
   --data-urlencode 'client_id={CLIENT_ID} ' \
   --data-urlencode 'client_secret={CLIENT_SECRET}' \
   --data-urlencode 'resource=https://management.core.windows.net'
   ```

   Este es un ejemplo de la respuesta de la solicitud:

   ```JSON
   {
       "token_type": "Bearer",
       "expires_in": "86399",
       "ext_expires_in": "86399",
       "expires_on": "1599663998",
       "not_before": "1599577298",
       "resource": "https://management.core.windows.net",
       "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJS…"
   }
   ```

2. Enviar una solicitud para la máquina virtual de prueba automática

   Asegúrese de que el token de portador y los parámetros se sustituyen por los valores correctos.

   ```JSON
   curl --location --request POST 'https://isvapp.azurewebsites.net/selftest-vm' \
   --header 'Content-Type: application/json' \
   --header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJS…' \
   --data-raw '{
       "DNSName": "avvm1.eastus.cloudapp.azure.com",
       "UserName": "azureuser",
       "Password": "SECURE_PASSWORD_FOR_THE_SSH_INTO_VM",
       "OS": "Linux",
       "PortNo": "22",
       "CompanyName": "COMPANY_NAME",
       "AppId": "CLIENT_ID_SAME_AS_USED_FOR_AAD_TOKEN ",
       "TenantId": "TENANT_ID_SAME_AS_USED_FOR_AAD_TOKEN"
   }'
   ```

   Respuesta de ejemplo de la llamada API de máquina virtual de prueba automática

   ```JSON
   {
       "TrackingId": "9bffc887-dd1d-40dd-a8a2-34cee4f4c4c3",
       "Response": "{\"SchemaVersion\":1,\"AppCertificationCategory\":\"Microsoft Single VM Certification\",\"ProviderID\":\"050DE427-2A99-46D4-817C-5354D3BF2AE8\",\"OSName\":\"Ubuntu 18.04\",\"OSDistro\":\"Ubuntu 18.04.5 LTS\",\"KernelVersion\":\"5.4.0-1023-azure\",\"KernelFullVersion\":\"Linux version 5.4.0-1023-azure (buildd@lgw01-amd64-053) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #23~18.04.1-Ubuntu SMP Thu Aug 20 14:46:48 UTC 2020\\n\",\"OSVersion\":\"18.04\",\"CreatedDate\":\"09/08/2020 01:13:47\",\"TestResult\":\"Pass\",\"APIVersion\":\"1.5\",\"Tests\":[{\"TestID\":\"48\",\"TestCaseName\":\"Bash History\",\"Description\":\"Bash history files should be cleared before creating the VM image.\",\"Result\":\"Passed\",\"ActualValue\":\"No file Exist\",\"RequiredValue\":\"1024\"},{\"TestID\":\"39\",\"TestCaseName\":\"Linux Agent Version\",\"Description\":\"Azure Linux Agent Version 2.2.41 and above should be installed.\",\"Result\":\"Passed\",\"ActualValue\":\"2.2.49\",\"RequiredValue\":\"2.2.41\"},{\"TestID\":\"40\",\"TestCaseName\":\"Required Kernel Parameters\",\"Description\":\"Verifies the following kernel parameters are set console=ttyS0, earlyprintk=ttyS0, rootdelay=300\",\"Result\":\"Warning\",\"ActualValue\":\"Missing Parameter: rootdelay=300\\r\\nMatched Parameter: console=ttyS0,earlyprintk=ttyS0\",\"RequiredValue\":\"console=ttyS0#earlyprintk=ttyS0#rootdelay=300\"},{\"TestID\":\"41\",\"TestCaseName\":\"Swap Partition on OS Disk\",\"Description\":\"Verifies that no Swap partitions are created on the OS disk.\",\"Result\":\"Passed\",\"ActualValue\":\"No. of Swap Partitions: 0\",\"RequiredValue\":\"swap\"},{\"TestID\":\"42\",\"TestCaseName\":\"Root Partition on OS Disk\",\"Description\":\"It is recommended that a single root partition is created for the OS disk.\",\"Result\":\"Passed\",\"ActualValue\":\"Root Partition: 1\",\"RequiredValue\":\"1\"},{\"TestID\":\"44\",\"TestCaseName\":\"OpenSSL Version\",\"Description\":\"OpenSSL Version should be >=0.9.8.\",\"Result\":\"Passed\",\"ActualValue\":\"1.1.1\",\"RequiredValue\":\"0.9.8\"},{\"TestID\":\"45\",\"TestCaseName\":\"Python Version\",\"Description\":\"Python version 2.6+ is highly recommended. \",\"Result\":\"Passed\",\"ActualValue\":\"2.7.17\",\"RequiredValue\":\"2.6\"},{\"TestID\":\"46\",\"TestCaseName\":\"Client Alive Interval\",\"Description\":\"It is recommended to set ClientAliveInterval to 180. On the application need, it can be set between 30 to 235. \\nIf you are enabling the SSH for your end users this value must be set as explained.\",\"Result\":\"Warning\",\"ActualValue\":\"120\",\"RequiredValue\":\"ClientAliveInterval 180\"},{\"TestID\":\"49\",\"TestCaseName\":\"OS Architecture\",\"Description\":\"Only 64-bit operating system should be supported.\",\"Result\":\"Passed\",\"ActualValue\":\"x86_64\\n\",\"RequiredValue\":\"x86_64,amd64\"},{\"TestID\":\"50\",\"TestCaseName\":\"Security threats\",\"Description\":\"Identifies OS with recent high profile vulnerability that may need patching.  Ignore warning if system was patched as appropriate.\",\"Result\":\"Passed\",\"ActualValue\":\"Ubuntu 18.04\",\"RequiredValue\":\"OS impacted by GHOSTS\"},{\"TestID\":\"51\",\"TestCaseName\":\"Auto Update\",\"Description\":\"Identifies if Linux Agent Auto Update is enabled or not.\",\"Result\":\"Passed\",\"ActualValue\":\"# AutoUpdate.Enabled=y\\n\",\"RequiredValue\":\"Yes\"},{\"TestID\":\"52\",\"TestCaseName\":\"SACK Vulnerability patch verification\",\"Description\":\"Checks if the running Kernel Version has SACK vulnerability patch.\",\"Result\":\"Passed\",\"ActualValue\":\"Ubuntu 18.04.5 LTS,Linux version 5.4.0-1023-azure (buildd@lgw01-amd64-053) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #23~18.04.1-Ubuntu SMP Thu Aug 20 14:46:48 UTC 2020\",\"RequiredValue\":\"Yes\"}]}"
   }
   ```

## <a name="next-steps"></a>Pasos siguientes

- Inicie sesión en el [Centro de partners](https://partner.microsoft.com/).
