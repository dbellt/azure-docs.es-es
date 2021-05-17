---
title: Inicio de sesión en Azure AD mediante el correo electrónico como Id. de inicio de sesión alternativo
description: Aprenda a permitir que los usuarios inicien sesión en Azure Active Directory con su dirección de correo electrónico como Id. de inicio de sesión alternativo
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 5/3/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: calui
ms.openlocfilehash: 222197249f9fa8c4a30a0361633229c1d6cc473a
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517593"
---
# <a name="sign-in-to-azure-ad-with-email-as-an-alternate-login-id-preview"></a>Inicio de sesión en Azure AD mediante el correo electrónico como Id. de inicio de sesión alternativo (versión preliminar)

> [!NOTE]
> El inicio de sesión en Azure AD con el correo electrónico como Id. de inicio de sesión alternativo es una característica de versión preliminar de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Muchas organizaciones desean que los usuarios puedan iniciar sesión en Azure Active Directory (Azure AD) con las mismas credenciales que su entorno de directorio local. Con este enfoque, conocido como autenticación híbrida, los usuarios solo necesitan recordar un conjunto de credenciales.

Algunas organizaciones no han migrado a la autenticación híbrida por las siguientes razones:

* De manera predeterminada, el nombre principal de usuario (UPN) de Azure AD se establece en el mismo valor que el UPN local.
* Al cambiar el UPN de Azure AD, se produce un desajuste entre los entornos local y de Azure AD que podría ocasionar problemas con determinadas aplicaciones y servicios.
* Debido a motivos empresariales o de cumplimiento, la organización no desea usar el UPN local para iniciar sesión en Azure.

Para facilitar la adopción de la autenticación híbrida, puede configurar Azure AD para que los usuarios puedan iniciar sesión con su correo electrónico como Id. de inicio de sesión alternativo. Por ejemplo, si *Contoso* cambia de nombre a *Fabrikam*, en lugar de seguir iniciando sesión con el UPN heredado `balas@contoso.com`, se puede usar el correo electrónico como Id. de inicio de sesión alternativo. Para acceder a una aplicación o un servicio, los usuarios iniciarían sesión en Azure AD con su correo electrónico que no es UPN, como `balas@fabrikam.com`.

En este artículo se muestra cómo habilitar y usar el correo electrónico como id. de inicio de sesión alternativo.

## <a name="before-you-begin"></a>Antes de empezar

Esto es lo que necesita saber sobre el correo electrónico como Id. de inicio de sesión alternativo:

