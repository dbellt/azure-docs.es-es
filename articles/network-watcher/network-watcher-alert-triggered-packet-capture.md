---
title: 'Uso de capturas de paquetes para realizar la supervisión proactiva de la red con alertas: Azure Functions'
titleSuffix: Azure Network Watcher
description: En este artículo se describe cómo crear una captura de paquetes desencadenada mediante alertas con Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: 75e6e7c4-b3ba-4173-8815-b00d7d824e11
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e6aa706b0e8b1ae69e8c8e1f19ff72b6ad67f1f1
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110676186"
---
# <a name="use-packet-capture-for-proactive-network-monitoring-with-alerts-and-azure-functions"></a>Uso de capturas de paquetes para realizar la supervisión proactiva de la red con alertas y Azure Functions

La captura de paquetes de Network Watcher crea sesiones de captura para realizar el seguimiento del tráfico dentro y fuera de máquinas virtuales. El archivo de captura puede tener un filtro que se define para realizar el seguimiento solo del tráfico que se quiere supervisar. Estos datos se almacenan luego en un blob de almacenamiento o de forma local en la máquina invitada.

Esta funcionalidad se puede iniciar de forma remota desde otros escenarios de automatización, como Azure Functions. La captura de paquetes ofrece la funcionalidad de ejecutar capturas proactivas en función de las anomalías de red definidas. Otros usos son la recopilación de estadísticas de red, la obtención de información sobre las intrusiones de red y la depuración de las comunicaciones cliente-servidor, entre otros.

Los recursos implementados en Azure se ejecutan las 24 horas, los 7 días de la semana. Ni usted ni su personal pueden supervisar activamente el estado de todos los recursos de manera ininterrumpida. ¿Qué ocurre si un problema se produce a las 2 a. m.?

Con Network Watcher, la característica de alertas y las funciones del ecosistema de Azure, puede responder de manera proactiva con los datos y las herramientas para resolver los problemas de su red.

![En el diagrama se muestra la extensión Network Watcher en una máquina virtual que fluye a un error de segmentos TCP enviados > 100, que fluye a Azure Functions, que fluye a Network Watcher que fluye de nuevo a la extensión Network Watcher.][scenario]


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerequisites

* La versión más reciente de [Azure PowerShell](/powershell/azure/install-Az-ps).
* Una instancia existente de Network Watcher. [Cree una instancia de Network Watcher](network-watcher-create.md) si aún no tiene una.
* Una máquina virtual en la misma región que la instancia de Network Watcher con la [extensión Windows](../virtual-machines/extensions/network-watcher-windows.md) o la [extensión de máquina virtual Linux](../virtual-machines/extensions/network-watcher-linux.md).

## <a name="scenario"></a>Escenario

En este ejemplo, la máquina virtual está enviando más segmentos TCP de lo habitual, y usted desea recibir alertas al respecto. Los segmentos TCP se usan aquí como ejemplo, pero podría usar cualquier condición de alerta.

Cuando recibe una alerta, quiere obtener datos de nivel de paquete para comprender por qué ha aumentado la comunicación. De este modo, puede realizar los pasos necesarios para devolver la máquina virtual al estado de comunicación normal.

En este escenario se supone que tiene una instancia existente de Network Watcher y un grupo de recursos con una máquina virtual válida.

En la lista siguiente se muestra información general sobre el flujo de trabajo que tiene lugar:

1. Se desencadena una alerta en la máquina virtual.
1. La alerta llama a la función de Azure a través de un webhook.
1. La función de Azure procesa la alerta y se inicia una sesión de captura de paquetes de Network Watcher.
1. La captura de paquetes se ejecuta en la máquina virtual y recopila el tráfico.
1. El archivo de captura de paquetes se carga en una cuenta de almacenamiento para su revisión y diagnóstico.

Para automatizar este proceso, crearemos y conectaremos una alerta en nuestra máquina virtual para que se active cuando se produzca el incidente. También crearemos una función para llamar en Network Watcher.

En este escenario hará lo siguiente:

* Creará una función de Azure que inicie una captura de paquetes.
* Creará una regla de alerta en una máquina virtual y configurará la regla de alerta para llamar a la función de Azure.

