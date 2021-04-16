---
author: robinsh
ms.author: robinsh
ms.service: iot-hub
ms.topic: include
ms.date: 10/26/2018
ms.openlocfilehash: 26c158145de6ce729d8a7060152b19fb14b63d58
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95563406"
---
## <a name="prepare-to-authenticate-azure-resource-manager-requests"></a>Prepararse para autenticar solicitudes de Azure Resource Manager
Debe autenticar todas las operaciones que se realizan en los recursos mediante [Azure Resource Manager][lnk-authenticate-arm] con Azure Active Directory (AD). La manera más sencilla de configurar esto es usar PowerShell o CLI de Azure.

Instale los [cmdlets de Azure PowerShell][lnk-powershell-install] antes de continuar.

En los pasos siguientes se muestra cómo configurar la autenticación de contraseña para una aplicación de AD mediante PowerShell. Puede ejecutar estos comandos en una sesión de PowerShell estándar.

1. Inicie sesión en su suscripción a Azure con el siguiente comando:

    ```powershell
    Connect-AzAccount
    ```

1. Si tiene varias suscripciones de Azure, el inicio de sesión en Azure le concede acceso a todas las suscripciones de Azure asociadas a sus credenciales. Use el siguiente comando para mostrar las suscripciones de Azure que están disponibles para su uso:

    ```powershell
    Get-AzSubscription
    ```

    Use el siguiente comando para seleccionar la suscripción que desea usar para ejecutar los comandos para administrar su centro de IoT. Puede usar el nombre de la suscripción o el identificador de la salida del comando anterior:

    ```powershell
    Select-AzSubscription `
        -SubscriptionName "{your subscription name}"
    ```

2. Anote los valores de **TenantId** y **SubscriptionId**. Los necesitará más adelante.
3. Cree una nueva aplicación de Azure Active Directory con el siguiente comando, reemplazando los marcadores de posición:
   
   * **{Nombre para mostrar}**: nombre para mostrar de la aplicación, por ejemplo, **MySampleApp**.
   * **{Dirección URL de la página principal}:** la dirección URL de la página principal de la aplicación, por ejemplo, **http:\//mysampleapp/home**. Esta dirección URL no tiene que señalar a una aplicación real.
   * **{Identificador de aplicación}** : identificador único, por ejemplo, **http:\//mysampleapp**. Esta dirección URL no tiene que señalar a una aplicación real.
   * **{Contraseña}:** una contraseña que se usa para autenticarse en la aplicación.
     
     ```powershell
     $SecurePassword=ConvertTo-SecureString {password} –asplaintext –force
     New-AzADApplication -DisplayName {Display name} -HomePage {Home page URL} -IdentifierUris {Application identifier} -Password $SecurePassword
     ```
4. Anote el **ApplicationId** de la aplicación que ha creado. Lo necesitará más adelante.
5. Cree una nueva entidad de servicio con el comando siguiente, reemplazando **{MyApplicationId}** por el valor de **ApplicationId** del paso anterior:
   
    ```powershell
    New-AzADServicePrincipal -ApplicationId {MyApplicationId}
    ```
6. Configure una asignación de rol con el comando siguiente, reemplazando **{MyApplicationId}** por su valor de **ApplicationId**.
   
    ```powershell
    New-AzRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName {MyApplicationId}
    ```

Ahora ha terminado de crear la aplicación de Azure AD que le permitirá autenticarse desde su aplicación de C# personalizada. En este tutorial necesitará más adelante los siguientes valores:

* TenantId
* SubscriptionId
* ApplicationId
* Contraseña

[lnk-authenticate-arm]: /rest/api/
[lnk-powershell-install]: /powershell/azure/install-az-ps