* La característica está disponible en la edición Azure AD Free y superiores.
* La característica habilita el inicio de sesión con *proxyAddresses* de dominio comprobadas de usuarios de Azure AD autenticados para la nube.
* Cuando un usuario inicia sesión con un correo electrónico que no es UPN, las notificaciones `unique_name` y `preferred_username` (si están presentes) del [Token de identificador](https://docs.microsoft.com/azure/active-directory/develop/id-tokens) tienen el valor del correo electrónico que no es UPN.
* Hay dos opciones para configurar la característica:
    * [Directiva de detección del dominio de inicio (HRD)](#enable-user-sign-in-with-an-email-address): use esta opción para habilitar la característica para todo el inquilino. Se necesitan privilegios de administrador global.
    * [Directiva de lanzamiento preconfigurado](#enable-staged-rollout-to-test-user-sign-in-with-an-email-address): use esta opción para probar la característica con grupos de Azure AD específicos. Se necesitan privilegios de administrador global.

## <a name="preview-limitations"></a>Limitaciones de vista previa

En el estado de versión preliminar actual, se aplican las siguientes limitaciones al correo electrónico como Id.de inicio de sesión alternativo:

* Los usuarios pueden ver su UPN aunque hayan iniciado sesión con su correo electrónico que no es UPN. Se puede observar el siguiente comportamiento de ejemplo:
    * Al usuario se le solicita que inicie sesión con su UPN cuando se le dirige al inicio de sesión de Azure AD con `login_hint=<non-UPN email>`.
    * Cuando un usuario inicia sesión con un correo electrónico que no es UPN y escribe una contraseña incorrecta, la página *"Escriba la contraseña"* cambia para mostrar el UPN.
    * En algunos sitios y aplicaciones de Microsoft, como Microsoft Office, el control **Administrador de cuentas** que normalmente se muestra en la esquina superior derecha puede mostrar el UPN del usuario en lugar del correo electrónico que no es UPN usado para iniciar sesión.

* Actualmente, algunos flujos no son compatibles con los correos electrónicos que no son UPN, como los siguientes:
    * Identity Protection no empareja los correos electrónicos que no son UPN con detección de riesgo *Credenciales filtradas*. Esta detección de riesgo usa el UPN para asociar las credenciales que se han filtrado. Para obtener más información, consulte [Detección y corrección de riesgos de Azure AD Identity Protection][identity-protection].
    * Las invitaciones B2B enviadas a un correo electrónico que no es UPN no son totalmente compatibles. Después de aceptar una invitación enviada a un correo electrónico que no es UPN, es posible que el inicio de sesión con el correo electrónico que no es UPN no funcione para el usuario invitado en el punto de conexión del inquilino del recurso.
    * Cuando un usuario ha iniciado sesión con un correo electrónico que no es UPN, no puede cambiar su contraseña. El autoservicio de restablecimiento de contraseña (SSPR) de Azure AD debe funcionar según lo previsto. Durante SSPR, el usuario puede ver su UPN si comprueba su identidad por medio de un correo electrónico alternativo.

* Los siguientes escenarios no se admiten. Inicie sesión con un correo electrónico que no es UPN en:
    * Dispositivos híbridos unidos a Azure AD
    * Dispositivos unidos a Azure AD
    * Skype Empresarial
    * Microsoft Office en macOS
    * OneDrive (cuando el flujo de inicio de sesión no implique autenticación multifactor)
    * Microsoft Teams en Internet
    * Flujos de credenciales de contraseña del propietario del recurso (ROPC)

* Los cambios realizados en la configuración de la característica en la directiva de HRD no se muestran de manera explícita en los registros de auditoría.
* La directiva de lanzamiento preconfigurado no funciona según lo previsto con los usuarios que se han incluido en varias directivas de lanzamiento preconfigurado.
* Dentro de un inquilino, el UPN de un usuario solo en la nube puede ser el mismo valor que la dirección proxy de otro usuario sincronizada desde el directorio local. En este escenario, con la característica habilitada, el usuario solo en la nube no puede iniciar sesión con su UPN. Hay más información sobre este problema en la sección [Solución de problemas](#troubleshoot).

## <a name="overview-of-alternate-login-id-options"></a>Información general sobre las opciones de Id. de inicio de sesión alternativo

Para iniciar sesión en Azure AD, los usuarios escriben un valor que identifica su cuenta de forma única. Antes, solo se podía usar el UPN de Azure AD como identificador de inicio de sesión.

En el caso de las organizaciones en las que el UPN local es el correo electrónico de inicio de sesión elegido por el usuario, este era un enfoque óptimo. Esas organizaciones configuraban el UPN de Azure AD con el mismo valor que el UPN local, y los usuarios tenían una experiencia de inicio de sesión coherente.

### <a name="alternate-login-id-for-ad-fs"></a>Id. de inicio de sesión alternativo para AD FS

Pero, en algunas organizaciones, el UPN local no se usa como identificador de inicio de sesión. En los entornos locales, se configuraría AD DS local para permitir el inicio de sesión con un Id. de inicio de sesión alternativo. El establecimiento del UPN de Azure AD en el mismo valor que el UPN local no es una opción, ya que Azure AD entonces exigiría que los usuarios iniciaran sesión con ese valor.

### <a name="alternate-login-id-in-azure-ad-connect"></a>Id. de inicio de sesión alternativo de Azure AD Connect

La solución habitual a este problema era configurar el UPN de Azure AD con la dirección de correo electrónico con la que el usuario espera iniciar sesión. Este enfoque funciona, aunque genera distintos UPN entre la instancia de AD local y Azure AD, y esta configuración no es compatible con todas las cargas de trabajo de Microsoft 365.

### <a name="email-as-an-alternate-login-id"></a>Correo electrónico como Id. de inicio de sesión alternativo

Un enfoque diferente consiste en sincronizar los UPN de Azure AD y AD local con el mismo valor y, a continuación, configurar Azure AD para permitir que los usuarios inicien sesión con un correo electrónico verificado. Para ello, defina una o más direcciones de correo electrónico en el atributo *ProxyAddresses* del usuario en el directorio local. El atributo *ProxyAddresses* se sincroniza en Azure AD automáticamente mediante Azure AD Connect.


| Opción | Descripción |
|---|---|
| [Id. de inicio de sesión alternativo para AD FS](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configuring-alternate-login-id) | Habilite el inicio de sesión con un atributo alternativo (como Mail) para los usuarios de AD FS. |
| [Id. de inicio de sesión alternativo de Azure AD Connect](https://docs.microsoft.com/azure/active-directory/hybrid/plan-connect-userprincipalname#alternate-login-id) | Sincronice un atributo alternativo (como Mail) como UPN de Azure AD. |
| Correo electrónico como Id. de inicio de sesión alternativo | Habilite el inicio de sesión con *proxyAddresses* de dominio comprobadas para los usuarios de Azure AD. |

## <a name="synchronize-sign-in-email-addresses-to-azure-ad"></a>Sincronizar las direcciones de correo electrónico de inicio de sesión con Azure AD

La autenticación tradicional de Active Directory Domain Services (AD DS) o Servicios de federación de Active Directory (AD FS) se produce directamente en la red y se controla mediante la infraestructura de AD DS. Con la autenticación híbrida, los usuarios pueden iniciar sesión directamente en Azure AD.

Para admitir este enfoque de autenticación híbrida, debe sincronizar el entorno de AD DS local con Azure AD mediante [Azure AD Connect][azure-ad-connect] y configurarlo para que use la sincronización de hash de contraseñas (PHS) o la autenticación de paso a través (PTA). Para más información, consulte [Selección del método de autenticación adecuado para la solución de identidad híbrida de Azure AD][hybrid-auth-methods].

En ambas opciones de configuración, el usuario envía su nombre de usuario y contraseña a Azure AD, que valida las credenciales y emite un vale. Cuando los usuarios inician sesión en Azure AD, se elimina la necesidad de que su organización albergue y administre una infraestructura de AD FS.

Uno de los atributos de usuario que se sincroniza automáticamente mediante Azure AD Connect es *ProxyAddresses*. Si los usuarios tienen una dirección de correo electrónico definida en el entorno de AD DS local como parte del atributo *ProxyAddresses*, se sincroniza automáticamente con Azure AD. Esta dirección de correo electrónico se puede usar directamente en el proceso de inicio de sesión de Azure AD como identificador de inicio de sesión alternativo.

> [!IMPORTANT]
> Solo los mensajes de correo electrónico de los dominios comprobados del inquilino se sincronizan con Azure AD. Cada inquilino de Azure AD tiene uno o varios dominios comprobados, para los que se ha comprobado la propiedad, y se enlazan de forma exclusiva con el inquilino.
>
> Para más información, consulte [Agregar y comprobar un nombre de dominio personalizado en Azure AD][verify-domain].

## <a name="enable-user-sign-in-with-an-email-address"></a>Habilitar el inicio de sesión de usuario con una dirección de correo electrónico

> [!NOTE]
> Esta opción de configuración usa la directiva de HRD. Para obtener más información, vea [Tipo de recurso homeRealmDiscoveryPolicy](/graph/api/resources/homeRealmDiscoveryPolicy?view=graph-rest-1.0&preserve-view=true).

Una vez que los usuarios con el atributo *ProxyAddresses* aplicado se sincronizan con Azure AD mediante Azure AD Connect, necesita habilitar la función para que los usuarios se registren con el correo electrónico como un ID de inicio de sesión alternativo para su inquilino. Esta característica indica a los servidores de inicio de sesión de Azure AD que no solo comprueben el identificador de inicio de sesión con los valores de UPN, sino también con los valores *ProxyAddresses* de la dirección de correo electrónico.

Durante la versión preliminar, actualmente solo puede habilitar el inicio de sesión con correo electrónico como una característica de identificador de inicio de sesión alternativo mediante PowerShell. Necesita permisos de *administrador global* para realizar los siguientes pasos:

1. Abra una sesión de PowerShell como administrador e instale el módulo *AzureADPreview* con el cmdlet [Install-Module][Install-Module]:

    ```powershell
    Install-Module AzureADPreview
    ```

    Si se le solicita, seleccione **Y** para instalar NuGet o para instalar desde un repositorio que no es de confianza.

1. Inicie sesión en el inquilino de Azure AD como *administrador global* mediante el cmdlet [Connect-AzureAD][Connect-AzureAD]:

    ```powershell
    Connect-AzureAD
    ```

    El comando devuelve información acerca de su cuenta, entorno e id. de inquilino.

1. Compruebe si *HomeRealmDiscoveryPolicy* ya existe en el inquilino con el cmdlet [Get-AzureADPolicy][Get-AzureADPolicy] como se indica a continuación:

    ```powershell
    Get-AzureADPolicy | Where-Object Type -eq "HomeRealmDiscoveryPolicy" | Format-List *
    ```

1. Si no hay ninguna directiva configurada actualmente, el comando no devuelve nada. Si se devuelve una directiva, omita este paso y vaya al paso siguiente para actualizar una directiva existente.

    Para agregar la directiva de *HomeRealmDiscoveryPolicy* al inquilino, use el cmdlet [New-AzureADPolicy][New-AzureADPolicy] y establezca el atributo *AlternateIdLogin* en *"Enabled": true* como se muestra en el siguiente ejemplo:

    ```powershell
    $AzureADPolicyDefinition = @(
      @{
         "HomeRealmDiscoveryPolicy" = @{
            "AlternateIdLogin" = @{
               "Enabled" = $true
            }
         }
      } | ConvertTo-JSON -Compress
    )
    $AzureADPolicyParameters = @{
      Definition            = $AzureADPolicyDefinition
      DisplayName           = "BasicAutoAccelerationPolicy"
      IsOrganizationDefault = $true
      Type                  = "HomeRealmDiscoveryPolicy"
    }
    New-AzureADPolicy @AzureADPolicyParameters
    ```

    Cuando la política se haya creado con éxito, el comando devuelve el Id. de la política, como se muestra en la siguiente salida de ejemplo:

    ```powershell
    Id                                   DisplayName                 Type                     IsOrganizationDefault
    --                                   -----------                 ----                     ---------------------
    5de3afbe-4b7a-4b33-86b0-7bbe308db7f7 BasicAutoAccelerationPolicy HomeRealmDiscoveryPolicy True
    ```

1. Si ya hay una política configurada, compruebe si el atributo *AlternateIdLogin*  está habilitado, como se muestra en la siguiente salida directiva de ejemplo:

    ```powershell
    Id : 5de3afbe-4b7a-4b33-86b0-7bbe308db7f7
    OdataType :
    AlternativeIdentifier :
    Definition : {{"HomeRealmDiscoveryPolicy" :{"AlternateIdLogin":{"Enabled": true}}}}
    DisplayName : BasicAutoAccelerationPolicy
    IsOrganizationDefault : True
    KeyCredentials : {}
    Type : HomeRealmDiscoveryPolicy
    ```

    Si la directiva existe pero el atributo *AlternateIdLogin* no está presente o habilitado, o si existen otros atributos en la directiva que desea conservar, actualice la directiva existente con el cmdlet [Set-AzureADPolicy][Set-AzureADPolicy].

    > [!IMPORTANT]
    > Al actualizar la directiva, asegúrese de que incluye cualquier configuración anterior y el nuevo atributo *AlternateIdLogin*.

    En el ejemplo siguiente se agrega el atributo *AlternateIdLogin* y se conserva el atributo  *AllowCloudPasswordValidation* que se puede haber establecido:

    ```powershell
    $AzureADPolicyDefinition = @(
      @{
         "HomeRealmDiscoveryPolicy" = @{
            "AllowCloudPasswordValidation" = $true
            "AlternateIdLogin" = @{
               "Enabled" = $true
            }
         }
      } | ConvertTo-JSON -Compress
    )
    $AzureADPolicyParameters = @{
      ID                    = "b581c39c-8fe3-4bb5-b53d-ea3de05abb4b"
      Definition            = $AzureADPolicyDefinition
      DisplayName           = "BasicAutoAccelerationPolicy"
      IsOrganizationDefault = $true
      Type                  = "HomeRealmDiscoveryPolicy"
    }
    
    Set-AzureADPolicy @AzureADPolicyParameters
    ```

    Confirme que la directiva actualizada muestra los cambios y que el atributo *AlternateIdLogin* está habilitado ahora:

    ```powershell
    Get-AzureADPolicy | Where-Object Type -eq "HomeRealmDiscoveryPolicy" | Format-List *
    ```

Con la directiva aplicada, puede tardar hasta una hora en propagarse para que los usuarios puedan iniciar sesión con su Id. de inicio de sesión alternativo.

## <a name="enable-staged-rollout-to-test-user-sign-in-with-an-email-address"></a>Habilitación del lanzamiento preconfigurado para probar el inicio de sesión de usuario con una dirección de correo electrónico  

> [!NOTE]
>Esta opción de configuración usa la directiva de lanzamiento preconfigurado. Para obtener más información, vea [Tipo de recurso featureRolloutPolicy](https://docs.microsoft.com/graph/api/resources/featurerolloutpolicy?view=graph-rest-1.0&preserve-view=true).

La directiva de lanzamiento preconfigurado permite a los administradores de inquilinos habilitar características para grupos de Azure AD específicos. Se recomienda que los administradores de inquilinos usen esta característica para probar el inicio de sesión de usuario con una dirección de correo electrónico. Cuando los administradores están listos para implementar esta característica en todo el inquilino, deben usar la [directiva de HDR](#enable-user-sign-in-with-an-email-address).  


Necesita permisos de *administrador global* para realizar los siguientes pasos:

1. Abra una sesión de PowerShell como administrador e instale el módulo *AzureADPreview* con el cmdlet [Install-Module][Install-Module]:

    ```powershell
    Install-Module AzureADPreview
    ```

    Si se le solicita, seleccione **Y** para instalar NuGet o para instalar desde un repositorio que no es de confianza.

1. Inicie sesión en el inquilino de Azure AD como *administrador global* mediante el cmdlet [Connect-AzureAD][Connect-AzureAD]:

    ```powershell
    Connect-AzureAD
    ```

    El comando devuelve información acerca de su cuenta, entorno e id. de inquilino.

1. Enumere todas las directivas de lanzamiento preconfigurado existentes con el siguiente cmdlet:
   
   ```powershell
   Get-AzureADMSFeatureRolloutPolicy
   ``` 

1. Si no hay directivas de lanzamiento preconfigurado para esta característica, cree una nueva directiva y tome nota de su identificador:

   ```powershell
   $AzureADMSFeatureRolloutPolicy = @{
      Feature    = "EmailAsAlternateId"
      DisplayName = "EmailAsAlternateId Rollout Policy"
      IsEnabled   = $true
   }
   New-AzureADMSFeatureRolloutPolicy @AzureADMSFeatureRolloutPolicy
   ```

1. Busque el identificador de directoryObject correspondiente al grupo que se va a agregar a la directiva de lanzamiento preconfigurado. Tome nota del valor devuelto para el parámetro *Id*, porque se usará en el paso siguiente.
   
   ```powershell
   Get-AzureADMSGroup -SearchString "Name of group to be added to the staged rollout policy"
   ```

1. Agregue el grupo a la directiva de lanzamiento preconfigurado, tal como se muestra en el ejemplo siguiente. Reemplace el valor del parámetro *-Id* por el valor devuelto de identificador de directiva en el paso 4 y reemplace el valor del parámetro *-RefObjectId* por el valor de *Id* anotado en el paso 5. Los usuarios del grupo pueden tardar hasta una hora en poder iniciar sesión en Azure AD con el correo electrónico como Id. de inicio de sesión alternativo.

   ```powershell
   Add-AzureADMSFeatureRolloutPolicyDirectoryObject -Id "ROLLOUT_POLICY_ID&quot; -RefObjectId &quot;GROUP_OBJECT_ID"
   ```
   
En el caso de los nuevos miembros agregados al grupo, pueden tardar hasta 24 horas en poder iniciar sesión en Azure AD con el correo electrónico como Id. de inicio de sesión alternativo.

### <a name="removing-groups"></a>Eliminación de grupos

Para quitar un grupo de una directiva de lanzamiento preconfigurado, ejecute el siguiente comando:

```powershell
Remove-AzureADMSFeatureRolloutPolicyDirectoryObject -Id "ROLLOUT_POLICY_ID&quot; -ObjectId &quot;GROUP_OBJECT_ID" 
```

### <a name="removing-policies"></a>Eliminación de directivas

Para quitar una directiva de lanzamiento preconfigurado, deshabilite primero la directiva y después quítela del sistema:

```powershell
Set-AzureADMSFeatureRolloutPolicy -Id "ROLLOUT_POLICY_ID" -IsEnabled $false 
Remove-AzureADMSFeatureRolloutPolicy -Id "ROLLOUT_POLICY_ID"
```

## <a name="test-user-sign-in-with-an-email-address"></a>Prueba del inicio de sesión de usuario con una dirección de correo electrónico

Para probar que los usuarios pueden iniciar sesión con el correo electrónico, vaya a [https://myprofile.microsoft.com][my-profile] e inicie sesión con un correo electrónico que no es UPN, como `balas@fabrikam.com`. La experiencia de inicio de sesión debe ser idéntica al inicio de sesión con el UPN.

## <a name="troubleshoot"></a>Solución de problemas

Si los usuarios tienen problemas para iniciar sesión con su dirección de correo electrónico, revise los siguientes pasos de solución de problemas:

1. Asegúrese de que ha pasado al menos una hora desde que se ha habilitado el correo electrónico como Id. de inicio de sesión alternativo. Si el usuario se ha agregado recientemente a un grupo de la directiva de lanzamiento preconfigurado, asegúrese de que han pasado al menos 24 horas desde que se hizo esto.
1. Si usa la directiva de HRD, confirme que el atributo *HomeRealmDiscoveryPolicy* de Azure AD tiene la propiedad de definición *AlternateIdLogin* establecida en *"Enabled": true* y la propiedad *IsOrganizationDefault* establecida en *True*:

    ```powershell
    Get-AzureADPolicy | Where-Object Type -eq "HomeRealmDiscoveryPolicy" | Format-List *
    ```
    Si usa la directiva de lanzamiento preconfigurado, confirme que el atributo *FeatureRolloutPolicy* de Azure AD tiene la propiedad *IsEnabled* establecida en *True*:

    ```powershell
    Get-AzureADMSFeatureRolloutPolicy
    ```
1. Asegúrese de que la cuenta de usuario tiene su dirección de correo electrónico establecida en el atributo *ProxyAddresses* de Azure AD.

### <a name="conflicting-values-between-cloud-only-and-synced-users"></a>Valores en conflicto entre usuarios sincronizados y solo en la nube

Dentro de un inquilino, el UPN de un usuario solo en la nube puede tomar el mismo valor que la dirección proxy de otro usuario sincronizada desde el directorio local. En este escenario, con la característica habilitada, el usuario solo en la nube no puede iniciar sesión con su UPN. Estos son los pasos para detectar instancias de este problema.

1. Abra una sesión de PowerShell como administrador e instale el módulo *AzureADPreview* con el cmdlet [Install-Module][Install-Module]:

    ```powershell
    Install-Module AzureADPreview
    ```

    Si se le solicita, seleccione **Y** para instalar NuGet o para instalar desde un repositorio que no es de confianza.

1. Inicie sesión en el inquilino de Azure AD como *administrador global* mediante el cmdlet [Connect-AzureAD][Connect-AzureAD]:

    ```powershell
    Connect-AzureAD
    ```

1. Obtenga los usuarios afectados.

    ```powershell
    # Get all users
    $allUsers = Get-AzureADUser -All $true
    
    # Get list of proxy addresses from all synced users
    $syncedProxyAddresses = $allUsers |
        Where-Object {$_.ImmutableId} |
        Select-Object -ExpandProperty ProxyAddresses |
        ForEach-Object {$_ -Replace "smtp:", ""}
    
    # Get list of user principal names from all cloud-only users
    $cloudOnlyUserPrincipalNames = $allUsers |
        Where-Object {!$_.ImmutableId} |
        Select-Object -ExpandProperty UserPrincipalName
    
    # Get intersection of two lists
    $duplicateValues = $syncedProxyAddresses |
        Where-Object {$cloudOnlyUserPrincipalNames -Contains $_}
    ``` 

1. Para generar los usuarios afectados:

    ```powershell
    # Output affected synced users
    $allUsers |
        Where-Object {$_.ImmutableId -And ($_.ProxyAddresses | Where-Object {($duplicateValues | ForEach-Object {"smtp:$_"}) -Contains $_}).Length -GT 0} |
        Select-Object ObjectId, DisplayName, UserPrincipalName, ProxyAddresses, ImmutableId, UserType
    
    # Output affected cloud-only users
    $allUsers |
        Where-Object {!$_.ImmutableId -And $duplicateValues -Contains $_.UserPrincipalName} |
        Select-Object ObjectId, DisplayName, UserPrincipalName, ProxyAddresses, ImmutableId, UserType
    ```

1. Para generar los usuarios afectados en CSV:

    ```powershell
    # Output affected users to CSV
    $allUsers |
        Where-Object {
            ($_.ImmutableId -And ($_.ProxyAddresses | Where-Object {($duplicateValues | ForEach-Object {"smtp:$_"}) -Contains $_}).Length -GT 0) -Or
            (!$_.ImmutableId -And $duplicateValues -Contains $_.UserPrincipalName)
        } |
        Select-Object ObjectId, DisplayName, UserPrincipalName, @{n="ProxyAddresses"; e={$_.ProxyAddresses -Join ','}}, @{n="IsSyncedUser"; e={$_.ImmutableId.Length -GT 0}}, UserType |
        Export-Csv -Path .\AffectedUsers.csv -NoTypeInformation
    ```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la identidad híbrida, como Proxy de aplicación de Azure AD o Azure AD Domain Services, consulte [Identidad híbrida Azure AD para el acceso y la gestión de las cargas de trabajo locales][hybrid-overview].

Para obtener más información sobre las operaciones de identidad híbrida, consulte cómo funciona [Sincronización hash de contraseña][phs-overview] o [Autenticación de paso a través][pta-overview].

<!-- INTERNAL LINKS -->
[verify-domain]: ../fundamentals/add-custom-domain.md
[hybrid-auth-methods]: ../hybrid/choose-ad-authn.md
[azure-ad-connect]: ../hybrid/whatis-azure-ad-connect.md
[hybrid-overview]: ../hybrid/cloud-governed-management-for-on-premises.md
[phs-overview]: ../hybrid/how-to-connect-password-hash-synchronization.md
[pta-overview]: ../hybrid/how-to-connect-pta-how-it-works.md
[identity-protection]: ../identity-protection/overview-identity-protection.md#risk-detection-and-remediation

<!-- EXTERNAL LINKS -->
[Install-Module]: /powershell/module/powershellget/install-module
[Connect-AzureAD]: /powershell/module/azuread/connect-azuread
[Get-AzureADPolicy]: /powershell/module/azuread/get-azureadpolicy
[New-AzureADPolicy]: /powershell/module/azuread/new-azureadpolicy
[Set-AzureADPolicy]: /powershell/module/azuread/set-azureadpolicy
[my-profile]: https://myprofile.microsoft.com