## <a name="create-an-azure-function"></a>Creación de una función de Azure

El primer paso es crear una función de Azure para procesar la alerta y crear una captura de paquetes.

1. En [Azure Portal](https://portal.azure.com), haga clic en **Crear un recurso** > **Proceso** > **Function App**.

    ![Creación de una aplicación de función][1-1]

2. En la hoja **Function App**, escriba los siguientes valores y seleccione **Aceptar** para crear la aplicación:

    |**Configuración** | **Valor** | **Detalles** |
    |---|---|---|
    |**Nombre de la aplicación**|PacketCaptureExample|Nombre de la aplicación de función.|
    |**Suscripción**|[Su suscripción]La suscripción para la que se crea la aplicación de función.||
    |**Grupo de recursos**|PacketCaptureRG|Grupo de recursos que contendrá la aplicación de función.|
    |**Plan de hospedaje**|Plan de consumo| Tipo de plan que usa la aplicación de función. Las opciones son Consumo o plan de Azure App Service. |
    |**Ubicación**|Centro de EE. UU.| Región en la que se creará la aplicación de función.|
    |**Storage Account**|{autogenerated}| Cuenta de almacenamiento que necesita Azure Functions para el almacenamiento de carácter general.|

3. En la hoja **Function Apps de PacketCaptureExample**, seleccione **Funciones** > **Función personalizada** > **+** .

4. Seleccione **HttpTrigger-Powershell** y, a continuación, especifique la información restante. Por último, para crear la función, seleccione **Crear**.

    |**Configuración** | **Valor** | **Detalles** |
    |---|---|---|
    |**Escenario**|Experimental|Tipo de escenario|
    |**Asigne un nombre a la función**|AlertPacketCapturePowerShell|Nombre de la función.|
    |**Nivel de autorización**|Función|Nivel de autorización de la función|

![Ejemplo de funciones][functions1]

> [!NOTE]
> La plantilla de PowerShell es experimental y no es totalmente compatible.

Para este ejemplo se requieren personalizaciones, que se explican en los siguientes pasos.

### <a name="add-modules"></a>Adición de módulos

Para usar cmdlets de PowerShell de Network Watcher, cargue el módulo más reciente de PowerShell en la aplicación de función.

1. Una vez instalados los últimos módulos de Azure PowerShell en su equipo local, ejecute el siguiente comando de PowerShell:

    ```powershell
    (Get-Module Az.Network).Path
    ```

    Este ejemplo le proporciona la ruta de acceso local de los módulos de Azure PowerShell. Estas carpetas se usan en un paso posterior. A continuación, se muestran los módulos usados:

   * Az.Network

   * Az.Accounts

   * Az.Resources

     ![Carpetas de PowerShell][functions5]

1. Seleccione **Configuración de Function App** > **Ir al Editor de App Service**.

    ![Configuración de Function App][functions2]

1. Haga clic con el botón derecho en la carpeta **AlertPacketCapturePowershell** y cree una carpeta llamada **azuremodules**. 

4. Cree una subcarpeta para cada módulo que necesite.

    ![Carpeta y subcarpetas][functions3]

    * Az.Network

    * Az.Accounts

    * Az.Resources

1. Haga clic con el botón derecho en la subcarpeta **Az.Network** y seleccione **Cargar archivos**. 

6. Vaya a los módulos de Azure. En la carpeta **Az.Network** local, seleccione todos los archivos de la carpeta. Después, seleccione **Aceptar**. 

7. Repita estos pasos para las carpetas **Az.Accounts** y **Az.Resources**.

    ![Carga de archivos][functions6]

1. Cuando haya finalizado, cada carpeta debe tener los archivos del módulo de PowerShell del equipo local.

    ![Archivos de PowerShell][functions7]

### <a name="authentication"></a>Authentication

Para usar los cmdlets de PowerShell es preciso autenticarse. En la aplicación de función, se configura la autenticación. Para ello, debe configurar las variables de entorno y cargar un archivo de clave cifrada en la aplicación de función.

> [!NOTE]
> Este escenario proporciona solo un ejemplo de cómo implementar la autenticación con Azure Functions. Existen otras formas de hacerlo.

#### <a name="encrypted-credentials"></a>Credenciales cifradas

El siguiente script de PowerShell crea un archivo de claves denominado **PassEncryptKey.key**. También proporciona una versión cifrada de la contraseña que se proporciona. Esta contraseña es la misma que la definida para la aplicación de Azure Active Directory que se usa para la autenticación.

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

En el Editor de App Service de la aplicación de función, cree una carpeta denominada **keys** en **AlertPacketCapturePowerShell**. A continuación, cargue el archivo **PassEncryptKey.key** que creó en el ejemplo anterior de PowerShell.

![Clave de funciones][functions8]

### <a name="retrieve-values-for-environment-variables"></a>Recuperación de valores para variables de entorno

El último requisito es configurar las variables de entorno necesarias para tener acceso a los valores para la autenticación. En la siguiente lista, se muestran las variables de entorno que se crean:

* AzureClientID

* AzureTenant

* AzureCredPassword


#### <a name="azureclientid"></a>AzureClientID

El id. de cliente es el id. de aplicación de una aplicación de Azure Active Directory.

1. Si aún no cuenta con una aplicación que pueda usar, ejecute el siguiente ejemplo para crear una.

    ```powershell
    $app = New-AzADApplication -DisplayName "ExampleAutomationAccount_MF" -HomePage "https://exampleapp.com" -IdentifierUris "https://exampleapp1.com/ExampleFunctionsAccount" -Password "<same password as defined earlier>"
    New-AzADServicePrincipal -ApplicationId $app.ApplicationId
    Start-Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $app.ApplicationId
    ```

   > [!NOTE]
   > La contraseña utilizada al crearse la aplicación debe ser la misma que se creó anteriormente al guardarse el archivo de clave.

1. En Azure Portal, seleccione **Suscripciones**. Seleccione la suscripción que desee usar y, a continuación, seleccione **Control de acceso (IAM)** .

    ![IAM de Functions][functions9]

1. Elija la cuenta que usará y seleccione **Propiedades**. Copie el id. de aplicación.

    ![Id. de aplicación de Functions][functions10]

#### <a name="azuretenant"></a>AzureTenant

Obtenga el id. de inquilino ejecutando el ejemplo siguiente de PowerShell:

```powershell
(Get-AzSubscription -SubscriptionName "<subscriptionName>").TenantId
```

#### <a name="azurecredpassword"></a>AzureCredPassword

El valor de la variable de entorno AzureCredPassword es el valor resultante de la ejecución del ejemplo siguiente de PowerShell. Se trata del mismo ejemplo mostrado en la sección **Credenciales cifradas** anterior. El valor necesario es el resultado de la variable `$Encryptedpassword`.  Esta es la contraseña de entidad de servicio que cifró mediante el script de PowerShell.

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

### <a name="store-the-environment-variables"></a>Almacenamiento de las variables de entorno

1. Vaya a la aplicación de función. A continuación, seleccione **Configuración de Function App** > **Configurar las opciones de la aplicación**.

    ![Configuración de aplicaciones][functions11]

1. Agregue las variables de entorno y sus valores a la configuración de la aplicación y seleccione **Guardar**.

    ![Configuración de la aplicación][functions12]

### <a name="add-powershell-to-the-function"></a>Adición de PowerShell a la función

Ahora es el momento de realizar llamadas en Network Watcher dentro de la función de Azure. La implementación de esta función puede variar según los requisitos. Sin embargo, el flujo general del código es el siguiente:

1. Procesar los parámetros de entrada.
2. Consultar las capturas de paquetes existentes para comprobar los límites y resolver los conflictos de nombres.
3. Crear una captura de paquetes con los parámetros adecuados.
4. Sondear la captura de paquetes periódicamente hasta que finalice.
5. Notificar al usuario que la sesión de captura de paquetes ha finalizado.

El ejemplo siguiente es un código de PowerShell que se puede usar en la función. Hay valores que deben reemplazarse por **subscriptionId**, **resourceGroupName** y **storageAccountName**.

```powershell
            #Import Azure PowerShell modules required to make calls to Network Watcher
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Accounts\Az.Accounts.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Network\Az.Network.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Resources\Az.Resources.psd1" -Global

            #Process alert request body
            $requestBody = Get-Content $req -Raw | ConvertFrom-Json

            #Storage account ID to save captures in
            $storageaccountid = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}"

            #Packet capture vars
            $packetcapturename = "PSAzureFunction"
            $packetCaptureLimit = 10
            $packetCaptureDuration = 10

            #Credentials
            $tenant = $env:AzureTenant
            $pw = $env:AzureCredPassword
            $clientid = $env:AzureClientId
            $keypath = "D:\home\site\wwwroot\AlertPacketCapturePowerShell\keys\PassEncryptKey.key"

            #Authentication
            $secpassword = $pw | ConvertTo-SecureString -Key (Get-Content $keypath)
            $credential = New-Object System.Management.Automation.PSCredential ($clientid, $secpassword)
            Connect-AzAccount -ServicePrincipal -Tenant $tenant -Credential $credential #-WarningAction SilentlyContinue | out-null


            #Get the VM that fired the alert
            if($requestBody.context.resourceType -eq "Microsoft.Compute/virtualMachines")
            {
                Write-Output ("Subscription ID: {0}" -f $requestBody.context.subscriptionId)
                Write-Output ("Resource Group:  {0}" -f $requestBody.context.resourceGroupName)
                Write-Output ("Resource Name:  {0}" -f $requestBody.context.resourceName)
                Write-Output ("Resource Type:  {0}" -f $requestBody.context.resourceType)

                #Get the Network Watcher in the VM's region
                $networkWatcher = Get-AzResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq $requestBody.context.resourceRegion}

                #Get existing packetCaptures
                $packetCaptures = Get-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher

                #Remove existing packet capture created by the function (if it exists)
                $packetCaptures | %{if($_.Name -eq $packetCaptureName)
                { 
                    Remove-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName $packetCaptureName
                }}

                #Initiate packet capture on the VM that fired the alert
                if ((Get-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher).Count -lt $packetCaptureLimit){
                    echo "Initiating Packet Capture"
                    New-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -TargetVirtualMachineId $requestBody.context.resourceId -PacketCaptureName $packetCaptureName -StorageAccountId $storageaccountid -TimeLimitInSeconds $packetCaptureDuration
                    Out-File -Encoding Ascii -FilePath $res -inputObject "Packet Capture created on ${requestBody.context.resourceID}"
                }
            } 
 ``` 
#### <a name="retrieve-the-function-url"></a>Recuperación de la dirección URL de la función 
1. Cuando haya creado la función, configure la alerta para que llame a la dirección URL asociada a la función. Para obtener este valor, copie la dirección URL de función de la aplicación de función.

    ![Búsqueda de la dirección URL de la función][functions13]

2. Copie la dirección URL de la función de su aplicación de función.

    ![Copia de la dirección URL de la función][2]

Si necesita propiedades personalizadas en la carga útil de la solicitud POST del webhook, consulte [Configuración de un webhook en una alerta de métrica de Azure](../azure-monitor/alerts/alerts-webhooks.md).

## <a name="configure-an-alert-on-a-vm"></a>Configuración de una alerta en una máquina virtual

Se pueden configurar alertas para que avisen a los usuarios cuando una métrica determinada supere un umbral que se le ha asignado. En este ejemplo, la alerta se encuentra en los segmentos TCP enviados, pero se puede activar para muchas otras métricas. En este ejemplo, se configura una alerta para llamar a un webhook que llama a la función.

### <a name="create-the-alert-rule"></a>Creación de la regla de alerta

Vaya a una máquina virtual existente y agregue una regla de alerta. Se puede encontrar documentación más detallada sobre la configuración de alertas en [Creación de alertas en Azure Monitor para servicios de Azure (Azure Portal)](../azure-monitor/alerts/alerts-classic-portal.md). Escriba los siguientes valores en la hoja **Regla de alertas** y, a continuación, seleccione **Aceptar**.

  |**Configuración** | **Valor** | **Detalles** |
  |---|---|---|
  |**Nombre**|TCP_Segments_Sent_Exceeded|Nombre de la regla de alerta.|
  |**Descripción**|Los segmentos TCP enviados superaron el umbral|La descripción de la regla de alerta.|
  |**Métrica**|Segmentos TCP enviados| La métrica que se usará para desencadenar la alerta. |
  |**Condition**|Mayor que| La condición que se va a usar al evaluar la métrica.|
  |**Umbral**|100| El valor de la métrica que desencadena la alerta. Este valor debe establecerse en un valor válido para su entorno.|
  |**Período**|En los últimos cinco minutos| Determina el período en el que se va a buscar el umbral de la métrica.|
  |**Webhook**|[Dirección URL del webhook de la aplicación de función]| La dirección URL del webhook de la aplicación de función que se creó en los pasos anteriores.|

> [!NOTE]
> De manera predeterminada, la métrica de los segmentos TCP no está habilitada. Para más información acerca de cómo habilitar métricas adicionales, visite [Habilitación de supervisión y diagnóstico](../azure-monitor/overview.md).

## <a name="review-the-results"></a>Revisión del resultado

Después de que se cumplan los criterios que desencadenen la alerta, se creará una captura de paquetes. Vaya a Network Watcher y, a continuación, seleccione **Captura de paquetes**. En esta página, puede seleccionar el vínculo del archivo de captura de paquetes para descargar la captura de paquetes.

![Visualización de captura de paquetes][functions14]

Si el archivo de captura se almacena localmente, puede iniciar sesión en la máquina virtual para recuperarlo.

Para obtener instrucciones sobre cómo descargar archivos desde cuentas de Azure Storage, consulte [Introducción a Azure Blob Storage mediante .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md). Otra herramienta que puede usar es el [Explorador de Storage](https://storageexplorer.com/).

Después de que se ha descargado la captura, puede verla con cualquier herramienta que pueda leer un archivo **.cap**. A continuación se muestran vínculos a dos de estas herramientas:

- [Analizador de mensajes de Microsoft](/message-analyzer/microsoft-message-analyzer-operating-guide).
- [WireShark](https://www.wireshark.org/)

## <a name="next-steps"></a>Pasos siguientes

Para saber cómo ver las capturas de paquetes, visite [Análisis de capturas de paquetes con Wireshark](network-watcher-deep-packet-inspection.md).


[1]: ./media/network-watcher-alert-triggered-packet-capture/figure1.png
[1-1]: ./media/network-watcher-alert-triggered-packet-capture/figure1-1.png
[2]: ./media/network-watcher-alert-triggered-packet-capture/figure2.png
[3]: ./media/network-watcher-alert-triggered-packet-capture/figure3.png
[functions1]:./media/network-watcher-alert-triggered-packet-capture/functions1.png
[functions2]:./media/network-watcher-alert-triggered-packet-capture/functions2.png
[functions3]:./media/network-watcher-alert-triggered-packet-capture/functions3.png
[functions4]:./media/network-watcher-alert-triggered-packet-capture/functions4.png
[functions5]:./media/network-watcher-alert-triggered-packet-capture/functions5.png
[functions6]:./media/network-watcher-alert-triggered-packet-capture/functions6.png
[functions7]:./media/network-watcher-alert-triggered-packet-capture/functions7.png
[functions8]:./media/network-watcher-alert-triggered-packet-capture/functions8.png
[functions9]:./media/network-watcher-alert-triggered-packet-capture/functions9.png
[functions10]:./media/network-watcher-alert-triggered-packet-capture/functions10.png
[functions11]:./media/network-watcher-alert-triggered-packet-capture/functions11.png
[functions12]:./media/network-watcher-alert-triggered-packet-capture/functions12.png
[functions13]:./media/network-watcher-alert-triggered-packet-capture/functions13.png
[functions14]:./media/network-watcher-alert-triggered-packet-capture/functions14.png
[scenario]:./media/network-watcher-alert-triggered-packet-capture/scenario.png
